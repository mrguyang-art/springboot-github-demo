# Spring Boot GitHub DevOps Demo

这是一个基于 Java 21、Spring Boot 和 Maven 的示例项目，用来演示从应用开发、自动化测试到容器镜像发布的基本 DevOps 流程。

项目包含 Spring Boot REST API、GitHub Actions CI、GitHub Pages 静态网站、Docker，以及通过 GitHub Releases 和 GitHub Container Registry（GHCR）发布版本。

## 功能概览

- **Spring Boot REST API**：提供欢迎消息接口和 Actuator 健康检查接口。
- **自动化 CI**：Push 到 main 或向 main 创建 Pull Request 时，自动执行 Maven 构建和测试。
- **GitHub Pages**：从 main 分支的 /docs 目录直接发布多页面静态项目网站，无需 Pages 专用 Actions 工作流。
- **Docker**：使用 Java 21 JRE 镜像运行打包后的 Spring Boot JAR。
- **版本发布**：推送符合版本命名规则的分支后，自动创建 GitHub Release、上传 JAR，并构建和推送 GHCR 镜像。

## 技术栈

- Java 21
- Spring Boot 3.5.5
- Maven
- Spring Web
- Spring Boot Actuator
- JUnit 5 / Spring Boot Test
- GitHub Actions
- GitHub Pages
- Docker / GitHub Container Registry

## 本地运行

确保本机已安装 Java 21 和 Maven。

~~~bash
mvn spring-boot:run
~~~

应用默认在 http://localhost:8080 启动。

### API 接口

| 方法 | 路径 | 用途 |
|---|---|---|
| GET | /api/hello | 返回欢迎消息 |
| GET | /actuator/health | 返回应用健康状态 |

GET /api/hello 示例响应：

~~~json
{
  "message": "Hello from Spring Boot!",
  "status": "success"
}
~~~

运行自动化测试和完整 Maven 验证：

~~~bash
mvn -B verify
~~~

## GitHub Actions CI

工作流文件：.github/workflows/ci.yml

触发条件：
- Push 到 main
- 创建或更新目标分支为 main 的 Pull Request
- 在 GitHub Actions 页面手动运行

主要步骤：
1. 检出代码。
2. 配置 Temurin Java 21 和 Maven 缓存。
3. 执行 mvn -B verify，编译项目并运行测试。
4. 上传生成的 JAR，Artifact 名称为 springboot-jar，保留 7 天。

可在仓库的 **Actions** 标签页查看工作流运行记录和日志。

## GitHub Pages

静态网站位于 docs/ 目录，包含首页、项目架构、API 文档、服务状态、CI/CD、版本发布、快速开始和关于项目等页面。

本项目使用 **Deploy from a branch** 发布，不需要专门的 Pages GitHub Actions 工作流。仓库设置为：

- **Settings → Pages → Build and deployment → Source**：Deploy from a branch
- **Branch**：main
- **Folder**：/docs

保存设置后，每次向 main 分支推送 docs/ 目录的变更，GitHub Pages 都会自动发布。网站内容由 GitHub Pages 直接托管，不需要构建步骤。

网站地址：

https://mrguyang-art.github.io/springboot-github-demo/

## Docker 本地构建与运行

Dockerfile 位于仓库根目录。它使用 eclipse-temurin:21-jre 作为运行环境，并将 Maven 构建生成的 JAR 复制到镜像中。

先在项目根目录构建应用：

~~~bash
mvn -B package
~~~

构建 Docker 镜像：

~~~bash
docker build -t springboot-github-demo:local .
~~~

启动容器：

~~~bash
docker run --rm -p 8080:8080 springboot-github-demo:local
~~~

然后访问 http://localhost:8080/api/hello。

## GitHub Releases 与 GHCR Packages

发布工作流：.github/workflows/release.yml

工作流会在推送符合版本格式的分支（例如 v1.0.1）时自动运行。它会：
1. 校验版本分支名称。
2. 使用该版本分支的代码执行 mvn -B verify。
3. 上传带版本号的 GitHub Actions Artifact（保留 30 天）。
4. 创建对应的 GitHub Release，自动生成 Release Notes 并附加 JAR。
5. 构建 Docker 镜像并推送到 GHCR。

### 发布新版本

版本分支命名采用语义化版本形式，例如 v1.0.1。建议先确保改动已合并到 main，再从 main 创建版本分支：

~~~bash
git checkout main
git pull origin main
git checkout -b v1.0.1
git push -u origin v1.0.1
~~~

推送版本分支后，**Release and publish** 工作流会自动执行。也可以在 GitHub **Actions → Release and publish → Run workflow** 手动运行：在页面的分支选择器中选择目标版本分支，并将 branch 输入值填写为相同的分支名称（例如 v1.0.1）。

### 发布产物

以 v1.0.0 为例：

- **GitHub Release**：https://github.com/mrguyang-art/springboot-github-demo/releases/tag/v1.0.0
- **JAR 文件**：Release 页面提供 Maven 构建生成的 JAR 下载。
- **GitHub Actions Artifact**：名称为 springboot-jar-1.0.0，保留 30 天。
- **GHCR 镜像标签**：
  - ghcr.io/mrguyang-art/springboot-github-demo:1.0.0
  - ghcr.io/mrguyang-art/springboot-github-demo:1.0
  - ghcr.io/mrguyang-art/springboot-github-demo:latest

拉取并运行指定版本镜像：

~~~bash
docker pull ghcr.io/mrguyang-art/springboot-github-demo:1.0.0
docker run --rm -p 8080:8080 ghcr.io/mrguyang-art/springboot-github-demo:1.0.0
~~~

访问 http://localhost:8080/api/hello 验证服务。

### GHCR 镜像可见性

首次发布后，GHCR Package 可能默认为 Private。若希望其他人无需登录即可拉取镜像，请进入 GitHub **个人主页 → Packages → 对应容器包 → Package settings**，将可见性设置为 **Public**。

如果镜像是 Private，需要先登录 GHCR：

~~~bash
docker login ghcr.io
~~~

## 项目结构

~~~text
.
├── .github/
│   └── workflows/
│       ├── ci.yml          # Maven 构建与测试
│       └── release.yml     # Release、JAR 与 GHCR 镜像发布
├── docs/
│   ├── assets/site.css     # 静态网站公共样式
│   ├── index.html          # 网站首页
│   ├── architecture.html   # 项目架构
│   ├── api.html            # API 文档
│   ├── status.html         # 服务状态说明
│   ├── cicd.html           # CI/CD 流程
│   ├── releases.html       # 版本发布
│   ├── guide.html          # 快速开始
│   └── about.html          # 关于项目
├── src/
│   ├── main/java/com/demo/
│   │   ├── DemoApplication.java
│   │   └── controller/
│   │       └── HelloController.java
│   └── test/java/com/demo/controller/
│       └── HelloControllerTest.java
├── Dockerfile
└── pom.xml
~~~
