# 接入指引

### 1 概述
#### 1.1 Demo下载
[Demo地址：https://github.com/qwwersong/MWLiveDemo](https://github.com/qwwersong/MWLiveDemo "Demo地址：")

#### 1.2 SDK下载
[SDK地址：https://github.com/qwwersong/MWLiveSDK](https://github.com/qwwersong/MWLiveSDK "SDK地址：")

### 2 下载及集成SDK
#### 2.1 SDK指引

**2.2.1 导入库**

Jar包：
![](http://data.eolinker.com/bixtBLD3ecc3c274c937d1aeae9427263b34db1c34d4e1f)
So库：
![](http://data.eolinker.com/kj8tQcYbd13dbdfd2a1151493ac1012983362df332d3a23)

>注意：目前So库只支持armeabi-v7a，SDK只提供jar包加so库的形式，后续会加入aar形式

**2.2.2 配置文件**

- 添加SDK引用。打开 app/build.gradle 文件，添加如下内容：

在 defaultConfig 节点添加 ndk 节点，指定支持的平台类型
```
ndk {
    abiFilters 'armeabi-v7a'
}
```

在 android 节点添加 sourceSets 节点，指定jni libs 目录

```
sourceSets {
    main {
        jniLibs.srcDirs = ['libs']
    }
}
```

//因为使用OpenGL绘制需要硬件加速
<uses-feature
	android:glEsVersion="0x00020000"
	android:required="true" />
//开启硬件加速
android:hardwareAccelerated="true"
```
- 添加混淆，添加如下内容：

```
-keep class com.montnets.mwlive.**{*;}
-keep class org.webrtc.**{*;}
-keep class nativeInterface.**{*;}
-keep class io.socket.**{*;}
```

### 3 功能实现流程
#### 3.1 初始化配置

```
LiveRoomConfig.sAppContext = this;
LiveRoom.getInstance().debug(true);
```

#### 3.2 聊天室功能
OnSocketStateListener：聊天室连接状态回调
OnReceivedMsgListener：聊天室接收消息回调
```
//userID    用户ID
//userName  用户名
//imgUrl    用户头像地址
IMUser user = new IMUser("userID", "userName", "imgUrl");
//roomIP	聊天室IP
//roomID	聊天室ID
LiveRoom.getInstance().loginRoom(roomIP, roomID, user, new OnSocketStateListener() {
					@Override
					public void onSuccess() {
						//聊天室登录成功
					}

					@Override
					public void onError(SocketException e) {
						switch (e.code) {
							case SocketException.ERROR_CONNECT:
								//聊天室登录失败... 重连中.....
								break;
							case SocketException.ERROR_CONNECT_TIME_OUT:
								//聊天室连接超时... 请检查网络....
								break;
							case SocketException.ERROR_RECEIVE_MSG_FAIL:
								//接收消息失败........
								break;
						}
					}
				}, new OnReceivedMsgListener() {
					@Override
					public void onReceivedMsg(JSONObject msg) {
						//收到消息
					}
				});
```

发送消息方法：

```
//发送信息
LiveRoom.getInstance().sendMsg(JSONObject msg);
```

>注意：聊天室回调返回聊天室状态和内容（JSONObject），发送内容（JSONObject）。

#### 3.3 播放器功能

两种方式：
一、通过Player对象控制播放器，需要传输Surface、Listener监听来实现
二、通过MWTextureView或者MWSurfaceView来实现，该类内部也是封装Player来实现的
（推荐使用MWTextureView来实现）
```
public class MWTextureView extends TextureView implements TextureView.SurfaceTextureListener {
    private SurfaceTexture surfaceTexture;
    private Player player;
    private int mScaleType;
    private Surface mSurface;

    public MWTextureView(Context context, AttributeSet attrs) {
        super(context, attrs);
        player = new Player(context, PlayerConstants.PLAYER_MW);
        player.setScaleType(PlayerConstants.VIDEO_SCALE_FIT);
        setSurfaceTextureListener(this);
    }

    @Override
    public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
        if (mSurface == null) {
            surfaceTexture = surface;
            mSurface = new Surface(surface);
            player.setSurface(mSurface);
            player.setScaleType(mScaleType);
        } else {
            setSurfaceTexture(surfaceTexture);
        }
    }

    @Override
    public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {
        surfaceTexture = surface;
        //必须要加这一句，在切换屏幕时，surface的绘制区域会改变，所以要根据surface重新获取绘制区域并设置
        mSurface = new Surface(surface);
        player.surfaceChange(mSurface);
        player.setScaleType(mScaleType);
    }

    @Override
    public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
        surfaceTexture = surface;
        return false;
    }

    @Override
    public void onSurfaceTextureUpdated(SurfaceTexture surface) {

    }

    /**
     * 设置播放监听
     * @param onPlayerListener 回调
     */
    public void setOnPlayerListener(OnPlayerListener onPlayerListener){
        player.setOnPlayerListener(onPlayerListener);
    }

    /**
     * 开始播放
     * @param playUrl 播放地址
     */
    public void startPlay(String playUrl){
        player.startPlay(playUrl, 0);
    }

    /**
     * 开始播放
     * @param uri   播放地址
     * @param startTime 在什么时间点播放，一般用于视频
     */
    public void startPlay(String uri, int startTime) {
        player.startPlay(uri, startTime);
    }

    /**
     * 停止播放
     */
    public void stopPlay() {
        player.stopPlay();
    }

    /**
     * 暂停播放
     */
    public void pausePlay() {
        player.pausePlay();
    }

    /**
     * 恢复播放
     */
    public void resumePlay() {
        player.resumePlay();
    }

    /**
     * 释放资源
     */
    public void release() {
        player.release();
    }

    /**
     * 设置缩放模式
     * @param scaleType 缩放状态
     */
    public void setScaleType(int scaleType){
        this.mScaleType = scaleType;
    }

    /**
     * 获取视频总时长
     * @return  总时长
     */
    public int getDuration(){
        return player.getDuration();
    }

    /**
     * 获取视频当前播放的时长
     * @return  当前时长
     */
    public int getCurrentTime(){
        return player.getCurrentTime();
    }

    /**
     * 调整播放进度
     * @param ms  要播放的时长
     */
    public void seekTo(int ms){
        player.seekTo(ms);
    }

    /**
     * 重置surface
     */
    public void reset(){
        player.surfaceChange(mSurface);
    }
	
	/**
     * 模式切换
     * @param playerType 播放器类型
     */
    public void setPlayerType(int playerType){
        player.setPlayerType(context, playerType);
    }
}
```
缩放模式：

| 类型  | 解释  |
| ------------ | ------------ |
| PlayerConstants.VIDEO_SCALE_ORIGIN  | 原始  |
| PlayerConstants.VIDEO_SCALE_FIT  | 适应  |
| PlayerConstants.VIDEO_SCALE_FILL  | 填充  |

```
public interface OnPlayerListener {

    /**
     * 播放异常
     */
    void onError(PlayException e);

    /**
     * 当前播放状态
     * @param code 播放状态码
     */
    void onPlayState(int code);

}
```
onPlayState：

| 类型  | 解释  |
| ------------ | ------------ |
| PlayerConstants.STATE_PLAYING  | 正在播放  |
| PlayerConstants.STATE_PAUSE  | 暂停  |
| PlayerConstants.STATE_FINISH  | 播放结束  |
| PlayerConstants.STATE_BUFFERING  | 缓冲中  |

onError：

| 类型  | 解释  |
| ------------ | ------------ |
| PlayException.VPC_OUT_OF_MEMORY  | 内存不足  |
| PlayException.VPC_NO_SOURCE_DEMUX  | 没有数据处理器  |
| PlayException.VPC_NETWORK_ERROR  | 网络错误  |
| PlayException.VPC_MEDIA_SPEC_ERROR  | 数据格式错误  |
| PlayException.VPC_NO_PLAY_OBJECT  | 无播放对象  |
| PlayException.VPC_NET_TIME_OUT  | 网络超时  |
| PlayException.MEDIA_PLAYER_ERROR  | MediaPlayer异常信息  |

#### 3.3.1 注意

直播模式： PlayerConstants.PLAYER_MW

回放模式： PlayerConstants.PLAYER_MEDIA
