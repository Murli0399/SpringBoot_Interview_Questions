# Spring Security, JWT, OAuth2 and RBAC

<details><summary>

## What is Spring Security?
</summary>
Spring Security is a powerful framework that provides authentication, authorization, and other security features for Java applications. It is built on top of the Spring Framework and offers comprehensive security services to ensure the protection of web and enterprise applications.

In short, Spring Security:

### 1. Authentication:
It handles user authentication by providing mechanisms for login, logout, and user session management. It supports various authentication methods such as username/password authentication, token-based authentication, and integration with external identity providers.

### 2. Authorization:
Spring Security enables fine-grained access control by defining access rules and permissions. It allows you to secure different parts of your application based on roles, authorities, or custom expressions, ensuring that only authorized users can access specific resources or perform certain actions.

### 3. Security Filters:
It integrates with the Servlet API to intercept and filter requests and responses, enabling the enforcement of security measures at various levels. Spring Security intercepts requests and applies security checks based on the configuration you define.

### 4. Integration with Other Technologies:
Spring Security seamlessly integrates with other components of the Spring ecosystem, such as Spring MVC, Spring Boot, and Spring Data. It provides easy configuration options and simplifies the integration of security features into Spring-based applications.

### 5. Extensibility:
Spring Security is highly customizable and extensible. It allows you to plug in custom authentication providers, implement custom authorization logic, and integrate with external authentication systems. You can also leverage its extensive set of built-in features and extensions to meet your specific security requirements.

By using Spring Security, developers can focus on application logic while delegating the complex security aspects to the framework. It provides a robust foundation for securing applications against common security threats and offers flexibility to adapt to various security requirements.
</details>


<details><summary>

## What is the difference between authentication and authorization?
</summary>

- **Authentication:** *who are you?* Verifying identity (password, JWT, OAuth2 login). Failure → **401 Unauthorized**.
- **Authorization:** *what may you do?* Checking roles/permissions for a resource. Failure → **403 Forbidden**.

In Spring Security, authentication produces an `Authentication` object stored in the `SecurityContextHolder`; authorization rules (`hasRole`, `@PreAuthorize`) read it.
</details>

<details><summary>

## How does the Spring Security filter chain work?
</summary>

Spring registers one servlet filter (`DelegatingFilterProxy` → `FilterChainProxy`) that runs an ordered `SecurityFilterChain` of filters, e.g.:

`CorsFilter` → `CsrfFilter` → `LogoutFilter` → `UsernamePasswordAuthenticationFilter` / `BearerTokenAuthenticationFilter` → `ExceptionTranslationFilter` → `AuthorizationFilter`

Configuration in Spring Security 6 (no more `WebSecurityConfigurerAdapter`):

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain api(HttpSecurity http, JwtAuthFilter jwtFilter) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())                      // stateless REST API with tokens
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**", "/actuator/health/**").permitAll()
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
}
```
</details>

<details><summary>

## How does JWT authentication work? Show the filter.
</summary>

1. User logs in with credentials → server verifies → returns a **signed** JWT (header.payload.signature) with claims such as `sub`, `roles`, `tenantId`, `exp`.
2. Client sends `Authorization: Bearer <token>` on each request.
3. A filter validates the signature and expiry, builds an `Authentication` and puts it in the `SecurityContext`. No server session needed.

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    private final JwtService jwt;

    public JwtAuthFilter(JwtService jwt) { this.jwt = jwt; }

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws ServletException, IOException {
        String header = req.getHeader(HttpHeaders.AUTHORIZATION);
        if (header != null && header.startsWith("Bearer ")) {
            try {
                Claims claims = jwt.parseAndValidate(header.substring(7));   // signature + exp
                List<GrantedAuthority> authorities = jwt.roles(claims).stream()
                    .map(r -> new SimpleGrantedAuthority("ROLE_" + r))
                    .collect(Collectors.toList());
                var auth = new UsernamePasswordAuthenticationToken(claims.getSubject(), null, authorities);
                SecurityContextHolder.getContext().setAuthentication(auth);
            } catch (JwtException e) {
                res.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid token");
                return;
            }
        }
        chain.doFilter(req, res);
    }
}
```

Alternatively use Spring's built-in resource server: `http.oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))` with `spring.security.oauth2.resourceserver.jwt.issuer-uri`.
</details>

<details><summary>

## How do you handle JWT expiry, logout and revocation?
</summary>

- Keep **access tokens short-lived** (5–15 min) and issue a **refresh token** (longer-lived, stored securely, rotated on every use).
- JWTs are stateless, so true logout needs state: store a **denylist** of revoked token IDs (`jti`) in Redis with TTL = remaining token life, or keep sessions in Redis and put only a session ID in the token.
- Never put secrets or PII in the payload: it's only Base64-encoded, not encrypted.
- Sign with RS256 (private/public key pair) when other services must verify tokens without being able to issue them.
</details>

<details><summary>

## Session-based vs token-based authentication?
</summary>

| | Session (cookie) | Token (JWT) |
|---|---|---|
| State | Server stores session (memory/Redis) | Stateless; data in token |
| Revocation | Easy: delete session | Hard: needs denylist or short expiry |
| Scaling | Needs shared session store (Spring Session + Redis) | Any instance can verify |
| CSRF risk | Yes (cookies auto-sent), needs CSRF protection | Not if sent in `Authorization` header |
| Best for | Browser apps, server-rendered pages | APIs, mobile, service-to-service |

Hybrid is common: JWT access token + Redis-backed session/refresh state for revocation.
</details>

<details><summary>

## Explain OAuth2 and the Authorization Code flow with PKCE.
</summary>

OAuth2 lets an app access resources **on behalf of a user** without seeing their password. Roles: resource owner (user), client (your app), authorization server (Google, Keycloak, Okta), resource server (your API).

**Authorization Code + PKCE** (for browser/mobile apps):
1. Client creates a random `code_verifier` and sends its hash (`code_challenge`) with the login redirect.
2. User logs in at the authorization server and consents.
3. Server redirects back with a short-lived **authorization code**.
4. Client exchanges code + original `code_verifier` for tokens. The server checks the hash matches, so a stolen code is useless.
5. Client calls APIs with the access token.

Other grants: **Client Credentials** (service-to-service, no user), Refresh Token. Implicit and password grants are deprecated. **OpenID Connect** adds an ID token on top of OAuth2 for login (identity).
</details>

<details><summary>

## How do you implement role-based access control (RBAC)?
</summary>

URL-level rules in the filter chain, plus method-level rules for business logic:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }

@PreAuthorize("hasAuthority('REPORTS_READ') and #tenantId == authentication.principal.tenantId")
public Report getReport(String tenantId, Long reportId) { ... }

@PostAuthorize("returnObject.ownerId == authentication.name")
public Document getDocument(Long id) { ... }
```

Model: `User` → many `Role`s → many `Permission`s. Check **permissions** in code (`hasAuthority`), not role names, so roles can change without code changes. For multi-level setups (platform admin → org admin → team member), scope each permission to a tenant/org and enforce the tenant filter centrally so no query forgets it. Cache resolved permissions in Redis, and evict on role change.
</details>

<details><summary>

## What is CSRF and when can you disable protection?
</summary>

Cross-Site Request Forgery: a malicious site makes the user's browser send a request to your site, and the browser automatically attaches the user's cookies. Spring Security blocks state-changing requests without a valid CSRF token.

You may disable it for **stateless APIs authenticated by a header token** (no cookies). Keep it enabled for cookie/session-based browser apps (or use `SameSite` cookies plus CSRF tokens).
</details>

<details><summary>

## How do you store passwords securely?
</summary>

Never store plain text or fast hashes (MD5/SHA-256). Use an adaptive, salted algorithm: **BCrypt** (default), Argon2 or SCrypt.

```java
PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder(); // {bcrypt}...
String hash = encoder.encode("s3cret");
boolean ok = encoder.matches("s3cret", hash);
```

The salt is stored inside the hash; the work factor makes brute force slow. Add rate limiting/lockout on login endpoints.
</details>

<details><summary>

## How do you secure service-to-service calls?
</summary>

- **OAuth2 Client Credentials:** each service gets its own client ID/secret and requests short-lived tokens with limited scopes.
- **mTLS:** both sides present certificates (often handled by a service mesh like Istio/Linkerd).
- Propagate the **user's token** downstream when the call is on behalf of a user, so downstream services can authorize the user too.
- Keep secrets in a secret manager (Kubernetes Secrets, Vault, GCP Secret Manager), never in Git.
</details>
