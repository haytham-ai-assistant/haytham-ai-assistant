---
name: github-mirror-access
description: 在无法直接访问 GitHub 时，提供可靠的镜像站和代理方案，帮助用户通过镜像站克隆仓库、下载文件、加速访问。包括镜像站列表、配置方法、使用示例和故障排查。
license: MIT
metadata:
  audience: developers
  environment: server-container
  category: network
---

## 何时使用

- 服务器容器环境无法连接 GitHub
- 国内网络访问 GitHub 速度慢或不稳定
- 需要克隆仓库、下载 Release 文件、浏览代码
- 需要配置 Git 或 CI/CD 使用镜像源

## 可用镜像站列表（2026 年 2 月测试）

| 镜像站 | 类型 | 使用方式 | 可用性 | 备注 |
|--------|------|----------|--------|------|
| **gitclone.com** | 国内缓存镜像 | `https://gitclone.com/github.com/用户/仓库` | ✅ 可用 | 支持 git clone、文件下载，首次访问需缓存 |
| **bgithub.xyz** | 域名替换 | 将 `github.com` 替换为 `bgithub.xyz` | ✅ 可用 | 直接替换，支持网页浏览和 git |
| **github.ur1.fun** | 域名替换 | 将 `github.com` 替换为 `github.ur1.fun` | ✅ 可用 | 直接替换 |
| **ghfast.top** | 代理前缀 | `https://ghfast.top/https://github.com/...` | ✅ 可用 | 支持文件下载和 git，多国节点 |
| **kkgithub.com** | 域名替换 | 将 `github.com` 替换为 `kkgithub.com` | ⚠️ 可能失效 | 历史上可用，测试中不稳定 |
| **hub.njuu.cf** / **hub.yzuu.cf** | 域名替换 | 将 `github.com` 替换为 `hub.njuu.cf` 等 | ❌ 可能失效 | 曾经流行，目前测试不可用 |
| **kgithub.com** | 域名替换 | 将 `github.com` 替换为 `kgithub.com` | ❌ 可能失效 | 测试不可用 |
| **ghproxy.com** / **mirror.ghproxy.com** | 代理前缀 | `https://ghproxy.com/https://github.com/...` | ⚠️ 不稳定 | 有时可用，有时连接失败 |
| **github.com.cnpmjs.org** | 域名替换 | 将 `github.com` 替换为 `github.com.cnpmjs.org` | ⚠️ 较慢 | 可用但速度可能较慢 |

## 配置方法

### 1. Git 全局配置镜像

```bash
# 使用 gitclone 镜像
git config --global url."https://gitclone.com/github.com/".insteadOf "https://github.com/"

# 使用 bgithub.xyz 镜像
git config --global url."https://bgithub.xyz/".insteadOf "https://github.com/"

# 恢复默认
git config --global --unset url."https://github.com/".insteadOf
```

### 2. 单次克隆使用镜像

```bash
# 直接替换域名
git clone https://bgithub.xyz/用户/仓库.git

# 使用代理前缀
git clone https://ghfast.top/https://github.com/用户/仓库.git
```

### 3. 下载 Release 文件

```bash
# 原始 URL
https://github.com/用户/仓库/releases/download/v1.0/package.zip

# 使用 ghfast.top 代理
https://ghfast.top/https://github.com/用户/仓库/releases/download/v1.0/package.zip

# 使用 gitclone（需先缓存）
https://gitclone.com/github.com/用户/仓库/releases/download/v1.0/package.zip
```

### 4. 环境变量配置

```bash
# 设置 Git 代理（如果需要 HTTP 代理）
export http_proxy=http://proxy_ip:port
export https_proxy=http://proxy_ip:port

# 设置 GitHub 镜像源环境变量（某些工具支持）
export GITHUB_PROXY=https://ghfast.top
```

### 5. Docker 容器内配置

```dockerfile
# Dockerfile 示例
RUN git config --global url."https://gitclone.com/github.com/".insteadOf "https://github.com/"
```

```yaml
# docker-compose.yml 示例
services:
  app:
    environment:
      - GITHUB_PROXY=https://ghfast.top
```

## 故障排查

### 测试镜像站可用性

```bash
# 测试 HTTP 访问
curl -I https://gitclone.com
curl -I https://bgithub.xyz

# 测试 Git 协议
timeout 5 git ls-remote https://gitclone.com/github.com/git/git.git

# 测试文件下载
curl -L https://ghfast.top/https://github.com/用户/仓库/archive/main.zip -o test.zip
```

### 常见问题

1. **镜像站不可用**：尝试其他镜像站，优先使用 ✅ 标记的。
2. **Git 克隆失败**：检查 Git 版本，确保支持 HTTPS。可尝试降低协议版本：`git config --global protocol.version 2`。
3. **下载速度慢**：不同镜像站在不同网络环境下表现不同，多试几个。
4. **证书错误**：某些镜像站可能使用自签名证书，可临时禁用验证：`git config --global http.sslVerify false`（不推荐）。

## 注意事项

- 镜像站非官方提供，存在失效风险，重要项目建议备份。
- 使用镜像站时，注意隐私和安全，避免传输敏感信息。
- 部分镜像站可能有流量限制或带宽限制，适合个人或小规模使用。
- 定期更新镜像站列表，关注社区反馈。

## 参考资源

- [GitHub 镜像站列表（持续更新）](https://blog.csdn.net/eytha/article/details/144797222)
- [国内访问 Github 的四种方法（2025 版）](https://blog.csdn.net/xinxiyinhe/article/details/145856498)
- [GitHub 加速镜像目前可用](https://qjfyx.com/githubacce/)
- [GitHub 文件加速集群](https://github.com/xiaoxuan6/github-mirror)