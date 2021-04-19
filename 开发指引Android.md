# IM SDK for Android 开发指引

## 概述
游密即时通讯SDK（IM SDK）为玩家提供完整的游戏内互动服务，游戏开发者无需关注IM通讯复杂的内部工作流程，只需调用IM SDK提供的接口，即可快速实现世界聊天、公会聊天、组队聊天、文字、表情、语音等多项功能。

## IM SDK接口调用顺序
![IM SDK接口调用顺序](https://youme.im/doc/images/im_sdk_interface_call_order.png)

## 四步集成IM SDK
### 第一步 注册账号
在[游密官网](https://console.youme.im/user/register)注册游密账号。

### 第二步 添加游戏，获取Appkey
在控制台添加游戏，获得接入需要的Appkey、Appsecret。

### 第三步 下载IM SDK包体
[下载地址](https://www.youme.im/download.php?type=IM)

### 第四步 开发环境配置
[开发环境配置](#快速接入)

## 快速接入
### 1. 导入IM SDK
#### jar包导入

- 将jar包目录`im_android\lib`下的`yim-sdk.jar`，`yim.jar`，`alisr.jar`，`usc.jar`,`android-support-v4.jar`,`gson-2.3.1.jar` 包拷贝到Android工程的`project`->`libs`目录下

#### so库导入

- 将so库目录`im_android\lib\armeabi`下的`libnlscppsdk.so`，`libuscasr.so`，`libuuid.so`，`libyim.so`库拷贝到Android工程的`project`->`libs`->`armeabi`目录下。(选择所需架构下的so文件)

#### 配置依赖项
如果接入的是带google语音识别的版本。
需要在app的build.gradle中添加如下内容：

```
allprojects {
   repositories {
    //添加google语音识别需要的
    jcenter()
    google()
    //添加结束
   }
}
```

需要在Module的build.gradle的dependencies中配置一下依赖项。

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')

//添加google语音识别需要的
    compile "io.grpc:grpc-okhttp:1.4.0"
    compile "io.grpc:grpc-protobuf-lite:1.4.0"
    compile "io.grpc:grpc-stub:1.4.0"
    compile 'javax.annotation:javax.annotation-api:1.2'
//添加结束
}
```

#### Android权限配置

- 在AndroidManifest.xml中添加如下配置信息:

``` xml
  <!-- 添加到application节点内 -->
  <application xxxx>
      <receiver android:name="com.youme.im.NetworkStatusReceiver" android:label="NetworkConnection" >
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
      </receiver>
  </application>
  <!-- 添加到跟application平级 -->
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.RECORD_AUDIO" />
  <!-- 获取地理位置的权限，可选 -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```
#### 混淆配置
- 如果打包或生成APK时需要混淆，则需要在proguard.cfg文件中添加如下代码：



```
     -keep class com.youme.** {*;}
     -keep class com.iflytek.**
     -keepattributes Signature 
```

- 
### 2. 初始化
成功导入SDK后，应用启动的第一时间需要调用初始化接口。

#### import包名
`import com.youme.imsdk.YIMClient;`

这是一个单例类，可以通过`getInstance()`获取到它的实例。通过它就可以调用IM SDK的API接口。

#### 初始化IM SDK

- **调用示例：**

  ``` java
      YIMClient.getInstance().init(context, "strAppKey", "strSecrect", 0);
  ```
  
- **接口与参数：**
  - `初始化接口`：
    `public int init(Context context, String appKey, String secretKey,int serverZone)`
    
  - `context`：Android App上下文环境
  - `appKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://console.youme.im/)获取、查看
  - `secrectKey`：用户游戏产品的密钥，可以在[游密官网](https://console.youme.im/)获取、查看
  - `serverZone`：IM服务器区域，一般使用`0`-中国，其余设置值查看[服务器部署地区定义](IMSDKAndroid.php#服务器部署地区定义)
  
- **返回值：**
	- `YIMConstInfo.ErrorCode`：错误码，详细描述见[错误码定义](IMSDKAndroid.php#错误码定义)
	
- **备注：**
  此接口本是异步操作，未给出异步回调，一般返回的错误码是Success即表示初始化成功。
### 3. 设置回调监听 （以下流程是参考2.3.0.11036版本及之前的流程新版sdk的接口流程清查询API Guides）
IM引擎底层对于耗时操作都采用异步回调的方式，函数调用会立即返回，操作结果java层会同步回调。因此，用户必须实现相关接口并在初始化完成以后注册到YIMService。常用功能需要注册登录回调，聊天室回调，消息回调，如需其它功能查看API接口。

- **调用示例：**

  ``` java

      import com.youme.imsdk.YIMClient;
      ...
      
      public class MainActivity extends Activity {
        ...
        private YouMeIMCallback mCallback = new YouMeIMCallback(); 
       
        protected void onCreate(Bundle savedInstanceState) {
           // 初始化		     
		     YIMClient.getInstance().init(this, "strAppKey", "strSecrect", 0);		
		     
		     // 设置监听
		     		     YIMClient.getInstance().registerMessageListener(mCallback);
		     		    
		     super.onCreate(savedInstanceState);
		     setContentView(R.layout.activity_main);
		     
		     ...
		   }		    
	     ...
	     
	     // 回调接口实现
	     private class YouMeIMCallback implements MessageEventCallback {
     
		       /**
             * 功能：接收到用户发来的消息
             *
             * @param message 消息内容结构体
             */
            public void onRecvMessage(YIMMessage message)
            {
               if (null == message)
                  return;
               int msgType = message.getMessageType();
               if (YIMConstInfo.MessageBodyType.TXT == msgType)
               {
                  Log.i("YOUMEIM", "接收到一条文本消息： " + ((YIMMessageBodyText)message.getMessageBody()).getMessageContent()+",消息ID："+message.getMessageID());
               }
               else if (YIMConstInfo.MessageBodyType.Voice == msgType)
               {
                  YIMMessageBodyAudio audioMessage = (YIMMessageBodyAudio)message.getMessageBody();
                  int audioDuration = audioMessage.getAudioTime();
                  String param = audioMessage.getParam();//自定义附加参数
                  String txt = audioMessage.getText();//语音转文字识别结果
                  long mRecvAudioMsgId = message.getMessageID();

                  Log.i("YOUMEIM", "接收到一个录音文件 ／ AudioTime： " + audioDuration + " 录音文本信息：" + txt + " messID: " + mRecvAudioMsgId);
                  
                  //下载语音文件
                  ....
               }
            }
		      //其余回调接口实现
		      ...
		   }
		   ...   
	   }
  ```
  
- **接口：**
		  	  
	- `消息回调注册接口`：
	  `public void registerMsgEventCallback(YIMEventCallback.MessageEventCallback callback)`
	  
### 4. 登录IM系统
成功初始化IM后，调用IM SDK登录接口登录IM系统。

- **调用示例：**

  ``` java
      YIMClient.getInstance().login("1001", "12345","", new YIMEventCallback.ResultCallback<String>() {
            @Override
            public void onSuccess(String userID) {
                Log.i("YOUMEIM","登录成功,"+userID);
            });
  ```
  
- **接口与参数：**
	- `登录接口`：
	  `public void login(String userId, String password, String token, final YIMEventCallback.ResultCallback<String> callback)`
	  
	- `userId`：由调⽤者分配，不可为空字符串，只可由字母或数字或下划线组成，用户首次登录会自动注册所用的用户ID和密码
	- `password`：⽤户密码，不可为空字符串，由调用者分配，二次登录时需与首次登录一致，否则会报UsernamePasswordError
	- `token`：用户验证token，可选，如不使用token验证传入:""
	- `callback`：登录回调
   
### 5. 进入聊天频道
登录成功后，如果有群组聊天需要，比如游戏里面的世界、工会、区域等，需要进入聊天频道；调用方为聊天频道设置一个唯一的频道ID，可以进入多个频道。

- **调用示例：**

  ``` java
      // 可在登录成功的回调里面调用
      YIMClient.getInstance().joinChatRoom("1234", new YIMEventCallback.ResultCallback<ChatRoom>() {

                    @Override
                    public void onSuccess(ChatRoom info) {
                        Log.i("YOUMEIM","加入频道成功,"+info.groupId);
                    }
                );
  ```
  
- **接口与参数**
	- `进入频道接口`：
	  `public void joinChatRoom(String roomId, final YIMEventCallback.ResultCallback<ChatRoom> callback)`
	  
	- `roomId`：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节
	- `callback`：加入频道回调
   
### 6. 发送文本消息
在成功登录IM后，即可发送IM消息。

- **调用示例：**

  ``` java
      // 发送私聊文本消息
      YIMClient.getInstance().sendTextMessage("1001", YIMConstInfo.ChatType.PrivateChat, "mMsgContent", String attachParam, new YIMEventCallback.ResultCallback<SendMessage>(){
            @Override
            public void onSuccess(SendMessage info) {
                Log.i("YOUMEIM","发送文本消息 成功，requestID:"+info.requestId + ",sendTime:"+info.sendTime+", reasonType: "+info.reasonType+", isForbiddenRoom: "+info.isForbidRoom+", endTime: "+info.endTime);
            }

            @Override
            public void onFailed(int errorcode, SendMessage info) {
                Log.i("YOUMEIM","发送文本消息 失败，requestID:"+info.requestId + ",sendTime:"+info.sendTime+", reasonType: "+info.reasonType+", isForbiddenRoom: "+info.isForbidRoom+", endTime: "+info.endTime);
            }
        });
  ```
  
- **接口与参数：**
	- `发文本消息接口`：
     `public void sendTextMessage(String recvId, int chatType, String msgContent, YIMEventCallback.ResultCallback<SendMessage> callback)`
     
	- `recvId`：接收者ID,（⽤户ID或者频道ID）
	- `chatType`：聊天类型,私聊传`1`，频道聊天传`2`；需要频道聊天得先成功进入频道后发文本消息，此频道内的成员才能接收到消息
	- `msgContent`：聊天内容
	- `attachParam`： 发送文本附加信息
	- `callback`：发送文本消息回调

- **发送文本消息回调：**：

  `YIMEventCallback.ResultCallback<SendMessage>`：

``` java
    public interface ResultCallback<SendMessage> {
   /**
    * @param msgInfo 发送消息回调信息    
    */
    public void onSuccess(SendMessage msgInfo);
   /**
    * @param msgInfo 发送消息回调信息    
    */
    public void onFailed(int errorcode, SendMessage msgInfo);
	}

```

- 备注：
  `SendMessage`类成员方法如下：
  `getRequestId()`: 消息序列号，用于校验一条消息发送成功与否的标识，long类型  
  `getSendTime()`: 消息发送时间，int类型
  `getIsForbidRoom()`: 若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，boolean类型
  `getReasonType()`: 若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它，int类型
  `getEndTime()`: 若在频道被禁言，禁言结束时间，long类型   

- **拓展功能：**
  发送消息时，可以将玩家头像、昵称、角色等级、vip等级等要素打包成json格式发送。




### 7. 发送语音消息
在成功登录IM后，可以发送语音消息
>- 按住语音按钮时，调用`startRecordAudioMessage`接口，启动录音
>- 松开按钮，调用`stopAndSendAudioMessage`接口，发送语音消息
>- 按住过程若需要取消发送，调用`cancleAudioMessage`取消本次语音发送

- **调用示例：**


  ``` java
      // 启动语音      
      YIMClient.getInstance().startRecordAudioMessage("1001",1,"带文字识别",true,false);
      // 停止并发送语音消息
      YIMClient.getInstance().stopAndSendAudioMessage(new YIMEventCallback.AudioMsgEventCallback(){
                @Override
                public void onStartSendAudioMessage(Long requestID, int errorcode, String strText, String strAudioPath, int audioTime) {
                    Log.i("YOUMEIM","开始发送语音回调，errorcode：" + errorcode + ", requestID: "+requestID+"， text："+strText
                            + ", audioPath: "+strAudioPath+", audioTime: "+audioTime);
                }

                @Override
                public void onSendAudioMessageStatus(int errorcode, SendVoiceMsgInfo voiceMsgInfo) {
                    Log.i("YOUMEIM","发送语音结果回调，errorcode：" + errorcode + ", requestID: "+voiceMsgInfo.getRequestId()+"， extraText："+voiceMsgInfo.getExtraText()
                            + ", localPath: "+voiceMsgInfo.getLocalPath()+", audioTime: "+voiceMsgInfo.getDuration()+", sendTime: "+voiceMsgInfo.getSendTime()
                            +", isForbidRoom: "+voiceMsgInfo.getIsForbidRoom()+", reasonType: "+voiceMsgInfo.getReasonType()+", endTime: "+voiceMsgInfo.getEndTime());
                }
            } );
      
      // 如果不想发送语音，调取消本次语音，调取消后收不到消息的发送回调
      YIMClient.getInstance().cancleAudioMessage();
  ```
  
- **接口与参数：**
	- `发送语音消息接口`：
	  ` public MessageSendStatus startRecordAudioMessage(String recvId, int chatType, String extraText, boolean recognizeText, boolean IsOpenOnlyRecognizeText)` 
	  	  
	- `结束录音接口`：
	  `public void stopAndSendAudioMessage(YIMEventCallback.AudioMsgEventCallback callback)`
	  
	- `取消录音接口`：
	  `public int cancleAudioMessage()`
	  
	- `strRecvID`：接收者ID（⽤户ID或者频道ID）
	- `chatType`：消息类型，表示私聊还是频道消息
	- `extraText`：发送语音消息附加信息，主要用于调用方特别需求
	- `recognizeText`：是否带文字识别
	- `IsOpenOnlyRecognizeText`：是否开启仅识别语音文本，不发送语音消息

- **回调接口与参数：**
   调用`stopAndSendAudioMessage`接口后，会收到发送语音的回调，如果录音成功会收到开始发送语音的回调`onStartSendAudioMessage`，无论录音是否成功都会收到发送语音消息结果的回调`onSendAudioMessageStatus`，`onStartSendAudioMessage`在接收时间上比`onSendAudioMessageStatus`快，常用于上屏显示。
   
``` java
	public interface AudioMsgEventCallback {
   /**
    * 录音结束，开始发送录音的通知，这个时候已经可以拿到语音文件进行播放
    *
    * @param requestID 请求ID（与startRecordAudioMessage输出参数requestID一致）
    * @param errorcode 错误码
    * @param strText  语音转文字识别的文本内容，如果带语音转文字参数为false，该字段为空字符串
    * @param strAudioPath  录音生成的wav文件的本地完整路径
    * @param audioTime 录音时长(单位秒) 
    */
    public void onStartSendAudioMessage(long requestID, int errorcode, String strText, String strAudioPath, int audioTime);
    
    /**
    * 自己的语音消息发送成功或者失败的通知
    * 
    * @param errorcode 错误码
    * @param voiceMsgInfo ：语音信息    
    */
    public void onSendAudioMessageStatus(int errorcode, SendVoiceMsgInfo voiceMsgInfo);
	}    
```  
    
- **备注：**
  此套接口是异步操作，发送语音消息成功的标识是收到`onStartSendAudioMessage`，或者`onSendAudioMessageStatus`回调的错误码是Success；调用`stopAndSendAudioMessage`同步返回值是Success才能收到回调；语音消息发送成功，接收方会收到`onRecvMessage`回调，能从该回调中下载语音文件。

### 8. 接收消息
>- 通过`onRecvMessage`接口被动接收消息，需要开发者实现

- **调用示例：**

  ``` java
      public void onRecvMessage(YIMMessage yimMessage) {
        Log.i("YOUMEIM", "接收到消息 ");

        if (null == yimMessage)
            return;
        int msgType = yimMessage.getMessageType();
        if (YIMConstInfo.MessageBodyType.TXT == msgType)
        {
            Log.i("YOUMEIM", "接收到一条文本消息： " + ((YIMMessageBodyText)yimMessage.getMessageBody()).getMessageContent()+",消息ID："+yimMessage.getMessageID());
        }
        else if (YIMConstInfo.MessageBodyType.Voice == msgType)
        {
            YIMMessageBodyAudio audioMessage = (YIMMessageBodyAudio)yimMessage.getMessageBody();
            int audioDuration = audioMessage.getAudioTime();
            String param = audioMessage.getParam();//自定义附加参数
            String txt = audioMessage.getText();//语音转文字识别结果
            long mRecvAudioMsgId = yimMessage.getMessageID();

            Log.i("YOUMEIM", "接收到一条语音消息，AudioTime： " + audioDuration + " 录音文本信息：" + txt + " messID: " + mRecvAudioMsgId);
            
            // 下载语音文件           
            YIMClient.getInstance().downloadAudioMessage(mRecvAudioMsgId, "save Path", new YIMEventCallback.DownloadFileCallback() {
                    @Override
                    public void onDownload(int errorcode, YIMMessage message, String savePath)    {
                        Log.i("YOUMEIM","手动下载语音消息回调，err: "+errorcode+", senderID: "+message.getSenderID()+", savePath: "+savePath);

                        YIMClient.getInstance().startPlayAudio(savePath, new YIMEventCallback.ResultCallback<String>() {
                            @Override
                            public void onSuccess(String info) {
                                Log.i("YOUMEIM","播放语音成功，"+info);
                            }

                            @Override
                            public void onFailed(int errorcode, String info) {
                                Log.i("YOUMEIM","播放语音失败，err: "+ errorcode+", info: "+info);
                            }
                        });
                    }
                });       
            }
                
        ...
    }
  ```

### 9. 接收语音消息并播放
>- 通过`getMessageType()`分拣出语音消息`（MessageBodyType.Voice）`
>- 用`getMessageID()`获得消息ID
>- 调用`downloadAudioMessage`接口下载语音消息
>- 成功下载语音消息后，调用`StartPlayAudio`接口播放语音消息

- **调用示例：**

  ``` java
      // 下载语音文件
      YIMClient.getInstance().downloadAudioMessage(mRecvAudioMsgId, "save Path", new YIMEventCallback.DownloadFileCallback() {
                    @Override
                    public void onDownload(int errorcode, YIMMessage message, String savePath)    {
                        Log.i("YOUMEIM","手动下载语音消息回调，err: "+errorcode+", senderID: "+message.getSenderID()+", savePath: "+savePath);

                        YIMClient.getInstance().startPlayAudio(savePath, new YIMEventCallback.ResultCallback<String>() {
                            @Override
                            public void onSuccess(String info) {
                                Log.i("YOUMEIM","播放语音成功，"+info);
                            }

                            @Override
                            public void onFailed(int errorcode, String info) {
                                Log.i("YOUMEIM","播放语音失败，err: "+ errorcode+", info: "+info);
                            }
                        });
                    }
                });     
      
  ```
  
- **接口和参数：**
  - `下载语音消息接口`：
    `public void downloadAudioMessage(long serial, String savePath, YIMEventCallback.DownloadFileCallback callback)`
    
  - `播放语音消息接口`：
    `public void startPlayAudio(String audioPath, YIMEventCallback.ResultCallback<String> callback)`  
    
	- `serial`：消息ID
	- `savePath`：语音文件保存路径
	- `audioPath`：待播放文件路径
	- `callback`：回调

- **回调接口与参数：**
  下载语音接口是异步操作，下载语音成功的标识是`onDownload`回调的错误码为Success，调用`downloadAudioMessage`接口同步返回值是Success才能收到`onDownload`回调。成功下载语音消息后即可播放语音消息。
  
  - `下载回调接口`：
  
  ``` java
  public interface DownloadFileCallback {
     /**
      * 下载回调
      *
      * @param errorcode 错误码
      * @param message 消息
      * @param savePath  文件存放路径
      */
      public void onDownload(int errorcode, YIMMessage message , String savePath);
  }    
  ```
    
  - `errorcode`：错误码
  - `message`：消息基类
  - `savePath`：保存路径

  - `播放完成回调接口`：
  
  ``` java
  public interface ResultCallback<String>{
     
     /**
      * @param audioPath 播放的音频文件路径    
      */
      public void onSuccess(String audioPath);
  
     /**
      * @param errorcode 错误码   
      * @param audioPath 播放的音频文件路径 
      */
      public void onFailed(int errorcode, String audioPath);
  }
  ```      

### 10. 登出IM系统
注销账号时，调用登出接口`logout`登出IM系统。

- **调用示例：**

  ``` java
      YIMClient.getInstance().logout(new YIMEventCallback.OperationCallback(){

            @Override
            public void onSuccess() {
                Log.i("YOUMEIM","登出成功，");
            }

            @Override
            public void onFailed(int errorcode) {
                Log.i("YOUMEIM","登出失败，code:"+errorcode);
            }
        });
  ```
  
- **接口：**
	- `登出接口`：
	  `public void logout(final YIMEventCallback.OperationCallback callback)` 	  
## 典型场景集成方案
主要分为世界频道聊天，用户私聊，直播聊天室；集成的时都需要先初始化SDK，登录IM系统

***流程图

### 启动应用，初始化SDK
应用启动的第一时间需要调用初始化接口。

- **接口与参数：**
  - `init`：初始化接口
  - `context`：Android App上下文环境
  - `appKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://console.youme.im/)获取、查看
  - `secrectKey`：用户游戏产品的密钥，可以在[游密官网](https://console.youme.im/)获取、查看
  - `serverZone`：IM服务器区域，一般使用`0`-中国，其余设置值查看[服务器部署地区定义](IMSDKAndroid.php#服务器部署地区定义)
  
- **代码示例与详细说明：**
	- [Android示例](#初始化IM SDK)

### 登录应用时，登录IM系统

![登录界面介绍](https://www.youme.im/doc/images/im_login.png)

>- 点击`登录`按钮时，调用IM SDK登录接口。

- **接口与参数：**
	- `login`：登录接口
	- `userId`：由调⽤者分配，不可为空字符串，只可由字母或数字或下划线组成
	- `password`：⽤户密码，不可为空字符串
	- `token`：用户验证token，可选，如不使用token验证传入:""
	- `callback`：登录回调

- **代码示例与详细说明：**
	- [Android示例](#4. 登录IM系统)

### 典型场景
#### 世界频道聊天
![频道](https://www.youme.im/doc/images/im_room.png)
>- 进入应用后，调用加入频道接口，进入世界、公会、区域等需要进入的聊天频道。
>- 应用需要为各个聊天频道设置一个唯一的频道ID。
>- 成功进入频道后，发送频道消息，文本、语音消息等。

- **接口与参数**
	- `joinChatRoom`：加入频道
	- `roomId`：请求加入的频道ID
	- `callback`：加入频道回调

- **代码示例与详细说明：**
	- [Android示例](#5. 进入聊天频道)

#### 用户私聊
>- 成功登录IM系统后，可和其它登录IM的用户发私聊消息，文本、语音消息等；发文本消息接口的聊天类型参数使用 `1`-私聊类型
>- 调用流程查看[发送文本消息](#发送文本消息)，[发送语音消息](#发送语音消息)

#### 直播聊天室
>- 用户集成直播SDK后，导入IM SDK
>- 初始化IM SDK
>- 登录IM 系统
>- 进入指定的聊天频道
>- 发送频道消息（例如：弹幕式），调用流程查看[发送文本消息](#发送文本消息)

### 发送文本消息
![频道](https://www.youme.im/doc/images/im_room_send_btn.png)
>- 点击发送按钮，调用发消息接口，将输入框中的内容发送出去
>- 发送出的消息出现在聊天框右侧
>- 表情消息可以将表情信息打包成Json格式发送

- **相关接口与参数：**
	- `sendTextMessage`：发文字消息接口
	- `recvId`：接收者ID（⽤户ID或者频道ID）
	- `chatType`：聊天类型
	- `msgContent`：聊天内容
	- `attachParam`： 发送文本附加信息
	- `callback`：发送文本消息回调
	
- **代码示例与详细说明：**
	- [Android示例](#6. 发送文本消息)

- **拓展功能：**
发送消息时，可以将玩家头像、昵称、角色等级、vip等级等要素打包成json格式发送。

### 发送语音消息
![发送语音消息](https://www.youme.im/doc/images/im_send_voice_message.jpg)
>- 按住语音按钮时，调用`startRecordAudioMessage`
>- 松开按钮，调用`stopAndSendAudioMessage`接口，发送语音消息
>- 按住过程若需要取消发送，调用`cancleAudioMessage`取消发送

- **相关接口与参数：**
	- `startRecordAudioMessage`：发送语音消息接口
	- `stopAndSendAudioMessage`：结束录音接口
	- `cancleAudioMessage`：取消录音接口
	- `strRecvID`：接收者ID（⽤户ID或者频道ID）
	- `chatType`：消息类型，表示私聊还是频道消息
	- `extraText`：语音消息附带信息
	- `recognizeText`：是否带文字识别
	- `IsOpenOnlyRecognizeText`：是否开启仅识别语音文本，不发送语音消息

- **代码示例与详细说明：**
	- [Android示例](#7. 发送语音消息)

### 接收消息
![接收消息](https://www.youme.im/doc/images/im_receive_message.jpg)
>- 通过`onRecvMessage`接口被动接收消息，需要开发者实现，接收消息进行相应的展示，如果是语音消息需要下载语音，以及下载完成后的语音播放

- **相关接口与参数：**
	- `onRecvMessage`：收消息接口

- **代码示例与详细说明：**
	- [Android示例](#8. 接收消息)

### 接收语音消息并播放
![接收语音消息](https://www.youme.im/doc/images/im_receive_message-2.jpg)
>- 通过`getMessageType()`分拣出语音消息`（MessageBodyType.Voice）`
>- ⽤`getMessageID()`获得消息ID
>- 点击语音气泡，调用`downloadAudioMessage`接口下载语音文件
>- 调用方播放语音文件

- **相关接口与参数：**
  - `downloadAudioMessage`：下载语音文件接口
  - `startPlayAudio`：播放语音文件接口
   
	- `serial`：消息ID
	- `savePath`：保存路径
	- `audioPath`：音频文件路径
	- `callback`：回调
	
- **代码示例与详细说明：**
	- [Android示例](#9. 接收语音消息并播放)

### 登出IM系统
![更换账号](https://www.youme.im/doc/images/im_change_account.jpg)
>- 如下两种情况需要登出IM系统：
>- 注销账号时，调用`logout`接口登出IM系统
>- 退出应用时，调用`logout`接口登出IM系统

- **相关接口：**
	- `logout`：登出接口
	
- **代码示例与详细说明：**
	- [Android示例](#10. 登出IM系统)

