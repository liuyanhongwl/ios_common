## 显示苹果的广告

```
ADBannerView *bannerView = [[ADBannerView alloc] initWithAdType:ADAdTypeBanner];
bannerView.delegate = self;
[self.view addSubview:bannerView];
   
self.canDisplayBannerAds = YES;

```