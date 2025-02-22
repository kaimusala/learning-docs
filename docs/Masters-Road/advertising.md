# 广告

广告的详细介绍可以阅读：[商业化指南 | 产品手册 (ark.online)](https://docs.ark.online/CreatorPortal/Advertising.html)

本章节只介绍接入广告的步骤

## 1.播放广告的API

播放广告的API ：**AdsService.showAd()**

该API需要传入两个参数，第一个参数是传入播放广告的类型，第二个参数是传入播放广告的回调

**播放激励广告：**

> **激励广告**的含义：需要全部看完，偶尔快看完了的时候会出现跳过按钮，玩家的选择权更小，一般用作看广告获得奖励，建议放之前说明清楚，引导玩家全部看完，以提高广告收益。

```ts
AdsService.showAd(AdsType.Reward, (isSuccess: boolean) => {
    if (isSuccess) {
        // 广告播放成功
    } else {
        // 广告播放失败
    }
})
```

**播放插屏广告：**

> 插屏广告的含义：可以不全部看完，看几秒后会有关闭的按钮，玩家的选择权更大，不建议太频繁播放。

```ts
AdsService.showAd(AdsType.Interstitial, (isSuccess: boolean) => {
    if (isSuccess) {
        // 广告播放成功
    } else {
        // 广告播放失败
    }
})
```

## 2.在创作者中心接入广告

广告只能够在发布游戏之后才能进行播放测试，所以首先我们需要将游戏发布。

发布过后，来到创作者中心，进行如下操作：

> **⑴** 点击“游戏服务”
>
> **⑵** 点击“广告接入”
>
> **⑶** 选择关联激励广告或者关联插屏广告

![image-20231012135424893](https://arkimg.ark.online/image-20231012135424893.webp)

接着在弹出的窗口中，选择要关联的游戏，点击 “关联广告位” 即可

![image-20231012135842173](https://arkimg.ark.online/image-20231012135842173.webp)