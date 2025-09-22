![Odoo 社区协会](https://odoo-community.org/readme-banner-image)

# AI OCA 桥接模块

| 标签 | 说明 |
|------|------|
| ![Beta 版本](https://img.shields.io/badge/maturity-Beta-yellow.png) | 开发阶段：Beta |
| ![AGPL-3 许可证](https://img.shields.io/badge/license-AGPL--3-blue.png) | 许可证：AGPL-3 |
| ![GitHub](https://img.shields.io/badge/github-OCA%2Fai-lightgray.png?logo=github) | 仓库：OCA/ai |
| ![Weblate](https://img.shields.io/badge/weblate-Translate%20me-F47D42.png) | 在 Weblate 上翻译 |
| ![Runboat](https://img.shields.io/badge/runboat-Try%20me-875A7B.png) | 在 Runboat 上试用 |

## 模块简介

此模块用于在 Odoo 和其他 AI 系统（如 n8n）之间创建桥接。

## 目录

- [使用场景](#使用场景)
- [配置](#配置)
- [使用方法](#使用方法)
- [已知问题 / 路线图](#已知问题--路线图)
- [Bug 跟踪](#bug-跟踪)
- [贡献者](#贡献者)

## 使用场景

目前，Odoo 与 AI 集成有两种不同的方法：

1. 在 Odoo 内部完成所有事情。
2. 使用其他工具并将 Odoo 与这些工具集成。

个人认为，由于以下原因，使用第二种方法会更好：

- Odoo 服务器旨在作为交易系统，而 AI 系统需要其他特性
- 一切变化太快，我不确定 Odoo 能否在这个领域跟上步伐
- 有一些开源工具可以完美地填补这一空白，它们就是为此目的而创建的

无论如何，OCA 对所有人开放，我们不打算强制一种固执己见的方式。为此，我们提供了这个模块，可以用作与 AI 系统的桥接。

## 配置

作为管理员，访问 `AI Bridge\AI Bridge`。

创建一个新的桥接。定义名称、模型、URL 和配置。

为了改进 AI 配置的视图，请使用组和域设置更好的过滤器。

### 配置实例

以下是不同复杂度的配置示例，从简单到复杂排列，帮助您逐步了解 AI 桥接的功能：

### 简单示例：客户支持消息生成器

这是一个适合初学者快速上手的基础配置示例：

1. **基本信息设置**
   - **名称**: 客户支持助手
   - **模型**: res.partner
   - **URL**: https://n8n.yourcompany.com/webhook/odoo-customer-support
   - **组**: base.group_user (所有用户可见)
   - **域**: [] (适用于所有合作伙伴记录)

2. **简化的载荷配置 JSON**
   ```json
   {
     "record_fields": ["name", "email", "phone", "street", "city"],
     "async": true,
     "result_processing": "message_post"
   }
   ```

3. **简化的 n8n 流程**
   
   n8n 可以执行以下简单流程：
   - 接收 Odoo 发送的合作伙伴基本信息
   - 调用 OpenAI API 生成标准化的客户问候语
   - 将结果发送回 Odoo

   n8n 响应示例：
   ```json
   {
     "result": {
       "body": "Dear " + $json["record"]["name"] + ",\n\nThank you for your interest in our services. This is an automated message from our system. Our team will contact you shortly.\n\nBest regards,\nCustomer Support Team",
       "subject": "Automated Greeting"
     }
   }
   ```

   这个简单示例展示了如何快速设置一个AI桥接，无需复杂配置即可实现基础的AI功能集成。

### 进阶示例：智能文档分析器

以下是一个更完整的配置示例，展示如何设置一个与 n8n 集成的 AI 桥接进行更复杂的文档分析：

1. **基本信息设置**
   - **名称**: 智能文档分析器
   - **模型**: account.invoice.analyzer
   - **URL**: https://n8n.yourcompany.com/webhook/odoo-ai-bridge
   - **组**: sales_team.group_sale_salesman
   - **域**: [('type', '=', 'out_invoice')] (仅适用于客户发票类型的记录)

2. **载荷配置 JSON 示例**
   ```json
   {
     "record_fields": ["name", "amount_total", "partner_id", "invoice_line_ids/name", "invoice_line_ids/price_unit", "invoice_line_ids/quantity"],
     "async": true,
     "result_processing": "message_post",
     "result_config": {
       "message_type": "comment",
       "subtype_xmlid": "mail.mt_comment"
     },
     "context": {
       "lang": "zh_CN"
     }
   }
   ```

3. **n8n 流程示例**
   
   当 Odoo 发送请求到 n8n webhook 时，n8n 可以执行以下流程：
   - 接收 Odoo 发送的数据（包含 account.invoice 表的相关字段）
   - 调用 OpenAI API 分析发票内容
   - 生成分析报告
   - 将结果发送回 Odoo 的 `_response_url`

   n8n 响应示例：
   ```json
   {
     "result": {
       "body": "According to analysis, this invoice amount is 10,000 yuan, customer is ABC Company. Suggest checking if the pricing of line item 3 complies with company standards.",
       "subject": "AI Invoice Analysis Report"
     }
   }
   ```

   这样，AI 分析的结果就会作为评论发布到原始 account.invoice 记录的 chatter 中。

### 自定义集成示例：不使用n8n的文档分类器

以下是一个更简单的配置示例，适合初学者快速上手：

1. **基本信息设置**
   - **名称**: 客户支持助手
   - **模型**: res.partner
   - **URL**: https://n8n.yourcompany.com/webhook/odoo-customer-support
   - **组**: base.group_user (所有用户可见)
   - **域**: [] (适用于所有合作伙伴记录)

2. **简化的载荷配置 JSON**
   ```json
   {
     "record_fields": ["name", "email", "phone", "street", "city"],
     "async": true,
     "result_processing": "message_post"
   }
   ```

3. **简化的 n8n 流程**
   
   n8n 可以执行以下简单流程：
   - 接收 Odoo 发送的合作伙伴基本信息
   - 调用 OpenAI API 生成标准化的客户问候语
   - 将结果发送回 Odoo

   n8n 响应示例：
   ```json
   {
     "result": {
       "body": "Dear " + $json["record"]["name"] + ",\n\nThank you for your interest in our services. This is an automated message from our system. Our team will contact you shortly.\n\nBest regards,\nCustomer Support Team",
       "subject": "Automated Greeting"
     }
   }
   ```

   这个简单示例展示了如何快速设置一个AI桥接，无需复杂配置即可实现基础的AI功能集成。

### 不使用n8n的示例：文档分类器

以下是一个不依赖n8n的示例，直接与自定义的AI服务集成：

1. **基本信息设置**
   - **名称**: 文档自动分类器
   - **模型**: document.page.classifier
   - **URL**: https://ai.yourcompany.com/api/classify-document
   - **组**: base.group_system
   - **域**: [] (适用于所有文档页面)

2. **载荷配置 JSON 示例**
   ```json
   {
     "record_fields": ["name", "content", "parent_id/name", "menu_id/name"],
     "async": true,
     "result_processing": "message_post",
     "auth_type": "bearer",
     "auth_params": {
       "token": "your-api-token-here"
     }
   }
   ```

3. **直接AI服务集成说明**

   这个示例展示了如何直接连接到您自己的AI服务，而不经过n8n：
   - Odoo会将文档页面的相关字段数据直接发送到您的AI服务URL
   - 请求将包含Bearer令牌进行身份验证（也可以根据您的需求使用其他认证方式）
   - AI服务接收数据后，分析文档内容并进行分类
   - 分析结果通过`_response_url`发送回Odoo

4. **自定义AI服务响应示例**
   
   您的AI服务应返回如下格式的JSON响应：
   ```json
   {
     "result": {
       "body": "Document '" + data["record"]["name"] + "' has been classified as: Technical Documentation.\n\nKeywords identified: API, integration, configuration\n\nSuggested improvements: Add more examples for better understanding",
       "subject": "AI Document Classification Result"
     }
   }
   ```

   这样，AI分析的结果就会作为评论发布到原始文档页面的chatter中。

   此示例的优势是直接集成，减少了中间环节，适合已有现成AI服务或希望构建更定制化解决方案的用户。

### 载荷配置

在外部系统上，您将收到一个 POST 载荷。包含的数据如下：

#### 通用

- \_odoo: 用于识别 Odoo 数据库的标准数据
- \_model: 相关对象的模型
- \_id: 相关对象的 ID
- \_response_url: 异步调用时用于调用响应的 URL

#### 记录载荷

添加一个名为 record 的新项目，其中包含所有字段。

### 异步和同步调用

新系统允许异步和同步调用。异步调用适用于不需要立即处理的任务。例如，审核发票并留下带有结果的评论。聊天消息也是如此。我们期望系统会给 AI 留出时间来回答，Odoo 的用户可以做其他事情。

同时，同步调用会冻结 Odoo 系统并等待答案。这在我们期望从 Odoo 用户那里获得一些反馈时是有意义的。例如，当我们打开一个操作时，这是有意义的。

在同步调用中，当 AI 系统在 webhook 上回答时，结果会被处理。另一方面，它会在同步调用中自动处理。

### 结果处理

对于系统的答案，我们希望对其进行处理。我们有以下选项：

#### 不处理

在这种情况下，结果将不执行任何操作。

#### 发布消息

我们将在系统的原始线程上发布消息。该线程由一个函数计算，因此可以在未来的模块中覆盖。它需要 `message_post` 函数的关键字参数。

#### 执行操作

它期望在用户界面上启动一个操作。这仅在同步调用中有意义。

它需要一个包含以下参数的操作项：

- action: 操作的 xmlid
- context: 传递给操作的上下文（非必需）
- res_id: 资源的 ID（非必需）

## 使用方法

使用 chatter 中的螺栓小部件执行不同的 AI 选项。

选项将根据配置进行筛选。

## 已知问题 / 路线图

- 定义使用和导入的示例
- 允许子字段。目前，只接受第一级字段。
- 当有大量数据时，信息弹出框无法正常工作。

## Bug 跟踪

Bug 在 [GitHub Issues](https://github.com/OCA/ai/issues) 上跟踪。如有问题，请先检查您的问题是否已被报告。如果您是第一个发现它的人，请通过提供详细且受欢迎的[反馈](https://github.com/OCA/ai/issues/new?body=module:%20ai_oca_bridge%0Aversion:%2018.0%0A%0A**Steps%20to%20reproduce**%0A-%20...%0A%0A**Current%20behavior**%0A%0A**Expected%20behavior**)来帮助我们解决它。

不要直接联系贡献者寻求支持或技术问题的帮助。

## 贡献者

### 作者

- Dixmit

### 贡献者

- [Dixmit](https://www.dixmit.com)
  - Enric Tobella

- [Sygel Technology](https://www.sygel.es)
  - Valentín Vinagre

- [Binhex](https://www.binhex.cloud/)
  - Adria Hortoneda

### 维护者

此模块由 OCA 维护。

![Odoo 社区协会](https://odoo-community.org/logo.png)

OCA（Odoo 社区协会）是一个非营利组织，其使命是支持 Odoo 功能的协作开发并促进其广泛使用。

此模块是 GitHub 上 [OCA/ai](https://github.com/OCA/ai/tree/18.0/ai_oca_bridge) 项目的一部分。

欢迎您的贡献。要了解如何贡献，请访问 https://odoo-community.org/page/Contribute。