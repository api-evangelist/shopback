# ShopBack

ShopBack is a Singapore-headquartered shopping, rewards and payments platform founded in 2014, operating across 13 markets in Asia-Pacific, Europe and the United States. Alongside its consumer cashback app it runs a merchant payments business — ShopBack Pay and ShopBack PayLater — with a public developer hub at [docs.shopback.com](https://docs.shopback.com/).

- Website: https://www.shopback.com/
- Developer hub: https://docs.shopback.com/
- Status: https://status.shopback.com/
- GitHub: https://github.com/shopback

## APIs

| API | Version | Production base URL |
|---|---|---|
| [Online Payments API](https://docs.shopback.com/reference/initiateorder) (Online Bespoke) | 2.0 | `https://prod-merchant-service.hoolah.co/merchant` |
| [In-Store Payments API](https://docs.shopback.com/reference/in-store-getting-started) | 1.4 | `https://integrations.shopback.sg/posi` (SG), `https://integrations.shopback.com.hk/posi` (HK) |

Both OpenAPI 3.0.1 documents in `openapi/` were assembled verbatim from the
per-operation OpenAPI definitions ShopBack publishes on each API reference page
of its developer hub. Twenty operations total.

## Artifacts

`openapi/` `overlays/` `authentication/` `conventions/` `errors/` `lifecycle/`
`changelog/` `sandbox/` `conformance/` `data-model/` `components/` `packages/`
`asyncapi/` `skills/` `mcp/` `llms/` `well-known/` `security/`

Notable absences, recorded rather than fabricated: no `/.well-known/` documents
on any host, no A2A agent card, no OAuth 2.0 scopes, no published MCP server, no
first-party SDK in any package registry, no AsyncAPI, and no published
vulnerability-disclosure or trust-center page.
