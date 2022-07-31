# 如何在各个操作系统中设置代理

## Clash for Windows 代理使用教程

> Clash for Windows 是一款增强模式的 Windows 代理工具，提供了丰富的自定义功能。
> 下载地址为: clash-windows.exe: http://file.devexp.cn/f/18065158-629611192-b9aff5?p=6993 (访问密码: 6993)

- 安装

1. 下载完成后，打开 Clash 安装程序，使用默认设置安装。
2. 安装成功后会自动运行 Clash ，屏幕底部任务栏将显示 Clash 图标![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qg14cxrkj200w00w0hk.jpg)

**如果你看不到 Clash 图标，它可能已隐藏，点击 可显示隐藏的图标。 双击任务栏 Clash 图标可显示软件主界面。**

3. 右键点击桌面上的 Clash for Windows 图标 属性 兼容性 选中「以管理员身份运行此程序」 确定。

- 导入节点

1. 前往「Clash」「配置 / Profiles」 粘贴「订阅地址」 点击「Download」
2. 下载成功后会在下方显示；
3. 点击「保存 / Save」，如果导入节点失败，请尝试 WIFI 或 4G 热点；
4. 成功导入节点后，鼠标移动至「Proxy」或「GLOBAL」可显示节点列表。

- 手动更新节点

为了能及时获得最新节点以及清楚本地无效的节点，你需要经常手动更新订阅网址： 前往「配置 / Profiles」界面，点击配置文件的 图标立即同步节点.

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qg3ozecrj20nm0graal.jpg)

- 选择代理模式

1.  前往「代理 / Proxies」界面，根据需求选择代理模式：

    - 全局连接 / Global：代理所有流量：

      > 适用于不依赖大陆服务的用户。 对国外流量效果非常好，大陆流量会被减速。

    - 如果出现以下情景时，可以需要规则判断 / Rules：此时只代理国外流量：

      - 无法打开国际网站；
      - 加载国际网站缓慢；
      - 无法解锁 Netflix / Spotify 等区域限制内容。
      - 适用于同时使用国内外服务的用户。

    - 直接连接 / Direct：不代理任何流量：
      > 选择此模式将导致无法翻墙，与开关 VPN 的效果一致。

2.  选择节点

    前往「代理 / Proxies」界面，点击测速图标，选择可用节点。

    注意：**不要选择「DIRECT」或「REJECT」节点，会导致断网**

3.  如果需要在终端中使用代理服务

    > 部分应用程序即不会跟随系统代理设置，也不能手动设置代理，则可以使用增强模式。开启增强模式将强制接管所有应用的网络请求， 有可能会影响部分应用程序的正常运行，建议仅在需要时开启增强模式，避免引起兼容性问题：

    1. 确保已勾选：「配置」「试验性功能」「使用内置接口」。
    2. 确保已取消勾选「设置为系统代理 / Set as system proxy」。
    3. 勾选「增强模式」。

## Clash for Android 代理使用教程

> 下载地址 cfa-2.5.4-premium-universal-release.apk: http://file.devexp.cn/f/18065158-629611304-69f684?p=6993 (访问密码: 6993)

**如果安裝失敗: 部分工具会将下载后的文件強制改名，导致按照失败。将文件名改为「clash.apk」后可以尝试再次安裝**

- 导入节点

  1. 打开 App “配置 / Profiles” “新配置 / New Profile” “URL”；
  2. 在“名称 / Name”中输入 [任意名称] ，在“URL”中输入 [订阅地址] ，在“自动更新（秒） / Auto Update (Seconds)”中输入 [86400] ；
  3. 轻点右上角图标 ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qgjikznoj200o00o0c9.jpg)，如果导入节点失败，请尝试 4G 或 WIFI ；
     成功导入节点后，轻点选中前面的小圆圈；
  4. 返回 App 主页，轻点“已停止 / Stopped” “代理 / Proxy”，可显示节点列表。

- 更新

  为了绕过 GFW 不断升级的封锁技术，建议你经常更新节点。最好每天同步节点：

  1. 确保关闭任何 VPN 包括 Clash 。
  2. 打开 App ，轻点“配置 / Profiles” 右侧图标“┇”“更新 / Update”，

- 选择节点

  前往 App 主页，轻点“已停止 / Stopped”“代理 / Proxy”，根据需求选择可用节点。

- 选择代理模式

  前往 App 主页，轻点“代理 / Proxy”“┇”“模式 / Mode”，根据您的需求选择代理模式。

## Shadowrocket for iOS 代理使用教程

> 中国大陆 App Store 不允许 VPN 类代理软件上架，如果你目前没有非大陆区 Apple ID ，可以自行注册美区 Apple ID 或者使用他人共享 的 Apple ID 。如果不知道如何操作，我可以提供付费服务

- 导入节点

  1. 前往「Shadowrocket」 点击屏幕右上角「✚」图标 「类型」「Subscribe」；
  2. 在「URL」中输入 [订阅地址] ，在「备注 / Remarks」中输入 [名称] ；
     ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qgpbaqfmj20u01sx40s.jpg ":size=300")

  3. 点击「保存 / Save」，如果导入节点失效，请尝试更换网络， 4G 或 WIFI 。
  4. 成功导入订阅后，回到 App 首页轻点项目显示节点列表。

- 手动更新节点

  前往「首页 / Home」向右滑动项目即可同步节点
  ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qgqe7bngj20da0ssgmh.jpg ":size=300")

- 导入配置

  1. 前往配置界面；
  2. 选择远程文件 -> 添加配置；
     ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qgr7fn6oj20u01sxwh7.jpg ":size=300")

  3. 输入https://assetsforcms.oss-cn-hongkong.aliyuncs.com/shadowrocket_66e0b52c10.conf，并确认；（这个文件由第三方提供，准确性和可用性并未得到验证）

  4. 选择刚添加的配置，使用配置。如果你有自定义规则的需求，请参考自定义规则，如果你不知道自定义规则的作用，请无视。
     ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4qgsw402tj20u01sxq65.jpg ":size=300")

- 选择节点

  1. 点击「首页 / Home」，更具你的需求选择节点。
  2. 找到可用节点
  3. 点击「设置 / Settings」「 延迟测试方法 / Test Method」「CONNECT」。
  4. 点击「首页 / Home」「 连通性测试 / Connectivity Test」 即可找到可用节点。
