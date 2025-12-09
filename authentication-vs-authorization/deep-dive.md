Authentication vs Authorization in Spring Security — The Part Most Developers Misunderstand

Most developers describe authentication as “login” and authorization as “permission checking.”
That’s directionally correct — but operationally useless.

In Spring Security, these two concerns run through different filters, different interfaces, and different decision engines.
If you don’t understand these internal pipelines, you’ll get stuck the moment you need to customize anything beyond default login.

This deep dive rewires your understanding at an architectural level — using a clear narrative and precise technical flow.

1. The One-Sentence Definitions (For Experts)
Authentication

Proves who you are by producing a fully populated Authentication object inside the SecurityContext.

Authorization

Determines what you can do by evaluating the authenticated principal against configured access rules.

These two pipelines are independent.
That independence is the root of Spring Security’s power — and the source of most confusion.

2. Authentication Pipeline — How Identity Is Actually Created

Think of authentication as a manufacturing process:

The request enters the conveyor belt (filter chain)

A specific station extracts credentials

The authentication manager validates them

A complete identity object is produced

The identity is stored for downstream use

Let’s break it down.

2.1 Request Hits the Security Filter Chain

Everything begins inside the SecurityFilterChain, not the controller.
This chain contains multiple filters — each responsible for a specific part of the security lifecycle.

2.2 Credential Extraction (Varies by Mechanism)

Different login methods → different filters:

Mechanism	Filter
Form Login	UsernamePasswordAuthenticationFilter
JWT	Custom filter extending OncePerRequestFilter
Basic Auth	BasicAuthenticationFilter

Each of these filters builds an incomplete Authentication object.
This object typically contains username + credentials, but no authorities yet.

2.3 AuthenticationManager and Provider Chain

This is where many developers have a mental blind spot.

Spring Security does not use a single validator.

It uses a chain of AuthenticationProviders.

Flow:

AuthenticationManager
    → Provider 1 (supports token? no → skip)
    → Provider 2 (supports token? yes → authenticate)


When a provider succeeds, it returns a fully populated Authentication object containing:

Principal

Granted Authorities

Authentication status = true

Additional metadata

This is the “identity certificate” that will be used later.

2.4 SecurityContext Is Written

After successful authentication, a filter writes the result to:

SecurityContextHolder.getContext().setAuthentication(authentication)


This is how the system “remembers” who you are for the rest of the request pipeline.

3. Authorization Pipeline — How Access Decisions Are Made

Once identity is created, another pipeline decides what you can do.

Authorization is NOT one thing — it’s a set of decision points.

3.1 Where Authorization Is Triggered

Authorization checks happen at:

URL level → FilterSecurityInterceptor

Method level → @PreAuthorize, @PostAuthorize

Custom voters → business rules

Expression handlers → complex SpEL conditions

This pipeline is activated after authentication.

3.2 AccessDecisionManager (The Brain)

At the core of authorization lies the AccessDecisionManager, which evaluates whether access should be granted.

It consults AccessDecisionVoters, which are like “security judges”:

RoleVoter → roles

AuthenticatedVoter → whether user is logged in

ExpressionVoter → SpEL conditions

Custom Voters → domain rules (e.g., “order owner can read order”)

Each voter gives one of three results:

GRANT
DENY
ABSTAIN


A final decision is made based on a strategy (e.g., unanimous, majority).

3.3 What Data Is Used During Authorization?

Authorities from the Authentication

Request metadata

Security expressions

Custom domain rules

Authorization is simply identity + rules → decision.

4. Why These Pipelines Are Independent

This separation is not accidental — it's strategic.

You can authenticate but still be forbidden

You can authorize without authentication (permitAll)

You can plug in multiple authenticators (JWT, OAuth2, Basic)

You can keep authorization rules unified

You can replace authentication without touching authorization logic

This makes security customizable for any architecture.

5. Real Production Example
Scenario:

User calls:

POST /orders/create

What happens internally?

Authentication:

JWT filter extracts token

AuthenticationManager validates

SecurityContext now holds:
Authentication(principal=User123, authorities=[ROLE_USER])

Authorization:

FilterSecurityInterceptor checks access rules

Rule requires: ROLE_ADMIN

User has: ROLE_USER

Voters evaluate

AccessDecisionManager returns: DENY

Result:
403 Forbidden

Identity was valid → but permissions were insufficient.

6. Diagram (Put in diagrams/auth-vs-authz.png)
            [ Request ]
                |
       SecurityFilterChain
                |
   -----------------------------------------
   |                                       |
[Authentication Pipeline]        [Authorization Pipeline]
   |                                       |
Credential Filters                    Security Interceptors
   |                                       |
AuthenticationManager               AccessDecisionManager
   |                                       |
AuthenticationProvider            AccessDecisionVoters
   |                                       |
[SecurityContext]                Grant / Deny / Abstain

7. Advanced Extensions You Can Add Later

Custom AuthenticationProvider (OTP, MFA, API keys)

Custom AccessDecisionVoter (domain-based rules)

Stateless SecurityContext for microservices

Combining JWT with method-level security

Multi-auth entry points for hybrid systems

8. Summary

Authentication creates identity

Authorization evaluates identity

They run in different filters and different chains

SecurityContext bridges the two pipelines

Mastery requires seeing them as separate, composable systems
