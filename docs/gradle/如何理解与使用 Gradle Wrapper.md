使用 Gradle 有两种方式：一种是直接下载 [Gradle 二进制分发包](https://gradle.org/releases/)（distribution） 解压到本地并配置环境变量；
另一种就是使用 Gradle Wrapper（以下简称 Wrapper）自动下载和配置 Gradle。

## Gradle Wrapper 是什么

Wrapper 可以理解为 Gradle 官方推出的一款“自动化的 Gradle 安装工具”，
旨在节省使用 Gradle 时安装和配置的时间成本，并统一项目协作时团队成员的构建工具的版本。

## 为什么使用 Gradle Wrapper

**Gradle 官方推荐将 Wrapper 作为使用 Gradle 的首选方式。** 但使用 Wrapper 有什么实质好处呢？
简单来说能获得以下好处：

- 标准化项目的构建工具版本，保证所有项目成员使用的构建工具版本一致，确保可靠性和健壮性；
- 使用 Gradle 不需要事先安装，版本的升级和降级，都可以使用 Wrapper 达成。

## Gradle Wrapper 的组成

下面通过 Gradle 创建一个 JVM 项目以方便介绍 Wrapper 的组成，以下是项目的整体结构：

```
.
├── app
│   ├── build.gradle.kts
│   └── src
│       ├── main
│       └── test
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle.kts
```

这里对 Wrapper 相关文件简单介绍：
- `gradle-wrapper.jar`：这是一个可执行 `jar` 格式的文件，职责是下载 Gradle 二进制分发包；
- `gradle-wrapper.properties`：通过该配置文件我们可以配置 Wrapper 运行时的一些信息，如：
Gradle 版本、下载的 Gradle 的存储路径等；
- `gradlew`，`gradlew.bat`：这两个文件分别是不同平台的 Wrapper 执行脚本，通过 Wrapper 使用 Gradle 时，
需要将 `gradle` 命令分别替换成 `gradlew` （Unix-like 系统）或 `gradlew.bat` （Windows 系统）。

## Gradle Wrapper 的工作流程

接下来简单了解 Wrapper 的工作流程，以对它有个整体认知。这是来自[官网](https://docs.gradle.org/current/userguide/gradle_wrapper.html)的 Wrapper 工作流程图：


![ Gradle Wrapper 的工作流程|700](https://raw.githubusercontent.com/yuki-mirai/pic-hub/main/images/2022-10-10/22-35-2022-10-10@22.35.04@2x.png)

整个流程可简单概括为 3 点：
1. 从服务器下载 Gradle 的二进制分发包到本地（会先从 `gradle-wrapper.properties` 获取相关配置）；
2. 解压分发包到 GRADLE_USER_HOME 路径下；
3. 用户通过 `gradlew` （Unix-like 系统）或 `gradlew.bat` （Windows 系统）脚本间接使用 Gradle。

在 `gradle-wrapper.properties` 配置没有被变更的情况下，第 1,2 点只会执行一次。

## 生成 Gradle Wrapper

每一个使用 Gradle 创建的项目都默认生成 Wrapper，但有时候你可能希望自己生成特定版本的 Wrapper，
Gradle 通过一个内置任务提供了这项能力。

**但在生成 Wrapper 之前，必须确保你本地已经安装或通过 Wrapper 下载了 Gradle，如果不知道如何安装，
请参见[手动安装 Gradle]()。**

Gradle 项目中，在 “build setup” 任务组中都可以找到一个名为 `wrapper` 的内置任务，
执行这个任务会在你项目的根目录下生成 Wrapper 相关文件。

该任务提供了以下几个选项（Options）让我们定制生成 Wrapper 的基本信息：

- `--gradle-version`：指定 Wrapper 下载的 Gradle 的版本；
- `--distribution-type`：指定下载的 Gradle 分发包的类型，有 `bin` （仅包含 Gradle 运行时的必要文件）
和 `all` （除必要的运行时文件，还包含源码和文档）两种，默认是 `bin`。
- `--gradle-distribution-url`：指向特定版本的 Gradle 分发包的资源 URL，如果使用了该选项，
`--gradle-version` 和 `--distribution-type` 选项将被忽略，因为该选项已经包含了前两个选项的信息，
如果想从私有的（公司）服务器上下载 Gradle，可以使用该选项。
- `--gradle-distribution-sha256-sum`：指定要下载的 Gradle 分发包的校验和，用于校验下载资源的合法性。

当前我机器上安装了 Gradle 7.4.2 版本，最新版本是 7.5.1 ，假设我想升级到最新版本，有两种方式：
- 第一种，也是最简单的，只要将 `gradle-wrapper.properties` 文件中的

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-7.4.2-bin.zip
```

改成

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-7.5.1-bin.zip
```

此时只要在项目根目录下使用 Wrapper 脚本文件执行命令（如：`./gradlew build`），
Wrapper 就会侦测到配置文件的变动并自动下载新的 Gradle。

对于小版本的升级完全可以使用这种方式，更加便捷；但是大版本的升级和降级就不太推荐，
因为大版本间 `gradle-wrapper.jar` 、`gradlew`  及 `gradlew.bat` 可能有差异，只修改配置的方式不太稳妥，
此时推荐第二种方式。

- 第二种，就是使用 `wrapper` 任务重新生成新的 Wrapper 相关文件。假如我想升级到 7.5.1 版本，
还希望把 Gradle 的源码和相关文档一并下载，可以在项目根目录下执行：

```sh
gradle wrapper --gradle-version 7.5.1 --distribution-type all
```

**注意以上的 `gradle` 命令是来自本地安装的 Gradle，如果你本地没有安装 Gradle，但通过 Wrapper 
下载了其它版本的 Gradle，请把 `gradle` 换成 `./gradlew` 或 `gradlew.bat`。**

执行成功后，你能看到的显著变化是 `gradle-wrapper.properties` 中的 `distributionUrl` 变成了：

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-7.5.1-all.zip
```

 注意此时这个新版本的 Gradle 还没有被下载。

## 使用 Gradle Wrapper

在上一节中我们生成 Wrapper 之后，新版本的 Gradle 还没有被拉取下来，拉取的时机是在我们使用
Wrapper 的时候，Wrapper 侦测到配置文件的变化，才会自动拉取。

通过 Wrapper 使用 Gradle 跟直接使用 Gradle 的体验是完全一致的，你只需根据操作系统决定是
用 `gradlew` 还是 `gradlew.bat` 来替换 `gradle` 命令。例如在 Unix-like 系统下，
你需要使用 `./gradlew build` 命令代替 `gradle build` 构建项目。

**你应该把所有的 Wrapper 相关文件添加到版本控制系统中，以便其它协作人拉取并使用 Wrapper。**

## 在构建脚本中配置 Gradle Wrapper

在[生成 Gradle Wrapper](#生成-Gradle-Wrapper)一节中，描述了如何通过 `wrapper` 任务来生成想要的 Wrapper ，
这个任务需要接收一些选项参数才能正确工作，但它们比较冗长，对于某些选项，
我们不希望每次运行 Wrapper 时都要通过命令行指定，此时可以考虑将这些选项配置到构建脚本中。

`wrapper` 任务接收的参数都被以属性的形式定义在任务类型 `Wrapper` （注意是大写）中，
因此你可以通过以下方式获取任务类型 `Wrapper` ，并配置其中的属性，这里使用 `Kotlin` 脚本配置，
也可以使用 `Groovy`：

```kotlin
tasks.wrapper {
    gradleVersion = "7.5.1"
    distributionType = Wrapper.DistributionType.ALL
}
```

这可以减少你使用 `wrapper` 任务时的选项参数输入。通过脚本方式配置的选项可以被命令行中指定的参数覆盖。

## 配置认证信息

如果你要从私有受保护的服务器（通过 `distributionUrl` 指向）上下载 Gradle，可能被要求提供认证信息。
Wrapper 提供了 HTTP Basic 认证方式，你可以通过两种方式将你的认证信息传递到私有服务器：

- 将认证信息直接嵌入到项目的  `gradle/wrapper/gradle-wrapper.properties` 文件的
`distributionUrl` 属性值中，以这样的形式：

```properties
distributionUrl=https://username:password@somehost/path/to/gradle-distribution.zip
```

注意 `gradle-wrapper.properties` 文件是要被提交到版本控制系统中的，意味着所有有权拉取项目的人都能
看到你的认证信息，因此不要轻易使用这种方式。

- 将认证信息以系统属性的形式配置在用户家目录下的 `.gradle/gradle.properties` 中，如：

```properties
systemProp.gradle.wrapperUser=username
systemProp.gradle.wrapperPassword=password
```

**注意：使用 HTTP Basic 认证时，你应该使用 HTTPS 协议而不是 HTTP，以确保你的认证信息不会轻易泄漏。**

## 验证 Wrapper 下载的 Gradle 合法性

我们知道网上很多正规的资源网站除了提供资源本体外，还会给你该资源对应的 SHA-256 哈希给你校验
资源的合法性，防止资源由于中间人攻击（man-in-the-middle）被篡改。

Gradle 官网也提供了每个版本的 Gradle 以及 Wrapper Jar 文件的 SHA-256 哈希（[可以在这里查看](https://gradle.org/release-checksums/)），
但是手动校验是比较繁琐的，而且如果使用 Wrapper 的话，它下载和解压 Gradle 是一套连贯的动作，
我们根本没有介入的机会。

不过 `wrapper` 任务提供了一个选项参数 `--gradle-distribution-sha256-sum` 供我们提前配置下载的 
Gradle 分发包的 SHA-256 哈希。比如我指定了 Gradle 7.5.1 all 版本的哈希和：

```sh
gradle wrapper --gradle-version 7.5.1 --distribution-type all \
--gradle-distribution-sha256-sum db9c8211ed63f61f60292c69e8sha0d89196f9eb36665e369e7f00ac4cc841c2219
```

Wrapper 在下载完 Gradle 后，在解压之前，会自动使用配置的哈希跟下载的 Gradle 的哈希比对，
将抛出异常，中断解压操作。

**注意：校验只会发生在该版本第一次下载完成后，如果你本地已经下载过该版本了，则不会进行校验。**

## 参考文档

[The Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)
