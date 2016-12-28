# 动手做“虾米音乐”app-欢迎页

![demo](http://i.giphy.com/37c2VScSRIu5y.gif)
  
  事情起源于Wing神的爆炸酷炫的splash，大家讨论起各式启动页、欢迎页，其中一款勾起我与海伦童鞋的兴趣：滴滴打车首次启动的动画效果。  
经分析，有以下几种组成：
	
1. App启动时的第一张图：入口Activity设置Theme background，将图片设为启动页，
	like this：
	
	```
		<style name="AppLaunchTheme" parent="AppTheme">  
			<item name="android:background">@drawable/splash_layers</item>  
			<item name="android:backgroundDimEnabled">false</item>  
			<item name="android:windowNoTitle">true</item>  
			<item name="android:windowContentOverlay">@null</item>  
			<item name="android:windowFullscreen">true</item>  
		</style>
	```
    
2. 利用Path绘制的LOGO动画启动页。
3. 视频欢迎页。等等，真的视频吗？视频不是很占空间吗？会不会是个GIF啊？GIF去适配各类机型是不是很麻烦啊？![表情](http://img3.qianzhan123.com/news/201605/18/20160622-24fdc23afcadde9c_360x190.jpg)
####就是要搞事情

带着这些问题，我踏上了探(fan)索(bian)之(yi)路。拨开一层一层外衣，在raw文件夹中，一个.mp4文件露出了微笑
。经查看smali文件得证，LOGO动画页确实由PATH绘制而成，对此感兴趣的，可以参看大神张旭童的博客[《给我一个Path，还你一个酷炫动画》](http://blog.csdn.net/zxt0601/article/details/53040506)学习，我这里就是要搞视频，来教大家如何烧制这道菜：
    
![demo](http://i.giphy.com/sG4JKXkVpLv4k.gif)

    主料：
        VideoView 适量
       
    辅料：
        ViewPager 少许
        ImageView 若干
        TextView 适量
        视频文件 若干
        
    做法：
        1、将VideoView去皮洗净后切块备用；
        2、额...我只是个想做厨师的程序猿，但菜谱就到这里，我已经看到围观群众已经不耐烦的咬瓜皮了。
        
####正经做法

首先解包虾米APP，取出视频及图片元素，啊嘞！怎么又成虾米音乐的欢迎页了？说好的滴滴呢？别急别急，滴滴虾米音乐的欢迎页也是视频的，而且是多个视频滑动播放的，年末大酬宾，买一赠一啦！：
    
1. 由VideoView(全屏)+ImageView组成ViewPager的Item,绑定至Fragment；
2. 将Fragment装入FragmentStatePagerAdapter
3. 将adapter装载至viewPager；
4. 放入适量视频文件、图片素材等佐料后起锅...（好像又跑偏了啊喂），对于viewpager fragment这些基本组件，大家应该信手拈来了，我就说说视频文件如何播放的,翻开fragment，来看看每个item都有什么（敲黑板，这个是重点）：  

*布局：*

```
<?xml version="1.0" encoding="utf-8"?>  
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="fill_parent"  
	android:layout_height="fill_parent">  
	
    <com.watire.xiamivd.FullScreenVideoView
        android:id="@+id/vvSplash"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" />
        
    <ImageView
        android:id="@+id/ivSlogan"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="20dp"
        android:src="@drawable/slogan_1"
        android:scaleType="fitEnd"
        android:layout_alignParentEnd="true" />  
        
</RelativeLayout>

```

*fragment：*
          
```
	mVideoView = findViewById(R.id.vvSplash);
	mvSlogan = findViewById(R.id.ivSlogan);
	mVideoView.setOnErrorListener(this);
	mVideoView.setOnPreparedListener(this);
	mVideoView.setVideoPath("android.resource://" + getActivity().getPackageName() + "/" + R.raw.xxx);
	mvSlogan.setImageResource(imgRes);
```        
给videoView setVideoPath即可设置视频路径，此处加载raw文件夹中资源，实现MediaPlayer.OnPreparedListener进行播放。

```
	@Override
	public void onPrepared(MediaPlayer mediaPlayer) {
		if (mVideoView != null) {
			mVideoView.requestFocus();
			mVideoView.setOnCompletionListener(this);
			mVideoView.seekTo(0);
			mVideoView.start();
		}
		return;
	}
```


然后实现MediaPlayer.OnCompletionListener, MediaPlayer.OnErrorListener来处理播放完成(控制viewpager跳转至下一页或已是最后一页，则关闭页面)和播放失败时的情况。

```
    @Override
    public void onCompletion(MediaPlayer mediaPlayer) {
        FragmentActivity localFragmentActivity = getActivity();
        if ((localFragmentActivity != null) && ((localFragmentActivity instanceof FullscreenActivity))) {
            ((FullscreenActivity) localFragmentActivity).next(position);
        }
    }

    @Override
    public boolean onError(MediaPlayer mediaPlayer, int i, int i1) {
        FragmentActivity localFragmentActivity = getActivity();
        if ((localFragmentActivity != null) && ((localFragmentActivity instanceof FullscreenActivity))) {
            ((FullscreenActivity) localFragmentActivity).next(position);
        }
        return true;
    }
```
    
另外，需要实现onPause() 和onResume(),在页面中断时停止播放、恢复时继续播放:

```
    public void onResume() {
        super.onResume();
        if (mHasPaused) {
            if (mVideoView != null) {
                mVideoView.seekTo(mVideoPosition);
                mVideoView.resume();
            }
        }
        return;
    }

    public void onPause() {
        super.onPause();
        if (mVideoView != null) {
            mVideoPosition = mVideoView.getCurrentPosition();
        }
        mHasPaused = true;
    }
```
   
在onDestroy()时停止播放（敲黑板，这个必考啊）:

```
    public void onDestroy() {
        super.onDestroy();
        if (mVideoView != null) {
            mVideoView.stopPlayback();
        }
        return;
    }
```
重点就是这些了，剩下的请前往[github](https://github.com/watire/xiamivd.git)!

ps. 推荐个github demo 运行神器：dryrun ，食用方法:

1. 连接手机；
2. 执行命令：dryrun https://github.com/watire/xiamivd.git；
3. 等待下载、安装。

是不是很简单呢，当然要先安装dryrun~~~~~！
