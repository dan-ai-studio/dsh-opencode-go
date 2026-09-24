# @dan-ai-studio/dsh-opencode-go

[English](README.en.md)

功能：让你在 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 中完美使用 OpenCode Go 订阅模型，支持流式回复、工具调用和图片输入。

插件会自动添加 OpenCode Go 所需的会话请求头、读取网关模型目录，并显示订阅用量，无需给模型配置协议、模态、上下文长度、最大输出token

## 功能说明

- **会话请求头**：每次请求包含 Harness User-Agent 和 `x-opencode-session`。同一会话保持相同 ID，最佳缓存命中率。
- **流式与历史**：支持流式输出、工具调用及历史回放，协议请求由 pi-ai 执行。
- **图片输入**：支持目录中声明图片能力的模型。
- **模型容量覆盖**：可按模型覆盖上下文窗口和最大输出，空值继承在线目录。
- **逐模型开关**：每个模型独立控制会话中的显示状态，修改立即生效；普通模型默认开启，过时模型默认关闭，也可单独开启。
- **提示与缓存**：插件不增加隐藏系统提示；会话 ID 用于网关路由。

## 安装与使用

兼容清单：
 `0.1.5-rc.1`、`0.1.5-rc.2`、`0.1.6-alpha.1`、`0.1.6-alpha.2`、`0.1.7-alpha.1`、`0.1.7-alpha.2` 和 `0.1.7-rc.1`

### 在 DSH 中安装（推荐）

1. 打开 DSH 的 **插件** 页面，点击右上角 **添加插件**。
2. 输入 `@dan-ai-studio/dsh-opencode-go`，点击 **安装**。
3. 安装成功后，如果出现 **立即启用**，点击即可。

然后打开 **设置 → OpenCode Go**，填入 API Key 并保存，即可在会话中选择 OpenCode Go 模型。

若当前 DSH 没有「添加插件」入口，可使用下面的命令行方式。

### 命令行安装（备选）

```sh
dsh plugin --profile web add @dan-ai-studio/dsh-opencode-go
```

安装后启动或重启 `dsh web`，然后：

1. 打开 **设置 → OpenCode Go**。
2. 填入 OpenCode Go API Key 并保存。
3. 在会话的模型选择器中选择 OpenCode Go 模型。

> 首次在某个 profile 中安装时，pnpm 会要求先决定依赖的构建脚本（例如 `@google/genai`、`protobufjs`）。如果安装因 `ERR_PNPM_IGNORED_BUILDS` 中断，请把 profile 的 `pnpm-workspace.yaml` 中 `allowBuilds` 的占位值改为 `true` 或 `false`，然后重试安装。

### 从 GitHub Release 安装

需要固定版本、或 registry 不可用时，可直接安装 Release 中的预构建包：

```sh
dsh plugin --profile web add https://github.com/dan-ai-studio/dsh-opencode-go/releases/download/v0.1.14/dan-ai-studio-dsh-opencode-go-0.1.14.tgz
```

### 源码构建

克隆仓库后本地打包安装：

```sh
git clone https://github.com/dan-ai-studio/dsh-opencode-go
cd dsh-opencode-go
npm ci --legacy-peer-deps
npm pack
dsh plugin --profile web add ./dan-ai-studio-dsh-opencode-go-0.1.14.tgz
```

开发依赖包含多代 DSH 的真实测试包，安装时需要 `--legacy-peer-deps`。Headless 用户将 `web` 换成 `headless`。

直接把 Git 地址交给 `dsh plugin add`（例如 `https://github.com/dan-ai-studio/dsh-opencode-go`）会在本机安装完整的开发依赖树并由 pnpm 构建；该依赖树体积大且需要 legacy peer 解析，因此不作为推荐方式。

### 无头模式

安装到 Headless profile：

```sh
dsh plugin --profile headless add @dan-ai-studio/dsh-opencode-go
```

将以下内容保存为 `headless.patch.yml`，选择默认模型：

```yaml
- id: agent-default-model
  config:
    provider: opencode-go
    model: deepseek-v4.1-flash
```

在 Bash 或 Zsh 中读取 API Key，然后运行任务：

```sh
read -s OPENCODE_API_KEY
export OPENCODE_API_KEY
dsh --profile headless --patch ./headless.patch.yml "你好"
```

模型 ID 须在当前网关目录中可用。Web 和 Headless 使用各自的 profile，需要分别安装插件。

### 从旧包迁移

npm 上的 `dsh-opencode-go` 是其他账号维护的独立包，与本仓库无关；本仓库的发布名为 `@dan-ai-studio/dsh-opencode-go`。从旧包迁移：

```sh
dsh plugin --profile web remove dsh-opencode-go
dsh plugin --profile web add @dan-ai-studio/dsh-opencode-go
```

设置与 API Key 会保留：新旧版本使用相同的设置命名空间（`llm-opencode-go`），无需重新配置。Headless 用户将 `web` 换成 `headless`；两个 profile 都装过时分别迁移。

## 升级插件

更新到 npm 最新版本：

```sh
dsh plugin --profile web update @dan-ai-studio/dsh-opencode-go --latest
```

也可以直接安装指定版本（版本需支持当前 DSH，见顶部兼容清单）：

```sh
dsh plugin --profile web add @dan-ai-studio/dsh-opencode-go@<版本>
```

习惯 GitHub Release 的话，用新版本的 Release 地址重复安装同样可以（会替换已装版本）。完成后重启 `dsh web` 并刷新浏览器。Headless 用户将 `web` 换成 `headless`；如果两个 profile 都安装了插件，需要分别升级。

## 订阅用量显示

用量每分钟刷新。临时网络或服务错误会保留同一账号的上次数据，并标明刷新失败、更新时间和错误原因；可在用量弹层中立即重试。首次获取失败或鉴权失败时不显示旧额度。模型目录、模型配置和用量的 JSON 请求遇到短暂连接重置时会在原有超时范围内额外重试一次；这不能保证故障中的网络恢复可用。

![OpenCode Go usage display](image.png)

## 界面语言

插件跟随 Harness 的界面语言，提供中文和英文文案，不另存一套语言设置。未手动指定语言时，Web 版按浏览器语言列表匹配中英文（浏览器通常跟随系统）；支持系统语言桥接的桌面版由宿主提供系统语言。没有匹配的语言时使用英文。

在 Harness 中手动选择的语言优先，切换后插件即时更新，保留未保存的表单内容。模型名称和 ID 保留原名；用量日期和容量数字按当前界面语言格式化。自动语言在启动时识别：Web 版更改浏览器语言后需刷新页面，桌面版更改系统语言后需重启 Harness。

## 模型开关

在 **设置 → OpenCode Go** 中，每个模型右侧的开关决定它是否出现在会话模型选择器中。开关立即保存，无需再点“保存”；容量和 API Key 的修改仍需保存。普通模型默认开启，过时模型默认关闭。你可以单独开启任意过时模型，也可以关闭普通模型。新出现且未单独配置的模型同样按此规则处理。

手动配置时，在插件的 `config` 下添加 `modelVisibility`，使用真实模型 ID 替换示例中的占位符：

```yaml
modelVisibility:
  your-model-id: false
  your-deprecated-model-id: true
```

只有列出的 ID 被显式覆盖。开关只影响模型选择器；已有会话仍可调用网关提供的隐藏模型，设置页也保留完整模型列表。

旧配置中的 `showDeprecatedModels` 和 `visibleModelIds` 不再控制显示状态。请使用逐模型开关或 `modelVisibility`；旧字段可以保留，不会妨碍插件加载。

## 常见问题

### 安装或重装时报 `ERR_PNPM_MISSING_TARBALL_INTEGRITY`

这是 pnpm 校验和与 dsh 安装流程的边界情况，不是包损坏：pnpm 要求远程 tarball 依赖在 lockfile 中带有校验和，但当本机缓存里已有同一份包时，解析不会重新下载，也就不会补写校验和；随后 dsh 的安装校验会拒绝它。从本地包切换到远程包、或重装到另一个 tarball 来源时可能出现。

可用以下任一方式处理：

1. **改用 registry 安装（推荐）**：`dsh plugin --profile web add @dan-ai-studio/dsh-opencode-go` 走 registry 解析，不会遇到该问题。
2. **下载后按本地路径安装**：先把 tarball 下载到本地（浏览器或 `curl -L -O`），再用绝对路径安装：

   ```sh
   dsh plugin --profile web add /absolute/path/to/dan-ai-studio-dsh-opencode-go-0.1.14.tgz
   ```

3. **在全新的 profile 中安装**：新 profile 没有该包的缓存，首次解析会正常下载并写入校验和。

直接删除 lockfile 重装或 `pnpm install --fix-lockfile` 不能解决（缓存仍在）。

### 安装日志出现 `missing peer` 警告

`pnpm peers check` 或安装日志中的 `missing peer @deepseek-ai/cordis`、`missing peer @deepseek-ai/dsh-*` 是预期现象：这些 DSH 服务包由 Harness 运行时按自身版本提供，不从 profile 的 `node_modules` 解析，profile 也有意关闭了 `autoInstallPeers`。不需要为此开启 `autoInstallPeers` 或手动安装这些包。

### 提示 `opencode-go` 路由已被占用

同一 profile 中只能有一个适配器提供 `opencode-go` 路由。如果已经通过其他插件或通用 pi-ai 配置接入 OpenCode Go，请先停用那一项配置。其他提供方可以继续使用。

### 没有出现预期的模型

先在 **设置 → OpenCode Go** 检查该模型的开关是否开启。过时模型默认关闭，单独打开它的开关后即可出现在会话模型选择器中，无需开启其他全局选项。

先确认插件已启用且 API Key 已配置，再刷新设置页中的模型列表。设置页读取或刷新列表时会请求网关 `/models`，并同步 [models.dev 的 OpenCode Go 配置](https://models.dev/api.json)。会话模型选择器遵守目录缓存时长，切换模型开关不会强制重新请求网关。模型的协议、上下文长度、输出上限和图片能力来自在线配置，新模型无需等待本插件或 pi-ai 发布新版本。

网关和在线配置已收录、且使用 Anthropic Messages、OpenAI Chat Completions 或 OpenAI Responses 协议的新模型，在下次设置页刷新或目录缓存到期后即可被发现。设置页刷新会绕过已有缓存，并通知已打开的会话模型选择器使用同一份新目录；直接请求尚未缓存的新模型也会立即重新同步。设置页显示完整模型列表。

网关已公布 ID 但尚无有效协议/能力配置的模型会在设置页标注“配置缺失”，开关保持关闭且不可操作，不进入对话模型选择器，避免一个未配置的模型阻断整个列表；直接调用时会说明原因。配置补齐后，刷新列表即可使用。仅凭模型 ID 无法可靠推断调用方式。上游新增全新协议或协议特例时，仍可能需要适配。

模型具有推理能力但没有可调节的推理档位时（如 `union-alpha`），仍可正常选择和使用，只是不显示推理强度选项。

提供可调节档位的模型，如果其传输协议在未选择档位时会显式关闭思考（`deepseek`、`zai`、`qwen`、`qwen-chat-template`），还会声明一个默认档位（模型支持 `high` 时用 `high`，否则用它提供的最高档位）。用户未选择档位时 DSH 使用该默认值，因此留空也会带上推理档位，而不是关闭思考。由服务商自行决定的协议不声明默认值，行为不变；显式选择的档位始终优先。

在线配置暂时不可达时，优先复用本次运行中成功获取的配置，以 pi-ai 内置配置作为备用。网关目录刷新失败时，已有请求和设置页均保留上次成功获取的列表；设置页同时显示警告和失败原因，说明当前展示的是缓存结果。首次读取就失败且没有缓存时，只显示错误，不会凭空生成模型列表。`refreshMinutes` 控制已有模型请求和会话选择器的目录缓存时长，不阻止设置页主动刷新或未知模型的即时发现。

## 卸载

在 DSH 的 **插件** 页面找到 `@dan-ai-studio/dsh-opencode-go`，点击 **卸载** 并确认，然后重启 `dsh web`（或 Desktop）并刷新页面。没有热更新能力的 profile（例如 Headless）请使用命令行：

```sh
dsh plugin --profile web remove @dan-ai-studio/dsh-opencode-go
# 或
dsh plugin --profile headless remove @dan-ai-studio/dsh-opencode-go
```

说明：

- 卸载会移除插件、`opencode-go` 路由及其模型；**API Key 保留在 profile 的凭证存储中**，不会随插件删除。需要一并清除时，请在卸载前于设置页或通过 DSH 的凭证工具处理。
- Web 和 Headless 使用各自的 profile；需要全部卸载时请分别操作。
- 重启应用后，插件页面不再显示该项。

## 反馈

遇到 bug 或有功能建议请提 issue。

## 许可证

[MIT](LICENSE)
