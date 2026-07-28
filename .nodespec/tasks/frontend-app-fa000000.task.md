# Task: Frontend App

> **Scope:** implement ONLY this node ("Frontend App"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Frontend Application
**Technology:** React
**Description:** Client-side web application or SPA

## Your Deliverable

**Working code for this component**, honoring the contracts and criteria below, plus its configuration artifacts and tests.

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Scaffold the React component.**
  Create the source layout, build files, and test harness this node's working code lives in.
  Start from the catalog's suggested structure: `src/App.tsx`, `src/main.tsx`, `vite.config.ts`, `tsconfig.json`.
- [ ] **T2 — Implement the integration with Frontend Static Assets (aws-s3) per Contract "Build Artifact Deployment" (dependency).**
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T3 — Implement the integration with API Gateway (aws-api-gateway) per Contract "API Calls" (rest).**
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves (unverified match): REQ-018 "The frontend communicates with the API layer for all data operations" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T4 — Implement the integration with Cognito User Pool (aws-cognito) per Contract "Auth SDK" (dependency).**
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
  ↳ serves (unverified match): REQ-018 "The frontend handles user authentication via the managed identity provider SDK" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T5 — Expose the interface CloudFront consumes, per Contract "CDN Delivery" (rest).**
  Record the endpoint/identifiers CloudFront needs in this node's config artifacts — coordinate with CloudFront.
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves (unverified match): REQ-018 "The application is served as a single-page application via the CDN" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T6 — Implement: "The application is capable of rendering WebGL/Three.js scenes" (REQ-018).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-018 "The application is capable of rendering WebGL/Three.js scenes"
- [ ] **T7 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

**Your first action — expand these work orders.** Each task above guarantees WHAT must be covered, not HOW. Before writing any code or configuration, expand every task with the concrete implementation steps for THIS technology in THIS project — the specific resources, settings, files, schemas, and tests — using the Configuration, Interface Contracts, Technology Guidance, and node context as your references. Record the expanded list in this section via update_artifact (propose_patches) after this doc is accepted, keeping task IDs, criterion citations, and open `[PLACEHOLDER: …]` tags intact. Resolve placeholders with the user through the proposal flow; this node is never complete while one remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-018: React Frontend Application
Category: functional | Status: in-progress
A React single-page application shall serve as the user-facing frontend, capable of WebGL/Three.js rendering, delivered via CDN, and integrated with the API and authentication layers.

**Acceptance criteria — your task boxes:**
- [ ] The application is served as a single-page application via the CDN
  → covered by Task T5
- [ ] The frontend communicates with the API layer for all data operations
  → covered by Task T3
- [ ] The frontend handles user authentication via the managed identity provider SDK
  → covered by Task T4
- [ ] The application is capable of rendering WebGL/Three.js scenes
  → covered by Task T6

## Interface Contracts

### SENDS TO: Frontend Static Assets (object-storage)
- **Contract:** Build Artifact Deployment
- **Protocol:** dependency
- **Their Technology:** aws-s3

**Schema:**
```
{
  "tool": "aws s3 sync ./dist s3://bucket-name --delete",
  "type": "deployment",
  "files": "index.html, *.js, *.css, assets/*",
  "description": "CI/CD pipeline uploads the React build output (dist/ or build/) to the S3 bucket. Not a runtime call — this is a deployment-time relationship."
}
```

### RECEIVES FROM: CloudFront (cdn)
- **Contract:** CDN Delivery
- **Protocol:** rest
- **Their Technology:** aws-cloudfront

**Schema:**
```
{
  "path": "/*",
  "type": "http_static_delivery",
  "method": "GET",
  "response": {
    "contentType": "text/html | application/javascript | text/css | image/*",
    "cacheControl": "per-asset (immutable for hashed bundles, no-cache for index.html)"
  },
  "description": "CloudFront serves the React SPA build artifacts from the S3 origin to the end user's browser. The SPA then runs client-side."
}
```

### SENDS TO: API Gateway (api-gateway)
- **Contract:** API Calls
- **Protocol:** rest
- **Their Technology:** aws-api-gateway

**Schema:**
```
{
  "auth": "JWT Bearer token in Authorization header, issued by Cognito, validated by API Gateway JWT authorizer",
  "path": "/api/{proxy+}",
  "type": "http_rest",
  "method": "ANY",
  "description": "React SPA makes authenticated REST calls to the API Gateway for all data operations."
}
```

### SENDS TO: Cognito User Pool (auth-provider)
- **Contract:** Auth SDK
- **Protocol:** dependency
- **Their Technology:** aws-cognito

**Schema:**
```
{
  "type": "sdk_integration",
  "library": "@aws-amplify/auth or amazon-cognito-identity-js",
  "operations": [
    "signUp",
    "signIn",
    "signOut",
    "getCurrentUser",
    "fetchAuthSession"
  ],
  "description": "React SPA integrates Cognito via the AWS Amplify Auth SDK or amazon-cognito-identity-js for sign up, sign in, sign out, and JWT token retrieval."
}
```

## Technology Guidance

_Reference for executing the Implementation Tasks above — apply where relevant. The task list stands even where this guidance is thin._

**Purpose:** Component-based UI library for building interactive single-page applications

**SDK Initialization:**
```
npm create vite@latest my-app -- --template react-ts && cd my-app && npm install
// src/App.tsx
export default function App() {
  return <div>Hello React</div>;
}
```

**Common API Patterns:**

#### Component with State
Functional component with useState hook
```
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

#### Data Fetching
Custom hook for data fetching with loading state
```
function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  useEffect(() => {
    fetch("/api/users").then(r => r.json()).then(setUsers).finally(() => setLoading(false));
  }, []);
  return { users, loading };
}
```

#### Context Provider
Context API for global state with typed custom hook
```
const AuthContext = createContext<AuthState | null>(null);
export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}
export const useAuth = () => useContext(AuthContext)!;
```

**Configuration Template:**
```
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
export default defineConfig({
  plugins: [react()],
  server: { port: 3000 },
  build: { sourcemap: true }
});

// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "jsx": "react-jsx",
    "strict": true,
    "moduleResolution": "bundler"
  }
}
```

**Best Practices:**
- Use functional components with hooks
- Implement proper state management (lift state up, use context sparingly)
- Memoize expensive computations with useMemo/useCallback
- Use React.lazy for code splitting
- Follow the single responsibility principle per component
- Use TypeScript for type safety
- Implement error boundaries
- Use Suspense for async data loading

**Anti-Patterns to Avoid:**
- Prop drilling through many levels instead of using Context or state management
- Using useEffect for derived state that should be computed during render
- Creating new object/function references in render causing unnecessary re-renders
- Using index as key in lists with dynamic ordering or mutations
- Putting business logic directly in components instead of custom hooks

**Security:** React auto-escapes JSX output preventing most XSS. Never use dangerouslySetInnerHTML with unsanitized input. Validate and sanitize URL-based props (href, src) to prevent javascript: injection. Use Content Security Policy headers. Avoid storing sensitive tokens in localStorage -- use httpOnly cookies. Sanitize user input before passing to third-party libraries that manipulate DOM directly.

**Integration Patterns:**
- React Router for client-side routing with lazy-loaded routes
- TanStack Query (React Query) for server state management and caching
- Zustand or Jotai for lightweight client state management
- Tailwind CSS or CSS Modules for scoped styling
- Vitest + React Testing Library for component testing

**Suggested File Structure:**
- `src/App.tsx` (source)
- `src/main.tsx` (source)
- `vite.config.ts` (config)
- `tsconfig.json` (config)

## Dependency Chain

Startup/initialization order based on edge directions and interaction patterns.

**Must be available BEFORE this node starts:**
- Frontend Static Assets (this node calls/depends on it via Build Artifact Deployment (dependency))
- API Gateway (this node calls/depends on it via API Calls (rest))
- Cognito User Pool (this node calls/depends on it via Auth SDK (dependency))

**Depends on THIS node being available:**
- CloudFront (calls this node via CDN Delivery (rest))

**Parent Container:** AWS (aws)
