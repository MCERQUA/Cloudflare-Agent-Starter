# Deployment Guide for Cloudflare Agents

This guide will walk you through the process of deploying your Cloudflare Worker agent to production and configuring various deployment options.

## Basic Deployment

The simplest way to deploy your worker is using the `npm run deploy` command, which runs `wrangler deploy` under the hood:

```bash
npm run deploy
```

This will:
1. Build your TypeScript code
2. Upload the worker to Cloudflare
3. Assign it a `*.workers.dev` subdomain based on your worker's name

After deployment, you'll see output similar to:

```
Uploaded agents-starter (1.23 sec)
Published agents-starter (0.45 sec)
  https://agents-starter.<your-subdomain>.workers.dev
```

## Authentication

Before you can deploy, you need to authenticate with Cloudflare:

```bash
npx wrangler login
```

This will open a browser window where you can log in to your Cloudflare account and authorize Wrangler.

If you need to log out or switch accounts:

```bash
npx wrangler logout
```

## Deployment Environments

You can set up different environments (e.g., staging, production) for your worker:

1. Add environments to your `wrangler.jsonc`:

```json
{
  "name": "agents-starter",
  "main": "src/index.ts",
  "compatibility_date": "2025-02-24",
  
  "env": {
    "staging": {
      "name": "agents-starter-staging",
      "vars": {
        "ENVIRONMENT": "staging",
        "API_URL": "https://staging-api.example.com"
      }
    },
    "production": {
      "name": "agents-starter",
      "vars": {
        "ENVIRONMENT": "production",
        "API_URL": "https://api.example.com"
      }
    }
  }
}
```

2. Deploy to a specific environment:

```bash
# Deploy to staging
npx wrangler deploy --env staging

# Deploy to production
npx wrangler deploy --env production
```

## Custom Domains

You can deploy your worker to a custom domain instead of using the default `*.workers.dev` subdomain:

1. Add the domain to your Cloudflare account:
   - Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/)
   - Add your domain and configure your DNS settings

2. Add the custom domain to your `wrangler.jsonc`:

```json
{
  "name": "agents-starter",
  "main": "src/index.ts",
  "compatibility_date": "2025-02-24",
  
  "routes": [
    {
      "pattern": "api.example.com/*",
      "zone_name": "example.com"
    }
  ]
}
```

3. Deploy your worker:

```bash
npm run deploy
```

Your worker will now be accessible at `https://api.example.com/*`.

## Route Patterns

You can deploy your worker to specific URL patterns:

```json
{
  "routes": [
    {
      "pattern": "api.example.com/v1/*",
      "zone_name": "example.com"
    }
  ]
}
```

This will only route requests matching `api.example.com/v1/*` to your worker.

## Deploying with GitHub Actions

You can automate deployments using GitHub Actions:

1. Create a `.github/workflows/deploy.yml` file:

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Deploy to Cloudflare Workers
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
```

2. Add your Cloudflare API token as a secret in your GitHub repository:
   - Generate an API token in the Cloudflare dashboard
   - Go to your GitHub repository settings
   - Add a new secret named `CF_API_TOKEN` with your Cloudflare API token

Now, whenever you push to the main branch, your worker will be automatically deployed.

## Rollbacks

If you need to roll back to a previous version of your worker:

1. List your worker versions:

```bash
npx wrangler versions list
```

2. Roll back to a specific version:

```bash
npx wrangler rollback --version <version-id>
```

## Scheduled Deployments

You can schedule your worker to run on a cron schedule:

1. Add a cron trigger to your `wrangler.jsonc`:

```json
{
  "triggers": {
    "crons": ["0 0 * * *"]  // Run at midnight every day
  }
}
```

2. Implement a scheduled handler in your worker:

```typescript
export default {
  async fetch(request, env, ctx): Promise<Response> {
    return new Response('Hello World!');
  },
  
  async scheduled(event, env, ctx) {
    // This runs on the schedule defined in wrangler.jsonc
    console.log('Running scheduled task:', event.cron);
    
    // Perform scheduled tasks
    await performDailyTasks(env);
    
    // No response is needed for scheduled events
  }
} satisfies ExportedHandler<Env>;

async function performDailyTasks(env) {
  // Implement your scheduled tasks here
  console.log('Performing daily tasks');
}
```

## Resource Bindings

You can bind various Cloudflare resources to your worker:

### KV Namespaces

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

### R2 Buckets

```json
{
  "r2_buckets": [
    {
      "binding": "MY_BUCKET",
      "bucket_name": "your-bucket-name"
    }
  ]
}
```

### D1 Databases

```json
{
  "d1_databases": [
    {
      "binding": "MY_DB",
      "database_id": "your-database-id"
    }
  ]
}
```

### Durable Objects

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

## Environment Variables and Secrets

### Environment Variables

Add environment variables to your `wrangler.jsonc`:

```json
{
  "vars": {
    "API_URL": "https://api.example.com",
    "DEBUG_MODE": "false"
  }
}
```

### Secrets

For sensitive information, use secrets instead of environment variables:

```bash
npx wrangler secret put API_KEY
```

This will prompt you to enter the secret value, which will be securely stored and made available to your worker.

## Deployment Monitoring

After deploying your worker, you can monitor its performance and errors:

1. Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/)
2. Navigate to Workers & Pages
3. Select your worker
4. View metrics, logs, and errors

## Deployment Limits

Be aware of the following limits when deploying workers:

- **Script Size**: Up to 1MB for the bundled worker
- **Number of Workers**: Varies by plan (Free plan: 30 workers)
- **Subrequests**: Up to 50 subrequests per worker invocation
- **CPU Time**: Up to 30 seconds per invocation
- **Memory**: Up to 128MB per worker

## Deployment Best Practices

1. **Use versioning**: Keep track of your worker versions
2. **Test before deploying**: Run tests locally before deploying
3. **Use staging environments**: Test changes in a staging environment before production
4. **Automate deployments**: Use CI/CD pipelines for consistent deployments
5. **Monitor performance**: Keep an eye on your worker's performance and errors
6. **Implement gradual rollouts**: Use traffic splitting for critical changes
7. **Document deployments**: Keep a record of what was deployed and when

## Troubleshooting Deployments

### Error: Unable to fetch account ID

Make sure you're logged in with `npx wrangler login`.

### Error: Worker size limit exceeded

Reduce the size of your worker by:
- Removing unnecessary dependencies
- Optimizing your code
- Splitting into multiple workers

### Error: Too many requests

You might be hitting rate limits. Wait a few minutes and try again.

### Error: Invalid configuration

Check your `wrangler.jsonc` for syntax errors or invalid configuration.

## Further Reading

- [Wrangler Configuration Documentation](https://developers.cloudflare.com/workers/wrangler/configuration/)
- [Cloudflare Workers Deployment Documentation](https://developers.cloudflare.com/workers/deployment/)
- [Cloudflare Workers Limits Documentation](https://developers.cloudflare.com/workers/platform/limits/)
