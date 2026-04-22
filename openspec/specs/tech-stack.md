# Tech Stack

## 架构概览

SkillSync 采用双层架构：**Rust 核心引擎** 负责所有文件系统操作、版本管理、同步逻辑；**原生 UI 层** 负责用户交互。两层之间通过 FFI 或 IPC 通信。

```
┌─────────────────────────────────────────┐
│           UI 层（SwiftUI）               │
│   技能列表 · 编辑器 · 同步配置 · 历史    │
└────────────────┬────────────────────────┘
                 │ Swift ↔ Rust FFI (uniffi-rs)
┌────────────────▼────────────────────────┐
│           核心引擎（Rust）               │
│   skill 解析 · 版本快照 · symlink 管理   │
│   文件监听 · 项目扫描 · 冲突检测         │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│              持久层                      │
│   SQLite（元数据）· 文件系统（skill 文件）│
└─────────────────────────────────────────┘
```

---

## UI 层

### SwiftUI（首选方案）

原生 macOS 体验是第一优先级。SwiftUI 能自然支持 macOS 的各种系统集成点，这对一个开发者工具尤为重要。

**关键理由：**

- MenuBar 应用、系统通知、Spotlight 集成开箱即用
- Security-Scoped Bookmarks（沙箱文件访问权限持久化）需要 AppKit/SwiftUI
- 文件选择对话框、Finder 集成、Quick Look 预览无需额外工作
- macOS 原生的 `.app` 包格式，便于分发和公证（Notarization）

**Rust ↔ Swift 桥接：**

使用 [uniffi-rs](https://github.com/mozilla/uniffi-rs) 生成 Swift bindings，Rust 核心暴露为 Swift 可调用的 framework。这是 Mozilla、1Password 等工程团队验证过的生产方案。

```
Rust crate (core/) 
  → uniffi 生成 SwiftPackage 
    → Swift 直接 import 调用
```

> **备选方案：Tauri 2.x**
> 若优先快速 MVP 而非原生体验，可用 Tauri + React。核心 Rust 逻辑完全复用，UI 用 Web 技术实现。缺点：WebView 渲染、系统权限集成更繁琐。可作为后续跨平台扩展（Windows/Linux）的路径。

---

## 核心引擎（Rust）

### 语言与工具链

- **Rust stable**，edition 2021
- **Cargo workspace** 管理多 crate：`core`（引擎逻辑）、`cli`（调试用命令行工具）、`ffi`（uniffi bindings）

### 关键依赖

#### 文件系统与监听

| Crate | 用途 |
|---|---|
| `notify` | 跨平台文件监听，macOS 使用 FSEvents backend，实现 skill 文件变更实时感知 |
| `walkdir` | 递归扫描 skill 目录树 |
| `glob` | 路径模式匹配（扫描各 Agent 的 skill 路径） |

#### Skill 解析

| Crate | 用途 |
|---|---|
| `gray_matter` | 解析 SKILL.md 的 YAML frontmatter（`name`、`description` 等字段） |
| `pulldown-cmark` | Markdown 解析，用于 skill 内容的结构化处理 |
| `serde` + `serde_yaml` | YAML 序列化/反序列化，frontmatter 数据模型 |

#### 持久化

| Crate | 用途 |
|---|---|
| `rusqlite` | SQLite 操作，存储 skill 元数据、快照索引、Agent 配置、同步状态 |
| `refinery` | SQL migration 管理，确保数据库 schema 版本可控 |

#### 版本管理

- 每次 skill 保存前，将当前内容序列化存入 SQLite（轻量快照）
- diff 使用 `similar` crate 实现语义级行差异对比
- 可选 Git backend：`git2` crate，供高级用户将 skill 库管理在 Git 仓库中

#### Symlink 与同步

- 标准库 `std::os::unix::fs::symlink` 处理 symlink 创建
- 自研同步引擎：维护「source skill → [target paths]」的映射关系，变更时 propagate 到所有目标

#### FFI 层

| Crate | 用途 |
|---|---|
| `uniffi` | 生成 Swift bindings，暴露核心 API |

---

## 持久层

### SQLite（via rusqlite）

存储所有元数据，skill 内容本身仍以文件形式存在磁盘。

**主要表设计方向：**

```sql
skills          -- skill 注册表（name, path, description, agent_targets）
snapshots       -- 版本快照（skill_id, content, created_at）
agent_configs   -- Agent 路径配置（agent_name, global_path, project_paths）
sync_state      -- 同步状态（skill_id, target_path, last_synced_at, checksum）
```

### 文件系统

skill 内容的 source of truth 是文件，SQLite 只存索引和元数据。这样即使数据库损坏，skill 文件本身不丢失，可以重建索引。

---

## macOS 系统集成

### 沙箱与权限

- 使用 **Security-Scoped Bookmarks** 持久化用户授权的目录访问，不在每次启动时重新请求
- 遵循最小权限原则：仅请求用户明确授权的目录的读写权限
- Keychain 存储任何敏感配置（如未来支持的远程同步凭证）

### 分发

- 目标：直接分发（非 App Store），保留对文件系统的完整访问能力
- 需要 **Hardened Runtime** + **Notarization**（苹果公证）
- 自动更新：集成 [Sparkle](https://sparkle-project.org/) framework

---

## 开发工具链

| 工具 | 用途 |
|---|---|
| Xcode | macOS app 构建、签名、打包 |
| Swift Package Manager | 管理 Rust-generated framework 作为 Swift 依赖 |
| `cargo-make` | 统一构建脚本（编译 Rust → 生成 bindings → 构建 Xcode project） |
| `cargo-nextest` | Rust 单元测试运行器 |
| GitHub Actions | CI：Rust 测试 + Xcode build + Notarization |

---

## 技术决策记录

**为什么不用 Electron？**
Electron 对于一个本质上是文件管理工具的 app 过于重量级。macOS 用户对内存占用和启动速度有较高预期，Rust + SwiftUI 的组合可以做到启动瞬时、常驻 MenuBar 几乎无感。

**为什么 skill 内容不存 SQLite？**
文件是 skill 的自然形态。用户可以直接在终端或编辑器里修改 skill，应用通过 `notify` 感知变更。将内容存数据库会破坏这种透明性，也让 Git 版本管理变得麻烦。

**为什么选 uniffi 而不是手写 C ABI？**
uniffi 从 Rust 类型自动生成 Swift bindings，类型安全，减少 FFI 层的手工维护量。Mozilla 在 Firefox iOS 上大量使用，工程成熟度有保障。