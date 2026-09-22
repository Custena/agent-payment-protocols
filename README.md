# The State of Agent Payment Protocols (April 2026)

_A neutral technical analysis of the protocols, rails, and standards enabling autonomous agent payments - written for developers operating APIs and MCP servers._

## Disclosure

This report is written and maintained by [Genesis Software Group](https://genesissoftware.io) in Copenhagen, the team behind [Custena](https://custena.com), a multi-protocol payment and compliance layer for agent commerce. We are participants in this market, not a neutral third party. The first five sections are written to be useful regardless of which payment infrastructure you choose; Section 6 names Custena once. Every load-bearing factual claim cites a primary source and every cited URL has been resolved. Corrections welcome via PR (see [CONTRIBUTING.md](CONTRIBUTING.md)).

## TL;DR

- In the six months between October 2025 and April 2026, more than ten payment protocols launched to let AI agents spend money on their own. They were shipped by Coinbase, Stripe, Lightning Labs, Google, Visa, Mastercard, American Express, OpenAI, Skyfire, and Alibaba. None of them interoperate.
- "Which protocol wins" is the wrong question. The same companies are simultaneously building, backing, or extending multiple protocols at once. Stripe co-authored MPP and joined the x402 Foundation. Visa runs TAP, published an MPP card SDK, and operates Intelligent Commerce Connect as a four-protocol bridge. The protocols are converging into a layered stack, not competing for one slot.
- For an API or MCP server operator, the practical posture is **multi-protocol**: accept whichever flow an arriving agent presents, with the dispatch handled by a middleware layer beneath your service. Picking a single protocol in 2026 is the same bet as picking a single credit-card network in 1996.
- For organizations running spending agents inside the EU, this stops being a purely technical question after **August 2, 2026**. The EU AI Act's Article 26 imposes deployer obligations - human oversight, audit logs, continuous monitoring - with penalties up to 3% of global turnover. The protocol is half the answer; the buyer-side compliance layer is the other half.

## Table of Contents

1. **Introduction** - The cambrian explosion of agent payment protocols
2. **The Protocol Landscape**
   - 2a. HTTP 402 protocols (x402, MPP, L402)
   - 2b. Traditional-rail protocols (Visa TAP + ICC, Mastercard Agent Pay, Amex ACE, Google AP2)
   - 2c. Commerce protocols (Google/Shopify UCP, OpenAI ACP)
   - 2d. Regional protocols (Alibaba/Ant)
3. **The Convergence Story**
4. **Decision Framework** - What should you support?
5. **The Compliance Overlay** - EU AI Act Article 26
6. **How Custena Fits**

---

## 1. Introduction - The Cambrian Explosion of Agent Payment Protocols

In the six months between October 2025 and April 2026, more than ten payment protocols launched to let AI agents spend money on their own. They were shipped by Coinbase, Stripe, Lightning Labs, Google, Visa, Mastercard, American Express, OpenAI, Skyfire, and Alibaba. Each speaks a different language. None of them interoperate.

The reason eight of the world's largest payments and software companies all moved in parallel is the same reason SaaS revenue charts are starting to bend: **the unit of consumption is changing from seats to calls.** Per-seat billing assumed a human at the keyboard. Once an agent does the work, one seat can drive a thousand API calls - or none. Vendors that don't reprice in usage terms watch their margins collapse. Gartner expects most enterprises to abandon assistive-AI pricing for outcome-based workflows by 2028, and separately predicts that by 2030 software vendors who bolt AI onto legacy stacks rather than redesigning for agentic execution will face margin compression of up to 80%.[^1] Usage-based billing, in turn, only works if payment can settle inline and programmatically - not through a checkout screen on every API call. That is the problem these twelve protocols are racing to solve. Whether every call _should_ settle automatically - or which calls warrant a hard cap, a policy check, or a human in the loop - is a different problem, and one the EU AI Act forces European deployers to answer. We come back to it in Section 5.

This is the first period in history where software agents can, without human input, discover a service, negotiate a price, and settle a payment. The infrastructure to make that happen did not exist in October 2025. By April 2026 it has been built, launched, and deployed in production - in parallel, with overlapping but incompatible designs. The x402 Foundation now sits under the Linux Foundation with Visa, Mastercard, Stripe, Google, AWS, Shopify, and Cloudflare as founding members.[^x2] Stripe and Tempo took MPP to mainnet on March 18, 2026 with 100+ services live at launch.[^m1] Mastercard's Agent Pay went live across 10+ markets,[^mc1] including Europe's first live agent payment via Santander.[^mc2] On the consumer side, Alibaba's Alipay processed 120 million AI-initiated transactions in a single week in February 2026,[^al1] a volume that dwarfs every Western pilot combined.

For anyone operating an API or MCP server today, this creates a concrete problem: **the agents calling your endpoint might speak x402, MPP, L402, a card-network identity protocol, or a platform-specific fiat flow.** Accept only one and you miss the rest. Try to support all of them and you're integrating against a moving target. Ignore the question entirely and your service becomes invisible to agent-native traffic - the fastest-growing segment in the industry, and the one Forrester has framed as "a turning point for micropayments" because, as the post puts it, "unlike human consumers, agents can't substitute free alternatives when a required service is unavailable."[^6]

This report is not a "which protocol will win" analysis. The framing turns out to be wrong. There is no "superior" credit-card network that owns 100% of the market; no merchant in 2026 would accept Mastercard but refuse Visa. The same logic applies here. What is actually happening - visible in the founding members of the x402 Foundation, the extension agreements between Stripe, Visa, and Lightspark, and the cross-protocol bridges already shipped by Google, Coinbase, and PayPal - is **convergence, not competition.** The protocols are arranging themselves into a layered stack. Commerce flows sit above identity flows, which sit above payment flows, which sit above settlement rails. A single agent transaction may touch three or four of these layers, routed through as many organizations.

The report has three goals:

1. **Map the landscape** - what each protocol does, who backs it, what state it is in, and what tradeoffs it imposes on integrators.
2. **Make the convergence story legible** - the evidence that protocols are being designed to interoperate, and the emerging layered architecture this produces.
3. **Provide a decision framework** - for an operator deciding what to support today, a clear sequence of questions that leads to a defensible answer, including when a multi-protocol approach beats picking a winner.

At the end of the report, we briefly discuss how Custena (built by Genesis Software Group) fits into this landscape. Everything before that section is written to be useful regardless of which payment infrastructure you choose.

**A note on regulation.** The EU AI Act enters force on August 2, 2026, with deployer obligations under Article 26 that apply to any organization using AI agents to make autonomous decisions - including spending decisions. Deployer fines reach 3% of global turnover.[^7] This is four months away as of publication, and it will reshape protocol selection in Europe from a technical choice into a compliance choice. We address this in Section 5.

[^1]: Gartner, ["Gartner Expects Most Enterprises to Abandon Assistive AI for Outcome-Focused Workflow by 2028"](https://www.gartner.com/en/newsroom/press-releases/2026-04-02-gartner-expects-most-enterprises-to-abandon-assistive-ai-for-outcome-focused-workflow-by-2028), April 2, 2026.

[^6]: Forrester, ["Why Stripe's Machine Payments Protocol Signals a Turning Point for Micropayments"](https://www.forrester.com/blogs/why-stripes-machine-payments-protocol-signals-a-turning-point-for-micropayments/), April 2026.

[^7]: Regulation (EU) 2024/1689 (EU AI Act), Article 26 (obligations of deployers) and Article 99 (penalties).

---

## 2. The Protocol Landscape

The protocols launched between October 2025 and April 2026 are not direct competitors. They occupy different positions in an emerging stack, and most production agent transactions already touch two or three layers in sequence. To make the landscape navigable, we group them by what they actually do:

- **HTTP 402 protocols** (x402, MPP, L402) embed payment directly in HTTP, reusing the long-dormant `402 Payment Required` status code as the challenge mechanism. They are the layer agents speak to APIs in. Settlement rails vary: x402 is stablecoin-only (USDC on Base, Solana, Stellar), L402 is Bitcoin Lightning, and MPP is rail-agnostic by design - the same envelope carries Tempo USDC, Stripe fiat, Lightning, Solana, and Stellar interchangeably.
- **Traditional-rail protocols** (Visa TAP, Mastercard Agent Pay, Amex ACE, Google AP2) add agent-identity and authorization credentials on top of the existing card networks. They answer "is this a legitimate agent, and what is it allowed to spend?" - they do not, by themselves, settle anything new.
- **Commerce protocols** (Google/Shopify UCP, OpenAI ACP) sit one layer above payments entirely, defining how agents discover catalogs, build carts, and negotiate purchases. Payment is a dependency, not the subject.
- **Regional protocols** (Alibaba/Ant Agentic Commerce Trust Protocol) are vertically integrated into a single ecosystem and operate at volumes the rest of the field has not reached.

The same agent transaction may use a commerce protocol to negotiate a purchase, an identity protocol to prove the agent's authority, a payment protocol to settle the charge, and a card or blockchain rail underneath to move the funds. Section 3 traces this stacking explicitly. The profiles below sketch each protocol's institutional position, distinguishing properties, and tradeoffs; the cited specs and repos carry the bytes-on-the-wire detail.

### 2a. HTTP 402 protocols

#### x402 - Coinbase, Cloudflare, and the x402 Foundation

x402 has the broadest institutional backing of any agent payment protocol today. The Linux Foundation announced the **x402 Foundation** on April 2, 2026 with founding members and ecosystem partners drawn from card networks, PSPs, hyperscalers, crypto infrastructure, and merchants - Visa, Mastercard, American Express, Stripe, Adyen, Fiserv, PPRO, Google, AWS, Microsoft, Cloudflare, Coinbase, Base, Circle, Solana Foundation, Polygon Labs, KakaoPay, Shopify, Sierra, Merit Systems, thirdweb, and Ampersend.ai.[^x2] Notably absent: Ant International, PayPal, and Skyfire, each of whom is pursuing their own protocol or settlement layer.

Settlement is stablecoin-only - USDC on Base by default, with Solana, Stellar, and other CAIP-registered chains added in V2 - delegated to a facilitator network that handles on-chain verification and broadcast.[^x4] **V2 shipped December 11, 2025** and added the framework for reusable wallet sessions, dynamic pricing, and a plug-in-driven multi-chain SDK.[^x6] The reference implementation is published under `coinbase/x402` with an active contributor base.[^x3]

Adoption signals are mixed. InfoQ reported 100M+ lifetime payments through Q1 2026,[^x7] but Coindesk's March 2026 review found weekly transaction counts declining and average values hovering near $0.20, with a substantial fraction of observed transactions flagged as test or synthetic traffic.[^x8] The honest read: x402's installed base of integrations is large and growing, but live agent-driven volume is harder to characterize than the press releases suggest.

#### MPP - Stripe and Tempo Labs

The Machine Payments Protocol went mainnet on **March 18, 2026** with 100+ live directory entries (Alchemy, Dune Analytics, Merit Systems, and Parallel Web Systems among them) and a broader ecosystem of working partners - Anthropic, OpenAI, Shopify, DoorDash, Mastercard, Nubank, Ramp, Revolut, Standard Chartered, Visa.[^m1] It is the youngest of the three HTTP 402 protocols and the only one to natively carry fiat alongside crypto: the same MPP envelope dispatches to Tempo USDC, Stripe `PaymentIntent` tokens, Lightning, Solana, and Stellar interchangeably.[^m2]

Two convergence signals matter. **Visa published a Card Specification SDK for MPP** in April 2026, letting Visa-issued cards satisfy MPP charge intents directly.[^m5] And **Lightspark extended MPP for Lightning settlement**, putting Bitcoin Lightning underneath the same envelope as Tempo USDC. The card networks and the Bitcoin community have, in effect, agreed to speak each other's language through MPP's method dispatch.

#### L402 - Lightning Labs and the original HTTP 402 protocol

L402 (originally LSAT) is the elder of the three, dating to a Lightning Labs post in March 2020.[^l1] It has been in production for six years, primarily inside Lightning Labs' own products. The 2026 wave revived it: Lightning Labs released the **Lightning Agent Tools** repo on **February 11, 2026** - a bundle of seven composable skills (`lnd`, `lightning-security-module`, `macaroon-bakery`, `lnget`, `aperture`, `lightning-mcp-server`, `commerce`)[^l2]

L402's distinguishing property is reusable credentials. Where x402 and MPP authorize a single payment per request (sessions excepted), L402 issues a macaroon - a bearer token tied to a Lightning payment hash with an attached caveat list - that can be presented on subsequent requests until its caveats expire.[^l4] In practice this means "pay one Lightning invoice for an hour of access" - a usage pattern x402 and MPP achieve only via newer session primitives. L402's principal cost is operational: it requires a Lightning node or a Lightning-as-a-service provider on both sides of the connection.

#### Comparison: x402 vs MPP vs L402

| Aspect                    | x402                               | MPP                        | L402                       |
| ------------------------- | ---------------------------------- | -------------------------- | -------------------------- |
| **Default settlement**    | USDC on Base (~2s via facilitator) | USDC on Tempo L1 (~500ms)  | Bitcoin Lightning (~1–3s)  |
| **Min practical payment** | ~$0.001                            | ~$0.0001                   | ~$0.00001                  |
| **Reusable credentials**  | V2 wallet sessions                 | Native sessions primitive  | Macaroons with caveats     |
| **Fiat support**          | No (V2 plugins planned)            | Yes (Stripe method)        | No                         |
| **Standards body**        | x402 Foundation (Linux Foundation) | IETF individual submission | bLIP / Lightning community |
| **Live since**            | Oct 2025 (V2: Dec 2025)            | March 18, 2026             | March 2020                 |

The three protocols above all add a per-transaction fee layer: x402 over USDC pays stablecoin transfer gas plus any facilitator fee, L402 pays Lightning routing fees, and MPP's on-chain methods pay chain gas. None of them names a **feeless settlement rail**. Nano (XNO) is that counter-example: a peer-to-peer Layer-1 ledger where a transfer confirms in under a second and settles with no protocol fee and no facilitator, bridge, or issuer that can freeze funds.[^n1] It is not an HTTP-402 protocol - it competes at the settlement layer beneath the 402 envelope - but for the multi-protocol dispatch posture this report recommends, it is an available settlement method an operator can list beside USDC and Lightning, and a zero fee floor matters most at the sub-cent values these protocols are built for.

### 2b. Traditional-rail protocols (cards + identity)

Where the HTTP 402 protocols replace the payment rail, these protocols leave the rail intact and add an **identity and authorization** layer for agents on top of it. The card networks already process tens of trillions of dollars annually; what they have not historically processed is non-human callers with cryptographically scoped spending authority. These four entries address that gap.

#### Visa Trusted Agent Protocol (TAP) and Intelligent Commerce Connect

TAP is a developer-preview specification using RFC 9421 HTTP Message Signatures to verify agent identity at the network edge.[^v1] It carries no payment data - it is the identity layer that sits **upstream** of whatever payment protocol the merchant accepts. Visa has been measured in its claims, naming approximately a dozen ecosystem partners (Adyen, Ant International, Checkout.com, Coinbase, CyberSource, Elavon, Fiserv, Microsoft, Nuvei, Shopify, Stripe, Worldpay) and describing TAP as in developer preview rather than at scale.[^v2]

The more consequential Visa announcement was **Visa Intelligent Commerce Connect (ICC)**, launched **April 8, 2026**.[^v3] Visa positions ICC as agnostic to underlying networks, protocols, and token vaults - a single Visa-operated endpoint that accepts and translates between the emerging agent payment standards (in practice today, TAP, MPP, ACP, and UCP) through one integration to the Visa Acceptance Platform. Pilot partners announced to date span hyperscalers, fintech infrastructure, and regional commerce groups (AWS and Aldar confirmed in Visa's public communications, with additional partners named across Visa's April 2026 announcements and partner briefings). ICC is the clearest institutional admission to date that no single agent payment protocol will dominate, and that the practical question for merchants is which abstraction handles the multiplexing for them.

#### Mastercard Agent Pay + Verifiable Intent

Mastercard has been the most aggressive on geographic rollout. Live agent-initiated transactions have been completed across nine APAC markets - Australia, New Zealand, Singapore, Malaysia, Thailand, India, South Korea, Hong Kong, and Taiwan - plus a Latin America wave spanning more than a dozen issuers and processors including BAC, Banco Galicia, Banco Itaú, Bancolombia, Banamex, Davivienda, and Santander.[^mc1] In March 2026, **Mastercard and Santander completed Europe's first live end-to-end agent-initiated payment** in a controlled environment.[^mc2] (Some outlets framed this as a commercial European rollout; the Mastercard press release itself describes a controlled environment, not unrestricted commerce.)

The architecture is a layered SD-JWT credential set (L1 issuer-bound identity, L2 user-signed intent with optional autonomous-mandate constraints, L3 short-lived agent-signed fulfillment splitting into network-facing and merchant-facing JWTs).[^mc3] Selective disclosure means the merchant sees the cart but not the payment credential; the network sees the authorization but not the cart contents - a privacy property the HTTP 402 protocols do not have. Mastercard's open-sourced **Verifiable Intent** spec makes the credential format reusable outside Mastercard's own rails. Agent identity verification is delegated to **Cloudflare's Web Bot Auth**, also built on RFC 9421 - the same primitive Visa TAP uses, and a quiet but important convergence on a single identity standard.[^mc4]

#### American Express ACE (Agentic Commerce Experiences)

Amex shipped its **ACE Developer Kit on April 14, 2026**, six days before this report's publication.[^a1] ACE pairs a card-rail integration with **Amex Agent Purchase Protection** - Amex's own framing claims industry first: existing cardmember protections extended to errors made by registered AI agents.

#### Google AP2 (Agent Payments Protocol)

AP2 takes a different approach to the same problem. Rather than HTTP signatures, it expresses every transaction as a chain of **Verifiable Digital Credentials** (VDCs) called **Mandates**. The user's agent issues an **Intent Mandate** ("find me white running shoes under $120"). The merchant returns a **Cart Mandate** that cryptographically binds the user identity to specific items and prices and is signed by the user when they approve. A separate **Payment Mandate** is shared with the payment network and issuer to flag agent involvement and whether the human is present at transaction time.[^ap1]

The current spec is V0.1, with 60+ collaborating organizations. A selected subset of partners spans card networks (Mastercard, Amex, JCB, UnionPay), PSPs (Adyen, Worldpay, PayPal), neo-banks (Revolut), crypto (Coinbase, Mysten Labs, Ant International), enterprise SaaS (Salesforce, ServiceNow, Intuit), and risk infrastructure (Forter).[^ap2] **Revolut announced AP2 support for Revolut Pay across the UK and EEA on January 19, 2026**, calling itself "one of the first EU payment methods compatible with Google's Agent Payments Protocol" - among the earliest live AP2 integrations from a European-licensed payment institution.[^ap3] AP2 is authorization-only; settlement happens on whatever rail the Payment Mandate references (cards, account-to-account, or digital assets).

### 2c. Commerce protocols (above payments)

These protocols are not payment specifications. They define how agents discover what is for sale, build carts, negotiate prices, and complete checkout flows. Payments are a sub-component, usually delegated to AP2 or to a card-network rail. We include them because the choice of commerce protocol determines which payment protocols will reach an operator's API in the first place.

#### Google + Shopify Universal Commerce Protocol (UCP)

UCP is an open standard for the full agentic commerce lifecycle - discovery → catalog → cart → checkout → post-purchase - built around a checkout state machine with statuses such as `incomplete`, `requires_escalation`, and `ready_for_complete`, plus a JSON-RPC 2.0 channel called the **Embedded Checkout Protocol (ECP)** for bi-directional merchant/agent messaging during checkout.[^ucp1] The merchant chooses the protocol version and capabilities, which keeps backwards compatibility tractable as the spec evolves. UCP is designed to interoperate with AP2 (payments), A2A (agent-to-agent communication), and MCP (tool use).

In production, UCP currently powers the checkout experience inside **Google AI Mode in Search** and inside the Gemini app's commerce surfaces.[^ucp2] For an API or MCP server operator, UCP is mostly orthogonal - it is built for retail commerce, not for tool-call monetization - but its conventions for session state and protocol negotiation are worth tracking, because they are likely to influence how MCP servers eventually expose paid capabilities.

#### Stripe + OpenAI Agentic Commerce Protocol (ACP)

ACP is the one production implementation in this group where the headline merchant deployment did not stick. The protocol itself, co-developed by OpenAI and Stripe, is alive and Apache 2.0 licensed. It currently ships an **Agentic Checkout API**, a **Delegate Payment** scheme (single-use PSP-issued tokens so the agent never holds raw card credentials), and a **Product Feed** format, with cart, orders, authentication, and MCP modules added in the April 17, 2026 release.[^acp1] The protocol is sound. What failed was OpenAI's flagship implementation: **ChatGPT Instant Checkout, launched in September 2025 with Etsy and over a million Shopify merchants, was wound down in early March 2026** after roughly five months of operation.[^acp2] Per industry coverage, only about a dozen of Shopify's millions of eligible merchants ever went live; users researched products in ChatGPT but completed purchases elsewhere; and missing capabilities - notably real-time inventory sync across millions of merchant catalogs and tax-collection infrastructure for merchant-of-record flows - compounded the problem. OpenAI explicitly pivoted to a product-discovery and referral-commission model rather than in-chat purchasing.[^acp3]

The pattern that failed was **consumer in-chat checkout** - the unit economics did not favor it and the surrounding compliance and fulfillment plumbing was not yet built. The Mastercard and Visa rollouts, the MPP-paid API integrations, and Alipay's volume (Section 2d) all argue that agent commerce itself is working. B2B agent-to-API monetization is structurally different from in-chat retail checkout, and is where the live developer volume currently sits.

### 2d. Regional protocols (Asia)

#### Alibaba/Ant Agentic Commerce Trust Protocol

Western coverage of agent payments tends to under-weight what is happening inside the Alipay ecosystem. In a single week in February 2026, **Alipay processed 120 million AI-initiated payments** through its Agentic Commerce Trust Protocol, primarily food and beverage purchases originating in the Qwen app and Taobao Instant Commerce.[^al1] That is roughly an order of magnitude greater than the cumulative live-transaction count claimed by all Western card-network and HTTP 402 deployments combined - in seven days.

The protocol itself is tightly integrated with Alipay's existing risk infrastructure and is not currently positioned as a cross-vendor open standard. For a Western API or MCP server operator, Alipay's volume is structurally inaccessible without a China-domestic entity. The reason it appears in this report is signalling: the demand-side question - whether agents will actually pay for things at scale - is empirically settled. The remaining questions are about which infrastructure handles the non-Chinese 99% of the global market.

### Sources and caveats

Protocol landscapes in this category move on a weekly cadence. Every specific date, partner count, transaction metric, and attribution in this section is sourced to the footnoted primary document. Where primary sources conflict (for example, MPP's daily finality numbers, or ICC pilot partner lists that expanded between Visa's April 8 launch and subsequent partner briefings), we have chosen the most conservative number or broader framing. Corrections and updates accepted via PR to `custena/agent-payment-protocols`.

[^x2]: Linux Foundation, ["Linux Foundation Launches the x402 Foundation"](https://www.linuxfoundation.org/press/linux-foundation-is-launching-the-x402-foundation-and-welcoming-the-contribution-of-the-x402-protocol), April 2, 2026.

[^x3]: x402 Specification v2, [`coinbase/x402` `specs/x402-specification-v2.md`](https://github.com/coinbase/x402).

[^x4]: x402.org, [Ecosystem - Facilitators](https://www.x402.org/ecosystem?filter=facilitators).

[^x6]: x402.org, ["x402 V2 Launch"](https://www.x402.org/writing/x402-v2-launch), December 11, 2025.

[^x7]: InfoQ Q1 2026 reporting on x402 lifetime payments. Note: the "active wallet" figures circulating in early 2026 (e.g., ~500k) do not appear in primary Coinbase or x402 Foundation disclosures and should be treated as unverified.

[^x8]: Coindesk, ["Coinbase-Backed AI Payments Protocol Wants to Fix Micropayments But Demand Is Just Not There Yet"](https://www.coindesk.com/markets/2026/03/11/coinbase-backed-ai-payments-protocol-wants-to-fix-micropayment-but-demand-is-just-not-there-yet), March 11, 2026; on-chain dashboards: [Artemis x402 dashboard](https://classic.artemis.ai/asset/x402).

[^m1]: Tempo Labs, ["Tempo Mainnet"](https://tempo.xyz/blog/mainnet/), March 18, 2026; The Block, ["Tempo Mainnet Goes Live with Machine Payments Protocol for Agents"](https://www.theblock.co/post/394131/tempo-mainnet-goes-live-with-machine-payments-protocol-for-agents).

[^m2]: MPP method specifications, [`tempoxyz/mpp-specs` - `specs/methods/`](https://github.com/tempoxyz/mpp-specs).

[^m4]: IETF Datatracker, [`draft-ryan-httpauth-payment`](https://datatracker.ietf.org/doc/draft-ryan-httpauth-payment/).

[^m5]: Visa, ["Visa Card Specification SDK for Machine Payments Protocol"](https://corporate.visa.com/en/sites/visa-perspectives/innovation/visa-card-specification-sdk-for-machine-payments-protocol.html), April 2026.

[^l1]: Lightning Labs, ["LSAT: Lightning Service Authentication Tokens"](https://lightning.engineering/posts/2020-03-30-lsat/), March 30, 2020.

[^l2]: Lightning Labs, ["Introducing the Lightning Agent Tools"](https://lightning.engineering/posts/2026-02-11-ln-agent-tools/), February 11, 2026.

[^l4]: L402 protocol specification, [`lightninglabs/L402`](https://github.com/lightninglabs/L402/blob/master/protocol-specification.md).

[^n1]: Nano (XNO) is an open, peer-to-peer digital currency; protocol and properties (feeless, near-instant finality, self-custodied, no single issuer) are documented at [nano.org](https://nano.org) and the protocol docs at [docs.nano.org](https://docs.nano.org). Primary source; verified 2026-09-22.

[^v1]: Visa Developer, ["Trusted Agent Protocol overview"](https://developer.visa.com/capabilities/trusted-agent-protocol).

[^v2]: Visa Investor Relations, ["Visa Introduces Trusted Agent Protocol - An Ecosystem-Led Framework for AI Commerce"](https://investor.visa.com/news/news-details/2025/Visa-Introduces-Trusted-Agent-Protocol-An-Ecosystem-Led-Framework-for-AI-Commerce/).

[^v3]: Visa Investor Relations, ["Visa Opens the Door to AI-Driven Shopping for Businesses Worldwide"](https://investor.visa.com/news/news-details/2026/Visa-Opens-the-Door-to-AI-Driven-Shopping-for-Businesses-Worldwide/), April 8, 2026.

[^mc1]: Mastercard, ["Mastercard Advances Agentic Payments in Latin America and the Caribbean with Live Transactions Completed Across the Region"](https://www.mastercard.com/news/latin-america/en/newsroom/press-releases/pr-en/2026/march/mastercard-advances-agentic-payments-in-latin-america-and-the-caribbean-with-live-transactions-completed-across-the-region/), March 2026.

[^mc2]: Mastercard, ["Santander and Mastercard Complete Europe's First Live End-to-End Payment Executed by an AI Agent"](https://www.mastercard.com/news/europe/en/newsroom/press-releases/en/2026/santander-and-mastercard-complete-europe-s-first-live-end-to-end-payment-executed-by-an-ai-agent/), April 2026.

[^mc3]: FintechWrapUp, ["Deep Dive: Mastercard Verifiable Intent"](https://www.fintechwrapup.com/p/deep-dive-mastercard-verifiable-intent).

[^mc4]: Cloudflare, ["Securing Agentic Commerce"](https://blog.cloudflare.com/secure-agentic-commerce/).

[^a1]: American Express, ["American Express Debuts Agentic Commerce Experiences (ACE) Developer Kit"](https://www.americanexpress.com/en-us/newsroom/articles/innovation/american-express-debuts-agentic-commerce-experiences--ace--devel.html), April 14, 2026; Fortune, ["American Express Debuts AI Payments Tools, Plus Industry-First Agent Purchase Protection"](https://fortune.com/2026/04/14/american-express-ai-payments-developers-purchase-protection/), April 14, 2026.

[^ap1]: AP2 Protocol, [Specification](https://ap2-protocol.org/specification/).

[^ap2]: Google Cloud, ["Announcing the Agents-to-Payments (AP2) Protocol"](https://cloud.google.com/blog/products/ai-machine-learning/announcing-agents-to-payments-ap2-protocol).

[^ap3]: Revolut, ["Revolut to Enable Frictionless Checkout Across All Agentic Commerce Platforms for the UK and EEA"](https://www.revolut.com/news/revolut_to_enable_frictionless_checkout_across_all_agentic_commerce_platforms_for_the_uk_and_eea/), January 19, 2026.

[^ucp1]: Shopify Engineering, ["Universal Commerce Protocol"](https://shopify.engineering/ucp); Google for Developers, [Merchant UCP](https://developers.google.com/merchant/ucp).

[^ucp2]: Google Blog, ["Agentic commerce: AI tools and protocols for retailers and platforms"](https://blog.google/products/ads-commerce/agentic-commerce-ai-tools-protocol-retailers-platforms/).

[^acp1]: [agenticcommerce.dev](https://www.agenticcommerce.dev/) (canonical spec); [`agentic-commerce-protocol/agentic-commerce-protocol`](https://github.com/agentic-commerce-protocol/agentic-commerce-protocol) (Apache 2.0).

[^acp2]: Digital Commerce 360, ["OpenAI shifts checkout plans in its agentic commerce strategy"](https://www.digitalcommerce360.com/2026/03/06/openai-shifts-checkout-plans-agentic-commerce-strategy/), March 6, 2026.

[^acp3]: The Keyword, ["OpenAI Ends Instant Checkout in ChatGPT"](https://www.thekeyword.co/news/openai-chatgpt-instant-checkout-scrapped).

[^al1]: BusinessWire / Alipay, ["Alipay AI Payment Exceeds 120 Million Transactions in One Week as Agentic Commerce Accelerates in China"](https://www.businesswire.com/news/home/20260213770962/en/Alipay-AI-Payment-Exceeds-120-Million-Transactions-in-One-Week-as-Agentic-Commerce-Accelerates-in-China), February 13, 2026.

---

## 3. The Convergence Story

The single most useful thing to know about this landscape is that the protocols are not competing for the same slot. They are converging into different layers of the same emerging stack - and the same handful of companies are simultaneously building, backing, or extending several of them at once.

The "which protocol will win" framing collapses on inspection. There is no "superior" credit-card network that owns 100% of the market. No merchant in 2026 would accept Mastercard but refuse Visa. The same logic applies here: the companies designing these protocols aren't picking one, and the evidence sits in their own partner lists.

### 3a. The companies aren't picking sides

Notice how the same names appear across protocol rosters that the press treats as competitors:

- **Stripe** co-authored MPP's IETF draft,[^m4] is a founding member of the x402 Foundation,[^x2] co-developed OpenAI's ACP,[^acp1] and is a UCP partner.[^ucp2]
- **Visa** has its own protocol (TAP),[^v1] is a founding member of the x402 Foundation,[^x2] published a Card Specification SDK that lets Visa-issued cards satisfy MPP charge intents,[^m5] and runs **Intelligent Commerce Connect** as a multi-protocol bridge across TAP, MPP, ACP, and UCP.[^v3]
- **Mastercard** is a founding member of the x402 Foundation,[^x2] runs Agent Pay on its own SD-JWT credential set,[^mc1] is an AP2 partner,[^ap2] and delegates agent identity to Cloudflare's Web Bot Auth - the same RFC 9421 primitive Visa TAP uses.[^mc4]
- **Google** is a founding member of the x402 Foundation,[^x2] originated AP2,[^ap2] and co-authored UCP with Shopify.[^ucp1]
- **Coinbase** built x402,[^x3] sits on the x402 Foundation,[^x2] and is an AP2 partner.[^ap2]
- **Adyen, American Express, and Shopify** appear in the x402 Foundation, the AP2 partner list, and the UCP roster simultaneously.[^x2] [^ap2] [^ucp2]
- **PayPal**, notably, is in AP2[^ap2] and UCP[^ucp2] but absent from the x402 Foundation, where its competitors Stripe and Adyen are present[^x2] - pursuing its own settlement layer rather than joining a stablecoin-anchored consortium.

The most telling structural moment was **Visa Intelligent Commerce Connect** on April 8, 2026.[^v3] Visa's product position is that the right thing to ship to merchants is a single endpoint that translates between TAP, MPP, ACP, and UCP simultaneously, rather than picking one. That is, in plain terms, an institutional admission from one of the field's largest backers that no single agent payment protocol will dominate.

### 3b. An emerging layered stack

What the cross-pollination produces is not a winner. It is a stack. Each protocol category sits at a different layer, and a single agent transaction can touch three or four of them in sequence:

```
┌───────────────────────────────────────────────────────────────┐
│  Commerce layer                                               │
│   UCP (Google + Shopify)  •  ACP (OpenAI + Stripe)            │
│   How agents discover catalogs, build carts, negotiate.       │
├───────────────────────────────────────────────────────────────┤
│  Identity & authorization layer                               │
│   Visa TAP  •  Mastercard Agent Pay (SD-JWT)                  │
│   Amex ACE  •  Google AP2 (Mandates)                          │
│   Cloudflare Web Bot Auth (shared RFC 9421 primitive)         │
│   Who is this agent? What is it allowed to spend?             │
├───────────────────────────────────────────────────────────────┤
│  Payment layer (HTTP 402)                                     │
│   x402  •  MPP  •  L402                                       │
│   The actual charge against an API or service.                │
├───────────────────────────────────────────────────────────────┤
│  Settlement rails                                             │
│   Card networks (Visa, Mastercard, Amex, JCB, UnionPay)       │
│   Stablecoins on Base, Solana, Stellar, Tempo L1              │
│   Bitcoin Lightning  •  Bank rails (Stripe, PayPal, Adyen)    │
└───────────────────────────────────────────────────────────────┘
```

The layers are not strictly disjoint. Visa TAP and Mastercard's identity layer both delegate to Cloudflare Web Bot Auth. MPP's method registry can route to card networks via Visa's Card SDK or to Lightning via Lightspark's extension. Visa's ICC bridge spans the commerce, identity, and payment layers in a single endpoint. But the stack analogy is the right one: each layer answers a different question, and a complete agent transaction touches several of them in sequence - usually routed through several different organizations.

### 3c. What this means for operators

The convergence story has a sharp practical implication for anyone running an API or MCP server: **picking one protocol is betting against the trend.** The companies designing these protocols are not picking one. ICC is Visa's product admission of this. The x402 Foundation's roster - including Stripe, who co-authored MPP - is the same admission from a different angle. Mastercard's open-source Verifiable Intent spec, which makes its credential format usable outside Mastercard's own rails, is a third.

The pragmatic posture is multi-protocol: accept whichever flow an arriving agent presents, and let the abstraction layer beneath your service handle the dispatch. Section 4 walks through what that decision looks like in practice - when to integrate one protocol natively, when to lean on a multi-protocol proxy, and which questions to answer first.

---

## 4. Decision Framework: What Should You Support?

There is no single right answer to "which protocol should I integrate." The right answer depends on two questions, asked in order. Each one rules out parts of the landscape.

### 4a. Who are your agents?

The first question is about the agents that will arrive at your service - not the ones you imagine, the ones you actually have data on.

If your traffic is dominated by **agent platforms with crypto wallets** (Coinbase Agent Kit, Base Sub-Accounts, Skyfire, Crossmint, agent frameworks shipping with embedded wallets), then x402 and L402 are natural fits. The agent already holds USDC or BTC; the payment flow settles in headers without account creation.

If your traffic is dominated by **enterprise agents acting on behalf of corporate buyers** (Copilot in M365, Salesforce Einstein, internal LangChain or n8n agents), the wallet question disappears: the buyer is a company with a card or a bank account, and the agent needs delegated authority to spend it. This is MPP's fiat method, AP2's Mandate model, or one of the card-network identity layers (Visa TAP, Mastercard Agent Pay, Amex ACE).

If your traffic is mixed - the most common case for any public API - you need to support both camps. A multi-protocol surface beats forcing every arriving agent to set up a wallet they don't have, or denying enterprise agents because they don't have one.

### 4b. How much integration work can you absorb?

The integration cost ladder, lowest to highest:

- **Use a multi-protocol proxy or middleware** that handles detection, verification, and dispatch. Your service stays oblivious; the proxy returns a 402 with whatever the arriving agent speaks, settles, and forwards the request. Hours, not weeks.
- **Integrate one protocol natively** (the one closest to your dominant traffic) and route the rest through a fallback. Days.
- **Build native adapters for each protocol** - viable only if agent payments are core to your business and you have engineering bandwidth to track six specs as they evolve. Weeks per protocol, plus ongoing maintenance.

The middle option is the most common posture for an API or MCP server today. The bottom option is rarely the right call: the specs are still moving (x402 V2 shipped December 2025, MPP's IETF draft is at -01, Mastercard's Verifiable Intent will iterate, AP2 is at V0.1), and committing engineering quarters to chasing them is leverage going to the wrong place.

---

## 5. The Compliance Overlay (EU AI Act)

In most of the world, picking an agent payment protocol is a technical decision. In the EU, after August 2, 2026, it stops being one.

### 5a. What Article 26 requires

Regulation (EU) 2024/1689 - the EU AI Act - imposes specific obligations on **deployers** of AI systems classified as high-risk under Annex III. The relevant subset for anyone whose product involves an agent making spending decisions:[^7]

- **Article 26(1):** deployers must use high-risk AI systems in accordance with the system's instructions and human oversight measures.
- **Article 26(2):** deployers must assign human oversight to natural persons with the necessary competence, training, and authority.
- **Article 26(5)-(6):** deployers must keep automatically generated logs for the duration the system is in use, and at minimum six months unless other Union law specifies otherwise.
- **Article 26(7):** deployers must monitor the operation of the high-risk AI system and inform the provider of incidents.

Penalties under **Article 99** reach **up to €15 million or 3% of total worldwide annual turnover**, whichever is higher, for non-compliance with Article 26 obligations.

This is roughly four months away as of publication. Whether a given agent payment use case is "high-risk" under Annex III is being argued by lawyers in real time, but an increasingly common posture among enterprise procurement organizations in regulated sectors is to assume that an agent making autonomous spending decisions on behalf of a corporate buyer is in scope, particularly in financial services, healthcare, and other Annex III categories.

### 5b. How the protocols stack against Article 26

The four obligations above translate into four concrete requirements at the protocol layer:

1. **Cryptographic evidence that a human authorized the agent to act, and within what limits** (Article 26(1), (2)).
2. **Tamper-evident logs of every transaction**, including the agent's identity, the user's intent, the merchant, and the amount (Article 26(5)-(6)).
3. **A human-in-the-loop escalation path** for transactions that exceed policy thresholds (Article 26(2)).
4. **Continuous monitoring with incident reporting** (Article 26(7)).

The protocols differ in how much compliance evidence they generate natively. **This isn't a ranking** - a US deployer can use any of them with comparable engineering effort. For a European deployer, the differences map directly to integration cost: more work to bolt compliance onto protocols that settle inline without generating evidence; less work to use ones designed with mandates and credentials in mind.

- **AP2** is the most compliance-aligned. Its three-Mandate structure (Intent / Cart / Payment) provides the cryptographic primitives - Verifiable Digital Credentials capturing intent, cart, and payment authorization - that an Article 26 audit trail can be built around. The Cart Mandate explicitly captures user authorization. AP2 was designed with this in mind.
- **Mastercard Agent Pay (Verifiable Intent)** carries constraint-bearing intent at the L2 SD-JWT layer - spending limits, merchant categories, time windows. The L3a network-facing payment mandate is a per-transaction signed credential. Selective disclosure means logs can satisfy audit requirements without leaking unrelated data.
- **Visa TAP plus ICC** generates RFC 9421-signed request logs at the network edge, well-suited to retention. Identity is verified upstream of payment, so the audit trail can isolate the agent involved in any incident.
- **MPP** carries authorization in the credential and a receipt in the response, but does not by itself bind the credential to a specific human-issued mandate. Compliance teams typically wrap MPP in a policy layer that enforces spending caps and logs the binding.
- **x402** and **L402** settle inline with no native audit trail beyond the on-chain transaction or the macaroon. For European deployers these protocols require an external policy and logging layer - the protocol settles, the compliance evidence is generated by whatever sits in front of it.

### 5c. What this implies for protocol selection in Europe

Two observations follow.

First, **Article 26's deployer obligations fall on the buyer side, not the seller.** A seller operating an API or MCP server doesn't acquire deployer obligations just because an AI agent paid them - the obligation sits with the organization that deployed the agent and authorized the spending decision.

For sellers, Section 5 is indirect: it shapes what kinds of evidence and credential formats their European enterprise customers will require in order to integrate. For buyers - any organization in Europe running spending agents - it is direct. They need a policy layer above the agent's outbound dispatch that enforces spending caps, escalates high-risk transactions to a human, and writes a compliant audit trail regardless of which protocol ultimately settled the payment. The protocol gives the inline settlement primitive; the buyer-side compliance layer gives the deployer-obligation hooks.

Second, **the European market is roughly four months from this becoming non-optional.** Enterprise procurement teams in regulated sectors are already asking vendors how they handle Article 26 in agent workflows, and a coherent compliance story is becoming a procurement filter alongside price and integration cost - regardless of which protocols sit underneath.

Custena (built by Genesis Software Group) is one implementation of this combination: a multi-protocol settlement surface for sellers, paired with a buyer-side governance layer that produces Article 26 evidence as a side-effect of normal operation. We discuss that briefly in the next section. The general principle holds whichever tool you use: in Europe, the protocol is half the question. The other half is what you put around it.

---

## 6. How Custena Fits

Custena is one specific implementation of the pattern Sections 4 and 5 describe: a multi-protocol settlement surface for sellers, paired with a buyer-side governance layer, with a unified ledger across both that produces Article 26 evidence as a side-effect of normal operation. Built by Genesis Software Group in Copenhagen, with European licensed rails as the foundation rather than an afterthought.

Four properties define the shape:

- **A multi-protocol seller proxy.** The seller registers their API or MCP server once. Whatever protocol the arriving agent speaks - x402, MPP, L402, or fiat top-up - Custena handles the 402 challenge, verification, and settlement; the seller's service receives the request as if it had been called directly.
- **A buyer-side settlement and governance layer.** Buyers fund one Custena balance (card, SEPA, or USDC) and spend it across whichever protocol the agent encounters - x402, MPP, L402, or fiat - without holding a crypto wallet or integrating each rail separately. An SDK and policy layer wrap that outbound spending with configurable spending caps, merchant categories, time windows, and human-in-the-loop escalation for transactions that exceed policy. This is where Article 26 deployer-obligation evidence is generated: each policy decision and each escalation is itself a logged event tied to a specific agent, mandate, and counterparty.
- **Built on European licensed rails.** Fiat flows through Stripe Connect under Stripe's EU EMI license; the Phase 2 USDC custodial layer is being built against the EU CASP framework. Most agent payment infrastructure today is US- or crypto-native and treats Europe as a localization problem. Custena treats it as the primary market.
- **A unified cross-protocol ledger.** Every transaction in either direction - seller-side payment received, buyer-side payment sent - emits a single ledger entry capturing the agent's identity, the spending mandate, the merchant, the amount, and the policy decisions taken at runtime. Logs are retained per the Article 26(5)-(6) horizon.

Custena is in early access as of publication. We'd love to hear your feedback - whether you operate an API or MCP server, run an enterprise compliance function for spending agents, or simply disagree with parts of this report. Contact details are below.

## Citing this report

Lorenzen, A. (2026). _The State of Agent Payment Protocols (April 2026)._ Genesis Software Group. https://github.com/Custena/agent-payment-protocols

---

_Written by [August Lorenzen](https://www.linkedin.com/in/august-plougheld-lorenzen/), founder of Genesis Software Group, builders of [Custena](https://custena.com). Copenhagen. Contact: [august@genesissoftware.io](mailto:august@genesissoftware.io). Licensed CC BY 4.0._
