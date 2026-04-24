# 替换下面的 REAL_PATH 为上面 find 出来的真实结果
REAL_PATH="/mnt/c/Users/lz567/Downloads/Llama3-8B-1.58-100B-tokens-TQ2_0.gguf"

# 再次尝试拷贝
cp "$REAL_PATH" ~/BitNet_Final/models/




wsl -d Ubuntu-24.04 -u lz56786

cd ~
source venv/bin/activate

# 手动运行一次推理 

~/BitNet_Final/build/bin/llama-cli \ -m ~/BitNet_Final/models/Llama3-8B-1.58-100B-tokens-TQ2_0.gguf \ -p "News: Apple released M4 chip. Category:" \ -n 8 -t 6 --temp 0.0


~/BitNet_Final/final_classify.py




# 进入模型目录
cd ~/BitNet/models

# 执行拷贝（注意：WSL 中路径是 /mnt/c/...）
cp /mnt/c/Users/lz567/Downloads/Llama3-8B-1.58-100B-tokens-TQ2_0.gguf .

# 验证文件是否已成功到达（应该显示 2.1G 或 2.2G）
ls -lh

# 进入模型目录
cd ~/BitNet/models

# 执行拷贝（注意：WSL 中路径是 /mnt/c/...）
cp /mnt/c/Users/lz567/Downloads/meta-llama-3-8b-instruct.Q4_K_M.gguf .

# 验证文件是否已成功到达（应该显示 2.1G 或 2.2G）
ls -lh

../build/bin/llama-cli -m meta-llama-3-8b-instruct.Q4_K_M.gguf \
    -p "The mathematical reason why BitNet 1.58b is more efficient than FP16 is" \
    --temp 0.4 \
    --repeat-penalty 1.1 \
    -t 6 \
    -n 128