# WEEK011 - 在 Docker 环境中开发 Spring Boot 项目

在 WEEK009 中，有一篇关于 Docker 的 [Getting Started Guide](../week009-spring-guides/guides/gs/spring-boot-docker/README.md)，在那篇教程中我们已经学习了如何编写 Dockerfile 将 Spring Boot 应用构建成 Docker 镜像。在这篇教程中，我们将继续使用之前的那个 Spring Boot 应用，并学习一些更深入的知识。

## 一个简单的 Dockerfile

我们还是先从一个简单的 Dockerfile 开始：

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

在这里我们定义了一个 `JAR_FILE` 变量，在 `docker build` 构建镜像时，通过 `--build-arg` 参数来指定 jar 文件的位置，这是为了创建一个适用于 Maven 和 Gradle 的通用 Dockerfile，因为 Maven 和 Gradle 构建出来的 jar 文件位置是不一样的。如果我们使用的是 Maven，使用下面的命令构建镜像：

```
# docker build --build-arg JAR_FILE=target/*.jar -t myorg/myapp .
```

如果我们使用的是 Gradle，则使用下面的命令构建镜像：

```
# docker build --build-arg JAR_FILE=build/libs/*.jar -t myorg/myapp .
```

如果我们的项目已经明确了是 Maven 项目，可以去掉 `JAR_FILE` 变量，简化 Dockerfile：

```
FROM openjdk:17-jdk-alpine
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

然后直接使用下面的命令构建镜像：

```
# docker build -t myorg/myapp .
```

使用 `docker run` 运行镜像：

```
# docker run -p 8080:8080 myorg/myapp
```

如果想看看打出来的镜像里面的内容具体是什么，可以通过 `--entrypoint` 参数将容器的入口命令修改为 `/bin/sh`： 

```
# docker run -ti --entrypoint /bin/sh myorg/myapp
```

这样就可以在 shell 终端中做一些我们想做的事，这在排查问题时非常有用，比如查看镜像里的文件：

```
/ # ls
app.jar  dev      home     media    proc     run      srv      tmp      var
bin      etc      lib      mnt      root     sbin     sys      usr
```

如果是一个已经在运行中的容器，我们可以通过 `docker exec` 来挂接：

```
docker exec -ti <container-id> /bin/sh
/ #
```

其中 `<container-id>` 可以是容器的 ID 或名称，可以通过 `docker ps` 查看。

### Dockerfile 中的 [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint)

在上面的 Dockerfile 文件中，我们在最后一行使用了 `ENTRYPOINT` 指令指定了容器在启动之后执行 `java -jar /app.jar` 命令：

```
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

上面这种 `ENTRYPOINT` 为 `exec 格式`，除此之外，`ENTRYPOINT` 还有另一种 `shell 格式`：

```
ENTRYPOINT java -jar /app.java
```

`shell 格式` 和 `exec 格式` 的区别在于：`shell 格式` 在启动时是使用 `/bin/sh -c` 来执行的，这会导致容器中 PID 为 1 的进程是 `/bin/sh` 而不是 ENTRYPOINT 中指定的 Java，所以当我们使用 `docker stop` 停止容器时，接收到 `SIGTERM` 信号的是 `/bin/sh` 而不是 Java，我们的 Java 进程就不能优雅退出了。为了解决这个问题，一般在执行的命令前加上 `exec`：

```
ENTRYPOINT exec java -jar /app.java
```

而 `exec 格式` 就是使用 `exec` 来执行的，可以确保容器中 PID 为 1 的进程是 ENTRYPOINT 中指定的进程，一般推荐使用这种格式，而且还可以配合 `CMD` 指令内置程序的命令行参数，`shell 格式` 下 `CMD` 是无效的。

当 `ENTRYPOINT` 中的命令比较长时，可以编写一个简单的 Shell 脚本，这时 Dockerfile 的内容如下：

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
COPY run_without_exec.sh run.sh
RUN chmod +x run.sh
ENTRYPOINT ["./run.sh"]
```

脚本 `run_without_exec.sh` 内容如下：

```
#!/bin/sh
java -jar /app.jar
```

构建镜像并运行，这时再执行 `docker stop` 命令，我们发现会等待 10s 左右才退出，这是因为程序 Java 程序不是容器里 PID 为 1 的进程，它接收不到退出信号，导致 `docker` 等 10s 后将容器强制 kill 掉了。解决这个问题的方法是在 Shell 脚本中使用 `exec` 执行命令：

```
#!/bin/sh
exec java -jar /app.jar
```

另外在 Shell 脚本中要慎用 `sudo` 命令，而应该使用 `gosu`，可以参见 [docker 与 gosu](https://cloud.tencent.com/developer/article/1454344) 这篇文章查看他们之间的区别。

还有一点需要注意的是，如果我们在 Dockerfile 中使用环境变量，下面这样的 `exec 格式` 是不行的：

```
ENTRYPOINT ["java", "${JAVA_OPTS}", "-jar", "/app.jar"]
```

因为 `${JAVA_OPTS}` 这种变量需要 Shell 才能解析，而上面的 `ENTRYPOINT` 直接使用 `exec` 执行，并没有使用 Shell。解决方法是将启动命令写到一个 Shell 文件里，或者直接在 `ENTRYPOINT` 中使用 Shell：

```
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -jar /app.jar"]
```

使用下面的命令验证 `${JAVA_OPTS}` 是否生效：

```
# docker run -p 8080:8080 -e "JAVA_OPTS=-Ddebug -Xmx128m" myorg/myapp
```

到目前为止，我们还没有用到命令行参数，如果像下面这样运行：

```
# docker run -p 8080:8080 myorg/myapp --debug
```

我们的本意是向 Java 进程传入 `--debug` 参数，可惜的是这并不生效，因为这个命令行参数被传给 `sh` 进程了。解决方法是在启动命令中使用 Shell 的两个特殊变量 `${0}` 和 `${@}`：

```
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -jar /app.jar ${0} ${@}"]
```

如果运行的是 Shell 脚本，直接使用 `${@}` 变量即可：

```
#!/bin/sh
exec java ${JAVA_OPTS} -jar /app.jar ${@}
```

### 更小的镜像

注意上面所使用的基础镜像是 `openjdk:17-jdk-alpine`，`alpine` 镜像相对于标准的 [openjdk](https://hub.docker.com/_/openjdk) 镜像要小，不过不能保证所有的 Java 版本都有 `alpine` 镜像，比如 [Java 11 就没有对应的 `alpine` 镜像](https://stackoverflow.com/questions/53375613/why-is-the-java-11-base-docker-image-so-large-openjdk11-jre-slim)。

或者你可以使用 label 中带 jre 的镜像，jre 比 jdk 要小一点，不过也不是所有的版本都有对应的 jre 镜像，而且要注意的是，有些程序的运行可能依赖 jdk，在 jre 环境下是运行不了的。

另外一个方法是使用 [Jigsaw 项目的 JLink 工具](https://openjdk.java.net/projects/jigsaw/quick-start#linker)。`Jigsaw` 是 OpenJDK 项目下的一个子项目，旨在为 Java SE 平台设计、实现一个标准的 **模块系统**（Module System），并应用到该平台和 JDK 中。使用 JLink 可以让你按需选择模块来构建出自定义的 JRE 环境，这比官方提供的 JRE 要更小。不过这种方法不具备通用性，你要为不同的应用构建不同的 JRE 环境，虽然你能得到一个更小的单个镜像，但是不同的 JRE 基础镜像加起来还是不小的。

综上所述，构建镜像时我们并不一定非要追求镜像的体积能最小。镜像的体积能小一点当然很好，可以让上传和下载都很快，不过一旦镜像被缓存过，这个优势也就没有了。所以，不要凭自己的小聪明来构建更小的镜像，而让镜像缓存失效。如果你使用的都是同样的基础镜像，那么应该尝试去优化应用层，尽可能的将那些变动最多的内容放在镜像的最高层，那些体积大的变动不多的内容放在镜像的低层。

## 使用分层优化 Dockerfile

Spring Boot 应用使用了一种 [可执行的 JAR 文件格式](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html)，这种 JAR 文件天然具有分层的特点。我们可以使用下面的命令将 JAR 文件解压：

```
# mkdir target/dependency && cd target/dependency
# jar -xf ../*.jar
```

JAR 文件的结构类似下面这样：

```
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-BOOT-INF
    +-classes
    |  +-com
    |     +-example
    |        +-YourClasses.class
    +-lib
       +-dependency1.jar
       +-dependency2.jar
```

所有的应用类放在 `BOOT-INF/classes` 目录下，而所有的依赖都在 `BOOT-INF/lib` 目录下。另外，`org.springframework.boot.loader` 目录下是 Spring Boot loader 的相关代码，是整个 JAR 文件能执行的关键，可以在 `META-INF/MANIFEST.MF` 文件中看到一些端倪：

```
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.example.demo.DemoApplication
```

接下来我们创建一个带分层的 Dockerfile 文件：

```
FROM openjdk:17-jdk-alpine
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java", "-cp", "app:app/lib/*", "com.example.demo.DemoApplication"]
```

这个文件中包含了三层，其中 `BOOT-INF/lib` 目录是程序的依赖部分，几乎很少变动，我们将这一层放在第一层，而变动最多的应用代码 `/BOOT-INF/classes` 放在第三层，这样每次构建时都可以充分利用第一层的缓存，加快构建和启动速度。

而且当 JAR 文件比较大时，解压后的 JAR 文件启动速度要更快一点。

> 有一点值得注意的是，我们上面看到 JAR 文件中还包括 Spring Boot loader 的相关代码，但是在 Dockerfile 里并没有用到，所以我们在 `ENTRYPOINT` 中需要手工指定启动类 `com.example.demo.DemoApplication`。不过我们也可以将 Spring Boot loader 拷贝到容器里，使用 `org.springframework.boot.loader.JarLauncher` 来启动应用。下面直接使用 `JarLauncher` 来启动应用：
> ```
> $ jar -xf demo.jar
> $ java org.springframework.boot.loader.JarLauncher
> ```
> 使用 `JarLauncher` 启动应用的好处是不用再硬编码启动类，对于任意的 Spring Boot 项目都适用，而且还可以保证 classpath 的加载顺序，在 `BOOT-INF` 目录下可以看到一个 `classpath.idx` 文件，`JarLauncher` 就是用它来构建 classpath 的。

### Spring Boot Layer Index

从 Spring Boot 2.3.0 开始，构建出的 JAR 文件中多了一个 `layers.idx` 文件，这个文件包含了 JAR 文件的分层信息，类似于下面这样：

```
- "dependencies":
  - "BOOT-INF/lib/"
- "spring-boot-loader":
  - "org/"
- "snapshot-dependencies":
- "application":
  - "BOOT-INF/classes/"
  - "BOOT-INF/classpath.idx"
  - "BOOT-INF/layers.idx"
  - "META-INF/"
```

除了 `layers.idx` 文件，在 lib 目录我们还可以看到一个 `spring-boot-jarmode-layertools.jar` 文件，这个依赖文件可以让你的应用以一种新的模式来运行（`jarmode=layertools`）：

```
$ java -Djarmode=layertools -jar ./target/demo-0.0.1-SNAPSHOT.jar
Usage:
  java -Djarmode=layertools -jar demo-0.0.1-SNAPSHOT.jar

Available commands:
  list     List layers from the jar that can be extracted
  extract  Extracts layers from the jar for image creation
  help     Help about any command
```

可以使用 `layertools` 的 `extract` 命令将 JAR 文件按照分层信息解压出来：

```
$ mkdir target/extracted
$ java -Djarmode=layertools -jar target/demo-0.0.1-SNAPSHOT.jar extract --destination target/extracted
```

可以看出这里包括了四个分层：

* dependencies
* spring-boot-loader
* snapshot-dependencies
* application

和之前我们手工创建的分层相比，多了一层 `snapshot-dependencies`，这比之前直接将所有的依赖都放在一层更好，因为 SNAPSHOT 依赖也是很容易发生变化的。使用这个分层信息，我们重新编写 Dockerfile：

```
FROM openjdk:11-jdk-alpine
ARG EXTRACTED=target/extracted
COPY ${EXTRACTED}/dependencies/ ./
COPY ${EXTRACTED}/spring-boot-loader/ ./
COPY ${EXTRACTED}/snapshot-dependencies/ ./
COPY ${EXTRACTED}/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

> 分层信息文件 `layers.idx` 是 Maven 或 Gradle 的 Spring Boot 插件在打包时自动生成的，如果要自定义 `layers.idx`，可以对插件进行配置，参见 Maven 文档 [Layered Jar or War](https://docs.spring.io/spring-boot/docs/2.7.0/maven-plugin/reference/htmlsingle/#packaging.layers)。

## 一些优化技巧

下面这些技巧可以让你的应用启动速度更快：

* 使用 [`spring-context-indexer`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-index)
* 不使用 [`Actuator`](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#actuator)
* 使用 Spring Boot 2.1 & Spring 5.1 以上版本
* 使用 [`spring.config.location`](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-application-property-files) 指定 Spring Boot 配置文件的位置
* 使用 `spring.jmx.enabled=false` 关闭 JMX
* 使用 `-noverify` 参数，关闭 JVM 的字节码校验（bytecode verification）
* 使用 `-XX:TieredStopAtLevel=1` 参数，开启 JIT 的 C1 编译器，也就是 `Client Compiler`
* 在 Java 8 中使用 `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap` 参数，在 Java 11 中已默认启用
* 尽可能使用多个 CPU 核启动应用，至少 2 个，最好 4 个；如果少于 4 个 CPU 核，可以考虑使用 `-Dspring.backgroundpreinitializer.ignore=true` 参数关闭预初始化，这可以阻止 Spring Boot 创建额外的线程

## 多阶段构建（Multi-Stage Build）

在使用上面的 Dockerfile 构建镜像时，都有一个前提条件，就是 JAR 文件已经提前编译打包好了。我们可以将打包这一步也放在 Dockerfile 中，通过使用多阶段构建，将 JAR 文件从一个镜像复制到另一个镜像：

```
FROM openjdk:17-jdk-alpine as build
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src
RUN ./mvnw install -DskipTests

FROM openjdk:17-jdk-alpine
ARG JAR_FILE=/app/target/*.jar
COPY --from=build ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

我们也可以将解压分层的步骤也放在 Dockerfile 里：

```
FROM openjdk:17-jdk-alpine as build
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src
RUN ./mvnw install -DskipTests
RUN mkdir -p target/extracted && java -Djarmode=layertools -jar target/*.jar extract --destination target/extracted

FROM openjdk:17-jdk-alpine
ARG EXTRACTED=/app/target/extracted
COPY --from=build ${EXTRACTED}/dependencies/ ./
COPY --from=build ${EXTRACTED}/spring-boot-loader/ ./
COPY --from=build ${EXTRACTED}/snapshot-dependencies/ ./
COPY --from=build ${EXTRACTED}/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

上面的第一个 `FROM` 为构建的第一阶段，被标记为 `build`，在第一阶段中，我们先将源码复制到容器里，然后执行 `./mvnw` 打包，最后使用 `layertools` 解压。注意这里的源码被分成了四层，前两层是构建工具 Maven Wrapper，后两层是 POM 配置文件和应用代码，这也算是一个比较小的优化。

在第二个阶段中，我们通过 `COPY --from=build` 可以从第一阶段构建的结果中复制文件。

### 实验特性

在上面的 Dockerfile 中，我们使用命令 `./mvnw install -DskipTests` 来构建并打包 Java 代码，每次构建时，Maven 会先从本地仓库获取依赖，如果本地仓库不存在，则会从远程仓库下载依赖。如果我们对代码做了一点点修改，也就意味了应用代码后面的分层缓存全部失效，这时 Maven 又会从远程仓库下载依赖，构建速度可想而知。

那么能不能将 Maven 构建时用到的本地仓库缓存起来呢？

从 Docker 18.09 版本开始，Docker 内置了一种新的构建工具 [BuildKit](https://github.com/moby/buildkit)，它被称为 Docker 下一代构建神器，它比 Docker 自带的构建工具提供了更丰富的构建特性。其中一个特性叫 [`Build Mounts`](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#build-mounts-run---mount)，能够将 `RUN` 指令的结果缓存起来，类似于数据卷的功能，并在其他地方复用。

在 Docker 18.09 之前的版本中，需要开启 Docker 的实验特性（在 Docker daemon 配置文件中添加）：

```
{
  "experimental": true
}
```

并在 Dockerfile 文件第一行写上：

```
# syntax=docker/dockerfile:experimental
```

不过 `Build Mounts` 特性已经内置在最新版本的 Docker 中了，如果你使用的是 Docker 18.09 以上的版本，这一步可以忽略。

然后对 Dockerfile 做一点修改：

```
FROM openjdk:17-jdk-alpine as build
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src
RUN --mount=type=cache,target=/root/.m2 ./mvnw install -DskipTests

FROM openjdk:17-jdk-alpine
ARG JAR_FILE=/app/target/*.jar
COPY --from=build ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

可以看出在 `RUN` 指令上加了一个新的参数 `--mount=type=cache,target=/root/.m2`，这个意思是将构建后的 `/root/.m2` 目录缓存起来，多次构建可以重复使用。

然后我们重新构建镜像，第一次构建仍然需要一点时间，但是后面无论你怎么修改代码，构建速度都会非常快了。

> 如果构建不生效，可以在 `docker build` 命令前面加上 `DOCKER_BUILDKIT=1` 开启 BuildKit 特性。

## 安全性

在容器中我们应该以非 root 账号启动应用程序，为了做到这一点，我们需要在 Dockerfile 中加一层，添加一个用户组和用户，并将该用户设置为当前用户：

```
RUN addgroup -S demo && adduser -S demo -G demo
USER demo
```

第一个 `RUN` 指令运行 `addgroup` 命令添加了一个用户组 `demo`，接着运行 `adduser` 命令添加一个用户 `demo` 并添加到用户组 `demo` 中，第二个指令 `USER` 用于设置当前用户为 `demo`，这样后面的命令都将以 `demo` 用户运行。

以非 root 账号启动应用程序这种做法也被称为 *最小权限原则*（the principle of least privilege）。

除了这个原则外，另一个值得注意的地方是，并不是所有的应用都需要完整的 JDK 环境，所以我们可以将基础镜像替换为 JRE，这不仅能减小镜像体积，也能提高一定的安全性。

## 构建插件

除了 `docker build` 命令，有很多的 Maven 和 Gradle 插件也可以用来构建镜像。

### Spring Boot Maven and Gradle Plugins

你可以使用 Spring Boot 官方的 Maven 和 Gradle 插件来构建镜像，这个插件使用了 [Buildpacks](https://buildpacks.io/) 来生成 OCI 格式的镜像（这和 `docker build` 是一样的）。你不需要编写 Dockerfile 文件，只要有能访问的 Docker 服务即可，无论是本地服务还是远程服务，如果是远程服务，使用 `DOCKER_HOST` 环境变量配置 Docker 地址。

使用下面的 Maven 命令构建镜像：

```
$ ./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=myorg/myapp
```

Buildpacks 对 Spring Boot 应用做了一些优化，也和上面一样对镜像进行分层，另外启动时还使用 [Memory Calculator](https://paketo.io/docs/reference/java-reference/#memory-calculator) 来动态调整 JVM 内存参数。

### Spotify Maven Plugin

另一个 Maven 插件 [Spotify](https://github.com/spotify/dockerfile-maven) 也是一个不错的选择，它不仅可以用于 Spring Boot 项目，也可以用于任意的 Maven 项目。它使用 Dockerfile 来构建 Docker 镜像，和 `docker build` 命令一样，只不过将这个过程集成到 Maven 的生命周期中。使用下面的命令构建镜像：

```
$ mvn com.spotify:dockerfile-maven-plugin:build -Ddockerfile.repository=myorg/myapp
```

> 不过看 [这个项目的首页](https://github.com/spotify/dockerfile-maven)，已经好几年没有维护了。

### Palantir Gradle Plugin

如果你使用的构建工具是 Gradle，还可以尝试下 [Palantir](https://github.com/palantir/gradle-docker) 这一款插件。它和 `Spotify` 一样，依赖于 Dockerfile 来构建镜像，就和执行 `docker build` 命令一样。

### Jib Maven and Gradle Plugins

Google 开源了一款名为 [Jib](https://github.com/GoogleContainerTools/jib) 的构建工具，它非常有意思。首先它不依赖于 Docker 环境，在没有安装 Docker 的情况下也可以使用，其次它也不需要 Dockerfile 文件，Jib 会自动使用分层技术来构建镜像，它会将应用代码和依赖分开来，并且将 snapshot 依赖单独放在一层，因为它变动的可能性更大。

如果没有 Docker 环境，那么 Jib 构建出来的镜像放在哪呢？答案是 Jib 会自动推送到镜像仓库，默认是 [Docker Hub](https://hub.docker.com/)，根据你镜像的名字也可以推送到其他镜像仓库。

所以在运行 Jib 之前确保你有权限访问镜像仓库，你可以注册一个 Docker Hub 账号，然后配置镜像仓库的用户名和密码，有两种配置方式：

* `${HOME}/.docker/config.json`

这个是默认的 Docker 配置文件，如果你机器上已经安装了 Docker，那么直接运行 `docker login` 命令登录镜像仓库就可以了，账号信息会自动配置在这个文件里:

```
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "..."
    }
  }
}
```

* `${HOME}/.m2/settings.xml`

这个是 Maven 的配置文件，如果你的机器上没有按照 Docker，可以将镜像仓库信息写在这个文件的 `servers` 配置项里：

```
    <server>
      <id>registry.hub.docker.com</id>
      <username>aneasystone</username>
      <password>...</password>
    </server>
```

配置完成后，执行下面的命令构建镜像：

```
$ mvn com.google.cloud.tools:jib-maven-plugin:build -Dimage=aneasystone/myapp

[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.example:demo >--------------------------
[INFO] Building demo 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- jib-maven-plugin:3.2.1:build (default-cli) @ demo ---
[WARNING] 'mainClass' configured in 'maven-jar-plugin' is not a valid Java class: ${start-class}
[INFO]
[INFO] Containerizing application to aneasystone/myapp...
[WARNING] Base image 'eclipse-temurin:11-jre' does not use a specific image digest - build may not be reproducible
[INFO] Using credentials from Docker config (C:\Users\aneasystone\.docker\config.json) for aneasystone/myapp
[INFO] The base image requires auth. Trying again for eclipse-temurin:11-jre...
[INFO] Using credentials from Docker config (C:\Users\aneasystone\.docker\config.json) for eclipse-temurin:11-jre
[INFO] Using base image with digest: sha256:dfc65cf47f7190abee866dc1236b18fd52bd1db94918b6d70f238d3cd2538606
[INFO]
[INFO] Container entrypoint set to [java, -cp, @/app/jib-classpath-file, com.example.demo.DemoApplication]
[INFO]
[INFO] Built and pushed image as aneasystone/myapp
[INFO] Executing tasks:
[INFO] [==============================] 100.0% complete
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:58 min
[INFO] Finished at: 2022-06-12T09:42:42+08:00
[INFO] ------------------------------------------------------------------------
```

构建结束后，可以看到镜像被推送到 [我的 Docker Hub 账户下](https://hub.docker.com/repository/docker/aneasystone/myapp) 了。

Jib 也支持使用 docker 来构建镜像，只需要将 `build` 改成 `dockerBuild` 即可：

```
$ mvn com.google.cloud.tools:jib-maven-plugin:dockerBuild -Dimage=aneasystone/myapp
```

## 持续集成

自动化是应用开发流程中的重要一环。现代化的自动化工具几乎都支持将源码构建成 Docker 镜像，下面稍微介绍几个这样的工具。

### Concourse

[Concourse](https://concourse-ci.org/) 是由 VMware Cloud Foundry 开发的一款基于流水线的自动化平台，用于 CI/CD。除 CLI 之外，Concourse 中的所有东西都是无状态的，并运行在容器中，所以它对容器有着很好的支持。[Docker Image Resource](https://github.com/concourse/docker-image-resource) 用于实时跟踪和构建 Docker 镜像。

下面是用 Concourse 构建 Docker 镜像的流水线示例：

```
resources:
- name: myapp
  type: git
  source:
    uri: https://github.com/myorg/myapp.git
- name: myapp-image
  type: docker-image
  source:
    email: {{docker-hub-email}}
    username: {{docker-hub-username}}
    password: {{docker-hub-password}}
    repository: myorg/myapp

jobs:
- name: main
  plan:
  - task: build
    file: myapp/src/main/ci/build.yml
  - put: myapp-image
    params:
      build: myapp
```

流水线结构还是比较清晰的，你需要定义两个东西：**resources**（用于描述输入或输出）和 **jobs**（用于指定对资源的动作），如果输入资源发生变动，就会触发一次新的构建。

### Jenkins

[Jenkins](https://jenkins.io/) 是另一款流行的自动化平台，它拥有非常丰富的特性，其中有一点特性和上面是类似的：[流水线](https://jenkins.io/doc/book/pipeline/docker/)。下面的 `Jenkinsfile` 文件使用 Maven 对 Spring Boot 项目进行构建，然后使用 Dockerfile 构建 Docker 镜像并推送到镜像仓库：

```
node {
    checkout scm
    sh './mvnw -B -DskipTests clean package'
    docker.build("myorg/myapp").push()
}
```

## Buildpacks

Spring Boot 官方的 Maven 或 Gradle 插件使用 [Buildpacks](https://buildpacks.io/) 来构建 Docker 镜像，和下面直接使用 [Pack CLI](https://buildpacks.io/docs/tools/pack/) 是完全一样的：

```
$ pack build myorg/myapp --builder=paketobuildpacks/builder:base --path=.
```

使用 Buildpacks 构建镜像无需编写 Dockerfile 文件，Buildpacks 会自动对镜像的分层进行优化，开发者不需要了解和关心镜像构建的细节。在低层（比如包含操作系统的基础镜像）和高层（包含中间件或语言特定的依赖）之间有一个叫做 ABI 的接口（[Application Binary Interface](https://en.wikipedia.org/wiki/Application_binary_interface)），这可以让一些类似于 Cloud Foundry 的平台，在不影响应用程序功能的前提下，对低层镜像进行安全更新和升级。

## Knative

[Knative](https://cloud.google.com/knative/) 是一个基于 Kubernetes 的平台，用于构建、部署和管理现代无服务器的工作负载（serverless workloads）。它的一大特性是将源码构建成 Docker 镜像，对开发人员和运维人员非常友好，这个工作主要是由 [Knative Build](https://github.com/knative/build) 完成的，它提供了 [很多的模板](https://github.com/knative/build-templates)，比如：[Kaniko](https://github.com/GoogleContainerTools/kaniko) 和 [Buildpacks](https://github.com/knative/build-templates/tree/master/buildpacks) 等。

> 目前，Knative Build 已经被 [Tekton Pipelines](https://github.com/tektoncd/pipeline) 所取代。

## 参考

1. [Spring Boot Docker](https://spring.io/guides/topicals/spring-boot-docker/)
1. [docker 与 gosu](https://cloud.tencent.com/developer/article/1454344)
1. [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
1. [Spring Boot Reference Documentation: Container Images](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#container-images)
1. [体验SpringBoot(2.3)应用制作Docker镜像(官方方案)](https://blog.csdn.net/boling_cavalry/article/details/106597358)
1. [BuildKit](https://yeasy.gitbook.io/docker_practice/buildx/buildkit)

## 更多

### 关于 `ENTRYPOINT` 的几点疑惑

为了验证上面说的 `ENTRYPOINT` 中必须使用 `exec` 来启动程序的说法，我写了三个 Dockerfile 文件，第一个是 `Dockerfile_exec`：

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

第二个是 `Dockerfile_shell_without_exec`：

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT java -jar /app.jar
```

第三个是 `Dockerfile_shell_with_exec`：

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT exec java -jar /app.jar
```

然后依次构建镜像：

```
# docker build -f Dockerfile_exec -t java-exec .
# docker build -f Dockerfile_shell_without_exec -t java-shell-without-exec .
# docker build -f Dockerfile_shell_with_exec -t java-shell-with-exec .
```

然后依次执行：

```
# docker run -it --rm --name java java-exec
```

再使用 `docker stop` 停止，可以看到响应时间很快：

```
# time docker stop java
java

real	0m0.278s
user	0m0.019s
sys	0m0.022s
```

这说明容器是正常结束的，因为如果 Java 进程没有接收到 `SIGTERM` 信号，是不会立即退出的，要等大约 10s 左右时间，docker 向容器发送 `SIGKILL` 信号时才会退出。不过奇怪的是，三个容器都可以正常退出，连 `ENTRYPOINT` 中不使用 `exec` 的 `java-shell-without-exec` 一样可以正常退出。

为了探究是怎么回事，使用 `docker images history` 查看并对比各个镜像：

```
[root@localhost demo]# docker image history java-exec --no-trunc | head -n 2
...
sha256:xx   18 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["java" "-jar" "/app.jar"]

[root@localhost demo]# docker image history java-shell-with-exec --no-trunc | head -n 2
...
sha256:xxx   18 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["/bin/sh" "-c" "exec java -jar /app.jar"]

[root@localhost demo]# docker image history java-shell-without-exec --no-trunc | head -n 2
...
sha256:xxx   18 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["/bin/sh" "-c" "java -jar /app.jar"]
```

可以发现 `java-shell-without-exec` 镜像中的 `ENTRYPOINT java -jar /app.jar` 被替换成了 `ENTRYPOINT ["/bin/sh" "-c" "java -jar /app.jar"]`，难道是 `docker build` 时对镜像做了什么优化导致的吗？但是目前并没有什么证据。

然后去看了 Dockerfile 官方文档中的 [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint) 的例子，又写了三个 Dockerfile，第一个是 `Dockerfile_top_exec`：

```
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

第二个是 `Dockerfile_top_shell_without_exec`：

```
FROM ubuntu
ENTRYPOINT top -b
```

第三个是 `Dockerfile_top_shell_with_exec`：

```
FROM ubuntu
ENTRYPOINT exec top -b
```

然后再依次构建镜像：

```
# docker build -f Dockerfile_top_exec -t top-exec .
# docker build -f Dockerfile_top_shell_with_exec -t top-shell-with-exec .
# docker build -f Dockerfile_top_shell_without_exec -t top-shell-without-exec .
```

当我们执行到 `top-shell-without-exec` 时：

```
# docker run -it --rm --name top top-shell-without-exec
```

奇怪的事情发生了，`docker stop` 花了 10s！

```
# time docker stop top
top

real	0m10.258s
user	0m0.016s
sys	0m0.027s
```

再使用 `docker images history` 查看并对比各个镜像：

```
[root@localhost test]# docker image history top-exec --no-trunc | head -n 2
...
sha256:xxx   8 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["top" "-b"]

[root@localhost test]# docker image history top-shell-with-exec --no-trunc | head -n 2
...
sha256:xxx   14 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["/bin/sh" "-c" "exec top -b"]

[root@localhost test]# docker image history top-shell-without-exec --no-trunc | head -n 2
...
sha256:xxx   14 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["/bin/sh" "-c" "top -b"]
```

和上面的例子几乎是一模一样的，结果证明，并不是 `docker build` 做的手脚。难道是 Java 进程比较特殊吗？为什么不使用 `exec` Java 进程也能优雅退出呢？后面可以使用 Python 或 Go 写两个小程序，验证其他进程是不是一样的？

另一个疑惑是，在 `ENTRYPOINT` 中指定命令行参数时：

```
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -jar /app.jar ${0} ${@}"]
```

上面的 `${0}` 表示第 0 个参数，也就是被执行的程序，类似的，`${1}` 表示第一个参数，`${2}` 表示第二个参数，以此类推，`${@}` 表示所有的参数。但是很显然，在这个命令中，直接使用 `${@}` 应该就可以了，这里的 `${0}` 参数是起什么作用呢？而且如果我们和上面一样去执行 `sh -c` 命令：

```
# sh -c "java -jar /app.jar ${0} ${@}" --debug
```

命令行参数是不能生效的，那么 `docker run` 到底是如何启动 `ENTRYPOINT` 中指定的命令的呢？
