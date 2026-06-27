# FileHawk - 智能文件下载管理器

## 项目方案文档 (Project Proposal)

---

### 1. 项目背景

在日常工作和学习中，用户经常需要从互联网下载各类文件——软件安装包、文档资料、多媒体资源等。操作系统自带的下载功能通常较为简陋，缺乏任务管理、断点续传、下载历史记录等高级特性。市面上的下载管理工具（如IDM、FDM）功能强大但多为商业软件或捆绑广告。

基于上述需求，本项目旨在开发一款轻量级、开源免费的Windows桌面下载管理工具 **FileHawk**，提供简洁美观的UI界面和实用的下载管理功能。

### 2. 功能需求分析

#### 2.1 核心功能

| 功能模块 | 描述 | 优先级 |
|---------|------|--------|
| HTTP/HTTPS下载 | 支持通过URL添加下载任务，自动解析文件名 | P0 |
| 多任务并发下载 | 支持同时下载多个文件，可配置并发数（1-10） | P0 |
| 实时进度显示 | 显示下载进度条、速度、已下载大小、剩余时间 | P0 |
| 断点续传 | 暂停后可恢复下载，不丢失已下载数据 | P1 |
| 下载历史管理 | SQLite数据库记录所有下载任务，支持增删查 | P0 |
| 下载完成通知 | 系统托盘气泡提示下载完成 | P2 |
| 文件管理 | 下载完成后可双击直接打开文件，可打开下载目录 | P1 |

#### 2.2 用户交互流程

```
启动应用 → 查看下载历史
         → 点击「新建下载」→ 输入URL → 验证格式 → 开始下载
         → 选中任务 → 暂停/恢复/取消/删除
         → 双击已完成任务 → 打开已下载文件
         → 清除已完成 → 清理历史记录
```

### 3. 技术选型理由

| 技术项 | 选型 | 理由 |
|--------|------|------|
| 开发框架 | .NET 10 + WinForms | 课程核心技术栈；原生Windows控件，性能稳定；无需额外运行时依赖 |
| 编程语言 | C# 14 | 课程教学语言；类型安全、async/await异步模型适合IO密集型任务 |
| 数据库 | SQLite (Microsoft.Data.Sqlite) | 轻量级嵌入式数据库，零配置部署；适合桌面应用本地存储 |
| HTTP客户端 | System.Net.Http.HttpClient | 内建支持异步、流式读取、Range请求（断点续传） |
| UI架构 | 纯代码构建UI（无设计器） | 更精确地控制控件属性；便于代码审查和版本控制 |
| 数据绑定 | BindingList<T> + INotifyPropertyChanged | 实现View-Model自动同步，减少手动刷新代码 |
| 并发控制 | SemaphoreSlim + async/await | 轻量级异步信号量控制并发数；不阻塞UI线程 |

### 4. 系统架构设计

#### 4.1 分层架构

```
┌─────────────────────────────────────┐
│  Presentation Layer (Forms/)        │
│  MainForm, AddDownloadDialog        │
│  ↑ 数据绑定 + 事件驱动              │
├─────────────────────────────────────┤
│  Business Logic Layer (Services/)    │
│  DownloadService (下载引擎)         │
│  ↑ 调用                             │
├─────────────────────────────────────┤
│  Data Access Layer (Services/)      │
│  DatabaseService (SQLite CRUD)      │
│  ↑                                 │
├─────────────────────────────────────┤
│  Model Layer (Models/)              │
│  DownloadItem, DownloadStatus       │
└─────────────────────────────────────┘
```

#### 4.2 核心类图

```
DatabaseService                    DownloadService
├── InsertDownload()              ├── AddDownloadAsync()
├── UpdateDownload()              ├── PauseDownload()
├── GetAllDownloads()             ├── ResumeDownloadAsync()
├── DeleteDownload()              ├── CancelDownload()
├── ClearAllDownloads()           ├── GetActiveDownloads()
└── GetStatistics()               └── events: ProgressChanged,
                                              Completed

DownloadItem (INotifyPropertyChanged)
├── Id, Url, FileName, SavePath
├── TotalBytes, DownloadedBytes
├── ProgressPercentage (computed)
├── Status, Speed
└── FormatFileSize() (static)
```

#### 4.3 下载流程状态机

```
                    ┌──────────┐
        ──添加──→  │ Pending  │
                    └────┬─────┘
                         │ 开始下载
                    ┌────▼─────┐
          ┌────────│Downloading│────────┐
          │ 暂停    └──────────┘  完成  │
     ┌────▼───┐                   ┌────▼────┐
     │ Paused │                   │Completed│
     └────┬───┘                   └─────────┘
          │ 恢复
          └──→ Downloading

     Downloading ──取消──→ Cancelled
     Downloading ──异常──→ Failed
```

### 5. 开发环境说明

| 环境项 | 版本/配置 |
|--------|----------|
| 操作系统 | Windows 11 10.0.26200 |
| IDE | Visual Studio Code / CLI |
| .NET SDK | 10.0.301 |
| 目标框架 | net10.0-windows |
| 编程语言 | C# 14 |
| 第三方依赖 | Microsoft.Data.Sqlite 10.0.0-preview.2 |
| 版本控制 | Git |
| 代码托管 | GitHub |

### 6. 项目结构

```
FileHawk/
├── FileHawk.sln
├── Project_Proposal.md          # 本文件 - 项目方案文档
├── Testing_Report.md            # 测试说明文档
└── FileHawk/
    ├── FileHawk.csproj          # 项目文件
    ├── Program.cs               # 应用程序入口
    ├── Models/
    │   └── DownloadItem.cs      # 下载任务数据模型
    ├── Services/
    │   ├── DatabaseService.cs   # SQLite数据访问层
    │   └── DownloadService.cs   # 下载引擎核心逻辑
    └── Forms/
        ├── MainForm.cs          # 主界面窗体
        └── AddDownloadDialog.cs # 新建下载对话框
```

### 7. RDD与TDD开发模式

#### 7.1 RDD (README-Driven Development)

本项目采用RDD开发模式，在编码前先编写项目方案文档（本文档），明确需求、架构和技术选型，确保每个开发阶段有清晰的目标。这种方式帮助我在AI辅助编程中更有效地向Claude Code传达设计意图。

#### 7.2 TDD (Test-Driven Development)

按照TDD流程：先设计测试用例 → 编写失败测试 → 实现功能代码 → 重构优化 → 回归测试。具体测试用例见 [Testing_Report.md](./Testing_Report.md)。

---

**项目地址**: [GitHub Repository]
**开发周期**: 2026年6月
**开发者**: [Student Name]
