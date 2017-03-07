![AdColony Logo and Title](assets/logo-title.png)

##基本情報##
###AdColony SDK 取得URL###

* AdColony-Android-SDK 
https://github.com/AdColony/AdColony-Android-SDK-3


***
AdColonyはアプリケーションのあらゆる場所にHD動画広告を配信することができます。動画を再生完了した時点でユーザに仮想通貨を付与する動画リワード広告も提供しています。

###注意###
* AdColony Android SDKの最新のバージョンは3.1.0です。
* 本SDKはAndroid OS 4.0(APIレベル14)から動作対象となります。
* GoogleのAdvertising IDを取得するため、プロジェクトの中にGoogle Play Services 10.0.1 を追加してください。追加しない場合表示できる広告の数は少なくなります。

***
###Contents###
* [Project Setup](#project-setup)
* [Showing Videos Ads](#showing-videos-ads)
    * [Showing Interstitial Ads](#showing-interstitial-ads)
    * [Showing Rewarded Interstitial Ads](#showing-rewarded-interstitial-ads)
* [APIリファレンス](https://adcolony-www-common.s3.amazonaws.com/Javadoc/3.0.4/index.html)
* [よくある質問](#よくある質問)
    * [基本情報に関して](#基本情報に関して)
    * [SDK仕様に関して](#sdk仕様に関して)
    * [動画再生に関して](#動画再生に関して)
    * [ストア申請に関して](#ストア申請に関して)

##Project Setup##
下記のステップに従って、AdColony SDKをアプリに導入してください。

####Step 1: Adcolonyアカウント情報の取得####

アカウント情報の取得は、以下の通りになります。

1.Glossomにてapp ID、zoneIDを発行しお渡しします。

2.広告を表示するコードを実装して下さい。


####Step 2: AdColonyライブラリを読み込む####
２つの方法がありますので、各環境に合わせて選択してください。

#####1. build.gradleでレポジトリからライブラリを読み込む場合#####


AdColonyを使うために必要なMavenのレポジトリの設定を、プロジェクトのbuild.gradleに下記のように記述してください。

```
/** 
 * Make sure you are configuring your project or module's 'repositories' configuration
 * block and not your buildscript's 'repositories' configuration block
 */
repositories {
    /** The default repository for Android Studio projects */
    jcenter() 

    /** The repository required for AdColony 3.0 and above */
    maven {
      url  "https://adcolony.bintray.com/AdColony"      
    }
}
```

また、モジュールのbuild.gradleにも下記の設定を記述してください。
```
dependencies {
  /** 
   * Any other dependencies your module has are placed in this dependency configuration
   */
  compile 'com.adcolony:sdk:3.1.0'
  compile 'com.google.android.gms:play-services-ads:10.0.1'
  compile 'com.android.support:support-annotations:25.0.1'
}

```

#####2. jarファイルを手動でimportする場合#####

1. [リンク](https://github.com/AdColony/AdColony-Android-SDK-3/tree/master/Library)からSDKライブラリファイルをダウンロードしてください。
2. adcolony.jarをプロジェクトのlibsフォルダに移動してください。
3. armeabi, armeabi-v7a, arm64-v8a, x86_64, and x86フォルダの中で必要なものをプロジェクトのlibsフォルダに移動してください。
4. モジュールのbuild.gradleのdependenciesとandroidのプロパティの部分に、下記の設定を追加してください。
```
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])

  /** Any other dependencies here */
}

android {
  /** Any other configurations here */

  sourceSets {
    main {
      jniLibs.srcDirs = ['libs']
    }
  }
}

```


***
####Step 3: AndroidManifest.xmlの修正####
"AndroidManifest.xml"に下記のパーミッションを追加してください。

1. INTERNET
2. ACCESS_NETWORK_STATE
3. WRITE_EXTERNAL_STORAGE (optional)
4. VIBRATE (optional)
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.VIBRATE" /> 
```

次に、下記のAcitivityの設定を<application>タグの間にコピーしてください。
```xml
<activity android:name="com.adcolony.sdk.AdColonyInterstitialActivity"
android:configChanges="keyboardHidden|orientation|screenSize"
android:hardwareAccelerated="true"/>

<activity android:name="com.adcolony.sdk.AdColonyAdViewActivity"
android:configChanges="keyboardHidden|orientation|screenSize"
android:hardwareAccelerated="true"/>
```


***

####Step 4: Google Play Services Ads Libraryに関して####
[Googles developer program policies](https://play.google.com/intl/ja/about/developer-content-policy/)に則って、
アプリが広告IDを取得するために、[Google Play Services SDK](https://developers.google.com/android/guides/setup)のAds Libraryのクラスにアクセスする必要があります。



####Step 5: Pro Guardの設定####
下記のPro Guardの設定を追記してください。

```
# For communication with AdColony's WebView
-keepclassmembers class * { 
    @android.webkit.JavascriptInterface <methods>; 
}
```
***

##Showing Videos Ads##

##Showing Interstitial Ads##
AdColonyインタースティシャルビデオは動画に続いてエンドカードが表示される動画広告です。

###Instructions###
[Project Setup](#project-setup)を設定した後に、以下のステップでインタースティシャル広告を表示することができます。
####Step 1: Adcolonyの設定####
Adcolonyライブラリをインポートしてください。
```java
import com.adcolony.sdk.*;
```
次に、以下のようにconfigureメソッドを**main Activity** の onCreateメソッドの中で呼び出してください。
```java
@Override
protected void onCreate( Bundle bundle )
{
  super.onCreate( bundle );
  AdColony.configure( this, APP_ID, ZONE_IDS );
  ...
}
```
**Note:** この設定関数は**main Activity's** onCreateメソッドで **一回** 実行してください。 また、アプリケーションが実行する間このAcitivityの参照は消さないようにしてください。

===
####Step 2: 動画広告の再生####
手順の概要としまして、

1. 再生させたいzoneIDのadの情報をリクエスト
2. 取得したadのインスタンスのshowメソッドを利用することで、動画を再生する

となります。

#####AdColonyInterstitialListenerの作成#####
AdColonyInterstitialListenerには、adの状態に応じて呼ばれるコールバックを定義することができます。
onRequestFilledはadのリクエストが成功したときに呼ばれるコールバックです。
listenerの詳細は[API Details](https://adcolony-www-common.s3.amazonaws.com/Javadoc/3.0.4/index.html)を参照してください。

```java
AdColonyInterstitialListener listener = new AdColonyInterstitialListener() 
{
    @Override
    public void onRequestFilled( AdColonyInterstitial ad )
    {
        /** Store and use this ad object to show your ad when appropriate */
    }
};
```

####adのリクエスト####
adのリクエストはrequestInterstitialのメソッドを使います。(第一引数は表示させたいadのzoneID、第二引数は上で定義したAdColonyInterstitialListener)

```java
AdColony.requestInterstitial( ZONE_ID, listener );
```

ZONE_IDは、登録したInterstitial用のIDかつAdColony.configureの引数にパラメータとして渡しておく必要があります。

####adの再生####
requestInterstitialadにより、指定したzoneIDのadのリクエストが成功した場合
onRequestFilledが呼ばれ、その際に再生可能なadオブジェクトが引数として渡されます。
このadオブジェクトのインスタンスメソッドのshowをcallすれば、adの再生を行うことが可能です。

```java
ad.show();
```


**Note:** また、これは基本の実装方法です。さらに詳細を確認していただくには、[インタースティシャル広告のサンプルアプリ](https://github.com/AdColony/AdColony-Android-SDK-3/tree/master/Demos/InterstitialDemo)と[API Details](https://adcolony-www-common.s3.amazonaws.com/Javadoc/3.0.4/index.html) を参照してください。

##Showing Rewarded Interstitial Ads##
AdColony Rewarded Interstitial Adsは<br>
[Showing Interstitial Ads](#showing-interstitial-ads)の上で実装した動画広告を再生完了した時点で、
ユーザにインセンティブ（仮想通貨やユーザー体験）を付与することができるシステムです。

[Basics](#basics)<br>
[Advanced Usage](#advanced-usage)
###Basics###
基本的な動画の再生方法に関しましては、インタースティシャル動画広告と同じです。
####Step 1: Adcolonyの設定####
Adcolonyライブラリをインポートしてください。
```java
import com.adcolony.sdk.*;
```
次に、以下のようにconfigureメソッドを**main Activity** の onCreateメソッドの中で呼び出してください。
```java
@Override
protected void onCreate( Bundle bundle )
{
  super.onCreate( bundle );
  AdColony.configure( this, APP_ID, ZONE_IDS );
  ...
}
```
**Note:** この設定関数は**main Activity's** onCreateメソッドで **一回** 実行してください。 また、アプリケーションが実行する間このAcitivityの参照は消さないようにしてください。

===
####Step 2: 動画広告の再生####
手順の概要としまして、

1. 再生させたいzoneIDのadの情報をリクエスト
2. 取得したadのインスタンスのshowメソッドを利用することで、動画を再生する

となります。

#####AdColonyInterstitialListenerの作成#####
AdColonyInterstitialListenerには、adの状態に応じて呼ばれるコールバックを定義することができます。
onRequestFilledはadのリクエストが成功したときに呼ばれるコールバックです。
listenerの詳細は[API Details](https://adcolony-www-common.s3.amazonaws.com/Javadoc/3.0.4/index.html)を参照してください。

```java
AdColonyInterstitialListener listener = new AdColonyInterstitialListener() 
{
    @Override
    public void onRequestFilled( AdColonyInterstitial ad )
    {
        /** Store and use this ad object to show your ad when appropriate */
    }
};
```

####adのリクエスト####
adのリクエストはrequestInterstitialのメソッドを使います。(第一引数は表示させたいadのzoneID、第二引数は上で定義したAdColonyInterstitialListener)

```java
AdColony.requestInterstitial( ZONE_ID, listener );
```

ZONE_IDは、登録したReward用のIDかつAdColony.configureの引数にパラメータとして渡しておく必要があります。



####adの再生####
requestInterstitialadにより、指定したzoneIDのadのリクエストが成功した場合
onRequestFilledが呼ばれ、その際に再生可能なadオブジェクトが引数として渡されます。
このadオブジェクトのインスタンスメソッドのshowをcallすれば、adの再生を行うことが可能です。

```java
ad.show();
```

===
###Advanced Usage###
[AdColonyRewardListener](#adcolonyrewardlistener)<br>
[Pre and Post-Popups](#pre-and-post-popups)<br>
[Rewarded App Options](#rewarded-app-options)<br>
[Server-Side Rewards](#server-side-rewards)
####AdColonyRewardListener####
Reward動画を再生した後にAdcolonyからアプリに通知します。AdColony.configureの後に追加してください。
```java
AdColonyRewardListener listener = new AdColonyRewardListener()
{
    @Override
    public void onReward( AdColonyReward reward )
    {
        /** Query the reward object for information here */
        if (reward.success()) {
            //Reward user
        } else {
        }
    }
};

/** Set reward listener for your app to be alerted of reward events */
AdColony.setRewardListener( listener );
```
===
####Pre and Post-Popups####
adのrequestの際に、下記のようにoptionsをパラメータとして渡してあげることで、ユーザに動画再生する前の確認メッセージ(Pre)及び成果通知の確認メッセージ(Post)を、ポップアップダイアログにて表示することが可能です。

事前の確認メッセージを表示する場合
```java
AdColonyAdOptions options = new AdColonyAdOptions()
    .enableConfirmationDialog( true )
AdColony.requestInterstitial( ZONE_ID, listener, options );
```
事後の確認 メッセージを表示する場合
```java
AdColonyAdOptions options = new AdColonyAdOptions()
    .enableResultsDialog( true )
AdColony.requestInterstitial( ZONE_ID, listener, options );
```
両方を表示する場合
```java
AdColonyAdOptions options = new AdColonyAdOptions()
    .enableConfirmationDialog( true )
    .enableResultsDialog( true );

AdColony.requestInterstitial( ZONE_ID, listener, options );
```

**Note:** さらに詳細を確認していただくには、[リワードインタースティシャル広告のサンプルアプリ](https://github.com/AdColony/AdColony-Android-SDK-3/tree/master/Demos/RewardedInterstitialDemo)と[API Details](https://adcolony-www-common.s3.amazonaws.com/Javadoc/3.0.4/index.html) を参照してください。


===

####Rewarded App Options####
以下の処理のように、configure時にAdColonyAppOptionsを渡してあげることで、サーバ上で仮想通貨の処理を行う場合に、ユーザIDをパラメータとして渡すことが可能になります。

```java
AdColonyAppOptions appOptions = new AdColonyAppOptions()
    .setUserID("userid");

/** Pass options with user id set with configure */
AdColony.configure(this, appOptions, APP_ID, ZONE_ID);

```

もし、アプリの起動中にユーザIDが変更された場合(別のユーザIDでログインし直した場合など)、以下の処理を記述すればそのタイミングでサーバにパラメータとして送るユーザIDも更新することが可能です。

```java
/** Get current AdColonyAppOptions and change user id */
AdColonyAppOptions appOptions = AdColony.getAppOptions()
    .setUserID("newuserid");

/** Send new information to AdColony */
AdColony.setAppOptions(appOptions);

```


####Server-Side Rewards####
仮想通貨の付与の際に、セキュリティを上げるため、ハッシングメッセージを使用して、開発者のサーバーへ仮想通貨を処理するコールバックを提供しています。この機能を利用するためには、サーバー側でゲームのユーザに仮想通貨を付与するコールバックURLを実装する必要があります。AdcolonyからこのURLに必要なパラメーターを追加して御社のサーバーにポストバックします。そのリクエストを受けてユーザに仮想通貨を付与してください。 <br><br>
Adcolonyはサーバー側がなくてもクライアント側のみで仮想通貨を付与することができますが、この方法は不正を完全に防げるのが困難なため推奨致しません。システム管理上のサーバーを使用できない場合、video-ad@glossom.co.jpに問い合わせて下さい。

####Step 1####
コールバック受信のために、サーバーのURLを作成してください。このURLには認証をかけないでください。コールバックURLは、申し込み書に記載のうえGlossomにご連絡してください。

===
####Step 2####
Adcolonyからサーバーへのリクエストがトランザクションに基づいた適切なものであることを確認してください。リクエストURLのフォーマットは以下になります。ただし括弧内のものはアプリやトラザクションによって動的に値が変わります。
```
[http://www.yourserver.com/anypath/callback_url.php]?id=[ID]&uid=[USER_ID]&zone=[ZONE_ID]&amount=[CURRENCY_AMOUNT]&currency=[CURRENCY_TYPE]&verifier=[HASH]&open_udid=[OPEN_UDID]&udid=[UDID]&odin1=[ODIN1]&mac_sha1=[MAC_SHA1]
```
URL Parameter | Type | Purpose
--- | --- | ---
id | Positive long integer | Unique Reward transaction ID
uid | String | AdColony device ID
amount | Positive integer | Amount of currency to reward
currency | String | Name of currency to reward
open_udid | String | OpenUDID
udid | String | Apple UDID
odin1 | String | Open Device Identification Number (ODIN)
mac_sha1 | String | SHA-1 hash of lowercase colon-separated MAC address
custom_id | String | Custom user ID
verifier | String | MD5 hash for message security

Androidの場合open_udid, udid, odin1, and mac_sha1は常に空です。 アプリからカスタマイズIDを渡したい場合、zoneのコールバックURLの後ろに“&custom_id=[CUSTOM_ID]”をつけてください。このIDも文字(verifier)の最後に追加してチェックしてください。<br><br>

コールバックの実装は、MD5がサポートしている言語であるならば、PHPにかぎらず言語の指定はありません。<br><br>

参考例として、下記のコードではPHPとMySQLを使って、URLのパラメーターの処理とMD5でのチェック、トラザクションの重複チェックを行なってます。
```php
<?php

    $MY_SECRET_KEY="Thiscomesfromadcolony.com";

    $trans_id=mysql_real_escape_string($_GET['id']); 
    $dev_id=mysql_real_escape_string($_GET['uid']); 
    $amt=mysql_real_escape_string($_GET['amount']); 
    $currency=mysql_real_escape_string($_GET['currency']); 
    $verifier=mysql_real_escape_string($_GET['verifier']);

    //verify hash 
    $test_string="".$trans_id.$dev_id.$amt.$currency.$MY_SECRET_KEY; 
    $test_result=md5($test_string);
    if($test_result!=$verifier)
    {
      echo"vc_decline";
      die; 
    }

    //check for a valid user 
    $user_id=//get your internal user id from the device id here 
    if(!$user_id)
    {
      echo"vc_decline";
      die; 
    }

    //insert the new transaction 
    $query="INSERT INTO AdColony_Transactions(id,amount,name,user_id,time)".
      "VALUES($trans_id,$amt,'$currency',$user_id,UTC_TIMESTAMP())"; 
    $result=mysql_query($query);
    if(!$result)
    {
      //check for duplicate on insertion 
      if(mysql_errno()==1062)
      {
        echo"vc_success";
        die; 
      }
      //otherwise insert failed and AdColony should retry later 
      else
      {
        echo"mysql error number".mysql_errno();
        die; 
      }
    }
    //award the user the appropriate amount and type of currency here 
    echo"vc_success";

?>
```
上記コードで使ったMySQLデータベースは下記のように作成できます。
```mysql
CREATE TABLE `AdColony_Transactions` (
`id` bigint(20) NOT NULL default '0',
`amount` int(11) default NULL,
`name` enum('Currency Name 1') default NULL, `user_id` int(11) default NULL,
`time` timestamp NULL default NULL,
PRIMARY KEY (`id`)
);
```
重複付与しないために、トラザクションidを必ず保存してください。毎回コールバックURLが呼ばれる時にトラザクションIDが重複してるかどうかをチェックしてください。既に処理された場合、再度ユーザへ付与する必要はなくレスポンスが成功した場合と同様の対応を行って下さい<br><br>
重複チェックの後に指定したタイプ＆数量の仮想通貨をユーザに付与してください。

===
####Step 3####
サーバーのレスポンスのフォーマットに関して<br>
* "vc_success"
  * トランザクションが終了。コールバックが受信され、ユーザが入金した場合、この値を返します。
* "vc_decline" or "vc_noreward"
  * トランザクションが終了。user_idが有効でない場合か、セキュリティチェックが渡されていない場合、この値を返します。
* 上記以外
  * サーバーへの再試行。エラーがある場合に使用されます。

**Note:** ユーザに対して付与を行わなくて良い場合は、user_idが無効、セキュリティが通らない、トランザクションがすでに報告され重複した場合のみになりますので、これらの条件に当てはまらないユーザには必ず付与を行って下さい。


##よくある質問##
###基本情報に関して###
- Q:各設定情報はどんな意味ですか
- A:
	- **App ID**: こちらは各アプリを指します。
	- **Zone ID**: こちらはアプリの下に紐づく、各掲載枠を指します。
	- **Call Back URL**: 動画再生の成果等を送るURLになります。設定していただかなくても本サービスはご利用頂けます。
	- **Custom ID**: CustomIDは、mediaのユーザーidを設定していただきます。設定した値はcall backを使用する場合、custom_idとして返します。

###SDK仕様に関して###
- Q: 縦画面の再生は可能か？
- A: 可能となります。

- Q: AdColonyダイアログででポップアップの国別で表記を可能か？
- A: アプリの配信国に沿った各々の表記を行う場合、アプリ内でダイアログのご実装をご自身でして頂く必要がございます。

- Q: 上限回数は何回ですか？
- A: 上限回数は変更可能です。変更希望の場合、担当者もしくは(video-ad@glossom.co.jp)までご連絡下さい。

- Q: インセンティブはコイン以外でも可能か？
- A: 可能です。御社側で、動画視聴の成果通知に合わせてご実装して頂く必要がございます。

- Q: custom_idの設定する方法おしえてください。
- A: CustomIDの設定はSDK側でconfigure関数に、下記のパラメータを渡してください。

		AdColonyAppOptions app_options = new AdColonyAppOptions().setUserID( "xxxx" );
		AdColony.configure( this, app_options, APP_ID, ZONE_IDS );


- Q:  サーバー側のレスポンスと再送信
- A: 下記でございます。

		- "vc_success"
		正常に処理が終わり、ユーザーへのコイン付与が成功した場合、AdColony側から再送信を行いません。
	
		- "vc_decline" または "vc_noreward"
		正常に処理が終わったが、uidの誤り/不正と判断された場合、AdColony側から再送信を行いません。
	
		- [上記以外に、他にレスポンスされる場合]
		AdColonyは定期的に再送信行います。異常な場合以外は、こちら利用は控えて下さい。

###動画再生に関して###
- Q: 動画再生ができない場合どうすればいいですか
- A: 下記をチェックしてください。
	- 正しいIDは使われているか？
		- 発行されたApp ID/Zone IDの対象OSをお誤りのないよう、ご利用下さい。
	- IDと実装マニュアルの形式は同様か？
		- 動画リワードの場合[Showing Rewarded Interstitial Ads](#showing-rewarded-interstitial-ads)
		- interstisialの場合[Showing Interstitial Ads](#showing-interstitial-ads)をご利用下さい。
	- 広告がダウンロードされていない
		- 広告準備が確認された後、再生を行って下さい。
	- adが期限切れになっているか否か
		- ad.isExpired()で確認できます。
		- 期限切れの場合は、adを再度リクエストしてください。


- Q: 配信可能な広告がない（少ない）ですが、どうすればいいですか
- A: 広告在庫は様々なロジックで配信制限を行っております。詳細は下記を参考にして下さい。
	- ユーザーが不正と判断された場合
	- 1ユーザあたりの配信上限回数への到達
	- eCPMが極端に低い
	- ユーザー数が極端に少ない、リリース前もしくはリリース直後
	- androidの場合、GoogleのAdvertising IDを取得するため、プロジェクトの中にGoogle Play Services 9.4.0 を追加してください。

###ストア申請に関して###
- Q: テスト切り替えの際の連絡はいつしたら良いか？
- A: リリース前に担当者へ配信切り替えのご連絡をして下さい。
