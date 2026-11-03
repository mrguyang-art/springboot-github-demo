# Spring Boot + Vue 前后端分离 DevOps Demo

这是一个用于演示前后端分离与 DevOps 流程的示例项目。后端维持 Java 21 + Spring Boot 3.5.5，前端采用 Vue 3 + Element Plus，通过 REST API 使用 HTTP/JSON 通信。两端独立开发、构建和部署。

另外，仓库根目录的 `docs/` 是独立的纯静态 HTML/CSS 文档站点，继续由 GitHub Pages 发布，不属于 Vue 前端，也不参与 Vue 构建。

## 功能概览

- **Vue 前端**：Vue 3、Element Plus、Vite，提供独立的交互式演示页面。
- **Spring Boot 后端**：提供 REST API 和 Actuator 健康检查，保持现有 Java/Spring Boot 技术栈。
- **前后端分离**：前端通过 HTTP/JSON 调用后端；本地开发由 Vite 代理 API，跨域访问由 Spring MVC CORS 配置控制。
- **独立 CI**：GitHub Actions 分别构建并验证 Maven 后端和 Vue 前端。
- **GitHub Pages**：从 main 分支的 /docs 目录直接发布静态项目文档，无需 Pages 专用 Actions 工作流。
- **Docker / Release**：后端继续使用现有 Dockerfile、GitHub Releases 和 GHCR 发布流程。

## 技术栈

| 层次 | 技术 |
|---|---|
| 前端 | Vue 3、Element Plus、Vite、Axios |
| 后端 | Java 21、Spring Boot 3.5.5、Spring Web、Actuator |
| API | REST、HTTP、JSON、Spring MVC CORS |
| 测试 | JUnit 5、Spring Boot Test、Maven |
| DevOps | GitHub Actions、Docker、GitHub Releases、GHCR |
| 静态文档 | HTML、CSS、GitHub Pages |

## 本地运行

### 1. 启动 Spring Boot 后端

确保安装 Java 21 和 Maven，在仓库根目录运行：

~~~bash
mvn spring-boot:run
~~~

后端默认地址：http://localhost:8080

### 2. 启动 Vue 前端

确保安装 Node.js 20+ 和 npm，在另一个终端运行：

~~~bash
cd frontend
npm install
npm run dev
~~~

打开 http://localhost:5173。Vite 将 `/api` 和 `/actuator` 请求代理到 http://localhost:8080。

### 3. 生产构建

后端：

~~~bash
mvn -B verify
~~~

前端：

~~~bash
cd frontend
npm run build
npm run preview
~~~

生产环境如需连接独立部署的后端，可设置 `VITE_API_BASE_URL`，例如：

~~~bash
VITE_API_BASE_URL=https://api.example.com npm run build
~~~

后端 CORS 允许来源可通过环境变量 `APP_CORS_ALLOWED_ORIGINS` 配置，多个来源用逗号分隔。默认允许本地 Vite 开发地址 `http://localhost:5173` 和 `http://127.0.0.1:5173`。

## API 接口

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

## 前后端分离架构

~~~text
浏览器
  |
  | Vue 3 + Element Plus (frontend/)
  | HTTP / JSON
  v
Spring Boot REST API (src/)
  |
  +-- GET /api/hello
  +-- GET /actuator/health

GitHub Pages (docs/)
  +-- 独立静态 HTML/CSS 文档站点
  +-- 不依赖 Vue，不参与前端构建
~~~

- `frontend/` 只负责页面、组件、交互和 API 请求。
- `src/` 只负责后端 API 和服务端逻辑；Spring Boot 不负责托管 Vue 页面。
- 本地开发使用 Vite proxy；独立部署时由 `VITE_API_BASE_URL` 指向 API 服务，并在后端配置对应 CORS 来源。
- `docs/` 维持现有静态页面用途，与 Vue SPA 分开发布。

## GitHub Actions CI

工作流文件：`.github/workflows/ci.yml`。

Push 到 main、向 main 创建/更新 Pull Request 或手动运行时，工作流分别执行：

1. **Backend**：配置 Java 21，运行 `mvn -B verify`，上传 Spring Boot JAR。
2. **Frontend**：配置 Node.js 20，在 `frontend/` 安装依赖并执行 `npm run build`，上传 `dist/` 构建产物。

可在仓库的 Actions 标签页查看两项任务的执行结果。

## GitHub Pages 静态文档站点

静态网站位于 `docs/` 目录，包含首页、项目架构、API 文档、服务状态、CI/CD、版本发布、快速开始和关于项目等页面。

本项目使用 **Deploy from a branch** 发布，不需要专门的 Pages GitHub Actions 工作流。仓库设置为：

- **Settings → Pages → Build and deployment → Source**：Deploy from a branch
- **Branch**：main
- **Folder**：/docs

每次向 main 推送 `docs/` 目录变更后，GitHub Pages 会自动发布。Vue 前端位于 `frontend/`，不改变此发布方式。

网站地址：https://mrguyang-art.github.io/springboot-github-demo/

## Docker 本地构建与运行（后端）

仓库根目录的 Dockerfile 继续只构建和运行 Spring Boot JAR：

~~~bash
mvn -B package
docker build -t springboot-github-demo:local .
docker run --rm -p 8080:8080 springboot-github-demo:local
~~~

访问 http://localhost:8080/api/hello 验证后端。

## GitHub Releases 与 GHCR Packages

发布工作流：`.github/workflows/release.yml`。推送符合版本格式的分支（例如 `v1.0.1`）时，工作流会校验版本、执行 Maven 测试、上传 JAR、创建 GitHub Release，并构建和推送后端 GHCR 镜像。当前 Docker/Release 流程面向 Spring Boot 后端；Vue 前端构建产物由 CI 单独验证和归档。

发布新版本前，建议确保改动已合并到 main，再从 main 创建版本分支：

~~~bash
git checkout main
git pull origin main
git checkout -b v1.0.1
git push -u origin v1.0.1
~~~

以 v1.0.0 为例：

- GitHub Release：https://github.com/mrguyang-art/springboot-github-demo/releases/tag/v1.0.0
- GHCR 镜像：`ghcr.io/mrguyang-art/springboot-github-demo:1.0.0`

## 项目结构

~~~text
.
├── .github/workflows/
│   ├── ci.yml                 # 后端与前端独立 CI
│   └── release.yml            # Spring Boot 后端 Release / GHCR
├── frontend/
│   ├── src/
│   │   ├── App.vue            # Vue 演示页面
│   │   ├── main.js            # Vue / Element Plus 入口
│   │   └── style.css          # 前端样式
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   └── README.md
├── docs/                      # GitHub Pages 静态文档，保持独立
├── src/main/java/com/demo/
│   ├── DemoApplication.java
│   ├── config/WebConfig.java  # API CORS 配置
│   └── controller/
│       └── HelloController.java
├── src/test/
├── Dockerfile
└── pom.xml
~~~
