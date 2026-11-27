# RestMcp_NoAuth

## Overview

This sample demonstrates an MCP (Model Context Protocol) Server on a DataPower Nano Gateway that exposes a REST (OpenAPI) backend as MCP tools, with no client authentication. It is part of the `Interact_GW_Master_Project` master test project used for automated regression runs and stack seeding ([AM-4491](https://ibm-middleware.atlassian.net/browse/AM-4491)).

The MCP Server (`rest-mcp-noauth-server`) maps operations of the Swagger Petstore API (`https://petstore.swagger.io/v2`) to MCP tools. Tool calls are translated into HTTP requests by an `Invoke` policy.

## MCP Tools

The following tools are exposed by this MCP Server:

### 1. modify_pet
Updates an existing pet's details (name, category, photo URLs, tags, status). Maps to `PUT /pet`.

**Required Parameters:** `name`, `photoUrls`
**Optional Parameters:** `id`, `category`, `tags`, `status` (available, pending, sold)

### 2. register_pet
Adds a new pet to the store. Maps to `POST /pet`.

**Required Parameters:** `name`, `photoUrls`
**Optional Parameters:** `id`, `category`, `tags`, `status` (available, pending, sold)

### 3. fetch_pet_by_id
Retrieves the details of a single pet. Maps to `GET /pet/{petId}`.

**Required Parameters:** `petId`

### 4. revise_pet_with_form
Updates a pet using form data. Maps to `POST /pet/{petId}`.

**Required Parameters:** `petId`
**Optional Parameters:** `name`, `status`

### 5. terminate_pet
Deletes a pet record. Maps to `DELETE /pet/{petId}`.

**Required Parameters:** `petId`
**Optional Parameters:** `api_key` (header)

## Components

**server/**
- **pet-store-spec.yml**: OpenAPI 3.0.4 specification for the Swagger Petstore API (server `https://petstore.swagger.io/v2`)
- **Rest_Mcp_NoAuth_Server.yml**: MCP Server configuration (protocol version `V2025_11_25`) that references the plan, policy sequence, and tools
- **Rest_Mcp_NoAuth_Tools.yml**: Defines the MCP tools and binds each one to a Petstore operation (`x-api-binding`)
- **Rest_Mcp_NoAuth_Plan.yml**: Plan that applies the quota to the MCP Server
- **Rest_Mcp_NoAuth_Quota.yml**: Quota limiting requests to 1000 per minute (HTTP 429 when exceeded)

**policies/**
- **Rest_Mcp_NoAuth_Invoke.yml**: `Invoke` policy that forwards requests to `https://petstore.swagger.io/v2` (30s timeout, TLS client profile `sample-tls-client-profile:1.0.0`)
- **Rest_Mcp_NoAuth_PolicySequence.yml**: Policy sequence that runs the invoke policy and attaches telemetry
- **Rest_Mcp_NoAuth_Telemetry.yml**: Telemetry configuration for monitoring and metrics collection

## Use Case

This example demonstrates how to:
1. Expose an existing OpenAPI-based REST service as MCP tools
2. Provide descriptive tool documentation and input schemas for LLM consumption
3. Expose tools without requiring client authentication (no `ExtractIdentity`, `Authenticate`, or `Authorize` policies)
4. Apply gateway policies (quota, telemetry) to MCP Server tools

For the authenticated equivalent, see `RestMcp_Subscription`.

## Prerequisites

The TLS client profile `sample-tls-client-profile:1.0.0` must exist in the target catalog before publishing.
