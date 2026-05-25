# 35 — Introdução ao Spring Security

tags: #springboot #segurança #springsecurity
links: [[36 - Autenticação vs Autorização]] | [[37 - JWT Conceito e Implementação]] | [[38 - Filtros de Segurança]] | [[🗺️ Mapa Principal]]

---

## O que é Spring Security

**Spring Security** é o módulo de segurança do ecossistema Spring. Ele fornece:

- **Autenticação** — quem é você? (login)
- **Autorização** — o que você pode fazer? (permissões)
- **Proteção contra ataques** — CSRF, clickjacking, session fixation, etc.

É um dos frameworks de segurança mais completos para Java, e praticamente obrigatório em qualquer API Spring Boot de produção.

---

## O que acontece ao adicionar Spring Security

Ao adicionar `spring-boot-starter-security` ao projeto, o Spring Boot automaticamente:

1. Bloqueia **todos** os endpoints (exige autenticação)
2. Habilita um formulário de login padrão em `/login`
3. Gera uma senha aleatória no console (para o usuário padrão `user`)
4. Habilita proteção CSRF
5. Configura headers de segurança HTTP

```
Console ao subir a aplicação:
Using generated security password: 3f5d7a2b-9c1e-4f8a-b2d6-0e7f9a3c5b1d

Isso é temporário — você vai sobrescrever com sua própria configuração.
```

---

## O modelo de segurança: Filter Chain

O Spring Security funciona como uma **cadeia de filtros** que intercepta cada requisição HTTP antes de chegar ao controller:

```mermaid
flowchart LR
    C([Requisição HTTP]) --> F1[CorsFilter]
    F1 --> F2[SecurityContextHolder\nFilter]
    F2 --> F3[JwtAuthFilter\n(seu filtro)]
    F3 --> F4[UsernamePassword\nAuthFilter]
    F4 --> F5[ExceptionTranslation\nFilter]
    F5 --> F6[AuthorizationFilter]
    F6 --> D([Controller])

    style F3 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F6 fill:#FAEEDA,stroke:#854F0B,color:#633806
```

Cada filtro decide: passa para o próximo, ou bloqueia a requisição.

---

## Dependências necessárias

```xml
<!-- pom.xml -->

<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- JWT — biblioteca jjwt -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.3</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.3</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.3</version>
    <scope>runtime</scope>
</dependency>

<!-- Para testes de segurança -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

---

## SecurityConfig — a classe central

```java
package com.empresa.api.config;

import com.empresa.api.security.JwtAuthFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity   // habilita @PreAuthorize, @PostAuthorize nos métodos
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final UserDetailsServiceImpl userDetailsService;

    public SecurityConfig(JwtAuthFilter jwtAuthFilter,
                          UserDetailsServiceImpl userDetailsService) {
        this.jwtAuthFilter = jwtAuthFilter;
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            // 1. Desabilita CSRF — APIs REST são stateless, CSRF não se aplica
            .csrf(AbstractHttpConfigurer::disable)

            // 2. Define regras de acesso por URL
            .authorizeHttpRequests(auth -> auth
                // Endpoints públicos — sem autenticação
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/produtos/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/api-docs/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()

                // Endpoints por role
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/**").hasRole("ADMIN")

                // Todo o restante exige autenticação
                .anyRequest().authenticated()
            )

            // 3. Stateless — sem sessão HTTP (APIs REST com JWT)
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )

            // 4. Configura o provider de autenticação
            .authenticationProvider(authenticationProvider())

            // 5. Adiciona o filtro JWT antes do filtro padrão
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)

            // 6. Handlers de erro de autenticação/autorização
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint((request, response, authException) -> {
                    // 401 quando não autenticado
                    response.setContentType("application/json");
                    response.setStatus(401);
                    response.getWriter().write("""
                        {"status":401,"erro":"Não autenticado","mensagem":"Token ausente ou inválido"}
                        """);
                })
                .accessDeniedHandler((request, response, accessDeniedException) -> {
                    // 403 quando autenticado mas sem permissão
                    response.setContentType("application/json");
                    response.setStatus(403);
                    response.getWriter().write("""
                        {"status":403,"erro":"Acesso negado","mensagem":"Você não tem permissão para esta operação"}
                        """);
                })
            )
            .build();
    }

    // Bean para codificar senhas — BCrypt é o padrão da indústria
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
        // BCrypt aplica salt automático e é resistente a brute force
    }

    // Provider que usa UserDetailsService para buscar o usuário
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    // AuthenticationManager — usado pelo AuthController para autenticar login
    @Bean
    public AuthenticationManager authenticationManager(
        AuthenticationConfiguration config
    ) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

## UserDetailsService — como o Spring carrega o usuário

```java
package com.empresa.api.security;

import com.empresa.api.usuario.UsuarioRepository;
import org.springframework.security.core.userdetails.*;
import org.springframework.stereotype.Service;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UsuarioRepository usuarioRepository;

    public UserDetailsServiceImpl(UsuarioRepository usuarioRepository) {
        this.usuarioRepository = usuarioRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        // Spring chama este método durante o login para buscar o usuário
        return usuarioRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "Usuário não encontrado com e-mail: " + email
            ));
        // Retorna UserDetails — sua entidade Usuario deve implementar esta interface
    }
}
```

---

## Entidade Usuario implementando UserDetails

```java
package com.empresa.api.usuario;

import jakarta.persistence.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.*;

@Entity
@Table(name = "usuarios")
public class Usuario implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 150)
    private String nome;

    @Column(nullable = false, unique = true, length = 200)
    private String email;

    @Column(name = "senha_hash", nullable = false)
    private String senhaHash;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private Role role = Role.USER;

    @Column(nullable = false)
    private boolean ativo = true;

    protected Usuario() {}

    public Usuario(String nome, String email, String senhaHash, Role role) {
        this.nome = nome;
        this.email = email;
        this.senhaHash = senhaHash;
        this.role = role;
    }

    // ===== IMPLEMENTAÇÃO DE UserDetails =====

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // Converte a Role para a autoridade que o Spring Security entende
        return List.of(new SimpleGrantedAuthority("ROLE_" + role.name()));
        // Ex: ROLE_USER, ROLE_ADMIN
    }

    @Override
    public String getPassword() {
        return senhaHash;  // hash da senha — nunca a senha em texto puro
    }

    @Override
    public String getUsername() {
        return email;  // usamos email como "username"
    }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return true; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return ativo; }

    // Getters adicionais
    public Long getId() { return id; }
    public String getNome() { return nome; }
    public String getEmail() { return email; }
    public Role getRole() { return role; }
}
```

```java
// Enum de roles
public enum Role {
    USER,
    ADMIN,
    MODERADOR
}
```

---

## Próximas notas
- [[36 - Autenticação vs Autorização]] — diferenças e implementação
- [[37 - JWT Conceito e Implementação]] — o token JWT completo
- [[38 - Filtros de Segurança]] — o filtro JWT na cadeia
