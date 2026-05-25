---
tags: [java, ano-3, segurança, oauth2, owasp, jwt, keycloak, criptografia]
trimestre: T12
meses: 37-39
---

# T12 · Segurança em Java
### Meses 37–39 · Ano 3

---

## 🔵 OWASP Top 10 para Java devs

| # | Vulnerabilidade | Mitigação em Java/Spring |
|---|----------------|--------------------------|
| A01 | Broken Access Control | `@PreAuthorize`, RBAC granular, testes de autorização |
| A02 | Cryptographic Failures | TLS 1.3, AES-256, nunca MD5/SHA1 para senhas |
| A03 | **Injection (SQL/NoSQL/LDAP)** | PreparedStatement, JPQL params, sanitização |
| A04 | Insecure Design | Threat modeling, design reviews |
| A05 | Security Misconfiguration | Security headers, profiles, secrets externos |
| A06 | Vulnerable Components | `mvn dependency:check`, Dependabot, OWASP Dependency Check |
| A07 | Auth/Session Failures | JWT com rotação, MFA, rate limiting em login |
| A08 | Software/Data Integrity | Verificar integridade de JARs, supply chain |
| A09 | Logging/Monitoring Failures | Logar eventos de segurança, alertas em tempo real |
| A10 | SSRF | Validar URLs, allowlist de hosts para chamadas externas |

---

## 🔵 Bloco 1 — Autenticação Robusta

### JWT — boas práticas

```java
@Service
public class JwtService {

    // NUNCA hardcode — sempre de variável de ambiente/Vault
    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.access-token-expiry:900}")  // 15 min
    private long accessTokenExpirySeconds;

    @Value("${jwt.refresh-token-expiry:86400}")  // 24h
    private long refreshTokenExpirySeconds;

    public String generateAccessToken(UserDetails user) {
        return Jwts.builder()
            .subject(user.getUsername())
            .issuedAt(new Date())
            .expiration(Date.from(Instant.now().plusSeconds(accessTokenExpirySeconds)))
            .claims(Map.of(
                "roles", user.getAuthorities().stream()
                              .map(GrantedAuthority::getAuthority).toList(),
                "type", "access"
            ))
            .signWith(getSigningKey(), Jwts.SIG.HS512)  // HS512 > HS256
            .compact();
    }

    // Refresh token — armazenar hash no banco
    public String generateRefreshToken(Long userId) {
        var token = UUID.randomUUID().toString();
        var hash = DigestUtils.sha256Hex(token);
        refreshTokenRepository.save(new RefreshToken(userId, hash,
            Instant.now().plusSeconds(refreshTokenExpirySeconds)));
        return token;
    }
}
```

### Keycloak — Identity Provider para produção

```yaml
# application.yml — Spring Boot como Resource Server
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://keycloak.empresa.com/realms/myrealm
          jwk-set-uri: https://keycloak.empresa.com/realms/myrealm/protocol/openid-connect/certs
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(keycloakJwtConverter())))
            .build();
    }

    @Bean
    public JwtAuthenticationConverter keycloakJwtConverter() {
        var converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(jwt -> {
            // Extrair roles do formato Keycloak
            var realmAccess = (Map<String, Object>) jwt.getClaims().get("realm_access");
            var roles = (List<String>) realmAccess.get("roles");
            return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.toUpperCase()))
                .collect(toList());
        });
        return converter;
    }
}
```

---

## 🔵 Bloco 2 — Proteção contra Injeções

### SQL Injection — o clássico

```java
// ❌ NUNCA — vulnerável a SQL injection
String query = "SELECT * FROM users WHERE username = '" + username + "'";
stmt.execute(query);

// ✅ SEMPRE — PreparedStatement
String query = "SELECT * FROM users WHERE username = ?";
var stmt = conn.prepareStatement(query);
stmt.setString(1, username);

// ✅ JPQL com parâmetros (seguro)
@Query("SELECT u FROM User u WHERE u.username = :username")
Optional<User> findByUsername(@Param("username") String username);

// ❌ JPQL com concatenação (vulnerável)
@Query("SELECT u FROM User u WHERE u.username = '" + username + "'")
```

### Command Injection

```java
// ❌ Nunca execute input do usuário como comando
Runtime.getRuntime().exec("convert " + filename);

// ✅ Use ProcessBuilder com lista de argumentos
new ProcessBuilder("convert", "-resize", "800x600", inputFile, outputFile)
    .directory(workDir)
    .start();
```

### Path Traversal

```java
// ❌ Vulnerável a ../../../etc/passwd
File file = new File("/upload/" + filename);

// ✅ Validar que o arquivo resultante está dentro do diretório permitido
Path uploadDir = Path.of("/upload").toRealPath();
Path filePath = uploadDir.resolve(filename).normalize();
if (!filePath.startsWith(uploadDir)) {
    throw new SecurityException("Path traversal detectado");
}
```

---

## 🔵 Bloco 3 — Criptografia Correta

```java
@Service
public class CryptoService {

    // Hashing de senhas — BCrypt, Argon2 ou SCrypt
    private final PasswordEncoder passwordEncoder = new BCryptPasswordEncoder(12);

    // NUNCA: MD5, SHA1, SHA256 sem salt para senhas
    // NUNCA: armazenar senha em plaintext
    // NUNCA: criptografia reversível para senhas

    // AES-256-GCM para dados sensíveis
    public byte[] encrypt(byte[] data, SecretKey key) throws Exception {
        var cipher = Cipher.getInstance("AES/GCM/NoPadding");
        var iv = new byte[12];
        new SecureRandom().nextBytes(iv);
        cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(128, iv));
        var encrypted = cipher.doFinal(data);
        // prepend IV ao ciphertext para uso no decrypt
        var result = new byte[iv.length + encrypted.length];
        System.arraycopy(iv, 0, result, 0, iv.length);
        System.arraycopy(encrypted, 0, result, iv.length, encrypted.length);
        return result;
    }

    // Para segredos: use AWS Secrets Manager, HashiCorp Vault, ou Azure Key Vault
    // NUNCA armazene secrets em application.yml no repositório
}
```

### Spring Vault — integração com HashiCorp Vault

```yaml
spring:
  cloud:
    vault:
      uri: https://vault.empresa.com
      authentication: kubernetes  # em k8s
      kubernetes:
        role: pedidos-service
      kv:
        enabled: true
        backend: secret
        default-context: pedidos-service
```

---

## 🔵 Bloco 4 — Security Headers e HTTPS

```java
@Configuration
@EnableWebSecurity
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .headers(headers -> headers
                .contentSecurityPolicy(csp ->
                    csp.policyDirectives("default-src 'self'; script-src 'self'"))
                .referrerPolicy(rp ->
                    rp.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN))
                .permissionsPolicy(pp ->
                    pp.policy("geolocation=(), microphone=(), camera=()"))
                .frameOptions(fo -> fo.deny())
                .httpStrictTransportSecurity(hsts -> hsts
                    .maxAgeInSeconds(31536000)
                    .includeSubDomains(true))
            )
            .build();
    }
}
```

---

## 🔗 Navegação

← [[T11 — Performance e JVM Deep Dive]]  
→ [[Ano 4 — Arquitetura Avançada]]
