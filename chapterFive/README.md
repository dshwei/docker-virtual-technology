
## 第五章：在Docker上开发你的应用程序

### 第一节.应用程序开发概述

这一页罗列出了使用Docker开发应用程序的资源

#### 1.在Docker上开发一个新的应用程序

如果你仅仅基于Docker开发一个新的分支的应用程序，检查这些资源，理解一些最常见的模式，这样你能从Docker获得更多的利益。

* 学习通过Dockerfile文件来构建一个镜像
* 使用多级构建的方式来保证你的镜像不要太臃肿
* 使用数据卷或者挂载的方式开管理应用程序
* 把应用程序扩展为集群服务
* 使用compose文件来定义你的应用程序栈
* 普通应用程序开发最好的例子

#### 2.学习指定语言基于Docker来开发应用程序

* Java开发者学习使用Docker开发应用程序链接 `https://github.com/docker/labs/tree/master/developer-tools/java/`
* 基于node.js来开发Docker应用程序链接 `https://github.com/docker/labs/tree/master/developer-tools/nodejs/porting`
* 基于Ruby来开发应用程序的链接 `https://github.com/docker/labs/tree/master/developer-tools/ruby`
* .Net的链接 `https://docs.docker.com/engine/examples/dotnetcore/`
*  ASP.NET的链接 `https://docs.docker.com/compose/aspnet-mssql-compose/`

#### 3.使用SDK或者API来开发

当你通过使用Docker写Dockerfile文件或者compose文件开发应用程序感到特别顺手的时候，这样的话提升一个级别，你可以使用docker引擎SDK通过Go语言或者Python语言直接调用Python的Api进行开发。

### 第二节.Docker开发最佳实践

以下开发模式已被证明有助于人们使用Docker构建应用程序。 如果您发现了我们应该添加的内容，可以给Docker官方提供建议。

#### 1.怎么保持你的镜像不臃肿

在启动容器或服务时，小镜像可以更快速地拉下来，并加载到内存中。 有几条经验法则可以保持镜像不臃肿：

* 从适当的基础镜像开始。 例如，如果您需要JDK，请考虑将您的镜像放在官方的openjdk镜像上，而不是从通用的Ubuntu镜像开始，并将openjdk作为Dockerfile的一部分安装。

* 使用多阶段构建。 例如，您可以使用maven镜像构建Java应用程序，然后重置为tomcat镜像，并将Java构件复制到正确位置以部署应用程序，所有这些都位于相同的Dockerfile中。 这意味着您的最终镜像不包含构建所引入的所有库和依赖项，但仅包含运行它们所需的工件和环境。


* * 如果需要使用不包含多级构建的Docker版本，请尽量减少Dockerfile中单独运行命令的数量，以减少镜像中的层数。 您可以通过将多个命令整合到单个RUN行并使用shell的机制将它们组合在一起来完成此操作。 考虑以下两个片段。 第一个在镜像中创建两个层，而第二个创建一个层。

    RUN apt-get -y update
    RUN apt-get install -y python

    RUN apt-get -y update && apt-get install -y python

* 如果您有多个共同的镜像，请考虑使用共享组件创建您自己的基本镜像，并在其上创建独特的镜像。 Docker只需要加载一次通用层，然后缓存。 这意味着您的衍生镜像更有效地使用Docker主机上的内存并更快地加载。

* 要保持生产镜像精简但允许调试，请考虑使用生产镜像作为调试镜像的基本镜像。 可以在生产镜像上添加其他测试或调试工具。

* 在构建镜像时，应始终使用有用的标签对其进行标记，这些标签将编码版本信息，预期目标（例如产品或测试），稳定性或在不同环境中部署应用程序时有用的其他信息进行编码。 不要依赖自动创建的最新标签。


#### 2.在何处和如何持久化应用程序数据

* 避免使用存储驱动程序将应用程序数据存储在容器的可写层中。 与使用数据卷或绑定挂载相比，这增加了容器的大小，并且从I/O角度来看效率较低。

* 而是使用数据卷存储数据。

* 适合使用绑定挂载的一种情况是在开发过程中，您可能需要挂载源目录或刚刚构建到容器中的二进制文件。 对于生产，请改用数据卷，将其安装到与您在开发过程中安装绑定安装程序相同的位置。

* 对于生产而言，使用秘密来存储服务使用的敏感应用程序数据，并将配置用于非敏感数据（如配置文件）。 如果您当前使用独立容器，请考虑迁移以使用单副本服务，以便您可以利用这些仅限于服务的功能。

#### 3.在可能的情况下使用集群服务

* 在可能的情况下，使用集群服务进行扩展的能力来设计您的应用程序。

* 即使您只需运行应用程序的单个实例，集群服务也会比独立容器提供多项优势。服务的配置是声明式的，Docker一直在努力使期望的和实际的状态保持同步。

* 网络和数据卷可以与集群服务连接和断开连接，并且Docker可以以不中断的方式重新部署各个服务容器。需要手动停止，移除和重新创建独立容器以适应配置更改。

* 一些功能（如存储机密和配置的功能）仅适用于服务而不是独立容器。这些功能允许您保持镜像尽可能通用，并避免将敏感数据存储在Docker镜像或容器本身内。

* 让`docker stack deploy`处理任何镜像，而不是使用`docker pull`。通过这种方式，您的部署不会尝试从发生故障的节点拉出。而且，当新节点添加到集群中时，镜像会自动拉出。

在集群服务的节点之间共享数据存在限制。 如果您将Docker用于AWS或Docker for Azure，则可以使用Cloudstor插件在集群服务节点之间共享数据。 您还可以将应用程序数据写入支持同时更新的单独数据库中。

#### 4.使用CI/CD进行测试和部署

* 当您检查对源代码管理的更改或创建拉取请求时，请使用`Docker Cloud`或其他`CI/CD`管道自动构建并标记Docker镜像并对其进行测试。 `Docker Cloud`也可以将测试过的应用直接部署到生产环境中。

* 通过要求您的开发，测试和安全团队在将镜像部署到生产环境中之前对其进行签名，您可以更进一步了解Docker EE。 通过这种方式，您可以确保在将镜像部署到生产环境之前，它已经通过开发，质量和安全团队进行测试和签名。

#### 5.开发和生产环境的差异

|                开发环境               |                  生产环境           | 
|--------------------------------------|------------------------------------|
| 使用绑定挂载使您的容器可以访问您的源代码    |       使用数据卷存储容器数据          |
| 使用Docker for Mac或Docker for Windows|如果可能的话，使用Docker EE，通过用户映射将Docker进程与主进程隔离开来| 
|           不要担心时间漂移	          |始终在Docker主机上和每个容器进程中运行NTP客户端，并将它们全部同步到同一个NTP服务器。 如果使用swarm服务，还要确保每个Docker节点将其时钟与容器同步到同一时间源。      	    | 

### 第三节.开发镜像

#### 1.编写Dockerfiles的最佳实践

这个部分的内容包含了建立高效镜像推荐的最佳实践和方法。

Docker通过阅读Dockerfile中的指令自动构建镜像，Dockerfile是一个包含构建给定镜像所需要的所有指令文本文件。 Dockerfile遵守特定的格式和指令集，您可以在Dockerfile参考中找到它们。

Docker镜像由只读层组成，每个层代表一个Dockerfile指令。 这些层是堆叠的，每个层都是上一层层变化的三角形。 看下面这个Dockerfile：

    FROM ubuntu:15.04
    COPY . /app
    RUN make /app
    CMD python /app/app.py

每一条指令创建一个镜像层

* FROM:通过ubuntu15.04镜像创建一个镜像层 
* COPY:从您的Docker客户端的当前目录添加文件
* RUN:使用make工具构建你的应用程序
* CMD:指定在容器内运行的命令

当您运行镜像并生成容器时，您会在基础镜像层的顶部添加一个新的可写镜像层（“容器镜像层”）。 对正在运行的容器所做的所有更改（如写入新文件，修改现有文件和删除文件）都会写入此可写容器层。

#### 2.一般准则和建议

##### 2.1.创建临时容器

由Dockerfile定义的映像应该生成尽可能短暂的容器。 通过“短暂的”，我们的意思是容器可以被停止和销毁，然后重建并用绝对最小的设置和配置代替。

##### 2.2.理解构建的上下文

当您发出docker build命令时，当前的工作目录被称为构建上下文。 默认情况下，假设Dockerfile位于此处，但可以使用文件标志（-f）指定不同的位置。 无论Dockerfile实际存在于哪里，当前目录中文件和目录的所有递归内容都将作为构建上下文发送到Docker守护进程。

* 构建上下文的例子

为构建上下文创建一个目录并且cd进到这个目录，将“hello”写入名为hello的文本文件，并创建一个Dockerfile来运行cat，从构建上下文（.）中构建镜像：

    mkdir myproject && cd myproject
    echo "hello" > hello
    echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
    docker build -t helloapp:v1 .

将Dockerfile和hello移入单独的目录并构建第二个版本的镜像（不依赖上一次构建的缓存）。 使用-f指向Dockerfile并指定构建上下文的目录：

    mkdir -p dockerfiles context
    mv Dockerfile dockerfiles && mv hello context
    docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context



无意中包含不需要构建镜像的文件会导致更大的构建上下文和更大的镜像大小。 这会增加构建镜像的时间，增加拉和推镜像的时间，容器运行时大小。 要查看构建上下文有多大，请在构建Dockerfile时查找如下所示的消息：

    Sending build context to Docker daemon  187.8MB

#### 3.通过stdin管道Dockerfile

Docker 17.05添加了通过stdin管道Dockerfile使用本地或远程构建上下文构建镜像的功能。 在早期版本中，使用stdin的Dockerfile构建一个镜像不会发送构建上下文。

##### 3.1.Docker17.04或者更低的版本

    docker build -t foo -<<EOF
    FROM busybox
    RUN echo "hello world"
    EOF

##### 3.2.Docker17.05和更高版本(本地构建上下)

    docker build -t . -f-<<EOF
    FROM busybox
    RUN echo "hello world"
    COPY . /my-copied-files
    EOF

##### 3.3.Docker17.05和更高的版本(远程构建上下文)

    docker build -t foo https://github.com/thajeztah/pgadmin4-docker.git -f-<<EOF
    FROM busybox
    COPY LICENSE config_local.py /usr/local/lib/python2.7/site-packages/pgadmin4/
    EOF

#### 4.排除.dockerignore

要排除与构建无关的文件（不重构源代码库），请使用.dockerignore文件。 该文件支持与.gitignore文件类似的排除模式。 

#### 5.使用多阶段构建

多阶段构建（在Docker 17.05或更高版本中）允许您大幅缩减最终镜像的大小，而不需要努力减少中间层和文件的数量。

由于镜像是在构建过程的最后阶段构建的，因此可以通过利用构建缓存来最小化镜像层。

例如，如果您的版本包含多个镜像层，您可以从较不经常更改的版本（以确保版本缓存可重复使用）对较频繁更改的版本进行排序：

* 安装构建应用程序所需的工具
* 安装或更新库依赖项
* 生成你的应用程序

Go应用程序的Dockerfile可能如下所示：

    FROM golang:1.9.2-alpine3.6 AS build

    # Install tools required for project
    # Run `docker build --no-cache .` to update dependencies
    RUN apk add --no-cache git
    RUN go get github.com/golang/dep/cmd/dep

    # List project dependencies with Gopkg.toml and Gopkg.lock
    # These layers are only re-built when Gopkg files are updated
    COPY Gopkg.lock Gopkg.toml /go/src/project/
    WORKDIR /go/src/project/
    # Install library dependencies
    RUN dep ensure -vendor-only

    # Copy the entire project and build it
    # This layer is rebuilt when a file changes in the project directory
    COPY . /go/src/project/
    RUN go build -o /bin/project

    # This results in a single layer image
    FROM scratch
    COPY --from=build /bin/project /bin/project
    ENTRYPOINT ["/bin/project"]
    CMD ["--help"]


#### 6.不要安装不必要的软件包

为了减少复杂性，依赖性，文件大小和构建时间，避免安装额外的或不必要的软件包，仅仅是因为它们可能“很高兴”。例如，您不需要在数据库镜像中包含文本编辑器。

#### 7.分离应用程序

每个容器应该只有一个焦点应用，解耦程序到多个容器中将会使容器在水平方向更加易于扩展和更容易重用。例如：一个web应用堆可能由三个分离的容器组成，每个容器都有自己独立的镜像通过分离的方式来管理web应用，数据库和内存缓存器。


限制每个容器对应一个进程是一个不错的规则，但这并不是硬性的规则，容器不仅仅能够通过初始化进程运行，很多程序可能会根据他们自己的额外的进程去运行。例如：Celery可以通过多个工作进程产生，Apache则可以针对每个请求产生一个进程。

用做好的方式来保证容器尽可能的干净和模块化。如果容器之间相互依赖的话，你可以使用docker网络来保证这些容器之间是可以通信的即可

#### 8.镜像层最少化原则

在老版本的docker中，镜像层数量最小化是很重要的，因为这可以确保应用的性能。下面的特征是添加减少这些限制

* 在docker1.10版本或者docker更高版本中，仅仅只有RUN,COPY,ADD指令能够创建镜像层。其他的指令只能创建临时或者中间的镜像，并且构建的时候并不能增加镜像的尺寸大小。

* 在Docker17.05及更高版本中，您可以执行多阶段构建，并仅将所需的工件复制到最终镜像中。 这允许您在中间构建阶段中包含工具和调试信息，而不会增加最终镜像的大小。

#### 9.对多行参数进行排序

只要有可能，通过按字母数字方式对多行参数进行排序，可以缓解以后的更改。 这有助于避免重复包并使列表更容易更新。 这也使PR更容易阅读和审查。 在反斜杠（\）之前添加空格也有帮助。

以下是buildpack-deps映像中的示例：

    RUN apt-get update && apt-get install -y \
      bzr \
      cvs \
      git \
      mercurial \
      subversion

#### 10.利用构建缓存

当构建镜像时，Docker会按步骤去执行Dockerfile中的指令，按指定的顺序执行每个指令。 在检查每条指令时，Docker会在其缓存中查找可以重用的现有镜像，而不是创建新的（重复）镜像。

如果您根本不想使用缓存，可以在docker build命令中使用--no-cache = true选项。 但是，如果你让Docker使用它的缓存，重要的是要了解它何时可以找到匹配的镜像。 Docker遵循的基本规则概述如下：

* 从已经在缓存中的父镜像开始，将下一条指令与从该基本镜像导出的所有子镜像进行比较，以查看它们中的一个是否使用完全相同的指令构建。 如果不是，则缓存无效。

* 在大多数情况下，只需将Dockerfile中的指令与其中一个子镜像进行比较即可。 但是，某些说明需要更多的检查和解释。

* 对于ADD和COPY指令，将检查镜像中文件的内容，并为每个文件计算校验和。 在这些校验和中不考虑文件的最后修改时间和最后访问时间。 在高速缓存查找期间，将校验和与现有映像中的校验和进行比较。 如果文件中有任何更改（例如内容和元数据），则缓存无效。

* 除了ADD和COPY命令之外，高速缓存检查不会查看容器中的文件以确定高速缓存匹配。 例如，在处理RUN apt-get -y update命令时，不检查容器中更新的文件以确定是否存在缓存命中。 在这种情况下，只需使用命令字符串本身来查找匹配项。

缓存无效后，所有后续Dockerfile命令都会生成新镜像，并且不使用缓存。

#### 11.Dockerfile指令集

这些建议旨在帮助您创建高效且可维护的Dockerfile。

##### 11.1.FROM

尽可能使用当前的官方存储库作为镜像的基础。 我们推荐Alpine镜像，因为它受到严格控制并且尺寸较小（目前小于5 MB），同时仍然是完整的Linux发行版。

FROM的使用格式如下：

    FROM <image> [AS <name>]

或者

    FROM <image>[:<tag>] [AS <name>]

或者

    FROM <image>[@<digest>] [AS <name>]

FROM指令初始化新的构建阶段并为后续指令设置基本镜像。 因此，有效的Dockerfile必须以FROM指令开头。 镜像可以是任何有效镜像-通过从公共存储库中提取镜像来启动它尤其容易。

* ARG是Dockerfile中唯一可以在FROM之前的指令。
* FROM可以在单个Dockerfile中多次出现以创建多个镜像，或者使用一个构建阶段作为另一个构建阶段的依赖项。 只需在每个新的FROM指令之前记下提交输出的最后一个镜像ID。 每个FROM指令清除先前指令创建的任何状态。
* 可以选择通过将`AS name`添加到FROM指令，可以将名称赋予新的构建阶段。 该名称可以在后续的`FROM`和`COPY --from = <name | index>`指令中使用，以引用此阶段构建的镜像。
* 标记或摘要值是可选的。 如果省略其中任何一个，则构建器默认采用最新标记。 如果找不到标记值，构建器将返回错误。

###### 理解ARG和FROM如何互动

* FROM指令支持在第一个FROM之前发生的任何ARG指令声明的变量。

        ARG  CODE_VERSION=latest
        FROM base:${CODE_VERSION}
        CMD  /code/run-app

        FROM extras:${CODE_VERSION}
        CMD  /code/run-extras

* 在FROM之前声明的ARG在构建阶段之外，因此在FROM之后的任何指令中都不能使用它。 要使用在第一个FROM之前声明的ARG的默认值，请使用没有构建阶段内的值的ARG指令：

        ARG VERSION=latest
        FROM busybox:$VERSION
        ARG VERSION
        RUN echo $VERSION > image_version

##### 11.2.LABEL

您可以为镜像添加标签，以帮助按项目组织镜像，记录许可信息，帮助实现自动化或出于其他原因。 对于每个标签，添加以LABEL开头并带有一个或多个键值对的行。 以下示例显示了不同的可接受格式。 内容包括解释性意见。

注意：必须引用带空格的字符串或必须转义空格。 内引号字符（“）也必须转义。

    LABEL com.example.version="0.0.1-beta"
    LABEL vendor1="ACME Incorporated"
    LABEL vendor2=ZENITH\ Incorporated
    LABEL com.example.release-date="2015-02-12"
    LABEL com.example.version.is-production=""

图像可以有多个标签。 在Docker 1.10之前，建议将所有标签组合到单个LABEL指令中，以防止创建额外的层。 这不再是必需的，但仍然支持组合标签。

    LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"

上面也可以写成下面这样

    LABEL vendor=ACME\ Incorporated \
          com.example.is-beta= \
          com.example.is-production="" \
          com.example.version="0.0.1-beta" \
          com.example.release-date="2015-02-12"

后续将会有一章的内容讲解Docker对象LABEL


##### 11.3.RUN

RUN指令有两种形式
* ` RUN <command>`（shell格式，命令以shell的形式运行，在Linux系统上默认是`/bin/sh -c`, windows上是`cmd /S /C`）
* `RUN ["executable", "param1", "param2"]`(exec 形式)

RUN指令将在当前镜像之上的新镜像层中执行任何命令并提交结果。 生成的已提交镜像将用于`Dockerfile`中的下一步。

分层`RUN`指令和生成提交符合Docker的核心概念，其中提交开销小，并且可以从镜像历史中的任何点创建容器，就像源代码控制一样。

exec执行形式可以避免`shell`字符串重写，并使用不包含指定`shell`可执行文件的基本镜像来运行`RUN`指令。

可以使用`SHELL`命令更改`shell`形式的默认`shell`指令。

在shell形式的指令中，您可以使用`\`（反斜杠）将单个`RUN`指令继续到下一行。 例如，考虑以下两行：

    RUN /bin/bash -c 'source $HOME/.bashrc; \
    echo $HOME'

上面的内容相当于下面的这一行

    RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'

###### 注意：

* 要使用除`“/ bin / sh”`之外的其他shell，请使用传入所需shell的exec形式。 例如，`RUN [“/ bin / bash”，“ - c”，“echo hello”`

* exec形式被解析为JSON数组，这意味着您必须在单词而不是单引号（'）周围使用双引号（“）。

* 与shell形式不同，exec形式不会调用命令shell。 这意味着不会发生正常的shell处理。 例如，`RUN [“echo”，“$ HOME”]`不会对`$HOME`执行变量替换。 如果你想要shell处理，那么要么使用shell形式，要么直接执行shell，例如：`RUN [“sh”，“ - c”，“echo $ HOME”]`。 当使用exec形式并直接执行shell时，就像shell形式的情况一样，它是执行环境变量扩展的shell，而不是docker。

* 在JSON形式中，必须转义反斜杠。 这在反斜杠是路径分隔符的Windows上尤为重要。 由于不是有效的JSON，以下行将被视为shell形式，并以意外方式失败：`RUN [“c：\windows\system32\tasklist.exe”]`; 此示例的正确语法是：`RUN [“C：\\\\窗户SYSTEM32\\ tasklist.exe“]`

在使用反斜杠分隔的多行上拆分长或复杂的RUN语句，以使Dockerfile更具可读性，可理解性和可维护性。


###### Run指令

1.APT-GET

RUN指令的最常用的使用案列可能就是`apt-get`应用，因为它在安装包的时候经常使用，所以`RUN apt-get`命令有几个需要注意的问题。

避免`RUN apt-get`升级和`dist-upgrade`，因为父映像中的许多“基本”包无法在非特权容器内升级。 如果父镜像中包含的包已过期，请与其维护人员联系。 如果您知道需要更新的特定包foo，请使用`apt-get install -y foo`自动更新。

始终将RUN apt-get update与apt-get install结合在同一个RUN语句中。 例如：

     RUN apt-get update && apt-get install -y \
            package-bar \
            package-baz \
            package-foo

在RUN语句中单独使用`apt-get update`会导致缓存问题，以及后续的`apt-get install`执行失败。 例如，假设你有一个Dockerfile：

    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl


构建镜像后，所有镜像层都在Docker缓存中。 假想您稍后通过添加额外的包来修改`apt-get install`：

    FROM ubuntu:14.04
        RUN apt-get update
        RUN apt-get install -y curl nginx

Docker将初始和修改的指令视为相同，并重用前面步骤中的缓存。 因此，不会执行apt-get更新，因为构建使用缓存版本。 由于apt-get更新未运行，因此您的构建可能会获得curl和nginx包的过时版本。

使用RUN apt-get update && apt-get install -y可确保您的Dockerfile安装最新的软件包版本，无需进一步编码或手动干预。 这种技术称为“缓存清除”。 您还可以通过指定包版本来实现缓存清除。 这称为版本固定，例如：

    RUN apt-get update && apt-get install -y \
            package-bar \
            package-baz \
            package-foo=1.3.*

版本固定会强制构建以检索特定版本，而不管缓存中的内容是什么。 此技术还可以减少由于所需包中的意外更改而导致的故障。

下面是一个结构良好的RUN指令，它演示了所有apt-get建议。

    RUN apt-get update && apt-get install -y \
        aufs-tools \
        automake \
        build-essential \
        curl \
        dpkg-sig \
        libcap-dev \
        libsqlite3-dev \
        mercurial \
        reprepro \
        ruby1.9.1 \
        ruby1.9.1-dev \
        s3cmd=1.1.* \
     && rm -rf /var/lib/apt/lists/*

`s3cmd`参数指定版本`1.1.*`。 如果映像以前使用的是旧版本，则指定新版本会导致`apt-get update`缓存失效，并确保安装新版本。 列出每行的包也可以防止包重复中的错误。

此外，当您通过删除/var/lib/apt/lists清理apt缓存时，它会减小镜像大小，因为apt缓存不存储在该镜像层中。 由于RUN语句以apt-get update开头，因此在apt-get install之前始终刷新包缓存。

 - 官方Debian和Ubuntu映像自动运行apt-get clean，因此不需要显式调用

2.使用pipes

某些RUN命令取决于使用管道符（|）将一个命令的输出传递到另一个命令的能力，如下例所示：

    RUN wget -O - https://some.site | wc -l > /number
    
Docker使用/ bin/sh -c解释器执行这些命令，该解释器仅评估管道中最后一个操作的退出代码以确定成功。 在上面的示例中，只要wc -l命令成功，即使wget命令失败，此构建步骤也会成功并生成新映像。

如果您希望命令因管道中任何阶段的错误而失败，请预先设置-o pipefail &&以确保意外错误可防止构建无意中成功。 例如：

    RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]

###### CMD指令

CMD指令有三种格式

* CMD ["executable","param1","param2"] (exec形式, 优先考虑形式)
* CMD ["param1","param2"] (默认参数入口点形式)
* CMD command param1 param2 (shell命令形式)

Dockerfile中只能有一条CMD指令。 如果列出多个CMD，则只有最后一个CMD才会生效。

CMD的主要目的是为执行容器提供默认值。 这些默认值可以包含可执行文件，也可以省略可执行文件，在这种情况下，您还必须指定ENTRYPOINT指令。

注意：如果使用CMD为ENTRYPOINT指令提供默认参数，则应使用JSON数组格式指定CMD和ENTRYPOINT指令。

注意：exec表单被解析为JSON数组，这意味着必须使用双引号（“）来围绕单词而不是单引号（'）

注意：与shell表单不同，exec表单不会调用命令shell。 这意味着不会发生正常的shell处理。 例如，CMD [“echo”，“$ HOME”]不会对$ HOME进行变量替换。 如果你想要shell处理，那么要么使用shell表单，要么直接执行shell，例如：CMD [“sh”，“ - c”，“echo $ HOME”]。 当使用exec表单并直接执行shell时，就像shell表单的情况一样，它是执行环境变量扩展的shell，而不是docker。

在shell或exec格式中使用时，CMD指令设置运行映像时要执行的命令。

如果你使用CMD的shell形式，那么<command>将在/bin/sh -c中执行：

        FROM ubuntu
        CMD echo "This is a test." | wc -

如果要在没有shell的情况下运行<command>，则必须将该命令表示为JSON数组，并提供可执行文件的完整路径。 此数组形式是CMD的首选格式。 任何其他参数必须在数组中单独表示为字符串：

        FROM ubuntu
        CMD ["/usr/bin/wc","--help"]

如果您希望容器每次都运行相同的可执行文件，那么您应该考虑将ENTRYPOINT与CMD结合使用。

如果用户指定了docker run的参数，那么它们将覆盖CMD中指定的默认值。

注意：不要将RUN与CMD混淆。 RUN实际上运行一个命令并提交结果; CMD在构建时不执行任何操作，但指定了镜像的预期命令。

CMD指令应该用于运行镜像包含的软件以及任何参数。 CMD应该几乎总是以CMD [“executable”，“param1”，“param2”......]的形式使用。 因此，如果图像用于服务，例如Apache和Rails，则可以运行类似CMD [“apache2”，“ - DFOREGROUND”]的内容。 实际上，建议将这种形式的指令用于任何基于服务的图像。

在大多数其他情况下，应该给CMD一个交互式shell，比如bash，python和perl。 例如，CMD [“perl”，“ - de0”]，CMD [“python”]或CMD [“php”，“ - a”]。 使用这个表单意味着当你执行像docker run -it python这样的东西时，你将被放入一个可用的shell中，准备好了。 CMD应该很少以CMD [“param”，“param”]的方式与ENTRYPOINT一起使用，除非您和您的预期用户已经非常熟悉ENTRYPOINT的工作原理。

###### EXPOSE指令

    EXPOSE <port> [<port>/<protocol>...]

EXPOSE指令是Docker容器在运行时用来指定监听的端口。 指定监听端口可以是TCP，也可是UDP，如果未指定协议，则默认为TCP。

EXPOSE指令实际上不发布端口。 它作为构建镜像的人和运行容器的人之间的一种文档，用于发布要发布的端口。 要在运行容器时实际发布端口，请在docker run上使用-p标志发布和映射一个或多个端口，或使用-P标志发布所有公开的端口并将它们映射到高阶端口。

默认情况下，EXPOSE假定为TCP。 您还可以指定UDP：

    EXPOSE 80/udp

要在TCP和UDP上公开，请包含两行：

    EXPOSE 80/tcp
    EXPOSE 80/udp

在这种情况下，如果将-P与docker run一起使用，则端口将针对TCP公开一次，针对UDP公开一次。 请记住，-P在主机上使用短暂的高阶主机端口，因此TCP和UDP的端口不同。

无论EXPOSE设置如何，您都可以使用-p标志在运行时覆盖它们。 例如

    docker run -p 80:80/tcp -p 80:80/udp ...
    
docker network命令支持创建用于容器之间通信的网络，而无需公开或发布特定端口，因为连接到网络的容器可以通过任何端口相互通信。 

###### ENV指令

    ENV <key> <value>
    ENV <key>=<value> ...

ENV指令将环境变量<key>设置为值<value>。 此值将在构建阶段中所有后续指令的环境中，并且也可以在许多内联替换。

ENV指令有两种形式。 第一种形式ENV <key> <value>，将单个变量设置为一个值。 第一个空格后面的整个字符串将被视为<value> - 包括空格字符。 该值将针对其他环境变量进行解释，因此如果未对其进行转义，则将删除引号字符。
    
第二种形式ENV <key> = <value> ...允许一次设置多个变量。 请注意，第二种形式在语法中使用等号（=），而第一种形式则不然。 与命令行解析一样，引号和反斜杠可用于在值内包含空格。
    
例如：

    ENV myName="John Doe" myDog=Rex\ The\ Dog \
        myCat=fluffy
    
或者

    ENV myName John Doe
    ENV myDog Rex The Dog
    ENV myCat fluffy

将在最终镜像中产生相同的净结果。

当从生成的镜像中运行容器时，使用ENV设置的环境变量将保持不变。 您可以使用docker inspect查看值，并使用docker run --env <key> = <value>更改它们。

注意：环境持久性可能会导致意外的副作用。 例如，设置ENV DEBIAN_FRONTEND非交互式可能会使基于Debian的图像上的apt-get用户感到困惑。 要为单个命令设置值，请使用RUN <key> = <value> <command>。
    
为了使新软件更易于运行，您可以使用ENV更新容器安装的软件的PATH环境变量。 例如，ENV PATH/usr/local/nginx/bin：$PATH确保CMD [“nginx”]正常工作。

ENV指令对于提供特定于您希望容纳的服务的必需环境变量也很有用，例如Postgres的PGDATA。

最后，ENV还可用于设置常用的版本号，以便更容易维护版本颠簸，如下例所示：

    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH

与在程序中使用常量变量（与硬编码值相反）类似，此方法允许您更改单个ENV指令以自动神奇地破坏容器中的软件版本。

每条ENV线都会创建一个新的中间层，就像RUN命令一样。 这意味着即使您在将来的镜像中取消设置环境变量，它仍然会在此镜像层中保留，并且可以转储其值。 您可以通过创建如下所示的Dockerfile来测试它，然后构建它。

    FROM alpine
    ENV ADMIN_USER="mark"
    RUN echo $ADMIN_USER > ./mark
    RUN unset ADMIN_USER
    CMD sh

    docker run --rm -it test sh echo $ADMIN_USER
    mark

要防止这种情况，并且确实取消设置环境变量，请使用带有shell命令的RUN命令，在单个镜像层中设置，使用和取消设置变量all。 您可以将命令分开; 要么＆&。 如果您使用第二种方法，并且其中一个命令失败，则docker构建也会失败。 这通常是一个好主意。 使用\作为Linux Dockerfiles的行继续符可以提高可读性。 您还可以将所有命令放入shell脚本中，并使用RUN命令运行该shell脚本。

    FROM alpine
    RUN export ADMIN_USER="mark" \
        && echo $ADMIN_USER > ./mark \
        && unset ADMIN_USER
    CMD sh

    docker run --rm -it test sh echo $ADMIN_USER


###### ADD指令和Copy指令

###### add指令

Add指令有两种格式

    ADD [--chown=<user>:<group>] <src>... <dest>
    ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]

注意：--chown功能仅在用于构建Linux容器的Dockerfiles上受支持，并且不适用于Windows容器。 由于用户和组所有权概念不能在Linux和Windows之间进行转换，因此使用/etc/ passwd和/etc/group将用户名和组名转换为ID会限制此功能仅适用于基于Linux OS的容器。

ADD指令从<src>复制新文件，目录或远程文件URL，并将它们添加到路径<dest>的镜像文件系统中。

可以指定多个<src>资源，但如果它们是文件或目录，则它们的路径将被解释为相对于构建上下文的源。

每个<src>可能包含通配符，匹配将使用Go的filepath.Match规则完成。 例如：

    ADD hom* /mydir/        # adds all files starting with "hom"
    ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

<dest>是绝对路径，或相对于WORKDIR的路径，源将在目标容器中复制到该路径中。

    ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
    ADD test /absoluteDir/         # adds "test" to /absoluteDir/

当添加一个包含特定字符的文件或者目录【例如"["或者"]"】，需要按照Golang规则转义这些路径，以防止它们被视为匹配模式。 例如，要添加名为arr[0].txt的文件，请使用以下命令：

    ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/

除非可选的--chown标志指定给定用户名，组名或UID/GID组合以请求添加内容的特定所有权，否则将使用UID和GID为0创建所有新文件和目录。 --chown标志的格式允许用户名和组名字符串或任意组合的直接整数UID和GID。 提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。 如果提供了用户名或组名，则容器的根文件系统/etc/passwd和/etc/group文件将分别用于执行从名称到整数UID或GID的转换。 以下示例显示了--chown标志的有效定义：

    ADD --chown=55:mygroup files* /somedir/
    ADD --chown=bin files* /somedir/
    ADD --chown=1 files* /somedir/
    ADD --chown=10:11 files* /somedir/
    
如果容器根文件系统不包含/etc/passwd或/etc/group文件，并且在--chown标志中使用了用户名或组名，则构建将在ADD操作上失败。 使用数字ID不需要查找，也不依赖于容器根文件系统内容。

在<src>是远程文件URL的情况下，目标将具有600的权限。如果正在检索的远程文件具有HTTP Last-Modified标头，则该标头的时间戳将用于设置目标文件上的mtime。 但是，与其他ADD期间处理的任何文件一样，mtime将不包含确定文件是否已更改和缓存更新。

*注意：如果通过将Dockerfile传递给STDIN（docker build - <somefile）来构建，则没有构建上下文，因此Dockerfile只能包含基于URL的ADD指令。 您还可以通过STDIN传递压缩存档：（docker build - <archive.tar.gz），存档根目录下的Dockerfile，存档的其余部分将用作构建的上下文。

*注意：如果您的URL文件使用身份验证进行保护，则需要使用`RUN wget`，`RUN curl`或使用容器内的其他工具，因为`ADD`指令不支持身份验证。

*注意：如果`<src>`的内容已更改，则第一个遇到的`ADD`指令将使来自`Dockerfile`的所有后续指令的高速缓存无效。 这包括使RUN指令的高速缓存无效。 
    

ADD指令遵循下面的原则

* <src>路径必须位于构建的上下文中;你不能添加../something/something，因为docker构建的第一步是将上下文目录（和子目录）发送到docker守护进程。

* 如果<src>是URL且<dest>不以尾部斜杠结尾，则从URL下载文件并将其复制到<dest>。

* 如果<src>是URL并且<dest>以尾部斜杠结尾，则从URL推断文件名，并将文件下载到<dest>/<filename>。例如，ADD http://example.com/foobar/将创建文件/ foobar。 URL必须具有非常重要的路径，以便在这种情况下可以发现适当的文件名（http://example.com将不起作用）。

* 如果<src>是目录，则复制目录的全部内容，包括文件系统元数据。

*注意：你的内容将不会被复制，仅仅复制它的内容

如果<src>是可识别的压缩格式（identity，gzip，bzip2或xz）的本地tar存档，则将其解压缩为目录。 远程URL中的资源未解压缩。 复制或解压缩目录时，它具有与tar -x相同的行为，结果是以下联合：
    
1.无论在目的地路径上存在什么，
2.源树的内容，在逐个文件的基础上解决了有利于“2.”的冲突。

*注意：文件是否被识别为可识别的压缩格式仅基于文件的内容而不是文件的名称来完成。 例如，如果空文件恰好以.tar.gz结尾，则不会将其识别为压缩文件，也不会生成任何类型的解压缩错误消息，而是将文件简单地复制到目标。

如果<src>是任何其他类型的文件，则将其与元数据一起单独复制。在这种情况下，如果<dest>以尾部斜杠/结束，则将其视为目录，<src>的内容将写入<dest>/ base（<src>）。

如果直接或由于使用通配符指定了多个<src>资源，则<dest>必须是目录，并且必须以斜杠/结尾。

如果<dest>不以尾部斜杠结束，则它将被视为常规文件，<src>的内容将写入<dest>。

如果<dest>不存在，则会在其路径中创建所有缺少的目录。

###### copy指令

copy指令有两种格式

    COPY [--chown=<user>:<group>] <src>... <dest>
    COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] (这种形式需要包含空格的路径)

*注意：--chown功能仅在用于构建Linux容器的Dockerfiles上受支持，并且不适用于Windows容器。 由于用户和组所有权概念不能在Linux和Windows之间进行转换，因此使用/etc/passwd和/etc/group将用户名和组名转换为ID会限制此功能仅适用于基于Linux OS的容器。

COPY指令从<src>复制新文件或目录，并将它们添加到路径<dest>的容器的文件系统中。
    
可以指定多个<src>资源，但文件和目录的路径将被解释为相对于构建上下文的源。
    
每个<src>可能包含通配符，匹配将使用Go的filepath.Match规则完成。 例如：
    

    COPY hom* /mydir/       
    COPY hom?.txt /mydir/    
    
<dest>是绝对路径，或相对于WORKDIR的路径，源将在目标容器中复制到该路径中。
    
    COPY test relativeDir/   
    COPY test /absoluteDir/ 

复制包含特殊字符（例如[和]）的文件或目录时，需要按照Golang规则转义这些路径，以防止它们被视为匹配模式。 例如，要复制名为arr [0] .txt的文件，请使用以下命令：

    COPY arr[[]0].txt /mydir/

除非可选的--chown标志指定给定用户名，组名或UID/GID组合以请求复制内容的特定所有权，否则将使用UID和GID为0创建所有新文件和目录。 --chown标志的格式允许用户名和组名字符串或任意组合的直接整数UID和GID。 提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。 如果提供了用户名或组名，则容器的根文件系统/etc/passwd和/etc/group文件将分别用于执行从名称到整数UID或GID的转换。 以下示例显示了--chown标志的有效定义：

    COPY --chown=55:mygroup files* /somedir/
    COPY --chown=bin files* /somedir/
    COPY --chown=1 files* /somedir/
    COPY --chown=10:11 files* /somedir/

如果容器根文件系统不包含/etc/passwd或/etc/group文件，并且在--chown标志中使用了用户名或组名，则构建将在COPY操作上失败。 使用数字ID不需要查找，也不依赖于容器根文件系统内容。

*注意：如果使用STDIN（docker build - <somefile）构建，则没有构建上下文，因此无法使用COPY。

可选地，COPY接受一个标志--from = <name | index>，可用于将源位置设置为先前的构建阶段（使用FROM .. AS <name>创建），而不是由发送的构建上下文用户。 该标志还接受为使用FROM指令启动的所有先前构建阶段分配的数字索引。 如果找不到具有指定名称的构建阶段，则尝试使用具有相同名称的图像。
    
copy指令遵循下面的规则

* <src>路径必须位于构建的上下文中; 你不能COPY ../something / something，因为docker build的第一步是将上下文目录（和子目录）发送到docker守护进程。
* 如果<src>是目录，则复制目录的全部内容，包括文件系统元数据。

*注意：不复制目录本身，只复制其内容。

** 如果<src>是任何其他类型的文件，则将其与元数据一起单独复制。在这种情况下，如果<dest>以尾部斜杠/结束，则将其视为目录，<src>的内容将写入<dest>/ base（<src>）。

** 如果直接或由于使用通配符指定了多个<src>资源，则<dest>必须是目录，并且必须以斜杠/结尾。

** 如果<dest>不以尾部斜杠结束，则它将被视为常规文件，<src>的内容将写入<dest>。

** 如果<dest>不存在，则会在其路径中创建所有缺少的目录。

尽管ADD和COPY在功能上相似，但一般来说，COPY是优选的。 那是因为它比ADD更透明。 COPY仅支持将本地文件基本复制到容器中，而ADD具有一些功能（如仅限本地的tar提取和远程URL支持），这些功能并不是很明显。 因此，ADD的最佳用途是将本地tar文件自动提取到镜像中，如ADD rootfs.tar.xz /中所示。

如果您有多个使用上下文中不同文件的Dockerfile步骤，请单独复制它们，而不是一次复制它们。 这可确保每个步骤的构建缓存仅在特定所需文件更改时失效（强制重新执行该步骤）。

例如： 

    COPY requirements.txt /tmp/
    RUN pip install --requirement /tmp/requirements.txt
    COPY . /tmp/

与放置COPY相比，RUN步骤的缓存失效更少。 /tmp/之前。

由于图像大小很重要，因此强烈建议不要使用ADD从远程URL中提取包。 你应该使用curl或wget代替。 这样，您可以删除提取后不再需要的文件，也不必在图像中添加其他图层。 例如，你应该避免做以下事情：

    ADD http://example.com/big.tar.xz /usr/src/things/
    RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
    RUN make -C /usr/src/things all

也可以像下面这样做

    RUN mkdir -p /usr/src/things \
        && curl -SL http://example.com/big.tar.xz \
        | tar -xJC /usr/src/things \
        && make -C /usr/src/things all

对于不需要ADD的tar自动提取功能的其他项目（文件，目录），应始终使用COPY。


###### ENTRYPOINT

ENTRYPOINT命令的两种格式

* exec格式

    ENTRYPOINT ["executable", "param1", "param2"] 
    
* shell格式

    ENTRYPOINT command param1 param2 

ENTRYPOINT允许您配置将作为可执行文件运行的容器。

例如，以下将使用其默认内容启动nginx，侦听端口80：

    docker run -i -t --rm -p 80:80 nginx

docker run <image>的命令行参数将在exec中的所有元素形成ENTRYPOINT后附加，并将覆盖使用CMD指定的所有元素。 这允许将参数传递给入口点，即docker run <image> -d将-d参数传递给入口点。 您可以使用docker run --entrypoint标志覆盖ENTRYPOINT指令。
    
shell表单阻止使用任何CMD或运行命令行参数，但缺点是ENTRYPOINT将作为/bin/sh -c的子命令启动，它不传递信号。 这意味着可执行文件不是容器的PID 1 - 并且不会接收Unix信号 - 因此您的可执行文件将不会从docker stop <container>接收SIGTERM。
    
只有Dockerfile中的最后一个ENTRYPOINT指令才会生效。

###### EXEC格式的ENTRYPOINT例子

您可以使用ENTRYPOINT的exec形式设置相当稳定的默认命令和参数，然后使用任一形式的CMD来设置更可能更改的其他默认值。

    FROM ubuntu
    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

运行容器时，您可以看到top是唯一的进程：

    $ docker run -it --rm --name test  top -H
    top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
    Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
    KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
        1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top

测试进一步的结果，你可以使用`docker exec`

    $ docker exec -it test ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
    root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux

你可以使用`docker stop test`优雅地请求`stop`来关闭

以下Dockerfile显示使用ENTRYPOINT在前台运行Apache（即，作为PID 1）：

    FROM debian:stable
    RUN apt-get update && apt-get install -y --force-yes apache2
    EXPOSE 80 443
    VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
    ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
 
如果需要为单个可执行文件编写启动脚本，可以使用exec和gosu命令确保最终的可执行文件接收Unix信号：

    #!/usr/bin/env bash
    set -e

    if [ "$1" = 'postgres' ]; then
        chown -R postgres "$PGDATA"

        if [ -z "$(ls -A "$PGDATA")" ]; then
            gosu postgres initdb
        fi

        exec gosu postgres "$@"
    fi

    exec "$@"

最后，如果您需要在关机时进行一些额外的清理（或与其他容器通信），或者协调多个可执行文件，您可能需要确保ENTRYPOINT脚本接收Unix信号，传递它们，然后 做了一些工作：

    #!/bin/sh
    # Note: I've written this using sh so it works in the busybox container too

    # USE the trap if you need to also do manual cleanup after the service is stopped,
    #     or need to start multiple services in the one container
    trap "echo TRAPed signal" HUP INT QUIT TERM

    # start service in background here
    /usr/sbin/apachectl start

    echo "[hit enter key to exit] or run 'docker stop <container>'"
    read

    # stop service and clean up here
    echo "stopping apache"
    /usr/sbin/apachectl stop

    echo "exited $0"
    
如果使用docker run -it -rm -p 80:80 --name test apache运行此映像，则可以使用docker exec或docker top检查容器的进程，然后让脚本停止Apache：

    $ docker exec -it test ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.1  0.0   4448   692 ?        Ss+  00:42   0:00 /bin/sh /run.sh 123 cmd cmd2
    root        19  0.0  0.2  71304  4440 ?        Ss   00:42   0:00 /usr/sbin/apache2 -k start
    www-data    20  0.2  0.2 360468  6004 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
    www-data    21  0.2  0.2 360468  6000 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
    root        81  0.0  0.1  15572  2140 ?        R+   00:44   0:00 ps aux
    $ docker top test
    PID                 USER                COMMAND
    10035               root                {run.sh} /bin/sh /run.sh 123 cmd cmd2
    10054               root                /usr/sbin/apache2 -k start
    10055               33                  /usr/sbin/apache2 -k start
    10056               33                  /usr/sbin/apache2 -k start
    $ /usr/bin/time docker stop test
    test
    real	0m 0.27s
    user	0m 0.03s
    sys	0m 0.03s

注意：您可以使用--entrypoint覆盖ENTRYPOINT设置，但这只能将二进制设置为exec（不会使用sh -c）。

注意：exec形式被解析为JSON数组，这意味着您必须使用双引号（“）来围绕单词而不是单引号（'）。


注意：与shell表单不同，exec表单不会调用命令shell。 这意味着不会发生正常的shell处理。 例如，ENTRYPOINT [“echo”，“$ HOME”]不会对$ HOME执行变量替换。 如果你想要shell处理，那么要么使用shell表单，要么直接执行shell，例如：ENTRYPOINT [“sh”，“ - c”，“echo $ HOME”]。 当使用exec表单并直接执行shell时，就像shell表单的情况一样，它是执行环境变量扩展的shell，而不是docker。

###### shell形式的ENTRYPOINT案列

您可以为ENTRYPOINT指定一个纯字符串，它将在/ bin / sh -c中执行。 此表单将使用shell处理来替换shell环境变量，并将忽略任何CMD或docker运行命令行参数。 要确保docker stop能正确发出任何长时间运行的ENTRYPOINT可执行文件，你需要记住用exec启动它：

    FROM ubuntu
    ENTRYPOINT exec top -b
    
运行此镜像时，您将看到单个PID 1进程：

    docker run -it --rm --name test top
    Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
    CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
    Load average: 0.08 0.03 0.05 2/98 6
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     R     3164   0%   0% top -b

哪个将在docker stop上干净地退出：

    $ /usr/bin/time docker stop test
    test
    real	0m 0.20s
    user	0m 0.02s
    sys	0m 0.04s

如果您忘记将exec添加到ENTRYPOINT的开头：

    FROM ubuntu
    ENTRYPOINT top -b
    CMD --ignored-param1

然后，您可以运行它（为下一步命名）：

    $ docker run -it --name test top --ignored-param2
    Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
    CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
    Load average: 0.01 0.02 0.05 2/101 7
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
        7     1 root     R     3164   0%   0% top -b
    
您可以从top的输出中看到指定的ENTRYPOINT不是PID 1。

如果然后运行docker stop test，容器将不会干净地退出 -  stop命令将被强制在超时后发送SIGKILL：    
    
    $ docker exec -it test ps aux
    PID   USER     COMMAND
        1 root     /bin/sh -c top -b cmd cmd2
        7 root     top -b
        8 root     ps aux
    $ /usr/bin/time docker stop test
    test
    real	0m 10.19s
    user	0m 0.04s
    sys	0m 0.03s
    
###### 理解CMD和ENTRYparty如何交互

CMD和ENTRYPOINT指令都定义了运行容器时执行的命令。 很少有规则描述他们的协作。

* Dockerfile应至少指定一个CMD或ENTRYPOINT命令。

* 使用容器作为可执行文件时，应定义ENTRYPOINT。

* CMD应该用作为ENTRYPOINT命令定义默认参数或在容器中执行ad-hoc命令的方法。

* 使用备用参数运行容器时，将覆盖CMD。


ENTRYPOINT的最佳用途是设置镜像的主命令，允许该镜像像该命令一样运行（然后使用CMD作为默认标志）。
    
让我们从命令行工具s3cmd的镜像示例开始：
    
       ENTRYPOINT ["s3cmd"]
       CMD ["--help"] 

现在可以像这样运行图像来显示命令的帮助：

    $ docker run s3cmd

或使用正确的参数执行命令：

    $ docker run s3cmd ls s3://mybucket

这很有用，因为镜像名称可以兼作二进制文件的引用，如上面的命令所示。

ENTRYPOINT指令也可以与辅助脚本结合使用，使其能够以与上述命令类似的方式运行，即使启动该工具可能需要多个步骤。

例如，Postgres官方镜像使用以下脚本作为其ENTRYPOINT：

    #!/bin/bash
    set -e

    if [ "$1" = 'postgres' ]; then
        chown -R postgres "$PGDATA"

        if [ -z "$(ls -A "$PGDATA")" ]; then
            gosu postgres initdb
        fi

        exec gosu postgres "$@"
    fi

    exec "$@"

将app配置为PID 1：此脚本使用exec Bash命令，以便最终运行的应用程序成为容器的PID 1.这允许应用程序接收发送到容器的任何Unix信号。

帮助程序脚本被复制到容器中并通过容器启动时的ENTRYPOINT运行：

    COPY ./docker-entrypoint.sh /
    ENTRYPOINT ["/docker-entrypoint.sh"]
    CMD ["postgres"]

该脚本允许用户以多种方式与Postgres交互。

它可以简单地启动Postgres：

    $ docker run postgres

或者，它可用于运行Postgres并将参数传递给服务器：

    $ docker run postgres postgres --help

最后，它还可以用来启动一个完全不同的工具，比如Bash：

    $ docker run --rm -it postgres bash


###### VOLUME

    VOLUME ["/data"]

VOLUME指令创建具有指定名称的安装点，并将其标记为从本机主机或其他容器保存外部安装的卷。 该值可以是JSON数组，VOLUME [“/var/log/”]或具有多个参数的纯字符串，例如VOLUME/var/log或VOLUME/var/log/var/db。

docker run命令使用基础映像中指定位置存在的任何数据初始化新创建的卷。 例如，请考虑以下Dockerfile片段：

    FROM ubuntu
    RUN mkdir /myvol
    RUN echo "hello world" > /myvol/greeting
    VOLUME /myvol

此Dockerfile会生成一个映像，该映像会导致docker run在/ myvol上创建新的挂载点，并将greeting文件复制到新创建的卷中。

VOLUME指令应用于公开由docker容器创建的任何数据库存储区域，配置存储或文件/文件夹。 强烈建议您将VOLUME用于镜像的任何可变和/或用户可维修部分。

有关指定卷的说明
关于Dockerfile中的卷，请记住以下事项。

* 基于Windows的容器上的卷：使用基于Windows的容器时，容器中卷的目标必须是以下之一：
** 不存在或空目录
** C以外的驱动器：

* 从Dockerfile中更改卷：如果任何构建步骤在声明后更改卷内的数据，那么这些更改将被丢弃。

* JSON格式：列表被解析为JSON数组。您必须用双引号（“）而不是单引号（'）括起来。

* 主机目录在容器运行时声明：主机目录（mountpoint）本质上是依赖于主机的。这是为了保持镜像的可移植性，因为不能保证给定的主机目录在所有主机上都可用。因此，您无法从Dockerfile中安装主机目录。 VOLUME指令不支持指定host-dir参数。您必须在创建或运行容器时指定安装点。


###### USER

    USER <user>[:<group>] or
    USER <UID>[:<GID>]

USER指令设置用户名（或UID）以及可选的用户组（或GID），以便在运行映像时以及Dockerfile中跟随它的任何RUN，CMD和ENTRYPOINT指令时使用。

警告：当用户没有主组时，将使用根组运行映像（或下一条指令）。

在Windows上，如果用户不是内置帐户，则必须先创建用户。 这可以使用作为Dockerfile一部分调用的net user命令来完成。

     FROM microsoft/windowsservercore
        # Create Windows user in the container
        RUN net user /add patrick
        # Set it for subsequent commands
        USER patrick

如果服务可以在没有权限的情况下运行，请使用USER更改为非root用户。 首先在Dockerfile中创建用户和组，例如RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres。

考虑一个显式的UID / GID

镜像中的用户和组被分配了非确定性UID/GID，因为无论镜像重建如何，都会分配“下一个”UID/GID。因此，如果它很重要，您应该分配一个显式的UID/GID。

由于Go存档/tar包处理稀疏文件时未解决的错误，尝试在Docker容器内创建具有非常大的UID的用户可能导致磁盘耗尽，因为容器层中的/var/log/faillog填充了NULL（\0）个字符。解决方法是将--no-log-init标志传递给useradd。 Debian/Ubuntu adduser包装器不支持此标志。

避免安装或使用sudo，因为它具有不可预测的TTY和可能导致问题的信号转发行为。如果您绝对需要类似于sudo的功能，例如将守护程序初始化为root但将其作为非root运行，请考虑使用“gosu”。

最后，为了减少层次和复杂性，请避免频繁地来回切换USER。

###### WORKDIR

    WORKDIR /path/to/workdir

WORKDIR指令为Dockerfile中的任何RUN，CMD，ENTRYPOINT，COPY和ADD指令设置工作目录。 如果WORKDIR不存在，即使它未在任何后续Dockerfile指令中使用，也将创建它。

WORKDIR指令可以在Dockerfile中多次使用。 如果提供了相对路径，则它将相对于先前WORKDIR指令的路径。 例如：

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd

此Dockerfile中最终pwd命令的输出为/a/b/c。

WORKDIR指令可以解析先前使用ENV设置的环境变量。您只能使用Dockerfile中显式设置的环境变量。例如：

    ENV DIRPATH /path
    WORKDIR $DIRPATH/$DIRNAME
    RUN pwd

此Dockerfile中最后一个pwd命令的输出将是/path/$DIRNAME

为了清晰和可靠，您应该始终使用WORKDIR的绝对路径。 此外，你应该使用WORKDIR而不是像RUN cd那样激增指令...... && do-something，这些指令难以阅读，故障排除和维护。


###### ONBUILD

    ONBUILD [INSTRUCTION]

当镜像用作另一个构建的基础时，ONBUILD指令向镜像添加将在稍后执行的触发指令。触发器将在下游构建的上下文中执行，就好像它是在下游Dockerfile中的FROM指令之后立即插入的一样。

任何构建指令都可以注册为触发器。

如果要构建将用作构建其他镜像的基础的镜像（例如，可以使用特定于用户的配置自定义的应用程序构建环境或守护程序），这将非常有用。

例如，如果您的镜像是可重用的Python应用程序构建器，则需要将应用程序源代码添加到特定目录中，并且可能需要在此之后调用构建脚本。您现在不能只调用ADD和RUN，因为您还无法访问应用程序源代码，并且每个应用程序构建都会有所不同。您可以简单地为应用程序开发人员提供一个样板Dockerfile来复制粘贴到他们的应用程序中，但这样做效率低，容易出错且难以更新，因为它与特定于应用程序的代码混合在一起。

解决方案是使用ONBUILD来注册预先指令，以便在下一个构建阶段运行。

以下是它的工作原理：

1.当遇到ONBUILD指令时，构建器会向正在构建的图像的元数据添加触发器。该指令不会影响当前构建。

2.在构建结束时，所有触发器的列表都存储在映像清单中的OnBuild键下。可以使用docker inspect命令检查它们。

3.稍后，可以使用FROM指令将图像用作新构建的基础。作为处理FROM指令的一部分，下游构建器查找ONBUILD触发器，并按照它们注册的顺序执行它们。如果任何触发器失败，则中止FROM指令，这反过来导致构建失败。如果所有触发器都成功，则FROM指令完成，并且构建继续照常进行。

4.执行后，触发器将从最终图像中清除。换句话说，它们不是由“大孩子”构建继承的。

例如，您可以添加以下内容：

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

警告：不允许使用ONBUILD ONBUILD链接ONBUILD指令。
警告：ONBUILD指令可能不会触发FROM或MAINTAINER指令。

在当前Dockerfile构建完成后执行ONBUILD命令。 ONBUILD在从当前镜像派生的任何子镜像中执行。将ONBUILD命令视为父Dockerfile为子Dockerfile提供的指令。

Docker构建在子Dockerfile中的任何命令之前执行ONBUILD命令。

ONBUILD对于将从给定镜像构建的镜像非常有用。例如，您可以使用ONBUILD作为语言堆栈映像，在Dockerfile中构建使用该语言编写的任意用户软件，如Ruby的ONBUILD变体中所示。

从ONBUILD构建的镜像应该获得一个单独的标记，例如：ruby：1.9-onbuild或ruby：2.0-onbuild。

将ADD或COPY放入ONBUILD时要小心。如果新构建的上下文缺少正在添加的资源，则“onbuild”映像将发生灾难性故障。如上所述，添加单独的标记有助于通过允许Dockerfile作者做出选择来缓解这种情况。




























