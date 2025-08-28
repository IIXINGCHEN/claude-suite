# Claude Suite Windows 编译完整指南

> 🎯 **目标**: 在Windows平台上成功编译Claude Suite桌面应用程序
> 
> ⏱️ **预估时间**: 3-4小时（首次安装）
> 
> 💻 **适用系统**: Windows 10 (1903+) / Windows 11 (x64)

## 📋 快速检查清单

在开始之前，请确认您的系统满足以下要求：

- [ ] Windows 10 (1903+) 或 Windows 11
- [ ] x64 架构处理器
- [ ] 至少 4GB RAM（推荐 8GB+）
- [ ] 至少 2GB 可用磁盘空间
- [ ] 管理员权限（用于安装软件）

## 🛠️ 第一阶段：开发环境准备

### 步骤 1: 安装 Node.js (必需)

1. **下载 Node.js LTS 版本**
   - 访问 [Node.js 官网](https://nodejs.org/)
   - 下载 "LTS" 版本（推荐 18.x 或更新）
   - 选择 "Windows Installer (.msi)" 64-bit

2. **安装 Node.js**
   ```cmd
   # 下载完成后，双击 .msi 文件
   # 使用默认安装选项，确保勾选 "Add to PATH"
   ```

3. **验证安装**
   ```cmd
   # 打开命令提示符或 PowerShell
   node --version
   npm --version
   
   # 应该显示类似输出：
   # v18.19.0
   # 10.2.3
   ```

### 步骤 2: 安装 Rust 工具链 (必需)

1. **下载 Rust 安装器**
   - 访问 [Rust 官网](https://rustup.rs/)
   - 下载 `rustup-init.exe`

2. **安装 Rust**
   ```cmd
   # 运行下载的 rustup-init.exe
   # 选择 "1) Proceed with installation (default)"
   # 等待安装完成（可能需要10-20分钟）
   ```

3. **验证安装**
   ```cmd
   # 重新打开命令提示符
   rustc --version
   cargo --version
   
   # 应该显示类似输出：
   # rustc 1.75.0 (82e1608df 2023-12-21)
   # cargo 1.75.0 (1d8b05cdd 2023-11-20)
   ```

### 步骤 3: 安装 Tauri CLI (必需)

```cmd
# 使用 Cargo 安装 Tauri CLI
cargo install @tauri-apps/cli

# 验证安装
tauri --version
# 应该显示: tauri-cli 2.x.x
```

### 步骤 4: 安装包管理器 (推荐 Bun)

**选项 A: 安装 Bun (推荐)**
```powershell
# 在 PowerShell 中运行
irm bun.sh/install.ps1 | iex

# 验证安装
bun --version
```

**选项 B: 使用 npm (备选)**
```cmd
# npm 已随 Node.js 安装，无需额外操作
npm --version
```

## 🔧 第二阶段：项目准备

### 步骤 5: 获取项目源码

如果您还没有项目源码：

```cmd
# 克隆项目（如果从 Git 仓库）
git clone https://github.com/xinhai-ai/claude-suite.git
cd claude-suite

# 或者如果您已有源码，进入项目目录
cd path\to\claude-suite
```

### 步骤 6: 安装项目依赖

```cmd
# 使用 Bun (推荐)
bun install

# 或使用 npm
npm install
```

**预期输出**: 
- 创建 `node_modules` 目录
- 生成 `bun.lockb` 或 `package-lock.json`
- 自动下载 Rust 依赖（可能需要几分钟）

## 🚀 第三阶段：编译构建

### 方式一: 开发模式 (用于测试)

```cmd
# 启动开发服务器
bun run tauri dev

# 或使用 npm
npm run tauri dev
```

**预期结果**:
- 编译前端和后端代码
- 自动打开应用程序窗口
- 支持热重载调试

### 方式二: 生产构建 (推荐)

```cmd
# 构建生产版本
bun run tauri build

# 或使用 npm
npm run tauri build
```

**构建产物位置**:
- **NSIS 安装包**: `src-tauri\target\release\bundle\nsis\`
- **MSI 安装包**: `src-tauri\target\release\bundle\msi\`
- **可执行文件**: `src-tauri\target\release\claude-suite.exe`

### 方式三: 单文件可执行程序

```cmd
# 构建单文件可执行程序
bun run scripts/build-executables.js windows

# 或使用 npm
npm run scripts/build-executables.js windows
```

**构建产物位置**:
- `src-tauri\binaries\claude-code-x86_64-pc-windows-msvc.exe`

## ✅ 第四阶段：验证测试

### 测试开发版本
1. 确认应用程序窗口正常打开
2. 测试基本功能（项目创建、设置等）
3. 检查控制台无致命错误

### 测试生产版本
1. 运行生成的 `.exe` 文件
2. 或安装 `.msi` / NSIS 安装包
3. 验证所有核心功能正常工作

## 🔧 常见问题解决

### 问题 1: Rust 编译失败

**症状**: 出现 "error: linking with `link.exe` failed" 或类似错误

**解决方案**:
```cmd
# 1. 确保安装了 Visual Studio Build Tools
# 下载并安装 "Build Tools for Visual Studio 2022"
# 选择 "C++ build tools" 工作负载

# 2. 更新 Rust 工具链
rustup update stable
rustup default stable

# 3. 清理并重新构建
cargo clean
bun run tauri build
```

### 问题 2: Node.js 版本不兼容

**症状**: "Node.js version X.X.X is not supported"

**解决方案**:
```cmd
# 1. 检查当前版本
node --version

# 2. 如果版本过低，卸载并重新安装
# 从控制面板卸载 Node.js
# 重新下载并安装最新 LTS 版本

# 3. 验证安装
node --version
npm --version
```

### 问题 3: 权限和防病毒软件冲突

**症状**: "Access denied" 或编译过程被中断

**解决方案**:
```cmd
# 1. 以管理员身份运行 PowerShell 或命令提示符
# 右键点击 -> "以管理员身份运行"

# 2. 临时禁用实时保护（Windows Defender）
# 设置 -> 更新和安全 -> Windows 安全中心 -> 病毒和威胁防护

# 3. 将项目目录添加到防病毒软件白名单
```

### 问题 4: 网络连接和依赖下载问题

**症状**: 依赖下载失败或超时

**解决方案**:
```cmd
# 1. 配置 npm 国内镜像
npm config set registry https://registry.npmmirror.com/

# 2. 配置 Rust 国内镜像
# 创建或编辑 %USERPROFILE%\.cargo\config.toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"

# 3. 重新安装依赖
bun install --force
```

### 问题 5: Tauri 构建失败

**症状**: "tauri build" 命令执行失败

**解决方案**:
```cmd
# 1. 检查 Tauri CLI 版本
tauri --version

# 2. 更新到最新版本
cargo install @tauri-apps/cli --force

# 3. 清理构建缓存
cd src-tauri
cargo clean
cd ..
bun run tauri build
```

### 问题 6: 内存不足错误

**症状**: "out of memory" 或编译过程崩溃

**解决方案**:
```cmd
# 1. 关闭其他应用程序释放内存
# 2. 增加虚拟内存（页面文件）大小
# 3. 使用增量编译
set CARGO_INCREMENTAL=1
bun run tauri build
```

## 📊 性能基准

**预期构建时间**:
- 首次构建: 15-25 分钟
- 增量构建: 2-5 分钟

**预期文件大小**:
- 安装包: 30-50 MB
- 单文件 exe: 40-60 MB

## 🎉 完成！

恭喜！您已成功在 Windows 上编译了 Claude Suite。

**下一步**:
1. 测试应用程序的所有功能
2. 根据需要进行配置和定制
3. 考虑创建桌面快捷方式或开始菜单项

---

**指南版本**: v1.0  
**最后更新**: 2025-01-28  
**支持**: 如有问题，请查看项目 Issues 或 Discussions
