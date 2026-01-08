# 🖼️ 寒枫的照片墙 | Hanphone's Photo Wall

[![GitHub Stars](https://img.shields.io/github/stars/HanphoneJan/photo-wall?style=for-the-badge&color=FFD700&logo=github)](https://github.com/HanphoneJan/photo-wall)
[![Online Demo](https://img.shields.io/badge/Online%20Demo-Visit-4169E1?style=for-the-badge&logo=chrome)](https://www.hanphone.top/atlas/)

## 📌 项目简介

一款集 **图片收集、可视化展示、跨端分享** 于一体的个人照片墙应用，专注于打造简洁优雅的图片管理体验，支持Web端与Android端访问，让每一张照片都有专属展示空间～

## 🏗️ 项目架构



### 🎯 核心定位

- 个人图片资产管理：集中存储、分类整理
- 可视化展示：高颜值画廊布局，支持多维度浏览
- 跨端分享：Web端在线访问 + PWA APP离线查看
- 轻量高效：无冗余功能，专注核心体验

### 🚀 技术架构栈

#### 🔹 前端体系 | Frontend

<p align="left">
  <img src="https://img.shields.io/badge/Vue3-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white" alt="Vue3">
  <img src="https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white" alt="Vite">
  <img src="https://img.shields.io/badge/ElementPlus-1989FA?style=for-the-badge&logo=element&logoColor=white" alt="ElementPlus">
  <img src="https://img.shields.io/badge/PWA-5A0FC8?style=for-the-badge&logo=pwa&logoColor=white" alt="PWA">
</p>


- 核心框架：Vue 3.2 + Composition API（高效组件复用与状态管理）
- 构建工具：Vite 4.0（秒级热更新，优化构建性能）
- UI组件库：Element-Plus 2.3（响应式设计，适配多终端）
- 增强特性：
  - PWA渐进式应用：Service Worker离线缓存 + Web Manifest桌面图标
  - 性能优化：Vite异步模块拆包、图片懒加载、CDN加速
  - 交互体验：平滑滚动、图片预览、拖拽排序

#### 🔹 服务端架构 | Backend

<p align="left">
  <img src="https://img.shields.io/badge/PHP8.1-777BB4?style=for-the-badge&logo=php&logoColor=white" alt="PHP">
  <img src="https://img.shields.io/badge/MySQL8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL">
  <img src="https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white" alt="JWT">
</p>


- 基础框架：PHP 8.1 + MVC架构（职责分离，易于维护）
- 接口设计：RESTful API（规范统一，支持前后端分离）
- 身份认证：JWT鉴权（无状态登录，保障接口安全）
- 数据存储：MySQL 8.0（关系型数据库，优化图片元数据存储）
- 功能模块：图片上传/压缩、分类管理、权限控制、访问统计

#### 🔹 移动端封装 | Mobile

<p align="left">
  <img src="https://img.shields.io/badge/Android-3DDC84?style=for-the-badge&logo=android&logoColor=white" alt="Android">
  <img src="https://img.shields.io/badge/WebView-000000?style=for-the-badge&logo=googlechrome&logoColor=white" alt="WebView">
</p>


- 封装方案：Android WebView（复用Web端功能，降低开发成本）
- 增强特性：
  - 离线缓存：支持图片本地存储，无网络时查看
  - 原生交互：调用相机/相册上传图片，适配系统权限
  - 沉浸式体验：全屏展示，优化移动端操作逻辑

## 📱 跨端支持

| 平台      | 访问方式                                              | 核心特性                           |
| --------- | ----------------------------------------------------- | ---------------------------------- |
| 🌐 Web端   | [www.hanphone.top/atlas](https://www.hanphone.top/atlas/) | 在线浏览、高清预览、分享链接       |
| 📱 Android | WebView封装APP                                        | 离线缓存、原生上传、沉浸式全屏展示 |

## ✨ 核心功能

1. 📤 图片管理：支持批量上传、标签检索
2. 🎨 可视化展示：网格布局、瀑布流、幻灯片播放模式
3. 📱 响应式设计：适配PC、平板、手机等多终端设备
4. 🚀 性能优化：图片懒加载、渐进式加载、缓存策略
5. 🔒 权限控制：私有相册加密、公开相册分享
6. 📥 离线访问：Android端支持图片本地缓存，无网络可看

## 🎨 设计理念

- 极简美学：以图片为核心，减少冗余UI元素
- 交互流畅：操作逻辑简单直观，注重细节体验
- 性能优先：优化加载速度，保障跨设备流畅运行

## 🐛 问题反馈

若使用过程中遇到bug或有功能建议，欢迎通过以下方式反馈：

- GitHub Issues：[提交反馈](https://github.com/HanphoneJan/photo-atlas/issues)
- 在线演示：[访问照片墙](https://www.hanphone.top/atlas/)  当前该在线演示已进行更新，但未同步到该项目

---

<p align="center">
  <sub>🌟 觉得不错？欢迎 Star 支持一下～</sub>
</p>
<p align="center">
  <img src="https://komarev.com/ghpvc/?username=HanphoneJan&label=Profile%20Views&color=brightgreen&style=flat-square" alt="Profile Views">
</p>