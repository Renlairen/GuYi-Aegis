# 🛡️ GuYi Aegis Pro - 企业级验证管理系统

> **📚 官方文档**: [**https://aegis.可爱.top/**](https://aegis.可爱.top/)  
> *(提示：v8.0 Enterprise 架构已全新升级，为了获得最佳的对接体验，请务必优先查阅官方文档)*

<p align="left">
  <a href="https://aegis.可爱.top/">
    <img src="https://img.shields.io/badge/Version-v8.0_Enterprise-6366f1.svg?style=flat-square&logo=github&logoColor=white" alt="Version">
  </a>
  <img src="https://img.shields.io/badge/Database-SQLite3_Accelerated-007AFF.svg?style=flat-square&logo=sqlite&logoColor=white" alt="Database">
  <img src="https://img.shields.io/badge/Security-Enterprise_Grade-34C759.svg?style=flat-square&logo=security-scorecard&logoColor=white" alt="Security">
  <img src="https://img.shields.io/badge/License-Proprietary-FF3B30.svg?style=flat-square" alt="License">
</p>

---

## 📖 产品概述

**GuYi Aegis Pro v8.0 Enterprise** 是一套专为独立开发者与中小微企业打造的 **高可用、低代码** 软件授权分发解决方案。

v8.0 版本彻底移除了传统数据库依赖，采用独家优化的 **SQLite3 文件数据库架构**，实现了读写性能的显著提升与“零配置”秒级部署。系统内置 **App Key 多租户隔离**、**自愈合内核** 以及 **云变量 2.0 引擎**，为您的软件资产提供固若金汤的保护与灵活高效的分发控制。

---

## 💎 核心特性 (Core Features)

### 🔐 1. 金融级安全防护体系
构建了从网络层到应用层的多维防御矩阵，确保业务数据零泄露。

-   **网络安全响应头**: `config.php` 自动设置 `X-Frame-Options` (防点击劫持)、`X-XSS-Protection` (防XSS)、`X-Content-Type-Options` (防MIME嗅探)、`Referrer-Policy`，提升HTTP安全基线。
-   **CSRF 全局防护**: 后台管理操作 (`cards.php`) 全程启用 Token 令牌校验，防止跨站请求伪造。配合数据层面的 PDO 预处理，有效阻断 SQL 注入与越权操作。
-   **强制安全合规**: 首次登录强制要求管理员重置默认密码，符合等保安全规范，提升系统初始安全性。
-   **会话安全强化**:
    -   管理员登录 (`cards.php`) 引入 `HMAC-SHA256` 签名的 `admin_trust` Cookie 实现更安全的自动登录。该 Cookie 绑定 `User-Agent` 和管理员密码哈希指纹，密码修改后旧 Cookie 立即失效。
    -   用户验证 (`auth_check.php`) 严格检测 `User-Agent` 匹配，防止会话劫持。
    -   **会话超时与过期**: `auth_check.php` 实现会话空闲超时（8小时）和卡密过期时间（严格使用 `<=`）双重判断，确保过期卡密和闲置会话及时失效。
-   **严格卡密校验**: `auth_check.php` 和 `database.php` 中的 `verifyCard` 逻辑对卡密状态（未激活、已激活、已封禁）、有效期（精确到秒）和设备绑定情况进行严格判断，防止多设备滥用和过期使用。
-   **后台防爆破**: `cards.php` 在管理员登录失败时，引入 `usleep()` 延迟机制，抵御暴力破解攻击。

### 🏢 2. 多租户 SaaS 隔离架构
一套系统即可支撑庞大的软件矩阵，实现集中化管理与数据隔离。

-   **App Key 租户隔离**: `database.php` 支持无限添加应用，每个应用拥有独立的 `App Key`。验证核心 (`verifyCard`) 强制要求 `App Key` 进行卡密数据、设备绑定、云变量的物理级逻辑隔离。
-   **灵活鉴权模式**: `Verifyfile/api.php` 接口支持针对特定应用进行鉴权，确保不同软件之间的用户数据和功能互不干扰。同时提供“免卡密”模式，仅凭 AppKey 即可获取公共变量。
-   **实时封禁控制台**: `cards.php` 后台支持毫秒级的卡密阻断 (`ban_card`) 与解封 (`unban_card`) 操作，异常情况（如设备异常或滥用）可立即处置。
-   **应用状态管理**: `cards.php` 可启用/禁用应用，禁用状态下的应用无法进行验证，实现细粒度的应用管控。

### ☁️ 3. 云变量引擎
无需更新软件即可动态控制内容，支持 **Upsert** 智能写入逻辑。

-   **独立变量表**: `database.php` 维护 `app_variables` 表，为每个应用存储独立的键值对变量。
-   **公开变量 (Public)**: `Verifyfile/api.php` 允许客户端无需卡密登录，仅凭 `App Key` 即可获取勾选为“公开”的变量（适用于全局公告、版本检测信息等）。
-   **私有变量 (Private)**: `Verifyfile/api.php` 在卡密验证通过后，与卡密有效期等信息一同下发所有变量（包括私有变量），适用于 VIP 下载链接、专用配置等敏感资源。
-   **灵活配置**: `cards.php` 后台可为每个应用添加、删除和管理专属云变量，并设置其公开/私有权限。

### ⚡ 4. 高性能自愈内核
基于 SQLite3 的深度优化，确保数据操作的原子性与高可用性。

-   **Database Self-Healing**: `database.php` 在初始化时，通过 `ensureColumnExists` 方法自动检测并修复/添加缺失的表结构和字段（例如多租户相关的 `app_id` 字段），防止因数据库版本不匹配导致的白屏或错误。
-   **连接池优化**: 针对 SQLite 的文件锁特性，`database.php` 在批量操作 (如 `batchUnbindCards`, `batchAddTime`) 中使用事务 (`beginTransaction`, `commit`, `rollBack`) 机制，确保数据一致性和并发处理效率。
-   **智能风控**: `database.php` 的 `verifyCard` 逻辑自动计算设备指纹 (Device Hash)。系统支持 `database.php` 中的 `cleanupExpiredDevices` 定期清理过期设备，后台管理 (`cards.php`) 支持一键解绑设备 (`batch_unbind`)。
-   **文件系统保护**: `config.php` 在数据目录 `/data` 下自动写入 `.htaccess` 文件 (`Order Deny,Allow\nDeny from all`)，禁止外部直接访问 SQLite 数据库文件，保障数据安全。

### 👨‍💻 5. 极致开发者体验 (DX)
-   **RESTful API**: `Verifyfile/api.php` 提供标准化的 JSON 接口设计，方便各类客户端集成，返回 `code`, `msg`, `data` 结构。
-   **开箱即用前端**: `index.php` 提供功能完善的前端验证页面，并搭配 `js/main.js` 处理卡密格式化、设备指纹生成、LocalStorage 记住卡密等功能，以及友好的提示与动画效果。
-   **多语言示例**: 官方文档 (aegis.可爱.top) 提供 **Python, Java, C#, Go, Node.js, EPL** 等 10+ 种主流语言的 Copy-Paste Ready 调用代码，加速开发进程。
-   **易于部署**: 采用 SQLite3 数据库，无需额外配置数据库服务，文件拷贝即可运行。

---

## 📂 部署架构与目录

v8.0 采用轻量化结构，请确保 `/data` 目录拥有 **777 读写权限**：

```text
/ (Web Root)
├── Verifyfile/           # [核心] 后端辅助功能目录
│   ├── captcha.php       # 验证码图片生成接口 (用于后台登录)
│   └── api.php           # 核心 API 接口 (供外部客户端调用进行验证和获取云变量)
├── backend/              # 后台管理界面的静态资源目录
│   └── logo.png          # 系统 Logo 图片
├── js/                   # 前端验证页面 (index.php) 的 JavaScript 文件
│   └── main.js           # 处理前端卡密验证逻辑、设备指纹生成和记住卡密功能
├── assets/               # 其他前端资源目录
│   ├── css/              # 样式文件 (如 all.min.css, css2.css, remixicon.css)
│   └── js/               # JavaScript 文件 (如 chart.js)
├── data/                 # [核心] 数据库目录 ★ 必须 777 权限
│   ├── cards.db          # SQLite 数据库文件 (系统自动创建和管理)
│   └── .htaccess         # Apache/Nginx 保护配置 (禁止直接访问 .db 文件，自动生成)
├── index.php             # 前端卡密验证页面 (用户在此输入卡密进行验证)
├── index1.php            # 卡密验证成功后跳转的页面 (可自定义展示卡密信息)
├── cards.php             # 后台管理系统主控制台 (管理员登录和管理所有功能)
├── verify.php            # 前端验证页面 (index.php) 的后端处理接口
├── auth_check.php        # 统一身份验证检查文件 (用于保护后台和敏感页面)
├── config.php            # 系统核心配置文件 (包含安全密钥、数据库路径、错误配置等)
└── database.php          # 数据库操作核心类 (封装所有数据库交互逻辑)

Copyright © 2026 GuYi Aegis Pro. All Rights Reserved.
