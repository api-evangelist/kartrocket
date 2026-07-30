---
name: Ship an order with Shiprocket
description: Rate-shop couriers, create an order, ship it, schedule pickup, and track it using the official Shiprocket MCP server.
api: mcp/kartrocket-mcp.yml
generated: '2026-07-19'
method: generated
source: https://github.com/bfrs/shiprocket-mcp
operations:
  - shipping_rate_calculator
  - order_create
  - order_ship
  - order_pickup_schedule
  - generate_shipment_label
  - order_track
---

# Ship an order with Shiprocket

Operating instructions for an agent using the official Shiprocket MCP server
(`io.github.bfrs/shiprocket-mcp`). Every step maps to a real MCP tool — do not
invent tool names.

## Auth
The MCP server logs in to the seller's Shiprocket account using
`SELLER_EMAIL` + `SELLER_PASSWORD` (see `authentication/kartrocket-authentication.yml`).
No per-call token handling is needed by the agent; the server holds the session.

## Steps
1. **Rate-shop** — call `shipping_rate_calculator` with the pickup and delivery
   pincodes, package weight, and COD/prepaid flag to list serviceable couriers,
   their rates, and estimated delivery. Use `estimated_date_of_delivery` if the
   caller only needs an ETA.
2. **Confirm pickup location** — call `list_pickup_addresses` and pick the
   configured pickup address the order should ship from.
3. **Create the order** — call `order_create` with the buyer address, line items,
   payment method, dimensions, and weight. Reuse the seller's own order id as the
   source order id (Shiprocket deduplicates on it — there is no idempotency-key
   header; see `conventions/kartrocket-conventions.yml`).
4. **Ship it** — call `order_ship` to assign a courier (by Shiprocket's rules or a
   named courier from step 1) and generate the AWB.
5. **Pickup + label** — call `order_pickup_schedule` to book the pickup and
   `generate_shipment_label` to produce the shipping label.
6. **Track** — call `order_track` with the AWB, Shiprocket Order ID, or source
   order id to report current status.

## Errors
Shiprocket returns non-2xx HTTP statuses with a JSON `{ message }` body (not
RFC 9457). On a validation error, surface the `message` to the caller and fix the
offending field before retrying. Re-login on token expiry (~10 days) is handled by
the MCP server.
