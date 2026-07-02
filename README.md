## 手机刷了第三方系统导致广东校园闪退的，还有想开热点或USB网络共享的看过来！

> [!IMPORTANT]
> 当前项目仅能在安卓终端运行，需要图形化的标准应用程序(apk)的请前往[这里](https://github.com/anshenglv/ESurfingClient-CVersion-Android-APK)

> [!IMPORTANT]
> 该项目的Releases如何运行：
> 
> ESurfingClient这个文件传到手机的/data/local/tmp/里，可以在比如MT管理器终端运行，获取shell或root权限后输入“/data/local/tmp/ESurfingClient”。第一次运行会报Segmentation fault，按Ctrl+C结束后找到/data/local/tmp/esurfing/ESurfingClient.json，和其他平台一样填写账号等信息，重新运行即可。目前退出程序的逻辑有些问题，同样影响APK版。

> [!NOTE]
> 自行构建需要下载android-ndk-r25c，在CLion添加工具链，并设好对应C和C++编译器路径
> 
> C Compiler: [NDK路径]/toolchains/llvm/prebuilt/[你的系统]/bin/clang.exe
> 
>C++ Compiler: [NDK路径]/toolchains/llvm/prebuilt/[你的系统]/bin/clang++.exe
>
> 然后在CMake里添加一个Android的配置文件，CMake选项填：
> 
> （以下提到的静态库在Code根目录/android-arm64-libs.tar.gz，已在WSL里编译好的）

>-DCMAKE_TOOLCHAIN_FILE="/你的android-ndk-r25c路径/build/cmake/android.toolchain.cmake"
>
>-DCMAKE_SYSTEM_NAME=Android
>
>-DANDROID_ABI=arm64-v8a
>
>-DANDROID_PLATFORM=android-24
>
>-DOPENSSL_ROOT_DIR="解压的静态库路径"
>
>-DOPENSSL_INCLUDE_DIR="解压的静态库路径/include"
>
>-DOPENSSL_CRYPTO_LIBRARY="解压的静态库路径/lib/libcrypto.a"
>
>-DOPENSSL_SSL_LIBRARY="解压的静态库路径/lib/libssl.a"
>
>-DCURL_ROOT="解压的静态库路径"
>
>-DCURL_INCLUDE_DIR="解压的静态库路径/include"
>
>-DCURL_LIBRARY="解压的静态库路径/lib/libcurl.a"

## 当前项目所做的改动

**1. 日志目录 — src/utils/Logger.c**
- get_log_dir(): 新增 #elif defined(__ANDROID__) 分支，Android 上使用 /data/local/tmp/esurfing/logs/，不再创建 /var/log/ 目录树
- init_logger(): Android 上错误提示改为"请使用 root 权限运行程序"

**2. 配置文件路径 — src/utils/PlatformUtils.c**
- Android 上配置文件固定为 /data/local/tmp/esurfing/ESurfingClient.json
- load_cfg() 中 Android 跳过 get_exec_dir()，改为 mkdir("/data/local/tmp/esurfing", 0755) 确保目录存在

**3. 禁用 Web 服务器和系统服务**

|     文件     |                  改动                     |
|:------------:|:-----------------------------------------------------------------:|
|CMakeLists.txt|	Android 编译使用精简源文件列表（排除 WebServer.c, mongoose.c, Service.c, SimProcess.c），排除 CopyPortal，strip 方式改为 -Wl,--strip-all|
|    Main.c    |--install/--uninstall/--help 用 #if !defined(__OPENWRT__) && !defined(__ANDROID__) 保护|
|DialerClient.c|start_web_server() 声明和调用用同样 guard 保护|
|  Shutdown.c  |stop_web_server()、restart_process() 声明和调用用同样 guard 保护|

**4. 构建方式（当前）**
- 使用 CLion（Windows） + Android NDK r25c
- CMake variable ANDROID 和编译器宏 __ANDROID__ 用于平台判断
- CMakeLists.txt 中已有 if(ANDROID) 处理链接（OpenSSL::SSL, CURL::libcurl, dl）
- 不支持 ESURFING_HOME 环境变量（已回退），路径硬编码为 /data/local/tmp/esurfing/
