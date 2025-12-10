
JWT Authentication — Deep Internal Mechanics (Spring Security)
1. What JWT Actually Solves

A JWT is not just a token.
It is a self-contained identity package that travels with every request.

It eliminates:

Server sessions

Central session store

Cross-node login issues

DB lookups for every request

JWT shifts identity from server memory → the client’s token.

This makes microservices scalable.

2. JWT Structure (The Real Meaning Behind Each Part)

A JWT has three parts:

Header

Payload (claims)

Signature

Example:

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.      
eyJzdWIiOiIxIiwiZXhwIjoxNzUzMjUwMDAsInJvbGVzIjpbIlVTRVIiLCJBRE1JTiJdfQ.      
ABC123xyzSignature

Header

Specifies algorithm + token type.

Payload

Contains claims like:

sub — identity

roles — authorities

iat — issued at

exp — expiry

jti — unique token ID

aud — audience

iss — issuer

Signature

Ensures the token is not tampered with.
Secret or private key signs it.

3. End-to-End Authentication Flow (Spring Security Internals)

Step 1: Client logs in → sends credentials
Step 2: Server authenticates user
Step 3: Server generates JWT and returns it
Step 4: Client sends JWT with every request
Step 5: JWT filter validates → builds Authentication
Step 6: Authorization checks role/permissions
Step 7: Controller executes

This flow is stateless.

4. Spring Security — The Real Pipeline

Spring Security uses a OncePerRequestFilter to intercept every request.

You create:

JwtAuthenticationFilter

JwtTokenProvider

CustomUserDetailsService

SecurityConfig

Core Mechanism:

Extract token

Validate signature + expiry

Extract roles

Build UsernamePasswordAuthenticationToken

Put into SecurityContext

5. High-Quality Sample JWT Filter (Production Style)
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtTokenProvider tokenProvider;

    public JwtAuthenticationFilter(JwtTokenProvider tokenProvider) {
        this.tokenProvider = tokenProvider;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        String token = tokenProvider.extractToken(request);

        if (token != null && tokenProvider.validate(token)) {
            Authentication auth = tokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }

        filterChain.doFilter(request, response);
    }
}

6. SecurityConfig — Essential Config
@Bean
public SecurityFilterChain filterChain(HttpSecurity http, JwtAuthenticationFilter jwtFilter) throws Exception {
    return http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/auth/**").permitAll()
                    .anyRequest().authenticated()
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
}

7. Common JWT Security Flaws (You Must Avoid)
❌ Mistake 1: Storing JWT in localStorage

Vulnerable to XSS.

❌ Mistake 2: Long expiration

Increases damage window.

❌ Mistake 3: Not using refresh tokens

Forces login too often.

❌ Mistake 4: Not rotating refresh tokens

Leaves you exposed to replay attacks.

❌ Mistake 5: Adding huge payload

JWT becomes bloated → slow.

8. When to Use JWT (And When NOT To)
Use JWT for:

✔ Microservices
✔ Mobile apps
✔ SPAs
✔ Gateway-based systems

Avoid JWT for:

❌ Server-side sessions (no need)
❌ Highly sensitive identity (banking login) → use OAuth2/OIDC
❌ Token must be instantly revocable → use DB-backed tokens or introspection

9. JWT Expiry Best Practices

Access Token → 5–15 minutes

Refresh Token → 7–14 days

Rotate refresh tokens every usage

Store refresh tokens only in HttpOnly cookies

10. Conclusion

JWT is not “just a token.”
It is a distributed identity system that shifts trust from server to signature.

Mastering JWT is mandatory for modern Spring microservices.
