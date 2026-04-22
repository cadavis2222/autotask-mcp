---
name: autotask_mcp
description: Use this skill when users need Autotask PSA operations through MCP, including tickets, companies, contacts, projects, time entries, billing, service calls, and notes.
metadata: {"openclaw":{"homepage":"https://github.com/wyre-technology/autotask-mcp","requires":{"bins":["node","npx"],"env":["AUTOTASK_USERNAME","AUTOTASK_SECRET","AUTOTASK_INTEGRATION_CODE"]}}}
---

# Autotask MCP Skill

This skill helps the agent reliably use the Autotask MCP server for MSP workflows.

## When to use

Use this skill when the user asks for any of the following:
- Search, create, or update companies, contacts, tickets, projects, tasks, or notes.
- Log or query time entries, ticket charges, expense reports, or billing items.
- Work with service calls, quote data, opportunities, products, services, contracts, invoices, or configuration items.
- Resolve Autotask field IDs and picklist/value mapping questions.

## Preconditions

Before calling Autotask tools:
1. Ensure credentials are available in environment variables:
- AUTOTASK_USERNAME
- AUTOTASK_SECRET
- AUTOTASK_INTEGRATION_CODE
2. If API zone autodetection fails, set AUTOTASK_API_URL explicitly.
3. Run connection check first with autotask_test_connection.

## Core execution pattern

1. Discover intent and scope:
- Ask clarifying questions only if required IDs are missing.
- For broad requests, start with search tools and then drill down.

2. Use category-first discovery when uncertain:
- Use autotask_list_categories.
- Use autotask_list_category_tools.
- Optionally use autotask_router for guided dispatch.

3. Safe operation sequence:
- Read/search first.
- Confirm target IDs.
- Perform create/update/delete action.
- Re-read entity to verify mutation.

4. Pagination and filters:
- Most search tools support page and pageSize.
- Start with default page size unless user asks for exhaustive results.
- Report if additional pages exist and ask whether to continue.

## Response style for users

- Summarize actions taken in plain language.
- Include key IDs returned by Autotask.
- Highlight partial failures and provide the next recovery step.
- For destructive operations, confirm intent before execution.

## High-value workflows

### Ticket triage
1. autotask_search_tickets
2. autotask_get_ticket_details
3. autotask_update_ticket
4. autotask_create_time_entry or autotask_create_ticket_note as needed

### Company and contact onboarding
1. autotask_create_company
2. autotask_create_contact
3. autotask_update_company_site_configuration (only after read)

### Project delivery
1. autotask_search_projects or autotask_create_project
2. autotask_list_phases or autotask_create_phase
3. autotask_search_tasks or autotask_create_task

### Billing review
1. autotask_search_billing_items
2. autotask_get_billing_item
3. autotask_search_billing_item_approval_levels

## Tool catalog

See TOOL_CATALOG.md in this skill directory for the complete tool inventory.
