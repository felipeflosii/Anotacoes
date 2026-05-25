# 38 — Filtros de Segurança

tags: #springboot #segurança #filtros #jwt
links: [[35 - Introdução ao Spring Security]] | [[37 - JWT Conceito e Implementação]] | [[🗺️ Mapa Principal]]

---

## O que é o JwtAuthFilter

O **JwtAuthFilter** é um filtro na cadeia do Spring Security que intercepta **cada requisição HTTP** e:

1. Verifica se tem o header `Authorization: Bearer <token>`
2. Extrai e valida o token JWT
3. Se válido, carrega o usuário e registra no `SecurityContext`
4. Passa a requisição adiante na cadeia

```
Requisição → JwtAuthFilter → (demais filtros) → Controller
                ↓
         Sem token ou inválido → 401 (JwtAuthFilter não bloqueia,
                                       o AuthorizationFilter bloqueia depois)
                ↓
         Com token válido → registra usuário no SecurityContext
```

---

## Implementação completa do JwtAuthFilter

```java
package com.empresa.api.security;

import com.empresa.api.usuario.UsuarioRepository;
import io.jsonwebtoken.*;
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
@Slf4j
// OncePerRequestFilter garante que o filtro executa UMA VEZ por requisição
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    public JwtAuthFilter(JwtService jwtService, UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain        // restante da cadeia de filtros
    ) throws ServletException, IOException {

        // 1. Extrair o header Authorization
        final String authHeader = request.getHeader("Authorization");

        // 2. Se não tem header ou não começa com "Bearer ", passa adiante sem autenticar
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
            // O AuthorizationFilter mais à frente vai barrar se o endpoint exigir auth
        }

        // 3. Extrair o token (remove "Bearer ")
        final String token = authHeader.substring(7);

        try {
            // 4. Extrair o email/subject do token
            final String email = jwtService.extrairEmail(token);

            // 5. Se email existe e não há autenticação no contexto ainda
            if (email != null && SecurityContextHolder.getContext().getAuthentication() == null) {

                // 6. Carregar o usuário do banco
                var userDetails = userDetailsService.loadUserByUsername(email);

                // 7. Validar o token (assinatura + expiração)
                if (jwtService.isTokenValido(token, userDetails)) {

                    // 8. Criar objeto de autenticação
                    var authToken = new UsernamePasswordAuthenticationToken(
                        userDetails,    // principal — o objeto Usuario
                        null,           // credentials — null (não precisamos da senha aqui)
                        userDetails.getAuthorities()  // roles/permissões
                    );

                    // 9. Adicionar detalhes da requisição
                    authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                    );

                    // 10. Registrar no SecurityContext — "este usuário está autenticado"
                    SecurityContextHolder.getContext().setAuthentication(authToken);

                    log.debug("Usuário '{}' autenticado via JWT", email);
                }
            }

        } catch (ExpiredJwtException e) {
            log.debug("Token JWT expirado para: {}", request.getRequestURI());
            // Não retorna erro aqui — o AuthorizationFilter cuidará disso
            // Se o endpoint for público, a requisição passa mesmo assim

        } catch (MalformedJwtException | SignatureException e) {
            log.warn("Token JWT inválido em {}: {}", request.getRequestURI(), e.getMessage());

        } catch (Exception e) {
            log.error("Erro ao processar token JWT: {}", e.getMessage());
        }

        // 11. Continua a cadeia de filtros SEMPRE (mesmo que a auth falhe)
        filterChain.doFilter(request, response);
    }

    // Rotas que devem pular este filtro completamente
    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        String path = request.getServletPath();
        return path.startsWith("/api/v1/auth/") ||
               path.startsWith("/swagger-ui/") ||
               path.startsWith("/api-docs/") ||
               path.equals("/actuator/health");
    }
}
```

---

## O SecurityContext — onde a autenticação vive

```java
// O SecurityContextHolder mantém a autenticação do usuário atual (por thread)

// Lendo o usuário autenticado em qualquer lugar:
Authentication auth = SecurityContextHolder.getContext().getAuthentication();

if (auth != null && auth.isAuthenticated()) {
    Usuario usuario = (Usuario) auth.getPrincipal();
    String email = usuario.getEmail();
    Role role = usuario.getRole();
}

// No Controller — mais elegante com @AuthenticationPrincipal:
@GetMapping("/meu-perfil")
public ResponseEntity<UsuarioResponse> meuPerfil(
    @AuthenticationPrincipal Usuario usuarioLogado
) {
    return ResponseEntity.ok(UsuarioResponse.from(usuarioLogado));
}

// Após cada requisição, o SecurityContext é limpo automaticamente
// (Spring Security faz isso — não há vazamento entre requests)
```

---

## Testando a segurança com curl

```bash
# 1. Fazer login
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"felipe@ex.com","senha":"Senha@123"}'

# Resposta:
# {
#   "token": "eyJhbGciOiJIUzI1NiJ9...",
#   "expiracaoEm": 1705401000000,
#   "usuario": { "id": 1, "nome": "Felipe", "email": "felipe@ex.com" }
# }

# 2. Usar o token
TOKEN="eyJhbGciOiJIUzI1NiJ9..."

curl http://localhost:8080/api/v1/clientes \
  -H "Authorization: Bearer $TOKEN"

# 3. Sem token — 401
curl http://localhost:8080/api/v1/clientes
# { "status": 401, "erro": "Não autenticado", ... }

# 4. Token expirado — 401
# (use um token antigo)
curl http://localhost:8080/api/v1/clientes \
  -H "Authorization: Bearer TOKEN_EXPIRADO"
# { "status": 401, "erro": "Não autenticado", ... }
```

---

## Configuração completa do application.yml para segurança

```yaml
app:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 86400000      # 24 horas em ms

# Para rodar localmente sem variável de ambiente:
# Crie application-dev.yml (não commitar!) com:
# app.jwt.secret: SuaChaveBase64AquiComMenosDe32Bytes=
```

---

## Resumo do fluxo completo de segurança

```
1. POST /auth/login { email, senha }
   → AuthController → AuthService
   → AuthenticationManager valida credenciais
   → JwtService gera token
   → Retorna { token: "eyJ..." }

2. GET /api/v1/pedidos
   Authorization: Bearer eyJ...
   → JwtAuthFilter extrai token
   → JwtService valida e extrai email
   → UserDetailsService carrega Usuario do banco
   → SecurityContext recebe o Usuario autenticado
   → AuthorizationFilter verifica permissões
   → PedidoController executa
   → Retorna dados

3. GET /api/v1/pedidos (sem token)
   → JwtAuthFilter: sem Authorization header → passa sem autenticar
   → AuthorizationFilter: endpoint exige auth → 401 Unauthorized

4. DELETE /api/v1/clientes/1 (usuário com role USER, não ADMIN)
   → JwtAuthFilter: token válido → usuario autenticado com role USER
   → AuthorizationFilter: endpoint exige ADMIN → 403 Forbidden
```

---

## Próximas notas
- [[39 - Padrão REST Completo]] — boas práticas de REST
- [[41 - Paginação e Ordenação]] — paginação em detalhe
