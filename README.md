Eternal Moments - 云端纪念册 (Serverless Photo Gallery)
Eternal Moments 是一个基于 HTML5 和 阿里云对象存储 (OSS) 构建的轻量级、无服务器（Serverless）个人纪念册应用。

本项目采用了极简的单页应用（SPA）架构，利用 OSS 既作为图床又作为轻量级 JSON 数据库，实现了照片瀑布流展示、留言板互动以及点赞功能。

🛠 技术栈 (Tech Stack)
前端核心:原生 HTML5 / CSS3 / ES6+ JavaScript。

UI 风格: Glassmorphism (毛玻璃特效) + 樱花飘落动画 + 响应式布局 (Grid/Flex)。

数据存储 (BaaS): 阿里云 OSS (Object Storage Service)。

媒体存储: 图片文件存放在 memories/ 目录下。

数据持久化: 留言和点赞数据存储在 site_data_v2.json 文件中。

SDK: aliyun-oss-sdk-6.17.1。

🚀 快速开始 (Quick Start)
1. 配置阿里云 OSS
在使用前，您必须在代码中配置您的阿里云凭证。

打开 index.html，找到 <script> 标签内的配置区域：

JavaScript
// ==========================================
// 🔴 必须配置: 填入你的阿里云信息
// ==========================================
const ossConfig = {
    region: 'oss-cn-beijing',      // 例如: oss-cn-beijing
    accessKeyId: 'YOUR_ACCESS_KEY_ID',
    accessKeySecret: 'YOUR_ACCESS_KEY_SECRET',
    bucket: 'dangphuongchinh'
};
⚠️ 安全警告: AccessKey 包含极其敏感的权限。在生产环境中，建议使用 STS (Security Token Service) 生成临时 Token，或限制该 Key 的权限仅能访问特定 Bucket。切勿将包含真实 Key 的代码直接提交到公开仓库 (GitHub/Gitee)。

2. CORS 设置
为了让浏览器能直接通过 JS 访问 OSS，您需要在阿里云 OSS 控制台的 权限管理 -> 跨域设置 (CORS) 中添加规则：

来源: * (或您的具体域名)

允许 Methods: GET, PUT, POST, HEAD

允许 Headers: *

3. 运行
直接在浏览器中打开 index.html 或是通过 VS Code 的 Live Server 启动即可。

🧩 核心模块与逻辑流定义 (Architecture & Logic Flow)
为了确保代码的可维护性与逻辑清晰度，本项目主要划分为以下三个核心处理模块。

模块 I: 数据初始化与同步 (Data Synchronization)
负责应用启动时的数据拉取及后续的状态同步。

输入 (Input):

阿里云 OSS 凭证 (ossConfig)。

目标 Bucket 中的 site_data_v2.json (数据库文件) 和 memories/ 前缀下的对象列表。

处理过程 (Process):

连接: 初始化 OSS Client。

并行/串行获取:

尝试 client.get('site_data_v2.json')。若文件不存在 (404)，则在内存中初始化空结构 { messages: [], likes: {} }。

执行 client.list() 获取图片列表，并按 lastModified 倒序排列。

解码: 将 JSON 文件的 Buffer 解码为 UTF-8 字符串并解析为 JS 对象 (siteData)。

输出 (Output):

更新内存中的 siteData 全局状态。

触发 UI 渲染函数 renderGallery() 和 renderMessages()。

模块 II: 图片上传 (Image Upload)
负责处理用户上传图片并写入元数据。

输入 (Input):

用户选择的 DOM File 对象 (fileInput)。

用户输入的描述文本 (descInput)。

处理过程 (Process):

校验: 检查是否选择了文件。

命名策略: 生成唯一文件名 memories/{Timestamp}_{OriginalName} 以防止覆盖。

元数据编码: 将描述文本进行 Base64 + URI 编码，封装在 meta: { description: ... } 中（注：目前用于存储，List 接口暂不直接返回 Meta，需单独 Head 请求或优化架构）。

云端写入: 调用 client.put() 上传文件。

输出 (Output):

上传成功后清空输入框。

递归调用 模块 I 重新拉取最新列表刷新 UI。

模块 III: 交互持久化 (Interaction Persistence)
处理留言和点赞，实现简易的“读写分离”策略（乐观更新）。

输入 (Input):

用户交互事件（点击点赞、发送留言）。

当前的全局状态 siteData。

处理过程 (Process):

乐观更新 (Optimistic UI): 先直接修改内存中的 siteData 并更新 DOM，使用户感觉“零延迟”。

序列化: 将更新后的 siteData 对象转换为 JSON 字符串。

全量覆盖: 调用 client.put('site_data_v2.json', blob) 覆盖云端旧文件。

输出 (Output):

云端数据库更新完成。

(异常处理) 若网络请求失败，控制台输出错误（生产环境应回滚 UI 状态）。

✨ 功能特性 (Features)
沉浸式视觉体验: 全局 CSS 变量管理配色，包含樱花飘落 JS 动画与加载状态遮罩。

云端画廊: 自动读取 OSS 指定目录下的所有图片，支持懒加载 (Lazy Load)。

留言板系统: 支持文本留言，按时间倒序排列。

点赞系统:

支持对单张照片点赞。

支持对单条留言点赞。

点赞状态通过文件名/ID 映射存储，保证数据一致性。

📂 目录结构 (Directory Structure)
Plaintext
.
├── index.html            # 核心入口 (包含 HTML/CSS/JS)
├── README.md             # 说明文档
└── (Cloud Storage)       # 云端逻辑结构
    ├── memories/         # 图片存储目录
    │   ├── 1735112233_img1.jpg
    │   └── ...
    └── site_data_v2.json # 核心数据库文件 (JSON)
🔮 未来扩展性建议 (Future Scalability)
基于您对代码扩展性的关注，若后续需要升级该项目，建议：

将配置抽离: 将 ossConfig 移至独立的 config.js 并从 .gitignore 中排除，增强安全性。

模块化重构: 将 CSS、JS 逻辑（OSS 操作、UI 渲染、事件监听）拆分为独立文件。

并发锁机制: 当前的 JSON 覆盖写操作在极高并发下存在“竞争条件 (Race Condition)”。若用户量增大，建议引入阿里云表格存储 (Tablestore) 或使用 Serverless 函数 (FC) 来处理原子化写入。
