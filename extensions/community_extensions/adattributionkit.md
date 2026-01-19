# Apple AdAttributionKit (AAK)

**Status:** Draft (Community Extension Proposal)  
**Related:** OpenRTB 2.x / 3.x Extensions Mechanism  
**Primary Contact / Editors:** *<your name / org>*  
**Version:** 0.1 (Draft)

## 1. Overview

Apple’s **AdAttributionKit** is a privacy-preserving attribution framework in which:

- **Ad networks** register with Apple, receive an **ad network identifier**, and **sign ads** to make them eligible for attribution and postbacks.   
- **Publisher apps** display those ads and must include allowed ad network identifiers in their app configuration for impressions to qualify.   
- **Advertised apps** update conversion values; Apple delivers **postbacks** (as **JWS**) to ad networks (and optionally developers).   

AdAttributionKit postbacks include fields such as **impression-type**, **ad-network-identifier**, **source-identifier**, **advertised-item-identifier**, **conversion-type**, **postback-identifier**, **did-win**, and **postback-sequence-index** (with some optional fields). 

This proposal defines a standardized way in OpenRTB to:

1. Signal **publisher eligibility constraints** (which ad networks are allowed/installed for AAK attribution in the publisher app).
2. Allow bidders to return **AdAttributionKit attribution materials** (e.g., a signed impression payload) required for publisher-side registration.
3. Support both **install** and **reengagement** flows (where applicable), aligned with AdAttributionKit conversion types.   

> Note: Apple indicates AdAttributionKit supports **JWS formatted impressions and postbacks**. 

---

## 2. Use Cases

### 2.1 App-install attribution via OpenRTB
A DSP bids with creatives that are eligible for AdAttributionKit attribution. The exchange/publisher app needs sufficient information to register the impression/click for attribution and later receive postbacks.

### 2.2 Reengagement campaigns (retargeting)
AdAttributionKit supports a conversion type representing reengagement (“re-engagement”).   
The bidder may need to indicate that the ad is eligible for reengagement measurement and provide the appropriate destination URL inputs.

### 2.3 Winner and runner-up postbacks
For install conversions, Apple indicates **one winner** plus up to **five non-winning** (“did-win”: false) postbacks for qualifying networks.   
(OpenRTB itself doesn’t deliver postbacks, but this impacts how participants interpret measurement and deduplication.)

---

## 3. Extension Name and Placement

### 3.1 Extension Key
`aakn` (AdAttributionKit Network)

### 3.2 Object Placement (OpenRTB 2.x)
- **BidRequest.Imp.ext.aakn** — publisher signals eligibility / constraints and capabilities
- **BidResponse.SeatBid.Bid.ext.aakn** — bidder returns AdAttributionKit materials

(Implementations MAY also attach `aakn` under `BidResponse.ext` if they prefer response-wide declarations, but the canonical placement is at the bid level to support multi-imp responses.)

### 3.3 Object Placement (OpenRTB 3.x)
- **Request.item[].spec.ext.aakn**
- **Response.seatbid[].bid[].ext.aakn**

---

## 4. `aakn` Object (Request)

### 4.1 Rationale
Publisher apps must be configured with ad network identifiers for attribution eligibility.   
The request object allows the publisher/exchange to communicate which AAK ad networks are supported/allowed in that app context.

### 4.2 Object Definition: `BidRequest.imp.ext.aakn`

| Field | Type | Scope | Description |
|---|---:|---|---|
| `netids` | `string[]` | recommended | List of AdAttributionKit ad network identifiers supported by the publisher app in this context (lowercase). Apple indicates ad network IDs are lowercase identifiers of the form `example123.adattribuitionkit`.  |
| `netlist` | `object` | optional | Reference to a remotely hosted list of supported AdAttributionKit ad network IDs (see `aaknetlist` below). Intended for large/maintained lists. (See also IAB’s SKAdNetwork ID list tooling as precedent.)  |
| `support_reengagement` | `boolean` | optional | Indicates whether the publisher placement is eligible for reengagement measurement flows (if buyer supports). |
| `require_jws_impression` | `boolean` | optional | If true, the buyer must supply a signed impression payload (JWS) in the response `aakn` object. AdAttributionKit uses JWS for postbacks and Apple describes JWS formatted impressions.  |

#### Notes
- `netids` and `netlist` are mutually compatible; if both are present, `netids` takes precedence for immediate filtering, while `netlist` may be used for auditing/completeness.

---

## 5. `aaknetlist` Object (Request)

### 5.1 Purpose
Some publishers will prefer to reference a maintained list (similar to how the ecosystem uses centralized SKAdNetwork ID lists). 

### 5.2 Object Definition: `BidRequest.imp.ext.aakn.netlist`

| Field | Type | Description |
|---|---:|---|
| `url` | `string` | HTTPS URL to fetch a JSON document containing supported AdAttributionKit ad network IDs. |
| `format` | `string` | Format identifier, e.g. `"aakn-netids-1"` |
| `max` | `integer` | Optional maximum number of entries the fetcher should accept. |
| `excl` | `string[]` | Optional list of excluded ad network IDs (e.g. blocked partners). |

### 5.3 Suggested JSON Format (`aakn-netids-1`)
```json
{
  "format": "aakn-netids-1",
  "generated_at": "2026-01-15T00:00:00Z",
  "netids": [
    "example123.adattribuitionkit",
    "example456.adattribuitionkit"
  ]
}
