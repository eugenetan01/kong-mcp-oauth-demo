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

1. Starting MCP inspector

   ```sh
   cd /gh/inspector
   npm start
   ```

2. generate certs for keycloak to be accessible over https - this is required for keycloak to start:

   ```bash
   cd ./certs
   brew install mkcert nss
   mkcert -install
   mkcert localhost
   ```

3. Update Kong Konnect Configuration

   - Edit `docker-compose.yaml` and update the Kong environment variables with your Konnect details:

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

4. Edit `docker-compose.yaml` and update the keycloak environment variables with your admin details or leave it as it is:

   ```yaml
   environment:
     KEYCLOAK_ADMIN: changeme
     KEYCLOAK_ADMIN_PASSWORD: changeme
   ```

5. Update Configuration

   - Update the Keycloak client secret in `deck/mcp_oauth.yaml` to match your Keycloak realm in mcp/demo.json - line 626 and 633 or leave it as it is:

   ```yaml
   # In the ai-mcp-oauth2 plugin config:
   client_id: changeme
   client_secret: changeme
   ```

6. Sign up for weather api api key access

   - Go to https://www.weatherapi.com/ and sign up for an account and create an api key to use in this demo

7. Update weather-api-key in request-transformer-advanced plugin in deck/deck.yaml file

   - Update the weather api key in `deck/deck.yaml` to match your apikey after you signed up on the weather api site in Step 8

8. Apply Kong Configuration with Deck

   ```bash
   # Navigate to deck configuration directory
   cd deck

   # Change line 7 of deck/deck.yaml to point to your control plane

   # Apply configuration to Kong Konnect
   deck gateway sync deck.yaml --konnect-addr https://<changeme>.api.konghq.com --konnect-token $YOUR_KONNECT_TOKEN --konnect-control-plane-name $CP_NAME

   ```

## Testing the Setup

1. Fetch the proxy token from terminal when starting mcp inspector like below:

   ```bash
   Starting MCP inspector...
   ⚙️ Proxy server listening on localhost:6277
   🔑 Session token: xxx
   Use this token to authenticate requests or set DANGEROUSLY_OMIT_AUTH=true to disable auth

   🚀 MCP Inspector is up and running at:
   http://localhost:6274/?MCP_PROXY_AUTH_TOKEN=xxx

   ```

2. Go to `localhost:6274` and choose:

   - Transport Type: `Streamable HTTP`
   - URL: `http://localhost:8000/weather`
   - Connection Type: `Via Proxy`

3. Paste the proxy token fron step 1 into the GUI of MCP inspector -> Configuration -> Proxy Session Token. Alternatively, just use the link with the `MCP_PROXY_AUTH_TOKEN` provided to load the token directly in config

4. Go to `Open Auth Settings`

5. Run the `guided oauth flow` or `quick oauth flow` and you should see a success token retrieved from keycloak - this proves the Oauth flow is working

6. Try to click connect on the left panel now - see it fails and check dev tools to see 401 error, or see kong logs. If the AI MCP Oauth plugin was configured correctly, it will direct you to Keycloak to login.

7. Login with these credentials loaded into the demo realm through the mcp/demo.json config:

   - User: john
   - Password: doe

8. If successful, you should see a `Connected` and be able to access the tools in the MCP server

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
