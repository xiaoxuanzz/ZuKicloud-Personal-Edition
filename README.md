# MyCloud - 个人云存储系统

MyCloud 是一个轻量级、高安全性的自托管个人云存储系统，基于 PHP + MySQL 构建，提供文件管理、在线预览、安全分享等功能。

## 功能特性

### 文件管理
- 文件上传（支持分片上传，单文件最大 2GB）
- 文件下载（支持直接下载和分享链接下载）
- 多级文件夹管理（创建、重命名、移动、复制）
- 批量操作（批量删除、批量恢复、批量打包下载）
- 文件搜索（实时搜索文件名）
- 回收站（30 天自动清理）

### 在线预览
- 图片：JPG、PNG、GIF、WebP、SVG、BMP、HEIC
- 视频：MP4、WebM、AVI、MOV、MKV、FLV、WMV
- 音频：MP3、WAV、Flac、AAC、OGG、M4A
- 文档：PDF、Word（DOC/DOCX）、Excel（XLS/XLSX/CSV）、PPT（PPTX）
- 文本：TXT、Markdown、JSON、XML、YAML 等
- 代码：JS、TS、PHP、Python、Java、C/C++、Go、Rust 等 50+ 种语言高亮

### 分享功能
- 单文件分享（可设置提取码和有效期）
- 批量文件分享（一个链接包含多个文件）
- 分享链接预览和下载

### 用户系统
- 用户注册/登录
- 记住我（30 天免登录）
- 密码强度检测
- 忘记密码

## 安全机制

MyCloud 内置多层安全防护：

| 安全特性 | 说明 |
|---------|------|
| **CSRF 保护** | 所有表单提交均需验证 CSRF Token |
| **登录限流** | 5 次/分钟，超限封锁 5 分钟 |
| **下载限流** | 30 次/分钟/IP，防止恶意刷流量 |
| **路径遍历防护** | 下载时验证文件真实路径，防止目录穿越 |
| **危险文件拦截** | 禁止上传 PHP、ASP、JSP 等可执行脚本文件 |
| **源码加密传输** | HTTPS 环境下页面输出 AES-256-CBC 加密 |
| **反调试检测** | 检测开发者工具打开，自动锁定页面 |
| **爬虫防护** | 检测并拦截 bot/spider/crawler 等自动化访问 |
| **API 密钥验证** | AJAX 请求需携带页面渲染密钥 |
| **安全事件日志** | 记录所有安全事件，可在安全面板查看 |
| **XSS 防护** | 所有用户输入经过 `htmlspecialchars` 过滤 |
| **安全响应头** | 设置 `X-Content-Type-Options`、`Content-Security-Policy` 等 |

### 服务器安全配置
- **Apache**：禁止目录浏览、禁止访问隐藏文件和配置文件
- **Nginx**：禁止目录浏览、屏蔽隐藏文件、屏蔽 `includes/` 目录
- **IIS**：通过 `web.config` 实现相同的防护规则

## 部署要求

### 环境要求

| 组件 | 最低版本 | 推荐版本 |
|------|---------|---------|
| PHP | 7.4 | 8.0+ |
| MySQL | 5.7 | 8.0+ |
| Web 服务器 | Apache 2.4 / Nginx 1.18 / IIS 10 | 最新稳定版 |

### PHP 扩展

必须启用以下 PHP 扩展：
- `pdo_mysql` — 数据库连接
- `json` — JSON 处理
- `session` — 会话管理
- `openssl` — 页面加密传输
- `zip` — 批量打包下载

### 磁盘空间

- 系统本身：约 5MB
- 存储空间：取决于上传文件大小，建议预留充足空间

### 目录权限

确保以下目录对 Web 服务器可写：
- `upload/` — 文件存储目录
- `config.php` — 配置文件（安装时需要写入）
- `data/` — 运行时数据目录

## 部署方法

### 方式一：安装向导（推荐）

1. 将项目文件上传到 Web 服务器目录
2. 确保 `upload/` 和 `config.php` 所在目录可写
3. 访问 `http://你的域名/install.php`
4. 按向导完成环境检测、数据库配置、站点设置
5. 安装完成后访问 `login.php` 登录使用

### 方式二：手动配置

1. 修改 `config.php`，填入数据库连接信息：
   ```php
   define('DB_HOST', 'localhost');
   define('DB_NAME', 'mycloud');
   define('DB_USER', 'your_db_user');
   define('DB_PASS', 'your_db_password');
   define('SITE_URL', 'https://your-domain.com');
   ```

2. 创建数据库并导入表结构（参考 `install.php` 中的建表 SQL）

3. 创建锁定文件防止重复安装：
   ```bash
   touch install.lock
   ```

### Nginx 配置

将 `nginx.conf.example` 中的规则添加到你的 `server {}` 块中：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/mycloud;
    index index.php;

    # 禁用目录浏览
    autoindex off;

    # 隐藏文件防护
    location ~ /\. {
        deny all;
    }

    # upload 目录防护
    location ^~ /upload/ {
        location ~* ^/upload/[^\.]+$ {
            deny all;
        }
    }

    # includes 目录禁止外部访问
    location ^~ /includes/ {
        deny all;
    }

    # PHP 处理
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### Apache 配置

项目已自带 `.htaccess` 文件，启用 `mod_rewrite` 即可：

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### IIS 配置

项目已自带 `web.config`，无需额外配置。

## 目录结构

```
mycloud/
├── assets/              # 前端资源（CSS、JS、字体、第三方库）
├── data/                # 运行时数据
├── error/               # 错误页面（403/404/500）
├── includes/            # 核心组件
│   ├── auth.php         # 认证、限流、安全工具函数
│   ├── db.php           # 数据库连接
│   ├── header.php       # 页面头部
│   ├── footer.php       # 页面底部
│   ├── preview.php      # 文件预览引擎
│   └── safe_render.php  # 页面加密渲染
├── upload/              # 文件存储目录
├── config.php           # 配置文件（安装后生成）
├── install.php          # 安装向导
├── login.php            # 登录页
├── register.php         # 注册页
├── index.php            # 主页/文件管理
├── download.php         # 文件下载
├── share.php            # 文件分享页
├── batch_share.php      # 批量分享页
├── security.php         # 安全面板
└── nginx.conf.example   # Nginx 配置示例
```

## 定时任务

建议配置 cron 每 30 分钟自动清理未完成的分片上传残留文件：

```bash
# Linux / macOS
*/30 * * * * php /path/to/mycloud/cron_cleanup_chunks.php >> /path/to/mycloud/data/cron.log 2>&1
```

Windows 任务计划程序：
```
schtasks /create /tn "MyCloud Cleanup" /tr "php C:\path\to\mycloud\cron_cleanup_chunks.php" /sc minute /mo 30
```

也可以通过 Web 访问（每日动态 token 验证）：
```
http://your-domain/cron_cleanup_chunks.php?token=mycloud_cleanup_20260625
```

## 常见问题

### 上传文件失败
检查 PHP 配置中的 `upload_max_filesize` 和 `post_max_size`，建议设为：
```ini
upload_max_filesize = 2G
post_max_size = 2G
max_execution_time = 300
```

### 页面加密不生效
页面加密（AES-256-CBC）仅在 HTTPS 或 localhost 下启用。请配置 SSL 证书。

### 数据库连接失败
确认 `config.php` 中的数据库信息正确，且 MySQL 服务正在运行。

## 版本

当前版本：1.0.0

## 开源协议

本项目采用 [MIT 许可证](LICENSE) 开源。
联系作者：https://space.bilibili.com/521205099
如果觉得对你有帮助的，可以支持一下up哦，感激不尽
