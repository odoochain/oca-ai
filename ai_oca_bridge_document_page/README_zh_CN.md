# AI OCA Bridge Document Page 中文操作说明

## 模块简介

AI OCA Bridge Document Page 模块允许用户将 OCA 的 Knowledge 应用程序用作 AI 代理的知识源，实现文档页面与外部 AI 系统的自动同步。该模块通过 AI 桥接机制，在文档页面创建、更新或删除时触发相应的操作，将文档内容同步到外部 AI 系统的知识库中。

## 功能说明

- 当创建新文档页面时，自动将文档内容发送到外部 AI 系统
- 当更新文档页面时，自动更新外部 AI 系统中的对应内容
- 当删除文档页面时，自动从外部 AI 系统中移除相应内容
- 支持 RAG（检索增强生成）能力的 AI 代理访问最新的文档知识

## 安装要求

安装此模块前，确保已安装以下依赖模块：
- ai_oca_bridge
- document_page

## 配置方法

使用此模块需要配置两个主要组件：
1. Odoo 端的桥接配置
2. 能够处理桥接请求的外部端点

为了使 AI 代理具备最新的 RAG 能力，您需要为每个活动的知识数据库创建至少**三个桥接**：

### 1. 创建文档页面创建桥接

此桥接用于将新建的文档页面添加到 AI 代理使用的外部数据库：

1. 以管理员身份登录 Odoo
2. 访问 `AI Bridge > AI Bridge` 菜单
3. 点击"创建"按钮
4. 填写以下信息：
   - **名称 (Name)**: 文档页面 AI 桥接 - 创建
   - **描述 (Description)**: 当创建新文档页面时触发此 AI 桥接
   - **模型 (Model)**: 选择 "Document Page" 模型
   - **使用场景 (Usage)**: 选择 "AI Thread Create"
   - **URL (URL)**: 输入你本地 Mirix 服务器的 API 端点 URL（如 `http://localhost:47283/ai/document/create`）
   - **认证类型 (Auth Type)**: 根据外部系统要求选择，示例中为 "none"
   - **载荷类型 (Payload Type)**: 根据端点配置选择，通常 "Record" 即可
   - **结果类型 (Result Type)**: 对于此场景，选择 "No processing"
   - **结果处理方式 (Result Management Or Result Kind)**: 选择 "Immediate"
   - **字段 (Fields)**: 添加至少外部端点期望的字段，如 content、display_name、draft_name 等
   - **过滤器 (Domain)**: 添加域以限制仅对特定文档触发桥接

### 2. 创建文档页面更新桥接

此桥接用于更新 AI 代理使用的外部数据库中的文档页面：

1. 同样在 `AI Bridge > AI Bridge` 菜单中创建新记录
2. 填写以下信息：
   - **名称**: 文档页面 AI 桥接 - 更新
   - **描述**: 当文档页面更新时触发此 AI 桥接
   - **模型**: 选择 "Document Page" 模型
   - **使用场景**: 选择 "AI Thread Write"
   - **URL**: 输入外部 AI 系统的 API 端点 URL（如 `http://localhost:47283/ai/document/update`）
   - **其他设置**: 与创建桥接类似，但确保配置适合更新操作

### 3. 创建文档页面删除桥接

此桥接用于当文档页面从 Odoo 中删除时，从外部数据库中移除相应内容：

1. 在 `AI Bridge > AI Bridge` 菜单中创建新记录
2. 填写以下信息：
   - **名称**: 文档页面 AI 桥接 - 删除
   - **描述**: 当文档页面删除时触发此 AI 桥接
   - **模型**: 选择 "Document Page" 模型
   - **使用场景**: 选择 "AI Thread Unlink"
   - **URL**: 输入外部 AI 系统的 API 端点 URL
   - **载荷类型**: 对于删除操作，通常选择 "None"
   - **其他设置**: 根据外部系统要求配置

## 使用指南

配置完成后，该模块的使用非常简单：

1. 创建、更新或删除符合桥接域条件的文档页面
2. 系统会自动触发相应的 AI 桥接
3. 您可以在 `AI Bridge > AI Bridge Executions` 菜单中查看桥接执行的状态和结果

## 测试方法

### 使用 Mirix 服务器测试

本模块已配置为与 Mirix AI 服务器协同工作。以下是完整的测试配置步骤：

#### 1. Mirix 服务器端点配置

Mirix 服务器提供以下 API 端点：

- **文档创建**: `POST http://localhost:47283/ai/document/create`
- **文档列表**: `GET http://localhost:47283/api/v1/document`

#### 2. 测试 Mirix 服务器连接

在配置 Odoo 桥接之前，建议先测试 Mirix 服务器是否正常运行：

**测试文档列表端点：**
```bash
curl -X GET http://localhost:47283/api/v1/document
```

**测试文档创建端点（Windows PowerShell）：**
```powershell
# 方法1：使用文件
@'
{
  "title": "测试文档",
  "content": "这是一个测试文档的内容",
  "document_type": "page",
  "tags": ["测试", "文档"],
  "metadata": {"author": "系统管理员"}
}
'@ | Out-File -FilePath "test_document.json" -Encoding UTF8

curl -X POST "http://localhost:47283/ai/document/create" `
  -H "Content-Type: application/json" `
  -d "@test_document.json"

# 方法2：使用 Invoke-RestMethod
$body = @{
    title = "测试文档"
    content = "这是一个测试文档的内容"
    document_type = "page"
    tags = @("测试", "文档")
    metadata = @{author = "系统管理员"}
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:47283/ai/document/create" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

#### 3. Odoo 桥接配置验证

配置完成后，验证桥接是否正常工作：

1. **创建测试文档**：
   - 在 Odoo 中创建一个新的文档页面
   - 填写必要字段（content、display_name、draft_name）
   - 保存文档

2. **检查执行记录**：
   - 访问 `AI Bridge > AI Bridge Executions` 菜单
   - 查看最近的执行记录
   - 确认状态为 "成功"

3. **验证外部系统**：
   - 使用文档列表 API 确认文档已同步
   - 检查 Mirix 服务器的响应数据

#### 4. 使用 n8n 工作流测试（可选）

为了测试外部端点的功能，您可以使用模块提供的示例 n8n 工作流：

1. 下载示例 n8n 工作流 JSON 文件：`static/description/RagCapabilitiesWithOdooKnowledge.json`
2. 将该工作流导入到您的 n8n 实例中
3. 根据您的需求更新工作流中的模型和数据库知识配置
4. 确保工作流中包含手动触发器以便进行测试
5. 创建、更新或删除文档页面，观察工作流的执行情况

**注意**：为了使测试正常工作，桥接配置中至少应包含以下字段：`content`、`display_name` 和 `draft_name`。

## 常见问题解决

1. **桥接执行失败**
   - 检查外部端点 URL 是否正确
   - 验证认证设置是否符合外部系统要求
   - 查看 AI Bridge Executions 中的错误日志获取详细信息

2. **文档没有同步到外部系统**
   - 确认文档符合桥接的域过滤条件
   - 检查 AI Bridge 配置中的字段设置是否正确
   - 验证外部系统是否正确接收并处理了请求

3. **RAG 功能不包含最新文档**
   - 确认所有三个桥接（创建、更新、删除）都已正确配置
   - 检查最近的文档变更是否成功触发了桥接执行
   - 验证外部 AI 系统的知识库是否已更新

## 演示数据

模块包含演示数据，展示了三个桥接的基本配置：
- Document Page AI Bridge - Create：当创建新文档页面时触发
- Document Page AI Bridge - Update：当更新文档页面时触发  
- Document Page AI Bridge - Delete：当删除文档页面时触发

这些演示数据可以帮助您理解如何正确配置桥接，但您需要根据实际环境修改 URL 和其他设置。

## 已知限制

- 此模块主要设计用于 RAG 能力，但也可以用于其他集成场景
- 外部 AI 系统需要能够处理来自 Odoo 的请求并正确更新其知识库
- 桥接执行的结果依赖于外部系统的响应和处理能力

## 贡献者

- Binhex
- Odoo Community Association (OCA)