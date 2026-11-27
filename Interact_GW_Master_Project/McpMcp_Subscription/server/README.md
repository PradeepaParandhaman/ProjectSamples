# McpMcp_Subscription

## Overview

This sample demonstrates an MCP (Model Context Protocol) Server on a DataPower Nano Gateway that proxies another remote MCP server and requires clients to authenticate with a subscription (client ID and client secret). It is part of the `Interact_GW_Master_Project` master test project used for automated regression runs and stack seeding ([AM-4491](https://ibm-middleware.atlassian.net/browse/AM-4491)).

The MCP Server (`mcp-mcp-subscription-server`) forwards requests to the public DeepWiki MCP server (`https://mcp.deepwiki.com/mcp`) through an `MCPInvoke` policy, after the caller has been identified, authenticated, and authorized against a subscription.

## MCP Tools

The following tools are exposed by this MCP Server:

### 1. ask_wiki_question
Asks any question about a GitHub repository's codebase and returns an AI-powered answer grounded in its DeepWiki.

**Parameters:**
- `repoName`: GitHub repository, or a list of repositories (max 10), in `owner/repo` format.
- `question`: The question to ask about the repository.

### 2. read_wiki_contents
Views documentation about a GitHub repository.

**Parameters:**
- `repoName`: GitHub repository in `owner/repo` format (e.g. `facebook/react`).

### 3. read_wiki_structure
Gets a list of documentation topics for a GitHub repository.

**Parameters:**
- `repoName`: GitHub repository in `owner/repo` format (e.g. `facebook/react`).

## Authentication

Callers must supply their subscription credentials in HTTP headers:
- `X-IBM-Client-Id`
- `X-IBM-Client-Secret`

The policy sequence runs `ExtractIdentity`, then `Authenticate` and `Authorize` (both using the `subscription` operation), and only then invokes the backend MCP server.

## Components

**server/**
- **Mcp_Mcp_Subscription_Server.yml**: MCP Server configuration (protocol version `V2025_11_25`) that references the plan, policy sequence, and tools
- **Mcp_Mcp_Subscription_Tools.yml**: Defines the MCP tools with descriptions and input schemas
- **Mcp_Mcp_Subscription_Plan.yml**: Plan that applies the quota to the MCP Server
- **Mcp_Mcp_Subscription_Quota.yml**: Quota limiting requests to 1000 per minute (HTTP 429 when exceeded)

**policies/**
- **Mcp_Mcp_Subscription_ExtractIdentity.yml**: Extracts the client ID and secret from the request headers
- **Mcp_Mcp_Subscription_Authenticate.yml**: Authenticates the client using its subscription
- **Mcp_Mcp_Subscription_Authorize.yml**: Authorizes the client using its subscription
- **Mcp_Mcp_Subscription_Invoke.yml**: `MCPInvoke` policy that routes requests to the backend MCP server over HTTP/SSE (30s timeout, TLS client profile `sample-tls-client-profile:1.0.0`)
- **Mcp_Mcp_Subscription_PolicySequence.yml**: Policy sequence: ExtractIdentity → Authenticate → Authorize → Invoke, with telemetry monitoring
- **Mcp_Mcp_Subscription_Telemetry.yml**: Telemetry configuration for monitoring and metrics collection

## Use Case

This example demonstrates how to:
1. Front an existing remote MCP server with a DataPower Nano Gateway
2. Restrict access to MCP tools to subscribed applications
3. Apply gateway policies (identity, authentication, authorization, quota, telemetry) to MCP Server tools

For the unauthenticated equivalent, see `McpMcp_NoAuth`.

## Prerequisites

- The TLS client profile `sample-tls-client-profile:1.0.0` must exist in the target catalog before publishing.
- A consumer application subscribed to a product containing this MCP Server, to obtain a client ID and secret.
