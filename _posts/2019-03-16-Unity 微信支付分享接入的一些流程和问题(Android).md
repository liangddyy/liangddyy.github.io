---
layout: post
title: Unity 微信支付和分享接入的一些流程和问题(Android)
categories: [Unity, SDK]
description: 微信支付接入
keywords: Unity,Android,SDK
---

### 使用

用As新建一个Module，在Gradle 的 dependencies 里 添加`api 'com.tencent.mm.opensdk:wechat-sdk-android-without-mta:+'`引入依赖库。不过建议单独下载jar文件引入 如：`compile files('libs/wechat-sdk-android-without-mta-5.1.6.jar')`

代码在最后面

### 遇到的一些问题

* 闪退

  需要在 Manifest.xml 的主Activity 添加这两行

  ```
  <!-- 需要添加以下两条配置-->
  <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
  <meta-data android:name="unityplayer.ForwardNativeEventsToDalvik" android:value="false" />
  ```

  参考：[unity 接入微信分享SDK总结](https://my.oschina.net/u/698044/blog/1828997)

* 错误码 “-6” 问题

  主要因为签名错误（可以核对签名MD5值 快速检查）。同时，更换签名后，可能需要卸载掉微信重装才可以彻底解决，这点很重要。

  参考：[Android集成微信分享功能应用签名生成方法及分享不生效的问题](https://blog.csdn.net/mpegfour/article/details/78293585)

  [微信第三方登录问题，及-6错误](https://blog.csdn.net/Friday_577/article/details/78971573)

* “由于不支持的分享类型，无法分享到微信”

  百分之百 APPKEY 错误。

### 支付分享等接口 Unity 用

支付，分享网址、小程序、图片等

```
public class WechatUtility {

    public static IWXAPI WXApi;

    public void RegisterApp(Context context, String appid) {
        WXApi = WXAPIFactory.createWXAPI(context, appid, true);
        WXApi.registerApp(appid);
    }

    public boolean IsWXInstalled() {
        return WXApi.isWXAppInstalled();
    }

    /**
     * 微信登录
     * @param message
     */
    public void WechatLogin(String message) {
        final SendAuth.Req req = new SendAuth.Req();
        req.scope = "snsapi_userinfo";
        req.state = message;
        WXApi.sendReq(req);
    }

    /**
     *  网页类型分享(不含图片）
     * @param url
     * @param title
     * @param des
     * @param isTimeLine
     */
    public void SharePageURL(String url, String title, String des, boolean isTimeLine) {
        SharePageURL(url, title, des, "", isTimeLine);
    }

    /**
     * 网页类型分享(包含图片）
     * @param url
     * @param title
     * @param des
     * @param imageUrl
     * @param isTimeLine
     */
    public void SharePageURL(String url, String title, String des, String imageUrl, boolean isTimeLine) {
        Log.d("unity", "分享:  " + isTimeLine);
        WXWebpageObject webPage = new WXWebpageObject();
        webPage.webpageUrl = url;
        WXMediaMessage msg = new WXMediaMessage(webPage);
        msg.title = title;
        msg.description = des;
        if (imageUrl != null && imageUrl != "") {
            Bitmap tmp = WXUtil.getLocalOrNetBitmap(imageUrl);
            Log.d("图片", "SharePageURL获取图片: " + (tmp == null));
            try {
                if (tmp != null) {
                    Bitmap thumb = Bitmap.createScaledBitmap(tmp, 120, 120, true);//压缩Bitmap
                    msg.thumbData = WXUtil.bmpToByteArray(thumb, true);
                }
            } catch (Exception e) {
                Log.d("unityWechat", "SharePageURL: 分享图片压缩出错。" + e.getMessage());
            }
        }

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("sharePageUrl"); // 唯一请求标识
        req.message = msg;

        // 朋友圈 或者 好友
        if (isTimeLine) {
            req.scene = SendMessageToWX.Req.WXSceneTimeline;
        } else {
            req.scene = SendMessageToWX.Req.WXSceneSession;
        }
        WXApi.sendReq(req);
    }

    /**
     * 微信支付
     * @param appId
     * @param partnerId
     * @param prepayId
     * @param packageValue
     * @param nonceStr
     * @param timeStamp
     * @param sign
     */
    public void Pay(String appId,String partnerId,String prepayId,String packageValue,String nonceStr,String timeStamp,String sign) {
        PayReq payRequest = new PayReq();
        payRequest.appId = appId;
        payRequest.partnerId = partnerId;
        payRequest.prepayId = prepayId;
        payRequest.packageValue = packageValue;
        payRequest.nonceStr = nonceStr;
        payRequest.timeStamp = timeStamp;
        payRequest.sign = sign;

        WXApi.sendReq(payRequest);
    }

    /**
     * 分享小程序
     * @param webPageUrl 兼容低版本的网页链接
     * @param type WXMiniProgramObject.MINIPTOGRAM_TYPE_RELEASE;// 正式版:0，测试版:1，体验版:2
     * @param id 小程序原始ID
     * @param path 小程序页面路径
     * @param title 小程序消息title
     * @param des 小程序消息desc
     * @param imageUrl 小程序消息封面图片URL，最终图片需要小于128k(这张图是大图，不用太压缩。不然会糊掉）
     */
    public void ShareMiniProgram(String webPageUrl,int type, String id,String path,String title,String des,String imageUrl) {
        WXMiniProgramObject miniProgramObj = new WXMiniProgramObject();
        miniProgramObj.webpageUrl = webPageUrl;
        miniProgramObj.miniprogramType = type;
        miniProgramObj.userName = id;
        miniProgramObj.path = path;
        WXMediaMessage msg = new WXMediaMessage(miniProgramObj);
        msg.title = title;
        msg.description = des;

        if (imageUrl != null && imageUrl != "") {
            Bitmap tmp = WXUtil.getLocalOrNetBitmap(imageUrl);
            Log.d("图片", "SharePageURL获取图片: " + (tmp == null));
            try {
                if (tmp != null) {
                    msg.thumbData = WXUtil.bmpToByteArrayNoCmpress(tmp, true);
                }
            } catch (Exception e) {
                Log.d("unityWechat", "SharePageURL: 分享图片压缩出错。" + e.getMessage());
            }
        }

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("miniProgram");
        req.message = msg;
        req.scene = SendMessageToWX.Req.WXSceneSession;  // 目前只支持会话
        WXApi.sendReq(req);
    }

    /**
     * 分享图片
     * @param bytes 图片
     * @param isTimeLine 分享至朋友圈还是好友
     */
    public void ShareImage(byte[] bytes,boolean isTimeLine){

        Bitmap bmp = WXUtil.Bytes2Bimap(bytes);
//初始化 WXImageObject 和 WXMediaMessage 对象
        WXImageObject imgObj = new WXImageObject(bmp);
        WXMediaMessage msg = new WXMediaMessage();
        msg.mediaObject = imgObj;

//设置缩略图
        Bitmap thumbBmp = Bitmap.createScaledBitmap(bmp, 600, 300, true);
        bmp.recycle();
        msg.thumbData = WXUtil.bmpToByteArray(thumbBmp, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("img");
        req.message = msg;
        // 朋友圈 或者 会话
        if(isTimeLine)
            req.scene = SendMessageToWX.Req.WXSceneTimeline;
        else
            req.scene = SendMessageToWX.Req.WXSceneSession;
//        req.userOpenId = getOpenId();

        WXApi.sendReq(req);
    }
}

```

### 其他脚本

同步下载图片、压缩图片等（微信的缩略图 分享图等有大小限制）

```
public class WXUtil {

    /**
     * 把网络资源图片转化成bitmap (一元购Web端那边给过来的是URL，垃圾微信不支持直接给URL，需要处理下使用）
     */
    public static Bitmap getLocalOrNetBitmap(String url) {
        Bitmap bitmap;
        InputStream in;
        BufferedOutputStream out;
        try {
            in = new BufferedInputStream(new URL(url).openStream(), 1024);
            final ByteArrayOutputStream dataStream = new ByteArrayOutputStream();
            out = new BufferedOutputStream(dataStream, 1024);
            copy(in, out);
            out.flush();
            byte[] data = dataStream.toByteArray();
            bitmap = BitmapFactory.decodeByteArray(data, 0, data.length);
            data = null;
            return bitmap;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    private static void copy(InputStream in, OutputStream out)
            throws IOException {
        byte[] b = new byte[1024];
        int read;
        while ((read = in.read(b)) != -1) {
            out.write(b, 0, read);
        }
    }

    public static byte[] bmpToByteArray(final Bitmap bmp, final boolean needRecycle) {
        int i;
        int j;
        if (bmp.getHeight() > bmp.getWidth()) {
            i = bmp.getWidth();
            j = bmp.getWidth();
        } else {
            i = bmp.getHeight();
            j = bmp.getHeight();
        }

        Bitmap localBitmap = Bitmap.createBitmap(i, j, Bitmap.Config.RGB_565);
        Canvas localCanvas = new Canvas(localBitmap);

        while (true) {
            localCanvas.drawBitmap(bmp, new Rect(0, 0, i, j), new Rect(0, 0, i, j), null);
            if (needRecycle)
                bmp.recycle();
            ByteArrayOutputStream localByteArrayOutputStream = new ByteArrayOutputStream();
            localBitmap.compress(Bitmap.CompressFormat.JPEG, 20,
                    localByteArrayOutputStream);
            localBitmap.recycle();
            byte[] arrayOfByte = localByteArrayOutputStream.toByteArray();
            try {
                localByteArrayOutputStream.close();
                return arrayOfByte;
            } catch (Exception e) {
                // F.out(e);
            }
            i = bmp.getHeight();
            j = bmp.getHeight();
        }
    }
    public static byte[] bmpToByteArrayNoCmpress(final Bitmap bmp, final boolean needRecycle) {
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        bmp.compress(Bitmap.CompressFormat.PNG, 100, output);
        if (needRecycle) {
            bmp.recycle();
        }

        byte[] result = output.toByteArray();
        try {
            output.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return result;
    }
    public static String buildTransaction(final String type) {
        return (type == null) ? String.valueOf(System.currentTimeMillis()) : type + System.currentTimeMillis();
    }

    public static Bitmap Bytes2Bimap(byte[] b) {
        if (b.length != 0) {
            return BitmapFactory.decodeByteArray(b, 0, b.length);
        } else {
            return null;
        }
    }
}
```

### 支付或分享回调

分享

```
public class WXEntryActivity extends Activity implements IWXAPIEventHandler {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        WechatUtility.WXApi.handleIntent(getIntent(), this);
    }

    @Override
    public void onReq(BaseReq baseReq) {
    }

    @Override
    public void onResp(BaseResp baseResp) {
        Log.d("UnityWechat", "onResp微信回调结果: " + baseResp.getType());
        switch (baseResp.getType()) {
            case ConstantsAPI.COMMAND_SENDAUTH: // 登录
                handLogin(baseResp);
                break;
            case ConstantsAPI.COMMAND_SENDMESSAGE_TO_WX:  // 分享
                handShare(baseResp);
                break;
            default:
                break;
        }
        finish();
    }


    private void handShare(BaseResp baseResp) {
        String result;
        switch (baseResp.errCode) {
            case 0:
                result = "分享成功";
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.ShareSuccessFunc, "");
                break;
            case -2:
                result = "取消分享";
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.ShareFailFunc, result);
                break;
            case -3:
                result = "分享失败";
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.ShareFailFunc, result);
                break;
            case -1:
            default:
                result = "其他原因 " + baseResp.errCode + " ErrorSting:" + baseResp.errStr;
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.ShareFailFunc, result);
                break;
        }
        Log.d("unity", "onResp分享结果：" + result + baseResp.errCode + "string:" + baseResp.errStr);
    }

    private void handLogin(BaseResp baseResp) {
        Log.d("UnityWechat", "handLogin:" + baseResp.errCode);
        switch (baseResp.errCode) {
            case BaseResp.ErrCode.ERR_OK:
                if (baseResp instanceof SendAuth.Resp) {
                    SendAuth.Resp newResp = (SendAuth.Resp) baseResp;

                    JSONObject json = new JSONObject();
                    try {
                        json.put("code", newResp.code);
                        json.put("state", newResp.state);
                        json.put("lang", newResp.lang);
                        json.put("country", newResp.country);
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                    UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, "LoginSuccess", json.toString());
                }
                break;
            case BaseResp.ErrCode.ERR_USER_CANCEL:
                //发送取消
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, "LoginFail", "取消微信登录");
                break;
            case BaseResp.ErrCode.ERR_AUTH_DENIED:
                //发送被拒绝
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, "LoginFail", "拒绝微信登录");
                break;
            default:
                //发送返回
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, "LoginFail", "微信登录错误:" + baseResp.errCode);
                Log.e("unityWechat", "微信登录错误: " + baseResp.errStr);
                break;
        }
    }

    @Override
    public void onPointerCaptureChanged(boolean hasCapture) {

    }
}
```

支付

```
public class WXPayEntryActivity extends Activity implements IWXAPIEventHandler {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        WechatUtility.WXApi.handleIntent(getIntent(), this);
    }

    @Override
    public void onReq(BaseReq baseReq) {
    }

    @Override
    public void onResp(BaseResp baseResp) {
        Log.e("UnityWechat ","进入支付回调页onResp "+baseResp.getType() +" 支付code "+baseResp.errCode);
        if (baseResp.getType() == ConstantsAPI.COMMAND_PAY_BY_WX) {
            handPay(baseResp);
        }
        finish();
    }


    private void handPay(BaseResp baseResp) {
        switch (baseResp.errCode)
        {
            case 0:
                Log.d("UnityWechat","支付完成");
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.PaySuccessFunc, "");
                break;
            case -1:
                Log.d("UnityWechat","支付失败");
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.PayFailFunc, "支付失败");
                break;
            case -2:
                Log.d("UnityWechat","取消支付");
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.PayFailFunc, "取消支付");
                break;
            default:
                String msg = "code:" + baseResp.errCode + "msg:" + baseResp.errStr;
                Log.d("UnityWechat",msg);
                UnityPlayer.UnitySendMessage(UnityWechatAPI.GameObjectHander, UnityWechatAPI.PayFailFunc, msg);
                    break;
        }
    }
}
```

