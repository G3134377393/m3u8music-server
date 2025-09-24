# m3u8music-server

音乐服务器 - iStoreOS
一个专为iStoreOS设计的音乐服务器，能够扫描指定文件夹中的音乐文件并生成M3U8播放列表。

功能特性
🎵 支持多种音频格式：MP3、FLAC、WAV、M4A、AAC、OGG、WMA
📱 生成标准M3U8播放列表文件
🌐 Web界面管理和下载
📥 支持单个文件下载
⚙️ 可配置音乐文件夹路径
🔄 实时文件扫描
安装和运行
1. 安装Node.js (在iStoreOS上)
# 更新软件包列表
opkg update
安装Node.js
opkg install node npm

2. 安装项目依赖
# 进入项目目录
cd /path/to/music-server
安装依赖
npm install

3. 配置音乐文件夹
编辑 server.js 文件，修改 config.musicFolder 为你的音乐文件夹路径：

const config = {
    musicFolder: '/mnt/sda1/music', // 修改为你的音乐文件夹路径
    // ...
};
4. 启动服务
# 启动服务器
npm start
或者直接运行
node server.js

使用方法
Web界面访问
打开浏览器访问：http://你的设备IP:8897

API接口
获取文件列表: GET /api/files
下载M3U8播放列表: GET /api/playlist.m3u8
获取配置: GET /api/config
更新文件夹路径: POST /api/config/folder
下载单个文件: GET /download/:filename
M3U8播放列表下载
直接访问：http://你的设备IP:8897/api/playlist.m3u8

配置说明
默认配置
端口: 8897
音乐文件夹: /mnt/sda1/music
支持格式: .mp3, .flac, .wav, .m4a, .aac, .ogg, .wma
修改配置
通过Web界面: 访问主页，点击"配置"按钮
通过API: 发送POST请求到 /api/config/folder
直接修改代码: 编辑 server.js 中的 config 对象
在iStoreOS上设置开机自启
1. 创建服务脚本
# 创建服务脚本
cat > /etc/init.d/music-server << 'EOF'
#!/bin/sh /etc/rc.common
START=99
STOP=10
USE_PROCD=1
PROG="/usr/bin/node"
ARGS="/path/to/music-server/server.js"
start_service() {
procd_open_instance
procd_set_param command PROGPROG PROGARGS
procd_set_param respawn
procd_set_param stdout 1
procd_set_param stderr 1
procd_close_instance
}
EOF
设置执行权限
chmod +x /etc/init.d/music-server
启用服务
/etc/init.d/music-server enable
启动服务
/etc/init.d/music-server start

2. 检查服务状态
# 查看服务状态
/etc/init.d/music-server status
查看日志
logread | grep music-server

常见问题
Q: 音乐文件夹不存在怎么办？
A: 通过Web界面或API更新正确的文件夹路径，或者创建对应的文件夹。

Q: 无法访问8897端口？
A: 检查防火墙设置，确保8897端口已开放。

Q: M3U8文件无法播放？
A: 确保播放器支持网络流媒体，并且设备网络连接正常。

Q: 如何添加更多音频格式支持？
A: 编辑 server.js 中的 config.supportedFormats 数组，添加新的文件扩展名。

技术栈
后端: Node.js + Express
前端: 原生HTML/CSS/JavaScript
文件系统: Node.js fs模块
跨域支持: CORS
