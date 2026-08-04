# ShopBack

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
