---
layout: page_v2
title: Integrating Prebid SDK iOS with AdMob
description: Integrating Prebid SDK iOS with AdMob
sidebarType: 2
---

<!-- markdownlint-disable-file MD046 -->

# Prebid SDK iOS with AdMob Integration Method
{:.no_toc}

- TOC
{:toc}

{% include mobile/intro-admob.md platform="ios" %}

## Setup

Prebid SDK is integrated into AdMob setup thru custom adapters. To integrate Prebid Adapters into your app, add the following line to your Podfile:

```pod
pod 'PrebidMobileAdMobAdapters'
```

### Initialization

{: .alert.alert-warning :}
**Warning:** The `GADMobileAds.sharedInstance().start()` should be called in the adapters bundle, otherwise, GMA SDK won't load the ads with error: `adView:didFailToReceiveAdWithError: SDK tried to perform a networking task before being initialized.`

To avoid the error add the following line to your app right after initialization of GMA SDK:

```swift
AdMobUtils.initializeGAD()
```

## Adunit Specific Instructions

### Banners

**Integration example(Swift)**:

{% capture gma12 %}// 1. Create a GADRequest
let gadRequest = Request()

// 2. Create a BannerView
gadBanner = GoogleMobileAds.BannerView(adSize: adSizeFor(cgSize: AD_SIZE))
gadBanner.adUnitID = AD_UNIT_ID
gadBanner.delegate = self
gadBanner.rootViewController = self

// Add GMA SDK banner view to the app UI
bannerView.addSubview(gadBanner)

// 3. Create an AdMobMediationBannerUtils
mediationDelegate = AdMobMediationBannerUtils(gadRequest: gadRequest, bannerView: gadBanner)

// 4. Create a MediationBannerAdUnit
prebidAdMobMediaitonAdUnit = MediationBannerAdUnit(
    configID: CONFIG_ID,
    size: AD_SIZE,
    mediationDelegate: mediationDelegate
)

// 5. Make a bid request to Prebid Server
prebidAdMobMediaitonAdUnit.fetchDemand { [weak self] result in
    PrebidDemoLogger.shared.info("Prebid demand fetch for AdMob \(result.name())")
    
    // 6. Load ad
    self?.gadBanner.load(gadRequest)
}
{% endcapture %}
{% capture gma11 %}// 1. Create GADRequest and GADBannerView
gadRequest = GADRequest()

gadBanner = GADBannerView(adSize: AD_SIZE)
gadBanner.delegate = self
gadBanner.rootViewController = self
    
gadBanner.adUnitID = AD_UNIT_ID
    
// 2. Create an AdMobMediationBannerUtils
mediationDelegate = AdMobMediationBannerUtils(gadRequest: gadRequest, bannerView: gadBanner)
    
// 3. Create the MediationBannerAdUnit
prebidAdMobMediaitonAdUnit = MediationBannerAdUnit(configID: CONFIG_ID,
                                                   size: AD_SIZE,
                                                   mediationDelegate: mediationDelegate)
    
// 4. Make a bid request
prebidAdMobMediaitonAdUnit.fetchDemand { [weak self] result in
    
// 5. Make an ad request to AdMob
    self?.gadBanner.load(self?.gadRequest)
}
{% endcapture %}

{% include code/gma-versions-tabs.html id="admob-banner" gma11=gma11 gma12=gma12 %}

#### Step 1: Create Request and BannerView
{:.no_toc}

This step is the same as for the original [AdMob integration](https://developers.google.com/admob/ios/banner). You don't have to make any modifications here.

#### Step 2: Create AdMobMediationBannerUtils
{:.no_toc}

The `AdMobMediationBannerUtils` is a helper class, which performs certain utility work for the `MediationBannerAdUnit`, such as passing the targeting keywords to the adapters and checking the visibility of the ad view.

#### Step 3: Create MediationBannerAdUnit
{:.no_toc}

The `MediationBannerAdUnit` is part of Prebid mediation API. This class is responsible for making a bid request and providing the winning bid and targeting keywords to mediating SDKs.  

#### Step 4: Make bid request
{:.no_toc}

The `fetchDemand` method makes a bid request to a Prebid server and returns a result in a completion handler.

#### Step 5: Make an Ad Request
{:.no_toc}

Make a regular AdMob's ad request. Everything else will be handled by Prebid adapters.

### Interstitials

**Integration example(Swift)**:

{% capture gma12 %}// 1. Create a Request
let gadRequest = Request()

// 2. Create an AdMobMediationInterstitialUtils
let mediationDelegate = AdMobMediationInterstitialUtils(gadRequest: gadRequest)

// 3. Create a MediationInterstitialAdUnit
admobAdUnit = MediationInterstitialAdUnit(
    configId: CONFIG_ID,
    mediationDelegate: mediationDelegate
)

// 4. Make a bid request to Prebid Server
admobAdUnit?.fetchDemand(completion: { [weak self] result in
    PrebidDemoLogger.shared.info("Prebid demand fetch for AdMob \(result.name())")
    
    // 5. Load the interstitial ad
    InterstitialAd.load(
        with: AD_UNIT_ID,
        request: gadRequest
    ) { [weak self] ad, error in
        guard let self = self else { return }
        
        if let error = error {
            PrebidDemoLogger.shared.error("\(error.localizedDescription)")
            return
        }
        
        // 6. Present the interstitial ad
        self.interstitial = ad
        self.interstitial?.fullScreenContentDelegate = self
        self.interstitial?.present(from: self)
    }
})
{% endcapture %}
{% capture gma11 %}
// 1. Create GADRequest
gadRequest = GADRequest()

// 2. Create AdMobMediationInterstitialUtils
let mediationDelegate = AdMobMediationInterstitialUtils(gadRequest: self.gadRequest)

// 3. Create MediationInterstitialAdUnit
admobAdUnit = MediationInterstitialAdUnit(configId: CONFIG_ID, mediationDelegate: mediationDelegate)

// 4. Make a bid request
admobAdUnit?.fetchDemand(completion: { [weak self]result in
    
// 5. Make an ad request to AdMob
GADInterstitialAd.load(withAdUnitID: AD_UNIT_ID, request: self?.gadRequest) { [weak self] ad, error in
    guard let self = self else { return }
    if let error = error {
        PBMLog.error(error.localizedDescription)
        return
    }
        
        // 6. Present the interstitial ad
        self.interstitial = ad
        self.interstitial?.fullScreenContentDelegate = self
        self.interstitial?.present(fromRootViewController: self)
    }
})
{% endcapture %}

{% include code/gma-versions-tabs.html id="admob-banner" gma11=gma11 gma12=gma12 %}

The **default** ad format for interstitial is **.banner**. In order to make a `multiformat bid request`, set the respective values into the `adFormats` property.

```swift
// Make bid request for video ad                                     
adUnit?.adFormats = [.video]

// Make bid request for both video amd banner ads                                     
adUnit?.adFormats = [.video, .banner]

// Make bid request for banner ad (default behaviour)                                     
adUnit?.adFormats = [.banner]
```

#### Step 1: Create Request
{:.no_toc}

This step is the same as for the original [AdMob integration](https://developers.google.com/admob/ios/interstitial#swift). You don't have to make any modifications here.

#### Step 2: Create AdMobMediationInterstitialUtils
{:.no_toc}

The `AdMobMediationInterstitialUtils` is a helper class, which performs certain utility work for the `MediationInterstitialAdUnit`, such as passing the targeting keywords to adapters and checking the visibility of the ad view.

#### Step 3: Create MediationInterstitialAdUnit
{:.no_toc}

The `MediationInterstitialAdUnit` is part of the Prebid mediation API. This class is responsible for making a bid request and providing a winning bid to the mediating SDKs.  

#### Step 4: Make bid request
{:.no_toc}

The `fetchDemand` method makes a bid request to a Prebid server and provides a result in a completion handler.

#### Step 5: Make an Ad Request
{:.no_toc}

Make a regular AdMob's ad request. Everything else will be handled by GMA SDK and prebid adapters.

#### Steps 6: Display an ad
{:.no_toc}

Once you receive the ad it will be ready for display. Follow the [AdMob instructions](https://developers.google.com/admob/ios/interstitial#swift) for displaying an ad.

### Rewarded

{% include mobile/rewarded-server-side-configuration.md %}

**Integration example(Swift)**:

{% capture gma12 %}// 1. Create a Request
let request = Request()

// 2. Create an AdMobMediationRewardedUtils
mediationDelegate = AdMobMediationRewardedUtils(gadRequest: request)

// 3. Create a MediationRewardedAdUnit
admobRewardedAdUnit = MediationRewardedAdUnit(
    configId: CONFIG_ID,
    mediationDelegate: mediationDelegate
)

// 4. Make a bid request to Prebid Server
admobRewardedAdUnit.fetchDemand { [weak self] result in
    guard let self = self else { return }
    PrebidDemoLogger.shared.info("Prebid demand fetch for AdMob \(result.name())")
    
    // 5. Load the rewarded ad
    RewardedAd.load(with: AD_UNIT_ID, request: request) { [weak self] ad, error in
        guard let self = self else { return }
        
        if let error = error {
            Log.error(error.localizedDescription)
            return
        }
        
        // 6. Present the rewarded ad
        self.gadRewardedAd = ad
        self.gadRewardedAd?.fullScreenContentDelegate = self
        DispatchQueue.main.asyncAfter(deadline: .now() + .seconds(3)) {
            self.gadRewardedAd?.present(
                from: self,
                userDidEarnRewardHandler: {
                    print("User did earn reward.")
                }
            )
        }
    }
}
{% endcapture %}
{% capture gma11 %}// 1. Create GADRequest
let request = GADRequest()

// 2. Create AdMobMediationInterstitialUtils
let mediationDelegate = AdMobMediationRewardedUtils(gadRequest: request)

// 3. Create MediationInterstitialAdUnit
admobRewardedAdUnit = MediationRewardedAdUnit(configId: CONFIG_ID, mediationDelegate: mediationDelegate)

// 4. Make a bid request
admobRewardedAdUnit.fetchDemand { [weak self] result in
    guard let self = self else { return }

// 5. Make an ad request to AdMob
GADRewardedAd.load(withAdUnitID: AD_UNIT_ID, request: request) { [weak self] ad, error in
    guard let self = self else { return }
    if let error = error {
        PBMLog.error(error.localizedDescription)
        return
    }

    // 6. Present the interstitial ad
    self.gadRewardedAd = ad
    self.gadRewardedAd?.fullScreenContentDelegate = self
    DispatchQueue.main.asyncAfter(deadline: .now() + .seconds(3)) {
        self.gadRewardedAd?.present(fromRootViewController: self, userDidEarnRewardHandler: {
            print("Reward user")
        })
    }
    }
}
{% endcapture %}

{% include code/gma-versions-tabs.html id="admob-rewarded" gma11=gma11 gma12=gma12 %}

The process of displaying the rewarded ad is the same as for displaying an Interstitial Ad.

To be notified when a user earns a reward follow the [AdMob intructions](https://developers.google.com/admob/ios/rewarded#show_the_ad).

#### Step 1: Create Request
{:.no_toc}

This step is the same as for the original [AdMob integration](https://developers.google.com/admob/ios/rewarded). You don't have to make any modifications here.

#### Step 2: Create MediationRewardedAdUnit
{:.no_toc}

The `AdMobMediationRewardedUtils` is a helper class, which performs certain utility work for the `MediationRewardedAdUnit`, like passing the targeting keywords to the adapters.

#### Step 3: Create MediationInterstitialAdUnit
{:.no_toc}

The `MediationRewardedAdUnit` is part of the Prebid mediation API. This class is responsible for making a bid request and providing a winning bid and targeting keywords to the adapters.  

#### Step 4: Make bid request
{:.no_toc}

The `fetchDemand` method makes a bid request to the a Prebid server and provides a result in a completion handler.

#### Step 5: Make an Ad Request
{:.no_toc}

Make a regular AdMob's ad request. Everything else will be handled by GMA SDK and prebid adapters.

#### Steps 6: Display an ad
{:.no_toc}

Once the rewarded ad is received you can display it. Follow the [AdMob instructions](https://developers.google.com/admob/ios/rewarded#swift) for displaying an ad.

### Native Ads

{: .alert.alert-warning :}
**Warning:** If you use Native Ads you **must** integrate AdMob Adapters via the source files instead of cocoapods integration or standalone framework.

In order to integrate AdMob adapters just add the adapters' source files to your app project.

**Integration example(Swift)**:

{% capture gma12 %}// 1. Create a Request
let gadRequest = Request()

// 2. Create an AdMobMediationNativeUtils
mediationDelegate = AdMobMediationNativeUtils(gadRequest: gadRequest)

// 3. Create a MediationNativeAdUnit
admobMediationNativeAdUnit = MediationNativeAdUnit(
    configId: CONFIG_ID,
    mediationDelegate: mediationDelegate
)

// 4. Configure MediationNativeAdUnit
admobMediationNativeAdUnit.addNativeAssets(nativeRequestAssets)
admobMediationNativeAdUnit.setContextType(.Social)
admobMediationNativeAdUnit.setPlacementType(.FeedContent)
admobMediationNativeAdUnit.setContextSubType(.Social)
admobMediationNativeAdUnit.addEventTracker(eventTrackers)

// 5. Make a bid request to Prebid Server
admobMediationNativeAdUnit.fetchDemand { [weak self] result in
    guard let self = self else { return }
    PrebidDemoLogger.shared.info("Prebid demand fetch for AdMob \(result.name())")
    
    // 6. Load the native ad
    self.adLoader = AdLoader(
        adUnitID: AD_UNIT_ID,
        rootViewController: self,
        adTypes: [ .native ],
        options: nil
    )
    
    self.adLoader?.delegate = self
    
    self.adLoader?.load(gadRequest)
}
{% endcapture %}
{% capture gma11 %}// 1. Create GAD Request
gadRequest = GADRequest()

// 2. Create AdMobMediationNativeUtils
mediationDelegate = AdMobMediationNativeUtils(gadRequest: gadRequest)

// 3. Create and configure MediationNativeAdUnit
nativeAdUnit = MediationNativeAdUnit(configId: prebidConfigId,
                                     mediationDelegate: mediationDelegate!)

nativeAdUnit.setContextType(ContextType.Social)
nativeAdUnit.setPlacementType(PlacementType.FeedContent)
nativeAdUnit.setContextSubType(ContextSubType.Social)
 
// 4. Set up assets for bid request
nativeAdUnit.addNativeAssets(nativeAssets)

// 5. Set up event tracker for bid request
nativeAdUnit.addEventTracker(eventTrackers)

// 6. Make a bid request
nativeAdUnit.fetchDemand { [weak self] result in
    guard let self = self else { return }

    // 7. Load AdMob Native ad
    self.adLoader = GADAdLoader(adUnitID: AD_UNIT_ID,
                                rootViewController: self.rootController,
                                adTypes: [ .native ],
                                options: nil)

    self.adLoader?.delegate = self

    self.adLoader?.load(self.gadRequest)
}
{% endcapture %}

{% include code/gma-versions-tabs.html id="admob-native" gma11=gma11 gma12=gma12 %}

#### Step 1: Create a Request
{:.no_toc}

Prepare the `Request` object before you make a bid request. It will be needed for the Prebid mediation utils.

#### Step 2: Create AdMobMediationNativeUtils
{:.no_toc}

The `AdMobMediationNativeUtils` is a helper class, which performs certain utility work for `MediationNativeAdUnit`, like passing the targeting keywords to adapters and checking the visibility of the ad view.

#### Step 3: Create and configure MediationNativeAdUnit
{:.no_toc}

The `MediationNativeAdUnit` is part of the Prebid mediation API. This class is responsible for making a bid request and providing a winning bid and targeting keywords to the adapters. For better targeting you should provide additional properties like `contextType` and `placementType`.

#### Step 4: Set up assets for bid request
{:.no_toc}

The bid request for native ads should have the description of any expected assets. The full spec for the native template can be found in the [Native Ad Specification from IAB](https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf).

Example of creating the assets array:

```swift
let image = NativeAssetImage(minimumWidth: 200, minimumHeight: 50, required: true)
image.type = ImageAsset.Main

let icon = NativeAssetImage(minimumWidth: 20, minimumHeight: 20, required: true)
icon.type = ImageAsset.Icon

let title = NativeAssetTitle(length: 90, required: true)

let body = NativeAssetData(type: DataAsset.description, required: true)

let cta = NativeAssetData(type: DataAsset.ctatext, required: true)

let sponsored = NativeAssetData(type: DataAsset.sponsored, required: true)

return [icon, title, image, body, cta, sponsored]
```

#### Step 5: Set up event tracker for bid request
{:.no_toc}

The bid request for mative ads may have a description of expected event trackers. The full spec for the Native template can be found in the [Native Ad Specification from IAB](https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf).

The example of creating the event trackers array:

```swift
let eventTrackers = [
    NativeEventTracker(event: EventType.Impression, methods: [EventTracking.Image,EventTracking.js])
]
```

#### Step 6: Make a bid request
{:.no_toc}

The `fetchDemand` method makes a bid request to Prebid server and provides a result in a completion handler.

#### Step 7: Load AdMob Native ad
{:.no_toc}

Now just load a native ad from AdMob according to the [AdMob instructions](https://developers.google.com/admob/ios/native/start).

## Additional Ad Unit Configuration

{% include mobile/rendering-adunit-config-ios.md %}

## Further Reading

- [Prebid Mobile Overview](/prebid-mobile/prebid-mobile)
- [Prebid SDK iOS Integration](/prebid-mobile/pbm-api/ios/code-integration-ios)
- [Prebid SDK iOS Global Parameters](/prebid-mobile/pbm-api/ios/pbm-targeting-ios)
