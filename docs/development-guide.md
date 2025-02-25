# Development Guide for Cloudflare Agents

This guide will help you understand how to develop and extend your Cloudflare Worker agent with more advanced functionality.

## Project Structure

Before diving into development, let's understand the project structure:

```
agents-starter/
├── src/                 # Source code
│   └── index.ts         # Main worker code
├── test/                # Test files
│   ├── index.spec.ts    # Tests for the worker
│   └── tsconfig.json    # TypeScript configuration for tests
├── wrangler.jsonc       # Cloudflare Wrangler configuration
└── worker-configuration.d.ts # Worker environment type definitions
```

## Development Workflow

1. **Make changes to your code** in the `src` directory
2. **Test locally** with `npm run dev`
3. **Write tests** in the `test` directory
4. **Run tests** with `npm test`
5. **Deploy** with `npm run deploy`

## Handling Different HTTP Methods

The default worker responds to all requests with "Hello World!". Here's how to handle different HTTP methods:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Get the HTTP method
    const method = request.method;
    
    // Handle different methods
    switch (method) {
      case 'GET':
        return new Response('This is a GET request');
      case 'POST':
        return new Response('This is a POST request');
      case 'PUT':
        return new Response('This is a PUT request');
      case 'DELETE':
        return new Response('This is a DELETE request');
      default:
        return new Response('Method not supported', { status: 405 });
    }
  },
} satisfies ExportedHandler<Env>;
```

## Routing Requests

You can implement basic routing to handle different URL paths:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // Handle different paths
    switch (path) {
      case '/':
        return new Response('Home page');
      case '/about':
        return new Response('About page');
      case '/api':
        return new Response('API endpoint');
      default:
        return new Response('Not found', { status: 404 });
    }
  },
} satisfies ExportedHandler<Env>;
```

## Working with JSON

To handle JSON requests and responses:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Check if the request is a POST to /api
    if (request.method === 'POST' && new URL(request.url).pathname === '/api') {
      try {
        // Parse the JSON body
        const body = await request.json();
        
        // Process the data
        const result = {
          message: 'Data received',
          data: body,
          timestamp: new Date().toISOString()
        };
        
        // Return a JSON response
        return new Response(JSON.stringify(result), {
          headers: {
            'Content-Type': 'application/json'
          }
        });
      } catch (error) {
        return new Response(JSON.stringify({ error: 'Invalid JSON' }), {
          status: 400,
          headers: {
            'Content-Type': 'application/json'
          }
        });
      }
    }
    
    // Handle other requests
    return new Response('Send a POST request to /api with JSON data');
  },
} satisfies ExportedHandler<Env>;
```

## Using Environment Variables

You can define environment variables in `wrangler.jsonc` and access them in your code:

1. Add variables to `wrangler.jsonc`:

```json
{
  "vars": {
    "API_KEY": "your-api-key",
    "DEBUG_MODE": "true"
  }
}
```

2. Update the `Env` interface in `worker-configuration.d.ts`:

```typescript
interface Env {
  API_KEY: string;
  DEBUG_MODE: string;
}
```

3. Use the variables in your code:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Access environment variables
    const apiKey = env.API_KEY;
    const debugMode = env.DEBUG_MODE === 'true';
    
    if (debugMode) {
      console.log('Debug mode is enabled');
    }
    
    return new Response('Worker is running');
  },
} satisfies ExportedHandler<Env>;
```

## Using Secrets

For sensitive information, use secrets instead of environment variables:

1. Add a secret using Wrangler:

```bash
npx wrangler secret put API_SECRET
```

2. Update the `Env` interface in `worker-configuration.d.ts`:

```typescript
interface Env {
  API_SECRET: string;
}
```

3. Use the secret in your code:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Access the secret
    const apiSecret = env.API_SECRET;
    
    // Use the secret (e.g., for API authentication)
    
    return new Response('Worker is running');
  },
} satisfies ExportedHandler<Env>;
```

## Working with Cloudflare KV

Cloudflare KV (Key-Value) storage allows you to store data that can be accessed by your worker:

1. Create a KV namespace:

```bash
npx wrangler kv:namespace create "MY_KV"
```

2. Add the namespace to `wrangler.jsonc`:

```json
{
  "kv_namespaces": [
    {
      "binding": "MY_KV",
      "id": "your-namespace-id"
    }
  ]
}
```

3. Update the `Env` interface in `worker-configuration.d.ts`:

```typescript
interface Env {
  MY_KV: KVNamespace;
}
```

4. Use KV in your code:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // Handle KV operations
    if (path.startsWith('/kv')) {
      const key = path.replace('/kv/', '');
      
      if (request.method === 'GET') {
        // Get a value
        const value = await env.MY_KV.get(key);
        if (value === null) {
          return new Response('Key not found', { status: 404 });
        }
        return new Response(value);
      } else if (request.method === 'PUT') {
        // Set a value
        const value = await request.text();
        await env.MY_KV.put(key, value);
        return new Response('Value stored');
      } else if (request.method === 'DELETE') {
        // Delete a value
        await env.MY_KV.delete(key);
        return new Response('Value deleted');
      }
    }
    
    return new Response('KV API: GET /kv/:key, PUT /kv/:key, DELETE /kv/:key');
  },
} satisfies ExportedHandler<Env>;
```

## Working with Durable Objects

Durable Objects provide strongly consistent storage and coordination:

1. Define a Durable Object class:

```typescript
export class Counter {
  private state: DurableObjectState;
  private count: number;

  constructor(state: DurableObjectState) {
    this.state = state;
    this.count = 0;
  }

  async fetch(request: Request): Promise<Response> {
    // Load stored count if it exists
    const stored = await this.state.storage.get<number>("count");
    this.count = stored || 0;

    // Handle the request
    if (request.method === "POST") {
      this.count++;
      await this.state.storage.put("count", this.count);
    }

    return new Response(this.count.toString());
  }
}
```

2. Add the Durable Object to `wrangler.jsonc`:

```json
{
  "durable_objects": {
    "bindings": [
      {
        "name": "COUNTER",
        "class_name": "Counter"
      }
    ]
  }
}
```

3. Update the `Env` interface in `worker-configuration.d.ts`:

```typescript
interface Env {
  COUNTER: DurableObjectNamespace;
}
```

4. Use the Durable Object in your worker:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    if (new URL(request.url).pathname === '/counter') {
      // Create an ID for the counter (using a fixed ID for simplicity)
      const id = env.COUNTER.idFromName('global-counter');
      
      // Get the Durable Object stub
      const stub = env.COUNTER.get(id);
      
      // Forward the request to the Durable Object
      return stub.fetch(request);
    }
    
    return new Response('Send a POST request to /counter to increment');
  },
} satisfies ExportedHandler<Env>;
```

## Handling CORS

To enable Cross-Origin Resource Sharing (CORS):

```typescript
function handleCors(request: Request): Response | null {
  // Check for OPTIONS request (preflight)
  if (request.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Max-Age': '86400',
      },
    });
  }
  
  // For actual requests, return null and handle normally
  return null;
}

export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Handle CORS preflight requests
    const corsResponse = handleCors(request);
    if (corsResponse) {
      return corsResponse;
    }
    
    // Handle the actual request
    const response = new Response('Hello World!');
    
    // Add CORS headers to the response
    response.headers.set('Access-Control-Allow-Origin', '*');
    
    return response;
  },
} satisfies ExportedHandler<Env>;
```

## Error Handling

Implement proper error handling in your worker:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    try {
      // Your worker logic here
      if (Math.random() < 0.5) {
        throw new Error('Random error for demonstration');
      }
      
      return new Response('Success!');
    } catch (error) {
      // Log the error
      console.error('Worker error:', error);
      
      // Return an error response
      return new Response(`Error: ${error.message}`, {
        status: 500,
        headers: {
          'Content-Type': 'text/plain'
        }
      });
    }
  },
} satisfies ExportedHandler<Env>;
```

## Using External APIs

You can make requests to external APIs from your worker:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    if (new URL(request.url).pathname === '/api/weather') {
      try {
        // Make a request to an external API
        const apiResponse = await fetch('https://api.example.com/weather?city=London');
        
        // Check if the request was successful
        if (!apiResponse.ok) {
          throw new Error(`API returned ${apiResponse.status}`);
        }
        
        // Get the response data
        const data = await apiResponse.json();
        
        // Return the data
        return new Response(JSON.stringify(data), {
          headers: {
            'Content-Type': 'application/json'
          }
        });
      } catch (error) {
        return new Response(`Error: ${error.message}`, { status: 500 });
      }
    }
    
    return new Response('Send a GET request to /api/weather');
  },
} satisfies ExportedHandler<Env>;
```

## Testing

Write tests for your worker using Vitest:

```typescript
// test/index.spec.ts
import { env, createExecutionContext, waitOnExecutionContext, SELF } from 'cloudflare:test';
import { describe, it, expect } from 'vitest';
import worker from '../src/index';

const IncomingRequest = Request<unknown, IncomingRequestCfProperties>;

describe('Worker tests', () => {
  it('responds with Hello World!', async () => {
    const request = new IncomingRequest('http://example.com');
    const ctx = createExecutionContext();
    const response = await worker.fetch(request, env, ctx);
    await waitOnExecutionContext(ctx);
    expect(await response.text()).toBe('Hello World!');
  });
  
  it('handles different HTTP methods', async () => {
    // Test GET request
    const getRequest = new IncomingRequest('http://example.com', {
      method: 'GET'
    });
    const getCtx = createExecutionContext();
    const getResponse = await worker.fetch(getRequest, env, getCtx);
    await waitOnExecutionContext(getCtx);
    expect(await getResponse.text()).toBe('This is a GET request');
    
    // Test POST request
    const postRequest = new IncomingRequest('http://example.com', {
      method: 'POST'
    });
    const postCtx = createExecutionContext();
    const postResponse = await worker.fetch(postRequest, env, postCtx);
    await waitOnExecutionContext(postCtx);
    expect(await postResponse.text()).toBe('This is a POST request');
  });
});
```

## Performance Optimization

Tips for optimizing your worker's performance:

1. **Minimize dependencies**: Each dependency increases your worker's size and cold start time
2. **Use caching**: Leverage Cloudflare's cache or implement your own caching logic
3. **Optimize network requests**: Minimize external API calls and use parallel requests when possible
4. **Use efficient data structures**: Choose appropriate data structures for your use case
5. **Implement timeouts**: Set timeouts for external API calls to prevent long-running requests

## Next Steps

Now that you understand how to develop your Cloudflare Worker, you might want to:

1. Explore the [Cloudflare Workers documentation](https://developers.cloudflare.com/workers/) for more advanced topics
2. Check out the [Examples](./examples.md) for more complex use cases
3. Learn about [deployment options](./deployment-guide.md) for your worker
