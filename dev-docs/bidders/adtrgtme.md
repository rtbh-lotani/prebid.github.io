---
layout: bidder
title: Adtarget.me
description: Prebid Adtarget.me bidder adapter
biddercode: adtrgtme
tcfeu_supported: false
usp_supported: false
coppa_supported: false
schain_supported: false
dchain_supported: false
media_types: banner
safeframes_ok: true
deals_supported: false
floors_supported: false
fpd_supported: true
pbjs: true
pbs: true
pbs_app_supported: true
prebid_member: false
multiformat_supported: will-bid-on-one
sidebarType: 1
---

### Note

The Adtrgtme bidding adapter requires setup before beginning. Please contact us at <support@adtarget.me>

### Bid Params

{: .table .table-bordered .table-striped }

| Name  | Scope    | Description | Example       | Type     |
|-------|----------|-------------|---------------|----------|
| `sid` | required | Site ID     | "1234567890"  | `string` |
| `zid` | optional | Zone ID     | `1234567890`  | `number` |
