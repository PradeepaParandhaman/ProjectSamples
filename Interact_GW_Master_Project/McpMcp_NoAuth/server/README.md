# McpMcp_NoAuth

## Overview

This sample demonstrates an MCP (Model Context Protocol) Server on a DataPower Nano Gateway that proxies another remote MCP server, with no client authentication. It is part of the `Interact_GW_Master_Project` master test project used for automated regression runs and stack seeding ([AM-4491](https://ibm-middleware.atlassian.net/browse/AM-4491)).

The MCP Server (`mcp-mcp-noauth-server`) forwards requests to the public DeepWiki MCP server (`https://mcp.deepwiki.com/mcp`) through an `MCPInvoke` policy, and exposes the DeepWiki tools to MCP clients through the gateway.

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

## Components

**server/**
- **Mcp_Mcp_NoAuth_Server.yml**: MCP Server configuration (protocol version `V2025_11_25`) that references the plan, policy sequence, and tools
- **Mcp_Mcp_NoAuth_Tools.yml**: Defines the MCP tools with descriptions and input schemas
- **Mcp_Mcp_NoAuth_Plan.yml**: Plan that applies the quota to the MCP Server
- **Mcp_Mcp_NoAuth_Quota.yml**: Quota limiting requests to 1000 per minute (HTTP 429 when exceeded)

**policies/**
- **Mcp_Mcp_NoAuth_Invoke.yml**: `MCPInvoke` policy that routes requests to the backend MCP server over HTTP/SSE (30s timeout, TLS client profile `sample-tls-client-profile:1.0.0`)
- **Mcp_Mcp_NoAuth_PolicySequence.yml**: Policy sequence that runs the invoke policy and attaches telemetry
- **Mcp_Mcp_NoAuth_Telemetry.yml**: Telemetry configuration for monitoring and metrics collection

## Use Case

This example demonstrates how to:
1. Front an existing remote MCP server with a DataPower Nano Gateway
2. Expose MCP tools without requiring client authentication (no `ExtractIdentity`, `Authenticate`, or `Authorize` policies)
3. Apply gateway policies (quota, telemetry) to MCP Server tools

For the authenticated equivalent, see `McpMcp_Subscription`.

## Prerequisites

The TLS client profile `sample-tls-client-profile:1.0.0` must exist in the target catalog before publishing.
