---
name: betterauth
description: Expert for Better Auth integration across Go backends and TanStack Query/React frontends. Use when setting up authentication, defining auth schemas, or implementing secure API communication between Go and Vite.
---
# Better Auth Integration Skill

This skill provides the source of truth for integrating Better Auth into a polyglot stack (Go backend + React frontend). It ensures secure session management, type-safe API calls, and seamless TanStack Query integration.

## Architectural Standards

### 1. Unified Auth Schema & Plugins
*   **Database:** Use the Better Auth standard schema. Standardize table names to `user`, `session`, `account`, `verification`.
*   **Plugins:** Enable and configure:
    *   `organization`: For multi-tenant isolation.
    *   `sso`: For SAML/OIDC enterprise login.
    *   `two-factor`: For TOTP/Email/SMS MFA.
    *   `admin`: For user impersonation and management.

### 2. Go Backend Integration (The Bridge)
*   **Middleware:** Implement a robust middleware that:
    1. Extracts the `better-auth.session_token` cookie.
    2. Validates it against the `session` table.
    3. Handles **Token Refresh** by checking the `expiresAt` and performing a silent refresh via the Better Auth API if needed.
*   **Context:** Store the full `User` and `Organization` objects in the request context for downstream RBAC.

### 3. TanStack Query Frontend (The Consumer)
*   **Session Lifecycle:**
    ```typescript
    export const useAuth = () => {
      return useQuery({
        queryKey: ['auth', 'session'],
        queryFn: async () => {
          const res = await authClient.getSession();
          if (res.error) throw res.error;
          return res.data;
        },
        staleTime: 1000 * 60 * 5, // 5 minutes
        refetchOnWindowFocus: true,
      });
    };
    ```
*   **Interceptors:** Implement an Axios interceptor that catches 401s and attempts an `authClient.refreshToken()` before failing and redirecting to login.

### 4. Security & Isolation
*   **CORS:** Strictly allow only your frontend origin. Enable `credentials: true`.
*   **CSRF:** Better Auth handles CSRF via a custom header or double-submit cookie. Ensure your Go backend validates these if performing non-GET requests.
*   **Tenant Mapping:** Every query MUST be filtered by `organizationId`. Never rely on frontend-provided IDs; always derive from the validated session.

## Interaction Protocol
*   **Input:** Database schema, Go server setup, or React frontend requirements.
*   **Output:** Detailed plugin configuration, Go middleware implementations, and TanStack Query hooks.

**Tag**: Start your response with `[BETTER-AUTH]`.

## Implementation Workflow

1.  **Schema Sync:** Define the Better Auth schema in the Go database.
2.  **API Bridge:** Implement the session verification middleware in Go.
3.  **Frontend Setup:** Initialize the Better Auth client in the Vite app.
4.  **Query Integration:** Wrap auth state in TanStack Query for global reactivity.

**Tag**: Start your response with `[BETTER-AUTH]`.
