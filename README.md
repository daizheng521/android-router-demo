本文是基于WMRouter，重新设计了我们公司云教室APP业务模式

一、页面路由架构






二、页面路由规则（Page Router）


1、Activity页面定义规则
1）页面接口定义（页面路由表）
在 Base Module （参考：Demo工程目录结构）中 定义 页面Path 和 页面参数。





a）页面路由表命名规则
[modle] + PageRouter

如支付组件：PayPageRouter.java

b）path声明

page path 字符串规则：/[model]/[page]
i. 必须以 "/" 开头
ii. 必须以 "第一段必须是组件名"，如支付组件: "/pay/***"

参数注释: 每个 path 必须详细说明需要的参数, 及其类型。
c）参数声明

在内部类 XXXPageRouter.ParamsKey 中定义Page参数
命名规则: [page]_[参数KEY]_[参数类型] = "key"，如：public static final String PAGE1_TIME_INT = "time";


页面路由表模板文件：

package com.yimi.router.lib1base;
/**
* Page Path and Params key fo Lib1
* <p>
* <strong> 注意 </strong>
* <pre>
* 1、page path 字符串规则:
* 1) 必须以 "/" 开头
* 2）必须以 "第一段必须是组件名", 如支付组件: "/pay/***"
*
* 2、参数注释: 每个 path 必须详细说明需要的参数, 及其类型。
* </pre>
*/
public class Lib1PageRouter {
// ------- pages --------
/**
* 打开 /lib1/page1
* <pre>
* 参数:
* 1. int {@link ParamsKey#PAGE1_PARAMS1_INT}
* 2. String {@link ParamsKey#PAGE1_PARAMS2_STRING}
*/
public static final String PAGE1 = "/lib1/page1";
/**
* 打开 /lib1/page1
* <pre>
* 参数:
* 1. String {@link ParamsKey#PAGE2_PARAMS1_STRING}
*/
public static final String PAGE2 = "/lib1/page2";
// ------ page params keys -------
/**
* page 跳转需要的参数Key
* <per>
* 命名规则: [page]_[参数KEY]_[参数类型] = "key"
* 如：public static final String PAGE1_TIME_INT = "time";
*/
public static class ParamsKey {
// lib1/page1
public static final String PAGE1_PARAMS1_INT = "params1";
public static final String PAGE1_PARAMS2_STRING = "params2";
// lib1/page2
public static final String PAGE2_PARAMS1_STRING = "params1";
}
}
2）页面注解
为模块中，需要对外暴露的Activity页面添加 注解：

@RouterUri(path = Lib1PageRouter.PAGE1)
public class Page1Activity extends AppCompatActivity {
}
2、Activity跳转Activity
使用 Router.startUri(context, path) 或 new DefaultUriRequest(context, path).start()

1）最基本的方式
Router.startUri(context, Constant.PTAH_APP_PAGE1);
2）复杂跳转(带数据、动画、回调等)
new DefaultUriRequest(context, Lib1PageRouter.PAGE1)
.putExtra(Lib1PageRouter.ParamsKey.PAGE1_PARAMS1_INT, 1)
.putExtra(Lib1PageRouter.ParamsKey.PAGE1_PARAMS2_STRING, "str")
.overridePendingTransition(R.anim.enter_activity, R.anim.exit_activity)
.onComplete(new OnCompleteListener() {
@Override
public void onSuccess(@NonNull UriRequest request) {
Toast.makeText(context, "跳转成功", Toast.LENGTH_SHORT).show();
}
@Override
public void onError(@NonNull UriRequest request, int resultCode) {
}
})
.start();
3）startActivityForResult
使用 new DefaultUriRequest(context, path).activityRequestCode(code).start()

/**
* 测试 startActivityForResult
*/
public void startForResult(View view) {
new DefaultUriRequest(context, Lib1PageRouter.PAGE2)
.activityRequestCode(1234)
.putExtra(Lib2PageRouter.ParamsKey.FRAGMENT_ACTIVITY1_PARAMS2_STRING, "str")
.start();
}


3、Fragment跳转Activity
使用 new FragmentUriRequest(fragment, path).start()

/**
* 测试 Fragment 跳转 外部Activity
*/
private void jumpFromFragment2Activity() {
new FragmentUriRequest(FragmentPage1.this, Lib1PageRouter.PAGE1)
.putExtra(Lib1PageRouter.ParamsKey.PAGE1_PARAMS1_INT, 1)
.putExtra(Lib1PageRouter.ParamsKey.PAGE1_PARAMS2_STRING, "str from lib2 fragment")
.overridePendingTransition(R.anim.enter_activity, R.anim.exit_activity)
.onComplete(new OnCompleteListener() {
@Override
public void onSuccess(@NonNull UriRequest request) {
Toast.makeText(FragmentPage1.this.getActivity(), "跳转成功", Toast.LENGTH_SHORT).show();
}
@Override
public void onError(@NonNull UriRequest request, int resultCode) {
}
})
.start();
}


4、Activity加载Fragment（模块内调用，可选）
1）Fragment页面注解
与 Activity页面注解使用 @RouterUri 不同的是，Fragment页面注解需要使用：@RouterPage

如：

@RouterPage(path = Constant.FRAGMENT1)public class FragmentPage1 extends Fragment {
}

2）Activity加载Fragment页面
使用 new FragmentTransactionUriRequest(activity, PageAnnotationHandler.SCHEME_HOST + path).add(R.id.fragment_container).start

/**
* 测试 Activity 加载 Fragment
*/
private void launchFragment(int msg1, String msg2) {
// 传统方案
/*
getSupportFragmentManager().beginTransaction().add(R.id.fragment_container,
new FragmentPage1()).commitAllowingStateLoss();
*/
// Router方案
new FragmentTransactionUriRequest(this, PageAnnotationHandler.SCHEME_HOST + Constant.FRAGMENT1)
.add(R.id.fragment_container)
.allowingStateLoss()
.putExtra("params1", msg1)
.putExtra("params2", msg2)
.start();
}


5、Fragment切换Fragment（模块内调用，可选）
1）Fragment页面注解
@RouterPage(path = Constant.FRAGMENT2)public class FragmentPage2 extends Fragment {
}

2）Fragment切换Fragment页面
使用 new FragmentTransactionUriRequest(FragmentPage1.this.getActivity(), PageAnnotationHandler.SCHEME_HOST + path).replace(R.id.fragment_container).start()

/**
* 测试 Fragment1 切换 Fragment2
*/
private void jumpFromFragment2Fragment() {
new FragmentTransactionUriRequest(FragmentPage1.this.getActivity(), PageAnnotationHandler.SCHEME_HOST + Constant.FRAGMENT2)
.replace(R.id.fragment_container)
.putExtra("params1", 1)
.putExtra("params2", "str")
.onComplete(new OnCompleteListener() {
@Override
public void onSuccess(@NonNull UriRequest request) {
Toast.makeText(FragmentPage1.this.getActivity(), "跳转成功", Toast.LENGTH_SHORT).show();
}
@Override
public void onError(@NonNull UriRequest request, int resultCode) {
}
})
.start();
}


三、组件通信架构


四、组件通信方案（Service Router）


1、服务定义规则
在 Base Module （参考：云教室工程目录结构）中 定义 服务接口类 和 服务路由表（服务key + 服务参数）。





1）服务接口类
定义需要对外暴露的 Sevice interface 文件，命名必须以”I" 开头，如：IAudioService.java

package com.yimi.router.lib1base;
public interface IAudioService {
void startAudio(String audioPath);
void stopAudio(String audioPath);
}
2）服务接口定义（服务路由表）
a）服务路由表命名规则

[modle] + ServiceRouter

如支付组件：PayServiceRouter.java

b）服务Key的定义

（i）每种Service至少注册一个默认实现，如：public static final @Default String VIDEO_DEFAULT = "default";
（ii）参数注释: 每个 path 必须详细说明需要的参数，可先以下三种:

无参数
--> 可使用 Router.getService(IService.class, "key1"); 获取Service

一个Context参数
--> 可使用 Router.getService(IService.class, "key1", context); 获取Service

其他不定参数,必须在此指定 params (一个{@link IFactory}的实现), 详见Demo
--> 可使用 Router.getService(IService.class, "key1", params); 获取Service

c）服务参数的定义

对于自定义服务参数，需要在 ”服务路由表中“ 定义好参数。该参数其实是一个实现了 IFactory 接口的 服务工厂类。

public static final class XXServiceParams implements IFactory {}


服务路由表模板：

package com.yimi.router.lib1base;
import android.content.Context;
import android.support.annotation.NonNull;
import android.support.annotation.StringDef;
import com.sankuai.waimai.router.service.IFactory;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
/**
* Service 路由接口信息
* <p>
* 注意:
* <pre>
* 1、每种Service至少注册一个默认实现，如：public static final @Default String VIDEO_DEFAULT = DEFAULT;
* 2、参数注释: 每个 path 必须详细说明需要的参数，可先以下三种:
* 1) 无参数
* --> 可使用 Router.getService(IService.class, "key1"); 获取Service
* 2) 一个Context参数
* --> 可使用 Router.getService(IService.class, "key1", context); 获取Service
* 3) 其他不定参数,必须在此指定 params (一个{@link IFactory}的实现), 详见Demo
* --> 可使用 Router.getService(IService.class, "key1", params); 获取Service
* </pre>
*/
public class Lib1ServiceRouter {
/**
* Service实现的默认版本
* <p>
* 该默认实现需要使用该变量和@{@link RouterService}注解来标识，
* 如：@RouterService(interfaces = IService.class, key = Lib2ServiceKey.DEFAULT)
*/
public static final String DEFAULT = "default";
@Retention(RetentionPolicy.SOURCE)
@StringDef({DEFAULT})
public @interface Default {
}
// -------- Service Key ---------
// video service 只有一个默认实现版本
/**
* Video Service 的默认实现
* <p>
* 参数: 无参数
*/
@Default
public static final String VIDEO_DEFAULT = DEFAULT;
// audio service 有多个实现版本，至少包含一个默认实现版本
/**
* Audio Service 的默认实现
* <p>
* 参数: 一个Context
*/
@Default
public static final String AUDIO_DEFAULT = DEFAULT;
/**
* Audio Service 的 type2 实现
* <p>
* 参数: {@link AudioParams}
*/
public static final String AUDIO_TYPE2 = "audio_type2";
// -------- Service 自定义Params ---------
/**
* params of {@link #AUDIO_TYPE2}
*/
public static final class AudioParams implements IFactory {
private Context context;
private boolean debug;
public AudioParams(Context context, boolean debug) {
this.context = context;
this.debug = debug;
}
@NonNull
@Override
public <T> T create(@NonNull Class<T> clazz) throws Exception {
return clazz.getConstructor(Context.class, boolean.class).newInstance(context, debug);
}
}
}
2）服务注解
使用 @RouterService

package com.yimi.router.lib1.service;
import android.util.Log;
import com.sankuai.waimai.router.annotation.RouterService;
import com.yimi.router.lib1base.IVideoService;
import com.yimi.router.lib1base.Lib1ServiceRouter;
/**
* VideoService 默认版本 - 无参
* <p>
* router service key: {@link Lib1ServiceRouter#VIDEO_DEFAULT}
* <pre>
* 注意:
* 1、@RouterService 中必须指定 interfaces、key
* 2、默认实现版本的 key 必须都等于 {@link Lib1ServiceRouter#DEFAULT}
* 3、单例模式 需指定: singleton = true. (建议同时使用 单例模式 + @RouterProvider注解)
*/
@RouterService(interfaces = IVideoService.class, key = Lib1ServiceRouter.VIDEO_DEFAULT)
public class VideoService implements IVideoService {
private static final String TAG = "VideoService";
public VideoService() {
}
@Override
public void startVideo(String videoPath) {
Log.i(TAG, "startVideo: ");
}
@Override
public void stopVideo(String videoPath) {
Log.i(TAG, "stopVideo: ");
}
}


2、单例服务定义
一般路由注解的服务默认是”非单例“的，也就是，每次调用 Router.getService() 获取的Service实例是不同的对象。反之，则是 ”单例“ 服务。

对于单例服务的定义，需要注意下面三点：

在 RouterService 注解中指定 singleton 属性为 true。
如： @RouterService(interfaces = ISocketService.class, key = ”default“, singleton = true)

Service实现本身也应该同时使用”单例模式“。
在Service单例模式的 getInstance() 方法上 添加 @RouterProvider注解。
@RouterProvider 注解仅能添加在 无参的静态方法上，该注解会使得 Router 在实例化无参服务时，优先调用该注解标记的方法。

package com.yimi.router.lib2.service;
import android.util.Log;
import com.sankuai.waimai.router.annotation.RouterProvider;
import com.sankuai.waimai.router.annotation.RouterService;
import com.yimi.router.lib2base.ISocketService;
import com.yimi.router.lib2base.Lib2ServiceRouter;
/**
* SocketService 默认版本 - 无参、单例
* <p>
* router service key: {@link Lib2ServiceRouter#SOCKET_DEFAULT}
* <pre>
* 注意:
* 1、@RouterService 中必须指定 interfaces、key
* 2、默认实现版本的 key 必须都等于 {@link Lib2ServiceRouter#DEFAULT}
* 3、单例模式 需指定: singleton = true. (建议同时使用 单例模式 + @RouterProvider注解)
*/
@RouterService(interfaces = ISocketService.class, key = Lib2ServiceRouter.SOCKET_DEFAULT, singleton = true)
public class SocketService implements ISocketService {
private static final String TAG = "SocketService";
// ----- 单例模式 + @RouterProvider注解 ----->>
private static volatile SocketService sInstance;
private SocketService() {
// init
}
@RouterProvider
public static SocketService getInstance() {
if (sInstance == null) {
synchronized (SocketService.class) {
if (sInstance == null) {
sInstance = new SocketService();
}
}
}
return sInstance;
}
// <<----- 单例模式 + @RouterProvider注解 -----
@Override
public boolean start() {
Log.i(TAG, "start: ");
return false;
}
@Override
public void send(String msg) {
Log.i(TAG, "send: ");
}
@Override
public boolean stop() {
Log.i(TAG, "stop: ");
return false;
}
}


3、服务调用
使用 Router.getService()

下面分表展示了 无参数的Service、参数为一个Context的Service、自定义参数的Service、单例 Service 的调用方法：

// ------- Service Router Test --------
/**
* 测试 调用 无参数的Service (in lib1)
* <p>
* 非单例，每次生成不同的对象。
*/
public void callNoParamsService(View view) {
IVideoService video = Router.getService(IVideoService.class, Lib1ServiceRouter.VIDEO_DEFAULT);
video.startVideo("");
updateServiceInfo(video);
}
/**
* 测试 调用 参数为一个Context的Service（in lib1）
*/
public void callAContextParamsService(View view) {
IAudioService audio = Router.getService(IAudioService.class, Lib1ServiceRouter.AUDIO_DEFAULT, context);
audio.startAudio("");
updateServiceInfo(audio);
}
/**
* 测试 调用 自定义参数的Service（in lib1）
*/
public void callCustomParamsService(View view) {
Lib1ServiceRouter.AudioParams params = new Lib1ServiceRouter.AudioParams(context, true);
IAudioService audio = Router.getService(IAudioService.class, Lib1ServiceRouter.AUDIO_TYPE2, params);
audio.startAudio("");
updateServiceInfo(audio);
}
/**
* 测试 调用 无参数的 单例 Service（in lib2）
* <p>
* 单例，每次生成同一个对象。
*/
public void callSingletonParamsService(View view) {
ISocketService socket = Router.getService(ISocketService.class, Lib2ServiceRouter.SOCKET_DEFAULT);
socket.send("");
updateServiceInfo(socket);
}
五、WMRouter 接入配置
1、配置 gradle 插件
1）在项目根 build.gradle 中添加：router plugin 依赖
buildscript {
repositories {
google()
jcenter()
}
dependencies {
classpath 'com.android.tools.build:gradle:3.2.1'
// NOTE: Do not place your application dependencies here; they belong
// in the individual module build.gradle files
// 添加WMRouter插件
classpath "com.sankuai.waimai.router:plugin:1.2.0"
}
}
2）在APP build.gradle 中 应用 router plugin
apply plugin: 'com.android.application'// 应用WMRouter插件apply plugin: 'WMRouter'
2、配置 WMRouter 依赖
在使用Router的 lib、app 的 build.gradle 中添加 WMRouter 依赖

// routerimplementation 'com.sankuai.waimai.router:router:1.2.0'annotationProcessor 'com.sankuai.waimai.router:compiler:1.2.0'
3、混淆配置
# WMRouter

# 使用了RouterService注解的实现类，需要避免Proguard把构造方法、方法等成员移除(shrink)或混淆(obfuscate)，导致无法反射调用。实现类的类名可以混淆。-keepclassmembers @com.sankuai.waimai.router.annotation.RouterService class * { *; }
六、Demo 与 参考
Router Demo: android-router

更多 WMRouter 使用方法，参考：WMRouter设计与使用文档

