# 📚 Telegram Z-Library to Kindle 自动化推书工作流

> 零代码、免服务器、24小时云端运行：在 Telegram 收到 Z-Library 电子书后，只需顺手「转发」一下，即可自动推送至你的 Kindle 电纸书（支持同时推给好友）。

---

## ✨ 方案亮点

- **零代码 & 零服务器成本**：基于 [Make.com](https://www.make.com/) 免费版（每月 1,000 次免费额度）搭建，无需购买 VPS 或配置代码运行环境。
- **双 Bot 接力架构**：保留原有的 Z-Library 官方绑定机器人专门负责找书，新建一个专属中转机器人负责收发文件，互不干扰。
- **24 小时云端待命**：配置完成后可彻底关闭浏览器和电脑，随时随地在手机端转发推书。
- **智能防错过滤**：内置过滤器自动拦截纯文本或 `/start` 等无效指令，仅在接收到电子书文件时触发推送。
- **支持好友共享**：可轻松扩展为同时推送给多台 Kindle，或通过分支路由实现多人共用同一个推书机器人。

---

## 🔄 工作流原理

```text
[Z-Library 官方 Bot] 
       │  (搜索并下载 .epub 电子书)
       ▼
  [你的 Telegram] 
       │  (长按文件 -> 转发)
       ▼
[中转 Bot (@BotFather创建)] 
       │  (Webhook 实时触发)
       ▼
[Make.com 云端自动化] 
  ├── 1. Watch Updates (监听新消息)
  ├── 2. Filter 过滤器 (仅放行包含 File ID 的文件)
  ├── 3. Download a File (抓取文件至云端内存)
  └── 4. Gmail Send Email (作为附件发送)
       │
       ▼
 [亚马逊 @kindle.com] ──▶ 自动同步至 Kindle 书架
```

---

## 🛠️ 保姆级搭建步骤

### 第一步：亚马逊后台配置（关键前提）

亚马逊的「Send to Kindle」服务仅接收白名单邮箱发来的文档：

1. 登录亚马逊官网，进入 **管理我的内容和设备** ➔ **首选项** ➔ **个人文档设置**。
2. 记下你设备对应的 **`@kindle.com`** 接收邮箱。
3. 在 **“已认可的个人文档电子邮箱列表”** 中，点击添加新的电子邮箱地址，填入你准备用来发信的邮箱（推荐 **Gmail**）。

---

### 第二步：创建 Telegram 中转机器人

> ⚠️ **注意**：请勿直接使用你原有的 Z-Library 机器人 Token，因为一个 Bot Token 只能绑定一个后台服务器。

1. 在 Telegram 搜索官方账号 **`@BotFather`**。
2. 发送指令 `/newbot`，按提示输入机器人的显示名称（如 `My Kindle Sender`）和英文用户名（必须以 `bot` 结尾，如 `my_kindle_sender_bot`）。
3. 复制保存生成的 **HTTP API Token**。
4. 点击 `@BotFather` 回复消息中的 `t.me/你的机器人用户名` 链接进入聊天窗口，点击底部的 **Start** 激活机器人。

---

### 第三步：在 Make.com 搭建自动化连线

1. 注册并登录 [Make.com](https://www.make.com/)，点击右上角 **Create a new scenario**。
2. **添加模块 1（监听消息）**：
   - 搜索并选择 **Telegram Bot** ➔ **Watch Updates**。
   - 点击 **Create a webhook** ➔ **Create a connection**，粘贴第二步获取的 **API Token** 并保存。
3. **添加模块 2（下载文件）**：
   - 点击第一个圆圈右侧的 `+` 号，选择 **Telegram Bot** ➔ **Download a File**。
   - 在 **File ID** 输入框中，从右侧变量列表依次展开并点选：`Message` ➔ `Document/Image` ➔ `File ID`（确保输入框内只保留这一个标签）。
4. **设置过滤器（防报错关键）**：
   - 点击模块 1 和模块 2 之间连线上的扳手图标，选择 **Set up a filter**。
   - **Condition** 第一栏选择 `Message: Document/Image: File ID`，条件操作符选择 **`Exists`**（存在），点击 Save。
5. **添加模块 3（发送邮件至 Kindle）**：
   - 在模块 2 右侧点击 `+` 号，搜索并选择 **Gmail** ➔ **Send an Email**（需登录授权你的发件 Gmail）。
   - **To**：填写你的 `xxx@kindle.com` 邮箱。
   - **Subject**：随意填写（如 `Send to Kindle`）。
   - **Attachments**：点击 `Add an attachment`，在 `Source file` 中选择上一步的 `Telegram Bot - Download a File`。
6. **测试与上线**：
   - 点击左下角 **Run once**，在 Telegram 中将一本 `.epub` 电子书转发给中转机器人，确认三个模块均亮起绿色对勾。
   - 点击底部工具栏的 **Save（保存）** 图标，并将页面右上角开关从 `Inactive` 拨至 **开启状态**（频率保持 `Immediately as data arrives`）。

---

## ⚠️ 常见问题与避坑指南

- **为什么必须选 EPUB 格式？**
  亚马逊官方已停止对 `.mobi` 和 `.azw3` 格式的邮件推送支持。在 Z-Library 找书时请优先选择 **`.epub`** 格式，亚马逊云端会自动转换排版并支持全平台阅读进度同步。
- **大文件报错怎么办？**
  Telegram 官方接口限制普通机器人只能下载 **20MB 以内**的文件。绝大多数纯文字小说（1MB~5MB）均可秒传；若遇到几十 MB 的大型 PDF 或扫描版书籍，建议下载后直接通过手机端 **Kindle App** 分享导入。
- **报错 `Empty mapped value`（映射值为空）？**
  这是因为向机器人发送了纯文本消息（如 `/start`），消息中不含文件。请确保严格按照第三步第 4 点配置了 `File ID Exists` 过滤器。

---

## 👯‍♀️ 进阶玩法：推书给朋友

1. **前置条件**：让朋友在他自己的亚马逊后台，把你的发件 Gmail 加入他的白名单，并把他的 `@kindle.com` 邮箱发给你。
2. **同时推送**：在 Make 的 Gmail 模块中，点击 `Add a recipient` 添加朋友的 Kindle 邮箱即可。
3. **按需单独推送**：在 Make 的 Scenarios 列表中点击 **Clone** 克隆一份工作流，绑定第二个中转机器人专门用来给朋友推书；或使用 **Router（路由器）** 模块根据发送者的 `From: ID` 自动分流至不同人的 Kindle。

---

## 📄 License

本项目仅作为个人效率工具与自动化工作流思路分享，请尊重版权，支持正版阅读。
