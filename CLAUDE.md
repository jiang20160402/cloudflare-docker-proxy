# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `npm run dev` - Start development server using Wrangler on localhost:8787
- `npm run build` - Build the project using Webpack
- `npm run format` - Format code using Prettier

### Deployment
Deploy using Cloudflare Workers:
- Production environment: uses `env.production` configuration
- Staging environment: uses `env.staging` configuration 
- Development environment: uses `env.dev` configuration with debug mode

## Architecture

This is a Cloudflare Workers-based Docker registry proxy that acts as a caching layer and proxy for multiple Docker registries.

### Core Components

**Main Handler (`src/index.js`):**
- Event-driven architecture using `addEventListener("fetch")`
- Single file containing all proxy logic
- Handles Docker Registry v2 API requests

**Route Configuration:**
- `routes` object maps hostnames to upstream registry URLs
- Supports multiple registries: Docker Hub, Quay.io, GCR, GitHub Container Registry, etc.
- Route resolution via `routeByHosts()` function based on request hostname

**Authentication Flow:**
- Implements Docker Registry v2 authentication
- Handles `/v2/auth` endpoint for token exchange
- Parses `WWW-Authenticate` headers from upstream registries
- Special handling for Docker Hub library images (auto-prepends `library/` namespace)

**Request Processing:**
1. Route resolution based on hostname
2. Authentication check at `/v2/` endpoint
3. Token fetching at `/v2/auth` if required
4. Request forwarding to upstream registry
5. Special redirects for Docker Hub library images

### Environment Configuration

**wrangler.toml environments:**
- `dev`: Local development with debug mode and single upstream
- `staging`: Staging environment with single route
- `production`: Production with multiple custom domain routes

**Global Variables:**
- `MODE`: Controls debug behavior and authentication realm URLs
- `TARGET_UPSTREAM`: Fallback upstream URL in debug mode

### Key Features

- Multi-registry proxy support via hostname-based routing
- Docker Hub library image path normalization
- OAuth2/Bearer token authentication proxy
- Custom domain support for production deployments
- Debug mode for local development