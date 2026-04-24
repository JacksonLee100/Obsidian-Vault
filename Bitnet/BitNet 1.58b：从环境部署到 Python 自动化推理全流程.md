这份总结按照部署的**生命周期**进行编排，涵盖了从底层环境到高层逻辑的所有关键指令。

---

# 🚀 BitNet 1.58b：从环境部署到 Python 自动化推理全流程

## 1. 环境初始化 (WSL + Venv)

首先在 WSL 原生文件系统中建立隔离的工作空间，避免与系统级 Python 包冲突。

Bash

```
# 进入主目录并创建项目文件夹
mkdir -p ~/BitNet_Final && cd ~/BitNet_Final

# 创建虚拟环境 (venv)
python3 -m venv venv

# 激活虚拟环境 (每次打开终端都需要执行此步)
source venv/bin/activate

# 确认环境 (路径应指向 ~/BitNet_Final/venv/...)
which python
```

---

## 2. 引擎编译 (源码构建)

针对你的本地 CPU（支持 AVX512）构建优化的推理引擎。

Bash

```
# 克隆仓库
git clone --recursive https://github.com/microsoft/BitNet.git
cd BitNet

# 配置并编译
mkdir -p build && cd build
cmake ..
make -j$(nproc)  # 使用所有可用 CPU 核心编译
```

---

## 3. 模型搬运 (关键 IO 优化)

**注意**：必须将模型从 Windows 挂载盘（/mnt/c）移动到 Linux 内部，否则推理时会出现延迟导致输出为空。

Bash

```
# 创建内部模型仓库
mkdir -p ~/BitNet_Final/models

# 定位并拷贝模型 (使用通配符处理路径大小写敏感问题)
cp /mnt/c/sw/Tst/BitNet/models/Llama3-8B-1.58-*.gguf ~/BitNet_Final/models/

# 验证模型大小 (TQ2_0 约为 2.4G)
ls -lh ~/BitNet_Final/models/
```

---

## 4. 普通测试 (命令行直连)

在编写脚本前，先用命令行确认模型引擎是否能正常工作。

Bash

```
# 手动运行一次推理
~/BitNet_Final/build/bin/llama-cli \
  -m ~/BitNet_Final/models/Llama3-8B-1.58-100B-tokens-TQ2_0.gguf \
  -p "News: Apple released M4 chip. Category:" \
  -n 8 -t 6 --temp 0.0
```

---

## 5. Python 自动化测试 (.py 脚本)

编写 Python 包装器，利用 `subprocess` 捕获输出并进行业务逻辑清洗。

### 脚本注入

Bash

```
cat <<EOF > ~/BitNet_Final/server.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import subprocess
import os

app = FastAPI(title="BitNet 图纸数据提取中台")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

CLI_PATH = os.path.expanduser("~/BitNet_Final/build/bin/llama-cli")
MODEL_PATH = os.path.expanduser("~/BitNet_Final/models/Llama3-8B-1.58-100B-tokens-TQ2_0.gguf")

class DrawingInput(BaseModel):
    text: str

@app.post("/extract")
async def extract_info(item: DrawingInput):
    # 只提取纯文本，过滤掉像坐标一样的纯数字，减小模型负担
    clean_text = " ".join([w for w in item.text.split() if not any(c.isdigit() for c in w)])[:500]

    # 极简指令，给它一个具体的开始符号
    prompt = (
        f"Drawing text: {clean_text}\n"
        f"Instruction: Extract JSON format.\n"
        f"JSON: {{\"Company\":"
    )
    
    cmd = [
        CLI_PATH, "-m", MODEL_PATH, 
        "-p", prompt, 
        "-n", "128", 
        "--temp", "0.0",          # 强制关闭随机性，最稳定的输出
        "--repeat-penalty", "1.5", # 加大惩罚，严禁复读
        "--top-k", "1"            # 只选概率最高的一个词
    ]
    # ... 其余逻辑不变 ...
    
    try:
        # 运行 BitNet 推理
        result = subprocess.run(cmd, capture_output=True, text=True, encoding='utf-8')
        
        # 这里的 stdout 包含了 Prompt + 结果，我们只需要结果部分
        raw_output = result.stdout
        # 简单的后处理：取 "JSON Result:" 之后的文字
        if "JSON Result:" in raw_output:
            prediction = raw_output.split("JSON Result:")[-1].strip()
        else:
            prediction = raw_output.strip()

        return {"status": "success", "data": prediction}
    except Exception as e:
        return {"status": "error", "message": str(e)}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
EOF
```

### 运行脚本

Bash

```
python3 ~/BitNet_Final/final_classify.py
```

---

## 🏁 总结与避坑指南

1. **路径大小写**：Windows 是不区分大小写的，但 WSL (Linux) 非常严格。务必检查 `BitNet` vs `bitnet`。
    
2. **Stdout vs Stderr**：`llama-cli` 的所有加载日志都在 `stderr`，真实的推理结果在 `stdout`。脚本必须通过管道分离它们。
    
3. **幻觉控制**：1.58b 极低比特模型非常容易“复读”，必须通过 `temp=0.0` 和 Python 的 `split('\n')[0]` 进行强制截断。