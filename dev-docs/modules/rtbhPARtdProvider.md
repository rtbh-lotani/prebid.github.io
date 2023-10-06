---
 layout: page_v2
 title: RTB House Prime Audience Realtime Data Module
 display_name: RTB House Prime Audience
 description: Injects Prime Audience advertiser tagging script
 page_type: module
 module_type: rtd
 module_code : rtbhPARtdProvider
 enable_download : true
 vendor_specific: true
 sidebarType : 1
 ---

 # RTB House Prime Audience Realtime Data Module
{:.no_toc}

* TOC
{:toc}

 ## Overview
This module is intended to be used by Prime Audience (https://primeaudience.com) partners and puts dedicated advertiser tagging script onto publisher's web page.

  {% include dev-docs/loads-external-javascript.md %}

## Integration

1. Build the RTB House Prime Audience RTD Module along with your bid adapter and other modules into your prebid.js build with:

    ```
    gulp build --modules="rtbhPARtdProvider,rtdModule,..."
    ```
2. Use `pbjs.setConfig()` method to instruct Prebid.js to initialize the module, as specified below:
    ```javascript
    pbjs.setConfig({
        "realTimeData": {
            "dataProviders": [
                {
                    "name": "rtbhPA",
                    "params": {
                        "tagId": "customerID",
                        "region": "us",
                    }
                }
            ]
        }
    });
    ``` 
## Configuration

This module is configured as part of the `realTimeData.dataProviders` object.

{: .table .table-bordered .table-striped }
| Name            | Scope    | Description                  | Example         |  Type    |
|:----------------|:---------|:-----------------------------|:----------------|:---------|
| `name`          | required | Real time data module name   | `'rtbhPA'`      | `string` |
| `params`        | required |                              |                 | `Object` |
| `params.tagId`  | required | Your Prime Audience customer ID, [Reach out to us](https://www.primeaudience.com/#contact) to know more! | `'customerID'` | `string` |
| `params.region` | optional | Publisher related geo region. Defaults to `'ams'`. Other values: `'us', 'asia'` | `'us'`   | `string` |

{: .alert.alert-warning :}
tagId is publisher specific customer ID which will be provided to the publisher by Prime Audience.

## Disclosure
This module loads external code using the passed parameter (params.tagId). The code is loaded from tags.creativecdn.com domain.