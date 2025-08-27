# Dokploy Deployment Guide for Archon

This guide explains how to deploy Archon on Dokploy with its built-in Traefik reverse proxy. No additional Nginx or proxy configuration is needed.

## Prerequisites

1. Dokploy installed on your VPS
2. Domain names pointed to your VPS:
   - `archone.gnoviawan.com` → VPS IP
   - `api.archone.gnoviawan.com` → VPS IP
   - `mcp.archone.gnoviawan.com` → VPS IP
   - `agents.archone.gnoviawan.com` → VPS IP
3. Supabase project with credentials

## Deployment Options

### Option 1: Single Compose Stack (Recommended)

Deploy all services as one stack in Dokploy:

1. **Create a new Compose project in Dokploy**
   - Name: `archon-stack`
   - Type: Docker Compose

2. **Upload configuration files**
   ```
   docker-compose.dokploy-clean.yml
   .env.dokploy (renamed to .env)
   ```

3. **Configure environment in Dokploy UI**
   - Go to Environment Variables
   - Update these critical values:
   ```
   SUPABASE_URL=https://your-project.supabase.co
   SUPABASE_SERVICE_KEY=your-actual-service-key
   OPENAI_API_KEY=your-openai-key (optional)
   ```

4. **Set up domains in Dokploy**
   - For each service, configure the domain:
     - Frontend → `archone.gnoviawan.com`
     - API Server → `api.archone.gnoviawan.com`
     - MCP Server → `mcp.archone.gnoviawan.com`
     - Agents → `agents.archone.gnoviawan.com`

5. **Deploy the stack**
   ```bash
   docker compose -f docker-compose.dokploy-clean.yml up -d --build
   ```

### Option 2: Separate Applications

Deploy each service as a separate Dokploy application:

#### 1. Frontend Application
- **Name**: `archon-frontend`
- **Type**: Dockerfile
- **Dockerfile**: `Dockerfile.dokploy`
- **Domain**: `archone.gnoviawan.com`
- **Port**: 3000
- **Environment Variables**:
  ```
  VITE_ARCHON_SERVER_URL=https://api.archone.gnoviawan.com
  VITE_ARCHON_MCP_URL=https://mcp.archone.gnoviawan.com
  VITE_ARCHON_AGENTS_URL=https://agents.archone.gnoviawan.com
  VITE_PROD=true
  ```

#### 2. API Server Application
- **Name**: `archon-server`
- **Type**: Dockerfile
- **Dockerfile**: `python/Dockerfile.server`
- **Domain**: `api.archone.gnoviawan.com`
- **Port**: 8181
- **Environment Variables**:
  ```
  SUPABASE_URL=your-supabase-url
  SUPABASE_SERVICE_KEY=your-service-key
  CORS_ALLOWED_ORIGINS=https://archone.gnoviawan.com
  ```

#### 3. MCP Server Application
- **Name**: `archon-mcp`
- **Type**: Dockerfile
- **Dockerfile**: `python/Dockerfile.mcp`
- **Domain**: `mcp.archone.gnoviawan.com`
- **Port**: 8051

#### 4. Agents Service Application
- **Name**: `archon-agents`
- **Type**: Dockerfile
- **Dockerfile**: `python/Dockerfile.agents`
- **Domain**: `agents.archone.gnoviawan.com`
- **Port**: 8052

## Post-Deployment Configuration

### 1. Enable SSL/TLS
Dokploy's Traefik will automatically handle Let's Encrypt certificates. Ensure:
- Auto-SSL is enabled in Dokploy settings
- Each domain is properly configured

### 2. Configure CORS
The backend services need proper CORS configuration. This is handled via environment variables:
```
CORS_ALLOWED_ORIGINS=https://archone.gnoviawan.com,https://api.archone.gnoviawan.com
```

### 3. Health Checks
Verify all services are running:
```bash
# Check frontend
curl https://archone.gnoviawan.com/health

# Check API
curl https://api.archone.gnoviawan.com/health

# Check MCP
curl https://mcp.archone.gnoviawan.com/health

# Check Agents
curl https://agents.archone.gnoviawan.com/health
```

## Traefik Configuration (Automatic)

Dokploy automatically configures Traefik with:
- SSL termination
- HTTP to HTTPS redirect
- Load balancing
- Health checks

The Docker labels in `docker-compose.dokploy-clean.yml` tell Traefik:
- Which domain to route to which service
- Which port to use internally
- Enable the service in Traefik

## Troubleshooting

### Services not accessible
1. Check Dokploy's Traefik dashboard
2. Verify DNS records are pointing to your VPS
3. Check service logs in Dokploy

### CORS errors
1. Ensure all domains are in CORS_ALLOWED_ORIGINS
2. Check that services are using HTTPS
3. Verify frontend is using correct API URLs

### Database connection issues
1. Verify SUPABASE_URL is correct
2. Ensure you're using the SERVICE_KEY, not ANON_KEY
3. Check network connectivity to Supabase

### Container health checks failing
1. View logs: `docker logs archon-server`
2. Check required environment variables
3. Ensure services have started successfully

## Important Notes

1. **No port binding**: Services use `expose` instead of `ports` - Traefik handles external access
2. **No custom networks**: Dokploy manages Docker networking
3. **No Nginx needed**: Traefik handles all reverse proxy functionality
4. **Automatic SSL**: Traefik manages Let's Encrypt certificates

## Monitoring

Use Dokploy's built-in monitoring:
- Service status dashboard
- Container logs
- Resource usage
- Traefik routing table

## Updating

To update Archon:
1. Pull latest changes from repository
2. Rebuild images in Dokploy
3. Redeploy services
4. Check health endpoints

## Backup

Important data to backup:
- Environment variables (.env file)
- Supabase database
- Any uploaded documents (if stored locally)

## Security Recommendations

1. Use strong Supabase service key
2. Enable rate limiting in Traefik
3. Keep API keys secure
4. Regularly update Docker images
5. Monitor access logs
