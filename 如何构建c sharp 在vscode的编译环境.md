要在 VS Code 中构建 C# 编译环境，目前最主流、最强大的方案是使用微软推出的 **C# Dev Kit**。这套工具能让 VS Code 获得接近 Visual Studio 的开发体验（如解决方案管理、单元测试视图等）。

以下是具体构建步骤：

---

## 1. 安装基础工具 (必备)

在配置 VS Code 之前，你的操作系统必须先有编译引擎。

- **安装 .NET SDK**：
    
    - 前往 [.NET 官网](https://dotnet.microsoft.com/download) 下载并安装最新的长期支持版（如 .NET 8.0）。
        
    - **验证**：打开终端输入 `dotnet --version`，看到版本号即安装成功。
        
- **安装 VS Code**：
    
    - 确保已安装最新版本的 [Visual Studio Code](https://code.visualstudio.com/)。
        

---

## 2. 配置 VS Code 扩展

打开 VS Code，进入扩展商店（`Ctrl+Shift+X`），搜索并安装：

1. **C# Dev Kit** (官方推荐)：它会自动安装 C# 语言支持、IntelliCode（AI 辅助）和 .NET Runtime Install Tool。
    
2. **C#** (由 Microsoft 提供)：基础的语言支持。
    

---

## 3. 创建你的第一个工程

VS Code 和传统的 IDE 不同，它更偏向于“文件夹”模式，但通过 C# Dev Kit，你可以管理 **Solution (.sln)**。

1. **新建文件夹**：在本地创建一个存放代码的目录。
    
2. **打开文件夹**：在 VS Code 中点击 `File` -> `Open Folder`。
    
3. **初始化项目**：
    
    - 按 `Ctrl+Shift+P` 打开命令面板。
        
    - 输入 `.NET: New Project...`。
        
    - 选择模板（例如 `Console App`）。
        
    - 给项目命名，VS Code 会自动生成 `.csproj` 文件和 `Program.cs`。
        

---

## 4. 编译与运行

你有两种主要方式来编译和运行程序：

### 方式 A：使用终端 (最快)

点击 VS Code 下方的 **Terminal** 标签页，输入：

Bash

```
dotnet build   # 仅编译
dotnet run     # 编译并直接运行
```

### 方式 B：使用 C# Dev Kit 视图

在 VS Code 左侧侧边栏，你会看到一个 **Solution Explorer (解决方案资源管理器)**。

- 右键点击你的项目节点。
    
- 选择 **Build** 或 **Debug -> Start New Instance**。
    

---

## 5. 配置调试环境 (F5)

如果你需要断点调试：

1. 切换到侧边栏的 **Run and Debug** (图标是一个小虫子)。
    
2. 点击 **"Create a launch.json file"**。
    
3. 在弹出的环境选择中，选择 **C#**。
    
4. VS Code 会自动生成 `.vscode/launch.json`。
    
5. **打断点**：在代码行号左侧点一下，出现红点。
    
6. **开始调试**：按 `F5`，程序会运行并在断点处停下，你可以查看变量和堆栈。
    

---

## 6. 进阶技巧 (可选)

- **代码格式化**：按 `Shift+Alt+F` 自动美化代码格式。
    
- **NuGet 包管理**：
    
    - 按 `Ctrl+Shift+P`，输入 `.NET: Add Package`，可以直接搜索并安装常用的库（如 `Newtonsoft.Json`）。
        
- **热重载 (Hot Reload)**：
    
    - 在终端运行 `dotnet watch`，这样你修改代码保存后，程序会自动重新编译并运行，无需手动重启。
        

---

### 常见问题排查

- **找不到 dotnet 命令**：通常是因为安装 SDK 后没有重启 VS Code 或电脑，导致环境变量未生效。
    
- **IntelliSense (语法提示) 不工作**：检查右下角状态栏。如果显示一个火焰图标 (OmniSharp) 或正在加载，请稍等片刻。如果报错，尝试重启 VS Code 或执行 `dotnet restore`。
    

你准备开发的是简单的控制台程序，还是复杂的 Web 接口（ASP.NET Core）？