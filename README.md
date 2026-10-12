# 网课小助手：DeepSeek / OpenAI 兼容 AI 版

基于 isMobile 编写的网课小助手 0.2.7 增加 AI 接口适配，修改版版本号为 0.2.8。支持 DeepSeek 和 OpenAI 兼容 Chat Completions 服务。

原脚本作者：isMobile；原脚本声明的许可证：MIT。AI 接口适配与本地修改由本仓库维护。

## 安装与使用

1. 在油猴（Tampermonkey）中停用原版网课小助手，避免两个脚本重复填写。
2. 新建用户脚本，用文本编辑器打开本仓库的 [`course-helper-ai.user.js`](course-helper-ai.user.js)，复制全部内容，替换油猴编辑器中的默认代码并保存。
3. 打开原脚本支持的超星或智慧树课程、作业页面，在浮动面板中切换到「答题」。
4. 选择「DeepSeek / OpenAI 兼容 AI」，填写自己的 DeepSeek API Key。
5. 点击「测试连接」。测试会向配置的接口发送一道简单题，消耗一次 API 调用；不会填写课程页面。
6. 显示连接成功后刷新课程页面。脚本将解析题目、调用 AI、转换答案并自动填写。
7. 在「设置」中调整自动提交、自动切题和间隔。超星章节作业的自动提交沿用原版开关，默认开启；不同页面按原脚本已有流程执行。

配置自动保存，无需把密钥写进脚本源码。更改答案来源、API 地址或模型后刷新页面，使后续自动答题使用新配置。

## DeepSeek 默认配置

| 配置项 | 默认值 |
|---|---|
| API 地址 | `https://api.deepseek.com/chat/completions` |
| API Key | 留空，填写自己的密钥 |
| 模型名称 | `deepseek-flash` |
| 超时秒数 | `120`，可设置为 10—600 秒 |
| JSON 模式 | 开启 |
| DeepSeek 思考模式 | 关闭，可手动开启 |

默认接口与模型名称依据 2026-10-12 查阅的 [DeepSeek 官方文档](https://api-docs.deepseek.com/)配置。密钥可在 [DeepSeek 开放平台](https://platform.deepseek.com/)申请；模型名称应使用账号实际可调用的模型 ID。

「DeepSeek 思考模式」只会对域名 `api.deepseek.com` 添加 `thinking` 参数。开启后可能需要增加超时时间。接口调用费用与余额由服务商账户管理。

## 接入其他 AI 服务

支持 **OpenAI 兼容的 Chat Completions 接口**：

- 请求为 `POST`，密钥放在 `Authorization: Bearer ...` 请求头中。
- 请求体包含 `model`、`messages`、`stream: false`。
- 回答读取 `choices[0].message.content`。
- 地址可以填写完整的 `/chat/completions` 地址，也可以填写基础路径，例如 `https://服务商域名/v1`；脚本会补齐 `/chat/completions`。
- 模型填写服务商提供的模型 ID。
- 若服务商不接受 `response_format`，取消「JSON 模式」。脚本仍要求模型生成 JSON 答案。
- 「额外请求参数」支持 JSON 对象，例如 `{"temperature":0.2}`。不能覆盖 `model`、`messages`、`stream`。
- HTTPS 服务可使用自定义域名；本机 `localhost`、`127.0.0.1`、`[::1]` 也可使用 HTTP。本机接口可以留空 API Key。

兼容格式参考 [OpenAI Chat Completions 官方文档](https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create)。本版不包含 Anthropic、Gemini 等服务的原生协议适配；这些服务需要提供对应的 OpenAI 兼容入口才能使用。

## 自动答题如何工作

1. 沿用原脚本解析页面题干、题型和选项。
2. AI 对选择题返回从 1 开始的选项序号；脚本将序号映射成页面原始选项文本。也兼容部分模型返回 A/B/C、选项原文或多选字母组合的情况。
3. 判断题转换成明确的 `true` / `false`；填空题按顺序返回多个答案；简答题返回一段文本。
4. 将结果转换为原脚本的 `data.items[].answer.values` 格式，供超星、智慧树及视频弹题处理逻辑使用。
5. AI 表示不确定、返回空答案、选项无法匹配或答案格式错误时，面板显示原因，该题不自动填写。

「有可用答案的比例达到多少自动提交」统计的是获得可用答案的题目比例，**不是经过平台验证的实际正确率**。模型生成的答案可能出错。

目前发送的是题干和选项的文本表示，未实现图片上传或视觉识别；需要看图才能解答的题目仍需人工处理。实际可填写的题型和页面范围取决于原脚本的页面解析与填写支持。

## 密钥与配置

- AI API Key 保存在油猴脚本存储中，从课程网站 `localStorage` 的配置副本中移除。
- 输入框默认遮掩密钥；密钥不会放进题目提示词或请求 URL。
- 自定义的 API 地址会接收题目和对应的 API Key，请填写你打算使用的服务地址。
- 选择「原题库海」可继续使用原来的题库接口和密钥；AI 请求失败不会自动切换到题库海。
- 移除了原版的远程 `downloadURL` / `updateURL`，避免原版自动更新覆盖本地修改。其他运行依赖沿用原版。

## 常见问题

| 现象 | 处理方法 |
|---|---|
| 提示填写 API Key | 在「答题」输入密钥并刷新页面 |
| HTTP 401 | 检查是否为开放平台的有效 API Key |
| HTTP 402 | 检查 API 账户余额 |
| HTTP 404 | 检查 API 地址和模型 ID |
| HTTP 429 | 增加答题间隔，检查服务商额度 |
| HTTP 400，提到 `response_format` | 取消 JSON 模式后重试 |
| 请求超时 | 增加超时秒数，或关闭 DeepSeek 思考模式 |
| 网络失败 | 检查接口能否访问，以及油猴对目标域名的连接权限 |
| 回答不是 JSON 或选项不能匹配 | 查看面板错误，更换模型或人工处理该题 |
| 填完密钥仍未开始 | 保存配置后刷新课程页面，确保原版脚本已停用 |

## 本地验证结果

| 验证层次 | 结果 |
|---|---|
| 整份 JavaScript 语法检查 | 通过 |
| 接口适配、答案转换、配置迁移、错误处理与取消请求 | 63 项通过 |
| 真实 Vue 界面、超星/智慧树模拟 DOM 填写、视频弹题 | 17 项通过 |
| 整份脚本启动、原有依赖、闭合 Shadow DOM 样式及配置保存 | 8 项通过 |
| 原始文件完整性 | SHA-256 未变化 |

合计 **88 项本地检查通过**。请求使用模拟返回，课程页面使用离线 DOM；未使用真实 API Key 调用 DeepSeek，也未在真实课程、考试或账户中验证提交结果。
