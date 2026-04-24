cat <<'EOF'> ~/Gemma_Server/server.py
from fastapi import FastAPI
from pydantic import BaseModel
import subprocess
import os
import re # 记得在文件开头导入 re

app = FastAPI(title="Gemma-2 图纸助手")

CLI_PATH = os.path.expanduser("~/BitNet_Final/build/bin/llama-cli")
MODEL_PATH = os.path.expanduser("~/Gemma_Server/models/gemma-2-2b-it-Q4_K_M.gguf")

class DrawingInput(BaseModel):
    text: str

```
@app.post("/extract")
async def extract_info(item: DrawingInput):
    prompt = (
        "<start_of_turn>user\n"
        "Analyze this OCR text. \n"
        "1. Identify DrawingType: 'Assembly' or 'Part'.\n"
        "2. Extract ProductName and DrawingNumber (N/A if not found).\n"
        "3. Provide BOM as list: [{'POS': '...', 'Original': '...', 'Chinese': '...'}].\n"
        f"Text: {item.text[:2000]}\n"
        "Return ONLY the JSON object. Do not explain.<end_of_turn>\n"
        "<start_of_turn>model\n"
        "{"
    )
    
    cmd = [CLI_PATH, "-m", MODEL_PATH, "-p", prompt, "-n", "1024", "--temp", "0.0", "-t", "4"]
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, encoding='utf-8')
        raw_output = result.stdout
        
        # --- 核心修复逻辑 ---
        # 1. 尝试找到第一个 { 和最后一个 } 之间的内容
        match = re.search(r'(\{.*\})', raw_output, re.DOTALL)
        if match:
            prediction = match.group(1)
        else:
            # 2. 如果没找到完整的，可能是被截断了，手动补齐右侧
            prediction = "{" + raw_output.split("{", 1)[-1]
            if not prediction.strip().endswith("}"):
                # 检查最后是否是列表项，强行收尾（这是一个保命逻辑）
                prediction = prediction.rsplit('}', 1)[0] + "}]}" if ']' in prediction else prediction + "}"
        
        return {"status": "success", "data": prediction}
    except Exception as e:
        return {"status": "error", "message": str(e)}
```

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
EOF