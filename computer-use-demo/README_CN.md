# Anthropic 计算机使用演示

> [!NOTE]
> 现已支持全新的 Claude 4 系列模型！最新的 Claude Opus 4.5 (claude-opus-4-5-20251101) 现已可用，同时还有 Claude Sonnet 4.5 (claude-sonnet-4-5-20250929)、Claude Sonnet 4 (claude-sonnet-4-20250514)、Claude Opus 4 (claude-opus-4-20250514) 和 Claude Haiku 4.5 (claude-haiku-4-5-20251001)。这些模型带来了下一代能力，使用更新的 str_replace_based_edit_tool 替代了之前的 str_replace_editor 工具。在最新版本中，undo_edit 命令已被移除，以提供更流畅的体验。

> [!CAUTION]
> 计算机使用是一项测试版功能。请注意，计算机使用带来的风险与标准 API 功能或聊天界面不同。当使用计算机使用功能与互联网交互时，这些风险会进一步增加。为了最小化风险，请考虑采取以下预防措施：
>
> 1. 使用具有最小权限的专用虚拟机或容器，以防止直接的系统攻击或意外操作。
> 2. 避免让模型访问敏感数据，如账户登录信息，以防止信息泄露。
> 3. 将互联网访问限制在允许的域名列表中，以减少接触恶意内容的风险。
> 4. 对于可能导致重大现实后果的决定，以及任何需要明确同意的任务（如接受 cookies、执行金融交易或同意服务条款），请让人类确认。
>
> 在某些情况下，Claude 会遵循内容中的指令，即使这与用户的指示相冲突。例如，网页上的指令或图像中包含的指令可能会覆盖用户指示或导致 Claude 出错。我们建议采取预防措施，将 Claude 与敏感数据和操作隔离，以避免与提示注入相关的风险。
>
> 最后，在你自己的产品中启用计算机使用功能之前，请告知最终用户相关风险并获得他们的同意。

本仓库帮助你开始在 Claude 上使用计算机使用功能，提供以下参考实现：

- 用于创建包含所有必要依赖项的 Docker 容器的构建文件
- 使用 Claude API、Bedrock 或 Vertex 访问 Claude Opus 4.5、Claude Sonnet 4.5、Claude Sonnet 4、Claude Opus 4、Claude Haiku 4.5、Claude 3.7 Sonnet 和 Claude 3.5 Sonnet 模型的计算机使用代理循环
- Anthropic 定义的计算机使用工具
- 用于与代理循环交互的 Streamlit 应用

请使用[此表单](https://forms.gle/BT1hpBrqDPDUrCqo7)提供关于模型响应质量、API 本身或文档质量的反馈 - 我们迫不及待地想听到你的意见！

> [!IMPORTANT]
> 本参考实现中使用的 Beta API 可能会发生变化。请参阅 [API 发布说明](https://docs.claude.com/en/release-notes/api) 获取最新信息。

> [!IMPORTANT]
> 这些组件之间的分离较弱：代理循环在 Claude 控制的容器中运行，一次只能由一个会话使用，如有必要，必须在会话之间重启或重置。

## 快速入门：运行 Docker 容器

### Claude API

> [!TIP]
> 你可以在 [Claude 控制台](https://console.anthropic.com/) 找到你的 API 密钥。

```bash
export ANTHROPIC_API_KEY=%your_api_key%
docker run \
    -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

容器运行后，请参阅下面的[访问演示应用](#访问演示应用)部分，了解如何连接到界面的说明。

### Bedrock

> [!TIP]
> 要在 Bedrock 上使用新的 Claude 3.7 Sonnet，你首先需要[请求模型访问权限](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access-modify.html)。

你需要传入具有适当权限的 AWS 凭证才能在 Bedrock 上使用 Claude。
你有几种 Bedrock 身份验证选项。有关更多详细信息和选项，请参阅 [boto3 文档](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html#environment-variables)。

#### 选项 1：（推荐）使用主机的 AWS 凭证文件和 AWS 配置文件

```bash
export AWS_PROFILE=<your_aws_profile>
docker run \
    -e API_PROVIDER=bedrock \
    -e AWS_PROFILE=$AWS_PROFILE \
    -e AWS_REGION=us-west-2 \
    -v $HOME/.aws:/home/computeruse/.aws \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

容器运行后，请参阅下面的[访问演示应用](#访问演示应用)部分，了解如何连接到界面的说明。

#### 选项 2：使用访问密钥和秘密密钥

```bash
export AWS_ACCESS_KEY_ID=%your_aws_access_key%
export AWS_SECRET_ACCESS_KEY=%your_aws_secret_access_key%
export AWS_SESSION_TOKEN=%your_aws_session_token%
docker run \
    -e API_PROVIDER=bedrock \
    -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
    -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
    -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
    -e AWS_REGION=us-west-2 \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

容器运行后，请参阅下面的[访问演示应用](#访问演示应用)部分，了解如何连接到界面的说明。

### Vertex

你需要传入具有适当权限的 Google Cloud 凭证才能在 Vertex 上使用 Claude。

```bash
docker build . -t computer-use-demo
gcloud auth application-default login
export VERTEX_REGION=%your_vertex_region%
export VERTEX_PROJECT_ID=%your_vertex_project_id%
docker run \
    -e API_PROVIDER=vertex \
    -e CLOUD_ML_REGION=$VERTEX_REGION \
    -e ANTHROPIC_VERTEX_PROJECT_ID=$VERTEX_PROJECT_ID \
    -v $HOME/.config/gcloud/application_default_credentials.json:/home/computeruse/.config/gcloud/application_default_credentials.json \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it computer-use-demo
```

容器运行后，请参阅下面的[访问演示应用](#访问演示应用)部分，了解如何连接到界面的说明。

此示例展示了如何使用 Google Cloud 应用默认凭证进行 Vertex 身份验证。
你也可以设置 `GOOGLE_APPLICATION_CREDENTIALS` 来使用任意凭证文件，有关更多详细信息，请参阅 [Google Cloud 身份验证文档](https://cloud.google.com/docs/authentication/application-default-credentials#GAC)。

### 访问演示应用

容器运行后，在浏览器中打开 [http://localhost:8080](http://localhost:8080) 以访问包含代理聊天和桌面视图的组合界面。

容器将设置（如 API 密钥和自定义系统提示）存储在 `~/.anthropic/` 中。挂载此目录以在容器运行之间保持这些设置。

其他访问点：

- 仅 Streamlit 界面：[http://localhost:8501](http://localhost:8501)
- 仅桌面视图：[http://localhost:6080/vnc.html](http://localhost:6080/vnc.html)
- 直接 VNC 连接：`vnc://localhost:5900`（用于 VNC 客户端）

## 屏幕尺寸

可以使用环境变量 `WIDTH` 和 `HEIGHT` 来设置屏幕尺寸。例如：

```bash
docker run \
    -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -e WIDTH=1920 \
    -e HEIGHT=1080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

我们不建议发送分辨率高于 [XGA/WXGA](https://en.wikipedia.org/wiki/Display_resolution_standards#XGA) 的截图，以避免与[图像调整大小](https://docs.claude.com/en/docs/build-with-claude/vision#evaluate-image-size)相关的问题。
依赖 API 中的图像调整大小行为会导致模型准确性降低和性能变慢，不如在工具中直接实现缩放。本项目中的 `computer` 工具实现展示了如何将图像和坐标从更高分辨率缩放到建议的分辨率。

在自己实现计算机使用时，我们建议使用 XGA 分辨率（1024x768）：

- 对于更高分辨率：将图像缩小到 XGA，让模型与此缩放版本交互，然后按比例将坐标映射回原始分辨率。
- 对于较低分辨率或较小设备（如移动设备）：在显示区域周围添加黑色填充，直到达到 1024x768。

## 开发

```bash
./setup.sh  # 配置 venv，安装开发依赖项，并安装 pre-commit 钩子
docker build . -t computer-use-demo:local  # 手动构建 docker 镜像（可选）
export ANTHROPIC_API_KEY=%your_api_key%
docker run \
    -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
    -v $(pwd)/computer_use_demo:/home/computeruse/computer_use_demo/ `# 挂载本地 python 模块用于开发` \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it computer-use-demo:local  # 也可以使用 ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

上面的 docker run 命令将仓库挂载到 docker 镜像内部，这样你可以从主机编辑文件。Streamlit 已经配置了自动重新加载功能。
