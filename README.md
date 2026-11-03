# Spring Boot + Vue 前后端分离 DevOps Demo

本项目采用前后端分离结构：后端使用 Java 21 + Spring Boot 3.5.5，前端使用 Vue 3 + Element Plus。两端位于独立目录、独立构建；根目录 `docs/` 保持纯静态 HTML/CSS，由 GitHub Pages 单独发布。

## 项目组织结构

```text
.
├── backend/                  # Spring Boot 后端
│   ├── src/
│   ├── pom.xml
│   ├── Dockerfile
│   └── README.md
├── frontend/                 # Vue 3 + Element Plus 前端
│   ├── src/
│   ├── package.json
│   └── README.md
├── docs/                     # 独立静态文档站点，GitHub Pages 发布
└── .github/workflows/
    ├── backend-ci.yml        # 仅后端变更触发
    ├── frontend-ci.yml       # 仅前端变更触发
    └── release.yml           # 后端 Release / GHCR
```

## 本地运行

### 后端

要求 Java 21 和 Maven：

```bash
cd backend
mvn spring-boot:run
```

后端默认运行在 http://localhost:8080。

- `GET /api/hello`：欢迎消息。
- `GET /actuator/health`：健康状态。

### 前端

要求 Node.js 20+ 和 npm：

```bash
cd frontend
npm install
npm run dev
```

打开 http://localhost:5173。Vite 将 `/api` 和 `/actuator` 请求代理到后端 8080 端口。

生产构建：

```bash
npm run build
npm run preview
```

如需调用独立部署的后端，可设置 `VITE_API_BASE_URL`。后端跨域来源可通过 `APP_CORS_ALLOWED_ORIGINS` 环境变量配置，多个来源以逗号分隔。

## 前后端职责

- `frontend/`：Vue 页面、组件、交互与 API 请求。
- `backend/`：Spring Boot REST API、业务逻辑、测试和后端容器构建。
- `docs/`：独立静态文档，不纳入 Vue 构建或 Spring Boot 打包。

## 独立 CI 与路径过滤

| 工作流 | 触发路径 | 构建内容 |
|---|---|---|
| `backend-ci.yml` | `backend/**`、自身工作流文件 | Java 21 + Maven verify |
| `frontend-ci.yml` | `frontend/**`、自身工作流文件 | Node.js 20 + npm build |

Push 和 Pull Request 均使用相同路径过滤。仅修改后端代码时只运行后端 CI；仅修改前端代码时只运行前端 CI；只修改 `docs/` 或 README 不触发这两个构建工作流。两个工作流均支持手动运行。

## 后端 Docker 与版本发布

在仓库根目录执行：

```bash
cd backend
mvn -B package
docker build -t springboot-github-demo:local .
docker run --rm -p 8080:8080 springboot-github-demo:local
```

`.github/workflows/release.yml` 负责后端 Maven 验证、JAR Release 和 GHCR 镜像发布，使用 `backend/` 作为 Docker build context。

## GitHub Pages

`docs/` 继续从 main 分支的 `/docs` 目录通过 **Settings → Pages → Deploy from a branch** 发布，不依赖 Vue 或上述 CI 工作流。

网站：https://mrguyang-art.github.io/springboot-github-demo/
