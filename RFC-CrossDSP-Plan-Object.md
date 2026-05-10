# RFC: Cross-DSP Plan Object — A "Super-Campaign" Container for Agency-Led, Multi-DSP Media Buying

| Field | Value |
|---|---|
| **Document type** | Request for Comments (Informational / Pre-Standards) |
| **Status** | Draft 0.1 |
| **Date** | 2026-05-09 |
| **Editor** | Sean Donahoe |
| **Intended host standard** | IAB Tech Lab AAMP (Agentic Advertising Management Protocols) 2.x, with object reuse from AdCOM v1.x and OpenDirect 2.1 |
| **Category** | Object-model extension (additive; does not modify existing objects) |
| **Audience** | IAB Tech Lab AAMP / OpenDirect / AdCOM working groups; agency holding-co engineering; DSP API teams |

---

## 1. Abstract

This RFC describes a gap in the current IAB Tech Lab object model: there is no standardized container that represents an agency-side **media plan** — a coordinated buy that spans multiple Demand-Side Platforms (DSPs), multiple insertion orders or campaigns, multiple deal types (programmatic guaranteed, PMP, open auction, direct), and a single set of outcome KPIs. We propose a new top-level domain object, provisionally named **`Plan`** (alternative: **`Program`**), that sits above the existing DSP-resident "campaign," "order," "insertion order," and "line item" objects and binds them into a single, agency-owned, cross-platform unit of work for budget allocation, pacing, performance roll-up, and outcome reporting.

This document is explicitly **not** a normative specification. It is a request for comment on the problem statement, the object boundary, the field set, and the recommendation that this work be chartered under AAMP 2.x where it can leverage AAMP's Buyer Agent and Deals Library substrate.

---

## 2. Status of This Memo

This document is a draft RFC published for discussion. It has no formal standing within IAB, IAB Tech Lab, AAMP, OpenDirect, AdCOM, or OpenRTB working groups. Distribution is unlimited. Comments should be directed to the editor.

---

## 3. Motivation

### 3.1 The agency reality

A typical mid-to-large agency campaign for a single brand-advertiser commonly runs across:

- 2–6 DSPs (e.g., The Trade Desk, DV360, Amazon DSP, Yahoo DSP, StackAdapt, Adobe Advertising)
- Multiple deal types within each DSP (open exchange, PMP, programmatic guaranteed)
- Direct publisher buys negotiated under OpenDirect / AAMP flows
- Walled-garden buys (Meta, YouTube, retail media networks) accessed through their own non-IAB APIs
- One or more measurement/MMM partners

Today, the agency reconciles all of this in **proprietary** layers: a media plan in a spreadsheet or planning tool (Mediaocean Prisma, Camphouse, Basis Compass, Wpromote Polaris, Smartly), a finance reconciliation in another, and a performance dashboard in a third. Each DSP exposes its own native hierarchy; nothing in the IAB stack represents the agency's view of the plan as a single object.

### 3.2 What's missing in the standards stack

| Standard | Scope | "Super-campaign" support |
|---|---|---|
| OpenRTB 2.x / 3.0 | Per-impression auction transaction | None — no campaign concept exposed to buyers |
| AdCOM v1.x | Domain objects (Placement, Ad, Site, App, User, etc.) | None — explicitly does not model campaigns |
| OpenDirect 2.1 | Direct/guaranteed booking workflow (RFP → Proposal → Order → Line Item) | Single buyer ↔ single seller; no cross-seller aggregation |
| AAMP 2.0 | Agentic buyer/seller transactions; Buyer Agent SDK accepts a "Campaign Brief" | The brief is an **input** to the agent, not a persistent cross-DSP record |

### 3.3 What's missing in DSP APIs

| DSP | Top of hierarchy | Cross-DSP awareness |
|---|---|---|
| The Trade Desk | Account → Advertiser → Campaign → Ad Group | None |
| DV360 | Partner → Advertiser → Campaign → Insertion Order → Line Item | None |
| Amazon DSP | Entity → Advertiser → Order → Line Item → Creative | None |
| Yahoo DSP | Account → Advertiser → Insertion Order → Line | None |

Every DSP's "Campaign" sits inside its own walled hierarchy. There is no standard object for "this Campaign in TTD and that IO in DV360 and that Order in Amazon DSP are all the same agency plan."

### 3.4 Why now

Three forces make this the right time to standardize:

1. **AAMP 2.0** ships a Buyer Agent SDK that already consumes a "campaign brief" — but the brief is ephemeral and single-buyer-side. Promoting it to a first-class, persistable, multi-DSP object is the natural next step.
2. **Agentic media planning** (Basis Compass, Wpromote Polaris IQ, Kochava StationOne) is rapidly proprietizing the cross-DSP plan layer. Without a standard, agencies will be locked into vendor-specific representations.
3. **Outcome-based buying** (CPA, ROAS, Lift, Brand) demands a cross-DSP unit of measurement. KPIs cannot be honestly evaluated inside any single DSP when budget and audience were split across many.

---

## 4. Terminology

Throughout this document, the key words MUST, SHOULD, MAY are used in the IETF RFC 2119 sense, scoped to recommendations within this proposal (not to existing IAB specs).

- **Plan** — The proposed new object. An agency-owned, cross-DSP, cross-channel container that binds child campaign/order references across multiple platforms under one budget, KPI set, flight, and stakeholder model. (Working name; final name TBD.)
- **Allocation** — A child object on a Plan representing a slice of the Plan's budget directed to a specific DSP, channel, tactic, or partner.
- **Outcome** — A measured business result tied to the Plan (e.g., conversions, lift, ROAS, brand metric).
- **Plan Reference** — A pointer from a DSP-native object (TTD Campaign, DV360 IO, Amazon Order, AAMP Deal) back to its parent `Plan.id`.
- **Brand-Advertiser** — The advertiser entity represented in DSP and IAB hierarchies (`Advertiser` in AdCOM/OpenDirect).
- **Holding-co / Agency** — The buying organization that owns the Plan. Often distinct from the Advertiser.

---

## 5. Existing Standards: A Closer Read

### 5.1 OpenRTB 2.x / 3.0 (with AdCOM)

OpenRTB is impression-level. The bidstream carries a `Bid` referencing an `Ad`/`Placement`, but no buyer-side budget container is exposed. Suitable host? **No.** Out of scope.

### 5.2 AdCOM v1.x

AdCOM defines reusable domain objects: `Placement`, `Ad`, `Audit`, `Site`, `App`, `User`, `Device`, `Distribution Channel`. AdCOM is intentionally a vocabulary, not a workflow. It does **not** model `Campaign`, `Order`, `Plan`, `Budget`. AdCOM is the right place to define a few **shared sub-objects** that a Plan would reference (e.g., audience, brand-safety, contextual targeting), but the Plan itself doesn't belong in AdCOM. Suitable host for sub-objects? **Yes (partial).**

### 5.3 OpenDirect 2.1

OpenDirect is a buyer↔seller direct-booking workflow. Its core objects are `RFP`, `Proposal`, `ProposalLineItem`, `Order`, `OrderLineItem`. Critically, an OpenDirect `Proposal` is the seller's **response to an RFP**, scoped to a single seller's inventory. It is not an agency-side cross-seller container. Reusing the term "Proposal" for the new object would conflict with OpenDirect semantics — hence this RFC uses **Plan**. Suitable host? **No, but adjacent.**

### 5.4 AAMP 2.0 (Agentic Advertising Management Protocols)

AAMP 2.0, released by IAB Tech Lab in early 2026, ships:

- A **Buyer Agent SDK** that accepts a campaign brief (budget, audience, KPIs, flight) and orchestrates discovery, negotiation, and booking against compliant Seller Agents.
- A **Seller Agent SDK** that incorporates OpenDirect and AdCOM to expose inventory, media kits, pricing, and a negotiation state machine.
- A **Deals Library** that persists negotiated deals discovered by the Buyer Agent.

**The AAMP Campaign Brief is the closest existing analogue to the proposed Plan**, but with three critical limitations:

1. The brief is treated as an **input parameter** to a Buyer Agent run, not as a persistent, versioned, top-level object.
2. The Buyer Agent in AAMP 2.0 is bounded to one buyer's perspective and one set of seller agents at a time. It does not natively support spanning multiple DSPs operating as buyer agents themselves (the agency-of-agents case).
3. There is no standardized "outcomes" rollup tying brief → bookings → measurement → reconciliation.

**Recommendation: AAMP is the right home for the Plan object.** AAMP already owns the agentic, automated, cross-counterparty workflow space; the Plan promotes its existing brief into a first-class object that can persist, version, and aggregate.

### 5.5 Other adjacent specs noted but not deep-dived

- **Ads.txt / Sellers.json / SupplyChain Object** — supply integrity, not buyer planning. Out of scope but a Plan should be able to reference a supply-chain policy.
- **Ads.cert** — bidstream signing. Out of scope.
- **GPP (Global Privacy Platform)** — consent signaling. Plans MUST be able to declare applicable privacy frameworks but do not redefine GPP.

---

## 6. Gap Analysis: What the Plan Object Would Solve

A cross-DSP Plan object would, at minimum, enable:

1. **Single source of truth for budget**: one number that all child DSP campaigns sum to, with versioned reallocations.
2. **Outcome-based KPI definition**: KPIs declared once at Plan level, applied to all children, with a defined rollup method.
3. **Pacing across DSPs**: total-plan pacing (front-loaded, even, performance-weighted) computed across all platforms rather than per-DSP.
4. **Aggregated reporting**: a stable identifier for joining DSP delivery data, ad-server data, MMM/MTA data, and finance data.
5. **Cross-DSP audience and brand-safety consistency**: one audience definition, one brand-safety policy, propagated by Plan-aware adapters into each DSP.
6. **Reconciliation and billing**: agency commission, fees, and reconciliation rules expressed once.
7. **Approvals and versioning**: change history with approver identity and timestamps; a defined immutable "as-booked" snapshot.
8. **Agentic substrate**: a Buyer Agent (AAMP) that takes a Plan, fans out to multiple Seller Agents and DSPs, and writes back booked references — closing the loop that AAMP 2.0 leaves open.

---

## 7. Proposed Object Model

### 7.1 Position in the hierarchy

```
                 ┌─────────────────────────────┐
                 │         Plan (NEW)          │  ← Agency-owned, cross-DSP
                 │  (proposed AAMP extension)  │
                 └──────────────┬──────────────┘
                                │ 1..n Allocations
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│  TTD Campaign │       │  DV360 IO     │       │  Amazon Order │
│  (or AAMP/    │       │  (or AAMP/    │       │  (or AAMP/    │
│   OpenDirect  │       │   OpenDirect  │       │   OpenDirect  │
│   Order)      │       │   Order)      │       │   Order)      │
└───────────────┘       └───────────────┘       └───────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
   Ad Groups /              Line Items /           Line Items /
   Line Items               Creatives              Creatives
```

The Plan **does not replace** any DSP-native or AAMP-native object. It sits **above** them and references them. DSP APIs would optionally surface a `plan_ref` field on their Campaign/Order objects to point back; reading the plan_ref on a foreign DSP allows correlation.

### 7.2 Object boundary principles

1. **Additive only.** Existing AAMP, OpenDirect, AdCOM, and DSP objects are not modified. Plan references existing objects by typed identifier.
2. **Reference, not replicate.** The Plan does not store creative assets, audience definitions, or inventory descriptors inline — it references AdCOM-defined sub-objects.
3. **Agency-owned namespace.** A Plan's `id` is issued by an agency or an AAMP-compliant Plan registry. It is **not** a DSP-issued ID.
4. **Versioned and signed.** Every state change produces a new revision. The "as-booked" version is signed and immutable.
5. **Vendor-neutral.** No DSP-specific fields; DSP-specific data lives in `extensions[]` keyed by vendor.

---

## 8. Field Proposal

The following is illustrative, not normative. Field names follow AAMP/AdCOM camelCase convention.

### 8.1 `Plan` (root object)

| Field | Type | Card. | Description |
|---|---|---|---|
| `id` | string (UUID/URN) | 1 | Globally unique Plan identifier issued by the originating agency or registry. |
| `version` | int | 1 | Monotonically increasing revision number. |
| `status` | enum | 1 | `draft`, `pending_approval`, `approved`, `in_market`, `paused`, `closed`, `archived`. |
| `name` | string | 1 | Human-readable plan name. |
| `code` | string | 0..1 | Agency-internal billing/job code. |
| `parentPlanId` | string | 0..1 | Optional parent Plan (for annual program → quarterly flight nesting). |
| `tenant` | object | 1 | `agency`, `holdingCo`, `team`, `owner`. See §8.2. |
| `advertiser` | object | 1 | AdCOM-style advertiser reference (`advertiserId`, `domain`, `categoryTaxonomy`). |
| `brand` | object | 0..1 | Sub-brand under advertiser, if relevant. |
| `flight` | object | 1 | `start`, `end`, `timezone`, optional `phases[]` for multi-phase plans. |
| `objective` | object | 1 | See §8.3 — primary objective and KPIs. |
| `budget` | object | 1 | See §8.4 — total, currency, fee model, contingency. |
| `allocations` | array | 1..n | See §8.5 — per-DSP, per-channel, per-tactic slices. |
| `targeting` | object | 0..1 | AdCOM-referenced shared audience + exclusions + geo + brand safety. |
| `creativeRefs` | array | 0..n | AdCOM `Ad` references intended for this plan. |
| `measurement` | object | 0..1 | See §8.6 — measurement partners, attribution model, study refs. |
| `compliance` | object | 1 | Privacy frameworks (GPP/TCF/USP), category restrictions, approvals. |
| `pacing` | object | 1 | See §8.7 — pacing strategy and rules. |
| `approvals` | array | 0..n | Approver records: identity, role, timestamp, signature. |
| `bookings` | array | 0..n | Pointers to booked DSP/AAMP/OpenDirect objects. See §8.8. |
| `outcomes` | object | 0..1 | Read-mostly aggregated performance roll-up. See §8.9. |
| `extensions` | map<string, any> | 0..1 | Vendor-keyed extension namespace. |
| `audit` | object | 1 | `createdAt`, `createdBy`, `updatedAt`, `updatedBy`, `signature`. |

### 8.2 `Plan.tenant`

```json
{
  "agencyId": "urn:iab:agency:wpp",
  "holdingCoId": "urn:iab:hco:wpp-group",
  "teamId": "team-emea-auto",
  "owner": { "userId": "u-1234", "email": "planner@agency.com" }
}
```

### 8.3 `Plan.objective`

| Field | Type | Description |
|---|---|---|
| `type` | enum | `awareness`, `consideration`, `conversion`, `retention`, `loyalty`, `outcome_custom` |
| `primaryKpi` | object | `metric` (e.g. `cpa`, `roas`, `vcr`, `lift_pct`, `reach`, `frequency`), `target`, `tolerance`, `direction` (`min`/`max`) |
| `secondaryKpis` | array | Same shape as primary. |
| `successCriteria` | string | Free-text business success definition. |
| `rollupMethod` | enum | How DSP-level KPIs aggregate to Plan KPI: `weightedByImpressions`, `weightedBySpend`, `sum`, `dedupedByMatchKey`. |

### 8.4 `Plan.budget`

| Field | Type | Description |
|---|---|---|
| `total` | decimal | Plan-wide gross or net budget. |
| `currency` | string | ISO 4217. |
| `basis` | enum | `gross`, `net`, `working_media`. |
| `feeModel` | object | `agencyCommissionPct`, `techFeePct`, `dataFeePct`, `flatFees[]`. |
| `contingencyPct` | decimal | Reserve held back from allocation. |
| `paymentTerms` | object | `net30`, etc. (for AAMP/OpenDirect bookings). |
| `glCode` | string | Optional general-ledger code for finance reconciliation. |

### 8.5 `Plan.allocations[]`

Each allocation is one slice of the Plan's budget pointed at one buyer-side execution path.

| Field | Type | Description |
|---|---|---|
| `id` | string | Allocation ID (Plan-scoped). |
| `name` | string | Human-readable, e.g. "TTD CTV Q3 Auto-Intender". |
| `channel` | enum | `display`, `video`, `ctv`, `audio`, `dooh`, `native`, `social`, `search`, `retail_media`, `direct`. |
| `dspRef` | object | `dsp` (e.g. `ttd`, `dv360`, `amazon-dsp`, `yahoo-dsp`, `aamp-buyer-agent`), `accountId`, `advertiserId`. |
| `dealType` | enum | `open_auction`, `pmp`, `programmatic_guaranteed`, `direct_io`, `aamp_negotiated`. |
| `amount` | decimal | Allocated budget within the Plan currency. |
| `share` | decimal | Optional share of `Plan.budget.total` (0–1) for derived allocations. |
| `targetingOverrides` | object | Allocation-level audience/geo overrides. |
| `pacingOverrides` | object | Per-allocation pacing rules. |
| `kpiOverrides` | array | Optional allocation-level KPIs that differ from Plan defaults. |
| `bookings[]` | array | DSP-native object refs after execution. See §8.8. |
| `status` | enum | Same vocabulary as Plan.status, scoped to allocation. |

### 8.6 `Plan.measurement`

| Field | Description |
|---|---|
| `attributionModel` | `last_touch`, `data_driven`, `mmm`, `incrementality`, `mta` |
| `windowDays` | Click and view windows. |
| `partners[]` | `{vendor, role, studyId}` — e.g. Nielsen, Comscore, DoubleVerify, IAS, MMM vendor. |
| `conversionEvents[]` | Reference to conversion definitions. |
| `dedupeKey` | The join key used to de-duplicate cross-DSP outcomes. |

### 8.7 `Plan.pacing`

| Field | Description |
|---|---|
| `strategy` | `even`, `front_loaded`, `back_loaded`, `performance_weighted`, `flighted_phases`, `outcome_optimized` |
| `reallocationPolicy` | `auto`, `agent_recommended`, `manual_only` |
| `reallocationCadence` | `realtime`, `hourly`, `daily`, `weekly` |
| `floorPctPerAllocation` | Minimum % of allocation budget that may not be moved. |
| `agentAuthority` | `read`, `recommend`, `execute_with_approval`, `execute` (AAMP Buyer Agent power level) |

### 8.8 `Plan.bookings[]` (and `Allocation.bookings[]`)

Pointers from the Plan into platform-native objects.

```json
{
  "platform": "ttd",
  "objectType": "Campaign",
  "objectId": "tt-c-9f2b...",
  "boundAt": "2026-05-12T13:00:00Z",
  "boundBy": "agent:aamp-buyer-001",
  "allocationId": "alloc-ctv-01",
  "checksum": "sha256-..."
}
```

For AAMP/OpenDirect bookings, `objectType` would be `Order` or `Deal` and `objectId` would be the AAMP Deals Library identifier.

### 8.9 `Plan.outcomes`

A read-mostly rollup updated by ingestion adapters. Not the source of truth — the source is each DSP's reporting API or the measurement partner.

| Field | Description |
|---|---|
| `asOf` | Timestamp of the rollup. |
| `delivered` | `{impressions, clicks, completions, spend, viewableImpressions}` |
| `kpiActuals` | Map of KPI metric → measured value, with rollup method applied. |
| `byAllocation[]` | Per-allocation rollups. |
| `pacingHealth` | `{plannedToDate, deliveredToDate, varianceFwd}` |
| `attribution` | Conversions / lift / ROAS by attribution model. |
| `dataLineage` | `{sources[], lastIngestedAt, dedupeAppliedAt}` |

---

## 9. Reference Examples

### 9.1 Minimal Plan (JSON)

```json
{
  "id": "urn:iab:plan:wpp:2026-q3-auto-launch",
  "version": 1,
  "status": "approved",
  "name": "Auto Co. EV Launch — Q3 2026",
  "tenant": {
    "agencyId": "urn:iab:agency:wpp-mindshare",
    "owner": { "email": "planner@mindshare.com" }
  },
  "advertiser": { "advertiserId": "adv-autoco-na", "domain": "autoco.com" },
  "flight": { "start": "2026-07-01", "end": "2026-09-30", "timezone": "America/New_York" },
  "objective": {
    "type": "consideration",
    "primaryKpi": { "metric": "cpa", "target": 18.50, "direction": "min" },
    "rollupMethod": "dedupedByMatchKey"
  },
  "budget": {
    "total": 4500000.00,
    "currency": "USD",
    "basis": "gross",
    "feeModel": { "agencyCommissionPct": 0.08, "techFeePct": 0.025 }
  },
  "allocations": [
    {
      "id": "alloc-ctv-ttd",
      "channel": "ctv",
      "dspRef": { "dsp": "ttd", "advertiserId": "ttd-adv-autoco" },
      "dealType": "pmp",
      "amount": 1800000.00
    },
    {
      "id": "alloc-display-dv360",
      "channel": "display",
      "dspRef": { "dsp": "dv360", "advertiserId": "dv-adv-autoco" },
      "dealType": "open_auction",
      "amount": 900000.00
    },
    {
      "id": "alloc-retail-amazon",
      "channel": "retail_media",
      "dspRef": { "dsp": "amazon-dsp", "advertiserId": "amz-adv-autoco" },
      "dealType": "open_auction",
      "amount": 1200000.00
    },
    {
      "id": "alloc-direct-aamp",
      "channel": "video",
      "dspRef": { "dsp": "aamp-buyer-agent", "agentId": "agent-buyer-7a" },
      "dealType": "aamp_negotiated",
      "amount": 600000.00
    }
  ],
  "pacing": { "strategy": "performance_weighted", "reallocationPolicy": "agent_recommended" },
  "compliance": { "privacyFrameworks": ["gpp"], "approvedCategories": ["IAB2"] },
  "measurement": {
    "attributionModel": "data_driven",
    "windowDays": { "clickDays": 30, "viewDays": 1 },
    "partners": [{ "vendor": "nielsen", "role": "reach_freq" }]
  }
}
```

### 9.2 Booking write-back from AAMP Buyer Agent

```json
{
  "planId": "urn:iab:plan:wpp:2026-q3-auto-launch",
  "allocationId": "alloc-direct-aamp",
  "booking": {
    "platform": "aamp",
    "objectType": "Deal",
    "objectId": "aamp-deal-3a91f",
    "amountCommitted": 600000.00,
    "boundAt": "2026-06-15T14:22:00Z",
    "boundBy": "agent:aamp-buyer-007",
    "approvals": [{ "userId": "u-1234", "at": "2026-06-15T14:20:00Z" }]
  }
}
```

---

## 10. Recommended Host Standard

**The Plan object SHOULD be chartered as a new object area within IAB Tech Lab AAMP 2.x**, for the following reasons:

1. AAMP already owns the agentic, multi-counterparty buying workflow space.
2. AAMP's existing **Campaign Brief** is the conceptual ancestor; promoting it to a first-class persistent object is an incremental, well-scoped move.
3. AAMP's **Buyer Agent SDK** is the natural execution engine that consumes a Plan and writes back bookings.
4. AAMP's **Deals Library** is the obvious target for the booking-reference write-back.
5. AAMP already composes AdCOM and OpenDirect, so Plan can reuse domain objects (audience, advertiser, ad, placement) without redefinition.

Sub-objects that are **not** plan-specific — particularly an aggregated `Audience` reference, a normalized `BrandSafetyPolicy`, and a normalized `KpiDefinition` — SHOULD be contributed back to **AdCOM** so non-Plan consumers (DSP APIs, OpenDirect proposals) can adopt them.

OpenDirect 2.x SHOULD be left alone. OpenRTB SHOULD be left alone except for an optional `plan_ref` extension on the buyer-side metadata.

Walled-garden APIs (Meta, YouTube, retail-media networks outside Amazon DSP) are out of scope for an IAB-hosted standard, but a Plan SHOULD allow `extensions["walled_garden"]` to capture allocations there for completeness of the agency view.

---

## 11. DSP API Recommendations

This RFC does not require DSP API changes. To realize full value, however, DSPs SHOULD optionally:

1. Accept a `plan_ref` on Campaign/Order/IO create/update calls.
2. Expose `plan_ref` in their reporting APIs so a Plan ingestion adapter can correlate without out-of-band joins.
3. Honor a Plan's `pacing.agentAuthority` value when an AAMP Buyer Agent operates against the DSP.

These are additive, optional fields; no breaking changes are proposed to TTD, DV360, Amazon DSP, or Yahoo DSP APIs.

---

## 12. Security, Privacy, and Trust Considerations

- **Authorization.** A Plan is agency-owned. DSP-side write-backs MUST authenticate the writer (Buyer Agent identity, agency user, or trafficking system) and the relationship to the agency tenant.
- **Signing.** The "as-booked" version of a Plan SHOULD be signed (Ed25519 or equivalent) so downstream finance/measurement systems can verify it has not been mutated.
- **PII.** A Plan MUST NOT carry user-level PII. Audience references are pointers to AdCOM/segment IDs. Conversion definitions reference event taxonomies, not user data.
- **Privacy frameworks.** `Plan.compliance.privacyFrameworks` MUST declare applicable consent regimes (GPP strings, TCF, USP); Buyer Agents MUST honor them when fanning out bookings.
- **Competitive isolation.** Two agencies' Plans within the same DSP MUST NOT be cross-readable; the `plan_ref` exposed to a DSP is opaque to other tenants.
- **Auditability.** Every Plan revision and every booking write-back is logged; AAMP's Buyer Agent already establishes an audit substrate that can be extended to cover Plan-level events.

---

## 13. Open Questions (For the Working Group)

1. **Naming.** `Plan` vs. `Program` vs. `MasterCampaign` vs. `Engagement`. "Plan" is the agency-vernacular term; "Program" reads better when nested (annual Program → quarterly Plans). Working group decision required.
2. **Mandatory vs. optional fields.** This RFC marks most fields optional. A profile system (basic / standard / outcome-rich) may serve adoption better than a single rigid schema.
3. **Governance of the Plan ID namespace.** Agency-issued URNs vs. AAMP-registry-issued IDs. Both have merit; trade-off is portability vs. uniqueness guarantees.
4. **Walled-garden representation.** Should walled-garden allocations be first-class allocations or quarantined to `extensions`? Practical answer is probably first-class, with `dspRef.dsp = "meta"` etc., even though the standard cannot govern those APIs.
5. **Outcome write paths.** Should the rollup be pull-based (Plan engine queries each DSP), push-based (DSPs send delivery to a Plan webhook), or hybrid? Likely hybrid; precedent in measurement integrations.
6. **Reconciliation / billing scope.** Should `feeModel` and `glCode` live here, or in a sibling Finance object? This RFC argues for inclusion because agencies will otherwise re-invent finance objects per vendor.
7. **Versioning semantics.** Append-only revisions vs. mutable current + immutable booked snapshots. AAMP's Deals Library already implements something close; align with it.
8. **Concurrency.** Two planners editing the same Plan: optimistic concurrency on `version`, or CRDT-style merge? Optimistic is simpler and matches existing AAMP patterns.

---

## 14. Implementation Phasing

A pragmatic adoption path:

- **Phase 0 — RFC public comment** (this document). 6–8 weeks, IAB Tech Lab AAMP working group.
- **Phase 1 — Charter a Plan sub-WG under AAMP.** Produce a normative draft schema. 3–4 months.
- **Phase 2 — Reference implementation.** Open-source Plan registry + AAMP Buyer Agent integration that consumes a Plan and writes bookings. 3–6 months.
- **Phase 3 — DSP optional adoption.** TTD / DV360 / Amazon DSP / Yahoo DSP add `plan_ref` fields. 6–12 months.
- **Phase 4 — Formal IAB Tech Lab final.** Versioned spec, conformance tests. 12+ months.

---

## 15. Out of Scope

- Creative production workflow.
- Ad-server (e.g., GAM, FreeWheel) campaign trafficking objects.
- MMM / MTA model definitions (only references to them).
- Talent, agency-internal HR, project-management objects.
- Walled-garden API harmonization.

---

## 16. References

| Spec | URL |
|---|---|
| AAMP — Agentic Advertising Management Protocols | https://iabtechlab.com/standards/aamp-agentic-advertising-management-protocols/ |
| AAMP GitHub | https://github.com/IABTechLab/AAMP |
| AAMP Buyer Agent docs | https://iabtechlab.github.io/buyer-agent/ |
| AdCOM — Advertising Common Object Model | https://iabtechlab.com/standards/adcom-advertising-common-object-model/ |
| AdCOM v1.0 (final) | https://github.com/InteractiveAdvertisingBureau/AdCOM/blob/main/AdCOM%20v1.0%20FINAL.md |
| OpenRTB v3.0 (final) | https://github.com/InteractiveAdvertisingBureau/openrtb/blob/main/OpenRTB%20v3.0%20FINAL.md |
| OpenDirect | https://iabtechlab.com/standards/opendirect/ |
| OpenDirect 2.1 (final) | https://github.com/InteractiveAdvertisingBureau/OpenDirect/blob/main/OpenDirect.v2.1.final.md |
| The Trade Desk Partner Portal — Campaign | https://partner.thetradedesk.com/v3/portal/api/doc/Campaign |
| DV360 API — Resource hierarchy | https://developers.google.com/display-video/api/guides/managing-line-items/resources |
| Amazon DSP API — Developer guide | https://advertising.amazon.com/API/docs/en-us/guides/dsp/developer-guide |

---

## 17. Appendix A — Cross-Standard Object Map

| Concept | OpenRTB 3.0 | AdCOM v1 | OpenDirect 2.1 | AAMP 2.0 | TTD | DV360 | Amazon DSP | **Proposed Plan** |
|---|---|---|---|---|---|---|---|---|
| Advertiser | (via Bid context) | `Advertiser` | `Advertiser` | inherited | Advertiser | Advertiser | Advertiser | references |
| Campaign / Order | n/a | n/a | `Order` | `Deal`/`Order` | Campaign | Campaign / IO | Order | references |
| Line Item / Ad Group | n/a | `Placement` (partial) | `OrderLineItem` | inherited | AdGroup | LineItem | LineItem | references |
| Cross-DSP container | **none** | **none** | **none (single seller)** | **Brief (ephemeral)** | **none** | **none** | **none** | **`Plan` (new)** |
| Budget aggregation | n/a | n/a | per-Order | per-Deal | per-Campaign | per-IO | per-Order | **per-Plan + allocations** |
| Outcome rollup | n/a | n/a | n/a | per-Deal | per-Campaign | per-IO | per-Order | **per-Plan + per-allocation** |

---

## 18. Appendix B — Glossary

- **AAMP** — Agentic Advertising Management Protocols (IAB Tech Lab umbrella, 2026).
- **AdCOM** — Advertising Common Object Model.
- **DSP** — Demand-Side Platform.
- **IO** — Insertion Order.
- **MMM** — Marketing Mix Modeling.
- **MTA** — Multi-Touch Attribution.
- **PMP** — Private Marketplace.
- **PG** — Programmatic Guaranteed.
- **SSP** — Supply-Side Platform.
- **URN** — Uniform Resource Name (used for Plan IDs in examples).

---

*End of RFC draft 0.1. Comments welcomed.*
