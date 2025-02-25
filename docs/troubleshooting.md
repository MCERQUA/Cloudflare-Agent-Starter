# Troubleshooting Guide

## Overview

This document provides solutions for common issues you might encounter when developing, testing, or deploying your Cloudflare Worker. Use this guide to diagnose and resolve problems quickly.

## Development Issues

### Local Development Server Won't Start

**Symptoms:**
- `npm run dev` fails to start the local development server
- Error messages about port conflicts or missing dependencies

**Possible Causes and Solutions:**

1. **Port 8787 is already in use**
   - Another process might be using port 8787
   - Solution: Kill the process using port 8787 or configure Wrangler to use a different port
   ```bash
   # Find the process using port 8787
   lsof -i :8787  # On macOS/Linux
   netstat -ano | findstr :8787  # On Windows
   
   # Kill the process
   kill -9 <PID>  # On macOS/Linux
   taskkill /F /PID <PID>  # On Windows
   
   # Or configure Wrangler to use a different port in wrangler.jsonc
   # Add: "dev": { "port": 8788 }
   ```

2. **Dependencies are not installed**
   - Solution: Install dependencies
   ```bash
   npm install
   ```

3. **Wrangler is not installed globally**
   - Solution: Use npx to run Wrangler or install it globally
   ```bash
   npx wrangler dev
   # or
   npm install -g wrangler
   ```

4. **Outdated Wrangler version**
   - Solution: Update Wrangler
   ```bash
   npm update wrangler
   # or if installed globally
   npm update -g wrangler
   ```

### TypeScript Compilation Errors

**Symptoms:**
- TypeScript errors when running `npm run dev` or `npm run deploy`
- Error messages about type mismatches or missing types

**Possible Causes and Solutions:**

1. **Incorrect TypeScript configuration**
   - Solution: Check `tsconfig.json` for proper configuration
   - Ensure `@cloudflare/workers-types` is properly referenced

2. **Outdated TypeScript types**
   - Solution: Update the `@cloudflare/workers-types` package
   ```bash
   npm update @cloudflare/workers-types
   ```

3. **Using Node.js APIs not available in Workers**
   - Solution: Replace Node.js-specific code with Web API equivalents
   - Example: Use `fetch()` instead of `http.request()`

4. **Missing type definitions for third-party libraries**
   - Solution: Install type definitions for the library
   ```bash
   npm install --save-dev @types/library-name
   ```

### Changes Not Reflected in Local Development

**Symptoms:**
- Changes to your code are not reflected when testing locally
- The local server seems to be serving an older version of your code

**Possible Causes and Solutions:**

1. **Wrangler caching issues**
   - Solution: Restart the development server
   ```bash
   # Stop the current server (Ctrl+C) and start it again
   npm run dev
   ```

2. **Browser caching**
   - Solution: Hard refresh the browser or disable caching in developer tools
   - Chrome/Edge: `Ctrl+Shift+R` or `Cmd+Shift+R` on Mac
   - Firefox: `Ctrl+F5` or `Cmd+Shift+R` on Mac
   - Or open DevTools > Network tab > Disable cache

3. **File not being watched by Wrangler**
   - Solution: Ensure the file is in a watched directory
   - Check if the file is excluded in `wrangler.jsonc`

## Testing Issues

### Tests Failing

**Symptoms:**
- Tests fail when running `npm test`
- Error messages from Vitest

**Possible Causes and Solutions:**

1. **Test environment configuration issues**
   - Solution: Check `vitest.config.mts` for proper configuration
   - Ensure the worker pool is properly configured

2. **Mismatch between test expectations and actual behavior**
   - Solution: Update tests to match the current behavior or fix the code
   - Check if the API has changed

3. **Missing mock data or fixtures**
   - Solution: Provide necessary mock data for tests
   - Ensure all external dependencies are properly mocked

4. **Async/await issues**
   - Solution: Ensure proper async/await usage in tests
   - Make sure promises are properly resolved before assertions

### Test Coverage Issues

**Symptoms:**
- Low test coverage
- Certain code paths are not being tested

**Possible Causes and Solutions:**

1. **Missing test cases**
   - Solution: Add tests for uncovered code paths
   - Use code coverage reports to identify untested code

2. **Conditional code that's hard to test**
   - Solution: Refactor code to make it more testable
   - Extract complex conditions into separate functions that can be tested independently

## Deployment Issues

### Deployment Fails

**Symptoms:**
- `npm run deploy` fails
- Error messages from Wrangler

**Possible Causes and Solutions:**

1. **Authentication issues**
   - Solution: Log in to Cloudflare
   ```bash
   npx wrangler login
   ```

2. **Missing or invalid account ID**
   - Solution: Check `wrangler.jsonc` for the correct account ID
   - Or let Wrangler detect it automatically by removing the account ID

3. **Worker size exceeds limits**
   - Solution: Reduce the size of your worker
   - Remove unnecessary dependencies
   - Split into multiple workers if necessary

4. **Invalid configuration in wrangler.jsonc**
   - Solution: Validate your `wrangler.jsonc` file
   - Check for syntax errors or invalid settings

5. **API token doesn't have the required permissions**
   - Solution: Generate a new API token with the correct permissions
   - Ensure the token has the "Edit Workers" permission

### Worker Deployed But Not Working

**Symptoms:**
- Deployment succeeds but the worker returns errors
- Worker doesn't behave as expected in production

**Possible Causes and Solutions:**

1. **Environment differences between local and production**
   - Solution: Ensure environment variables are properly set in `wrangler.jsonc`
   - Check for any environment-specific code

2. **Missing bindings or resources**
   - Solution: Check that all required KV namespaces, Durable Objects, etc. are properly configured
   - Verify that bindings in `wrangler.jsonc` match what your code expects

3. **CORS issues**
   - Solution: Add proper CORS headers to your responses
   ```typescript
   new Response(data, {
     headers: {
       'Access-Control-Allow-Origin': '*',
       'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
       'Access-Control-Allow-Headers': 'Content-Type'
     }
   })
   ```

4. **Rate limiting or other Cloudflare restrictions**
   - Solution: Check Cloudflare dashboard for any rate limiting or security settings that might be affecting your worker

## Runtime Issues

### Worker Crashes or Returns Errors

**Symptoms:**
- Worker returns 500 errors
- Error messages in Cloudflare logs

**Possible Causes and Solutions:**

1. **Unhandled exceptions**
   - Solution: Add try/catch blocks around critical code
   ```typescript
   try {
     // Your code here
   } catch (error) {
     return new Response(`Error: ${error.message}`, { status: 500 });
   }
   ```

2. **Memory limits exceeded**
   - Solution: Optimize memory usage
   - Avoid storing large objects in memory
   - Process data in smaller chunks

3. **CPU time limits exceeded**
   - Solution: Optimize CPU-intensive operations
   - Consider using Cloudflare's unbound workers for longer-running tasks

4. **Subrequest limits exceeded**
   - Solution: Reduce the number of subrequests
   - Batch requests when possible
   - Cache results to avoid repeated requests

### Performance Issues

**Symptoms:**
- Worker is slow to respond
- High latency in production

**Possible Causes and Solutions:**

1. **Inefficient code**
   - Solution: Profile and optimize your code
   - Look for unnecessary loops or expensive operations

2. **External API calls**
   - Solution: Cache responses from external APIs
   - Use Cloudflare's cache API or KV for caching

3. **Cold starts**
   - Solution: Keep your worker warm with scheduled requests
   - Optimize initialization code

4. **Large response sizes**
   - Solution: Compress responses
   - Only return necessary data
   - Consider pagination for large datasets

## Debugging Techniques

### Local Debugging

1. **Console logging**
   - Add `console.log()` statements to your code
   - Logs will appear in the terminal when running `npm run dev`

2. **Debugger statements**
   - Add `debugger;` statements to your code
   - Open DevTools in your browser when testing locally

3. **Request inspection**
   - Log request details to understand what's being received
   ```typescript
   console.log({
     url: request.url,
     method: request.method,
     headers: Object.fromEntries(request.headers.entries()),
     // For POST requests:
     // body: await request.text() // or request.json() for JSON
   });
   ```

### Production Debugging

1. **Cloudflare Workers Logs**
   - Enable logging in `wrangler.jsonc`:
   ```json
   {
     "observability": {
       "enabled": true
     }
   }
   ```
   - View logs in the Cloudflare dashboard

2. **Custom error responses**
   - Return detailed error information in development
   - Return sanitized errors in production
   ```typescript
   const isDev = env.ENVIRONMENT === 'development';
   
   try {
     // Your code here
   } catch (error) {
     return new Response(
       JSON.stringify({
         error: error.message,
         stack: isDev ? error.stack : undefined
       }),
       { 
         status: 500,
         headers: { 'Content-Type': 'application/json' }
       }
     );
   }
   ```

3. **Health check endpoint**
   - Add a `/health` or `/status` endpoint that returns diagnostic information
   ```typescript
   if (url.pathname === '/health') {
     return new Response(
       JSON.stringify({
         status: 'ok',
         version: '1.0.0',
         timestamp: new Date().toISOString()
       }),
       { 
         headers: { 'Content-Type': 'application/json' }
       }
     );
   }
   ```

## Common Error Messages and Solutions

### "Module not found"

**Error:**
```
Error: Cannot find module './some-file'
```

**Solution:**
- Check that the file exists at the specified path
- Check for typos in the import statement
- Ensure the file extension is correct (`.ts`, `.js`, etc.)

### "Cannot read property 'X' of undefined"

**Error:**
```
TypeError: Cannot read property 'X' of undefined
```

**Solution:**
- Use optional chaining to safely access properties
```typescript
const value = object?.property?.nestedProperty;
```
- Add null checks before accessing properties
```typescript
if (object && object.property) {
  const value = object.property.nestedProperty;
}
```

### "Unexpected token"

**Error:**
```
SyntaxError: Unexpected token
```

**Solution:**
- Check for syntax errors in your code
- Ensure all brackets, parentheses, and braces are properly closed
- Check for missing semicolons or commas

### "Worker size limit exceeded"

**Error:**
```
Error: Worker size limit exceeded
```

**Solution:**
- Reduce the size of your worker
- Remove unnecessary dependencies
- Use tree-shaking to eliminate unused code
- Split into multiple workers if necessary

### "Fetch failed"

**Error:**
```
TypeError: Failed to fetch
```

**Solution:**
- Check the URL you're fetching from
- Ensure the external service is available
- Add error handling around fetch calls
```typescript
try {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`HTTP error! Status: ${response.status}`);
  }
  return response;
} catch (error) {
  console.error('Fetch error:', error);
  // Handle the error appropriately
}
```

## Getting Additional Help

If you're still experiencing issues after trying the solutions in this guide:

1. **Check the Cloudflare Workers documentation**
   - [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)

2. **Search the Cloudflare Workers GitHub repository**
   - [Cloudflare Workers GitHub](https://github.com/cloudflare/workers-sdk)

3. **Ask for help in the Cloudflare Workers Discord**
   - [Cloudflare Discord](https://discord.gg/cloudflaredev)

4. **Search or ask questions on Stack Overflow**
   - [Stack Overflow - Cloudflare Workers](https://stackoverflow.com/questions/tagged/cloudflare-workers)

5. **Contact Cloudflare Support**
   - [Cloudflare Support](https://support.cloudflare.com/)
