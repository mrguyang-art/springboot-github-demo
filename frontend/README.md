# Vue 前端

Vue 3 + Element Plus + Vite，通过 HTTP REST API 调用 `backend/` 中的 Spring Boot 服务。

## 启动

先在另一个终端启动后端：

```bash
cd backend
mvn spring-boot:run
```

再启动前端：

```bash
cd frontend
npm install
npm run dev
```

打开 http://localhost:5173。Vite 将 `/api` 和 `/actuator` 请求代理到 localhost:8080。

## 构建

```bash
npm run build
npm run preview
```

如需连接独立部署的 API，可设置 `VITE_API_BASE_URL`。

## 项目边界

- `frontend/`：前端 UI、交互和构建。
- `backend/`：Spring Boot REST API 和后端逻辑。
- `docs/`：独立 HTML/CSS 静态文档，由 GitHub Pages 发布，不参与前端构建。
