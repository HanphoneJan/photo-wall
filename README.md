# 🖼️ 寒枫的照片墙 | Hanphone's Photo Wall

[![GitHub Stars](https://img.shields.io/github/stars/HanphoneJan/photo-wall?style=for-the-badge&color=FFD700&logo=github)](https://github.com/HanphoneJan/photo-wall)
[![Online Demo](https://img.shields.io/badge/Online%20Demo-Visit-4169E1?style=for-the-badge&logo=chrome)](https://hanphone.cn/atlas/)

## 📌 项目简介

一个简洁优雅的**个人照片墙应用**，采用便利贴（瀑布流）布局展示精选图片。支持标签分类、互动点赞、智能搜索，让你轻松打造属于自己的视觉空间。

你可以用它展示旅行风景、设计灵感、生活瞬间，或通过用户系统邀请朋友们一起分享精彩照片。无需复杂的社交功能，专注于纯粹的美图展示与收藏体验。

## 🏗️ 项目架构

```
photo-wall/
├── src/                    # 前端Vue3项目源码
│   ├── api/               # API接口配置
│   ├── assets/            # 静态资源
│   ├── components/        # 组件
│   ├── router/            # 路由配置
│   ├── store/             # Pinia状态管理
│   ├── styles/            # 全局样式
│   ├── utils/             # 工具函数
│   └── views/             # 页面视图
├── backend/               # 后端服务
│   ├── nodejs/           # Node.js + Express + PostgreSQL实现
│   └── archive-php/      # PHP + MySQL实现（归档）
├── src-android/          # Android原生客户端
├── docs/                 # 项目文档
├── public/               # 静态资源
└── vite.config.ts        # Vite配置
```

### 🎯 核心定位

- **🖼️ 个人照片墙**：打造专属的视觉空间，展示自己喜欢的图片收藏
- **👥 好友共创**：通过用户系统邀请朋友们一起分享精彩瞬间
- **🎨 双布局设计**：支持便利贴风格和瀑布流风格，自由切换浏览体验
- **🏷️ 标签管理**：灵活的多标签分类，轻松组织和筛选照片
- **❤️ 轻量互动**：点赞收藏、访问统计，简单的互动反馈
- **📱 多端访问**：Web在线浏览 + PWA离线体验 + Android WebView封装
- **🔒 安全可靠**：JWT认证保护，完善的权限管理，安心分享

### 🎨 页面设计风格

项目采用**新野兽派（Neo-Brutalism）**设计语言，强调原始、直白、富有个性的视觉表达：

- **高对比色彩**：鲜艳的便利贴配色搭配深色边框，视觉冲击力强
- **粗粝边框**：4px实线边框，营造手工质感
- **硬阴影效果**：偏移阴影增加立体感，复古印刷风格
- **胶带装饰**：模拟真实便利贴的胶带固定效果
- **几何元素**：背景装饰性几何图形，增强空间层次

### 📐 双布局模式

照片墙提供两种独特的图片展示布局，满足不同浏览偏好：

#### 1️⃣ 便利贴布局（默认）

**特点**：
- 彩色便利贴卡片设计，8种鲜艳配色循环使用
- 每张便利贴带有轻微旋转角度，模拟真实张贴效果
- 胶带装饰元素，增强手工感
- 硬阴影和粗边框，新野兽派风格
- 网格背景 + 噪点纹理，营造复古氛围

**适用场景**：强调个性和创意展示，适合艺术作品、设计灵感类内容

#### 2️⃣ 瀑布流布局

**特点**：
- 经典的 Masonry 瀑布流排列
- 极简主义设计，无间隙图片展示
- 悬停显示图片信息和操作按钮
- 渐变背景 + 微妙网格纹理

**适用场景**：追求沉浸式浏览体验，适合摄影作品、风景图片类内容

### 🚀 技术架构栈

#### 🔹 前端体系 | Frontend

<p align="left">
  <img src="https://img.shields.io/badge/Vue3-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white" alt="Vue3">
  <img src="https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white" alt="Vite">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/ElementPlus-1989FA?style=for-the-badge&logo=element&logoColor=white" alt="ElementPlus">
  <img src="https://img.shields.io/badge/Pinia-FFD859?style=for-the-badge&logo=pinia&logoColor=black" alt="Pinia">
  <img src="https://img.shields.io/badge/PWA-5A0FC8?style=for-the-badge&logo=pwa&logoColor=white" alt="PWA">
</p>

- **核心框架**：Vue 3.5 + Composition API + TypeScript
- **构建工具**：Vite 6.0（秒级热更新，优化构建性能）
- **状态管理**：Pinia 2.3（轻量级、TypeScript友好）
- **UI组件库**：Element Plus 2.9（响应式设计，适配多终端）
- **增强特性**：
  - PWA渐进式应用：Service Worker离线缓存 + Web Manifest桌面图标
  - 图片预览：v-viewer集成
  - 国际化：vue-i18n支持
  - 性能优化：Vite异步模块拆包、图片懒加载

#### 🔹 后端体系 | Backend

<p align="left">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white" alt="Express">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white" alt="JWT">
</p>

- **基础框架**：Node.js + Express 5.1
- **数据存储**：PostgreSQL（生产环境）/ MySQL（PHP版本）
- **鉴权方案**：JWT + Session 双模式
- **核心功能**：图片管理、标签系统、点赞互动、访问统计、后台管理

## 📖 文档导航

| 文档 | 说明 |
|------|------|
| [📘 前端技术文档](./docs/FRONTEND.md) | Vue3项目结构、开发规范、组件说明 |
| [📗 后端技术文档](./docs/BACKEND.md) | Node.js服务架构、数据库设计、部署指南 |
| [📙 接口文档](./docs/API.md) | 完整API接口定义、请求/响应格式 |

## 🚀 快速开始

### 环境要求

- Node.js 18+
- PostgreSQL 14+（或 MySQL 5.7+）
- pnpm / npm

### 1. 克隆项目

```bash
git clone https://github.com/HanphoneJan/photo-wall.git
cd photo-wall
```

### 2. 启动前端

```bash
# 安装依赖
pnpm install

# 开发模式
pnpm dev

# 构建生产环境
pnpm build
```

### 3. 启动后端

```bash
cd backend/nodejs

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env
# 编辑 .env 配置数据库连接

# 启动服务
npm start
```

默认端口：前端 `3000`，后端 `4001`

## ✨ 核心功能

| 功能 | 说明 |
|------|------|
| 📤 **图片上传** | 支持批量添加图片，轻松扩充你的照片墙 |
| 🖼️ **瀑布流展示** | 自适应瀑布流布局，支持图片预览、懒加载 |
| 🏷️ **标签系统** | 多标签分类，支持标签筛选和检索 |
| 🔍 **智能搜索** | 按标题、作者关键词快速查找图片 |
| ❤️ **点赞互动** | 用户点赞收藏，实时显示点赞状态 |
| 📊 **访问统计** | 记录访问量，了解照片墙浏览情况 |
| 🛠️ **后台管理** | 图片审核发布、标签管理、用户权限控制 |
| 📱 **PWA支持** | 离线访问、添加到主屏幕、类原生应用体验 |

## 📂 目录结构详解

```
src/
├── api/                      # API接口配置
│   ├── api.ts               # 接口地址定义
│   └── interceptor.ts       # 请求拦截器
├── assets/                   # 静态资源
├── components/               # Vue组件
│   ├── Atlas.vue            # 照片墙主组件
│   ├── admin/               # 后台管理组件
│   ├── layout/              # 布局组件
│   └── login/               # 登录相关组件
├── router/
│   └── router.ts            # 路由配置
├── store/
│   └── store.ts             # Pinia状态管理
├── styles/                   # 全局样式
├── utils/                    # 工具函数
├── views/                    # 页面视图
│   ├── Home.vue             # 首页
│   └── AdminHome.vue        # 管理后台
├── App.vue                   # 根组件
└── main.ts                   # 入口文件
```

## 🎨 项目特点

- 🎯 **专注个人场景**：摒弃复杂功能，专注于个人图片展示与管理
- 🔄 **前后端分离**：Vue3 + Node.js 架构，接口清晰易于扩展
- 📱 **多端适配**：Web、PWA、Android WebView 全覆盖
- 🔒 **安全可靠**：JWT 鉴权、参数化查询防注入、XSS 防护
- ⚡ **性能优化**：图片懒加载、Vite 快速构建、Workbox 缓存策略
- 🐳 **部署友好**：支持 Docker、PM2、Nginx 等多种部署方式
## Star History

<a href="https://www.star-history.com/?repos=HanphoneJan%2Fphoto-wall&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/image?repos=HanphoneJan/photo-wall&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/image?repos=HanphoneJan/photo-wall&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/image?repos=HanphoneJan/photo-wall&type=date&legend=top-left" />
 </picture>
</a>
