# Claude 客户支持代理

一个先进的、完全可定制的客户支持聊天界面，由 Claude 驱动，并利用 Amazon Bedrock 知识库进行知识检索。
![预览](tutorial/preview.png)

## 主要功能

- 使用 Anthropic 的 Claude 模型的 AI 驱动聊天
- 用于上下文知识检索的 Amazon Bedrock 集成
- 实时思考和调试信息显示
- 知识库来源可视化
- 用户情绪检测和适当的代理重定向
- 使用 shadcn/ui 组件的高度可定制 UI

## 开始使用

1. 克隆此仓库
2. 安装依赖项：`npm install`
3. 设置环境变量（参见配置部分）
4. 运行开发服务器：`npm run dev`
5. 在浏览器中打开 [http://localhost:3000](http://localhost:3000)

## 配置

在根目录创建一个 `.env.local` 文件，包含以下变量：

```
ANTHROPIC_API_KEY=your_anthropic_api_key
BAWS_ACCESS_KEY_ID=your_aws_access_key
BAWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

注意：我们在 AWS 环境变量前面添加 'B' 的原因将在部署部分讨论。

## 如何获取密钥

### Claude API 密钥

1. 访问 [console.anthropic.com](https://console.anthropic.com/dashboard)
2. 注册或登录你的账户
3. 点击"Get API keys"
4. 复制密钥并粘贴到你的 `.env.local` 文件中

### AWS 访问密钥和秘密密钥

按照以下步骤获取你的 AWS 凭证：

1. 登录 AWS 管理控制台
2. 导航到 IAM（身份和访问管理）仪表板

3. 在左侧边栏中，点击"Users"

4. 点击"Create user"并按照提示创建新用户
   ![添加用户](tutorial/create-user.png)
5. 在"Set Permission"页面，选择"Attach policies directly"策略
   ![附加策略](tutorial/attach.png)
5. 在权限页面，使用"AmazonBedrockFullAccess"策略
   ![附加策略](tutorial/bedrock.png)
6. 检查并创建用户
7. 在摘要页面，点击"Create access key"。
8. 然后选择"Application running on an AWS compute service"。如果需要，添加描述，然后点击"Create"。
9. 现在你将看到显示的 Access Key ID 和 Secret Access Key。请注意，这些密钥仅在创建时显示一次，所以请确保安全保存它们。
   ![访问密钥](tutorial/access-keys.png)
8. 复制这些密钥并粘贴到你的 `.env.local` 文件中

注意：确保保管好你的密钥，永远不要公开分享它们。


## Amazon Bedrock RAG 集成

该项目使用 Amazon Bedrock 进行检索增强生成（RAG）。设置步骤：

1. 确保你有一个具有 Bedrock 访问权限的 AWS 账户。
2. 在你想要的 AWS 区域创建一个 Bedrock 知识库。
3. 在知识库中索引你的文档/来源。有关更多信息，请查看"如何创建自己的知识库"部分。
4. 在 `ChatArea.tsx` 中，用你的知识库 ID 和名称更新 `knowledgeBases` 数组：

```typescript
const knowledgeBases: KnowledgeBase[] = [
  { id: "your-knowledge-base-id", name: "Your KB Name" },
  // 根据需要添加更多知识库
];
```

应用程序将在对话期间使用这些知识库进行上下文检索。

### 如何创建自己的知识库

要创建你自己的知识库：

1. 进入你的 AWS 控制台并选择 Amazon Bedrock。
2. 在左侧菜单中，点击"More"下的"Knowledge base"。

3. 点击"Create knowledge base"。
   ![创建知识库](tutorial/create-knowledge-base.png)
4. 为你的知识库命名。你可以保留"Create a new service role"。
5. 为你的知识库选择一个来源。在这个例子中，我们将使用 Amazon S3 存储服务。
   ![选择来源](tutorial/choose-source.png)

   注意：如果你使用 S3 存储服务，你需要先创建一个存储桶来上传你的文件。或者，你也可以在创建知识库后上传文件。

6. 点击"Next"。
7. 为你的知识库选择一个位置。这可以是 S3 存储桶、文件夹，甚至是单个文档。
8. 点击"Next"。
9. 选择你偏好的嵌入模型。在这种情况下，我们将使用 Titan Text Embeddings 2。
10. 选择"Quick create a new vector store"。
11. 确认并创建你的知识库。
12. 完成后，从知识库概览中获取你的知识库 ID。


## 切换模型

该项目支持多个 Claude 模型。要在模型之间切换：

1. 在 `ChatArea.tsx` 中，`models` 数组定义了可用的模型：

```typescript
const models: Model[] = [
  { id: "claude-3-haiku-20240307", name: "Claude 3 Haiku" },
  { id: "claude-3-5-sonnet-20240620", name: "Claude 3.5 Sonnet" },
  // 根据需要添加更多模型
];
```

2. `selectedModel` 状态变量控制当前选择的模型：

```typescript
const [selectedModel, setSelectedModel] = useState("claude-3-haiku-20240307");
```

3. 要在 UI 中实现模型切换，使用一个更新 `selectedModel` 的下拉组件。


## 自定义

该项目利用 shadcn/ui 组件，提供高度的可定制性：

* 修改 `components/ui` 目录中的 UI 组件
* 调整 `app/globals.css` 中的主题
* 在各个组件文件中自定义布局和功能
* 通过编辑 `styles/themes.js` 文件修改主题颜色和样式：

```javascript
// styles/themes.js
export const themes = {
  neutral: {
    light: {
      // 中性主题的浅色模式颜色
    },
    dark: {
      // 中性主题的深色模式颜色
    }
  },
  // 在此处添加更多主题
};
```
你可以通过调整此文件中的颜色值来添加新主题或修改现有主题。

## 使用 AWS Amplify 部署

要使用 AWS Amplify 部署此应用程序，请按照以下步骤操作：

1. 进入你的 AWS 控制台并选择 Amplify。
2. 点击"Create new app"（稍后添加图片链接）。
3. 选择 GitHub（或你偏好的提供商）作为来源。
4. 选择此仓库。
5. 编辑 YAML 文件以包含：

   ```yaml
   version: 1
   frontend:
     phases:
       preBuild:
         commands:
           - npm ci --cache .npm --prefer-offline
       build:
         commands:
           - npm run build # Next.js 构建首先运行
           - echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY" >> .env
           - echo "KNOWLEDGE_BASE_ID=$KNOWLEDGE_BASE_ID" >> .env
           - echo "BAWS_ACCESS_KEY_ID=$BAWS_ACCESS_KEY_ID" >> .env
           - echo "BAWS_SECRET_ACCESS_KEY=$BAWS_SECRET_ACCESS_KEY" >> .env
     artifacts:
       baseDirectory: .next
       files:
         - "**/*"
     cache:
       paths:
         - .next/cache/**/*
         - .npm/**/*
   ```

6. 选择创建新的服务角色或使用现有的。有关更多信息，请参阅"服务角色"部分。
7. 点击"Advanced settings"并添加你的环境变量：

   ```
   ANTHROPIC_API_KEY=your_anthropic_api_key
   BAWS_ACCESS_KEY_ID=your_aws_access_key
   BAWS_SECRET_ACCESS_KEY=your_aws_secret_key
   ```
   我们在密钥前面添加 'B' 的原因是因为 AWS 不允许 Amplify 中的密钥以"AWS"开头。

8. 点击"Save and deploy"开始部署过程。

你的应用程序现在将使用 AWS Amplify 进行部署。


### 服务角色

一旦你的应用程序部署完成，如果你选择创建新的服务角色：

1. 进入你的部署页面
2. 选择你刚刚创建的部署
3. 点击"App settings"
4. 复制 Service role ARN
5. 进入 IAM 控制台并找到此角色
6. 将"AmazonBedrockFullAccess"策略附加到该角色

这确保你的 Amplify 应用程序具有与 Amazon Bedrock 交互所需的权限。

## 自定义部署和开发
该项目现在支持灵活的部署和开发配置，允许你根据需要包含或排除特定组件（左侧边栏、右侧边栏）。

### 配置
侧边栏的包含由 config.ts 文件控制，该文件使用环境变量设置配置：
```typescript
type Config = {
  includeLeftSidebar: boolean;
  includeRightSidebar: boolean;
};

const config: Config = {
  includeLeftSidebar: process.env.NEXT_PUBLIC_INCLUDE_LEFT_SIDEBAR === "true",
  includeRightSidebar: process.env.NEXT_PUBLIC_INCLUDE_RIGHT_SIDEBAR === "true",
};

export default config;
```

此配置使用两个环境变量：

- NEXT_PUBLIC_INCLUDE_LEFT_SIDEBAR：设置为 "true" 以包含左侧边栏
- NEXT_PUBLIC_INCLUDE_RIGHT_SIDEBAR：设置为 "true" 以包含右侧边栏

## NPM 脚本
package.json 包含几个用于不同配置的新脚本：

```bash
npm run dev: 运行带有两个侧边栏的完整应用（默认）
npm run build: 构建带有两个侧边栏的完整应用（默认）
npm run dev:full: 与 npm run dev 相同
npm run dev:left: 运行只有左侧边栏的应用
npm run dev:right: 运行只有右侧边栏的应用
npm run dev:chat: 运行只有聊天区域的应用（无侧边栏）
npm run build:full: 与 npm run build 相同
npm run build:left: 构建只有左侧边栏的应用
npm run build:right: 构建只有右侧边栏的应用
npm run build:chat: 构建只有聊天区域的应用（无侧边栏）
```

### 使用方法
要使用特定配置：

- 开发环境：运行所需的脚本（例如，npm run dev:left）
- 生产环境：使用所需的脚本构建（例如，npm run build:right）

这些脚本在运行或构建应用程序之前设置适当的环境变量，允许你轻松在不同配置之间切换。
这种灵活性允许你根据特定需求定制应用程序的布局，无论是用于测试、开发还是生产部署。

## 附录

该项目是一个原型，以"按原样"的基础提供。它不适用于生产用途，可能包含错误、缺陷或不一致之处。使用此原型即表示你承认并同意：
- 该软件以预发布、测试版或试用形式提供。
- 它可能不适合生产或关键任务环境。
- 开发者不对因使用该软件而导致的任何问题、数据丢失或损害负责。
- 不提供任何形式的明示或暗示的保证或担保。
- 对此原型的支持可能有限或不可用。
- 使用此原型的风险由你自行承担。我们鼓励你报告任何问题或提供反馈，以帮助改进未来版本。
