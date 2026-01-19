# Apple AdAttributionKit (AAK)

**Status:** Draft (Community Extension Proposal)  
**Related:** OpenRTB 2.x / 3.x Extensions Mechanism  
**Version:** 0.1 (Draft)

Sponsors: TBD

Document verison support: AdAttributionKit versions 1.0. Support for newer versions will be brought up for consideration within the IAB TL Programmatic working group subcommittee.

## 1. Overview

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

---

## 3. Extension Name and Placement

### 3.1 Extension Key
`adattributionkit` (AdAttributionKit Network)

### 3.2 Object Placement (OpenRTB 2.x)
- **BidRequest.Imp.ext.adattributionkit** — publisher signals eligibility / constraints and capabilities
- **BidResponse.SeatBid.Bid.ext.adattributionkit** — bidder returns AdAttributionKit materials


### 3.3 Object Placement (OpenRTB 3.x)
- **Request.item[].spec.ext.adattributionkit**
- **Response.seatbid[].bid[].ext.adattributionkit**

---

## 4. Bid Request

### Object: `BidRequest.imp.ext.adattributionkit`

When traffic is eligible for AdAttributionKit, SSPs should include a new `adattributionkit` object under `BidRequest.imp.ext`. This object informs DSPs that they can respond with AAK data for attribution.

The object is only present if both the SSP SDK version and the OS version (iOS 17.4+) support AdAttributionKit.

| Attribute | Type | Description |
|-----------|------|-------------|
| version | string; required | Version of AdAttributionKit supported (e.g., "1.0"). Dependent on both the OS version and the SDK version. |
| sourceapp | string; required | The App Store ID of the publisher's app. |
| skadnetids | array of strings; required | A subset of `SKAdNetworkItem` entries in the publisher app's `Info.plist` that are relevant to the bid request. These are the AdNetwork IDs that the DSP can use for attribution. |
| ext | object; optional | Placeholder for exchange-specific extensions to OpenRTB. |
| ext.sko | integer; optional | Indicates whether SKOverlay is available. `1` = available, `0` = not available. |

### Example Bid Request

```json
{
  "imp": [
    {
      "ext": {
        "adattributionkit": {
          "version": "1.0",
          "sourceapp": "123123123",
          "skadnetids": [
            "m8dbw4sv7c.skadnetwork",
            "m2jqnlggk3.adattributionkit"
          ],
          "ext": {
            "sko": 1
          }
        }
      }
    }
  ]
}
```

          "version": "1.0",
---

## 5. Bid Response

### Object: `BidResponse.seatbid.bid.ext.adattributionkit`

If the bid request indicated AAK support, DSPs can return AAK attribution data using a custom extension field under `BidResponse.seatbid.bid.ext.adattributionkit`.

| Attribute | Type | Description |
|-----------|------|-------------|
| jwt | string; required | Signed compact JWS object to be used on device for AAK implementation. This contains the signed attribution data. |
| version | string; required | Version of AdAttributionKit (e.g., "1.0"). |
| itunesitem | string; required | The App Store ID of the advertised app. |
| cpp | string; optional | The Custom Product Page ID (PPID) for the advertised app. |
| reengagementurl | string; optional | The re-engagement URL for Custom Click attribution. Only supported on iOS 18+. |
| ext | object; optional | Placeholder for exchange-specific extensions to OpenRTB. |
| ext.skoverlay | object; optional | Object containing SKOverlay configuration parameters. |

### Object: `ext.skoverlay`

Configuration for SKOverlay presentation.

| Attribute | Type | Description |
|-----------|------|-------------|
| show | integer | Whether to show the SKOverlay. `1` = show, `0` = do not show. |
| delay | integer | Delay in seconds before showing the overlay. |
| companion_delay | integer | Delay in seconds for companion overlay. |
| pos | integer | Position of the overlay. |
| autoclose | integer | Auto-close delay in seconds. |
| dismissible | integer | Whether the overlay is dismissible. `1` = dismissible, `0` = not dismissible. |
| click_on_view | integer | Whether clicks on the view trigger the overlay. `1` = enabled, `0` = disabled. |

### Example Bid Response

```json
{
  "seatbid": [
    {
      "bid": [
        {
          "ext": {
            "adattributionkit": {
              "jwt": "eyJhbGciOiJFUzI1NiIsImtpZCI6ImZha2Uua2V5In0.eyJpbXByZXNzaW9uLXR5cGUiOiJhcHAtaW1wcmVzc2lvbiIsImFkLW5ldHdvcmstaWRlbnRpZmllciI6Im15ZHNwLmFkYXR0cmlidXRpb25raXQiLCJwdWJsaXNoZXItaXRlbS1pZGVudGlmaWVyIjowLCJzb3VyY2UtaWRlbnRpZmllciI6MTIzNCwidGltZXN0YW1wIjoxNzAwMDAwMDAwfQ.signature",
              "version": "1.0",
              "itunesitem": "12345678",
              "cpp": "d7db643c-f84f-41d5-b2b3-fce30bf73640",
              "reengagementurl": "https://app.com/re",
              "ext": {
                "skoverlay": {
                  "show": 1,
                  "delay": 5,
                  "companion_delay": 3,
                  "pos": 1,
                  "autoclose": 3,
                  "dismissible": 1,
                  "click_on_view": 0
                }
              }
            }
          }
        }
      ]
    }
  ]
}
```
