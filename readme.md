# Kong Gateway Setup with MCP (Model Context Protocol)

This gist demonstrates how to spin up Kong Gateway with MCP OAuth2 integration using Docker Compose and deck (Kong's declarative configuration tool).

## Architecture Overview

This setup includes:

- **Kong Gateway 3.12**: Connected to Kong Konnect (Control Plane)
- **Keycloak**: OAuth2/OIDC authentication server for MCP
- **MCP Proxy Service**: Weather API integration with OAuth2 protection
- **AI MCP Plugins**: OAuth2 and MCP proxy plugins for AI assistant integration

## Prerequisites

1. **Docker & Docker Compose** installed
2. **Kong Konnect Account** with control plane configured
3. **Deck CLI** installed (`brew install deck` or download from Kong)
4. **mkcert** for HTTPS certificates (`brew install mkcert nss`)
5. **Weather API Key** from weatherapi.com (for MCP weather service)
6. **MCP Inspector** (optional, for testing MCP integration)

## Quick Start

### 1. Starting MCP inspector

```sh
cd /gh/inspector
npm start

# run this in terminal - to disable CORS requirements when starting the browser on Chrome:
open -n -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --user-data-dir="/tmp/chrome_dev_test" --disable-web-security
```

### 2. Also check to ensure:

a. `/etc/hosts` file has below config:

```
127.0.0.1 keycloakmcp.local.test
127.0.0.1 kongdev.local.test
```

### 3. docker compose file already has resolved the same hostnames in point 1 to `host.docker.internal`

- to ensure consistency when resolving hostname of services in containers when accessing from browser on localhost or from within containers
- (i.e. Kong accesss keycloak introspection endpoints etc vs MCP inspector accesss keycloak auth server endpoints)

### 4. generate certs for keycloak to be accessible over https:

```bash
cd ./certs
brew install mkcert nss
mkcert -install
mkcert keycloakmcp.local.test
```

### 5. Update Kong Konnect Configuration

Edit `docker-compose.yaml` and update the Kong environment variables with your Konnect details:

```yaml
KONG_CLUSTER_CONTROL_PLANE: YOUR_CONTROL_PLANE_ENDPOINT
KONG_CLUSTER_SERVER_NAME: YOUR_CONTROL_PLANE_SERVER_NAME
KONG_CLUSTER_TELEMETRY_ENDPOINT: YOUR_TELEMETRY_ENDPOINT
KONG_CLUSTER_TELEMETRY_SERVER_NAME: YOUR_TELEMETRY_SERVER_NAME
KONG_CLUSTER_CERT: |
  -----BEGIN CERTIFICATE-----
  YOUR_CLUSTER_CERTIFICATE
  -----END CERTIFICATE-----
KONG_CLUSTER_CERT_KEY: |
  -----BEGIN PRIVATE KEY-----
  YOUR_PRIVATE_KEY
  -----END PRIVATE KEY-----
```

### 6. Edit `docker-compose.yaml` and update the keycloak environment variables with your admin details:

```yaml
environment:
  KEYCLOAK_ADMIN: changeme
  KEYCLOAK_ADMIN_PASSWORD: changeme
```

### 7. Update Configuration

Update the Keycloak client secret in `deck/mcp_oauth.yaml` to match your Keycloak realm in mcp/demo.json - line 626 and 633:

```yaml
# In the ai-mcp-oauth2 plugin config:
client_id: changeme
client_secret: changeme
```

### 8. Apply Kong Configuration with Deck

```bash
# Navigate to deck configuration directory
cd deck

# Validate configuration
deck validate

# Apply configuration to Kong Konnect
deck gateway sync deck.yaml --konnect-addr https://<changeme>.api.konghq.com --konnect-token $YOUR_KONNECT_TOKEN --konnect-control-plane-name $cp_name

```

### 8. Sign up for weather api api key access

Go to https://www.weatherapi.com/ and sign up for an account and create an api key to use in this demo

## Services & Ports

| Service      | Port | Purpose                           |
| ------------ | ---- | --------------------------------- |
| Kong Gateway | 8000 | HTTP Proxy                        |
| Keycloak     | 8441 | Auth Server HTTPS                 |
| Weather API  | 443  | External service (weatherapi.com) |

## Key Configuration Features

### MCP (Model Context Protocol) Integration

- **OAuth2 Protection**: Weather API protected with Keycloak OAuth2
- **MCP Proxy Plugin**: Converts API to MCP tool format for AI assistants
- **CORS Support**: Configured for MCP Inspector and browser clients
- **Tag-based Deployment**: Uses `mcpdemo` tag for selective deployment

## Testing the Setup

### 1. Fetch the proxy token from terminal when starting mcp inspector like below:

```bash
Starting MCP inspector...
⚙️ Proxy server listening on localhost:6277
🔑 Session token: 3760887cedd6e1d625de18935f43df7209ced43e7118ffa168a45cdb6c470a58
Use this token to authenticate requests or set DANGEROUSLY_OMIT_AUTH=true to disable auth

🚀 MCP Inspector is up and running at:
http://localhost:6274/?MCP_PROXY_AUTH_TOKEN=3760887cedd6e1d625de18935f43df7209ced43e7118ffa168a45cdb6c470a58

```

### 2. go to `localhost:6274` and choose:

- Transport Type: `Streamable HTTP`
- URL: `http://kongdev.local.test:8000/weather`
- Connection Type: `Via Proxy`

### 3. try to connect - see it fails and check dev tools to see 401 error, or see kong logs

### 4. go to `Open Auth Settings`

### 5. run the `guided oauth flow` or `quick oauth flow`

### 6. Paste the proxy token fron step 1 into the GUI of MCP inspector -> Configuration -> Proxy Session Token

### 7. Click Connect and see it works - you should see a `Connected` and be able to access the tools in the MCP server

## Configuration Files

### docker-compose.yaml

- Defines Kong Gateway and Keycloak containers
- Kong Gateway connected to Konnect control plane (requires credential updates)
- Keycloak with SSL certificates and demo realm import
- Minimal setup - no additional dependencies required

### deck/mcp_oauth.yaml

- Kong declarative configuration for MCP
- Weather API service with OAuth2 protection
- AI MCP OAuth2 and proxy plugins
- CORS configuration for MCP Inspector
- Tag-based deployment (`mcpdemo`)

### mcp/demo.json

- Keycloak realm configuration
- OAuth clients and user definitions
- Authentication flows and security settings
