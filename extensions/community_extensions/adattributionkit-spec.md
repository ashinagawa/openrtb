# AdAttributionKit

## Abstract

This document describes an extension to OpenRTB for Apple's [AdAttributionKit](https://developer.apple.com/documentation/AdAttributionKit) framework. AdAttributionKit (AAK) is Apple's new attribution framework that expands on SKAdNetwork to improve transparency and measurement while respecting user privacy.

## Overview

This document describes how AdAttributionKit (AAK) data should be communicated in OpenRTB bid requests and bid responses. AAK currently lacks a standardized bid request and bid response specification, which could result in inconsistent implementations and attribution behavior as more DSPs adopt AAK.

The specification defines:
- A new extension to the Bid Request object to communicate AAK support and parameters
- A new extension to the Bid Response object to return AAK attribution data

## Participant Responsibilities

### SSPs
- Indicate AAK support by including the `adattributionkit` extension in bid requests when traffic is eligible
- Pass AAK data from DSP bid responses to SDKs via ad responses

### DSPs
- Return AAK attribution data in bid responses when the bid request indicates AAK support
- Include signed JWS objects for attribution

### SDKs
- Process AAK data from ad responses to handle attributions using the AAK framework
- Create `AppImpression` objects and manage attribution flows

## Limitations

- AdAttributionKit is only applicable to **iOS 17.4+** devices
- Re-engagement flows are only applicable to **iOS 18+** devices
- Publishers must configure AdNetwork IDs for AAK in their `Info.plist`
- Compatible AdNetwork IDs can be used in both SKAdNetwork and AAK sections

For more information on publisher configuration, see Apple's documentation: [Configuring a Publisher App](https://developer.apple.com/documentation/adattributionkit/configuring-a-publisher-app)

---

# Bid Request

## Object: `BidRequest.imp.ext.adattributionkit`

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

### Example Bid Request with Both AAK and SKAN

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
        },
        "skadn": {
          "version": "4.0",
          "versions": [
            "2.0",
            "2.2",
            "3.0",
            "4.0"
          ],
          "sourceapp": "123123123",
          "skadnetids": [
            "m8dbw4sv7c.skadnetwork",
            "tl55sbb4fm.skadnetwork",
            "6xzpu9s2p8.skadnetwork",
            "m2jqnlggk3.adattributionkit"
          ],
          "ext": {
            "sko": 0
          }
        }
      }
    }
  ]
}
```

---

# Bid Response

## Object: `BidResponse.seatbid.bid.ext.adattributionkit`

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

---

# SDK Implementation Guide

## Intended Usage

DSPs will include the AAK data in bid responses for SSPs to pass to SDKs via ad responses. SDKs then use this data to process attributions with the AAK framework:

1. **Create AppImpression**: SDK creates `AppImpression` object with the `jwt` from the bid response.
   - [Apple Documentation: AppImpression](https://developer.apple.com/documentation/adattributionkit/appimpression)

2. **Load StoreKit Product View**: SDK loads `SKStoreProductViewController` with the `AppImpression` object.
   - [Apple Documentation: SKStoreProductViewController](https://developer.apple.com/documentation/storekit/skstoreproductviewcontroller)

3. **Configure SKOverlay**: SDK uses the `AppImpression` object to configure `SKOverlay.AppConfiguration`.
   - [Apple Documentation: SKOverlay.AppConfiguration](https://developer.apple.com/documentation/storekit/skoverlay/appconfiguration)

4. **View-Through Attribution**: SDK uses the `AppImpression` object to begin and end View-Through attribution.
   - [Begin: Apple Documentation](https://developer.apple.com/documentation/adattributionkit/appimpression/beginview())
   - [End: Apple Documentation](https://developer.apple.com/documentation/adattributionkit/appimpression/endview())

5. **Custom Click Attribution (Re-engagement)**: SDK uses the `AppImpression` object to call `handleTap(reengagementURL:)` with the `reengagementurl` for Custom Click attribution.
   - [Apple Documentation: handleTap](https://developer.apple.com/documentation/adattributionkit/appimpression/handletap(reengagementurl:))

---

# Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | TBD | Initial release |
