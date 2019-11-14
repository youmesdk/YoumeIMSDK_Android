# IM SDK for Android 使用指南

## IMSDK简介
### 1.功能目标

开发者在成功接入IMSDK后，无需部署任何服务器即可拥有双人及多人的即时通讯能力

### 2.SDK目录讲解

- 库目录

  `im_android\lib` 包含了必须的jar和so库，请完整导入Android工程的库目录。

### 3.关键类及描述

- YIMClient

  `IM SDK引擎服务类,单例模式。该类提供SDK操作的相关方法,例如:初始化，登录，登出，消息发送等`

- YIMMessage

  `IM消息类，包含消息体和消息基本属性：例如消息ID，消息类型，发送者ID，接收者ID等信息`

- IYIMMessageBodyBase

  `IM消息体基类`

- YIMMessageBodyAudio

  `IM语音消息类，包含语音消息除基本属性外的其它信息：例如语音时长，语音本地存放地址，语音文件大小等信息`

- YIMMessageBodyText

  `IM文本消息类，包含文本消息除基本属性外的其它信息：文本内容`
  
- YIMMessageBodyCustom

  `IM自定义消息类，包含自定义消息除基本属性外的其它信息：自定义消息内容`
  
- YIMMessageBodyFile

  `IM文件消息类，包含文件消息除基本属性外的其它信息：文件名，文件大小，文件类型，文件扩展信息等` 
  
- YIMMessageBodyGift

  `IM礼物消息类，包含礼物消息除基本属性外的其它信息：礼物ID，礼物数量，主播ID，礼物信息` 
  
- YIMConstInfo

  `IM 枚举定义`      

### 4.消息类型定义

``` java
YIMConstInfo.MessageBodyType{
    public final static int Unknow = 0;
    public final static int TXT = 1;        //文本消息
    public final static int CustomMesssage = 2;  //用户自定义消息
    public final static int Emoji = 3;
    public final static int Image = 4;
    public final static int Voice = 5;      //语音消息
    public final static int Video = 6;
    public final static int File = 7;
    public final static int Gift = 8;
}
```

### 5.聊天类型定义

``` java
YIMConstInfo.ChatType{
    public final static int Unknow = 0;
    public final static int PrivateChat = 1;
    public final static int RoomChat = 2;
}
```

## IMSDK操作指引
### 申请使用游密IM引擎SDK

- 首先请与游密商务联系，提供公司名称、游戏的名称、联系人电话、邮箱、 QQ等以申请IM引擎的使用权限。审批通过后会得到有效的AppKey和AppSecret，这些信息属于私密信息，请妥善保管

### 导入SDK
#### 1.jar包导入

- 将jar包目录`im_android\lib`下的`yim-sdk.jar`，`yim.jar`，`Msc.jar`，`alisr.jar`，`usc.jar`，`Sunflower.jar`,`android-support-v4.jar`,`gson-2.3.1.jar` 包拷贝到Android工程的`project`->`libs`目录下

#### 2.so库导入

- 将so库目录`im_android\lib\armeabi`下的`libmsc.so`，`libnlscppsdk.so`，`libuscasr.so`，`libuuid.so`，`libyim.so`库拷贝到Android工程的`project`->`libs`->`armeabi`目录下。(如上选择所需架构下的so文件)

#### 3.Android权限配置

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

### import包名

`import com.youme.imsdk.YIMClient;`

### 初始化SDK

- 功能：初始化IM SDK

- 原型：

``` java
    /**
     *
     * @param context Android App环境上下文
     * @param appKey  在 www.youme.im 注册后获得的app标识
     * @param secretKey  在 www.youme.im 注册后获得的app安全密钥
     * @param serverZone 设置IM服务器区域，参考YIMService.ServerZone定义
     * @return
     */
     public int init(Context context, String appKey, String secretKey,int serverZone)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义) 

- 备注：该方法必须在调用其它SDK API之前调用

### 设置监听
IM引擎底层对于耗时操作都采用异步回调的方式，函数调用会立即返回，操作结果java层会同步回调。因此，用户必须实现相关接口并在初始化完成以后注册到YIMClient

#### 重连回调 

- 设置重连回调方法：

```java
     /**
     * 设置重连回调
     * @param callback 重连回调接口
     */
    public void registerReconnectCallback(YIMEventCallback.ReconnectCallback callback);
```

- 重连回调接口定义

```java
   public interface ReconnectCallback{
      /**
       * 开始重连通知    
       */
      public void onStartReconnect();

      /**
       * 重连结果通知
       * @param result 0-重连成功，1-重连失败，再次重连，2-重连失败
       */
      public void onRecvReconnectResult(int result);
}
```

#### 用户进出频道通知 

- 设置用户进出频道通知方法：YIMClient.registerUserJoinLeaveCallback

``` java
    /**
     * 设置用户进出房间通知监听
     * @param callback 登录登出回调接口
     */
    public void registerUserJoinLeaveCallback(YIMEventCallback.UserJoinLeaveChannelCallback callback) {
        mLoginListener = l;
    }
```

- 用户进出频道接口定义：

``` java
    public interface UserJoinLeaveChannelCallback {
        /**
         * 功能：其他用户进出频道通知
         *
         * @param eventType：频道事件类型，UserJoinRoom=0 UserLeaveRoom=1
         * @param info：用户和频道信息
         */
        public void joinLeaveNotify(YIMClient.ChannelEventType eventType, UserChatRoom info);       
    }
```

#### 用户被踢通知 

- 设置用户被踢通知方法：YIMClient.registerKickOffCallback

``` java
    /**
     * 设置用户被踢通知监听
     * @param callback 用户被踢通知接口
     */
    public void registerKickOffCallback(YIMEventCallback.KickOffCallback callback)
```

- 用户被踢通知接口定义

``` java
    public interface KickOffCallback {
        /**
         * 被踢下线通知
         *         
         */
        public void onKickOff();       
    }
```

#### 消息回调 

- 设置消息处理回调方法：

``` java
    /**
     * 设置消息相关的回调
     * @param callback 消息相关回调接口
     */
    public void registerMsgEventCallback(YIMEventCallback.MessageEventCallback callback)
```

- 消息处理回调接口定义：

``` java
    public interface MessageEventCallback {
        
        /**
         * 功能：接收到用户发来的消息
         *
         * @param message 消息内容结构体
         */
        public void onRecvMessage(YIMMessage message);

        
        /**
         * 功能：仅语音识别回调（调用startRecordAudioMessage时开启仅需要语音识别结果才会有该回调，不会发送语音消息，停止语音后产生该回调）
         *
         * @param errorcode 错误码
         * @param requestID：请求ID（与startRecordAudioMessage输出参数requestID一致）
         * @param text：语音识别结果
         */
        public void onGetRecognizeSpeechText(int errorcode, long requestID, String text);


        /**
         * 功能：新消息通知（默认自动接收消息，只有调用setReceiveMessageSwitch设置为不自动接收消息，才会收到该回调），有新消息的时候会通知该回调，频道消息会通知消息来自哪个频道ID
         * @param chatType：聊天类型
	       * @param targetID：频道ID
         */
        public void OnRecvNewMessage( int chatType, String targetID );
        
         /*
         * 功能：录音音量变化回调, 频率:1s 约8次
         * @param volume：音量值(0到1)
         */
        public void OnRecordVolume(float volume);
    }
```

#### 自动下载语音消息回调 

- 设置自动下载语音消息回调方法：

```java
     /**
     * 设置自动下载回调
     * @param callback 自动下载回调接口
     */
   public void registerAutoDownloadCallback(YIMEventCallback.AutoDownloadVoiceCallback callback);
```

- 自动下载语音消息回调接口定义

```java
   public interface AutoDownloadVoiceCallback{
   /**
    * 功能：自动下载回调
	  * @param errorcode：错误码
	  * @param message 消息
	  * @param savePath 保存路径
    */
   public void onAutoDownload(int errorcode, YIMMessage message , String savePath);   

}
```

#### 地理位置信息回调 

- 设置地理位置回调方法：

```java
     /**
     * 设置lbs信息回调
     * @param callback lbs回调接口
     */
   public void registerGetLocationCallback(YIMEventCallback.GetLocationCallback callback);
```

- lbs回调接口定义

```java
   public interface GetLocationCallback{
   /**
    * 功能：地理位置信息回调
	  * @param errorcode：错误码  
	  * @param location：地理位置信息
    */
   public void onUpdateLocation(int errorcode, GeographyLocation location);   

}
```

#### 举报回调 

- 设置举报回调方法：

```java
     /**
     * 设置举报结果回调
     * @param callback 举报回调接口
     */
   public void registerAccusationCallback(YIMEventCallback.AccusationResultCallback callback);
```

- 举报回调接口定义

```java
   public interface AccusationResultCallback{
   /**
    * 功能：举报结果通知
	  * @param result：举报处理结果  
	  * @param userID：用户ID
	  * @param accusationTime：举报时间戳
    */
   public void onAccusationResultNotify(int result, String userID, int accusationTime);   

}
```

#### 公告回调 

- 设置公告回调方法：

```java
     /**
     * 设置公告回调
     * @param callback 公告回调接口
     */
   public void registerNoticeCallback(YIMEventCallback.NoticeCallback callback);
```

- 公告回调接口定义

```java
   public interface NoticeCallback{
   /**
    * 功能：接收公告通知
	  * @param notice：公告信息	 
    */
   public void onRecvNotice(NoticeInfo notice);  
   
   /*
	 * 功能：撤销公告通知
	 * @param noticeID：公告ID
	 * @param channelID：频道ID
	 */
    public void onCancelNotice(long noticeID, String channelID); 

}
```

#### 用户信息变更通知 

- 设置用户信息变更监听方法：

```java
     /**
     * 设置用户信息变更监听
     * @param callback 用户信息变更接口
     */
   public void registerUserProfileChangeCallback(YIMEventCallback.UserProfileChangeCallback callback);
```

- 用户信息变更接口定义

```java
   public interface UserProfileChangeCallback{
   /**
    * 功能：用户信息变更通知
	  * @param userID：用户ID	 
    */
   public void onUserInfoChangeNotify(String userID);      
}
```

#### 好友相关通知

- 设置好友相关通知方法：

```java
     /**
     * 设置好友相关通知监听
     * @param callback 好友相关通知接口
     */
   public void registerFriendNotifyCallback(YIMEventCallback.FriendNotifyCallback callback);
```

- 好友相关通知接口定义

```java
   public interface FriendNotifyCallback{
   /**
    * 功能：被邀请添加好友通知（需要验证）
	  * @param userID：用户ID
    * @param comments：备注或验证信息    
    */
   public void onBeRequestAddFriendNotify(String userID, String comments);  
   
   /*
    * 功能：被添加为好友通知（不需要验证）
    * @param userID：用户ID
    * @param comments：备注或验证信息
    */
   public void onBeAddFriendNotify(String userID, String comments); 
   
   /*
    * 功能：请求添加好友结果通知(需要好友验证，待被请求方处理后回调)
    * @param userID：用户ID
    * @param comments：备注或验证信息
    * @param dealResult：处理结果	0：同意	1：拒绝
    */
   public void onRequestAddFriendResultNotify(String userID, String comments, int dealResult);

   /*
    * 功能：被好友删除通知
    * @param userID：用户ID
    */
   public void onBeDeleteFriendNotify(String userID);

}
```

## 用户管理
### 用户登录 

- 功能：指定用户ID登录IM系统,登录为异步过程，通过回调参数返回是否成功，成功后方能进行后续操作。

- 原型：

``` java    
    /**
     *
     * @param userId 用户ID，由调用者分配，不可为空字符串，只可由字母或数字或下划线组成，长度限制为255字节
     * @param password 登录密码，不可为空字符串，如无特殊要求可以设置为固定字符串
     * @param token 用户验证token，使用服务器token验证模式时使用，如不使用token验证传入:""，由restAPI获取token值
     * @param callback 登录回调
     */
    public void login(String userId, String password, String token, final YIMEventCallback.ResultCallback<String> callback)
```

- 回调参数：

  `YIMEventCallback.ResultCallback<String>`: 
  
  ``` java
  public interface ResultCallback<String> {
     /**
      * @param info 用户ID    
      */
      public void onSuccess(String info);
  
     /**
      * @param errorcode 错误码   
      * @param info 用户ID 
      */
      public void onFailed(int errorcode, String info);
  }
  ```

### 用户登出 

- 功能：登出游密IM云服务器，这是一个异步操作，操作结果会通过回调参数返回
- 原型：

``` java
    /**
     * @param callback 登出回调    
     */
    public void logout(final YIMEventCallback.OperationCallback callback)
```
  
- 回调参数：

  `YIMEventCallback.OperationCallback`: 
  
  ``` java   
  public interface OperationCallback {
      public void onSuccess();
  
     /**
      * @param errorcode 错误码   
      */
      public void onFailed(int errorcode);
  }    
  ```  
  
### 被用户踢出通知
同一个用户ID在多台设备上登录时，后登录的会把先登录的踢下线，收到onKickOff()通知。

- 原型

``` java
    public interface KickOffCallback {
        /**
         * 被踢下线通知
         *         
         */
        public void onKickOff();       
    }
```  

### 用户信息
#### 设置用户的详细信息 

- 功能：设置用户信息后，在游密消息管理后台才能看到消息发送者的详细信息。
- 原型：

``` java
/**
     *
     * @param userInfo 用户详细信息，请设置YIMExtraUserInfo的以下属性：
     *   `nickName`：用户昵称
     *   `serverAreaID`：游戏服ID
     *   `serverArea`：游戏服名称
     *   `locationID`：游戏大区ID
     *   `location`：游戏大区名
     *   `Platform`：平台名称，比如：应用宝
     *   `PlatformID`：平台ID
     *   `level`：角色等级
     *   `vipLevel`：玩家VIP等级
     *   `extra`：其它自定义扩展信息，建义用json字符串
     *   `userID`：用户ID
     * @return
     */
    public int setUserInfo(YIMExtraUserInfo userInfo)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

#### 获取指定用户的详细信息 
这是一个异步操作，操作结果会通过回调参数返回。

- 原型：

``` java
    /**
     *
     * @param userID 用户ID
     * @param callback 查询用户信息回调
     * 
     */
    public void getUserInfo(String userID, YIMEventCallback.ResultCallback<YIMExtraUserInfo> callback)
```

- 回调参数：

`YIMEventCallback.ResultCallback<YIMExtraUserInfo>`: 

```java
public interface ResultCallback<YIMExtraUserInfo> {
  /**
   * @param info 用户详细信息    
   */
  public void onSuccess(YIMExtraUserInfo info);
  
  /**
   * @param errorcode 错误码   
   * @param info 用户详细信息 
   */
  public void onFailed(int errorcode, YIMExtraUserInfo info);
} 

```

- 备注：
  `YIMExtraUserInfo`类定义参考[设置用户的详细信息](#设置用户的详细信息)接口参数。

#### 查询用户在线状态 
这是一个异步操作，操作结果会通过回调参数返回。

- 原型：

``` java
    /**
     *
     * @param userID 用户ID
     * @param callback 查询结果回调
     * 
     */
    public void queryUserStatus(String userID, YIMEventCallback.ResultCallback<UserStatusInfo> callback)
```

- 回调参数：

`YIMEventCallback.ResultCallback<UserStatusInfo>`:

``` java
public interface ResultCallback<UserStatusInfo> {
   /**
    * @param statusInfo 用户状态信息    
    */
    public void onSuccess(UserStatusInfo statusInfo);
  
   /**
    * @param errorcode 错误码   
    * @param statusInfo 用户状态信息 
    */
    public void onFailed(int errorcode, UserStatusInfo statusInfo);    
}    
```

- 备注：
  `UserStatusInfo`类成员变量如下：
  `userID`: 用户ID，String类型
  `status`: 用户状态，Integer类型,0-在线，1-离线，2-未知
  `isOnline()`: 是否在线，返回boolean类型

## 频道管理
### 加入频道 

- 功能：加入聊天频道进行群组聊天，这是一个异步操作，操作结果会通过回调参数返回。
- 原型：

``` java
    /**
     * 加入聊天频道
     * @param roomId 频道ID，由调用者定义，如果频道不存在则后台自动创建，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节
     * @param callback 加入频道回调
     */
    public void joinChatRoom(String roomId, final YIMEventCallback.ResultCallback<ChatRoom> callback)
```
  
- 回调参数：

  `YIMEventCallback.ResultCallback<ChatRoom>`:

  ``` java
  public interface ResultCallback<ChatRoom> {
      /**
       * @param roomInfo 频道信息    
       */
       public void onSuccess(ChatRoom roomInfo);
  
      /**
       * @param errorcode 错误码   
       * @param roomInfo 频道信息 
       */
       public void onFailed(int errorcode, ChatRoom roomInfo);
  }         
  ```  
  
- 备注：
  `ChatRoom`类成员变量如下：
  `groupId`: 频道ID，String类型

### 离开频道

- 功能：退出聊天频道，这是一个异步操作，操作结果会通过回调参数返回
- 原型：

``` java
    /**
     * 退出聊天频道
     * @param roomId 频道ID
     * @param callback 退出频道回调
     */
    public void leaveChatRoom(String roomId,YIMEventCallback.ResultCallback<ChatRoom> callback)
```

- 回调参数：

  `YIMEventCallback.ResultCallback<ChatRoom>`:
  
  ``` java
  public interface ResultCallback<ChatRoom> {
      /**
       * @param roomInfo 频道信息    
       */
       public void onSuccess(ChatRoom roomInfo);
  
      /**
       * @param errorcode 错误码   
       * @param roomInfo 频道信息 
       */
       public void onFailed(int errorcode, ChatRoom roomInfo);   
  }     
  ```  
  
- 备注：
  `ChatRoom`类成员变量如下：
  `groupId`: 频道ID，String类型    

### 离开所有频道 

- 功能：离开所有频道，这是一个异步操作，操作结果会通过回调参数返回
- 原型：

``` java
    /**
     * 离开所有频道    
     * @param callback 离开所有频道回调
     */
    public void leaveAllChatRooms(YIMEventCallback.OperationCallback callback)
```
 
- 回调参数：

  `YIMEventCallback.OperationCallback`:

  ``` java
  public interface OperationCallback {
      
       public void onSuccess();
  
      /**
       * @param errorcode 错误码      
       */
       public void onFailed(int errorcode);
  }     
  ```    

### 用户进出频道通知 
此功能默认不开启，需要的请联系我们开启此服务。联系我们，可以通过专属游密支持群或者技术支持的大群。

``` java
public interface UserJoinLeaveChannelCallback {
        /**
         * 功能：其他用户进出频道通知
         *
         * @param eventType：频道事件类型，UserJoinRoom=0 UserLeaveRoom=1
         * @param info：用户和频道信息
         */
        public void joinLeaveNotify(YIMClient.ChannelEventType eventType, UserChatRoom info);       
}
```

### 获取房间成员数量
这是一个异步操作，操作结果会通过回调参数返回。

- 原型：

``` java
  /* 
	* 功能：获取房间成员数量
	* @param roomId：频道ID(已成功加入此频道才能获取该频道的人数)
	* @param callback 获取频道成员数量回调
	*/
    public void getRoomMemberCount(String roomId, YIMEventCallback.ResultCallback<RoomInfo> callback)
```       

- 回调参数：

  `YIMEventCallback.ResultCallback<RoomInfo>`:

  ``` java
  public interface ResultCallback<RoomInfo> {
      /**
       * @param roomInfo 频道及成员信息    
       */
       public void onSuccess(RoomInfo roomInfo);
  
      /**
       * @param errorcode 错误码   
       * @param roomInfo 频道及成员信息 
       */
       public void onFailed(int errorcode, RoomInfo roomInfo);
  }     
  ```
  
- 备注：
  `RoomInfo`类成员变量如下：
  `roomID`: 频道ID，String类型  
  `memberCount`: 频道内成员数量，int类型

## 消息管理  
游密IM可以进行多种消息类型的交互，比如文本，自定义，表情，图片，语音，文件，礼物等。

### 接收消息 

- 功能：消息接收采用异步回调的方式，在MessageEventCallback接口中的onRecvMessage(YIMMessage message)中处理。

``` java
public interface MessageEventCallback {
    /**
    * 接收到用户发来的消息
    *
    * @param message 消息内容结构体
    */
    public void onRecvMessage(YIMMessage message);
}    
```

- 备注：
  `YIMMessage` 消息基类成员方法如下：
  `getSenderID()`： 返回消息发送者ID，String型
  `getReceiveID()`： 返回消息接收者ID，String型
  `getChatType()`：返回聊天类型，私聊/频道聊天，int型
  `getMessageID()`： 返回消息ID，long型
  `getMessageType()`： 返回消息类型，0-未知类型，1-文本消息，2-自定义消息，3-表情，4-图片，5-语音，6-视频，7-文件，8-礼物  `getCreateTime()`： 消息收发时间  
  `getIsRead()`：返回消息是否已读，boolean型
  `getDistance()`：若是附近频道的消息，返回该消息用户与自己的地理位置距离 
  `getMessageBody()`：返回消息体，IYIMMessageBodyBase型
  
  `YIMMessageBodyText`实现`IYIMMessageBodyBase`接口，类成员方法如下：
  `getMessageContent()`： 返回文本消息内容，String型
  `getAttachParam()`： 返回发送文本附加信息内容，String型
  
  `YIMMessageBodyCustom`实现`IYIMMessageBodyBase`接口，类成员方法如下：
  `getMessageContent()`： 返回自定义消息内容，byte[]型
  
  `YIMMessageBodyFile`实现`IYIMMessageBodyBase`接口，类成员方法如下：
  `getFileName()`： 返回文件名，String型
  `getFileType()`： 返回文件类型，int类型，0-其它，1-音频（语音），2-图片，3-视频
  `getFileSize()`： 返回文件大小，int型
  `getFileExtension()`： 返回文件扩展信息，String型
  `getExtParam()`： 返回附加信息，String型
  
  `YIMMessageBodyGift`实现`IYIMMessageBodyBase`接口，类成员方法如下：
  `getAnchor()`： 返回主播ID，String型
  `getGiftID()`： 返回礼物ID，Integer型
  `getGiftCount()`： 返回礼物数量，Integer型
  `getExtParam()`： 返回附加参数，YIMExtraGifParam型
  
  `YIMMessageBodyAudio`实现`IYIMMessageBodyBase`接口，类成员方法如下：
  `getText()`： 若使用的是语音转文字录音，返回值为语音识别的文本内容，否则是空，String型
  `getAudioTime()`： 返回语音时长（单位：秒），int型
  `getParam()`： 返回发送语音时的附加参数，String型

- 示例：

``` java
    public void onRecvMessage(YIMMessage message){
    if (null == message)
        return;
    int msgType = message.getMessageType();
    if (YIMConstInfo.MessageBodyType.TXT == msgType){
        showRecv("接收到一条文本消息： " + ((YIMMessageBodyText)message.getMessageBody()).getMessageContent());
    }else if (YIMConstInfo.MessageBodyType.Voice == msgType){
        YIMMessageBodyAudio audioMessage = (YIMMessageBodyAudio)message.getMessageBody();
        int audioDuration = audioMessage.getAudioTime();
        string param = audioMessage.getParam();//自定义附加参数
        string txt = audioMessage.getText();//语音转文字识别结果
        mRecvAudioMsgId = message.getMessageID();
        //下载语音文件
        YIMClient.getInstance().downloadAudioMessage(mRecvAudioMsgId, "/sdcard/xxx/xxx.wav", new YIMEventCallback.DownloadFileCallback() {
             @Override
             public void onDownload(int errorcode, YIMMessage message, String savePath) {
              showRecv("download callback，err: "+errorcode+", senderID: "+message.getSenderID()+", savePath: "+savePath);
          });
    }
}
```

### 发送文本消息 

- 功能：发送文本消息到指定接收者，这是一个异步操作，操作结果会通过回调参数返回

- 原型：

``` java
    /**
     * 发送文本消息
     * @param recvId 消息接收者ID
     * @param chatType 聊天类型，详见ChatType
     * @param msgContent 消息内容
     * @param attachParam 附加信息
     * @param callback 发送文本消息回调
     * 
     */
    public void sendTextMessage(String recvId, int chatType, String msgContent, String attachParam, YIMEventCallback.ResultCallback<SendMessage> callback)
```

- 回调参数：

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

### 群发文本消息 

- 功能：用于群发文本消息的接口，**每次不要超过200个用户**。
- 原型：

``` java
    /**
     *
     * @param recvLists 接受消息的用户ID列表
     * @param strText 文本消息内容
     * @return
     */
    public int multiSendTextMessage(List<String> recvLists,String strText)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

### 发送礼物消息 

- 功能：给主播发送礼物消息的接口，支持在游密主播后台查看礼物消息信息和统计信息。客户端还是通过`OnRecvMessage`接收消息。`giftID`为`0`可以表示普通的主播留言。
- 原型：

``` java
    /**
     *
     * @param anchorId 游密后台设置的对应的主播游戏id
     * @param channel 主播频道ID，通过`JoinChatRoom`进入频道的用户可以接收到消息。
     * @param giftID 礼物物品ID，特别的是`0`表示只是留言
     * @param giftCount 礼物物品数量
     * @param extParam 扩展内容，参考YIMExtraGifParam类熟悉说明
     * @param callback 发送礼物消息回调
     */
    public void sendGift(String anchorId,String channel,int giftID,int giftCount,YIMExtraGifParam extParam, YIMEventCallback.ResultCallback<SendMessage> callback)
```
  
- 回调参数：

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
  `SendMessage`类成员变量如下：
  `getRequestId()`: 消息序列号，用于校验一条消息发送成功与否的标识，long类型  
  `getSendTime()`: 消息发送时间，int类型
  `getIsForbidRoom()`: 若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，boolean类型
  `getReasonType()`: 若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它，int类型
  `getEndTime()`: 若在频道被禁言，禁言结束时间，long类型

### 发送自定义消息 

- 功能：发送用户自定义消息，自定义数据是二进制数据。异步返回结果通过回调参数返回。
- 原型：

```java
    /**
     *
     * @param recvId 接收者ID（用户ID或者频道ID）
     * @param chatType 聊天类型
     * @param customMessage 聊天内容
     * @param bufferLen  二进制数据数组长度
     * @param callback 发送二进制消息回调
     */
    public void sendCustomMessage(String recvId, int chatType, byte[] customMessage, int bufferLen, YIMEventCallback.ResultCallback<SendMessage> callback)
```

- 回调参数：

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
  `SendMessage`类成员变量如下：
  `getRequestId()`: 消息序列号，用于校验一条消息发送成功与否的标识，long类型  
  `getSendTime()`: 消息发送时间，int类型
  `getIsForbidRoom()`: 若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，boolean类型
  `getReasonType()`: 若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它，int类型
  `getEndTime()`: 若在频道被禁言，禁言结束时间，long类型

### 发送文件消息
#### 发送文件 

- 功能：发送文件类型消息
- 原型：

``` java
    /**
     *
     * @param recvId 接收者（用户ID或者频道ID）
     * @param chatType 聊天类型
     * @param filePath 要发送的文件绝对路径，包括文件名
     * @param extParam 扩展内容，格式自己定义，自己解析
     * @param fileType 文件类型，0-其它，1-音频，2-图片，3-视频
     * @param callback 发送文件回调
     */
    public void sendFile(String recvId, int chatType,String filePath,String extParam, int fileType, YIMEventCallback.ResultCallback<SendMessage> callback)
```
  
- 回调参数：

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
  `SendMessage`类成员变量如下：
  `getRequestId()`: 消息序列号，用于校验一条消息发送成功与否的标识，long类型  
  `getSendTime()`: 消息发送时间，int类型
  `getIsForbidRoom()`: 若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，boolean类型
  `getReasonType()`: 若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它，int类型
  `getEndTime()`: 若在频道被禁言，禁言结束时间，long类型
 
#### 下载文件 

- 功能：下载文件消息的文件内容,这是一个异步操作，操作结果会通过回调接口返回
- 原型：

  ``` java
     /**
      * 下载文件
      * @param serial 消息ID
      * @param savePath 文件保存路径（带文件名的全路径），如果目录不存在，SDK会自动创建，必须保证该路径有可写权限
      * @param callback 下载文件回调
      */
      public void downloadFile(long serial, String savePath, YIMEventCallback.DownloadFileCallback callback)
  ```

- 回调参数：

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

- 备注：
  `YIMMessage`类详细定义查看[接收消息](#接收消息)备注

### 语音消息管理
#### 设置自动下载语音消息 
- 功能：设置是否自动下载语音消息。
setDownloadAudioMessageSwitch()在初始化之后，启动语音之前调用;若设置了自动下载语音消息，不需再调用downloadAudioMessage()接口，收到语音消息时会自动下载，自动下载完成也会收到AutoDownloadVoiceCallback接口对应的onAutoDownload()回调。

- 原型：

``` java
    /**
     *
     * @param download `true`：自动下载语音消息，`false`：不自动下载语音消息(默认)。
     * @return 错误码，详细描述见错误码定义。
     */
    public int setDownloadAudioMessageSwitch(boolean download)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

#### 设置语音识别的语言 
`startRecordAudioMessage`提供语音识别功能，将语音识别为文字，默认输入语音为普通话，识别文字为简体中文，可通过`setSpeechRecognizeLanguage`函数设置语言；若需要识别为繁体中文，需要联系我们配置此服务。

- 原型：

``` java
/** * 设置语音识别语言 * @param language：语言 */public int setSpeechRecognizeLanguage(int language)
```

- 参数：
  `language`：语言（0-普通话 1-粤语 2-四川话 3-河南话 4-英语 5-繁体中文）

#### 发送语音消息
##### 开始录音 

- 功能：开始录音
- 原型：

``` java
    /**
     * 开始录音,带语音转文字识别
     * @param recvId 消息接收者ID，私聊传入userid，频道聊天传入roomid
     * @param chatType 聊天类型，详见ChatType，1是私聊，2是频道聊天
     * @param extraText 语音消息附带信息
     * @param recognizeText 是否带文字识别
     * @param IsOpenOnlyRecognizeText 是否开启仅识别语音文本，不发送语音消息
     * @return YIMClient.MessageSendStatus
     */
    public MessageSendStatus startRecordAudioMessage(String recvId, int chatType, String extraText, boolean recognizeText, boolean IsOpenOnlyRecognizeText);
    
```

- 返回：
  启动录音是否成功，MessageSendStatus包含错误码和消息ID

##### 取消录音 

- 功能：取消录音
- 原型：

``` java
    /**
     * 取消录音
     * @return
     */
    public int cancleAudioMessage()
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

##### 停止录音并发送 

- 功能：停止录音并发送出去,这是一个异步操作，操作结果会通过回调参数返回
- 原型：

``` java
    /**
     * 停止录音并发送
     * @param callback 发送语音消息回调
     * 
     */
    public void stopAndSendAudioMessage(YIMEventCallback.AudioMsgEventCallback callback)
```
   
- 回调参数：

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

- 备注：
  `SendVoiceMsgInfo`类成员方法如下：
  `getRequestId()`: 消息序列号，用于校验一条消息发送成功与否的标识，long类型  
  `getText()`: 语音转文字识别的文本内容，如果没有用带语音转文字的接口，该字段为空字符串，String类型
  `getLocalPath()`: 录音生成的wav文件的本地完整路径，String类型
  `getDuration()`: 录音时长(单位秒) ，int类型
  `getSendTime()`: 语音发送时间，int类型
  `getIsForbidRoom()`: 若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言，boolean类型
  `getReasonType()`: 禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它，int类型
  `getEndTime()`: 若在频道被禁言，禁言结束时间，long类型
  
  ***如果只识别语音文本，不发送语音消息，收到以下回调：***

``` java
public interface MessageEventCallback {
   /**
    * 语音的识别文本回调
    *
    * @param errorcode 错误码
    * @param requestID 消息ID
    * @param text 返回的语音识别文本    
    */
    public void onGetRecognizeSpeechText(int errorcode, long requestID, String text);
}    
```
  
#### 接收语音消息 
接收消息接口参考收消息通过getMessageType()分拣出语音消息类型：MessageBodyType.Voice，用getMessageID()获得消息ID。然后调用函数downloadAudioMessage下载语音消息，下载完成后调用方播放。

- 原型：

``` java
    /**
     * 下载语音文件
     * @param serial 语音消息ID
     * @param savePath 文件保存路径（带文件名的全路径），如果目录不存在，SDK会自动创建，必须保证该路径有可写权限
     * @param callback 下载语音文件回调, 若设置了自动下载回调监听，自动下载回调接口也会收到回调信息
     */
    public void downloadAudioMessage(long serial, String savePath, YIMEventCallback.DownloadFileCallback callback)
```

- 回调参数：

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

- 备注：
  `YIMMessage`类详细定义查看[接收消息](#接收消息)备注

#### 语音播放
##### 播放语音  
- 功能： 可以使用SDK内置的播放接口进行播放，目前只支持WAV格式。
- 原型：

  ``` java
     /**
      *
      * @param audioPath 语音文件的完整路径。
      * @param callback 播放语音回调
      */
      public void startPlayAudio(String audioPath, YIMEventCallback.ResultCallback<String> callback)
  ```

- 回调参数：

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
 
##### 停止语音播放 

- 功能：停止播放当前语音。
- 原型：

``` java
    /**
     * @return 错误码，详细描述见错误码定义
     */
     public int stopPlayAudio()
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

##### 设置语音播放音量 

- 功能：设置语音音量。
- 原型：

``` java
    /**
     *
     * @param volume 音量值，取值范围`0f` - `1.0f`，默认值为 `1.0f`。
     */
    public void setVolume(float volume)
```

#### 录音缓存 
##### 设置录音缓存目录  
- 功能：设置录音时用于保存录音文件的缓存目录，如果没有设置，SDK会在APP默认缓存路径下创建一个文件夹用于保存音频文件。
- 原型：

``` java
    /**
     * 下载语音文件
     * @param path 缓存目录绝对路径，如果目录不存在，SDK会自动创建。
     * 
     */
    public void setAudioCachePath(String path);
```

##### 获取当前设置的录音缓存目录 

- 功能：获取当前设置的录音缓存目录。
- 原型：

``` java
    /**
     *
     * @return 返回当前设置的录音缓存目录的完整路径。
     */
    public String getAudioCachePath()
```

- 返回：
  返回当前设置的录音缓存目录的完整路径。

##### 清理录音缓存目录 

- 功能：清空当前设置的录音缓存目录。
- 原型：

``` java
    /**
     *
     * @return `true`：表示清理成功，`false`：表示清理操作失败。
     */
    public boolean clearAudioCachePath()
```

- 返回：
  `true`：表示清理成功，`false`：表示清理操作失败。

#### 设置下载保存目录 
- 功能：设置语音消息或者文件的保存目录
下载保存目录是downloadAudioMessage/downloadFile的默认下载目录，对下载语音消息和文件均适用；设置下载保存目录在初始化之后，启动语音之前调用，若设置了下载保存目录，调用downloadAudioMessage()/downloadFile()时其中的savePath参数可以为空字符串。
- 原型：

``` java
    /**
     * 设置语音消息或者文件的保存目录
     * @param path 下载目录路径，必须保证该路径可写
     * @return 错误码，详细描述见错误码定义
     */
    public int setDownloadDir(String path);
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

#### 将接收的语音消息标记为已播放

- 功能：播放语音消息后，标记该语音消息为已播放。

- 原型：

  ``` java
     /**
      *
      * @param messageID 语音消息ID
      * @param played 是否已播放，`true`为已播放，`false`为未播放。
      * @return
      */
      public int setVoiceMsgPlayed(long messageID, boolean played)
  ```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)  
 
#### 获取麦克风状态 
这是一个异步操作，操作结果会通过回调参数返回。（该接口会临时占用麦克风）


- 原型：

  ``` java
     /**
      *
      * 获取麦克风状态
      * @param callback 获取麦克风状态回调      
      */
      public void getMicrophoneStatus(YIMEventCallback.GetMicStatusCallback callback)
  ```

- 回调参数：

  ``` java
  public interface GetMicStatusCallback{
      /**
       * 获取麦克风状态回调
       * @param status  麦克风的状态，0-可用，1-无权限，2-静音，3-不可用
       */
      public void onGetMicrophoneStatus(int status);
  }
  ```
    
### 录音上传
#### 启动录音 

- 原型：

  ``` java
     /**
      *
      * @param translateToText 是否进行语音转文字识别
      * @return YIMClient.MessageSendStatus
      */
      public YIMClient.MessageSendStatus startAudioSpeech(boolean translateToText)
  ```

- 返回：
  消息发送结果，MessageSendStatus包含错误码和消息ID

#### 停止录音并发送 

- 功能：该接口只返回音频文件的下载链接，不会自动发送,这是一个异步操作，操作结果会通过回调参数返回。
- 原型：

  ``` java
     /**
      * 停止录音
      * @param callback 上传录音回调      
      */
      public void stopAudioSpeech(YIMEventCallback.SpeechEventCallback callback)
  ```

- 相关接口：

  ``` java
      /**
       * 取消录音
       * @return 错误码
       */
    public int cancleAudioMessage()
  ```

- 回调参数：

``` java
public interface SpeechEventCallback {
   /**
    * StartAudioSpeech 接口对应的结果回调通知
    * @param errorcode 错误码，0 表示正常
    * @param iRequestID 消息ID
    * @param strDownloadURL 语音文件下载地址（wav格式）
    * @param iDuration 录音时长，单位秒
    * @param iFileSize 文件大小，字节
    * @param strLocalPath 本地语音文件路径
    * @param strText 语音识别文本，可能为空null or ""
    */
    public void onStopAudioSpeechStatus(int errorcode, SpeechMessageInfo speechMsgInfo);
}    
```

- 备注：
  `SpeechMessageInfo`类成员方法如下：
  `getDownloadURL()`: 语音文件下载地址（wav格式），String类型  
  `getDuration()`: 录音时长，单位秒，int类型
  `getFileSize()`: 文件大小，字节，int类型
  `getLocalPath()`: 本地语音文件路径 ，String类型
  `getRequestID()`: 消息ID，long类型
  `getText()`: 语音识别文本，可能为空null or ""，String类型
  
#### 根据url下载语音 
- 功能：通过onStopAudioSpeechStatus收到url地址后，通过这个接口下载
- 原型：

``` java
    /**
    * 下载语音文件
    * @param fromUrl 语音文件URL
    * @param savePath 语音文件的本地存放地址,带文件名的全路径  
    * @param callback 下载文件回调 
    */
    public void downloadFileByUrl( String fromUrl, String savePath, YIMEventCallback.DownloadByUrlCallback callback )
```

- 回调参数：

``` java
public interface DownloadByUrlCallback {

    /**
     * 下载文件回调
     *
     * @param errorcode 错误码
     * @param fromUrl  文件url
     * @param savePath  文件存放路径
     */
     public void onDownloadByUrl(int errorcode, String fromUrl,  String savePath);
}        
```

### 消息屏蔽
#### 屏蔽/解除屏蔽用户消息 
若屏蔽用户的消息，此屏蔽用户发送的私聊/频道消息都接收不到。

- 原型：

``` java
    /*
     * 功能：屏蔽/解除屏蔽用户消息
     * @param userID：用户ID
     * @param block：true-屏蔽 false-解除屏蔽
     * @param callback 屏蔽用户回调
     */
     public void blockUser(String userID, boolean block, YIMEventCallback.ResultCallback<BlockUserInfo> callback)
```

- 回调参数：

``` java
public interface ResultCallback<BlockUserInfo> {
    /**
     * @param blockInfo 屏蔽信息    
     */
     public void onSuccess(BlockUserInfo blockInfo);
  
    /**
     * @param errorcode 错误码   
     * @param blockInfo 屏蔽信息 
     */
     public void onFailed(int errorcode, BlockUserInfo blockInfo);
}
```  

- 备注：
  `BlockUserInfo`类成员方法如下：
  `getUserID()`: 用户ID， String类型
  `getBlock()`: 是否屏蔽，true-屏蔽 false-解除屏蔽，boolean类型   

#### 解除所有已屏蔽用户 
- 原型：

``` java
    /*
     * 功能：解除所有已屏蔽用户
     * @param callback 解除用户屏蔽回调
     */
    public void unBlockAllUser(YIMEventCallback.OperationCallback callback)
```

- 回调参数：

``` java
public interface OperationCallback {
   
    public void onSuccess();
  
   /**
    * 解除所有已屏蔽用户回调
    * @param errorcode 错误码   
    */
    public void onFailed(int errorcode);
}    
```
  
#### 获取被屏蔽消息用户 
- 原型：

  ``` java
     /*
      * 功能：获取被屏蔽消息用户
      * @param callback 获取屏蔽用户回调
      */
      public void getBlockUsers(YIMEventCallback.ResultCallback<ArrayList<String>> callback)
  ```

- 回调参数：

``` java
public interface ResultCallback<ArrayList<String>> {
    
   /**
    * @param userList 用户ID列表    
    */
    public void onSuccess(ArrayList<String> userList);
  
   /**
    * @param errorcode 错误码   
    * @param userList 用户ID列表 
    */
    public void onFailed(int errorcode, ArrayList<String> userList);
}
```

### 消息功能设置   
#### 设置消息已读

- 功能：收到消息后，设置消息为已读。

- 原型：

  ``` java
     /**
      *
      * @param messageID 消息ID
      * @param read 是否已读，`true`为已读，`false`为未读。
      * @return
      */
      public int setMessageRead(long messageID, boolean read)
  ```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义) 
  
#### 批量设置消息已读

- 功能：收到消息后，设置消息为已读。

- 原型：

  ``` java
     /**
      *
      * @param userID 发消息的用户ID；用户ID为空字符串时，将登录用户的所有消息设置为已读/未读；用户ID不为空字符串时，将接收的由该用户ID发送的消息设置为已读/未读
      * @param read 是否已读，`true`为已读，`false`为未读。
      * @return
      */
      public int setAllMessageRead(String userID, boolean read)
  ```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

#### 切换获取消息模式 

- 功能：在自动接收消息和手动接收消息间切换，默认是自动接收消息。
- 原型：

``` java
    /**
     *
     * @param targets 频道ID，可以指定多个
     * @param autoReceive `true`为自动接收消息，`false`为手动接收消息，默认为`true`。
     * @return
     */
    public int setReceiveMessageSwitch(List<String> targets, boolean autoReceive)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

#### 手动获取消息 

- 功能：在手动接收消息模式，需要调用该接口后才能收到 `OnRecvMessage` 通知。
- 原型：

``` java
    /**
     *
     * @param targets 指定拉取消息的频道ID，可以指定多个
     * @return
     */
    public int getNewMessage(List<String> targets)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)
- 备注：这是一个异步操作，操作结果会通过回调接口返回

#### 新消息通知 

- 功能：setReceiveMessageSwitch(false)后，进入手动接收消息模式，有新消息 的时候会通知该回调，频道消息会通知消息来自哪个频道ID

- 原型：

``` java
public interface MessageEventCallback {
    /**
     *
     * @param chatType 聊天类型
     * @param targets 频道ID     
     */
    public void onRecvNewMessage( int chatType, String targetID);
}    
```

### 消息记录管理
#### 设置是否保存频道聊天记录 

- 功能：设置是否在本地保存频道聊天记录，默认不保存。私聊历史记录默认保存。
- 原型：

``` java
   /**
     *
     * @param roomIDs 频道id，指定要保存本地历史记录的频道ID，可以一次设置多个，对应 `JoinChatRoom` 指定的ID。
     * @param isSave `true`表示保存聊天记录，`false`表示不保存聊天记录。
     * @return
     */
    public int setRoomHistoryMessageSwitch(List<String> roomIDs,boolean isSave)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

#### 拉取频道最近聊天记录 

- 功能：从服务器拉取频道最近的聊天历史记录。这个功能默认不开启，需要的请联系我们修改服务器配置。联系我们，可以通过专属游密支持群或者技术支持的大群。这是一个异步操作，操作结果会通过回调接口返回。

- 原型：

  ``` java
     /**
      *
      * @param roomID 频道ID
      * @param count 消息数量(最大200条)
      * @param direction 历史消息排序方向 0：按时间戳升序	1：按时间戳逆序
      * @param callback 查询历史记录回调
      */
      public void queryRoomHistoryMessageFromServer(String roomID, int count, int direction, YIMEventCallback.ResultCallback<RoomHistoryMsgInfo> callback)
  ```

- 回调参数：

``` java
public interface ResultCallback<RoomHistoryMsgInfo> {  
    
    /**
     * @param msgInfo 频道消息记录    
     */
     public void onSuccess(RoomHistoryMsgInfo msgInfo);
  
    /**
     * @param errorcode 错误码   
     * @param msgInfo 频道消息记录 
     */
     public void onFailed(int errorcode, RoomHistoryMsgInfo msgInfo);
}
```

- 备注：
  `RoomHistoryMsgInfo`类成员方法如下:
  `getRoomID()`: 获取频道ID，String类型
  `getRemain()`: 获取剩余消息记录数量，int类型
  `getMessageList()`: 获取消息记录列表，ArrayList<YIMMessage>
  `YIMMessage`类详细定义查看[接收消息](#接收消息)备注

#### 本地历史记录管理
##### 查询本地聊天历史记录 

- 功能: 获取本地消息历史记录，这是一个异步操作，操作结果会通过回调接口返回
- 原型:

``` java
    /**
     *
     * @param targetID 私聊用户ID或频道ID
     * @param chatType 表示查询私聊或者频道聊天的历史记录
     * @param startMessageID 起始历史记录消息id（与requestid不同），为`0`表示首次查询，将倒序获取`count`条记录
     * @param count 最多获取多少条
     * @param direction 历史记录查询方向，`startMessageID=0`时,direction使用默认值0；`startMessageID>0`时，`0`表示查询比`startMessageID`小的消息，`1`表示查询比`startMessageID`大的消息
     * @param callback 查询本地历史记录回调
     */
     public void queryHistoryMessage(String targetID, int chatType, long startMessageID, int count, int direction, YIMEventCallback.ResultCallback<YIMHistoryMessage> callback)
```

- 相关函数：
  对于房间本地记录，需要先设置自动保存房间消息。`setRoomHistoryMessageSwitch`

- 回调参数：

``` java
public interface ResultCallback<YIMHistoryMessage> {   
   /**
    * @param msgInfo 历史消息记录    
    */
    public void onSuccess(YIMHistoryMessage msgInfo);
  
   /**
    * @param errorcode 错误码   
    * @param msgInfo 历史消息记录 
    */
    public void onFailed(int errorcode, YIMHistoryMessage msgInfo);
}
```

- 备注：
  `YIMHistoryMessage`类成员变量如下：
  `targetID`：用户ID/频道ID，String型
  `remain`：剩余历史消息条数，int型
  `messageList`：消息列表，YIMHistoryMessageBody型
  
  `YIMHistoryMessageBody`类成员方法如下：
  `getSenderID()`：返回消息发送者ID，String型
  `getReceiveID()`：返回消息接收者ID，String型
  `getMessageID()`：返回消息ID，Long型
  `getChatType()`：返回聊天类型，私聊/频道聊天，int型
  `getMessageType()`：返回消息类型，0-未知类型，1-文本消息，2-自定义消息，3-表情，         4-图片，5-语音，6-视频，7-文件，8-礼物  
  `getText()`：返回文本消息内容/语音消息的文本识别内容，String型  
  `getLocalPath()`：返回语音消息文件的本地路径，String型
  `getCreateTime()`：返回消息收发时间，int型  
  `getIsRead()`：返回消息是否已读，boolean型
  `getParam()`：如果是语音消息，返回语音消息的自定义附加参数
  `getDuration()`：如果是语音消息，返回语音消息时长，int型  
  `getCustomMesssageContent()`：如果是自定义消息，返回自定义消息内容，byte[]型
  `getVoiceToText()`：返回语音消息的文本识别内容，String型
  `getFileName()`：如果是文件消息，返回文件名，String型
  `getFileExtension()`：如果是文件消息，返回文件扩展信息，String型
  `getFileSize()`：如果是文件消息，返回文件大小，int型
  `getIsPlayed()`：如果是语音消息，返回语音消息是否已播放，boolean型

##### 根据时间清理本地聊天历史记录 

- 功能：清理本地历史记录
- 原型：

``` java
    /**
     *
     * @param chatType 指定是清理频道消息还是私聊消息
     * @param time Unix timestamp,精确到秒，表示删除这个时间点之前的所有历史记录
     * @return
     */
    public int deleteHistoryMessageBeforeTime(int chatType, long time)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)
- 备注：建议定期清理本地历史记录。

##### 根据消息ID清理本地聊天历史记录 

- 功能：清理本地历史记录
- 原型：

``` java
    /**
     *
     * @param messageID 将删除指定消息ID的历史记录，对应 `YIMEngine.HistoryMsg` 的 `MessageID` 属性值
     * @return
     */
    public int deleteHistoryMessageByID(long messageID)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)
- 备注：建议定期清理本地历史记录。

##### 根据用户ID清理本地私聊历史记录 

- 功能：根据用户ID删除对应的本地私聊历史记录，保留消息ID白名单中的消息记录,白名单列表为空时删除与该用户ID间的所有本地私聊记录。
- 原型：

``` java
    /**
     *
     * @param targetID 用户ID
     * @param chatType 指定清理私聊消息
     * @param excludeMesList 与该用户ID的私聊消息ID白名单
     * @return
     */
    public int deleteSpecifiedHistoryMessage(String targetID, int chatType, long[] excludeMesList)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)
- 备注：建议定期清理本地历史记录。

##### 根据用户ID或者房间ID清理本地聊天历史记录 

- 功能：根据用户ID或者房间ID删除以指定的起始消息ID开始的n条本地私聊历史记录，起始消息ID和消息数量使用默认值时则删除所有消息。
- 原型：

``` java
    /**
     *
     * @param targetID 用户ID或者房间ID
     * @param chatType 聊天类型
     * @param startMessageID 起始消息ID(默认值0表示最近的一条消息)
     * @param count 消息数量(默认值0表示删除所有消息)
     * @return
     */
    public int deleteHistoryMessageByTarget(String targetID, int chatType, long startMessageID, int count)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)
- 备注：建议定期清理本地历史记录。

##### 获取最近私聊联系人列表 

- 功能：该接口是根据本地历史消息记录生成的最近联系人列表，按最后聊天时间倒序排列。该列表会受清理历史记录消息的接口影响。这是一个异步操作，操作结果会通过回调接口返回。
- 原型：

``` java
    /**
     * 获取最近联系人
     * @param callback 获取最近联系人回调
     */
    public void getHistoryContact(YIMEventCallback.ResultCallback<ArrayList<ContactsSessionInfo>> callback)
```

- 回调参数：

``` java
public interface ResultCallback<ArrayList<ContactsSessionInfo>>{
   /**
    * @param contactLists 最近联系人列表    
    */
    public void onSuccess(ArrayList<ContactsSessionInfo> contactLists);
  
   /**
    * @param errorcode 错误码   
    * @param contactLists 最近联系人列表 
    */
    public void onFailed(int errorcode, ArrayList<ContactsSessionInfo> contactLists);
}    
```

- 备注：
  `ContactsSessionInfo`类成员变量如下：
  `strContactID`：联系人ID
  `iMessageType`：消息类型，0-未知类型，1-文本消息，2-自定义消息，3-表情，4-图片，5-语音，6-视频，7-文件，8-礼物
  `strMessageContent`：消息内容
  `iCreateTime`：消息收发时间 
  `iNotReadMsgNum`：消息未读数量（此值有意义需结合SetMessageRead接口使用，可在收到消息后设置消息已读）
  `strLocalPath`：如果是语音或者文件消息，本地存放路径
  `strExtraInfo`：如果是语音或者文件消息，额外信息(FileSize,FileName,FileExtension,Param等Key)

### 高级功能
#### 文本翻译 
- 功能：将文本翻译成指定语言的文本，这是一个异步操作，操作结果会通过回调接口返回
- 原型：

```java
    /**
     *  文本翻译
     *  @pararm text 原语言字符串
     *  @param destLangCode 目标语言编码 LanguageCode
     *  @param srcLangCode 原语言编码 LanguageCode
      */
    public void translateText( String text, int destLangCode, int srcLangCode, YIMEventCallback.ResultCallback<TranlateTextInfo> callback)
```

- 回调参数：

``` java
public interface ResultCallback<TranlateTextInfo> {
      
   /**
    * @param translateTextInfo 文本翻译信息    
    */
    public void onSuccess(TranlateTextInfo translateTextInfo);
  
   /**
    * @param errorcode 错误码   
    * @param translateTextInfo 文本翻译信息 
    */
    public void onFailed(int errorcode, TranlateTextInfo translateTextInfo);
}
``` 

- 备注：
  `TranlateTextInfo`: 类成员变量如下：
  `requestID`: 消息ID
  `text`: 翻译结果
  `srcLangCode`: 翻译原语言编码 可查看LanguageCode枚举，Integer类型
  `destLangCode`: 翻译目标语言编码 可查看LanguageCode枚举，Integer类型 
 
#### 敏感词过滤 
考虑到客户端可能需要传递一些自定义消息，关键字过滤方法就直接提供出来，客户端可以选择是否过滤关键字，并且可以根据匹配到的等级能进行广告过滤。比如`level`返回的值为'2'表示广告，那么可以在发送时选择不发送，接收放可以自定义是否显示该消息。

- 功能：过滤敏感词
- 原型：

``` java
    /**
     *
     * @param strSource 原字符串
     * @param level 匹配到的敏感词等级会放到level对象的value里
     * @return 过滤后的字符串，敏感词替换为"**"
     */
    public String getFilterText(String strSource, IMEngine.IntegerVal level);
```

- 返回：
  过滤后的字符串，敏感词替换为"**"
  
## 游戏暂停与恢复
### 游戏暂停通知 
- 功能；建议游戏暂停时通知该接口，以便于得到更好重连效果；
调用onPause(false),在游戏暂停后依旧接收IM消息；
调用onPause(true),游戏暂停，即使IM是登录状态也不会接收IM消息；在游戏恢复运行时会主动拉取暂停期间未接收的消息，收到onRecvMessage()回调。
- 原型：

  ``` java
     /*
      * 功能：暂停
      * @param pauseReceiveMessage  是否暂停接收IM消息，true-暂停接收 false-不暂停接收      
      */
      public void onPause(bool pauseReceiveMessage);
  ```  

### 游戏恢复运行通知 
- 功能：建议游戏恢复运行时通知该接口，以便于得到更好重连效果
- 原型：

  ```java
     public void onResume();
  ```

## 用户举报和禁言
### 用户举报 

- 原型：


``` java
   /**
    * 举报
    * @param userID：被举报用户ID
    * @param source：来源（私聊/频道）
    * @param reason：原因，由用户定义枚举
    * @param description：原因描述
    * @param extraParam：附加信息JSON格式 ({"nickname":"","server_area":"","level":"","vip_level":""}）
    */
    public int accusation(String userID, int source, int reason, String description, String extraParam)
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

- 举报处理结果通知


``` java
public interface AccusationResultCallback {
   /**
    * 举报处理结果回调
    * @param result 处理结果，0-忽略，1-警告，2-禁言
    * @param userID 被举报用户
    * @param accusationTime 举报时间
    */
    public void onAccusationResultNotify(int result, String userID, int accusationTime);
}    
```
  
### 禁言查询 
查询所在频道的禁言状态。

- 原型：

  ``` java
      public void getForbiddenSpeakInfo(YIMEventCallback.ResultCallback<ArrayList<YIMForbiddenSpeakInfo>> callback)
  ```

- 回调参数：

``` java
public interface ResultCallback<ArrayList<YIMForbiddenSpeakInfo>> {
   /**
    * @param forbiddenInfoList 禁言状态列表,每一个代表一个频道的禁言状态    
    */
    public void onSuccess(ArrayList<YIMForbiddenSpeakInfo>  forbiddenInfoList);
  
   /**
    * @param errorcode 错误码   
    * @param forbiddenInfoList 禁言状态列表,每一个代表一个频道的禁言状态 
    */
    public void onFailed(int errorcode, ArrayList<YIMForbiddenSpeakInfo>  forbiddenInfoList);
 }    
 ```
 
 - 备注：
  `YIMForbiddenSpeakInfo` 类成员方法如下：
  `getChannelID()`：返回频道ID，String类型
  `getIsForbidRoom()`：返回此频道是否被禁言，true-被禁言，false-未被禁言，boolean类型
  `getReasonType()`：返回禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它，int类型
  `getEndTime()`：若被禁言，返回禁言结束时间，long型

## 公告管理
基本流程：申请开通公告功能->后台添加新公告然后设置公告发送时间,公告消息类型,发送时间,接收公告的频道等。->(客户端流程) 设置对应监听-> 调用对应接口->回调接收
### 公告功能简述
1.公告发送由后台配置，如类型、周期、发送时间、内容、链接、目标频道、次数、起始结束时间等。
2.公告三种类型：跑马灯，聊天框，置顶公告，(1) 跑马灯，聊天框公告可设置发送时间，次数和间隔（从指定时间点开始隔固定间隔时间发送多次，界面展示及显示时长由客户端决定）；(2) 置顶公告需设置开始和结束时间（该段时间内展示）。
3.三种公告均有一次性、周期性两种循环属性：  一次性公告，到达指定时间点，发送该条公告;  周期性公告，跟一次性公告发送规则一致，但是可以设置发送周期（在每周哪几天的指定时间发送）。
  4.跑马灯与聊天框公告只有发送时间点在线的用户才能收到该公告，显示规则由客户端自己决定，两者区别主要是界面显示的区分。
5.置顶公告有显示起始和结束时间，表示该时段内显示，公告发送时间点在线的用户会收到该公告，公告发送时间点未在线用户，在公告显示时段登录，登录后可通过查询公告接口查到该公告。
6.公告撤销仅针对置顶公告，公告显示时段撤销公告，客户端会收到公告撤销通知，界面进行更新。

### 接收公告 
管理员在后台发布公告，当到达指定时间会收到该公告，界面根据不同类型的公告进行展示。

- 原型：

``` java
public interface NoticeCallback {
    /**
     * 接收公告回调
     * @param notice 公告信息
     */
     public void onRecvNotice(NoticeInfo notice);
}
```

- 备注：
  `NoticeInfo`类成员变量如下：
  `iNoticeID`：公告ID，Long型
  `iNoticeType`：公告类型，Integer型
  `strChannelID`：发布公告的频道ID，String型
  `strContent`：公告内容，String型
  `strLinkText`：公告链接关键字，String型
  `strLinkAddr`：公告链接地址，String型
  `iBeginTime`：公告开始时间，Integer型
  `iEndTime`：公告结束时间，Integer型

### 撤销公告 
对于某些类型的公告（如置顶公告），需要在界面展示一段时间，如果管理员在该时间段执行撤销该公告，会收到撤销公告通知，界面进行相应更新。

- 原型：

  ``` java
  public interface NoticeCallback {
     /**
      * 撤销公告（置顶公告）
      * @param noticeID 公告ID
      * @param channelID 发布公告的频道ID
      */
      public void onCancelNotice(long noticeID, String channelID);
  }    
  ```

### 查询公告 
公告在配置的时间点下发到客户端，对于某些类型的公告（如置顶公告）需要在某个时间段显示在，如果用户在公告下发时间点未在线，而在公告展示时间段内登录，应用可根据自己的需要决定是否展示该公告，`queryNotice`查询公告，结果通过上面的`onRecvNotice`异步回调返回。

- 原型：

  ``` java
      public int queryNotice()
  ```
  
- 返回值：
  错误码，详细描述见[错误码定义](#错误码定义)  
  
## 地理位置
### 获取自己当前地理位置 
- 功能：获取用户当前地理位置，这是一个异步操作，操作结果会通过回调接口返回。
- 原型：

``` java
   /**
    * 获取地理定位
    * @return
    */
    public int getCurrentLocation()
```

- 返回：
  错误码，详细描述见[错误码定义](#错误码定义)

- 异步回调接口：

``` java
public interface GetLocationCallback
{
    /**
     * 获取自己位置回调
     * @param errorcode 错误码 ，详细描述见错误码定义
     * @param location 当前地理位置及行政区划信息。
     */

     public void onUpdateLocation(int errorcode, GeographyLocation location);
}
```

- 备注：
  `GeographyLocation`类成员变量如下：
  `iDistrictCode`：int类型，地理区域编码，可根据此参数组合附近频道id
  `strCountry`：String类型，国家
  `strProvince`：String类型，省份
  `strCity`：String类型，城市
  `strDistrictCounty`：String类型，区县
  `strStreet`：String类型，街道
  `fLongitude`：double类型，经度
  `fLatitude`：double类型，纬度

### 获取附近的人 
- 条件：需要开通LBS定位服务才能使用
- 功能：获取附近的目标(人 房间) ，若已开启服务，此功能生效的前提是自己和附近的人都获取了自己的地理位置，即调用了IM的获取当前地理位置接口。
- 原型：

```java
  /**
   * 获取附近的目标  
   * @param count: 获取附近的目标数量（一次最大200）
   * @param serverAreaID：区服（对应设置用户信息中的区服，如果只需要获得本服务器的，要填；否则填""）
   * @param districtlevel：行政区划等级，一般填0，0-未知，1-国家，2-省份，3-城市，4-区县，5-街道
   * @param resetStartDistance：是否重置查找起始距离（是否从距自己0米开始查找）
   * @param callback  获取附近人回调
   */
   public void getNearbyObjects(int count, String serverAreaID, int districtlevel , boolean resetStartDistance, YIMEventCallback.ResultCallback<RelativeLocationInfo> callback )
```

- 回调参数：

``` java
public interface ResultCallback<RelativeLocationInfo>
{
    
   /**
    * @param locationInfo 位置信息    
    */
    public void onSuccess(RelativeLocationInfo  locationInfo);
  
   /**
    * @param errorcode 错误码   
    * @param locationInfo 位置信息  
    */
    public void onFailed(int errorcode, RelativeLocationInfo  locationInfo);
}
```

- 备注：
  `RelativeLocationInfo`:类成员如下
  `startDistance`: 查找起始距离，int类型
  `endDistance`: 查找终止距离，int类型
  `relativeLocations`: 附近的人地理位置信息列表，ArrayList<RelativeLocation>类型
  
  `RelativeLocation`类成员变量如下：
  `iDistance`: 与自己的距离，int类型
  `fLongitude`: 经度，double类型
  `fLatitude`: 纬度，double类型
  `strUserID`: 附近用户ID，String类型
  `strCountry`: 国家，String类型
  `strProvince`: 省份，String类型
  `strCity` 城市，String类型
  `strDistrictCounty` 区县，String类型
  `strStreet` 街道，String类型

### 地理位置更新 
- 功能：从资源和耗电方面的考虑，SDK不自动监听地理位置的变化，如果调用方有需要可调用`setUpdateInterval`接口，设置更新时间间隔，SDK会按设定的时间间隔监听位置变化并通知上层。（如果应用对地理位置变化关注度不大，最好不要设置自动更新）

- 原型：

``` java
   /**
    * @param interval 更新间隔时间 单位：分钟
    */
    public void setUpdateInterval(int interval)
```

### 获取与指定用户距离  
获取与指定用户距离之前，需要调用`getCurrentLocation`成功获取自己的地理位置,指定的用户也调用`getCurrentLocation`成功获取其地理位置。

- 原型：

  ``` java
     /**
      * @param userID 用户ID
      * @param callback  获取距离回调
      */      public void getDistance(String userID, YIMEventCallback.ResultCallback<UserDistanceInfo> callback)
  ```
  
- 回调参数：

  ``` java
  public interface ResultCallback<UserDistanceInfo>
  {     
      /**
       * @param distanceInfo 距离信息    
       */
       public void onSuccess(UserDistanceInfo distanceInfo);
  
      /**
       * @param errorcode 错误码   
       * @param distanceInfo 距离信息  
       */
       public void onFailed(int errorcode, UserDistanceInfo distanceInfo);
  }  
  ```
  
- 备注：  
  `UserDistanceInfo`: 类成员变量如下
  `userID`: 用户ID, String类型
  `distance`: 距离（米），int类型  

## 关系链管理
### 用户信息管理
#### 通知用户资料变更
当好友的用户资料变更时会收到此通知，使用方根据需要决定是否重新获取资料变更好友的用户信息。

- 原型

  ``` java
  public interface UserProfileChangeCallback {
     /**
      * 用户信息变更通知
      * @param userID 用户资料变更好友的用户ID    
      */ 
      public void onUserInfoChangeNotify(String userID);
  }
  ``` 

#### 设置用户信息
设置用户的基本资料，昵称，性别，个性签名，地理位置等。

- 原型

  ``` java
  /**
   * 设置用户信息
   * @param userInfo 用户基本信息   
   * @param callback 设置用户资料回调
   */
  public void setUserProfileInfo(YIMUserSettingInfo userInfo, YIMEventCallback.OperationCallback callback)
  ```
    
- 备注：
  `YIMUserSettingInfo`类成员如下：  
  `nickName`：String类型，昵称，长度在64bytes以内
  `sex`：Integer类型，性别，0-未知性别，1-男性，2-女性
  `signature`：String类型，个性签名，长度在120bytes以内
  `country`：String类型，国家  
  `province`：String类型，省份
  `city`：String类型，城市
  `extraInfo`：String类型，扩展信息，JSON格式   
    
- 回调参数：

  ``` java 
  public interface OperationCallback {
     
      public void onSuccess();
  
     /**
      * @param errorcode 错误码   
      */
      public void onFailed(int errorcode);
  }
  ```
    
#### 查询用户基本资料
查询用户的基本资料，昵称，性别，个性签名，头像，地理位置，被添加权限等。

- 原型

  ``` java
  /**
   * 查询用户信息
   * @param userID 用户ID   
   * @param callback 查询用户基本资料回调
   */ 
  public void getUserProfileInfo(String userID, YIMEventCallback.ResultCallback<UserProfileInfo> callback)
  ```
    
- 回调参数：

  ``` java  
  public interface ResultCallback<UserProfileInfo> {
    
     /**
      * @param userInfo 用户资料    
      */
      public void onSuccess(UserProfileInfo userInfo);
  
     /**
      * @param errorcode 错误码   
      * @param userInfo 用户资料  
      */
      public void onFailed(int errorcode, UserProfileInfo userInfo);
  }
  ```
    
- 备注：
  `UserProfileInfo`类成员如下：  
  `userID`：String类型，用户ID
  `photoUrl`：String类型，头像url
  `onlineState`：int类型，在线状态，0-在线,1-离线                
  `beAddPermission`：int类型，被好友添加权限，0-不允许被添加,1-需要验证,                   
                   2-允许被添加，不需要验证  
  `foundPermission`：int类型，被查找时是否显示被添加权限，
                    0-被其它用户查找时显示自己的被添加权限，
                    1-被其它用户查找时不显示自己的被添加权限                    
  `nickName`：String类型，昵称
  `sex`：int类型，性别，0-未知性别，1-男性，2-女性
  `signature`：String类型，个性签名
  `country`：String类型，国家  
  `province`：String类型，省份
  `city`：String类型，城市
  `extraInfo`：String类型，扩展信息，JSON格式
  
#### 设置用户头像

- 原型

  ``` java
  /**
   * 设置用户头像
   * @param photoPath：本地图片绝对路径，url长度在500bytes内，图片大小在100kb内   
   * @param callback 设置头像回调
   */ 
  public void setUserProfilePhoto(String photoPath, YIMEventCallback.ResultCallback<String> callback)
  ```

- 回调参数：

  ``` java 
  public interface ResultCallback<String> {
     
     /**
      * @param photoUrl：头像的下载URL   
      */
      public void onSuccess(String photoUrl);
  
     /**
      * @param errorcode 错误码   
      * @param photoUrl：头像的下载URL 
      */
      public void onFailed(int errorcode, String photoUrl);
  }
  ```
    
#### 切换用户状态

- 原型

  ``` java
  /**
   * 切换用户在线状态
   * @param userID：用户ID
   * @param userStatus：用户在线状态，0-在线，1-离线，2-隐身
   * @param callback 切换在线状态回调
   */  
  public void switchUserStatus(String userID, int userStatus, YIMEventCallback.OperationCallback callback)
  ```

- 回调参数：

  ``` java 
  public interface OperationCallback {
     
      public void onSuccess();
  
     /**
      * @param errorcode 错误码   
      */
      public void onFailed(int errorcode);
  }
  ```
  
#### 设置好友添加权限

- 原型

  ``` java
  /**
   * 设置好友添加权限
   * @param beFound：能否被别人查找到，true-能被查找，false-不能被查找
   * @param beAddPermission：被其它用户添加的权限，0-不允许被添加，1-需要验证，2-允许被添加，不需要验证
   * @param callback 设置添加权限回调
   */ 
  public void setAddPermission(boolean beFound, int beAddPermission, YIMEventCallback.OperationCallback callback)
  ```

- 回调参数：

  ``` java 
  public interface OperationCallback {   
      public void onSuccess();
  
     /**
      * @param errorcode 错误码   
      */
      public void onFailed(int errorcode);
  }
  ```
  
### 好友管理  
#### 查找好友
查找将要添加的好友，获取该好友的简要信息。

- 原型

  ``` java
  /**
   * 查找好友
   * @param findType：查找类型，0-按ID查找，1-按昵称查找  
   * @param target：对应查找类型选择的用户ID或昵称 
   * @param callback 查找好友回调
   */ 
  public void findUser(int findType, String target, YIMEventCallback.ResultCallback<ArrayList<YIMUserBriefInfo>> callback)
  ```

- 回调参数：

  ``` java  
  public interface ResultCallback<ArrayList<YIMUserBriefInfo>> {
     
     /**
      * @param userList：用户简要信息列表   
      */
      public void onSuccess(ArrayList<YIMUserBriefInfo> userList);
  
     /**
      * @param errorcode 错误码   
      * @param userList：用户简要信息列表 
      */
      public void onFailed(int errorcode, ArrayList<YIMUserBriefInfo> userList);
  }
  ```

- 备注：
  `YIMUserBriefInfo`类成员如下：  
  `userID`：String类型，用户ID
  `nickName`：String类型，用户昵称
  `userStatus`：int类型，在线状态，0-在线，1-离线，2-隐身     
       
#### 添加好友

- 原型

  ``` java
  /**
   * 添加好友
   * @param users：需要添加为好友的用户ID列表  
   * @param comments：备注或验证信息，长度最大128bytes 
   * @param callback 请求添加好友回调
   */ 
  public void requestAddFriend(List<String> users, String comments, YIMEventCallback.ResultCallback<String> callback)
  ```

- 回调参数：

  ``` java  
  public interface ResultCallback<String> { 
     
     /**
      * @param userID：用户ID    
      */
      public void onSuccess(String userID);
  
     /**
      * @param errorcode 错误码   
      * @param userID：用户ID  
      */
      public void onFailed(int errorcode, String userID);
  }
  ```
 
- 备注：
  被请求方会收到被添加为好友的通知，被请求方如果设置了需要验证才能被添加，会收到下面的回调：
  
  ``` java  
  public interface FriendNotifyCallback { 
      /**
       * 被请求添加为好友通知（需要验证）
       * @param userID：请求方的用户ID  
       * @param comments：备注或验证信息 
       */
       public void onBeRequestAddFriendNotify(String userID, String comments);
  }
  ```     
      
  被请求方如果设置了不需要验证就能被添加，收到下面的回调：
  
  ``` java 
  public interface FriendNotifyCallback {
     /**
      * 被请求添加为好友通知（不需要验证）
      * @param userID：请求方的用户ID  
      * @param comments：备注或验证信息 
      */  
      public void onBeAddFriendNotify(String userID, String comments);
  }
  ```      
  
#### 处理好友请求
当前用户有被其它用户请求添加为好友的请求时，处理添加请求。

- 原型

  ``` java
  /**
   * 处理被添加为好友的请求
   * @param userID：请求方的用户ID 
   * @param dealResult：处理结果，0-同意，1-拒绝 
   * @param callback 处理请求回调
   */
  public void dealBeRequestAddFriend(String userID, int dealResult, YIMEventCallback.ResultCallback<YIMFriendDealResult> callback)
  ```

- 回调参数：

  ``` java 
  public interface ResultCallback<YIMFriendDealResult> {
     
     /**
      * @param  dealResultInfo：处理被请求添加好友结果信息    
      */
      public void onSuccess(YIMFriendDealResult dealResultInfo);
  
     /**
      * @param errorcode 错误码   
      * @param  dealResultInfo：处理被请求添加好友结果信息  
      */
      public void onFailed(int errorcode, YIMFriendDealResult  dealResultInfo);
  }
  ``` 
  
- 备注：
 
   如果被请求方设置了需要验证才能被添加为好友，在被请求方成功处理了请求方的好友请求后，请求方能收到添加好友请求结果的通知，收到下面的回调：
   
  ``` java 
  public interface FriendNotifyCallback {
      /**
       * 请求添加好友结果通知(需要好友验证，待被请求方处理后回调)   
       * @param userID：请求方的用户ID
       * @param comments：备注或验证信息 
       * @param dealResult：处理结果，0-同意，1-拒绝 
       */   
       public void onRequestAddFriendResultNotify(String userID, String comments, Interger dealResult);
  }
  ```  
  
  `YIMFriendDealResult`: 类成员变量如下
  `userID`: 请求方的用户ID, String类型
  `comments`: 备注或验证信息，String类型
  `dealResult`: 处理结果，0-同意，1-拒绝 ，int类型
    
#### 删除好友

- 原型

  ``` java
  /**
   * 删除好友
   * @param users：需删除的好友用户ID列表 
   * @param deleteType：删除类型，0-双向删除，1-单向删除(删除方在被删除方好友列表依然存在) 
   * @param callback 删除好友回调
   */
  public void deleteFriend(List<String> users, int deleteType, YIMEventCallback.ResultCallback<String> callback)
  ```

- 回调参数：

  ``` java 
  public interface ResultCallback<String> {
     
     /**
      * @param userID：用户ID    
      */
      public void onSuccess(String userID);
  
     /**
      * @param errorcode 错误码   
      * @param userID：用户ID  
      */
      public void onFailed(int errorcode, String userID);
  }
  ```  
  
- 备注：
 
   如果删除方采用的是双向删除，被删除方能收到其被好友删除的通知，收到下面的回调：
   
  ``` java 
  public interface FriendNotifyCallback {
     /**
      * 被好友删除通知   
      * @param userID：删除方的用户ID    
      */ 
      public void onBeDeleteFriendNotify(String userID);
  }
  ```
   
#### 拉黑好友

- 原型

  ``` java
  /**
   * 拉黑好友
   * @param type：拉黑类型，0-拉黑，1-解除拉黑 
   * @param users：需拉黑的好友用户ID列表
   * @param callback 拉黑好友的回调
   */
  public void blackFriend(int type, List<String> users, YIMEventCallback.ResultCallback<YIMBlackFriendInfo> callback)
  ```

- 回调参数：

  ``` java
  public interface ResultCallback<YIMBlackFriendInfo> {  
     
     /**
      * @param blackInfo：拉黑用户相关信息    
      */
      public void onSuccess(YIMBlackFriendInfo blackInfo);
  
     /**
      * @param errorcode 错误码   
      * @param blackInfo：拉黑用户相关信息 
      */
      public void onFailed(int errorcode, YIMBlackFriendInfo blackInfo);
  }
  ```    
  
- 备注：
  `YIMBlackFriendInfo`: 类成员方法如下
  `type`: 拉黑类型，0-拉黑，1-解除拉黑, int类型
  `userID`: 被拉黑方用户ID，String类型   
  
#### 查询好友列表
查询当前用户已添加的好友，也能查找被拉黑的好友。

- 原型

  ``` java
  /**
   * 查询我的好友
   * @param type：查找类型，0-正常状态的好友，1-被拉黑的好友 
   * @param startIndex：起始序号，作用为分批查询好友（例如：第一次从序号0开始查询30条，第二次就从序号30开始查询相应的count数）
   * @param count：数量（一次最大100）
   * @param callback 查询我的好友回调
   */
  public void queryFriends(int type, int startIndex, int count, YIMEventCallback.ResultCallback<YIMFriendListInfo> callback)
  ```

- 回调参数：

  ``` java 
  public interface ResultCallback<YIMFriendListInfo> {
      
     /**
      * @param friendListInfo：好友列表信息    
      */
      public void onSuccess(YIMFriendListInfo friendListInfo);
  
     /**
      * @param errorcode 错误码   
      * @param friendListInfo：好友列表信息 
      */
      public void onFailed(int errorcode, YIMFriendListInfo friendListInfo);
  }
  ```
  
- 备注：
  `YIMFriendListInfo`类成员如下：
  `type`: 查找类型，0-正常状态的好友，1-被拉黑的好友, int类型
  `startIndex`: 起始序号，int类型
  `userList`: 好友列表，ArrayList<YIMUserBriefInfo>类型
  
  `YIMUserBriefInfo`类成员如下：  
  `userID`：String类型，用户ID
  `nickName`：String类型，用户昵称
  `userStatus`：int类型，在线状态，0-在线，1-离线，2-隐身   
  
#### 查询好友请求列表
查询当前用户收到的被添加请求的好友列表。

- 原型

  ``` java
  /**
   * 查询好友请求列表   
   * @param startIndex：起始序号，作用为分批查询好友请求（例如：第一次从序号0开始查询10条，第二次就从序号10开始查询相应的count数）
   * @param count：数量（一次最大20）
   * @param callback 查询好友请求列表回调
   */
  public void queryFriendRequestList(int startIndex, int count, YIMEventCallback.ResultCallback<YIMFriendRequestInfoList> callback)
  ```  

- 回调：

  ``` java  
  public interface ResultCallback<YIMFriendRequestInfoList> {
     
     /**
      * @param requestInfoList：请求信息列表    
      */
      public void onSuccess(YIMFriendRequestInfoList requestInfoList);
  
     /**
      * @param errorcode 错误码   
      * @param requestInfoList：请求信息列表 
      */
      public void onFailed(int errorcode, YIMFriendRequestInfoList requestInfoList);
  }
  ```
  
- 备注：
  `YIMFriendRequestInfoList`类成员如下:
  `startIndex`: 起始序号, int类型
  `userList`: 好友请求列表，ArrayList<YIMFriendRequestInfo>类型
  
  `YIMFriendRequestInfo`类成员如下：  
  `askerID`：String类型，请求方ID
  `askerNickname`：String类型，请求方昵称
  `inviteeID`：String类型，受邀方ID
  `inviteeNickname`：String类型，受邀方昵称
  `validateInfo`：String类型，验证信息
  `addStatus`：int类型，好友请求状态，
                 0-添加完成，
                 1-添加失败，  
                 2-等待对方验证， 
                 3-等待我验证
  `createTime`：int类型，请求的发送或接收时间
  
## 服务器部署地区定义

``` java
YIMService.ServerZone{
    public final static int China = 0; // 默认中国

    public final static int Singapore = 1; // 新加坡
    public final static int America = 2; // 美国
    public final static int HongKong = 3; // 香港
    public final static int Korea = 4; // 韩国
    public final static int Australia = 5; // 澳洲
    public final static int Deutschland = 6; // 德国
    public final static int Brazil = 7; // 巴西
    public final static int India = 8; // 印度
    public final static int Japan = 9; // 日本
    public final static int Ireland = 10; // 爱尔兰
}
```
  
## 错误码定义

``` 
 java

	YIMConstInfo.ErrorCode{
    public final static int Success = 0;
    public final static int EngineNotInit = 1;
    public final static int NotLogin = 2;
    public final static int ParamInvalid = 3;
    public final static int TimeOut = 4;
    public final static int StatusError = 5;
    public final static int SDKInvalid = 6;
    public final static int AlreadyLogin = 7;
    public final static int ServerError = 8;
    public final static int NetError = 9;
    public final static int LoginSessionError = 10;
    public final static int NotStartUp = 11;
    public final static int FileNotExist = 12;
    public final static int SendFileError = 13;
    public final static int UploadFailed = 14;		//上传失败 一般都是网络限制上传了
    public final static int UsernamePasswordError = 15;
    public final static int UserStatusError = 16;
    public final static int MessageTooLong = 17;   //消息太长
    public final static int ReceiverTooLong = 18;  //接收方ID过长(检查房间名)
    public final static int InvalidChatType = 19;  //无效聊天类型(私聊、聊天室)
    public final static int InvalidReceiver = 20;  //无效用户ID（私聊接受者为数字格式ID）
    public final static int UnknowError = 21;
    public final static int InvalidAppkey = 22; //无效APPKEY
	public final static int ForbiddenSpeak = 23, //被禁言
	public final static int CreateFileFailed = 24;
	public final static int UnsupportFormat = 25; //不支持的文件格式
	public final static int ReceiverEmpty = 26;  //接收方为空
	public final static int RoomIDTooLong = 27;//房间名太长
	public final static int ContentInvalid = 28;//聊天内容严重非法
	public final static int NoLocationAuthrize = 29;//未打开定位权限
	public final static int UnknowLocation = 30;    //未知位置
    public final static int Unsupport  = 31;        //不支持该接口
    public final static int NoAudioDevice = 32;     //无音频设备
    public final static int AudioDriver = 33;       //音频驱动问题
    public final static int DeviceStatusInvalid = 34,       //设备状态错误
    public final static int ResolveFileError = 35,          //文件解析错误
    public final static int ReadWriteFileError = 36,        //文件读写错误
    public final static int NoLangCode = 37,                //语言编码错误
    public final static int TranslateUnable = 38,           //翻译接口不可用
    public final static int SpeechAccentInvalid = 39,       //语音识别方言无效
    public final static int SpeechLanguageInvalid = 40,     //语音识别语言无效
    public final static int HasIllegalText = 41;            //消息含非法字符
    public final static int AdvertisementMessage = 42;      //消息涉嫌广告
    public final static int AlreadyBlock = 43;              //用户已经被屏蔽
    public final static int NotBlock = 44;                  //用户未被屏蔽
	public final static int MessageBlocked = 45;            //消息被屏蔽
	public final static int LocationTimeout = 46;           //定位超时        
    public final static int NotJoinRoom = 47,               //未加入该房间
    public final static int LoginTokenInvalid = 48,         //登录token错误
    public final static int CreateDirectoryFailed = 49,     //创建目录失败
    public final static int InitFailed = 50;                //初始化失败
    public final static int Disconnect = 51;               //与服务器断开
    public final static int TheSameParam = 52;             //设置参数相同
    public final static int QueryUserInfoFail = 53;         //查询用户信息失败
    public final static int SetUserInfoFail = 54;           //设置用户信息失败
    public final static int UpdateUserOnlineStateFail = 55; //更新用户在线状态失败
    public final static int NickNameTooLong = 56;           //昵称太长(> 64 bytes)
    public final static int SignatureTooLong = 57;          //个性签名太长(> 120 bytes)
    public final static int NeedFriendVerify = 58;         //需要好友验证信息
    public final static int BeRefuse = 59;                 //添加好友被拒绝
    public final static int HasNotRegisterUserInfo = 60;   //未注册用户信息
    public final static int AlreadyFriend = 61;            //已经是好友
	public final static int NotFriend = 62;               //非好友
	public final static int NotBlack = 63;                 //不在黑名单中
    public final static int PhotoUrlTooLong = 64;         //头像url过长(>500 bytes)
    public final static int PhotoSizeTooLarge = 65;       //头像太大（>100 kb）
    public final static int ChannelMemberOverflow = 66;   // 达到频道人数上限
    //服务器的错误码
    public final static int ALREADYFRIENDS = 1000;
    public final static int LoginInvalid = 1001;

    //语音部分错误码
    public final static int PTT_Start = 2000;
    public final static int PTT_Fail = 2001;
    public final static int PTT_DownloadFail = 2002;
    public final static int PTT_GetUploadTokenFail = 2003;
    public final static int PTT_UploadFail = 2004;
    public final static int PTT_NotSpeech = 2005;
    public final static int PTT_DeviceStatusError = 2006,       //音频设备状态错误
    public final static int PTT_IsSpeeching = 2007,             //已开启语音
    public final static int PTT_FileNotExist = 2008,
    public final static int PTT_ReachMaxDuration = 2009,        //达到语音最大时长限制
    public final static int PTT_SpeechTooShort = 2010,          //语音时长太短
    public final static int PTT_StartAudioRecordFailed = 2011,  //启动语音失败
    public final static int PTT_SpeechTimeout = 2012,           //音频输入超时
    public final static int PTT_IsPlaying = 2013,               //正在播放
    public final static int PTT_NotStartPlay = 2014,            //未开始播放
    public final static int PTT_CancelPlay = 2015,              //主动取消播放
    public final static int PTT_NotStartRecord = 2016,          //未开始语音
    public final static int PTT_NotInit = 2017,                 // 未初始化
    public final static int PTT_InitFailed = 2018,              // 初始化失败
    public final static int PTT_Authorize = 2019,               // 录音权限
    public final static int PTT_StartRecordFailed = 2020,       // 启动录音失败
    public final static int PTT_StopRecordFailed = 2021,        // 停止录音失败
    public final static int PTT_UnsupprtFormat = 2022,          // 不支持的格式
    public final static int PTT_ResolveFileError = 2023,        // 解析文件错误
    public final static int PTT_ReadWriteFileError = 2024,      // 读写文件错误
    public final static int PTT_ConvertFileFailed = 2025,       // 文件转换失败
    public final static int PTT_NoAudioDevice = 2026,           // 无音频设备
    public final static int PTT_NoDriver = 2027,                // 驱动问题
    public final static int PTT_StartPlayFailed = 2028,         // 启动播放失败
    public final static int PTT_StopPlayFailed = 2029,          // 停止播放失败
    public final static int InvalidUserId = 3000,               // 无效的用户ID
    public final static int InvalidRoomId = 3001,               // 无效的房间ID
    public final static int InvalidPassword = 3002,             // 无效的密码
    public final static int InvalidStoragaePath = 3003,         // 无效的存储路径
    public final static int InvalidAppKey = 3004,               // 无效的appkey
    public final static int InvalidContext = 3005,              // 无效的上下文环境
    public final static int InvaliddSecretKey = 3006,           // 无效的secretkey
    
    public final static int Fail = 10000;
    }
```


