# NVD 镜像

[English](README.md)

基于 GitHub Actions 的自动化工具，定期下载并镜像 [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) 所使用的 [NVD（国家漏洞数据库）](https://nvd.nist.gov/)。数据库打包为压缩归档文件并发布为 GitHub Release，方便所有用户直接下载使用，无需每次从头下载。

## 为什么需要这个项目？

OWASP Dependency-Check 在扫描依赖之前需要下载完整的 NVD 数据库。这个过程通常很慢，并且受 NVD API 速率限制。本项目通过提供预构建的、定期更新的数据库来解决这个问题。

## 工作原理

1. GitHub Actions 工作流在**每天 00:00 UTC** 自动运行（也支持手动触发）。
2. 使用 `dependency-check-maven` 插件下载/更新完整的 NVD 数据库。
3. 数据库在多次运行之间通过缓存实现增量更新。
4. 最终结果压缩为 `nvd-database.tar.gz`，以 `nvd-data-latest` 标签发布为 GitHub Release。

## 下载

从 [Releases 页面](https://github.com/diktamen/nvd-mirror/releases/tag/nvd-data-latest) 下载最新的数据库归档文件。

## 使用方法

1. 下载并解压归档文件：

   ```bash
   wget https://github.com/diktamen/nvd-mirror/releases/download/nvd-data-latest/nvd-database.tar.gz
   tar -xzf nvd-database.tar.gz
   ```

2. 将 `dependency-check-maven` 配置指向解压后的数据目录：

   **Maven 命令行：**
   ```bash
   mvn org.owasp:dependency-check-maven:check \
     -DdataDirectory=./dc-data \
   ```

   **pom.xml 配置：**
   ```xml
   <plugin>
     <groupId>org.owasp</groupId>
     <artifactId>dependency-check-maven</artifactId>
     <version>13.0.0</version>
     <configuration>
       <dataDirectory>/path/to/dc-data</dataDirectory>
     </configuration>
   </plugin>
   ```

## 技术栈

- **Java 17** (Temurin)
- **Maven** + OWASP Dependency-Check Maven Plugin 13.0.0
- **GitHub Actions**（定时 CI 流水线）
- **GitHub Releases**（制品分发）

## 配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 定时任务 | `0 0 * * *`（每天 00:00 UTC） | 每天增量更新；工作流会自动重新启用，避免 60 天无活动被禁用 |
| NVD API 延迟 | 500ms | 符合速率限制要求 |
| 数据目录 | `dc-data/` | NVD 数据库存储位置 |
| Release 标签 | `nvd-data-latest` | 始终指向最新构建 |
| 超时时间 | 720 分钟 | 最大工作流运行时长 |

### 所需 Secrets

- `NVD_API_KEY` — 你的 [NVD API 密钥](https://nvd.nist.gov/developers/request-an-api-key)，用于获得更高的 API 速率限制。
- `GITHUB_TOKEN` — GitHub Actions 自动提供。

## Fork 部署自己的镜像

1. Fork 本仓库。
2. 在仓库 Secrets 中添加你的 `NVD_API_KEY`（**Settings > Secrets and variables > Actions**）。
3. 为 Fork 的仓库启用 GitHub Actions。
4. 工作流将按计划自动运行，也可以在 Actions 页面手动触发。

## 许可证

本项目按原样提供，仅作便利之用。NVD 数据由 NIST 提供，受 [NVD 使用条款](https://nvd.nist.gov/faq) 约束。
