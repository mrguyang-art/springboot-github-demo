# Spring Boot GitHub Demo

一个使用 Java 21、Spring Boot 3.5 和 Maven 的入门项目，演示 GitHub Actions CI 流水线与 GitHub Pages 自动部署。

## 技术栈
- Java 21
- Spring Boot 3.5.5
- Maven
- JUnit 5 / Spring Boot Test
- GitHub Actions
- GitHub Pages

## 本地运行
执行 `mvn spring-boot:run` 启动项目。

接口：
- `GET http://localhost:8080/api/hello` — 欢迎消息
- `GET http://localhost:8080/actuator/health` — 健康检查

执行 `mvn -B verify` 编译并运行测试。

## GitHub Actions 流水线
工作流文件：`.github/workflows/ci.yml`。Push 到 `main` 或创建针对 `main` 的 Pull Request 后，会自动检出代码、配置 Temurin Java 21 和 Maven 缓存，并执行 `mvn -B verify`。在仓库的 Actions 标签页可以查看日志和结果。

## GitHub Pages
静态演示站点位于 `docs/index.html`，部署工作流为 `.github/workflows/pages.yml`。

首次启用请在仓库打开 Settings → Pages，在 Build and deployment → Source 中选择 GitHub Actions 并保存。之后每次 Push 到 `main`，Pages 工作流会自动发布 `docs/`。部署成功后访问 https://mrguyang-art.github.io/springboot-github-demo/ 。首次部署需要等待工作流完成。

## 项目结构
- `.github/workflows/ci.yml`：自动构建与测试
- `.github/workflows/pages.yml`：自动部署 Pages
- `docs/index.html`：展示网站
- `src/main/java/com/demo/`：Spring Boot 应用
- `src/test/java/`：自动化测试
- `pom.xml`：Maven 配置

## 推送验证
本次提交用于再次触发 `main` 分支上的 GitHub Actions 构建与 Pages 部署工作流。
