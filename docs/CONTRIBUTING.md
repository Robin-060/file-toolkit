# 队友上手指南 — FileToolkit

> 本文档写给接手本项目的新队员。照着做,就能从零搭建环境、跑通项目、并按步骤完成 M1(MVP 核心功能)开发。
> 更宏观的规划见 [`ROADMAP.md`](./ROADMAP.md),产品需求见 [`PRD.md`](./PRD.md),编码规范见 [`AGENTS.md`](./AGENTS.md)。

| 项目     | 内容         |
| -------- | ------------ |
| 文档版本 | v1.0         |
| 创建日期 | 2026-07-07   |
| 当前阶段 | M0 已完成,M1 待开发 |

---

## 一、快速开始(15 分钟)

### 1.1 下载代码

```bash
git clone https://github.com/Robin-060/file-toolkit.git
cd file-toolkit
```

> 如果不用 git,也可从 GitHub 下载 ZIP 解压。

### 1.2 环境要求

| 工具 | 版本要求 | 安装方式 | 验证命令 |
| --- | --- | --- | --- |
| Node.js | ≥ 18 | [nodejs.org](https://nodejs.org) | `node --version` |
| pnpm | ≥ 8 | `npm i -g pnpm` | `pnpm --version` |
| Rust | stable | [rustup.rs](https://rustup.rs) | `rustc --version` |
| **Windows 额外** | MSVC Build Tools | winget 或 [VS 官网](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022) | 装好后终端里 `where link` 能看到 MSVC 版 |

> ⚠️ Windows 上 MSVC Build Tools 安装时要勾选 **"使用 C++ 的桌面开发"** 工作负载,否则 Rust 编译会报 `linker 'link.exe' not found`。

### 1.3 跑通项目

```bash
# 安装前端依赖(只需一次)
pnpm install

# 启动开发模式
pnpm tauri dev
```

**预期结果**:
- 首次运行会编译 Rust 后端(1~3 分钟),耐心等
- 弹出一个桌面窗口,标题 **FileToolkit**
- 输入框中打"世界",点击 **"调用 Rust 后端"** 按钮
- 页面显示 `来自 Rust 后端的问候:你好,世界!FileToolkit 已就绪。`

**看到这句话 = 环境 100% 就绪,可以开始写代码了。**

### 1.4 VS Code 配置

用 VS Code 打开项目根目录即可。推荐装以下插件:

- Vue - Official (Vue 语法支持)
- rust-analyzer (Rust 代码补全/跳转)
- Tauri (Tauri 专用工具)

> 项目根 `Cargo.toml` 已配置为 workspace,rut-analyzer 会自动识别 `src-tauri/` 下的 Rust 代码。

---

## 二、项目结构速览(5 分钟搞懂)

```
file-toolkit/
├── docs/                         ← 所有文档(全 .md 格式)
│   ├── AGENTS.md                 # AI 与人类协作规范(含编码规则)
│   ├── PRD.md                    # 产品需求(功能详情、技术架构)
│   ├── ROADMAP.md                # 全项目路线图(38 步)
│   └── CONTRIBUTING.md           # 本文档
│
├── src/                          ← Vue 3 前端(TypeScript)
│   ├── App.vue                   # 根组件(已有调通示例)
│   ├── main.ts                   # 入口
│   ├── pages/                    # 功能页面(图片/PDF/重命名/查重 → 你要写的)
│   ├── components/               # 通用 UI 组件(拖拽区/进度条 → 你要写的)
│   ├── composables/              # 组合式函数
│   ├── store/                    # Pinia 状态管理
│   └── assets/                   # 静态资源
│
├── src-tauri/                    ← Rust 后端
│   ├── src/
│   │   ├── lib.rs                # Tauri 应用组装
│   │   ├── main.rs               # 程序入口
│   │   ├── commands/             # 暴露给前端的命令(★ 核心开发区域)
│   │   │   ├── mod.rs            # 模块注册 + greet 示例
│   │   │   ├── image.rs          # 图片处理 → 你要写
│   │   │   ├── pdf.rs            # PDF 处理 → 你要写
│   │   │   ├── rename.rs         # 批量重命名 → 你要写
│   │   │   └── dedup.rs          # 查重 → 你要写
│   │   ├── pipeline/             # 流水线引擎(M3 再写)
│   │   ├── worker/               # 任务队列 + 进度 → 你要写
│   │   └── common/               # 共享类型/错误 → 你要写
│   ├── Cargo.toml                # Rust 依赖
│   └── tauri.conf.json           # Tauri 配置
│
├── Cargo.toml                    # 根工作空间清单
├── package.json                  # 前端依赖 + scripts
├── pnpm-workspace.yaml           # pnpm 配置
└── .gitignore                    # 已配置好,不要改
```

### 核心开发模式

**前端 ↔ 后端通信流程**(参考已有 `App.vue` 中的 `greet` 示例):

```
前端 Vue 组件                        Rust 后端
───────────                        ──────────
invoke("命令名", {参数})  ───────►  #[tauri::command]
                                   fn 命令名(参数) -> 返回值
接收返回值 / 事件 ◄───────────────  return / emit 事件
```

---

## 三、开发工作流(每次写功能的标准流程)

### 写一个新功能的标准流程

1. **读文档**:看 PRD.md 对应功能的需求描述
2. **写后端**:在 `src-tauri/src/commands/xxx.rs` 中实现 Rust 命令
3. **注册命令**:在 `src-tauri/src/lib.rs` 的 `generate_handler![]` 中添加新命令
4. **写前端**:在 `src/pages/XxxPage.vue` 中写 UI,用 `invoke("命令", {参数})` 调用
5. **调通验证**:`pnpm tauri dev` 启动,在窗口中测试
6. **提交代码**:小步提交,描述清楚做了什么

### 后端开发速查

```rust
// 在 commands/xxx.rs 中定义一个 Tauri 命令:
#[tauri::command]
pub fn my_command(input: String) -> Result<String, String> {
    // 处理逻辑...
    Ok(format!("处理完成: {}", input))
}

// 在 lib.rs 中注册:
.invoke_handler(tauri::generate_handler![
    commands::greet,
    commands::xxx::my_command,  // ← 新增这行
])
```

### 前端开发速查

```vue
<script setup lang="ts">
import { invoke } from "@tauri-apps/api/core";
import { ref } from "vue";

const result = ref("");

async function handleAction() {
  result.value = await invoke<string>("my_command", { input: "hello" });
}
</script>

<template>
  <button @click="handleAction">执行</button>
  <p>{{ result }}</p>
</template>
```

### 长任务(带进度)

如果操作耗时(如批量处理 100 个文件),不能用简单的 `invoke`(它会阻塞直到完成)。改用**事件流**:

```rust
// Rust 后端:边处理边发送进度事件
use tauri::Emitter;
app_handle.emit("task-progress", ProgressPayload { current: 5, total: 100 })?;
```

```typescript
// 前端:监听事件更新进度条
import { listen } from "@tauri-apps/api/event";
const unlisten = await listen("task-progress", (event) => {
  console.log(event.payload); // { current: 5, total: 100 }
});
```

---

## 四、M1 任务清单(你要做的,按顺序)

> M1 目标:实现 4 个 MVP 功能(图片/PDF/重命名/查重),让产品首次真正可用。
> **顺序很重要**:先做基础(Step 1-3),再做功能(Step 4-7),最后收尾(Step 8-10)。
> 每步完成后打勾 ✅。

---

### Step 1 — 统一错误处理与共享类型

- **目标**:建立后端错误模型 + 共享数据结构,所有后续功能复用。
- **要做什么**:
  1. 在 `common/error.rs` 中用 `thiserror` 定义错误枚举(文件不存在/IO 错误/格式不支持/任务取消等)
  2. 在 `common/types.rs` 中定义通用结构体:`Task { id, input_path, output_path, status, progress }`、`Progress { current, total }`、`TaskStatus` 枚举等
  3. 确保错误类型实现 `serde::Serialize`,可传给前端
- **为什么先做这个**:所有后续功能都用同一套错误和类型,不规范好后面返工成本高
- **产出文件**:`common/error.rs`、`common/types.rs`
- **验收**:
  - [ ] 错误类型能序列化为 JSON 传给前端
  - [ ] 类型定义被 commands/pipeline/worker 的 `mod.rs` 引用

---

### Step 2 — 任务队列与进度上报框架

- **目标**:实现 `worker` 模块,让批量操作能**并行、有进度条、失败不崩溃、可取消**。
- **要做什么**:
  1. 实现 `worker/mod.rs`:核心是一个接受 `Vec<Task>` + 闭包 → 用 `rayon` 多核并行执行 → 通过 `tauri::Emitter` 回传进度事件
  2. 每个 task 的执行结果(成功/失败)记录到 `Task` 的 status 字段
  3. 支持"取消"标志位,检查后优雅退出
- **关键技术点**:
  - 用 `rayon::ThreadPool` 控制并行度
  - 用 `Arc<AtomicBool>` 实现取消信号
  - 用 `app_handle.emit("task-progress", ...)` 推进度
- **产出文件**:`worker/mod.rs`
- **验收**:
  - [ ] 给 100 个模拟 task,并行跑,前端能看到进度百分比
  - [ ] 中间故意让某个 task 失败,不影响其余
  - [ ] 点取消按钮,正在跑的 task 在 1-2 秒内停下
  - [ ] Rust 编译通过(`cargo check --manifest-path src-tauri/Cargo.toml`)

---

### Step 3 — 前端基础设施

- **目标**:接入状态管理 + 路由 + UI 组件库,让后续功能页有统一框架。
- **要做什么**:
  1. 装 Pinia:`pnpm add pinia`,在 `main.ts` 中注册
  2. 创建 `store/task.ts`:集中管理当前任务队列、进度、状态
  3. 装 Element Plus:`pnpm add element-plus`,在 `main.ts` 引入
  4. 创建基础布局组件:`components/AppLayout.vue`(侧边栏导航 + 主内容区)
  5. 配置路由:图片/PDF/重命名/查重 4 个页面的路由占位
  6. (可选)补装 ESLint + Prettier:`pnpm add -D prettier eslint @eslint/js typescript-eslint eslint-plugin-vue vue-eslint-parser`,配置 `.prettierrc.json` 和 `eslint.config.js`
- **为什么用 Element Plus**:国产组件库中文文档好,队友上手快;你喜欢别的(Naive UI / Ant Design Vue)也可以换。
- **产出文件**:`store/task.ts`、`components/AppLayout.vue`、`main.ts`(改造)
- **验收**:
  - [ ] 应用打开后有侧边栏 + 主区布局
  - [ ] 点侧边栏可切换 4 个占位页面
  - [ ] `pnpm lint` 通过(如果装了 ESLint)

---

### Step 4 — 图片批量压缩/转换 ★

- **目标**:支持 JPG/PNG/WebP/GIF/BMP 批量导入→压缩/转换格式/调整尺寸→输出。
- **要做什么**:
  1. **后端** `commands/image.rs`:
     - 实现 `#[tauri::command] fn compress_images(files: Vec<String>, quality: u8, format: String, output_dir: String) -> Result<Vec<TaskResult>, String>`
     - 用 Rust 的 `image` crate 读文件→根据需要 resize→按指定质量/格式编码→写输出文件
     - 每处理完一个文件,emit 进度事件
     - 依赖选型:优先用 `image` crate(纯 Rust,无外部依赖);如果要更好的 WebP/AVIF 支持,后续切 `libvips`
  2. **前端** `pages/ImagePage.vue`:
     - 拖拽区或文件选择按钮,支持多选
     - 参数面板:输出格式(JPG/PNG/WebP)、质量滑块(0-100)、宽度/高度约束(可选)
     - 输出目录选择
     - 处理按钮 + 进度条 + 结果列表(每个文件的 原大小 → 新大小)
     - 参考 `App.vue` 的 invoke 模式与 task store
  3. 在 `lib.rs` 中注册 `compress_images` 命令
- **产出文件**:`commands/image.rs`、`pages/ImagePage.vue`
- **验收**:
  - [ ] 拖入 50 张图片,设定质量 80%,输出到新目录,全部完成
  - [ ] 处理前后体积对比可见
  - [ ] 失败隔离:坏文件被跳过,不影响其余
  - [ ] 进度条实时更新
  - [ ] 可取消

---

### Step 5 — PDF 合并/拆分/压缩 ★

- **目标**:拖入多个 PDF 一键合并,按页码区间拆分成多个,压缩 PDF 体积。
- **要做什么**:
  1. **后端** `commands/pdf.rs`:
     - 合并:`fn merge_pdfs(files: Vec<String>, output_path: String)` → 用 `lopdf` 逐页拷贝到新文档
     - 拆分:`fn split_pdf(file: String, ranges: Vec<(u32, u32)>, output_dir: String)` → 按页码范围提取
     - 压缩:`fn compress_pdf(file: String, output_path: String)` → 重采样图片 + 压缩流
     - 安装依赖:在 `src-tauri/Cargo.toml` 的 `[dependencies]` 下加 `lopdf = "0.34"`
  2. **前端** `pages/PdfPage.vue`:
     - Tab 切换:合并 / 拆分 / 压缩
     - 合并模式:拖入多个 PDF,拖拽排序,输出文件名输入
     - 拆分模式:选一个 PDF,输入页码范围(如 `1-5,6-10`)
     - 压缩模式:拖入 PDF,显示压缩前后体积
  3. 在 `lib.rs` 中注册所有 PDF 命令
- **产出文件**:`commands/pdf.rs`、`pages/PdfPage.vue`
- **验收**:
  - [ ] 合并 3 个 PDF → 1 个,页序正确
  - [ ] 拆分为多个文件,每份页码范围正确
  - [ ] 压缩后体积下降,内容完整

---

### Step 6 — 文件批量重命名 ★

- **目标**:用模板变量(`{name}` `{date}` `{index:3}` 等)批量改名,执行前预览,冲突时报错。
- **要做什么**:
  1. **后端** `commands/rename.rs`:
     - `fn preview_rename(files: Vec<String>, pattern: String) -> Vec<{old: String, new: String, conflict: bool}>`
       - 解析 pattern 里的 `{name}`(原文件名)、`{ext}`(扩展名)、`{index:3}`(序号,补齐 3 位)、`{date:yyyy-MM-dd}`(文件修改日期)
       - 检测同名冲突
     - `fn execute_rename(plan: Vec<{old: String, new: String}>)` → 实际执行 `std::fs::rename`
       - 优先用回收站(`trash` crate),失败了再用带后缀的备份
  2. **前端** `pages/RenamePage.vue`:
     - 拖入文件列表
     - 模板输入框 + 常用变量快捷按钮
     - **实时预览**新文件名列表(绿色正常/红色冲突)
     - 确认后执行,支持撤销
  3. 安装依赖:`chrono`(日期格式化)、`trash`(移入回收站)
- **产出文件**:`commands/rename.rs`、`pages/RenamePage.vue`
- **验收**:
  - [ ] 模板 `{name}-v2-{index:3}` 预览正确
  - [ ] 重名冲突时阻止执行并标红
  - [ ] 执行后文件确实改了名
  - [ ] 撤销能恢复原名

---

### Step 7 — 重复文件查重 ★

- **目标**:扫描目录,找出内容完全相同的重复文件,分组显示,智能保留一个。
- **要做什么**:
  1. **后端** `commands/dedup.rs`:
     - `fn scan_duplicates(dir: String) -> Vec<Vec<{path: String, size: u64, modified: u64}>>`
       - 第一步:按文件大小分组,相同大小的才继续
       - 第二步:对同大小组,读取内容算 `blake3` 哈希(极快)
       - 第三步:哈希相同的归为重复组,emit 进度事件
     - `fn delete_duplicates(files: Vec<String>)` → 移到回收站
     - 安装依赖:在 `Cargo.toml` 加 `blake3 = "1"`
  2. **前端** `pages/DedupPage.vue`:
     - 选目录 → 开始扫描
     - 扫描进度条
     - 结果:重复组列表,每组显示文件名/大小/修改日期
     - "智能保留"下拉(保留最新/最大/第一个),一键删除其余
  3. **性能注意**:
     - 大目录(10GB+)扫描用流式读,不要一次全加载进内存
     - 用 `rayon` 并行 hash
- **产出文件**:`commands/dedup.rs`、`pages/DedupPage.vue`
- **验收**:
  - [ ] 扫描包含重复图片的目录,正确识别并分组
  - [ ] "保留最新"策略:最旧的文件被删除,最新的保留
  - [ ] 大目录扫描有进度,可取消
  - [ ] 扫描 1000 个文件(总 1GB)在 30 秒内完成

---

### Step 8 — 统一通用组件(边做边抽)

- **目标**:4 个功能页里反复出现的 UI 模式(文件拖拽/选择、输出目录选择、进度条、结果列表)抽成**可复用组件**。
- **要做什么**:
  - `components/FileDropZone.vue`:拖拽 / 点击选文件,emit 文件列表
  - `components/TaskProgress.vue`:百分比进度条 + 取消按钮
  - `components/ResultList.vue`:处理结果列表(成功/失败/对比信息)
  - `composables/useBatchTask.ts`:封装 `invoke` + 事件监听 + store 更新
- **注意**:不用等 4 个功能全部写完再做,把前两个功能(图片/PDF)写完后就可以开始抽共性。
- **产出文件**:`components/FileDropZone.vue`、`components/TaskProgress.vue`、`composables/useBatchTask.ts`
- **验收**:
  - [ ] 4 个功能页都用这套组件,行为一致

---

### Step 9 — 集成测试与 Bug 修复

- **目标**:把 4 个功能串起来跑一遍,修复边界问题。
- **要做什么**:
  1. 写一份测试清单 `docs/design/m1-test-cases.md`,覆盖:
     - 正常流程(10 个文件以内)
     - 大批量(100+ 文件)
     - 大文件(单文件 1GB+)
     - 边界:空目录、特殊字符路径、只读文件、损坏文件
     - 取消/暂停
     - 深色模式显示
  2. 逐项跑一遍,记录 bug
  3. 修复 bug
- **产出文件**:`docs/design/m1-test-cases.md`
- **验收**:
  - [ ] 测试清单全部通过
  - [ ] 无已知崩溃/卡死问题
  - [ ] 1000+ 文件处理不超内存

---

### Step 10 — M1 发布

- **目标**:打 tag,更新 README,内部发布 alpha 版本。
- **要做什么**:
  1. 更新根 `README.md`:把 🚧 改为 ✅,给功能截图
  2. 写 `CHANGELOG.md`
  3. `git tag v0.1.0-alpha && git push --tags`
  4. 在 GitHub 上创建 Release
- **产出文件**:`CHANGELOG.md`、GitHub Release
- **验收**:
  - [ ] 别人 clone + `pnpm install` + `pnpm tauri dev` 能跑
  - [ ] 4 个功能都可用

---

## 五、编码与协作规范(必须遵守)

完整的规则见 [`AGENTS.md`](./AGENTS.md),这里提炼最重要的几条:

| 规则 | 说明 |
| --- | --- |
| **所有文档用 .md** | 不要生成 .docx/.pdf 格式的文档 |
| **前端必须 TypeScript** | 禁止纯 JS 文件 |
| **Rust 函数要有 `///` 注释** | 公共 API 必须文档化 |
| **commit 前跑 lint** | 前端 `pnpm lint`,后端 `cargo clippy`、`cargo fmt` |
| **文件操作类功能必须遵守** | 失败隔离(单文件坏不影响整批) + 可取消 + 预览(dry-run) |
| **批处理前做预览** | 重命名、删除类操作,执行前展示将要改什么 |
| **默认不覆盖原文件** | 输出到独立目录或加后缀,除非用户确认 |
| **文档先于代码** | 每个功能先写设计文档再写代码 |
| **隐私红线** | 文件内容绝不上传网络,默认零遥测 |

### 提交消息格式

```
feat: 实现图片批量压缩功能
fix: 修复 PDF 合并页序错误
docs: 补充 M1 测试用例
refactor: 抽取 FileDropZone 通用组件
```

---

## 六、常用命令速查

```bash
# === 开发 ===
pnpm tauri dev          # 启动完整开发环境(前端+后端)
pnpm dev                # 仅启动前端 Vite(调试 UI 用)

# === 后端 ===
cargo check --manifest-path src-tauri/Cargo.toml   # 检查 Rust 编译
cargo fmt --manifest-path src-tauri/Cargo.toml       # Rust 格式化
cargo clippy --manifest-path src-tauri/Cargo.toml -- -D warnings  # Rust 静态检查

# === 前端 ===
pnpm lint               # ESLint 检查
pnpm format             # Prettier 格式化
pnpm build              # 构建前端产物

# === git ===
git status
git add -A
git commit -m "feat: 描述"
git push

# === 加 Rust 依赖(在 src-tauri 下操作) ===
cd src-tauri
cargo add lopdf         # PDF 操作
cargo add blake3        # 快速哈希
cargo add image         # 图片处理
cargo add rayon         # 并行
cargo add chrono        # 日期时间
cargo add trash         # 回收站操作
cargo add thiserror     # 错误类型
```

---

## 七、FAQ — 常见踩坑

### Q1: rust-analyzer 报 "failed to find any projects"

项目根 `Cargo.toml` 是 workspace 清单。如果还不生效,在 VS Code 中按 `Ctrl+Shift+P` → 输入 "rust-analyzer: Restart server" 回车。

### Q2: `pnpm tauri dev` 报找不到 esbuild

`pnpm-workspace.yaml` 中已配置 `onlyBuiltDependencies: [esbuild]`。如果还报错:
```bash
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Q3: `cargo build` 报 `linker 'link.exe' not found`(Windows)

MSVC Build Tools 没装或没勾选 C++ 工作负载。重新运行 Visual Studio Installer,确保选上 "使用 C++ 的桌面开发"。

### Q4: 中文路径/文件名有问题

Rust 的 `std::fs` 和我们的代码都支持 UTF-8 路径。如果遇到,用 `PathBuf` 而非 `String` 传路径。

### Q5: 大文件(1GB+)处理时崩溃

用**流式读取**(`BufReader` 分块读)而非 `fs::read_to_string`,配合 `rayon` 并行处理。

### Q6: 想加新的 Rust 依赖

```bash
cd src-tauri
cargo add 包名
```
不要手动改 `Cargo.toml`,用 `cargo add` 保证格式正确。

---

## 八、总结:你现在要做的,按顺序

```
Step 1  错误/类型      ← 今天做,为后面打地基
  ↓
Step 2  任务队列        ← 今天或明天,连着 Step 1
  ↓
Step 3  前端基建        ← 明天,UI 框架立起来
  ↓
Step 4  图片压缩 ★      ← 第一个完整功能,走通全流程最重要
  ↓
Step 5  PDF 合并 ★
  ↓
Step 6  批量重命名 ★
  ↓
Step 7  查重 ★
  ↓
Step 8  通用组件        ← 边做边抽,不单独耗时间
  ↓
Step 9  测试 + 修 Bug
  ↓
Step 10 发布 v0.1.0
```

> 💡 **建议**:Step 1-3 基础做完后,Step 4-7 四个功能如果有多人可以**并行开发**(各自独立的 `commands/xxx.rs` 和 `pages/XxxPage.vue`,没有代码冲突)。

---

有问题随时提 issue 或联系项目发起人。祝顺利! 🚀
