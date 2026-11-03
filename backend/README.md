# Spring Boot 后端

Java 21 + Spring Boot 3.5.5 REST API。

## 本地运行

```bash
mvn spring-boot:run
```

服务默认运行于 http://localhost:8080。

- `GET /api/hello`
- `GET /actuator/health`

## 验证与打包

```bash
mvn -B verify
mvn -B package
```

## Docker

在 `backend/` 目录执行：

```bash
docker build -t springboot-github-demo:local .
docker run --rm -p 8080:8080 springboot-github-demo:local
```

CORS 允许来源通过 `APP_CORS_ALLOWED_ORIGINS` 配置，多个来源用逗号分隔。
