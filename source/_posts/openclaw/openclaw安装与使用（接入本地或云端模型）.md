
---
title: openclaw安装与使用（接入本地或云端模型）
tag: [openclaw]
date: 2026-04-20
archive: false
categories:
- AI
- openclaw
---

# openclaw

## 1、OpenClaw安装

*比做：OpenClaw安装前置要求：*

| 组件            | 要求                       | 验证命令                                                     |
| --------------- | -------------------------- | ------------------------------------------------------------ |
| Node.js         | 需安装npm和node            | npm -v / node -v                                             |
| Git for Windows | 必须安装，用于执行bash脚本 | where.exe git                                                |
| PowerShell      | Windows自带                | -                                                            |
| Git Bash        | npm需要构建bash环境        | 需配置路径npm config set script-shell（指定你的路径） "D:\Program Files\Git\bin\bash.exe" |

### 一、源码安装步骤：

#### 1.从github上下载源代码

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

*如果clone报错：连接超时，进行这步操作。*

1. 首先打开你的翻墙软件VPN。若是没有，推荐一款我在用的：https://sakura-cat1.com/#/register?code=nX5mrgn9
2. 强制git使用系统代理。通常代理软件的 HTTP 监听端口是 `7890` 或 `10809`，请根据你的软件实际端口修改下面的命令：用我推荐的软件是7897

```bash
# 设置 HTTP 代理
git config --global http.proxy 127.0.0.1:7897

# 设置 HTTPS 代理
git config --global https.proxy 127.0.0.1:7897
```

*取消代理设置（如果不需要了）：*

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

#### 2.安装依赖并构建

```bash
# 安装依赖（约 1000+ 包，可能需要几分钟）
npm install
# 构建项目（会生成 dist 目录）
npm run build
# 全局链接，使命令行可用
npm link
# 验证安装
openclaw --version
# 应显示：2026.3.3
```

npm 安装失败执行以下步骤，另开一个管理员powershell

*使用pnpm安装，后续npm 相关命令需全部替换为pnpm 命令。*

```bash
# npm 安装失败执行以下步骤
npm install -g pnpm
# 关闭所有命令行
# 清理旧文件：删除 node_modules 目录，删除 package-lock.json 文件
# 进入项目目录
cd D:\AI\openclaw
# 安装依赖
pnpm install
# 重新构建
pnpm build
# 或者：pnpm run build
```

- **npm**：每次运行脚本时，都会重新生成一个临时的环境变量路径，稍微慢一点点。
- **pnpm**：利用硬链接（Hard Links）和全局存储，执行速度非常快，且更节省磁盘空间。

### 二、启动配置向导

```bash
openclaw onboard --install-daemon
```

#### 配置选项选择

1. 我明白默认情况下这是个人使用的，共享/多用户需要使用锁屏保护。是否继续？

   **设置模式**

   快速开始

2. **快速开始**

   - 网关端口：18789
   - 网关绑定：回环地址 (127.0.0.1)
   - 网关认证：令牌（默认）
   - Tailscale 暴露：关闭
   - 直连聊天频道。

3. **模型/认证提供商**

   Anthropic (Claude CLI + 设置令牌 + API 密钥)

   BytePlus (火山引擎)

4. **我选择千问模型**

   - standard API Key for China

5. **输入百炼api key**

6. **设置默认模型**

7. **Channel status** **频道状态**:展示了支持多种聊天平台和协议的软件

8. **How channels work**频道如何接入（我先跳过了，后续可设置）

9. **Search provider**选择聊天软件：再次跳过Skip for now (Configure later with openclaw configure --section web)

10. **Skills status**现有的Skill状态

11. **Configure skills now? (recommended)**现在开始设置skill。可选或跳过，后面用到那个装那个。

12. **Enable hooks?**是否要启用**“钩子”**：我选了boot-md,系统启动时自动加载 Markdown 文件或配置.

13. 全布设置完毕后，第一次会自动启动。

### 三、后续启动openclaw

首先启动openclaw服务

```bash
# 官方守护进程模式
openclaw daemon start
# 查看运行状态
openclaw status
# 查看日志
openclaw logs
# 停止服务
openclaw daemon stop
```

稍等一会儿，会出现一个新弹窗提示listening on ws://127.0.0.1:18789。然后浏览器打开127.0.0.1:18789即可进入。新弹窗不可关闭，关闭后服务终止。

*若提示需要通过openclaw dashboard启动，则在原有窗口输入：*

```bash
openclaw dashboard
```

## 2、与其他软件集成配置

选择接入聊天软件 

```bash
openclaw channels add
```

重新配置openclaw浏览器

```bash
openclaw configure --section web
```

## 3、连接远程API Key

### 方式一：使用命令行配置（推荐）

这种方式通过交互式引导完成，简单快捷，适合新手。

1. **启动配置向导**
   在 PowerShell 或终端中执行以下命令，进入交互式问答模式：

   ```bash
   openclaw onboard
   ```

2. **按提示操作**
   根据向导的提示，依次选择你的模型提供商（如阿里云百炼）、输入 API Key 和对应的 Base URL，并选择默认模型即可。

或者，你也可以使用更直接的命令来设置，例如配置通义千问模型：

```bash
# 1. 设置主模型
openclaw config set agents.defaults.model.primary "dashscope-api/qwen3.6-plus"

# 2. 配置你的 API Key (将 sk-xxx 替换为你的真实密钥)
openclaw config set providers.dashscope-api.apikey "sk-你的千问API Key"

# 3. 配置 Base URL (国内使用)
openclaw config set providers.dashscope-api.baseurl "https://dashscope.aliyuncs.com/compatible-mode/v1"

# 4. 重启服务使配置生效
openclaw gateway restart
```

配置完成后，可以通过 `openclaw chat "你好"` 来测试是否成功。

### 方式二：手动修改配置文件

这种方式更灵活，适合需要精细调整参数的技术用户。

1. **找到配置文件**

   - **Windows:** `C:\Users\你的用户名\.openclaw\openclaw.json`
   - **macOS / Linux:** `~/.openclaw/openclaw.json`

2. **编辑配置文件**
   用文本编辑器打开 `openclaw.json`，找到 `model` 或 `models` 相关的配置部分，并根据你使用的模型服务商进行修改。

   **示例：配置阿里云百炼 (DashScope)**

   ```
   {
     "models": {
       "mode": "merge",
       "providers": {
         "dashscope-api": {
           "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
           "apiKey": "sk-你的千问API Key",
           "models": [
             {
               "id": "qwen3.6-plus",
               "name": "通义千问Qwen3.6-Plus",
               "contextWindow": 1000000,
               "maxTokens": 65536,
               "temperature": 0.7
             }
           ]
         }
       }
     },
     "agents": {
       "defaults": {
         "model": {
           "primary": "dashscope-api/qwen3.6-plus"
         }
       }
     }
   }
   ```

   请将 `apiKey` 的值替换为你自己的 API Key。

3. **重启服务**
   保存文件后，在终端中执行以下命令重启 OpenClaw 服务，使配置生效：

   ```bash
   openclaw gateway restart
   ```

### 进阶：使用环境变量（更安全）

为了避免将 API Key 直接明文写在配置文件中，推荐使用环境变量。

1. **设置环境变量**

   - **Windows (PowerShell):**

     ```powershell
     # 设置永久环境变量
     [Environment]::SetEnvironmentVariable("DASHSCOPE_API_KEY", "sk-你的千问API Key", "User")
     ```

   - **macOS / Linux (Zsh/Bash):**

     bash

     

     ```powershell
     # 在 ~/.zshrc 或 ~/.bashrc 文件中添加
     export DASHSCOPE_API_KEY="sk-你的千问API Key"
     # 然后执行 source ~/.zshrc 使其生效
     ```

2. **在配置文件中引用**
   修改 `openclaw.json`，将 `apiKey` 的值改为环境变量的引用格式：

   ```bash
   {
     "model": {
       "provider": "dashscope",
       "apiKey": "${DASHSCOPE_API_KEY}",
       "baseUrl": "https://dashscope.aliyuncs.com/api/v1"
     }
   }
   ```

   重启服务后，OpenClaw 会自动读取你设置的环境变量来获取 API Key。

## 4、接入本地部署模型

### 方式一：命令行配置三步曲（没跑通）

假设你已经用 `llama.cpp` 在本地启动了模型服务（默认端口 8080），并且模型名称为 `my-local-model`。

请在 PowerShell 或终端中依次执行以下三条命令：

#### 1. 配置 API 地址

告诉 OpenClaw 去哪里找你的本地模型服务。

```bash
# 主模型Qwen3.5-9B-Q4_K_M.gguf  视觉模型mmproj-Qwen3.5-9B-BF16.gguf

openclaw config set models.providers.ollama.baseUrl "http://localhost:8080/v1"
openclaw config set models.providers.ollama.models "[{\"id\": \"my-local-model\", \"name\": \"My Local Llama\"}]"
```

*注意：虽然这里用的是 `ollama` 作为提供商名称，但这只是为了复用配置结构。关键是 `baseUrl` 要指向你 `llama.cpp` 的地址。*

#### 2. 配置 API Key

`llama.cpp` 默认不需要 API Key，但 OpenClaw 要求填写，你可以随便填一个占位符。

```bash
openclaw config set models.providers.ollama.apiKey "not-needed"
```

#### 3. 设置默认模型

指定 OpenClaw 默认使用哪个模型。这里的 `my-local-model` 需要和你启动 `llama-server` 时定义的模型 ID 一致。

```bash
openclaw config set agents.defaults.model.primary "ollama/my-local-model"
```

*注意：这里的前缀 `ollama/` 对应了第一步配置中的 `models.providers.ollama`。*

#### 验证与生效

1. **验证配置**：你可以使用以下命令查看配置是否写入成功：

   ```bash
   openclaw config get models.providers.ollama
   ```

   如果返回的信息中包含你刚才设置的 `baseUrl` 和 `apiKey`，说明配置成功。

2. **重启服务**：配置修改后，必须重启 OpenClaw 才能生效。

   ```bash
   openclaw gateway restart
   ```

完成以上步骤后，你的 OpenClaw 就已经通过命令行成功接入本地模型了。

### 方式二：手动修改配置文件（已跑通）

1. 找到配置文件（通常在 `C:\Users\你的用户名\.openclaw\openclaw.json` 或项目根目录）。

2. *注意：建议复制出一个副本备份，再修改*

3. 在 `providers` 部分添加一个自定义配置（或者直接修改现有的），并在 `agents` 中指定使用它。

   **修改后的 JSON 配置示例：**

```json
{
  "agents": {
    "defaults": {
      "workspace": "C:\\Users\\13789\\.openclaw\\workspace",
      "models": {
        "local-llama/Qwen3.5-9B-Q4_K_M": {
          "alias": "Local Qwen 9B"
        }
      },
      "model": {
        "primary": "local-llama/Qwen3.5-9B-Q4_K_M"
        // 注意：这里已经移除了 fallbacks 数组
      }
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "3965a5f9cf5efc7d9a343c4b1869d9e49e2961f5454253d3"
    },
    "port": 18789,
    "bind": "loopback"
    // 为了简洁，省略了 tailscale 和 nodes 的默认配置，保留它们也没问题
  },
  "tools": {
    "profile": "coding",
    "web": {
      "search": {
        "provider": "duckduckgo",
        "enabled": true
      }
    }
  },
  "models": {
    "mode": "merge",
    "providers": {
      "local-llama": {
        "baseUrl": "http://localhost:8080/v1",
        "apiKey": "sk-no-key-needed",
        "api": "openai-completions",
        "models": [
          {
            "id": "Qwen3.5-9B-Q4_K_M",
            "name": "Qwen3.5-9B-Q4_K_M",
            "input": ["text", "image"],
            "contextWindow": 32768,
            "maxTokens": 32768
          }
        ]
      }
    }
  },
  "auth": {
    "profiles": {
      "local-llama:default": {
        "provider": "local-llama",
        "mode": "api_key"
      }
    }
  }
}
```

**关键参数解释：**

- `baseUrl`: 填 `http://localhost:8080/v1`。注意末尾的 `/v1` 是必须的，因为 `llama.cpp` 模拟的是 OpenAI 的 v1 接口。
- `apiKey`: `llama.cpp` 默认不需要 Key，随便填一个（如 `not-needed`）即可。
- `id`: 这里填 `my-local-model`，OpenClaw 会通过这个名字来调用它。

| 你的配置 (baseUrl) | OpenClaw 自动拼接的路径 | 最终访问的地址                              | 结果                  |
| ------------------ | ----------------------- | ------------------------------------------- | --------------------- |
| 不带 /v1           | `/v1/chat/completions`  | `http://localhost:8080/v1/chat/completions` | 404 报错 (找不到路)   |
| 带 /v1             | `/chat/completions`     | `http://localhost:8080/v1/chat/completions` | 成功 (找到了兼容接口) |

**重启并测试：**

1. 保存 `openclaw.json` 文件。
2. 重启 OpenClaw 服务：

```bash
openclaw gateway restart
```

### 常见问题排查

- **报错 "Connection refused" 或 "ECONNREFUSED"**：
  - 说明 OpenClaw 连不上 `llama.cpp`。请检查 `llama-server` 是否正在运行。
  - 检查端口号是否一致（配置文件里是 8080，启动命令里也是 8080）。
- **报错 "404 Not Found"**：
  - 通常是因为 `baseUrl` 没写对。一定要确保是 `http://localhost:8080/v1`，不要漏掉 `/v1`。
- **响应很慢**：
  - 这是本地部署的正常现象，取决于你的 CPU/GPU 性能。如果太慢，可以尝试换更小的模型（如 Qwen-3B 或 Q4_K_S 量化版本）。

现在，你的 OpenClaw 就已经完全由你自己的电脑驱动了，数据完全本地化！
