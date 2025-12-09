Authentication vs Authorization — Deep Dive (Spring Security Internals)

This module explores the internal mechanics of Authentication and Authorization in Spring Security — far beyond the typical “login vs permission” explanation.

The goal is to provide a clear, technically accurate, and architecture-level understanding of how identity is created, how access decisions are made, and how both pipelines interact inside the Spring Security filter chain.

📚 Contents
1. deep-dive.md

A comprehensive article covering:

The real difference between authentication & authorization

Internal flow of Spring Security's filter chain

How AuthenticationProviders work under the hood

How AccessDecisionManager + Voters make authorization decisions

Production-level examples

A clean conceptual diagram

Extension patterns (custom providers, voters, MFA, etc.)

This is the main document.

🎯 Why This Matters

Understanding these two pipelines is essential for:

Building secure microservices (JWT, OAuth2, Gateway security)

Debugging complex permission issues

Implementing custom authentication mechanisms

Writing business-specific authorization logic

Designing scalable, maintainable security architecture

This knowledge separates mid-level developers from true security engineers.

📌 How to Use This Folder

Start with deep-dive.md

Study the diagram to form a mental model

Apply concepts in real code (custom filters, providers, voters)

Use this as a reference when designing authentication flows

Extend the article as you learn more patterns

🚧 Coming Soon (Planned)

You can add these later as your repo evolves:

custom-authentication-provider.md

custom-access-decision-voter.md

method-security-internals.md

stateless-securitycontext.md

This repo is intended to become a complete Spring Security reference.

💡 Contribution & Notes

This is part of a larger project documenting deep Spring Security internals.
Content will be expanded and refined as new concepts are implemented in real-world microservices.
