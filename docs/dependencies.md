# Project Dependencies

## Overview

This document provides information about the dependencies used in the Cloudflare Agent project. Understanding these dependencies is important for maintaining, extending, and troubleshooting the project.

## Core Dependencies

### Production Dependencies

Currently, this starter project does not have any production dependencies listed in `package.json`. This is intentional, as Cloudflare Workers are designed to be lightweight and self-contained.

### Development Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| `@cloudflare/vitest-pool-workers` | `^0.6.4` | Provides a test environment specifically designed for Cloudflare Workers |
| `@cloudflare/workers-types` | `^4.20250224.0` | TypeScript type definitions for Cloudflare Workers |
| `typescript` | `^5.5.2` | TypeScript compiler for type checking and transpilation |
| `vitest` | `~2.1.9` | Testing framework for running unit and integration tests |
| `wrangler` | `^3.109.3` | Cloudflare's CLI tool for developing, testing, and deploying Workers |

## Dependency Details

### @cloudflare/vitest-pool-workers

This package provides a test environment specifically designed for Cloudflare Workers. It allows you to run tests against your worker code in an environment that closely resembles the actual Cloudflare Workers runtime.

**Key features:**
- Simulates the Cloudflare Workers runtime environment
- Provides access to Workers-specific APIs in tests
- Enables both unit and integration testing of Workers

**Usage in the project:**
- Used in `vitest.config.mts` to configure the test environment
- Enables the tests in `test/index.spec.ts` to run in a Workers-like environment

### @cloudflare/workers-types

This package provides TypeScript type definitions for Cloudflare Workers. It includes types for the Workers runtime, bindings, and other Cloudflare-specific features.

**Key features:**
- Type definitions for the Workers runtime
- Types for KV, Durable Objects, R2, and other Cloudflare services
- Updated regularly to match the latest Workers features

**Usage in the project:**
- Referenced in `tsconfig.json` to provide type checking for Workers-specific code
- Used in `src/index.ts` for type checking the worker handler
- Used in `worker-configuration.d.ts` for environment type definitions

### typescript

TypeScript is a typed superset of JavaScript that compiles to plain JavaScript. It adds static types to JavaScript, which helps catch errors during development.

**Key features:**
- Static type checking
- Modern JavaScript features
- Improved tooling and IDE support

**Usage in the project:**
- Configured in `tsconfig.json`
- Used to write type-safe code in `src/index.ts` and other files
- Compiled to JavaScript during the build process

### vitest

Vitest is a testing framework for JavaScript and TypeScript. It's designed to be fast and compatible with the Vite build tool.

**Key features:**
- Fast test execution
- Compatible with Jest's API
- Built-in code coverage
- TypeScript support

**Usage in the project:**
- Configured in `vitest.config.mts`
- Used to write and run tests in `test/index.spec.ts`
- Provides assertions and test structure

### wrangler

Wrangler is Cloudflare's command-line tool for developing, testing, and deploying Workers.

**Key features:**
- Local development server
- Deployment to Cloudflare
- Management of KV namespaces, Durable Objects, and other resources
- Configuration via `wrangler.jsonc`

**Usage in the project:**
- Configured in `wrangler.jsonc`
- Used in npm scripts (`npm run dev`, `npm run deploy`)
- Manages the development and deployment workflow

## Adding Dependencies

When adding new dependencies to the project, consider the following:

1. **Bundle size**: Cloudflare Workers have a size limit (currently 1MB for the bundled worker). Be mindful of adding large dependencies.
2. **Compatibility**: Ensure that dependencies are compatible with the Workers runtime. Not all Node.js APIs are available in Workers.
3. **Performance**: Dependencies can impact cold start time and overall performance. Choose lightweight alternatives when possible.
4. **Security**: Regularly update dependencies to address security vulnerabilities.

### How to Add a Dependency

To add a production dependency:

```bash
npm install dependency-name
```

To add a development dependency:

```bash
npm install --save-dev dependency-name
```

After adding a dependency, update this document to include information about the new dependency.

## Recommended Dependencies

Here are some recommended dependencies that work well with Cloudflare Workers:

### Routing

- **itty-router**: Lightweight router for Workers
- **worktop**: Router and utilities for Workers

### Data Handling

- **zod**: Schema validation library
- **superjson**: Serialization library

### Utilities

- **date-fns**: Date manipulation library
- **nanoid**: Small, secure ID generator

### UI (for HTML generation)

- **htm**: JSX-like syntax without a build step
- **uhtml**: Tiny HTML templating library

## Dependency Management Best Practices

1. **Keep dependencies up to date**: Regularly update dependencies to get bug fixes and security patches.
2. **Use exact versions**: Consider using exact versions (`"dependency": "1.2.3"` instead of `"dependency": "^1.2.3"`) for more predictable builds.
3. **Audit dependencies**: Run `npm audit` regularly to check for security vulnerabilities.
4. **Document dependencies**: Update this document when adding or removing dependencies.
5. **Consider alternatives**: Before adding a dependency, consider if the functionality can be implemented with native APIs or smaller alternatives.
