# Getting Started with Your Cloudflare Agent

This guide will walk you through the process of setting up, running, and deploying your Cloudflare Worker agent.

## Prerequisites

Before you begin, make sure you have the following installed:

- [Node.js](https://nodejs.org/) (version 16 or later)
- [npm](https://www.npmjs.com/) (comes with Node.js)
- A [Cloudflare account](https://dash.cloudflare.com/sign-up) (free tier is sufficient to start)

## Setup

1. **Clone or download the project**

   If you haven't already, clone or download this project to your local machine.

2. **Install dependencies**

   Navigate to the project directory and install the dependencies:

   ```bash
   cd agents-starter
   npm install
   ```

3. **Configure your Cloudflare account (for deployment)**

   To deploy your worker, you'll need to authenticate with Cloudflare:

   ```bash
   npx wrangler login
   ```

   This will open a browser window where you can log in to your Cloudflare account and authorize Wrangler.

## Local Development

1. **Start the development server**

   Run the following command to start a local development server:

   ```bash
   npm run dev
   ```

   This will start a local server at http://localhost:8787 where you can test your worker.

2. **Make changes to your worker**

   Open `src/index.ts` in your code editor and make changes to the worker code. The development server will automatically reload when you save changes.

3. **Test your worker**

   You can test your worker by sending requests to http://localhost:8787. For example:

   ```bash
   curl http://localhost:8787
   ```

   This should return "Hello World!" based on the default implementation.

4. **Run tests**

   You can run the tests with:

   ```bash
   npm test
   ```

## Understanding the Code

The main worker code is in `src/index.ts`. Here's a breakdown of the default implementation:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    return new Response('Hello World!');
  },
} satisfies ExportedHandler<Env>;
```

This simple worker responds to all HTTP requests with "Hello World!". Here's what each part does:

- `fetch`: This is the main handler function that processes incoming HTTP requests
- `request`: Contains information about the incoming HTTP request
- `env`: Contains environment variables and bindings defined in `wrangler.jsonc`
- `ctx`: Provides methods like `waitUntil()` for handling background tasks
- `Response`: Creates an HTTP response to send back to the client

## Deployment

When you're ready to deploy your worker to Cloudflare's global network:

1. **Deploy your worker**

   ```bash
   npm run deploy
   ```

   This will build your worker and deploy it to Cloudflare.

2. **View your deployed worker**

   After deployment, Wrangler will output a URL where your worker is accessible, typically in the format:
   
   ```
   https://agents-starter.<your-subdomain>.workers.dev
   ```

   You can visit this URL in your browser or use tools like curl to interact with your deployed worker.

## Next Steps

Now that you have your worker up and running, you might want to:

1. Modify the worker to handle different types of requests (see the [Development Guide](./development-guide.md))
2. Add bindings to connect to Cloudflare services like KV, R2, or D1 (see the [Development Guide](./development-guide.md))
3. Set up custom domains for your worker (see the [Deployment Guide](./deployment-guide.md))
4. Explore more complex examples (see the [Examples](./examples.md))

## Troubleshooting

- **Error: Cannot find wrangler configuration file**
  
  Make sure you're running commands from the project root directory where `wrangler.jsonc` is located.

- **Error: Failed to deploy worker**
  
  Ensure you've logged in with `npx wrangler login` and have the necessary permissions in your Cloudflare account.

- **Error: Cannot connect to localhost:8787**
  
  Check that the development server is running and that no other service is using port 8787.

## Getting Help

If you encounter issues not covered here:

- Check the [Cloudflare Workers documentation](https://developers.cloudflare.com/workers/)
- Visit the [Cloudflare Workers Discord](https://discord.gg/cloudflaredev)
- Search for solutions on the [Cloudflare Community forums](https://community.cloudflare.com/)
