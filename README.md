# test-maven
## Maven生命周期及依赖管理
### 1. 标签与版本规则
```
<project> ：POM 的根元素，定义了一些POM相关的规范。
<modelVersion>: POM版本号，我们用的是 Maven3 这里只能是4.0.0
<groupId> <artifactId> <version>：项目的坐标，用于确定一个唯一的项目，下面会详细提到
<packaging>：打包类型，如jar/war/pom
<name>：当前项目的名称
<dependencies>：当前项目引入的依赖
<dependency>：单个需要引入的具体的依赖包
<scope>：依赖的范围，常见的有 compile 和 test，不同的范围起到包隔离的作用，这个在后面依赖相关的文章中详细讲
```
1.1 版本号规则
Maven 的版本号规则实际上也是业界的通过规则，
{主版本号}.{次版本号}.{增量版本号}-{里程碑版本}
主版本号：一般是指的当前的项目有了重大的架构变动，版本之间几乎完全不兼容，例如：最近出的 SpringBoot3 就已经放弃了Java8，如果不升级 JDK的话，还是只能使用SpringBoot2
次版本号：一般是指的项目的迭代版本，这种版本会修复大量的bug，带来一些小的新特性等，但是没有什么架构上的重大变化。
增量版本号：一般是用于修复一些紧急bug，改动量很小时，可以使用增量版本号。
里程碑版本：就是项目的当前版本处于一个什么样的阶段了，常见的里程碑版本有 SNAPSHOT，alpha，beta，release，GA 等。
在里程碑的版本中，标注SNAPSHOT的为开发版，此时会存在大量的代码变动，alpha和beta分别对应的是内测版与公测版这三个版本都是属于不稳定版本，使用的时候非常容易踩坑，所以一般只用的demo体验，在正式环境中不能使用。
release和GA都属于是稳定的正式版本，可以在正式环境中使用。

### 2 生命周期
在之前的内容中，我们使用了一个简单的 demo 来验证 Maven中的依赖传递、继承、聚合等特性，在构建demo的时候，已经在不知情的情况下，使用过了生命周期与插件。

在验证的过程中，多次使用到了一个指令 mvn clean install ，我们已经知道了 clean 是清理当前项目的 target 目录，install 是将当前项目打包并推送到本地仓库中去。这里的 clean 与 install 就是 Maven 的生命周期中的某个阶段，在每个阶段实际完成工作的就是插件。

可能这段话比较抽象，没关系，接下来会详细的讲解，先看一本篇的主要内容：

Maven 生命周期详解：
生命周期中的 阶段 与 插件目标
常见的生命周期指令
插件与生命周期绑定使用
2.生命周期
我们在开发中描述的项目的生命周期，一般是指的是 编译、测试、打包、部署等过程，每个项目的构建生命周期或多或少都有一些差异，Maven 对构建的过程进行的抽象和统一，提出了 Maven生命周期这一个抽象的概念，它的作用是定义一条执行流程，而不会完成实际的工作，在每个流程节点中的工作都会交给具体实例对象去完成。

这个执行的流程中包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成等步骤，几乎覆盖了项目构建中的所有流程节点，我们可以按需选择其中的一部分步骤生成自己的项目包。

Maven 对流程、流程节点、具体工作都有专用的名词，如下：

流程：生命周期（lifeCycle）
流程节点：阶段 （phase）
具体工作：插件目标 （plugin:goal）
Maven 的生命周期有三个，分别是clean、default、site，其中 site 使用的较少，另外两个都是经常会使用到的。这三个生命周期彼此是隔离开的个体，它们可以单独运行，也可以组合起来运行，例如：
单独清理
mvn clean

单独构建
mvn package
mvn install
mvn deploy

清理并构建
mvn clean install

仔细看上面的构建指令：mvn install，而不是 mvn default，这是因为在 mvn 指令后面接的是生命周期中的某个阶段，接下来就详细解释下阶段的含义。

Maven 定义了 3 个生命周期META-INF/plexus/components.xml：default 生命周期clean生命周期site生命周期这些生命周期是相互独立的，每个生命周期包含多个阶段(phase)。并且，这些阶段是有序的，也就是说，后面的阶段依赖于前面的阶段。当执行某个阶段的时候，会先执行它前面的阶段。执行 Maven 生命周期的命令格式如下：
default 生命周期
default生命周期是在没有任何关联插件的情况下定义的，是 Maven 的主要生命周期，用于构建应用程序，共包含 23 个阶段。
```
<phases>
  <!-- 验证项目是否正确，并且所有必要的信息可用于完成构建过程 -->
  <phase>validate</phase>
  <!-- 建立初始化状态，例如设置属性 -->
  <phase>initialize</phase>
  <!-- 生成要包含在编译阶段的源代码 -->
  <phase>generate-sources</phase>
  <!-- 处理源代码 -->
  <phase>process-sources</phase>
  <!-- 生成要包含在包中的资源 -->
  <phase>generate-resources</phase>
  <!-- 将资源复制并处理到目标目录中，为打包阶段做好准备。 -->
  <phase>process-resources</phase>
  <!-- 编译项目的源代码  -->
  <phase>compile</phase>
  <!-- 对编译生成的文件进行后处理，例如对 Java 类进行字节码增强/优化 -->
  <phase>process-classes</phase>
  <!-- 生成要包含在编译阶段的任何测试源代码 -->
  <phase>generate-test-sources</phase>
  <!-- 处理测试源代码 -->
  <phase>process-test-sources</phase>
  <!-- 生成要包含在编译阶段的测试源代码 -->
  <phase>generate-test-resources</phase>
  <!-- 处理从测试代码文件编译生成的文件 -->
  <phase>process-test-resources</phase>
  <!-- 编译测试源代码 -->
  <phase>test-compile</phase>
  <!-- 处理从测试代码文件编译生成的文件 -->
  <phase>process-test-classes</phase>
  <!-- 使用合适的单元测试框架（Junit 就是其中之一）运行测试 -->
  <phase>test</phase>
  <!-- 在实际打包之前，执行任何的必要的操作为打包做准备 -->
  <phase>prepare-package</phase>
  <!-- 获取已编译的代码并将其打包成可分发的格式，例如 JAR、WAR 或 EAR 文件 -->
  <phase>package</phase>
  <!-- 在执行集成测试之前执行所需的操作。 例如，设置所需的环境 -->
  <phase>pre-integration-test</phase>
  <!-- 处理并在必要时部署软件包到集成测试可以运行的环境 -->
  <phase>integration-test</phase>
  <!-- 执行集成测试后执行所需的操作。 例如，清理环境  -->
  <phase>post-integration-test</phase>
  <!-- 运行任何检查以验证打的包是否有效并符合质量标准。 -->
  <phase>verify</phase>
  <!-- 	将包安装到本地仓库中，可以作为本地其他项目的依赖 -->
  <phase>install</phase>
  <!-- 将最终的项目包复制到远程仓库中与其他开发者和项目共享 -->
  <phase>deploy</phase>
</phases>
```
根据前面提到的阶段间依赖关系理论，当我们执行 mvn test命令的时候，会执行从 validate 到 test 的所有阶段，这也就解释了为什么执行测试的时候，项目的代码能够自动编译。

clean 生命周期
clean 生命周期的目的是清理项目，共包含 3 个阶段：

pre-clean
clean
post-clean
```
<phases>
  <!--  执行一些需要在clean之前完成的工作 -->
  <phase>pre-clean</phase>
  <!--  移除所有上一次构建生成的文件 -->
  <phase>clean</phase>
  <!--  执行一些需要在clean之后立刻完成的工作 -->
  <phase>post-clean</phase>
</phases>
<default-phases>
  <clean>
    org.apache.maven.plugins:maven-clean-plugin:2.5:clean
  </clean>
</default-phases>
```
根据前面提到的阶段间依赖关系理论，当我们执行 mvn clean 的时候，会执行 clean 生命周期中的 pre-clean 和 clean 阶段。

site 生命周期
site 生命周期的目的是建立和发布项目站点，共包含 4 个阶段：

pre-site
site
post-site
site-deploy
```
<phases>
  <!--  执行一些需要在生成站点文档之前完成的工作 -->
  <phase>pre-site</phase>
  <!--  生成项目的站点文档作 -->
  <phase>site</phase>
  <!--  执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 -->
  <phase>post-site</phase>
  <!--  将生成的站点文档部署到特定的服务器上 -->
  <phase>site-deploy</phase>
</phases>
<default-phases>
  <site>
    org.apache.maven.plugins:maven-site-plugin:3.3:site
  </site>
  <site-deploy>
    org.apache.maven.plugins:maven-site-plugin:3.3:deploy
  </site-deploy>
</default-phases>
```
Maven 能够基于 pom.xml 所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。

你可以将 Maven 插件理解为一组任务的集合，用户可以通过命令行直接运行指定插件的任务，也可以将插件任务挂载到构建生命周期，随着生命周期运行。Maven 插件被分为下面两种类型：Build plugins：在构建时执行。Reporting plugins：在网站生成过程中执行。#


上图就是 Maven 生命周期中的完整的阶段，第一次看到这个图的话，可能会看的有点眼花缭乱，但是没有关系，我们暂时先不管这些阶段在做什么，先对这些步骤做一下拆解。

在运行的时候会按照上图中从上到下的顺序运行，直到运行到指定的阶段为止，例如上面的 mvn clean install 指令，会按下面的步骤运行：

找到clean生命周期，从pre-clean开始往下执行，直到clean阶段执行完成。
找到default生命周期，从validate开始往下执行，直到install阶段执行完成。
这里的没有指定 site 直接忽略，clean和 default 大概会执行20多个阶段，看起来还是很多，但实际上这20多个阶段中有很大一部分是不会运行的，这涉及到一个特性：没有绑定插件目标的阶段，在生命周期中不会被调用
这就好办了，我们只需要关注绑定了插件目标的阶段就好了，Maven 针对不同的打包格式，提供了不同的默认插件目标绑定规则，以打包成jar来举例：

clean：默认是清除target目录中的所有文件，避免将历史版本打到新的包中造成一些不在预期中的问题。
process-resources：将资源文件src/main/resources下的文件复制到target/classes目录中。
compile：将src/main/java下的代码编译成 class 文件，也放到target/classes目录中。
process-test-resources：将资源文件src/test/resources下的文件复制到target/test-classes目录中。
test-compile：将src/test/java下的代码编译成 class 文件，也放到target/test-classes目录中。
test：运行单元测试并在target/surefire-reports中生成测试报告。
package：将资源文件、class文件、pom文件打包成一个jar包。
install：将生成的jar包推送到本地仓库中。
deploy：将生成的jar包推送到远程仓库中。
现在我们执行一次deploy指令，看看控制台中打印出来的结果，对比一下是不是这些插件目标，篇幅问题，这里只放demo-a部分的结果，用红色字体标出的插件目标：
2.2.插件目标
插件目标非常容易理解，其实就是在某个插件中的某个功能，例如：compiler:compile是属于 maven-compiler-plugin 里面的一个功能，compiler 是目标前缀(goalPrefix)，compile就是目标（goal）。

两者组成了一个坐标，绑定到了compile这个阶段上，当运行到这个阶段的时候，就会通过这个坐标找到对应编译功能的代码并运行。

跳过测试插件目标
我们每次执行构建时，到走到生命周期中的test阶段，就会把项目中的单元测试全都运行一遍，单元测试较多的时候运行会比较耗时，在项目没有强制要求不需自动运行单元测试的情况下，我们可以通过下面的指令跳过测试阶段。


除了Maven自带的默认插件以外，我们还可以引入官方或者三方编写的非默认的插件，与<dependency>类似，在<plugin>标签中通过坐标引入插件，通过<executions>执行计划

例如，我们有时候不仅需要将jar包推送到私服中，还需要将源码一并推送，方便使用者可以直接查看源码中的注释以及实现代码，这时候就需要引入源码插件。
这里在install这个阶段加入了生成源码的插件目标，也就是说在install阶段上有了两个插件目标，从执行结果来看，两个插件目标都被依次执行了。这涉及到绑定插件目标的另一个特性：**同一个阶段中可以绑定多个插件目标，在这个阶段中，会按照POM中定义的顺序执行。**

### 依赖管理
在Java中，继承的作用，往往是为了子类能够复用父类的某些特性、功能、方法等，如果父级是抽象类，还可以抽取子级中的功能特性，如果父级是接口，还可以在父级中定义一些规范（例如抽象方法），让每个子类都按照父级中的约定进行实现。

在Maven中，模块间的继承与Java的继承是类似的，可以将共性抽取到父 pom 中，通过子 pom 继承父 pom 来复用配置和约束。

2.1.可继承的标签
可继承的标签太多了，不一一例举，这里就列一些我们在开发中常用的可继承的标签：

groupId、version：坐标分组和版本，artifactId 不能继承。
dependencies：依赖配置
denpendencyManagement：依赖管理配置
properties：自定义属性，类似于定义一个变量
repositories：仓库配置
distributionManagement：项目部署的仓库配置
build：插件、插件管理、源码输出位置等配置
2.2.超级POM
所有的 pom.xml 文件都会默认继承 super pom，在 super pom 中定义了这么几个配置：

构件与插件仓库的地址
源码、测试代码以及资源文件resources的默认路径
编译及打包后的文件路径

mvn dependency:tree

maven helper 插件

import scope

举例

依赖路径

带测试scope的依赖引入

依赖排除



