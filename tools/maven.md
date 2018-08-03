# Maven

## 0. 核心概念

### 0.1 坐标
`groupId`：组织标识（包名） <br />
`artifactId`：项目名称 <br />
`version`：项目的当前版本 <br />

### 0.2 打包方式
`packaging`: `jar` 和 `war` 两种

## 1. 常用命令
`$ mvn clean package`  // 清理并打包

`$ mvn install` // 更新依赖，并打包

## 2. 生命周期
定义： Maven生命周期就是为了对所有的构建过程进行抽象和统一，包括项目清理，初始化，编译，打包，测试，部署等几乎所有构建步骤。
 Maven有三套相互独立的生命周期，请注意这里说的是"三套"，而且"相互独立"，这三套生命周期分别是：

1. Clean Lifecycle，在进行真正的构建之前进行一些清理工作。
Clean生命周期一共包含了三个阶段：
    ```
    pre-clean： 执行一些需要在clean之前完成的工作
    clean： 移除所有上一次构建生成的文件
    post-clean： 执行一些需要在clean之后立刻完成的工作
    ```
    注： 运行后一阶段， 则前面的所有阶段将被执行。如： 执行`$ mvn post-clean`，那么 pre-clean、clean都将被执行。

2. Default Lifecycle， 构建的核心部分，编译，测试，打包，部署等等。 是生命周期中最最要的一个， 包含内容较多：
    ```
    validate
    generate-sources
    process-sources
    generate-resources
    process-resources 复制并处理资源文件，至目标目录，准备打包。
    compile 编译项目的源代码。
    process-classes
    generate-test-sources
    process-test-sources
    generate-test-resources
    process-test-resources 复制并处理资源文件，至目标测试目录。
    test-compile 编译测试源代码。
    process-test-classes
    test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
    prepare-package
    package 接受编译好的代码，打包成可发布的格式，如 JAR 。
    pre-integration-test
    integration-test
    post-integration-test
    verify
    install 将包安装至本地仓库，以让其它项目依赖。
    deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享。
    ```

3. Site Lifecycle， 生成项目报告，站点，发布站点。 也包含三个阶段：
    ```
    site： 生成项目的站点文档
    post-site： 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
    site-deploy： 将生成的站点文档部署到特定的服务器上
    ```
    注： 这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这可是Maven相当强大的功能，Manager比较喜欢，文档及统计数据自动生成
