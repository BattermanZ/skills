# DHL eCommerce Track & Trace API notes

Session-derived reference for checking DHL NL/eCommerce parcel statuses when maintaining Hatchdoor package-tracking notes.

## Endpoint

Use the public Track & Trace endpoint documented by DHL eCommerce:

```text
GET https://api-gw.dhlparcel.nl/track-trace?key=<tracking-code>
Accept: application/json
```

Example:

```bash
curl -sS -H 'Accept: application/json' \
  'https://api-gw.dhlparcel.nl/track-trace?key=JVGLOTC0116668120'
```

## Useful response fields

The endpoint returns a JSON array of shipment objects. Inspect:

- `barcode` — DHL internal barcode.
- `barcodes` — aliases, including the user-facing tracking code.
- `events[]` — event timeline; each event has `category`, `status`, `timestamp`, and often `localTimestamp`.
- `deliveredAt` — actual delivery timestamp when delivered.
- `returnedAt` — return completion timestamp when it is a return shipment.
- `type` — e.g. `RETOUR` for a return.
- `isReturn` — boolean; if true, describe the result as delivered/returned back to shipper, not simply delivered to recipient.

## Vault update pattern

When a tracked shipment is confirmed delivered/returned:

1. Update the shipment note with an `## Online status check` section before `## Notes`.
2. Record the check date, final category/status, delivered/returned timestamp, shipment type, and DHL barcode.
3. Add `status/archived` to the shipment note frontmatter if the tracking is no longer active.
4. Update the `[[Package tracking]]` hub:
   - remove it from `## Active shipments`;
   - add it under a delivered/returned section with a short result.
5. Confirm Hatchdoor git sync after the debounced write completes (`pending: 0`, `unpushed: 0`).

## Pitfalls

- The marketing Track & Trace web page may not render the actual status in static extraction; use the API endpoint for a grounded status check.
- If `isReturn: true` or `type: RETOUR`, avoid saying only “delivered”; say “delivered back to shipper” or “returned to shipper”.
- Receiver postcode can expose additional details, but basic status is often available from the tracking code alone.
