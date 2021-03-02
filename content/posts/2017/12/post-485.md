---
title: '[Unity] iOS申請時のMissing Push Notificationを対処する'
author: しゃまとん
date: 2017-12-07T15:21:29+00:00
url: /posts/485
featured_image: /images/posts/2017/12/apple_logo_PNG19689.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

UnityでiOS向けにPush通知を利用していない状態でbuildして何も気にせずアプリ申請
（ここではXcode等からArchiveしてUploadすること）を行うとAppleからアップロードしたファイルに関しての通知が来ます。

問題なければ、完了した旨のメールが来るのみですが、下記のような内容です。  
ちなみに確認したバージョンは`5.6.0f3`なので、他のバージョンでは対応が異なる可能性があります。

```text
Dear developer,

We have discovered one or more issues with your recent delivery for "アプリ名". Your delivery was successful, but you may wish to correct the following issues in your next delivery:

Missing Push Notification Entitlement - Your app appears to register with the Apple Push Notification service, but the app signature's entitlements do not include the "aps-environment" entitlement. If your app uses the Apple Push Notification service, make sure your App ID is enabled for Push Notification in the Provisioning Portal, and resubmit after signing your app with a Distribution provisioning profile that includes the "aps-environment" entitlement. Xcode 8 does not automatically copy the aps-environment entitlement from provisioning profiles at build time. This behavior is intentional. To use this entitlement, either enable Push Notifications in the project editor's Capabilities pane, or manually add the entitlement to your entitlements file. For more information, see https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1.

After you’ve corrected the issues, you can use Xcode or Application Loader to upload a new binary to iTunes Connect.

Regards,

The App Store team
```

どうやらプッシュ通知に関しての設定が有効になっているけど使うための証明書が見つからないそうで、
直してくださいという内容みたいです。

Unity側のBuild設定でPush通知を無効にする対応できればいいのですが、
なさそうなので使わない場合は該当の箇所に対して別途修正をする必要がありました。

UnityでiOS向けのBuildを行い、完了したらXcodeを実行し、
該当の箇所（Classes/UnityAppContoller.mm）をコメントアウトします。

```objectivec
#if UNITY_USES_REMOTE_NOTIFICATIONS
/*
- (void)application:(UIApplication*)application didReceiveRemoteNotification:(NSDictionary*)userInfo
{
    AppController_SendNotificationWithArg(kUnityDidReceiveRemoteNotification, userInfo);
    UnitySendRemoteNotification(userInfo);
}

- (void)application:(UIApplication*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken
{
    AppController_SendNotificationWithArg(kUnityDidRegisterForRemoteNotificationsWithDeviceToken, deviceToken);
    UnitySendDeviceToken(deviceToken);
}
*/
#if !UNITY_TVOS
/*
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))handler
{
    AppController_SendNotificationWithArg(kUnityDidReceiveRemoteNotification, userInfo);
    UnitySendRemoteNotification(userInfo);
    if (handler)
    {
        handler(UIBackgroundFetchResultNoData);
    }
}
*/
#endif
/*
- (void)application:(UIApplication*)application didFailToRegisterForRemoteNotificationsWithError:(NSError*)error
{
    AppController_SendNotificationWithArg(kUnityDidFailToRegisterForRemoteNotificationsWithError, error);
    UnitySendRemoteNotificationError(error);
}
*/
#endif
```

これでArchiveしてアップロードすると先程の通知なくバイナリの配置をすることができました。  
以上です。

■ 参考
{{< blogcard url="https://blog.77jp.net/missing-push-notification-entitlement-ios">}}
