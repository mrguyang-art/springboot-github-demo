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

## GitHub Releases 与 GHCR Packages

发布流水线位于 `.github/workflows/release.yml`，Docker 构建文件为仓库根目录的 `Dockerfile`。

### 发布一个版本

1. 确保代码已合并到 `main`，且 CI 测试通过。
2. 在本地创建并推送语义化版本标签，例如：

   ```bash
   git checkout main
   git pull
   git tag v1.0.0
   git push origin v1.0.0
   ```

3. 打开仓库的 Actions 页面，查看 **Release and publish** 工作流。
4. 工作流成功后，在 Releases 页面可下载构建好的 JAR；GHCR 中会有对应的容器镜像。

### 发布产物

- GitHub Release：自动生成 Release Notes，并附加 Spring Boot JAR。
- GitHub Actions Artifact：保存名为 `springboot-jar-1.0.0` 的构建产物，保留 30 天。
- GHCR：`ghcr.io/mrguyang-art/springboot-github-demo:1.0.0`、`:1.0` 和 `:latest`。

拉取并运行镜像：

```bash
docker pull ghcr.io/mrguyang-art/springboot-github-demo:1.0.0
docker run --rm -p 8080:8080 ghcr.io/mrguyang-art/springboot-github-demo:1.0.0
```

访问 `http://localhost:8080/api/hello` 验证服务。

### GHCR 可见性

首次推送后，镜像可能默认为 Private。若希望公开拉取，请进入 GitHub 个人主页 → Packages → 对应容器包 → Package settings，将 Visibility 设置为 Public。公开后无需登录即可拉取；私有包需要先执行 `docker login ghcr.io`。

### 手动触发

也可以在 Actions → **Release and publish** → Run workflow 中输入已存在的版本标签（例如 `v1.0.0`），重新执行该版本的发布流程。
