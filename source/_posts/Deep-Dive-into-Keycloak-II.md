---
title: Deep Dive into Keycloak II
date: 2025-12-02 20:05:40
categories:
- Deep Dive
- Keycloak
tags:
- Deep Dive
- Keycloak
---

# **1. What is the difference between OAuth2 and OpenID Connect (OIDC)?**

### **Answer:**

OAuth2 is an **authorization** framework — it gives applications access to protected resources via access tokens.
OIDC is an **authentication** layer built on top of OAuth2 — it provides identity information by adding **ID tokens**, user info endpoint, and standardized flows.

You use OAuth2 when an app needs *permissions*
You use OIDC when an app needs *login + identity*.

---

# **2. How does Keycloak handle session management compared to stateless JWT systems?**

### **Answer:**

Keycloak maintains a **server-side session** for each authenticated user.
Even though it issues JWTs, it “peeks” into the token’s session ID to enforce logout and token invalidation.
This means:

* Keycloak can force logout of any user
* Admin can revoke access tokens instantly
* Tokens are short-lived; refresh tokens renew them

Stateless JWT-only systems *cannot* revoke tokens without additional infrastructure.

---

# **3. What’s the purpose of a Keycloak Realm?**

### **Answer:**

A Realm is an isolated authentication domain.
Each realm has:

* its own clients
* identity providers
* users
* roles
* token settings

Realms do not share users.
This isolates tenants for multi-tenant apps.

---

# **4. Explain the OAuth2 Authorization Code Flow with PKCE.**

### **Answer:**

Used by public clients (SPA, mobile).
Steps:

1. Client generates a **code verifier** and **challenge**
2. User is redirected to Keycloak login
3. After successful login, Keycloak returns an **authorization code**
4. Client exchanges the code for tokens including the **code verifier**
5. Keycloak recomputes the challenge to verify request integrity

PKCE prevents code interception attacks.

---

# **5. What Keycloak feature would you use for Single Sign-On across multiple apps?**

### **Answer:**

**SSO session + OIDC Authorization Code Flow.**
Apps use the same Keycloak realm; once the user is logged into one app, Keycloak issues tokens to others without re-authentication.

---

# **6. How do you configure role-based access control in Keycloak?**

### **Answer:**

* Create **realm roles** or **client roles**
* Map roles to users or groups
* Configure clients to include roles in tokens (scope + mappers)
* Your service enforces authorization based on the roles claim in JWT

---

# **7. What is the difference between Realm Roles and Client Roles?**

### **Answer:**

* Realm roles are global across the entire realm
* Client roles belong to a specific client (service)

Client roles prevent namespace collisions and support service-specific authorization.

---

# **8. How do you secure a Spring Boot REST API using Keycloak?**

### **Answer:**

For Spring Security 6 / Boot 3:

1. Use the OAuth2 Resource Server starter
2. Configure `issuer-uri` pointed to Keycloak realm
3. Enable JWT validation
4. Add authorization rules based on roles in JWT

Resource server verifies tokens by fetching Keycloak’s public keys (JWKS).

---

# **9. What are Keycloak Identity Providers?**

### **Answer:**

Identity Providers allow Keycloak to delegate login to external systems like:

* Google
* GitHub
* Azure AD
* Another Keycloak realm

Keycloak becomes an identity broker.

---

# **10. How does token revocation work in Keycloak?**

### **Answer:**

Keycloak stores sessions server-side.
When an admin revokes tokens or logs a user out:

* Active tokens become invalid
* Refresh token rotation prevents re-use
* Next request with old token fails with 401

This is a major advantage over purely stateless JWT systems.

---

# **11. What is Offline Session and Offline Token?**

### **Answer:**

Offline tokens are long-lived refresh tokens not tied to active user sessions.
Useful for background jobs or CLI tools.
Users do not need to stay logged in.

---

# **12. How do you integrate Keycloak with an API Gateway (e.g., Kong, Nginx, Spring Cloud Gateway)?**

### **Answer:**

Use the gateway as an **OAuth2 resource server**.
It:

* validates JWTs from Keycloak
* checks scopes/roles
* forwards authorized requests to microservices
* optionally strips sensitive headers

Microservices behind the gateway become stateless and do not validate tokens directly.

---

# **13. What is the Keycloak Admin API used for?**

### **Answer:**

Automating user and client management:

* create/update users
* reset passwords
* assign roles
* manage groups
* manage clients
* trigger impersonation

Often used in CI/CD or multi-tenant systems.

---

# **14. How would you scale Keycloak for production?**

### **Answer:**

* Run multiple Keycloak nodes behind a load balancer
* Use a shared database (PostgreSQL recommended)
* Use sticky sessions only if needed (usually not needed post-Keycloak 17+)
* Enable cross-node session cache with Infinispan
* Offload static content via reverse proxy

Keycloak 21+ with Quarkus has much lower memory footprint and better clustering.

---

# **15. What are the common pitfalls when integrating SPAs (React) with Keycloak?**

### **Answer:**

1. Storing tokens insecurely (never use localStorage → use memory with silent refresh).
2. Using Implicit Flow (deprecated).
3. Not handling token refresh properly.
4. Not using PKCE.
5. Not validating roles on backend.

Best practice: React = public client using Authorization Code + PKCE.

---

# **16. How do service-to-service calls work with Keycloak?**

### **Answer:**

Use **Client Credentials Flow**:

* No user involved
* The client authenticates with client secret
* Keycloak returns an access token
* Service uses token to call downstream service

This is machine-to-machine authentication.

---

# **17. What’s the difference between Access Token and ID Token?**

### **Answer:**

* Access token = permissions for APIs (authorization)
* ID token = identity info about the user (authentication)

APIs should ignore ID tokens.

---

# **18. How do you restrict which scopes a client may request?**

### **Answer:**

Use **Client Scopes**:

* Default client scopes
* Optional client scopes
* Protocol mappers to include additional claims
* Scope parameter to control token content

This prevents over-privileged tokens.

---

# **19. What is the Keycloak Token Exchange?**

### **Answer:**

Allows exchanging one token for another.
Use cases:

* service-to-service delegation
* switching realms
* getting a token for another client
* impersonation workflows

Powerful in microservice architectures.

---

# **20. How do you debug token validation issues?**

### **Steps to answer:**

1. Decode token using JWT.io
2. Verify issuer (iss) matches realm
3. Confirm audience (aud) includes client ID
4. Check signature using JWKS
5. Check token expiration
6. Check scope/roles included

Most validation failures come from misconfigured `issuer-uri` or missing mappers.
