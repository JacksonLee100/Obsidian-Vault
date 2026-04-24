
如果你希望利用 **`free-claude-code`** 这个专门为 Claude Code 设计的开源代理（通常它在处理工具调用、文件读写协议上比通用的 LiteLLM 更贴合 Claude 的“脾气”），我们需要微调一下部署链路。

在你的 **ThinkPad P14s** 上，现在的链路是：

`Claude CLI` → `free-claude-code (代理)` → `LM Studio (推理)`

---

### 🚀 带 `free-claude-code` 的全流程部署命令

#### 第一步：准备推理后端 (LM Studio)

1. **加载 Qwen2.5-Coder-7B**。
    
2. **关键设置**：
    
    - **Context Length**: 设置为 **`32768`**。  
        上下文长度：设置为 32768。
        
    - **GPU Offload**: 设置为 **`10-15`**。
        
3. **Start Server**：确保端口是 **`1234`**，模式选 **`Anthropic-compatible`**。
    

#### 第二步：配置 free-claude-code 代理

1. **进入项目目录**：
    
    PowerShell
    
    ```
    cd C:\sw\Tst\Tst\FreeclaudeTst  # 你的项目路径
    ```
    
2. **修改 `.env` 配置文件**：
    
    用记事本打开 `.env`，确保内容如下（直接指向 LM Studio）：
    
    Ini, TOML
    
    ```
    # 指向本地推理引擎
    ANTHROPIC_BASE_URL=http://localhost:1234/v1
    
    # 这里的模型 ID 必须和 LM Studio 右侧显示的完全一致
    MODEL_OPUS="qwen2.5-coder-7b-instruct"
    MODEL_SONNET="qwen2.5-coder-7b-instruct"
    MODEL_HAIKU="qwen2.5-coder-1.5b-instruct"
    
    # 代理自身的认证 Token（随便起，比如 freecc）
    ANTHROPIC_AUTH_TOKEN=freecc
    ```
    
3. **安装依赖并启动代理**：
    
    PowerShell
    
    ```
    # 安装运行环境
    pip install "litellm[proxy]" uvicorn fastapi
    
    # 启动代理服务器，运行在 8082 端口
    uv run uvicorn server:app --host 0.0.0.0 --port 8082
    ```
    
    _（保持此窗口不要关闭，它是翻译官）_
    

#### 第三步：连接 Claude CLI

再开一个新终端，执行：

PowerShell

```
# 1. 确保没有 API_KEY 干扰，因为 free-claude-code 优先识别 AUTH_TOKEN
$env:ANTHROPIC_API_KEY=$null
$env:ANTHROPIC_AUTH_TOKEN="freecc"

# 2. 指向代理服务器（注意这里通常需要带 /v1，取决于代理脚本的写法）
$env:ANTHROPIC_BASE_URL="http://localhost:8082/v1"

# 3. 启动
claude --model qwen2.5-coder-7b-instruct
```

---

### 🛠️ 常见报错排坑

1. **404 Not Found (v1/v1/messages)**:
    
    如果启动后报错 404，说明路径重叠了。请尝试将 `ANTHROPIC_BASE_URL` 改为 `http://localhost:8082` (去掉末尾的 `/v1`)。
    
2. **Empty Response (HTTP 200)**:
    
    如果仍然出现这个，说明 `free-claude-code` 没能成功过滤 `thinking` 参数。
    
    - **解决**：在 `free-claude-code` 的源码里找到处理请求的地方，手动加上参数过滤。或者简单点，直接退回到我之前给你的 **LiteLLM** 方案，因为 LiteLLM 的 `--drop_params` 是目前处理这个问题的“工业级”方案。
        
3. **P14s 卡死**:
    
    如果运行 `uv run` 时 CPU 100% 且没有反应，可以尝试不用 `uv`，直接用 Python 运行：
    
    PowerShell
    
    ```
    python -m uvicorn server:app --host 0.0.0.0 --port 8082
    ```
    

---

### 📝 总结：为什么用这个代理？

使用 `free-claude-code` 的好处是它内部可能已经硬编码了一些针对 Claude Code 工具调用（比如读取你的本地 C 盘文件）的补丁，这能让它在执行“修改代码”这种任务时比纯 LiteLLM 更聪明一点。

**现在代理服务器跑起来了吗？如果启动了，输入 `create a file` 试试，这是检验成功的唯一标准！**