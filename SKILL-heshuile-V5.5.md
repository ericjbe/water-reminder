---
name: heshuile-wisecho-v5.5
version: V5.5
date: 2026-02-28
description: 喝水了吗·智慧时光 开发维护总规范 V5.5。FND水色系配色（#0085D0/#EB6100/#0B1120），语音AI+文字输入，关怀提醒引导，广告乱码修复。包含公司内网4090和阿里云两套完整部署方案。
---

# 喝水了吗 · 智慧时光  开发维护 Skill V5.5

> ⚠️ **本文件是唯一权威SKILL版本**。GitHub上只保留最新版本，旧版本全部删除。
> 每次修改代码，SKILL版本号和index.html版本号必须同步升级。

---

## 零、强制更新纪律（每次改代码必须执行）

```
□ 1. node --check 验证JS语法通过
□ 2. 在 index.html <head> 版本注释块顶部新增 [NEW] Vx.x 行
□ 3. 更新本文件 name / version / date / description 字段
□ 4. 在版本历史表格新增一行（不修改旧行）
□ 5. 同时输出 index.html 和 SKILL-heshuile-Vx.x.md 两个文件
□ 6. 告知用户：「请将这两个文件上传到GitHub，删除旧SKILL文件」
```

**版本号命名规则**

| 情况 | 版本号 |
|------|--------|
| 新增功能模块 | V5.5 → V5.6 |
| 小修复/文字调整 | V5.5 → V5.5.1 |
| 重大架构重构 | V5.x → V6.0 |

---

## 一、项目概述

| 项目 | 内容 |
|------|------|
| 应用名称 | 喝水了吗 · 智慧时光 |
| 线上地址 | https://ericjbe.github.io/water-reminder/ |
| 技术形式 | 单文件 PWA（全部代码在一个 index.html） |
| 数据存储 | localStorage（无后端，无服务器） |
| AI后端 | 本地知识库 + Claude API fallback |
| 联系人 | 水若善 / 盛兴崑 |
| 手机 | 18680666987 |
| 微信 | FCLFUND |
| 邮箱 | artstoneusa@gmail.com |

---

## 二、FND 水色系配色规范（V5.4 确立，不可更改）

### 2.1 CSS 变量（完整）

```css
:root {
  --primary:        #0085D0;   /* FND蓝  · 主按钮·链接·导航激活 */
  --accent:         #EB6100;   /* FND橙  · CTA·强调·警示 */
  --gold:           #F59E0B;   /* 暖金色 · 积分·勋章·等级 */
  --success:        #06ffa5;   /* 成功绿 */
  --bg1:            #0B1120;   /* 深海军蓝 · 全局背景 */
  --bg2:            #1E293B;   /* 深石板蓝 · 卡片/面板 */
  --water:          #38BDF8;   /* 水色高亮 */
  --water-deep:     #005A8D;   /* 深水色 · Hover */
  --water-light:    #E6F6FD;   /* 浅水色 · 装饰 */
  --water-gradient: linear-gradient(135deg, #0085D0 0%, #005A8D 100%);
}
```

### 2.2 禁止出现的旧颜色

| 旧颜色 | 必须替换为 |
|--------|-----------|
| #00d4ff | #0085D0 |
| #ff006e | #EB6100 |
| #8338ec | #005A8D |
| #0a192f | #0B1120 |
| #112240 / #1a1a2e / #16213e | #1E293B |

### 2.3 FND 水色系标准色值参考

```
FND Blue:      #0085D0   主品牌蓝（信任·科技）
FND Orange:    #EB6100   行动橙（CTA·强调）
Water Deep:    #005A8D   深水色（Hover）
Water Glow:    #38BDF8   发光高亮
Dark BG:       #0B1120   深海军蓝背景
Dark Surface:  #1E293B   石板灰卡片
Orange Hover:  #C44F00   橙色悬停
Water Light:   #E6F6FD   浅水蓝装饰
```

---

## 三、架构规则

### 3.1 单文件约束

所有 HTML / CSS / JS 在一个 `index.html`，修改后必须做语法验证：

```bash
# 提取主script块验证
python3 -c "
import re
c = open('index.html', encoding='utf-8').read()
s = max(re.findall(r'<script[^>]*>([\s\S]*?)</script>', c), key=len)
open('/tmp/chk.js','w',encoding='utf-8').write(s)
"
node --check /tmp/chk.js
```

### 3.2 常见Bug预防

- ❌ onclick 内联事件中不能嵌套同种引号
- ✅ 用 `&#39;` 替代内嵌单引号，或提取为独立函数
- ❌ JS字符串中不能有字面换行（用 `\n` 代替）
- ✅ DOM操作前先判空：`var el = document.getElementById('x'); if(el) el.xxx`

### 3.3 localStorage 键名

| 键名 | 内容 |
|------|------|
| heshuileData | 主数据（含广告、提醒、喝水记录）|
| heshuileUsers | 用户表 |
| heshuileMembers | 会员激活状态 |
| heshuilePendingOrders | 待审核订单 |
| heshuileListenProgress | 听书进度 |

---

## 四、子系统功能清单（当前 V5.5 状态）

| 功能 | 状态 | 说明 |
|------|------|------|
| 滴答时钟 | ✅ 完成 | 模拟+数字双表盘，嘀嗒音效 |
| 喝水提醒 | ✅ 完成 | 每小时自动提醒，进度条，一键记录 |
| 关怀提醒 | ✅ 完成 | 生日/纪念日提醒，系统通知推送 |
| 会员系统 | ✅ 完成 | 免费/VIP/企业三档，激活码，3天试用 |
| 支付流程 | ✅ 完成 | 微信/支付宝/USDT，管理员审核 |
| 通讯录裂变 | ✅ 完成 | 生日场景授权，推荐积分，二维码 |
| 水学院 | ✅ 完成 | 课程/经典/苏格拉底问答 |
| 语音助手 | ✅ V5.5完成 | 文字输入+语音识别，本地KB+Claude API |
| 水健康日报 | ✅ 完成 | 每日水健康分析，邮件发送 |
| 广告系统 | ✅ V5.5修复 | 乱码缓存强制清除，v5.4版本号检测 |
| FND配色 | ✅ V5.4完成 | 全站64处颜色已替换 |
| Service Worker | ✅ V5.5完成 | 后台推送支持（需浏览器支持）|

---

## 五、语音助手功能说明（V5.5）

### 5.1 工作流程

```
用户输入（文字或语音）
    ↓
1. 本地水学知识库匹配（18条，即时响应）
    ↓ 无匹配
2. 指令识别（几点了/喝水进度/打开面板等）
    ↓ 无匹配
3. Claude API 智能回答（100字以内，水老人格）
    ↓ API失败
4. 兜底提示（引导说水学话题）
```

### 5.2 语音唤醒

唤醒词：**「智回」**（也识别「知回」「志回」「之回」）
唤醒后15秒内说指令，超时自动休眠。

### 5.3 Claude API 接入（已写入代码）

```javascript
// 语音面板已接入，system prompt：
// "你是'水老'，水科学专家。只用中文，100字以内，聚焦水学话题"
// model: claude-sonnet-4-20250514
```

---

## 六、关怀提醒功能说明（V5.5）

### 6.1 工作原理（不需要阿里云）

| 场景 | 效果 | 要求 |
|------|------|------|
| 浏览器开着 | 弹窗+语音播报 | 无需授权 |
| 手机锁屏 | 系统通知推送到通知栏 | 点击「开启系统通知」授权 |
| 浏览器关闭 | 后台Service Worker推送 | 安装到桌面（PWA）|

### 6.2 用户操作路径

```
底部导航「💝关怀」→ 面板顶部「🔔 开启系统通知推送」
→ 弹出系统授权弹窗 → 允许
→ 点「🧪 测试发一条提醒」确认可收到
→ 设置联系人提醒 → 到点自动推送
```

### 6.3 整点喝水提醒规则

每天 7:00 至 21:00 整点（共15次），当日饮水量 < 80% 时自动推送系统通知。

---

## 七、水老AI服务器（公司4090·局域网）

### 7.1 架构

```
手机/浏览器（同一WiFi）
    ↓↑ HTTP / WebSocket
wisecho_server.py（FastAPI，端口8000）
    ↓↑
tts_helper.py → PowerShell → System.Speech（慧慧中文）→ WAV
    ↓↑
Ollama qwen2.5:7b（本地推理）
    ↓↑
wisecho_db（30408块向量，docs.json + vecs.bin）
```

### 7.2 关键文件路径

| 文件 | 路径 |
|------|------|
| wisecho_server.py | C:\Users\sinos\heshuile\Data\waterontology\ |
| tts_helper.py | 同上 |
| wisecho_db\ | C:\Users\sinos\heshuile\Data\wisecho_db\ |

### 7.3 接口清单

| 接口 | 方法 | 说明 |
|------|------|------|
| / | GET | 状态检查，返回JSON |
| /tts?text=xxx | GET | 单条TTS，返回WAV |
| /ws/ask | WebSocket | 文字流问答 |
| /ws/audio | WebSocket | 音频流（智回听书）|

### 7.4 iOS Safari播放必要条件

```python
# server返回必须带这两个头，否则iOS无法播放
return Response(audio_bytes, media_type="audio/wav", headers={
    "Accept-Ranges": "bytes",
    "Content-Length": str(len(audio_bytes))
})
```

---

## 八、🏢 公司内网 4090 完整部署方案

> 目标：让公司内网所有人（手机/电脑）通过 http://192.168.3.22:8000 访问水老AI服务

### Step 1：固定电脑IP地址

每次重启路由器，电脑IP可能变化，必须设置固定IP。

```
操作路径：
控制面板 → 网络和共享中心 → 以太网（或WiFi）→ 属性
→ Internet 协议版本4（TCP/IPv4）→ 属性
→ 选择「使用下面的IP地址」
```

填写：
```
IP 地址：    192.168.3.22
子网掩码：   255.255.255.0
默认网关：   192.168.3.1（你的路由器地址，不确定就看路由器背面）
首选DNS：    114.114.114.114
备用DNS：    8.8.8.8
```

点确定后，在PowerShell里确认：
```powershell
ipconfig | findstr "IPv4"
# 应该显示：IPv4 地址 . . . . . . . : 192.168.3.22
```

### Step 2：Windows防火墙放行 8000 端口

```powershell
# 以管理员身份运行PowerShell，执行以下命令：

# 检查规则是否已存在
netsh advfirewall firewall show rule name="Wisecho8000"

# 如果不存在，添加规则
netsh advfirewall firewall add rule `
  name="Wisecho8000" `
  dir=in `
  action=allow `
  protocol=TCP `
  localport=8000 `
  profile=any

# 确认添加成功
netsh advfirewall firewall show rule name="Wisecho8000"
```

### Step 3：确认Python环境和依赖

```powershell
# 检查Python版本（需要3.8+）
python --version

# 检查依赖是否已安装
pip list | findstr -i "fastapi uvicorn websockets numpy"

# 如有缺失，安装依赖
pip install fastapi uvicorn websockets numpy python-docx tqdm
```

### Step 4：确认Ollama和模型

```powershell
# 检查Ollama是否运行
ollama list

# 如果没有qwen2.5:7b，拉取模型
ollama pull qwen2.5:7b

# 确认ollama服务在运行
curl http://127.0.0.1:11434/api/tags
```

### Step 5：启动水老AI服务器

```powershell
# 进入项目目录
cd "C:\Users\sinos\heshuile\Data\waterontology"

# 启动服务
python wisecho_server.py

# 正常启动后会显示：
# INFO:     Started server process [xxxx]
# INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Step 6：验证服务正常

```powershell
# 新开一个PowerShell窗口测试（不要关闭服务窗口）

# 1. 状态检查
curl http://192.168.3.22:8000/

# 2. TTS测试（生成WAV文件）
curl "http://192.168.3.22:8000/tts?text=上善若水" -o test.wav
# 然后双击 test.wav 确认有中文语音

# 3. 手机测试（手机连公司WiFi）
# 手机浏览器打开：http://192.168.3.22:8000/tts?text=喝水了吗
# iOS Safari应该直接播放语音
```

### Step 7（可选）：开机自启动

方法A：创建批处理文件放入启动文件夹
```powershell
# 在PowerShell里一步步执行：

# 第一步：创建批处理文件内容
$content = "@echo off`r`ncd C:\Users\sinos\heshuile\Data\waterontology`r`nstart /min python wisecho_server.py"

# 第二步：写入批处理文件
$startupPath = "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\wisecho_start.bat"
Set-Content -Path $startupPath -Value $content -Encoding ASCII

# 第三步：确认文件已创建
Get-Item $startupPath
```

方法B：任务计划程序（更稳定）
```
开始菜单 → 任务计划程序 → 创建基本任务
→ 名称：WisechoServer
→ 触发器：计算机启动时
→ 操作：启动程序
→ 程序：python
→ 参数：wisecho_server.py
→ 起始于：C:\Users\sinos\heshuile\Data\waterontology
→ 勾选「最高权限运行」
```

### Step 8：在 index.html 里配置内网地址

确认 index.html 里的连接地址是内网地址：
```javascript
const WISECHO_URL = 'http://192.168.3.22:8000';
const WISECHO_WS  = 'ws://192.168.3.22:8000/ws/ask';
```

### 内网部署验证清单

```
□ ipconfig 确认 IP 是 192.168.3.22
□ 防火墙规则 Wisecho8000 存在
□ ollama list 能看到 qwen2.5:7b
□ python wisecho_server.py 启动无报错
□ PC浏览器 http://192.168.3.22:8000 → 返回JSON状态
□ PC浏览器 http://192.168.3.22:8000/tts?text=测试 → 能播放声音
□ 手机（同WiFi）访问 → 能播放声音
□ index.html 语音问答面板连接成功
□ 重启电脑后IP不变，服务自动启动（如配置了自启）
```

### 常见内网故障

| 症状 | 原因 | 解决 |
|------|------|------|
| 手机访问超时 | 防火墙未放行 | Step 2 重新添加规则 |
| 手机访问超时 | 手机和电脑不同WiFi | 确认同一网络 |
| 服务启动失败 | 端口已占用 | `netstat -ano | findstr :8000`，结束占用进程 |
| TTS无声音 | System.Speech未初始化 | 确认tts_helper.py路径正确 |
| iOS播放不了 | 缺Accept-Ranges | 检查server返回头 |
| Ollama无响应 | 模型未拉取 | `ollama pull qwen2.5:7b` |

---

## 九、☁️ 阿里云公网完整部署方案

> 目标：让全球任何手机都能访问 https://wisecho.cn（或你的域名），7×24小时在线

### 9.1 推荐配置与费用

| 资源 | 规格 | 月费估算 |
|------|------|---------|
| ECS云服务器 | 2核4G，Ubuntu 22.04 LTS | 60~120元（新用户首购更便宜）|
| 公网带宽 | 固定5Mbps | 20~40元 |
| 域名 | wisecho.cn（推荐）| 约5元/月（年付60元）|
| SSL证书 | 阿里云免费DV | 0元 |
| 通义千问API | qwen-plus（替代本地Ollama）| 约15~30元 |
| 通义TTS | CosyVoice（替代System.Speech）| 约3~8元 |
| **合计** | | **约103~198元/月** |

> 💡 相比本地4090（电费约200元+不稳定+下班断网），阿里云更划算且24/7可用。

### 9.2 第一阶段：购买并登录ECS

1. 登录 https://ecs.console.aliyun.com
2. 选择**按量付费**（先试用，稳定后换包年）
3. 地域选**华南1广州**或**华东2上海**（国内速度快）
4. 系统镜像：**Ubuntu 22.04 LTS 64位**
5. 带宽：固定带宽5Mbps
6. 安全组：**开放80、443、8000端口**（重要！）

```bash
# 购买后在控制台「重置实例密码」，然后SSH登录：
ssh root@你的公网IP地址

# 第一次登录后更新系统（必做）
apt update && apt upgrade -y
```

### 9.3 第二阶段：ECS系统初始化

```bash
# 1. 安装必要软件
apt install python3 python3-pip python3-venv git nginx curl -y

# 2. 创建项目目录
mkdir -p /opt/wisecho
cd /opt/wisecho

# 3. 创建Python虚拟环境
python3 -m venv venv
source venv/bin/activate

# 4. 安装Python依赖（注意：阿里云Linux，不用pyttsx3/System.Speech）
pip install fastapi uvicorn websockets numpy dashscope python-docx tqdm

# 5. 确认安装成功
pip list | grep -E "fastapi|uvicorn|dashscope"
```

### 9.4 第三阶段：上传向量库（30408块）

**Windows端打包（PowerShell，一行一行执行）：**

```powershell
# 第一步：打包
Compress-Archive -Path "C:\Users\sinos\heshuile\Data\wisecho_db" -DestinationPath "C:\Users\sinos\Desktop\wisecho_db.zip"

# 第二步：确认文件大小（应该几百MB）
Get-Item "C:\Users\sinos\Desktop\wisecho_db.zip" | Select-Object Length
```

**上传方式（推荐：OSS中转）：**

```bash
# 方式A：阿里云OSS中转（推荐，速度快）
# 1. 在阿里云控制台创建OSS Bucket（选同地域）
# 2. 在Windows阿里云控制台上传 wisecho_db.zip 到Bucket
# 3. 在ECS上下载：
cd /opt/wisecho
wget "https://你的bucket名.oss-cn-xxx.aliyuncs.com/wisecho_db.zip"
unzip wisecho_db.zip
ls wisecho_db/  # 确认文件存在

# 方式B：SCP直传（Windows需安装OpenSSH）
# 在Windows PowerShell执行：
scp "C:\Users\sinos\Desktop\wisecho_db.zip" root@你的公网IP:/opt/wisecho/
# 然后在ECS上解压：
# ssh root@公网IP "cd /opt/wisecho && unzip wisecho_db.zip"
```

### 9.5 第四阶段：修改server使用通义千问API

本地用Ollama，阿里云用**通义千问API**（qwen-plus），需修改server里的两个函数：

**申请API Key：**
登录 https://dashscope.console.aliyun.com → API-KEY管理 → 创建API Key

**修改wisecho_server.py的关键函数：**

```python
import os
DASHSCOPE_API_KEY = os.environ.get('DASHSCOPE_API_KEY', '')

# ── 替换 embed 函数（原来调Ollama）──
def get_embed(text: str) -> list:
    from dashscope import TextEmbedding
    resp = TextEmbedding.call(
        model='text-embedding-v3',
        input=text,
        api_key=DASHSCOPE_API_KEY
    )
    return resp.output['embeddings'][0]['embedding']

# ── 替换 generate 流式输出（原来调Ollama）──
async def generate_stream(prompt: str):
    from dashscope import Generation
    responses = Generation.call(
        model='qwen-plus',
        prompt=prompt,
        stream=True,
        api_key=DASHSCOPE_API_KEY
    )
    for resp in responses:
        if resp.output and resp.output.text:
            yield resp.output.text
```

### 9.6 第五阶段：TTS替换（Linux版，替代System.Speech）

Linux没有Windows的System.Speech，改用**阿里云CosyVoice TTS**：

```python
# pip install dashscope（已安装）

def tts_to_wav_cloud(text: str) -> bytes:
    from dashscope.audio.tts_v2 import SpeechSynthesizer
    synthesizer = SpeechSynthesizer(
        model='cosyvoice-v1',
        voice='longxiaochun',      # 成熟稳重男声，接近"水老"人格
        # 其他可选：longxiaoxia（女声）、longlaoshi（老师音）
        api_key=DASHSCOPE_API_KEY
    )
    audio_bytes = synthesizer.call(text)
    return audio_bytes

# 在 /tts 接口里调用
@app.get("/tts")
async def tts_endpoint(text: str):
    audio = tts_to_wav_cloud(text)
    return Response(audio, media_type="audio/wav", headers={
        "Accept-Ranges": "bytes",
        "Content-Length": str(len(audio))
    })
```

费用：约0.001元/百字，日均500次调用约0.5元，月均约15元。

### 9.7 第六阶段：Nginx反向代理配置

```bash
# 创建Nginx配置文件
cat > /etc/nginx/sites-available/wisecho << 'NGINXEOF'
# HTTP → 强制跳转HTTPS
server {
    listen 80;
    server_name 你的域名.com;
    return 301 https://$host$request_uri;
}

# HTTPS主配置
server {
    listen 443 ssl http2;
    server_name 你的域名.com;

    ssl_certificate     /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    # SSL安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # ─── WebSocket（必须有这三行，否则WS连接失败）───
    location /ws/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }

    # ─── 普通HTTP接口（TTS等）───
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        # 音频文件允许Range请求（iOS播放必须）
        proxy_set_header Range $http_range;
        proxy_set_header If-Range $http_if_range;
    }
}
NGINXEOF

# 启用配置
ln -s /etc/nginx/sites-available/wisecho /etc/nginx/sites-enabled/

# 删除默认配置（避免冲突）
rm -f /etc/nginx/sites-enabled/default

# 测试配置语法
nginx -t

# 重新加载Nginx
systemctl reload nginx
```

### 9.8 第七阶段：SSL证书（两种方式选一）

**方式A：Let's Encrypt 免费证书（自动续期，推荐）**

```bash
# 1. 安装certbot
apt install certbot python3-certbot-nginx -y

# 2. 申请证书（域名必须已解析到ECS公网IP）
certbot --nginx -d 你的域名.com

# 3. 测试自动续期
certbot renew --dry-run

# 4. 设置自动续期定时任务（certbot会自动添加，确认一下）
systemctl list-timers | grep certbot
```

**方式B：阿里云免费DV证书（手动，1年有效）**

```bash
# 1. 阿里云控制台 → SSL证书 → 免费证书 → 申请 → 填写域名
# 2. 下载证书（选Nginx格式），得到 cert.pem 和 key.pem
# 3. 上传到ECS：
mkdir -p /etc/nginx/ssl
# 把 cert.pem 和 key.pem 上传到 /etc/nginx/ssl/
# 4. 确认Nginx配置中路径正确，reload Nginx
```

### 9.9 第八阶段：systemd开机自启

```bash
# 创建服务文件
cat > /etc/systemd/system/wisecho.service << 'SVCEOF'
[Unit]
Description=Wisecho AI Water Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/wisecho
Environment=DASHSCOPE_API_KEY=你的通义千问API密钥
ExecStart=/opt/wisecho/venv/bin/python wisecho_server.py
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
SVCEOF

# 重新加载服务配置
systemctl daemon-reload

# 设置开机自启
systemctl enable wisecho

# 立即启动
systemctl start wisecho

# 查看状态（应该显示 active (running)）
systemctl status wisecho

# 实时查看日志
journalctl -u wisecho -f
```

### 9.10 第九阶段：更新App连接地址

修改 `index.html` 中的两行配置：

```javascript
// 原内网地址（注释掉）
// const WISECHO_URL = 'http://192.168.3.22:8000';
// const WISECHO_WS  = 'ws://192.168.3.22:8000/ws/ask';

// 改为阿里云地址
const WISECHO_URL = 'https://wisecho.cn';
const WISECHO_WS  = 'wss://wisecho.cn/ws/ask';
```

然后上传到GitHub，GitHub Pages自动发布。

### 9.11 阿里云安全组配置（容易被忽略！）

```
阿里云控制台 → ECS → 安全组 → 配置规则 → 手动添加：

入方向规则：
端口80   协议TCP  来源0.0.0.0/0  允许  （HTTP）
端口443  协议TCP  来源0.0.0.0/0  允许  （HTTPS）
端口8000 协议TCP  来源0.0.0.0/0  允许  （直接访问，测试用）
端口22   协议TCP  来源0.0.0.0/0  允许  （SSH，可限制为你的IP）

注意：即使wisecho_server.py启动成功，不加安全组规则也访问不了！
```

### 9.12 域名解析配置

```
阿里云控制台 → 域名 → 解析设置 → 添加记录：

记录类型：A
主机记录：@（代表根域名）
记录值：你的ECS公网IP
TTL：10分钟

再添加一条：
记录类型：A  
主机记录：www
记录值：你的ECS公网IP
```

### 9.13 阿里云部署完整验证清单

```
□ SSH登录ECS成功
□ systemctl status wisecho → active (running)
□ curl http://127.0.0.1:8000 → 返回JSON状态
□ nginx -t → 配置语法正确，systemctl reload nginx
□ 域名已解析（ping 你的域名.com → 显示ECS公网IP）
□ https://你的域名.com 可以访问（200 OK）
□ https://你的域名.com/tts?text=上善若水 → 返回WAV音频
□ WSS wss://你的域名.com/ws/ask → 连接成功
□ 手机Safari访问 → 能播放音频
□ 重启ECS后服务自动恢复（systemctl is-enabled wisecho）
□ index.html中的WISECHO_URL已改为https地址
□ GitHub Pages发布后，手机访问功能正常
```

### 9.14 常见阿里云故障

| 症状 | 原因 | 解决 |
|------|------|------|
| 域名访问超时 | 安全组未开80/443 | 9.11节添加规则 |
| 域名访问超时 | 域名未解析到ECS | 9.12节配置DNS |
| HTTPS证书错误 | 证书路径配置错误 | 检查/etc/nginx/ssl/路径 |
| WS连接失败 | Nginx缺少upgrade配置 | 检查9.7节WebSocket配置 |
| iOS音频不播放 | 缺Accept-Ranges头 | 检查server返回头 |
| ECS TTS无声音 | 误用了Windows System.Speech | 改用dashscope CosyVoice |
| API请求失败 | API Key未设置 | systemctl中Environment配置 |
| 服务启动失败 | 依赖未安装 | pip install fastapi uvicorn dashscope |

---

## 十、关键配置速查

```javascript
// ── 激活码密钥（不可更改）──
var ACTIVATION_SECRET = 'heshuile2026shuiruoshan';

// ── 通讯录永远不上云 ──
var CLOUD_CONFIG = { enabled: false, apiUrl: '', apiKey: '' };

// ── AI服务器地址（二选一）──
// 内网测试时用：
const WISECHO_URL = 'http://192.168.3.22:8000';
const WISECHO_WS  = 'ws://192.168.3.22:8000/ws/ask';
// 阿里云上线后改为：
// const WISECHO_URL = 'https://wisecho.cn';
// const WISECHO_WS  = 'wss://wisecho.cn/ws/ask';
```

---

## 十一、GitHub 仓库文件清单

| 文件 | 说明 |
|------|------|
| index.html | 主应用（当前 V5.5）|
| SKILL-heshuile-V5.5.md | 本规范文件（唯一权威版本）|
| members.json | 云端会员白名单 |
| 盛兴崑收款码.jpg | 收款码 |

> ⚠️ 每次上传GitHub后，请删除旧版SKILL文件，只保留最新版本

---

## 十二、版本历史（权威唯一记录）

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| V1.0 | 2026-01 | 初始版本（346行）|
| V2.0 | 2026-02 | 会员系统、支付、激活码 |
| V3.0 | 2026-02-20 | 水学院wisdomPanel、导航重构 |
| V4.0 | 2026-02-20 | 水健康报告、通讯录裂变引擎 |
| V4.1 | 2026-02-20 | 邮件发给用户自己、数据分层 |
| V5.0 | 2026-02-20 | 智回听书融合，语音第一设计 |
| V5.1 | 2026-02-22 | 水老AI服务器：30408块RAG，WebSocket |
| V5.2 | 2026-02-22 | TTS System.Speech，iOS Safari可播放 |
| V5.3 | 2026-02-27 | 重建SKILL，建立强制版本更新纪律 |
| V5.4 | 2026-02-27 | FND水色系配色（#0085D0/#EB6100/#0B1120），开屏版本公告 |
| **V5.5** | **2026-02-28** | **语音AI文字输入+Claude API接入，关怀提醒引导UI+SW后台推送，广告乱码缓存修复，补充内网4090和阿里云双部署完整方案** |
