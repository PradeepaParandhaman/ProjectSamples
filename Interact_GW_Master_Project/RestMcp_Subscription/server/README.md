# RestMcp_Subscription

## Overview

This sample demonstrates an MCP (Model Context Protocol) Server on a DataPower Nano Gateway that exposes a REST (OpenAPI) backend as MCP tools and requires clients to authenticate with a subscription (client ID and client secret). It is part of the `Interact_GW_Master_Project` master test project used for automated regression runs and stack seeding ([AM-4491](https://ibm-middleware.atlassian.net/browse/AM-4491)).

The MCP Server (`rest-mcp-subscription-server`) maps operations of the Swagger Petstore API (`https://petstore.swagger.io/v2`) to MCP tools. Tool calls are translated into HTTP requests by an `Invoke` policy, after the caller has been identified, authenticated, and authorized against a subscription.

## MCP Tools

The following tools are exposed by this MCP Server:

### 1. update_petstore_pet
Updates an existing pet's information. Maps to `PUT /pet`.

**Required Parameters:** `name`, `photoUrls`
**Optional Parameters:** `id`, `category`, `tags`, `status` (available, pending, sold)

### 2. create_petstore_pet
Adds a new pet record to the store. Maps to `POST /pet`.

**Required Parameters:** `name`, `photoUrls`
**Optional Parameters:** `id`, `category`, `tags`, `status` (available, pending, sold)

### 3. retrieve_petstore_pet_by_id
Retrieves a pet's full details. Maps to `GET /pet/{petId}`.

**Required Parameters:** `petId`

### 4. update_petstore_pet_from_form
Updates a pet using form data. Maps to `POST /pet/{petId}`.

**Required Parameters:** `petId`
**Optional Parameters:** `name`, `status`

### 5. delete_petstore_pet
Deletes a pet record. Maps to `DELETE /pet/{petId}`.

**Required Parameters:** `petId`
**Optional Parameters:** `api_key` (header)

## Authentication

Callers must supply their subscription credentials in HTTP headers:
- `X-IBM-Client-Id`
- `X-IBM-Client-Secret`

The policy sequence runs `ExtractIdentity`, then `Authenticate` and `Authorize` (both using the `subscription` operation), and only then invokes the backend.

## Components

**server/**
- **pet-store-spec.yml**: OpenAPI 3.0.4 specification for the Swagger Petstore API (server `https://petstore.swagger.io/v2`)
- **Rest_Mcp_Subscription_Server.yml**: MCP Server configuration (protocol version `V2025_11_25`) that references the plan, policy sequence, and tools
- **Rest_Mcp_Subscription_Tools.yml**: Defines the MCP tools and binds each one to a Petstore operation (`x-api-binding`)
- **Rest_Mcp_Subscription_Plan.yml**: Plan that applies the quota to the MCP Server
- **Rest_Mcp_Subscription_Quota.yml**: Quota limiting requests to 1000 per minute (HTTP 429 when exceeded)

**policies/**
- **Rest_Mcp_Subscription_ExtractIdentity.yml**: Extracts the client ID and secret from the request headers
- **Rest_Mcp_Subscription_Authenticate.yml**: Authenticates the client using its subscription
- **Rest_Mcp_Subscription_Authorize.yml**: Authorizes the client using its subscription
- **Rest_Mcp_Subscription_Invoke.yml**: `Invoke` policy that forwards requests to `https://petstore.swagger.io/v2` (30s timeout, TLS client profile `sample-tls-client-profile:1.0.0`)
- **Rest_Mcp_Subscription_PolicySequence.yml**: Policy sequence: ExtractIdentity → Authenticate → Authorize → Invoke, with telemetry monitoring
- **Rest_Mcp_Subscription_Telemetry.yml**: Telemetry configuration for monitoring and metrics collection

## Use Case

This example demonstrates how to:
1. Expose an existing OpenAPI-based REST service as MCP tools
2. Restrict access to MCP tools to subscribed applications
3. Apply gateway policies (identity, authentication, authorization, quota, telemetry) to MCP Server tools

For the unauthenticated equivalent, see `RestMcp_NoAuth`.

## Prerequisites

- The TLS client profile `sample-tls-client-profile:1.0.0` must exist in the target catalog before publishing.
- A consumer application subscribed to a product containing this MCP Server, to obtain a client ID and secret.
