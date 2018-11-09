关于发布一个 Jar 包到中央仓库, 网上已经有很多教程, 但在我看来都是说的很简略, 没有说出为什么要这么做, 这里记录一下自己发布 Jar 的过程.

#### 章序

首先这里是一篇官网文章 [Guide to uploading artifacts to the Central Repository](https://maven.apache.org/repository/guide-central-repository-upload.html) , 主要内容就是说 pom.xml 需要有啥信息, 另外还描述了其他项目( 即非 Apache 基金会之类的组织的项目 )是通过 OSS 进行托管的.

这里便有了第二篇教程 [OSSRH Guide](https://central.sonatype.org/pages/ossrh-guide.html), 主要是说首先你要注册一个账号, 然后创建一个项目 ticket, 这里姑且叫车票吧, 主要是 groupId 的申请, 就像域名一样都需要管理, 最后才能发布项目.

另外 OSS 的教程内均有视频教程, 不过都是发布在 Youtube 就是了, 需要翻墙.( 在我看来, 人手一梯子是开发者的基本素养 - -* )

#### 注册账户及申请 ticket

这里注册极其简单, 不过记得收好自己的用户名和密码就是了, 常识常识, 找不到连接的[戳这里](https://issues.sonatype.org/secure/Signup!default.jspa).

然后要去申请 ticket, 这里申请通过后才能上传 Jar 包.

当注册成功后, 会进入到主界面, 顶部有一个大大的 <strong>Create</strong> 按钮, 点击后就会有申请 ticket 的工单了, 如下图.

![图片1](https://github.com/itfinally/itfinally.github.io/blob/master/photos/OSS-Issue-1.png?raw=true)

![图片2](https://github.com/itfinally/itfinally.github.io/blob/master/photos/OSS-Issue-2.png?raw=true)

填写完之后提交, 由于时差一般要等一天左右的时间, 审批过了之后 issue 的 status 就会变成绿色的 RESOLVED, 这个时候就可以上传 Jar 了.

审核的时候管理员会询问 groupId 是否是你的网站域名, 而且需要出示证明, 一般 以 com.github 或 io.github 开头的话, 管理员就会清楚这是个人项目, 审核也会快一点.

另外在第一次发布 release 版本之后也需要在该 issue 下回复管理员, 让TA们 close 掉这个 issue.

注: 需要看 issue 的话, 在网站的上方点击 Issue 就会看到自己提交的 issue 了.

#### 创建密钥

 到这里之后, 就有自己的账号以及上传 Jar 用的 groupId 了.
 
如果有留意看章序里第二个链接的网站的话, 里面是有一个 Deployment 标题的内容, 上面列举了 Java 所有可用的依赖管理工具, 当然这里只关心 Maven, 点进去就有对应管理工具做发布的教程, Maven 的[戳这里](https://central.sonatype.org/pages/apache-maven.html).

在这份教程内, 开篇就有介绍 nexus-staging-maven-plugin, 这个插件是后话, 主要就是省掉发布项目后还要上去仓库点击 Close & Release 的流程.

首先先来关注密钥, 所有上传到中央库的包必须有加密签名, 用的是 GPG, 关于这个加密工具, 可以参考阮一峰的[教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html).

当然别忘了官网这个[签名指引](https://central.sonatype.org/pages/working-with-pgp-signatures.html).

Anyway, OSS 教程上面用的是 GPG2, 那最好就用 gpg2, 免得诸多不便, 这里以 Mac 环境为例.

- `brew install gpg2`    // 安装 gpg2
- `gpg2 --gen-key`       // 生成密钥, 中间需要添一些
- `gpg2 --list-keys`     // 查看密钥

在生成密钥的过程中需要输入一个密码, 这个密码也要收好, 后面还会用到.

最后执行的 `gpg2 --list-key` 后出来的示意图如下.

![gpg-key](https://github.com/itfinally/itfinally.github.io/blob/master/photos/gpg-key.png?raw=true)

其中红色框框内的那串字符可以用来代替用户, 上传密钥的时候只需要指定这个字符串就行了, 比如 `gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 8C1559D600F77C3A312D9B3F203F7565CE7CA089`.

需要看自己的密钥是否上传成功, 只需要将 `--send-keys` 参数替换为 `--recv-keys` 就行了.

注: keyserver 指定的密钥服务器是官网上给出的地址, 在教程内可以找到该 url.

#### 生成 Maven access user token

这时候我们已经可以为 Jar 添加加密签名了, 不过还需要为 maven client 生成一个用户名和密码, 这一步官网在哪说没看到, 倒是在视频教程上看到了, 首先需要用一开始就创建好的账号登陆[中央仓库](https://oss.sonatype.org/#view-repositories), 登陆后点击用户名, 这里会有一个 profile 选项.

进入后就会有如下画面, 一顿操作下来就行了, access token 生成后就不会再变.

![access_token](https://github.com/itfinally/itfinally.github.io/blob/master/photos/maven_central.png?raw=true)

#### 配置 setting.xml 与 pom.xml

首先配置 setting.xml.

```xml
<setting>
  <servers>
    <server>
      <id>oss-deploy-server</id>
      <username>刚才 access token 的 username</username>
      <password>刚才 access token 的 password</password>
    </server>
  </servers>
  
  <profiles>
    <profile>
      <id>release</id>
        <activation>
          <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
          <gpg.executable>gpg2</gpg.executable>
          <gpg.passphrase>生成密钥时输入的密码</gpg.passphrase>
        </properties>
    </profile>
  </profiles>
</setting>
```

如果不清楚 profile 是干嘛的, 那需要补一下 maven 关于 profile 的知识. 

这里一句话带过, 就是说如果在打包时, 比如 `mvn deply -P profile-id`, 这里就会根据 id 指定执行对应 profile 内的配置, setting.xml 这里的配置是全局有效的.

至此, setting.xml 已经配置完毕, 接下来看 pom.xml.

网上大部分教程都是都是直接在 parent 内继承 `org.sonatype.oss.oss-parent`, 那实际上现在有比较多项目都是 spring boot 开发的, 所以不能按这种方式配置.

根据官网教程, pom.xml 内需要配置 licenses, scm, distributionManagement 三种信息, 并且需要加上 maven-source-plugin, maven-javadoc-plugin, maven-gpg-plugin 三个插件, 这里相关的配置在此就不再提及了, [官网](https://central.sonatype.org/pages/apache-maven.html)说的比我还清楚.

主要留意 distributionManagement 标签内的信息, 发布地址是配置在该标签内, 并且是通过 id 进行标识, 这个 id 必须要和刚才在  setting.xml 内配置的 server 标签内的 id 保持一致, 否则上传的时候就会看到大名鼎鼎的 401 状态码.

最后一步, 添加 nexus-staging-maven-plugin, 这个插件是自动发布用的, 在执行 `mvn deploy` 后无需再跑到仓库上点击 Close & Release. 当然这个插件我也不会再说, 官网都有描述, 唯一注意的是这个插件上的配置也有一个 serverId, 记得也是和 setting.xml 内配置的 server 标签内的 id 保持一致.

注意, nexus-staging-maven-plugin 只能用在发布 release 版本上, snapshot 版本不能使用该插件( 因为快照版是不会被同步到中央仓库的 ), 所以这个时候最好用 profile 标签配置下.

另外, 最好先发个快照版测试流程, 毕竟发布 release 版本是一件很严肃的事情.

至此, 所有工作完毕, 执行 `mvn deploy` 就会开始上传了, 如果哟与 profile 配置的话记得带上 -P 喔.

#### 大插曲

如果自己存在多份 maven 的配置的话, 记得带上 `--settings` 参数显式指定使用的配置文件, 否则的话老是收到 401 而又觉得自己配置没错就不好了.

