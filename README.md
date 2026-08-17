# LAN-Intercom
局域网对讲
基于局域网 UDP 信令的语音对讲 Android 应用，支持 PTT（Push-to-Talk）半双工模式，采用 MiuiX 风格 UI。
功能特性
PTT 半双工对讲：长按说话、松开收听，模拟真实对讲机体验
NSD 设备发现：自动搜索同一局域网内的对讲设备，点击设备卡片一键发起对讲
高清语音：16-bit PCM 16kHz 直传，回声消除 + 降噪 + 自动增益控制
扬声器输出：通话音频通过 `setCommunicationDevice` 强制路由到扬声器
对讲通知：通话中显示"'设备名'对讲中。"通知，支持下拉展开，点击打开 App
语音安全：全程不缓存、不记录、不保存，播放完立即销毁
常驻后台：前台 Service + WakeLock + 开机自启
Vulkan 支持：声明 Vulkan 图形渲染特性
MiuiX 主题：Xiaomi HyperOS 风格 UI 组件
系统要求
Android 13 (API 33) 及以上
JDK 17
Android SDK 34 (Compile SDK)
Gradle 8.5
项目结构
```
GB28181Intercom/
├── app/
│   ├── build.gradle.kts
│   ├── proguard-rules.pro
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/mike/intercom/
│       │   ├── App.kt                         # Application 入口
│       │   ├── MainActivity.kt                # 主 Activity
│       │   ├── audio/                         # 音频处理
│       │   │   ├── AudioCapturer.kt           # 麦克风采集 (PCM 16kHz + AEC/NS/AGC)
│       │   │   ├── AudioPlayer.kt             # 音频播放
│       │   │   ├── AudioTransport.kt          # UDP 音频传输
│       │   │   ├── BeepGenerator.kt           # 提示音
│       │   │   └── G711Codec.kt              # G.711 编解码 (遗留)
│       │   ├── data/                          # 数据与设置
│       │   │   ├── AppSettings.kt
│       │   │   └── SettingsManager.kt
│       │   ├── discovery/
│       │   │   └── DeviceDiscovery.kt         # NSD 设备发现
│       │   ├── notification/
│       │   │   └── NotificationHelper.kt      # 通知管理
│       │   ├── permission/
│       │   │   └── PermissionManager.kt       # 权限管理
│       │   ├── receiver/
│       │   │   └── BootReceiver.kt            # 开机自启
│       │   ├── service/
│       │   │   └── IntercomService.kt         # 前台 Service + PTT 逻辑
│       │   ├── signal/                        # UDP 信令
│       │   │   ├── SignalServer.kt            # 信令服务 (CALL/ACCEPT/HANGUP等)
│       │   │   └── SignalListener.kt          # 回调接口
│       │   ├── viewmodel/
│       │   │   └── IntercomViewModel.kt
│       │   └── ui/
│       │       ├── icons/
│       │       │   └── AppIcons.kt
│       │       ├── theme/                     # MiuiX 主题
│       │       │   ├── Color.kt
│       │       │   ├── Theme.kt
│       │       │   ├── ThemeManager.kt
│       │       │   └── Type.kt
│       │       └── screens/                   # Compose 界面
│       │           ├── WelcomeScreen.kt       # 欢迎页
│       │           ├── PermissionScreen.kt    # 权限请求
│       │           ├── NameInputScreen.kt     # 设备命名
│       │           ├── HomeScreen.kt          # 主页 (设备列表)
│       │           ├── IntercomScreen.kt      # 对讲界面 (长按说话)
│       │           └── SettingsScreen.kt      # 设置
│       └── res/                               # 资源文件
├── build-apk.ps1                              # 一键构建脚本
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── gradlew / gradlew.bat
└── gradle/wrapper/gradle-wrapper.properties
```
构建方法
方法一：使用 Android Studio
安装 Android Studio (Hedgehog 或更高版本)
打开项目目录 `GB28181Intercom`
等待 Gradle 同步完成
点击 Build → Build Bundle(s) / APK(s) → Build APK(s)
APK 输出路径：`app/build/outputs/apk/debug/app-debug.apk`
方法二：命令行构建
前置条件
安装 JDK 17：
```
   winget install Microsoft.OpenJDK.17
   ```
安装 Android SDK Command-line Tools：
下载: https://developer.android.com/studio#command-line-tools-only
解压到: `%LOCALAPPDATA%\\\\Android\\\\Sdk\\\\cmdline-tools\\\\latest`
配置环境变量：
```powershell
   \\\[Environment]::SetEnvironmentVariable("JAVA\\\_HOME", "C:\\\\Program Files\\\\Microsoft\\\\jdk-17", "User")
   \\\[Environment]::SetEnvironmentVariable("ANDROID\\\_HOME", "$env:LOCALAPPDATA\\\\Android\\\\Sdk", "User")
   ```
创建 `local.properties` 文件：
```properties
   sdk.dir=C\\\\:\\\\\\\\Users\\\\\\\\<用户名>\\\\\\\\AppData\\\\\\\\Local\\\\\\\\Android\\\\\\\\Sdk
   ```
构建 APK
```powershell
cd GB28181Intercom

# Debug APK
.\\\\gradlew.bat assembleDebug

# Release APK (需要签名配置)
.\\\\gradlew.bat assembleRelease
```
APK 输出路径：
Debug: `app/build/outputs/apk/debug/app-debug.apk`
Release: `app/build/outputs/apk/release/app-release-unsigned.apk`
方法三：使用构建脚本
运行项目根目录下的 `build-apk.ps1`：
```powershell
.\\\\build-apk.ps1
```
使用说明
首次启动
打开 App，进入欢迎页
授予所需权限（麦克风、通知、附近设备）
输入设备名称（用于其他设备发现你）
进入主页，自动搜索局域网内的对讲设备
发起对讲
确保两台设备连接同一 Wi-Fi
在主页设备列表中点击目标设备
进入对讲界面，长按按钮说话，松开收听
点击挂断按钮结束对讲
接收对讲
收到来电后自动接听，显示"'设备名'对讲中。"通知
下拉通知可展开查看完整内容，点击通知打开 App
语音从扬声器输出
设置
配置项	说明	默认值
设备名称	用于设备发现显示	我的手机
信令端口	UDP 信令端口	5060
音频端口	UDP 音频传输端口	5004
采样率	PCM 采样率	16000 Hz
权限说明
首次启动需要授予：
麦克风 - 语音对讲
通知 - 对讲中通知
附近设备 - NSD 设备发现 (Android 13+)
常驻后台 - 关闭电池优化
开机自启动 - 厂商设置中开启
技术实现
信令协议
基于 UDP 的轻量信令，消息格式为管道分隔的文本：
消息	格式	说明
CALL	`CALL|deviceId|deviceName|signalPort|audioPort`	发起对讲
RINGING	`RINGING`	振铃中
ACCEPT	`ACCEPT|audioPort`	接听
REJECT	`REJECT|reason`	拒绝
HANGUP	`HANGUP`	挂断
KEEPALIVE	`KEEPALIVE`	心跳保活
音频传输
格式：16-bit PCM, 单声道, 16kHz
传输：UDP 直传，4 字节序号头 + PCM 数据
采集：AudioRecord (VOICE_COMMUNICATION 源)
播放：AudioTrack (USAGE_VOICE_COMMUNICATION)
音效：回声消除 (AEC) + 降噪 (NS) + 自动增益控制 (AGC)
路由：`AudioManager.setCommunicationDevice` 强制扬声器输出
PTT 半双工模式
说话：长按按钮 → 停止播放 → 启动麦克风采集 → 发送 PCM
收听：松开按钮 → 停止麦克风 → 启动播放 → 接收 PCM
同一时间只能说话或收听，避免回声
设备发现
基于 Android NSD (Network Service Discovery)
注册本地服务，监听其他设备的服务广播
自动更新设备列表，支持并发解析
点击设备卡片一键发起对讲
语音安全
音频数据仅在内存中流转
不写入文件系统
不记录日志
播放完成立即清空缓冲区
通话结束后销毁所有音频对象
技术栈
Kotlin + Jetpack Compose
MiuiX (Xiaomi HyperOS 风格 UI 组件)
StateFlow (响应式状态管理)
Foreground Service + WakeLock
NSD (Network Service Discovery)
UDP Socket (信令 + 音频)
AudioRecord / AudioTrack + AEC/NS/AGC
Vulkan 图形渲染支持
版本
当前版本：1.0 Beta