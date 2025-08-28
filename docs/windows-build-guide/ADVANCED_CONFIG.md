# Claude Suite Windows 高级配置指南

> 🎯 **目标**: 优化构建性能，自定义构建选项，解决复杂问题
> 
> 👥 **适用对象**: 有经验的开发者，需要定制化构建的用户

## 🚀 构建性能优化

### 1. 并行编译优化

**配置 Rust 编译器并行度**:
```toml
# 在 src-tauri/.cargo/config.toml 中添加
[build]
jobs = 4  # 根据 CPU 核心数调整

[target.x86_64-pc-windows-msvc]
rustflags = ["-C", "target-cpu=native"]  # 针对本机 CPU 优化
```

**配置 Node.js 构建并行度**:
```cmd
# 设置环境变量
set UV_THREADPOOL_SIZE=8
set NODE_OPTIONS=--max-old-space-size=4096

# 或在 PowerShell 中
$env:UV_THREADPOOL_SIZE=8
$env:NODE_OPTIONS="--max-old-space-size=4096"
```

### 2. 缓存优化

**启用 Rust 增量编译**:
```cmd
# 设置环境变量（永久）
setx CARGO_INCREMENTAL 1
setx RUSTC_WRAPPER sccache  # 如果安装了 sccache
```

**配置 npm/Bun 缓存**:
```cmd
# npm 缓存配置
npm config set cache C:\npm-cache
npm config set cache-min 86400

# Bun 缓存配置
bun config set cache-dir C:\bun-cache
```

### 3. 内存优化

**大型项目内存配置**:
```cmd
# 增加 Node.js 堆内存限制
set NODE_OPTIONS=--max-old-space-size=8192

# 配置 Rust 编译器内存使用
set CARGO_BUILD_JOBS=2  # 减少并行任务以节省内存
```

## 🔧 自定义构建配置

### 1. Tauri 配置定制

**编辑 `src-tauri/tauri.conf.json`**:
```json
{
  "build": {
    "beforeBuildCommand": "bun run build",
    "beforeDevCommand": "bun run dev",
    "devPath": "http://localhost:1420",
    "distDir": "../dist"
  },
  "bundle": {
    "active": true,
    "targets": ["msi", "nsis"],
    "identifier": "com.claude.suite",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/icon.ico"
    ],
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": "",
      "wix": {
        "language": ["zh-CN", "en-US"]
      }
    }
  }
}
```

### 2. Cargo 构建配置

**优化 `src-tauri/Cargo.toml`**:
```toml
[profile.release]
opt-level = "z"          # 最大体积优化
lto = "fat"              # 完整链接时优化
codegen-units = 1        # 单个代码生成单元
panic = "abort"          # 异常时直接终止
strip = "symbols"        # 移除调试符号
overflow-checks = false  # 禁用溢出检查

[profile.dev]
opt-level = 1            # 开发时轻度优化
debug = true             # 保留调试信息
incremental = true       # 启用增量编译

# 针对依赖的优化
[profile.dev.package."*"]
opt-level = 2            # 依赖包使用更高优化级别
```

### 3. Vite 构建配置

**优化 `vite.config.ts`**:
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    target: 'esnext',
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,  // 移除 console.log
        drop_debugger: true  // 移除 debugger
      }
    },
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu']
        }
      }
    },
    chunkSizeWarningLimit: 1000
  },
  server: {
    port: 1420,
    strictPort: true
  }
})
```

## 🛡️ 安全和签名配置

### 1. 代码签名设置

**Windows 代码签名证书**:
```json
// 在 tauri.conf.json 中配置
{
  "bundle": {
    "windows": {
      "certificateThumbprint": "YOUR_CERT_THUMBPRINT",
      "digestAlgorithm": "sha256",
      "timestampUrl": "http://timestamp.digicert.com"
    }
  }
}
```

**使用 PowerShell 签名**:
```powershell
# 手动签名可执行文件
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert
Set-AuthenticodeSignature -FilePath ".\target\release\claude-suite.exe" -Certificate $cert
```

### 2. 安全策略配置

**内容安全策略 (CSP)**:
```json
// 在 tauri.conf.json 中配置
{
  "security": {
    "csp": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
  }
}
```

## 🔍 调试和诊断

### 1. 详细日志配置

**启用详细构建日志**:
```cmd
# Rust 详细日志
set RUST_LOG=debug
set CARGO_LOG=cargo::core::compiler::fingerprint=info

# Tauri 详细日志
set TAURI_DEBUG=1

# 执行构建
bun run tauri build
```

### 2. 性能分析

**构建时间分析**:
```cmd
# 安装 cargo-timing
cargo install cargo-timing

# 分析构建时间
cd src-tauri
cargo timing --release
```

**包大小分析**:
```cmd
# 安装 cargo-bloat
cargo install cargo-bloat

# 分析二进制文件大小
cd src-tauri
cargo bloat --release
```

### 3. 依赖审计

**安全审计**:
```cmd
# 安装 cargo-audit
cargo install cargo-audit

# 执行安全审计
cd src-tauri
cargo audit
```

## 🌐 跨版本兼容性

### 1. 多版本 Node.js 管理

**使用 nvm-windows**:
```cmd
# 安装 nvm-windows
# 从 GitHub 下载安装包

# 安装和切换 Node.js 版本
nvm install 18.19.0
nvm install 20.10.0
nvm use 18.19.0
```

### 2. Rust 工具链管理

**多工具链配置**:
```cmd
# 安装特定版本
rustup install 1.75.0
rustup install nightly

# 为项目设置特定工具链
cd claude-suite
rustup override set 1.75.0
```

## 📦 分发和部署

### 1. 自动化构建脚本

**创建 `build.bat`**:
```batch
@echo off
echo 开始构建 Claude Suite...

echo 1. 清理旧构建...
if exist "src-tauri\target\release" rmdir /s /q "src-tauri\target\release"
if exist "dist" rmdir /s /q "dist"

echo 2. 安装依赖...
bun install

echo 3. 构建前端...
bun run build

echo 4. 构建 Tauri 应用...
bun run tauri build

echo 5. 构建完成！
echo 安装包位置: src-tauri\target\release\bundle\
pause
```

### 2. CI/CD 集成

**GitHub Actions 配置示例**:
```yaml
name: Build Windows
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
      - run: bun install
      - run: bun run tauri build
      - uses: actions/upload-artifact@v3
        with:
          name: windows-build
          path: src-tauri/target/release/bundle/
```

---

**配置指南版本**: v1.0  
**适用于**: Claude Suite v1.1.4+  
**最后更新**: 2025-01-28
