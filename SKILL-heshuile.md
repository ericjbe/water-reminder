---
name: heshuile-water-reminder
version: V5.0
date: 2026-02-20
description: 喝水了吗（Have You Drunk Water）· 智慧时光 — 开发维护总规范。单文件 HTML PWA，内嵌「智回听书」水学音频系统。涵盖会员、支付、激活、通讯录、裂变、语音、水学课程、水健康报告全部子系统。修改此应用时必须遵循本规范。
---

# 喝水了吗 · 智慧时光 开发维护 Skill V5.0

---

## 一、项目概述

「喝水了吗」是部署在 GitHub Pages 的单文件 PWA 应用：  
**`https://ericjbe.github.io/water-reminder/`**

**技术约束**：所有 HTML + CSS + JS 在一个 `index.html` 中，无后端，数据存 `localStorage`。

### 战略定位（三层）

```
第一层：工具价值     喝水提醒 + 关怀关系 + 健康日报
第二层：内容价值     水学智慧课程（内嵌「智回听书」系统）
第三层：商业价值     会员订阅 + 社交裂变 + 企业定制
```

### 核心理念：「听书式」水学传播

> 参照智回(Wisecho)听书系统设计：以**语音为第一界面**，  
> 兼顾盲人用户习惯，让用户「闭眼也能学水学、得健康」。

联系方式：手机 18680666987 · 微信 FCLFUND · 邮箱 artstoneusa@gmail.com

---

## 二、架构规则（必须遵守）

### 2.1 单文件约束

- 所有代码必须在一个 `index.html` 文件中
- 文件中有多个 `<script>` 块，主业务逻辑在**最后一个**
- 修改后必须用 `node --check` 验证 JS 语法：

```bash
python3 -c "
import re
with open('index.html','r') as f: c=f.read()
scripts = re.findall(r'<script[^>]*>(.*?)</script>', c, re.DOTALL)
with open('/tmp/check.js','w') as f: f.write(scripts[-1])
"
node --check /tmp/check.js
```

### 2.2 引号陷阱（最常见 Bug 来源）

- ❌ 禁止在 onclick 内联事件中嵌套同种引号
- ✅ 方案1：用 `&#39;` 替代内嵌单引号
- ✅ 方案2：提取为独立函数 + 事件绑定（`el.onclick = function(){...}`）
- ✅ 方案3：用模板字符串拼接 HTML 时用 `"` 包裹属性值

### 2.3 DOM 元素防空保护

```javascript
// 所有 getElementById 必须加 null 守卫
var el = document.getElementById('someId');
if (el) el.textContent = value;
```

### 2.4 localStorage 键名规范

所有键名必须以 `heshuile` 前缀开头（例外：`currentUser`、`pwaDismissed`）：

| 键名 | 内容 |
|------|------|
| `heshuileData` | 主数据（滴答/主题/音效/好友/提醒/签到/联系人/水学进度）|
| `heshuileUsers` | 用户表 `{phone: {name,pwd,plan,expiry,email,...}}` |
| `heshuileMembers` | 会员激活状态缓存 |
| `heshuilePendingOrders` | 待审核支付订单 |
| `heshuileActivityLog` | 活动日志（提醒触发、课程学习、祝福发送）|
| `heshuileReportHistory` | 水健康日报历史（最近30天）|
| `heshuileContactSend` | 裂变发送进度 `{sentIds[],totalSent,lastBatch}` |
| `heshuileListenProgress` | 「智回听书」收听进度 `{bookKey, chapterIdx, lessonIdx, position}` |
| `heshuileListenHistory` | 收听历史记录（最近100条）|

### 2.5 正则表达式规范

- 禁止在正则中直接写实际换行符：`/[\n]+/` → 必须写 `/[\r\n]+/`
- 所有跨行正则用 `[^\r\n]` 而不是 `[^\n]`

---

## 三、子系统详解

### 3.1 用户与会员系统

**会员等级**：`free` → `monthly` → `quarterly` → `vip`(yearly) → `enterprise`

**数据结构**（存在 `heshuileUsers[phone]`）：
```javascript
{
  phone: '13800138001',
  name: '张三',
  email: 'zhang@email.com',   // 用于接收水健康报告
  pwd: 'hashed_password',
  plan: 'vip',                // free/monthly/quarterly/vip/enterprise
  expiry: '2027-01-01',
  inviteCode: 'HSL_ABC123',   // 邀请码（裂变追踪）
  invitedBy: 'HSL_XYZ456',    // 被谁邀请
  listenLevel: 3              // 听书等级（完成课程数）
}
```

**会员权益分层**：

| 功能 | 免费 | 月度 | 季度 | 年度VIP | 企业 |
|------|------|------|------|---------|------|
| 喝水提醒 | 3条 | 10条 | 20条 | 无限 | 无限 |
| 关怀人数 | 3人 | 10人 | 20人 | 无限 | 无限 |
| 水学听书 | 前3章 | 前5章 | 全部 | 全部+速听 | 全部+下载 |
| 水健康报告 | 7天 | 30天 | 90天 | 365天 | 自定义 |
| 数据存储 | 本地 | 本地 | 本地+手动云备 | 自动云同步 | 管理员可查 |
| 联系人数据 | **永远本地** | **永远本地** | **永远本地** | **永远本地** | **永远本地** |

> ⚠️ **联系人数据原则**：无论何种会员，联系人数据永远只存用户设备本地，不上传服务器。这是核心信任承诺。

### 3.2 三重会员激活系统

#### 层级1：激活链接（即时）
```
https://ericjbe.github.io/water-reminder/?activate=PLAN_PHONE_HASH
```

#### 层级2：激活码手动输入
- 生成规则：`btoa(phone + '_' + plan + '_' + ACTIVATION_SECRET).slice(0,12).toUpperCase()`
- 用户在 设置→我的账户→激活码 输入
- 函数：`activateByCode()` / `activateByCodeS()`

#### 层级3：云端校验
- URL：`https://raw.githubusercontent.com/ericjbe/water-reminder/main/members.json`
- 每6小时自动校验一次
- 密钥：`var ACTIVATION_SECRET = 'heshuile2026shuiruoshan';`

### 3.3 支付系统

**流程**：用户选套餐 → 扫收款码 → 点「已完成付款」→ 弹出友好确认卡 → 一键发邮件通知管理员 → 管理员审核 → 发激活码

**收款信息**（PAY_CONFIG）：
```javascript
var PAY_CONFIG = {
  contactPhone: '18680666987',
  contactWechat: 'FCLFUND',
  contactName: '盛兴崑',
  wechatPayQR: 'GitHub raw URL',   // 微信收款码图片
  alipayQR: 'GitHub raw URL',       // 支付宝收款码图片
  prices: {
    monthly:    { amount: 9.99,  display: '¥9.99/月',   name: '月度会员' },
    quarterly:  { amount: 19.99, display: '¥19.99/季',  name: '季度会员' },
    yearly:     { amount: 199,   display: '¥199/年',    name: '年度VIP 🔥' },
    enterprise: { amount: 9999,  display: '¥9999/年',   name: '企业版' }
  }
};
```

**管理员邮件通知**：支付后自动生成 mailto 链接，收件人 = 管理员两个邮箱（不是用户）。

### 3.4 通讯录系统

**导入方式（按优先级）**：
1. Android Chrome Contact Picker API（`navigator.contacts.select`）
2. vCard 文件（`.vcf`）解析——iPhone/安卓导出通讯录
3. CSV 文件（`姓名,手机号` 格式）
4. 批量文本录入（每行：`姓名 手机号`）
5. 手动单个添加

**关键原则**：
- 联系人数据 **永远不上传云端**（已注释掉所有上传代码）
- `contacts[]` 只存 localStorage
- `friends[]` = 已设置关怀提醒的联系人子集

### 3.5 春节/节日祝福裂变系统

- 面板：`festivalPanel`
- 祝福发送追踪：每次发送记入活动日志
- 收件人选择：`showFestivalPicker()` 弹出联系人选择器（含搜索）
- 祝福模板：4套场景（水提醒/生日/节日/水学邀请）

### 3.6 通讯录裂变引擎（viralPanel）

**进入路径**：底部「共享」→ 关怀 → 底部「🚀一键群发·启动裂变引擎」

**批次规则**：每批9人（微信群发上限），系统自动分批

**工作流程**：
1. `viralInit()` 加载联系人，过滤已发，计算批次
2. `viralRenderBatch()` 渲染当前9人批次 + 自动生成文案
3. 用户点「📋 复制文案」→ 去微信群发
4. 返回点「✅ 已发送此批」→ `viralMarkSent()` 记录 → 自动跳下一批

**消息模板**（4套场景）：
- 💧 喝水提醒：邀请用户用「喝水了吗」
- 🎂 生日祝福：个性化生日水学祝福
- 🎊 节日祝福：节日水学名言
- 📚 水学邀请：邀请听《水科学》课程

**裂变追踪**：每个邀请链接附加 `?invite=userId&from=viral`，追踪1×3裂变效果。

### 3.7 水健康日报系统（waterReportPanel）

**进入路径**：首页 → 今日饮水区 → 📊水健康报告

**报告发送原则**：
> ✅ 报告发给**用户自己**的邮箱  
> ❌ 不能发给管理员  
> 如用户未留邮箱，弹窗让用户填写并保存

**报告内容**：
- 今日饮水 ml / 目标 / 完成率
- 健康评分（A/B/C/D）
- 今日水学课程摘要（内嵌课程内容）
- 提醒执行次数
- 7日趋势图

**数据存储**：`heshuileReportHistory`，保留30天，全等级本地存储。

---

## 四、🎧 智回听书·水学音频系统（核心子系统）

> 这是应用的灵魂功能。参照「智回(Wisecho)听书」系统设计，  
> 将水学内容以**语音朗读为第一优先**的方式传达给用户。  
> 兼顾盲人用户习惯：用户闭上眼睛，也能完整学习水学内容。

### 4.1 设计哲学

```
智回听书原则（来自Wisecho V1.0手册）：
├── 语音 > 文字：内容优先用声音传达
├── 流式播放：不等加载完成，边生成边播放
├── 情境感知：根据时间/行为自动推送合适内容
├── 零操作学习：打开即听，不需要点击翻页
└── 「水老」人格：以水老身份讲述，有温度有深度
```

### 4.2 内容架构（W_BOOKS）

```javascript
var W_BOOKS = {
  philosophy: {        // 水哲学·道德经/庄子
    name: '水哲学', emoji: '🌊',
    reqLevel: 'free', freeChapters: 3,
    chapters: [...]
  },
  science: {           // 水科学·三卷本（智回RAG核心内容）
    name: '水科学', emoji: '💧',
    reqLevel: 'monthly',
    chapters: [...]
  },
  wisdom: {            // 水智慧·孙子兵法/商道
    name: '水智慧', emoji: '☯️',
    reqLevel: 'quarterly',
    chapters: [...]
  },
  health: {            // 水健康·养生实践
    name: '水健康', emoji: '🌿',
    reqLevel: 'free', freeChapters: 2,
    chapters: [...]
  }
};
```

每个 lesson 的数据结构：
```javascript
{
  title: '第1课·水的哲学本质',
  original: '上善若水。水善利万物而不争。',  // 原典引文
  content: '老子以水喻道...',                // 现代解读（300-500字）
  audioHint: '语速0.85，富有感情',           // TTS朗读提示
  duration: 180,                              // 预计朗读秒数
  tags: ['道德经', '哲学', '入门']
}
```

### 4.3 每日四次触达机制（必须全部实现）

**触达1 — 晨间推送（每日7:00）**
```javascript
function scheduleDailyWaterLesson() {
  // 计算到明天7:00的毫秒数，setTimeout
  // 触发系统通知：「📚 今日水学：[课程标题]，点击收听」
  // 通知点击 → openWisdomPanel() → 自动播放
  fireNotification('📚 今日水学：' + lesson.title, lesson.book + '·点击收听');
}
```

**触达2 — 首页每日水学条**
```html
<!-- 首页滚动区·今日饮水区域·提醒列表之前 -->
<div id="dailyWisdomBar" onclick="openTodayLesson()">
  <!-- 今日课程标题 + 书名 + 「▶ 收听」按钮 -->
</div>
```

**触达3 — 语音助手开场白**
```javascript
function toggleVoice() {
  // 点🎤按钮时：先朗读今日课程片段（约30秒）
  // 朗读结束后自动开启语音识别，等待用户问问题
  // 这样用户每次"问水老"都先听到水学内容
}
```

**触达4 — 喝水提醒联动**
```javascript
function checkReminders() {
  // 每次提醒触发时，Toast消息附带今日水学名言
  toast('⏰ 该喝水了！「' + getTodayQuote().text.slice(0,20) + '」');
}
```

### 4.4 核心函数规范

#### getTodayLesson() — 今日课程（按日期自动推进）
```javascript
function getTodayLesson() {
  // 从所有书的所有章节展平成一维数组
  // 以2026-01-01为基准日，按自然天数取余轮播
  // 确保用户每天打开看到的是不同课程
  var dayIdx = Math.floor((Date.now() - new Date(2026,0,1)) / 864e5);
  return allLessons[Math.abs(dayIdx) % allLessons.length];
}
```

#### autoReadTodayLesson() — 自动朗读
```javascript
function autoReadTodayLesson() {
  var lesson = getTodayLesson();
  // 朗读顺序：书名 → 课程标题 → 原典 → 现代解读（前100字）
  // 使用 SpeechSynthesisUtterance，lang='zh-CN', rate=0.88
  // 朗读完毕后不自动跳转，保持安静
}
```

#### wInitBooks() — 水学院著作Tab初始化
```javascript
function wInitBooks() {
  // 渲染所有书籍列表
  // 高亮今日课程所在书籍/章节
  // 显示收听进度（从 heshuileListenProgress 读取）
  // 未解锁课程显示「🔒 升级会员解锁」
}
```

#### 收听进度保存
```javascript
function saveListenProgress(bookKey, chapterIdx, lessonIdx) {
  localStorage.setItem('heshuileListenProgress', JSON.stringify({
    bookKey, chapterIdx, lessonIdx,
    savedAt: new Date().toISOString()
  }));
  // 同时记录到活动日志
  logActivity('listen', W_BOOKS[bookKey].chapters[chapterIdx].lessons[lessonIdx].title);
}
```

### 4.5 「智回听书」进阶功能（按优先级）

**P1 — 当前已实现**：
- Web Speech Synthesis API 朗读
- 按日期自动推进课程
- 首页水学条 + 7:00推送 + 语音助手联动 + 提醒带名言

**P2 — 下阶段实现**：
- 收听进度保存（离开再回来从断点继续）
- 速度控制（0.75x / 1.0x / 1.25x / 1.5x）
- 朗读完一课自动播下一课（连续模式）
- 「复习模式」：随机播放已听过的课程

**P3 — 高级会员功能（对接本地4090服务器）**：
- WebSocket 连接 RTX 4090 服务器
- 高质量 TTS（替代浏览器自带语音）
- 「水老问答」：语音提问 → RAG检索《水科学》→ 流式语音回答
- 对应智回手册中的 `/ws/debate` WebSocket 接口

```javascript
// P3 高质量语音（年度VIP+）
var WISECHO_WS_URL = 'ws://192.168.x.x:8000/ws/debate'; // 4090服务器
function connectWisechoServer() {
  if (!isVipUser()) return;
  var ws = new WebSocket(WISECHO_WS_URL);
  ws.onmessage = function(e) {
    // 接收PCM音频块，实时播放（仿WisechoPlayer.java）
    playAudioChunk(e.data);
  };
}
```

### 4.6 「水老」AI人格

> 所有语音内容以「水老」身份讲述：
> - 称谓：用户叫「水友」，自称「水老」
> - 语气：深沉、智慧、温暖、不急促
> - 开场白：「水友，今日水学，来自[书名]，[课程标题]。」
> - 结语：「上善若水，愿你健康每一天。」

---

## 五、UI/UX 设计规范（2026-02 确立，不可更改）

### 5.1 底部导航（5栏固定）

```
🏠 首页 | 📣 共享 | 🎓 水学院 | 🎁 积分 | ⚙️ 设置
```

- 「水学院」= wisdomPanel（包含时钟·著作·祝福三Tab）
- **禁止**在底部导航加第6个Tab
- **禁止**修改Tab顺序

### 5.2 顶部导航（极简）

只保留：`左侧签到积分按钮` + `右侧会员等级徽章`  
禁止在顶部加入任何其他按钮。

### 5.3 首页布局顺序（自上而下，不可乱序）

```
1. 顶部导航栏（签到 + 会员徽章）
2. 滴答时钟主体
3. 今日饮水区
   3a. 今日水学条（dailyWisdomBar）← 新增，固定位置
   3b. 即将到来的提醒列表
4. 功能快捷按钮行（经典·语音·关怀·水健康报告）
```

### 5.4 功能去重原则

| 功能 | 唯一入口 | 禁止在哪里重复 |
|------|---------|--------------|
| 登录/注册 | 设置面板 sLoginForm | 首页、其他面板 |
| 水学院 | 底部「🎓 水学院」 | 顶部导航、首页按钮 |
| 经典/语音/关怀/水健康报告 | 首页「今日饮水」区底部4按钮 | 水学院panel内 |
| 裂变发送 | 关怀面板底部「🚀一键群发」 | 其他地方 |

### 5.5 wisdomPanel三Tab内容（固定）

```
Tab0 时钟：滴答时钟 + 今日水学速览卡（wTodayLessonCard）+ 今日名言
Tab1 著作：水学书籍列表 + 章节展开 + 逐课收听（核心听书Tab）
Tab2 祝福：生成祝福文案 + 发送
```

---

## 六、修改流程规范

### 6.1 修改前

1. 读本 SKILL 相关章节
2. 确认影响范围（哪些面板/函数）
3. 制作修改计划（不跳步骤）

### 6.2 修改中

- 优先用 `str_replace` 精确替换，不整体重写
- 每次注入后立即验证 JS 语法
- 内联 onclick 中调用函数，一律用 `&#39;` 或事件绑定

### 6.3 修改后（必做验证）

```python
import re, subprocess
from collections import Counter
with open('index.html','r',encoding='utf-8') as f: c = f.read()
scripts = re.findall(r'<script[^>]*>(.*?)</script>', c, re.DOTALL)
with open('/tmp/check.js','w') as f: f.write(scripts[-1])
r = subprocess.run(['node','--check','/tmp/check.js'], capture_output=True)
ids = re.findall(r'id="([^"]+)"', c)
dupes = {k:v for k,v in Counter(ids).items() if v>1}
ko = re.findall(r'[가-힣]+', c)
print(f"JS: {'✅' if r.returncode==0 else '❌'+r.stderr.decode()[:100]}")
print(f"重复ID: {'✅ 无' if not dupes else '❌'+str(list(dupes.keys()))}")
print(f"韩文: {'✅ '+str(len(ko))+'处' if ko else '❌ 丢失'}")
```

---

## 七、SKILL 同步规则

**原则**：每次代码修改，SKILL 同步更新，同时输出两个文件。

**Claude 操作流程**：
1. 修改 `index.html` → 通过验证 → 输出到 `/mnt/user-data/outputs/index.html`
2. 同步更新 SKILL → 输出到 `/mnt/user-data/outputs/SKILL-heshuile.md`
3. 告知用户：「本次修改已完成，请将 index.html 和 SKILL-heshuile.md 都上传到 GitHub 仓库根目录」

**GitHub 仓库**：`ericjbe/water-reminder`  
**注意**：Claude 无法直接 push GitHub，用户需手动上传。

---

## 八、GitHub 仓库文件清单

| 文件 | 说明 |
|------|------|
| `index.html` | 主应用（唯一部署文件）|
| `SKILL-heshuile.md` | 本规范文件（必须与 index.html 同步）|
| `members.json` | 云端会员白名单（手动更新）|
| `盛兴崑收款码.jpg` | 微信/支付宝收款码 |

---

## 九、关键配置速查

只改值，不改结构：

```javascript
// 激活码密钥
var ACTIVATION_SECRET = 'heshuile2026shuiruoshan';

// 云端会员校验
var MEMBERS_CLOUD = {
  enabled: true,
  url: 'https://raw.githubusercontent.com/ericjbe/water-reminder/main/members.json',
  checkInterval: 6 * 60 * 60 * 1000
};

// 支付配置
var PAY_CONFIG = {
  contactPhone: '18680666987',
  contactWechat: 'FCLFUND',
  contactName: '盛兴崑',
  wechatPayQR: 'GitHub raw URL',
  alipayQR: 'GitHub raw URL'
};

// 通讯录云端同步（永远关闭）
var CLOUD_CONFIG = { enabled: false, apiUrl: '', apiKey: '' };

// 管理员密码
var ADMIN_KEY = '...';

// 智回听书服务器（4090，仅年度VIP+）
var WISECHO_WS_URL = ''; // ws://192.168.x.x:8000/ws/debate
```

---

## 十、常见故障排查

| 症状 | 原因 | 解决 |
|------|------|------|
| 页面空白 | JS 语法错误（引号嵌套） | `node --check` 定位 |
| TypeError: null | getElementById 找不到元素 | 加 null 守卫 |
| 正则语法错误 | 正则里有实际换行 | 改为 `[\r\n]` |
| 会员状态丢失 | localStorage 被清 / 换设备 | 激活链接或云端校验恢复 |
| 收款码无法扫 | 手机不能扫自己屏幕 | 用「保存到相册」功能 |
| 通讯录读不到 | 不是 Android Chrome / 非 HTTPS | 降级到 vCard 文件导入 |
| 语音不播放 | 浏览器未授权 / iOS Safari 限制 | 需用户手动点击触发 |
| 激活码不匹配 | 密钥被改 / 手机号格式不对 | 检查 ACTIVATION_SECRET |
| 韩文乱码 | 编码保存问题 | 检查编码，保留235处韩文 |

---

## 十一、版本历史

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| V1.0 | 2026-01-xx | 初始版本（346行）|
| V2.0 | 2026-02-xx | 会员系统、支付、激活码 |
| V3.0 | 2026-02-20 | 水学院（wisdomPanel）、导航重构 |
| V4.0 | 2026-02-20 | 水健康报告、通讯录裂变引擎 |
| V4.1 | 2026-02-20 | 邮件发给用户自己、数据分层、水学每日推送 |
| **V5.0** | **2026-02-20** | **全面重构SKILL，融合「智回听书」系统，语音第一设计** |
