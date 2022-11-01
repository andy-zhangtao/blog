# 如何免费使用 Goland

> 仅供尝试，有钱最好还是支持正版

## 2022.2 版本适用

1. 点击这里下载 Crack 文件 jetbrains-202202.jar: http://file.devexp.cn/f/18065158-649491477-b4b562?p=6993 (访问密码: 6993)

2. 在 Goland 中打开任何项目或创建一个新项目,打开帮助 -> 自定义 VM Options...

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4fgynliyaj20e80wijsz.jpg ":size=100")

3. **关闭 golang**
4. 在文件中添加以下数据:

   ```
   -Xms1024m
   -XX:ReservedCodeCacheSize=1024m
   -Xmx2048m
   -Dsplash=true

   -XX:+IgnoreUnrecognizedVMOptions
   -XX:+UseG1GC
   -XX:SoftRefLRUPolicyMSPerMB=50
   -XX:CICompilerCount=2
   -XX:+HeapDumpOnOutOfMemoryError
   -XX:-OmitStackTraceInFastThrow
   -ea
   -Dsun.tools.attach.tmp.only=true
   -Dsun.io.useCanonCaches=false
   -Dsun.java2d.metal=true
   -Djdk.http.auth.tunneling.disabledSchemes=""
   -Djdk.attach.allowAttachSelf=true
   -Djdk.module.illegalAccess.silent=true
   -Dkotlinx.coroutines.debug=off
   -XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
   -XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof

   --add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
   --add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED

   -javaagent:<jetbrains.jar path>=jetbrains
   ```

   注意点:

   - Xms/Xmx 需要根据实际情况调整，避免资源不足
   - `<jetbrains.jar path>`是全路径。
   - 最后一定要添加=jetbrains

5. 打开`goland`，从下面注册服务器中选择一个进行注册:

   ```
   http://licence.fit.cvut.cz:9000
   https://jblicense2.wappworks.com
   https://jetbrains-license.learning.casareal.co.jp
   https://license.fahai.org
   https://lic.gotoweb.top
   https://flm.nighthawkcodingsociety.com
   https://fls-jetbrains.spacetechies.com
   https://jbls.x-root.info
   https://license-server.tmk.edu.hk
   https://jenkins.wf-wolves.de
   http://bumblebee.bhasvic.ac.uk
   http://cse-lic-02.engineering.cwru.edu
   http://lic-server.mephi.ru
   http://license.runtime.kz
   https://licenses.cerebotani.it
   ```

6. 注册成功。 Enjoy it！

## 2022.2 以下版本适用

1. 点击这里下载 Crack 文件 JetbrainsIdesCrack_5_3_1_KeepMyLic.jar: https://url58.ctfile.com/f/18065158-623428178-c12609?p=6993 (访问密码: 6993)

2. 在 Goland 中打开任何项目或创建一个新项目,打开帮助 -> 自定义 VM Options...

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4fgynliyaj20e80wijsz.jpg ":size=100")

3. 在文件的末尾，在新的一行中，插入文本“-javaagent:<PATH_TO_JetbrainsIdesCrack_5_3_1_KeepMyLic.jar> ”
   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4fh21ahshj219m04wq3h.jpg ":size=200")

4. 保存.vmoptions 文件 。 然后重启 goland。

5. 打开 KeepLicense_keys 文件夹(点击这里下载 KeepLicense_keys.zip: https://url58.ctfile.com/f/18065158-623428750-4939ae?p=6993 (访问密码: 6993) )

6. 将扩展名为 .key 的相应密钥文件移动到位于/Users/your_user/Library/Application Support/JetBrains/<IDE_name>
   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4fh6ertb4j20km0qignb.jpg ":size=100")

7. 重启 Goland 后，再打开 About 窗口，你会看到没有关于许可证到期日期的信息。

注意:
使用这种激活方式后，如果打开 Help -> Register 窗口，其中的 Close 按钮 ​​ 将被禁用，只能单击 Exit 按钮。但此时会关闭 IDE。同时会丢失未保存的更改。所以不要好奇的点击 Register。
