# Vue 前端

独立的 Vue 3 + Element Plus 单页应用，通过 HTTP REST API 调用 Spring Boot 后端。

## 环境要求

- Node.js 20+
- npm 10+
- Spring Boot 后端运行在 `http://localhost:8080`

## 启动

在仓库根目录启动后端：

```bash
mvn spring-boot:run
```

另开终端启动前端：

```bash
cd frontend
npm install
npm run dev
```

打开 http://localhost:5173。Vite 将 `/api` 和 `/actuator` 请求代理到后端 8080 端口。

## 构建

```bash
npm run build
npm run preview
```

生产环境可通过 `VITE_API_BASE_URL` 指定后端 API 地址；未设置时使用当前站点同源路径。

## 项目边界

- `frontend/`：Vue UI、页面交互和前端构建。
- `src/`：Spring Boot REST API、服务端逻辑。
- `docs/`：独立的静态 HTML/CSS 文档站点，由 GitHub Pages 从 main 分支的 `/docs` 发布，不参与 Vue 构建。
