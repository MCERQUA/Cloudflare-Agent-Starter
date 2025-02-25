# Cloudflare Workers Examples

This document provides practical examples of how to use Cloudflare Workers for different use cases. Each example includes a complete code sample that you can adapt for your own projects.

## Table of Contents

1. [Basic API Server](#basic-api-server)
2. [HTML Website](#html-website)
3. [JSON API with CRUD Operations](#json-api-with-crud-operations)
4. [Authentication Middleware](#authentication-middleware)
5. [Image Transformation](#image-transformation)
6. [A/B Testing](#ab-testing)
7. [Geolocation-based Routing](#geolocation-based-routing)
8. [Scheduled Tasks](#scheduled-tasks)
9. [WebSocket Proxy](#websocket-proxy)
10. [Integrating with External APIs](#integrating-with-external-apis)

## Basic API Server

A simple API server that handles different routes and methods:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    const method = request.method;
    
    // API routes
    if (path.startsWith('/api')) {
      // Users endpoint
      if (path === '/api/users') {
        if (method === 'GET') {
          return new Response(JSON.stringify({
            users: [
              { id: 1, name: 'Alice' },
              { id: 2, name: 'Bob' },
              { id: 3, name: 'Charlie' }
            ]
          }), {
            headers: { 'Content-Type': 'application/json' }
          });
        } else if (method === 'POST') {
          try {
            const body = await request.json();
            return new Response(JSON.stringify({
              message: 'User created',
              user: body
            }), {
              status: 201,
              headers: { 'Content-Type': 'application/json' }
            });
          } catch (error) {
            return new Response(JSON.stringify({
              error: 'Invalid JSON'
            }), {
              status: 400,
              headers: { 'Content-Type': 'application/json' }
            });
          }
        }
      }
      
      // Products endpoint
      if (path === '/api/products') {
        if (method === 'GET') {
          return new Response(JSON.stringify({
            products: [
              { id: 1, name: 'Laptop', price: 999 },
              { id: 2, name: 'Phone', price: 699 },
              { id: 3, name: 'Tablet', price: 499 }
            ]
          }), {
            headers: { 'Content-Type': 'application/json' }
          });
        }
      }
      
      // Default API response
      return new Response(JSON.stringify({
        error: 'Not found'
      }), {
        status: 404,
        headers: { 'Content-Type': 'application/json' }
      });
    }
    
    // Default response for non-API routes
    return new Response('Welcome to the API server. Use /api/users or /api/products endpoints.');
  },
} satisfies ExportedHandler<Env>;
```

## HTML Website

A simple HTML website with multiple pages:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // HTML content for different pages
    const pages = {
      '/': `
        <!DOCTYPE html>
        <html>
          <head>
            <title>My Website</title>
            <style>
              body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
              nav { margin-bottom: 20px; }
              nav a { margin-right: 10px; }
            </style>
          </head>
          <body>
            <nav>
              <a href="/">Home</a>
              <a href="/about">About</a>
              <a href="/contact">Contact</a>
            </nav>
            <h1>Welcome to My Website</h1>
            <p>This is a simple website served by a Cloudflare Worker.</p>
          </body>
        </html>
      `,
      '/about': `
        <!DOCTYPE html>
        <html>
          <head>
            <title>About - My Website</title>
            <style>
              body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
              nav { margin-bottom: 20px; }
              nav a { margin-right: 10px; }
            </style>
          </head>
          <body>
            <nav>
              <a href="/">Home</a>
              <a href="/about">About</a>
              <a href="/contact">Contact</a>
            </nav>
            <h1>About Us</h1>
            <p>We are a company that specializes in Cloudflare Workers.</p>
          </body>
        </html>
      `,
      '/contact': `
        <!DOCTYPE html>
        <html>
          <head>
            <title>Contact - My Website</title>
            <style>
              body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
              nav { margin-bottom: 20px; }
              nav a { margin-right: 10px; }
            </style>
          </head>
          <body>
            <nav>
              <a href="/">Home</a>
              <a href="/about">About</a>
              <a href="/contact">Contact</a>
            </nav>
            <h1>Contact Us</h1>
            <p>Email: contact@example.com</p>
          </body>
        </html>
      `
    };
    
    // Return the requested page or a 404 page
    if (pages[path]) {
      return new Response(pages[path], {
        headers: { 'Content-Type': 'text/html' }
      });
    } else {
      return new Response(`
        <!DOCTYPE html>
        <html>
          <head>
            <title>404 - Page Not Found</title>
            <style>
              body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
              nav { margin-bottom: 20px; }
              nav a { margin-right: 10px; }
            </style>
          </head>
          <body>
            <nav>
              <a href="/">Home</a>
              <a href="/about">About</a>
              <a href="/contact">Contact</a>
            </nav>
            <h1>404 - Page Not Found</h1>
            <p>The page you requested does not exist.</p>
          </body>
        </html>
      `, {
        status: 404,
        headers: { 'Content-Type': 'text/html' }
      });
    }
  },
} satisfies ExportedHandler<Env>;
```

## JSON API with CRUD Operations

A more complete API with Create, Read, Update, Delete operations using Cloudflare KV:

```typescript
// First, update your worker-configuration.d.ts:
interface Env {
  USERS_KV: KVNamespace;
}

// Then, implement the worker:
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    const method = request.method;
    
    // Add CORS headers
    const corsHeaders = {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type',
    };
    
    // Handle OPTIONS request for CORS
    if (method === 'OPTIONS') {
      return new Response(null, {
        headers: corsHeaders
      });
    }
    
    // Handle /api/users endpoint
    if (path.startsWith('/api/users')) {
      // Get a specific user by ID
      if (path.match(/^\/api\/users\/[a-zA-Z0-9-]+$/) && method === 'GET') {
        const userId = path.split('/').pop();
        const user = await env.USERS_KV.get(userId);
        
        if (!user) {
          return new Response(JSON.stringify({ error: 'User not found' }), {
            status: 404,
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        }
        
        return new Response(user, {
          headers: { 'Content-Type': 'application/json', ...corsHeaders }
        });
      }
      
      // Update a specific user by ID
      if (path.match(/^\/api\/users\/[a-zA-Z0-9-]+$/) && method === 'PUT') {
        const userId = path.split('/').pop();
        const existingUser = await env.USERS_KV.get(userId);
        
        if (!existingUser) {
          return new Response(JSON.stringify({ error: 'User not found' }), {
            status: 404,
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        }
        
        try {
          const updatedUser = await request.json();
          await env.USERS_KV.put(userId, JSON.stringify(updatedUser));
          
          return new Response(JSON.stringify(updatedUser), {
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        } catch (error) {
          return new Response(JSON.stringify({ error: 'Invalid JSON' }), {
            status: 400,
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        }
      }
      
      // Delete a specific user by ID
      if (path.match(/^\/api\/users\/[a-zA-Z0-9-]+$/) && method === 'DELETE') {
        const userId = path.split('/').pop();
        const existingUser = await env.USERS_KV.get(userId);
        
        if (!existingUser) {
          return new Response(JSON.stringify({ error: 'User not found' }), {
            status: 404,
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        }
        
        await env.USERS_KV.delete(userId);
        
        return new Response(JSON.stringify({ message: 'User deleted' }), {
          headers: { 'Content-Type': 'application/json', ...corsHeaders }
        });
      }
      
      // List all users
      if (path === '/api/users' && method === 'GET') {
        const { keys } = await env.USERS_KV.list();
        const users = [];
        
        for (const key of keys) {
          const user = await env.USERS_KV.get(key.name);
          if (user) {
            users.push(JSON.parse(user));
          }
        }
        
        return new Response(JSON.stringify({ users }), {
          headers: { 'Content-Type': 'application/json', ...corsHeaders }
        });
      }
      
      // Create a new user
      if (path === '/api/users' && method === 'POST') {
        try {
          const newUser = await request.json();
          
          // Generate a unique ID
          const userId = crypto.randomUUID();
          newUser.id = userId;
          
          await env.USERS_KV.put(userId, JSON.stringify(newUser));
          
          return new Response(JSON.stringify(newUser), {
            status: 201,
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        } catch (error) {
          return new Response(JSON.stringify({ error: 'Invalid JSON' }), {
            status: 400,
            headers: { 'Content-Type': 'application/json', ...corsHeaders }
          });
        }
      }
    }
    
    // Default response for unknown routes
    return new Response(JSON.stringify({ error: 'Not found' }), {
      status: 404,
      headers: { 'Content-Type': 'application/json', ...corsHeaders }
    });
  },
} satisfies ExportedHandler<Env>;
```

## Authentication Middleware

A worker that implements JWT authentication:

```typescript
// First, update your worker-configuration.d.ts:
interface Env {
  JWT_SECRET: string;
}

// Then, implement the worker:
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // Public routes
    if (path === '/login') {
      if (request.method === 'POST') {
        try {
          const { username, password } = await request.json();
          
          // In a real application, you would validate against a database
          if (username === 'admin' && password === 'password') {
            // Create a JWT token
            const token = await createJWT(
              { sub: 'admin', role: 'admin' },
              env.JWT_SECRET
            );
            
            return new Response(JSON.stringify({ token }), {
              headers: { 'Content-Type': 'application/json' }
            });
          } else {
            return new Response(JSON.stringify({ error: 'Invalid credentials' }), {
              status: 401,
              headers: { 'Content-Type': 'application/json' }
            });
          }
        } catch (error) {
          return new Response(JSON.stringify({ error: 'Invalid JSON' }), {
            status: 400,
            headers: { 'Content-Type': 'application/json' }
          });
        }
      } else {
        return new Response(JSON.stringify({ error: 'Method not allowed' }), {
          status: 405,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    // Protected routes
    if (path.startsWith('/api')) {
      // Verify JWT token
      const authHeader = request.headers.get('Authorization');
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return new Response(JSON.stringify({ error: 'Unauthorized' }), {
          status: 401,
          headers: { 'Content-Type': 'application/json' }
        });
      }
      
      const token = authHeader.split(' ')[1];
      try {
        const payload = await verifyJWT(token, env.JWT_SECRET);
        
        // Add user info to request
        const newRequest = new Request(request);
        newRequest.user = payload;
        
        // Handle protected routes
        if (path === '/api/profile') {
          return new Response(JSON.stringify({
            message: 'Protected data',
            user: payload
          }), {
            headers: { 'Content-Type': 'application/json' }
          });
        }
        
        return new Response(JSON.stringify({ error: 'Not found' }), {
          status: 404,
          headers: { 'Content-Type': 'application/json' }
        });
      } catch (error) {
        return new Response(JSON.stringify({ error: 'Invalid token' }), {
          status: 401,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    // Default response
    return new Response('Welcome to the API. Please login to access protected routes.');
  },
} satisfies ExportedHandler<Env>;

// JWT helper functions
async function createJWT(payload, secret) {
  const encoder = new TextEncoder();
  const header = { alg: 'HS256', typ: 'JWT' };
  
  // Add expiration time (1 hour)
  payload.exp = Math.floor(Date.now() / 1000) + 3600;
  payload.iat = Math.floor(Date.now() / 1000);
  
  const encodedHeader = btoa(JSON.stringify(header));
  const encodedPayload = btoa(JSON.stringify(payload));
  
  const signature = await crypto.subtle.sign(
    { name: 'HMAC', hash: 'SHA-256' },
    await crypto.subtle.importKey(
      'raw',
      encoder.encode(secret),
      { name: 'HMAC', hash: 'SHA-256' },
      false,
      ['sign']
    ),
    encoder.encode(`${encodedHeader}.${encodedPayload}`)
  );
  
  const encodedSignature = btoa(String.fromCharCode(...new Uint8Array(signature)));
  
  return `${encodedHeader}.${encodedPayload}.${encodedSignature}`;
}

async function verifyJWT(token, secret) {
  const [encodedHeader, encodedPayload, encodedSignature] = token.split('.');
  
  const encoder = new TextEncoder();
  const signature = new Uint8Array(
    atob(encodedSignature)
      .split('')
      .map(c => c.charCodeAt(0))
  );
  
  const isValid = await crypto.subtle.verify(
    { name: 'HMAC', hash: 'SHA-256' },
    await crypto.subtle.importKey(
      'raw',
      encoder.encode(secret),
      { name: 'HMAC', hash: 'SHA-256' },
      false,
      ['verify']
    ),
    signature,
    encoder.encode(`${encodedHeader}.${encodedPayload}`)
  );
  
  if (!isValid) {
    throw new Error('Invalid signature');
  }
  
  const payload = JSON.parse(atob(encodedPayload));
  
  // Check if token is expired
  if (payload.exp && payload.exp < Math.floor(Date.now() / 1000)) {
    throw new Error('Token expired');
  }
  
  return payload;
}
```

## Image Transformation

A worker that resizes and optimizes images on the fly:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // Only process image requests
    if (path.match(/\.(jpg|jpeg|png|gif|webp)$/i)) {
      // Parse query parameters
      const width = parseInt(url.searchParams.get('width') || '0');
      const height = parseInt(url.searchParams.get('height') || '0');
      const quality = parseInt(url.searchParams.get('quality') || '80');
      const format = url.searchParams.get('format') || 'webp';
      
      // If no transformation parameters, return the original image
      if (!width && !height) {
        // Fetch the original image from the origin
        const originUrl = `https://example.com${path}`;
        const originResponse = await fetch(originUrl);
        
        if (!originResponse.ok) {
          return new Response('Image not found', { status: 404 });
        }
        
        return originResponse;
      }
      
      // Fetch the original image from the origin
      const originUrl = `https://example.com${path}`;
      const originResponse = await fetch(originUrl);
      
      if (!originResponse.ok) {
        return new Response('Image not found', { status: 404 });
      }
      
      // Get the image data
      const imageData = await originResponse.arrayBuffer();
      
      // In a real implementation, you would use a library like Sharp or Jimp
      // to resize and optimize the image. Since we can't do that directly in
      // this example, we'll just return the original image with a note.
      
      return new Response(
        `This is where image transformation would happen:
        - Original image: ${path}
        - Width: ${width || 'auto'}
        - Height: ${height || 'auto'}
        - Quality: ${quality}
        - Format: ${format}
        
        In a real implementation, you would use a service like Cloudflare Images
        or a third-party image processing service.`,
        { headers: { 'Content-Type': 'text/plain' } }
      );
    }
    
    // Default response for non-image requests
    return new Response('Image transformation service. Append width, height, quality, and format parameters to image URLs.');
  },
} satisfies ExportedHandler<Env>;
```

## A/B Testing

A worker that implements A/B testing for a website:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Get or set the test variant from a cookie
    let variant = getVariantFromCookie(request);
    
    if (!variant) {
      // Randomly assign a variant (A or B)
      variant = Math.random() < 0.5 ? 'A' : 'B';
    }
    
    // Fetch the original page
    const originResponse = await fetch(request);
    const originBody = await originResponse.text();
    
    // Create a response with the modified content
    let modifiedBody;
    
    if (variant === 'A') {
      // Variant A: Original version
      modifiedBody = originBody;
    } else {
      // Variant B: Modified version
      modifiedBody = originBody
        .replace(
          '<h1>Welcome to our Website</h1>',
          '<h1>Experience our Amazing Website</h1>'
        )
        .replace(
          '<button>Sign Up</button>',
          '<button style="background-color: red; color: white;">Sign Up Now</button>'
        );
    }
    
    // Create a new response with the modified body
    const response = new Response(modifiedBody, originResponse);
    
    // Set the variant cookie
    response.headers.set(
      'Set-Cookie',
      `variant=${variant}; path=/; max-age=3600; SameSite=Strict`
    );
    
    // Add a custom header for tracking
    response.headers.set('X-AB-Variant', variant);
    
    return response;
  },
} satisfies ExportedHandler<Env>;

// Helper function to get the variant from a cookie
function getVariantFromCookie(request) {
  const cookieHeader = request.headers.get('Cookie');
  if (!cookieHeader) return null;
  
  const cookies = cookieHeader.split(';');
  for (const cookie of cookies) {
    const [name, value] = cookie.trim().split('=');
    if (name === 'variant') {
      return value;
    }
  }
  
  return null;
}
```

## Geolocation-based Routing

A worker that serves different content based on the user's location:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Get the user's country from the CF-IPCountry header
    const country = request.headers.get('CF-IPCountry') || 'XX';
    
    // Define content for different regions
    const regions = {
      US: {
        currency: 'USD',
        greeting: 'Hello',
        offers: ['Free shipping on orders over $50', '20% off summer collection']
      },
      CA: {
        currency: 'CAD',
        greeting: 'Hello',
        offers: ['Free shipping on orders over $60 CAD', '15% off winter collection']
      },
      GB: {
        currency: 'GBP',
        greeting: 'Hello',
        offers: ['Free shipping on orders over £40', '10% off for new customers']
      },
      FR: {
        currency: 'EUR',
        greeting: 'Bonjour',
        offers: ['Livraison gratuite pour les commandes de plus de 45€', '15% de réduction sur la nouvelle collection']
      },
      DE: {
        currency: 'EUR',
        greeting: 'Hallo',
        offers: ['Kostenloser Versand ab 45€', '10% Rabatt für Neukunden']
      },
      // Default for unknown countries
      XX: {
        currency: 'USD',
        greeting: 'Hello',
        offers: ['International shipping available', '10% off your first order']
      }
    };
    
    // Get the region-specific content
    const regionContent = regions[country] || regions.XX;
    
    // Create an HTML response with the region-specific content
    const html = `
      <!DOCTYPE html>
      <html>
        <head>
          <title>Welcome to our Store</title>
          <style>
            body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
            .offers { margin-top: 20px; }
            .offer { background-color: #f0f0f0; padding: 10px; margin-bottom: 10px; border-radius: 5px; }
          </style>
        </head>
        <body>
          <h1>${regionContent.greeting} from Our Store</h1>
          <p>We've detected that you're browsing from ${country}.</p>
          <p>All prices are shown in ${regionContent.currency}.</p>
          
          <div class="offers">
            <h2>Special Offers for Your Region</h2>
            ${regionContent.offers.map(offer => `<div class="offer">${offer}</div>`).join('')}
          </div>
          
          <p>You can change your region in the settings if needed.</p>
        </body>
      </html>
    `;
    
    return new Response(html, {
      headers: { 'Content-Type': 'text/html' }
    });
  },
} satisfies ExportedHandler<Env>;
```

## Scheduled Tasks

A worker that performs scheduled tasks using Cron Triggers:

```typescript
// First, update your worker-configuration.d.ts:
interface Env {
  REPORTS_KV: KVNamespace;
}

// Then, implement the worker:
export default {
  // Handle HTTP requests
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    
    // Endpoint to view the latest report
    if (url.pathname === '/latest-report') {
      const report = await env.REPORTS_KV.get('latest_report');
      
      if (!report) {
        return new Response('No report available yet', { status: 404 });
      }
      
      return new Response(report, {
        headers: { 'Content-Type': 'application/json' }
      });
    }
    
    // Default response
    return new Response('Scheduled task worker. Visit /latest-report to see the latest generated report.');
  },
  
  // Handle scheduled events
  async scheduled(event, env, ctx) {
    console.log(`Running scheduled task: ${event.cron}`);
    
    // Different tasks based on the cron schedule
    if (event.cron === '0 0 * * *') {
      // Daily task at midnight
      await generateDailyReport(env);
    } else if (event.cron === '0 0 * * 0') {
      // Weekly task on Sundays
      await generateWeeklyReport(env);
    } else if (event.cron === '0 0 1 * *') {
      // Monthly task on the 1st of each month
      await generateMonthlyReport(env);
    }
  }
} satisfies ExportedHandler<Env>;

// Helper function to generate a daily report
async function generateDailyReport(env) {
  // In a real application, you would fetch data from various sources
  // and generate a comprehensive report
  
  const report = {
    type: 'daily',
    generated_at: new Date().toISOString(),
    data: {
      visitors: Math.floor(Math.random() * 1000),
      pageviews: Math.floor(Math.random() * 5000),
      conversions: Math.floor(Math.random() * 100),
      revenue: Math.floor(Math.random() * 10000) / 100
    }
  };
  
  // Store the report in KV
  await env.REPORTS_KV.put('latest_report', JSON.stringify(report));
  await env.REPORTS_KV.put(`report_${report.generated_at}`, JSON.stringify(report));
  
  console.log('Daily report generated and stored');
}

// Helper function to generate a weekly report
async function generateWeeklyReport(env) {
  // Similar to daily report but with weekly aggregated data
  const report = {
    type: 'weekly',
    generated_at: new Date().toISOString(),
    data: {
      visitors: Math.floor(Math.random() * 7000),
      pageviews: Math.floor(Math.random() * 35000),
      conversions: Math.floor(Math.random() * 700),
      revenue: Math.floor(Math.random() * 70000) / 100,
      top_pages: [
        { url: '/product-1', views: Math.floor(Math.random() * 5000) },
        { url: '/product-2', views: Math.floor(Math.random() * 4000) },
        { url: '/product-3', views: Math.floor(Math.random() * 3000) }
      ]
    }
  };
  
  // Store the report in KV
  await env.REPORTS_KV.put('latest_report', JSON.stringify(report));
  await env.REPORTS_KV.put(`report_${report.generated_at}`, JSON.stringify(report));
  
  console.log('Weekly report generated and stored');
}

// Helper function to generate a monthly report
async function generateMonthlyReport(env) {
  // Similar to weekly report but with monthly aggregated data
  const report = {
    type: 'monthly',
    generated_at: new Date().toISOString(),
    data: {
      visitors: Math.floor(Math.random() * 30000),
      pageviews: Math.floor(Math.random() * 150000),
      conversions: Math.floor(Math.random() * 3000),
      revenue: Math.floor(Math.random() * 300000) / 100,
      top_products: [
        { name: 'Product 1', sales: Math.floor(Math.random() * 1000) },
        { name: 'Product 2', sales: Math.floor(Math.random() * 800) },
        { name: 'Product 3', sales: Math.floor(Math.random() * 600) }
      ],
      growth: {
        visitors: Math.floor(Math.random() * 30) - 10,
        revenue: Math.floor(Math.random() * 30) - 10
      }
    }
  };
  
  // Store the report in KV
  await env.REPORTS_KV.put('latest_report', JSON.stringify(report));
  await env.REPORTS_KV.put(`report_${report.generated_at}`, JSON.stringify(report));
  
  console.log('Monthly report generated and stored');
}
```

## WebSocket Proxy

A worker that proxies WebSocket connections:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const upgradeHeader = request.headers.get('Upgrade');
    
    if (!upgradeHeader || upgradeHeader !== 'websocket') {
      return new Response('Expected Upgrade: websocket', { status: 426 });
    }
    
    // Create a new WebSocket pair
    const [client, server] = Object.values(new WebSocketPair());
    
    // Accept the client connection
    server.accept();
    
    // Set up event handlers for the server WebSocket
    server.addEventListener('message', async event => {
      try {
        // Log the message
        console.log('Received message from client:', event.data);
        
        // Echo the message back to the client
        server.send(`Echo: ${event.data}`);
        
        // You could also forward the message to another WebSocket server
        // const backendSocket = new WebSocket('wss://backend.example.com');
        // backendSocket.send(event.data);
      } catch (error) {
        console.error('Error handling message:', error);
      }
    });
    
    server.addEventListener('close', event => {
      console.log('WebSocket closed with code:', event.code);
    });
    
    server.addEventListener('error', event => {
      console.error('WebSocket error:', event);
    });
    
    // Return the client WebSocket as the response
    return new Response(null, {
      status: 101,
      webSocket: client
    });
  },
} satisfies ExportedHandler<Env>;
```

## Integrating with External APIs

A worker that integrates with external APIs:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // Weather API endpoint
    if (path === '/api/weather') {
      const city = url.searchParams.get('city') || 'London';
      
      try {
        // Make a request to an external weather API
        const apiResponse = await fetch(
          `https://api.weatherapi.com/v1/current.json?key=${env.WEATHER_API_KEY}&q=${city}`
        );
        
        if (!apiResponse.ok) {
          throw new Error(`Weather API returned ${apiResponse.status}`);
        }
        
        const data = await apiResponse.json();
        
        // Transform the data
        const transformedData = {
          location: data.location.name,
          country: data.location.country,
          temperature: {
            celsius: data.current.temp_c,
            fahrenheit: data.current.temp_f
          },
          condition: data.current.condition.text,
          humidity: data.current.humidity,
          wind: {
            kph: data.current.wind_kph,
            mph: data.current.wind_mph,
            direction: data.current.wind_dir
          },
          updated: data.current.last_updated
        };
        
        // Return the transformed data
        return new Response(JSON.stringify(transformedData), {
          headers: {
            'Content-Type': 'application/json',
            'Cache-Control': 'max-age=300' // Cache for 5 minutes
          }
        });
      } catch (error) {
        return new Response(JSON.stringify({ error: error.message }), {
          status: 500,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    // News API endpoint
    if (path === '/api/news') {
      const category = url.searchParams.get('category') || 'technology';
      
      try {
        // Make a request to an external news API
        const apiResponse = await fetch(
          `https://newsapi.org/v2/top-headlines?category=${category}&apiKey=${env.NEWS_API_KEY}`
        );
        
        if (!apiResponse.ok) {
          throw new Error(`News API returned ${apiResponse.status}`);
        }
        
        const data = await apiResponse.json();
        
        // Transform and filter the data
        const transformedData = {
          category,
          articles: data.articles.map(article => ({
            title: article.title,
            description: article.description,
            url: article.url,
            source: article.source.name,
            published: article.publishedAt
          })).slice(0, 10) // Limit to 10 articles
        };
        
        // Return the transformed data
        return new Response(JSON.stringify(transformedData), {
          headers: {
            'Content-Type': 'application/json',
            'Cache-Control': 'max-age=1800' // Cache for 30 minutes
          }
        });
      } catch (error) {
        return new Response(JSON.stringify({ error: error.message }), {
          status: 500,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    // Currency conversion endpoint
    if (path === '/api/currency') {
      const from = url.searchParams.get('from') || 'USD';
      const to = url.searchParams.get('to') || 'EUR';
      const amount = parseFloat(url.searchParams.get('amount') || '1');
      
      try {
        // Make a request to an external currency API
        const apiResponse = await fetch(
          `https://api.exchangerate.host/convert?from=${from}&to=${to}&amount=${amount}`
        );
        
        if (!apiResponse.ok) {
          throw new Error(`Currency API returned ${apiResponse.status}`);
        }
        
        const data = await apiResponse.json();
        
        // Return the conversion result
        return new Response(JSON.stringify({
          from,
          to,
          amount,
          result: data.result,
          rate: data.info.rate,
          timestamp: data.info.timestamp
        }), {
          headers: {
            'Content-Type': 'application/json',
            'Cache-Control': 'max-age=3600' // Cache for 1 hour
          }
        });
      } catch (error) {
        return new Response(JSON.stringify({ error: error.message }), {
          status: 500,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    // Default response
    return new Response('API Gateway. Available endpoints: /api/weather, /api/news, /api/currency');
  },
} satisfies ExportedHandler<Env>;
```
