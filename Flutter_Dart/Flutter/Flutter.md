# Flutter



### 1. Flutter 安装

1. [中国Flutter]: https://flutter.cn/

2. 开始使用 — Windows — 获取 Flutter SDK — 下载

3. 将压缩包解压 — 找到 bin 目录 — 复制 bin 目录路径

4. 进入Win10 环境变量设置 — 系统变量 — Path — 将复制的目录添加

5. 回到中国Flutter的官网，来到网页底端 — 使用镜像

6. 选择Flutter 社区镜像 — 再次进入Win10环境变量设置 — 系统变量

   ```
   添加：
   FLUTTER_STORAGE_BASE_URL: https://storage.flutter-io.cn
   PUB_HOSTED_URL: https://pub.flutter-io.cn
   ```

7. 检查Flutter版本：

   ```powershell
PS C:\Windows\System32> flutter --version
   Flutter 2.0.5 • channel stable • https://github.com/flutter/flutter.git
   Framework • revision adc687823a (5 days ago) • 2021-04-16 09:40:20 -0700
   Engine • revision b09f014e96
   Tools • Dart 2.12.3
   ```
   
8. 检查Flutter配置是否完好

   ```cmd
   PS C:\Windows\System32> flutter doctor
   Doctor summary (to see all details, run flutter doctor -v):
   [✓] Flutter (Channel stable, 2.0.5, on Microsoft Windows [Version 10.0.19042.804], locale zh-CN)
   [✗] Android toolchain - develop for Android devices
       ✗ Unable to locate Android SDK.
         Install Android Studio from: https://developer.android.com/studio/index.html
         On first launch it will assist you in installing the Android SDK components.
         (or visit https://flutter.dev/docs/get-started/install/windows#android-setup for detailed instructions).
         If the Android SDK has been installed to a custom location, please use
         `flutter config --android-sdk` to update to that location.
   
   [✓] Chrome - develop for the web
   [!] Android Studio (not installed)
   [✓] Connected device (2 available)
   
   ! Doctor found issues in 2 categories.
   ```

9. 可以看到 Android 栏是错误，因为没有安装 Android Studio，安装Android Studio后

10. 运行命令，即可

   ```powershell
   flutter config --android-sdk
   ```

11. 默认可以使用Web



### 2. Flutter 项目

