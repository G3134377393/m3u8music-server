# 音乐服务器（Music Server）使用说明

简洁的本地音乐目录读取与M3U8播放列表生成服务，提供现代化前端界面，支持歌曲预览、排序管理、M3U8下载、端口更改等功能。适用于 Windows、本地 Linux、OpenWrt/iStoreOS 等环境。

## 功能概览

- 静态页面（public）提供现代化管理界面
  - 主题切换（亮色/暗色/跟随浏览器）
  - 操作面板：预览歌曲、生成M3U8、刷新文件、服务器信息
  - “复制下载链接”一键复制当前服务的 M3U8 下载地址
- 歌曲预览
  - 从后端实时获取音乐文件列表（支持多个音频格式）
  - 支持拖拽调整排序、保存手动排序
  - 支持按文件名/修改时间/创建时间/诞生时间/文件大小等排序
- M3U8播放列表生成与下载
  - 基于当前访问主机与端口生成可用的播放列表链接
- 端口更改
  - 前端输入新的端口，后端校验并切换服务端口
  - 自动重定向到新端口（不再写死 localhost，按当前主机名与协议）
- 配置管理
  - 通过前端保存配置（musicFolder、sortBy、sortOrder 等）
  - 支持 OpenWrt/iStoreOS 一键安装脚本 install.sh
- 日志记录
  - 日志写入 /opt/music-server/logs/server.log（安装脚本会创建）

## 目录结构

```
server/
├── config/          # 配置加载与保存
├── logs/            # 本地运行日志（Windows）
├── public/          # 前端静态资源 (index.html, styles.css, script.js)
├── routes/          # API 路由 (routes/api.js)
├── utils/           # 后端工具模块 (staticHandler, fileManager, portManager 等)
├── install.sh       # 一键安装脚本 (OpenWrt/iStoreOS/其他Linux)
├── server.js        # 服务器入口
├── logger.js        # 日志模块 (被 server.js 引用)
├── song-order.json  # 可选，保存手动排序结果
└── music-server-config.json # 运行时配置（安装后位于 /opt/music-server）
```

## 快速开始（Windows 本地开发）

1. 安装 Node.js（建议 16+）
2. 在项目根目录运行
   - `node server.js`
3. 浏览器打开
   - `http://localhost:<端口>`（默认端口见配置或日志）
4. 使用前端界面进行：
   - 设置音乐文件夹路径
   - 预览歌曲，拖拽排序并保存
   - 生成或复制 M3U8 下载链接
   - 更改服务端口（会自动重定向到新端口）

## 安装部署（OpenWrt/iStoreOS 或其他 Linux）

- 使用脚本自动安装（需 root 权限）：
  1. 上传项目到目标机器
  2. 授权并执行
     - `chmod +x install.sh`
     - `sudo ./install.sh`
  3. 脚本会：
     - 检测系统类型并尝试安装 Node.js（OpenWrt 使用 opkg，Debian 使用 apt，RedHat 使用 yum；其他系统需要手动安装）
     - 创建服务目录 `/opt/music-server`
     - 拷贝必要文件与目录：`server.js`、`package.json`、`logger.js`、`public`、`routes`、`utils`、`config`、`song-order.json（若存在）`
     - 生成/复制配置文件 `/opt/music-server/music-server-config.json`
     - 创建日志目录与文件 `/opt/music-server/logs/server.log`
     - 创建系统服务（OpenWrt: /etc/init.d/music-server；其他Linux: systemd）
     - 尝试开放防火墙端口（OpenWrt 使用 UCI，其他系统需手动）
  4. 完成后脚本会显示访问地址与 M3U8 下载链接

- 服务管理
  - OpenWrt/iStoreOS:
    - 启动：`/etc/init.d/music-server start`
    - 停止：`/etc/init.d/music-server stop`
    - 重启：`/etc/init.d/music-server restart`
    - 状态：`/etc/init.d/music-server status`
  - systemd（其他Linux）：
    - 启动：`systemctl start music-server`
    - 停止：`systemctl stop music-server`
    - 重启：`systemctl restart music-server`
    - 状态：`systemctl status music-server`

## 配置说明（music-server-config.json）

安装脚本会在 `/opt/music-server/music-server-config.json` 生成或复制该文件，主要字段如下：

- port: 端口号（1024-65535），例如 5222
- musicFolder: 音乐目录路径（例如 `/mnt/sata1-1/alist/音乐`）
- sortBy: 默认排序字段（name、mtime、ctime、birthtime、size、api）
- sortOrder: 排序方向（asc、desc）
- includeSubfolders: 是否包含子目录（布尔）
- customDomain: 可选自定义域名（不用于强制重定向，仅信息展示）
- useSSL: 是否启用 SSL（布尔；如果使用反向代理提供 HTTPS，请保持此项为 false，由代理处理）
- corsOrigins: CORS 来源白名单（数组；默认 `["*"]`）
- trustProxy: 是否信任反向代理头部（布尔）
- version: 版本信息（字符串）

前端“保存配置”会更新并写入该文件（在本地运行时路径为项目根；安装后路径为 `/opt/music-server/music-server-config.json`）。

## 前端使用教程

- 主题切换：亮色/暗色/自动（跟随系统）
- 端口配置：
  - 输入新的端口并点击“更改端口”
  - 后端校验可用性，成功后自动重定向到新端口
  - 重定向基于当前访问协议与主机名，避免硬编码 localhost
- 音乐文件夹：
  - 填写音乐目录路径（后端会按该目录扫描音频文件）
  - 点击“保存配置”生效，同时清除已有的手动排序数据，以便新的排序生效
- 排序设置：
  - 选择排序方式和顺序，保存后立即生效（预览窗口打开时会自动刷新）
- 操作面板：
  - 预览歌曲：打开预览模态框，支持拖拽排序与保存
  - 生成M3U8：在新窗口打开 `/api/playlist.m3u8` 下载
  - 复制下载链接：将 `protocol//hostname:port/api/playlist.m3u8` 复制到剪贴板
  - 刷新文件：刷新页面资源
  - 服务器信息：显示当前端口、音乐文件夹、可访问地址列表、M3U8地址

## API 列表（主要）

- GET `/api/files`：返回音乐文件列表
- GET `/api/config`：获取配置与服务器信息
- POST `/api/config`：保存配置（musicFolder、sortBy、sortOrder）
- GET `/api/song-order`：获取已保存的手动排序
- POST `/api/save-song-order`：保存手动排序（传入歌曲路径数组）
- POST `/api/clear-song-order`：清除保存的手动排序
- POST `/api/change-port`：请求更改服务端口（返回 `redirectUrl`、`newPort` 等）
- GET `/api/playlist.m3u8`：生成并下载 M3U8 播放列表（基于请求 Host 构造流媒体 URL）

说明：
- `/api/playlist.m3u8` 内部基于 `req.headers.host` 构建流地址前缀，保证在不同 IP/端口访问时链接正确。
- `/api/change-port` 返回的重定向地址不再写死 localhost，改为使用请求头的 Host 与协议。

## 日志与排查

- 安装版日志文件：`/opt/music-server/logs/server.log`
  - 若日志为空：
    - 确认 install.sh 已拷贝 `logger.js`（server.js 依赖此文件）
    - 确认 install.sh 已创建 `logs` 目录与空的 `server.log`
    - 确认服务已启动（见服务管理命令）
- 本地开发日志：项目根目录下 `logs/`（如有）
- 遇到前端错误：
  - 打开浏览器控制台查看报错（例如 `Cannot read properties of undefined (reading 'renderSongs')`）
  - 强制刷新缓存（Ctrl+F5）
  - 检查网络面板，确认 `/api/files`、`/api/config` 正常返回
- 端口更改后跳到 localhost：
  - 已修复：后端 routes/api.js 与前端 script.js 不再写死 localhost
  - 如果通过反向代理，确保 `X-Forwarded-Host` 与 `X-Forwarded-Proto` 正确传递

## 防火墙与网络

- OpenWrt/iStoreOS：install.sh 会通过 UCI 自动添加端口放行规则
- 其他 Linux：请手动开放端口（例如使用 ufw 或 firewalld）
- 反向代理（如 Nginx/Caddy）：
  - 请传递并保留 `X-Forwarded-Proto` 与 `X-Forwarded-Host`
  - 后端会优先使用这些头来构造重定向与下载链接

## 常见问题（FAQ）

- Q: 更改端口后仍未跳转或“打不开页面”？
  - A: 可能是浏览器缓存或防火墙阻挡。请 Ctrl+F5 刷新、检查新端口是否开放、查看日志。确认没有硬编码 localhost（现版本已修复）。
- Q: 安装后访问无画面？
  - A: 检查是否完整拷贝 `public/`、`routes/`、`utils/`、`config/`、`logger.js`、`server.js`、`package.json`，并且 `/opt/music-server/logs/server.log` 存在。install.sh 已修复自动拷贝与日志目录创建。
- Q: 预览时提示 “获取歌曲列表失败: Cannot read properties of undefined (reading 'renderSongs')”？
  - A: 现版本已统一 SongManager.renderSongs 的定义并调用上下文。若仍报错，请清空浏览器缓存后重试，或提供控制台堆栈给我们定位。
- Q: M3U8 地址总是 localhost？
  - A: 现版本已改为选择首个非本地地址生成 m3u8Url，并在前端“复制下载链接”基于当前访问的主机与端口生成。

## 注意事项

- 请勿在仓库中存储敏感信息或密钥
- 通过反向代理提供 HTTPS 时，请确保代理层处理证书与 TLS
- 音乐目录访问需要系统权限，请确保服务用户具备读取权限
- 大量文件时，初次扫描可能耗时；建议将音乐目录组织良好
- 手动排序保存会生成 `song-order.json`，若不需要可通过“清除手动排序”删除

## 版本与维护

- 版本信息见 `package.json` 与前端“服务器信息”弹窗
- 如需新增功能（例如更多复制按钮、批量操作等），可在 `public/index.html` 与 `public/script.js` 扩展 UI 与逻辑
- 提交问题时请附带：
  - 浏览器控制台错误
  - `/opt/music-server/logs/server.log` 末尾几十行
  - 访问方式（直连/反代）、系统类型（OpenWrt/Debian/RedHat/Windows）

祝使用顺利 🎵
