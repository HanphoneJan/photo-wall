<template>
  <!-- 汉堡    摄影师摄影师师照片墙墙风格导航组件
    设计特点：
    - 采用暗色调基础，突出照片内容
    - 融入胶片、暗房元素的视觉语言
    - 简约而有质感的交互效果
    - 支持明暗主题切换，适应不同照片风格
  -->
  
  <!-- 汉堡菜单按钮 - 模拟相机快门设计 -->
  <el-button 
    class="hamburger-button" 
    :class="{ 'hamburger-button-hidden': !showHamburger }"
    @click="toggleSidebar" 
    circle
  >
    <el-icon :size="20">
      <Expand v-if="!sidebarOpen" />
      <Fold v-else />
    </el-icon>
  </el-button>
  
  <!-- 侧边栏 - 模拟相机侧边控制面板 -->
  <div class="sidebar" :class="{ 'sidebar-open': sidebarOpen }">
    <div class="sidebar-content">
      <!-- 品牌标识 - 摄影工作室风格 -->
      <div class="studio-logo">
        <span class="logo-text">Hanphone</span>
        <span class="logo-subtext">GALLERY</span>
      </div>
      
      <!-- 顶部导航区域 -->
      <div class="top-navigation">
        <!-- 搜索框 - 模拟相机对焦环设计 -->
        <div class="search-container">
          <el-icon class="search-icon" :size="18">
            <Search />
          </el-icon>
          <el-input
            @focus="checkInput"
            @blur="notSearching()"
            class="search-input"
            placeholder="搜索照片..."
            v-model="searchInput"
            @input="searchItems" 
            size="small"
            clearable
          ></el-input>
          
          <!-- 搜索结果 - 模拟胶片负片效果 -->
          <ul v-if="!searching && searchList.length" class="search-suggestions">
            <li
              v-for="item in searchList"
              :key="item.id"
              @click="getItemInfo(item.path)"
              class="search-item"
            >
              <span class="item-title">{{ item.title }}</span>
              <span class="item-author">by {{ item.author }}</span>
            </li>
          </ul>
        </div>
        
        <!-- 导航菜单 -->
        <el-menu 
          :default-active="activeIndex" 
          class="sidebar-menu" 
          @select="handleSelect"
          background-color="transparent"
        >
          <el-menu-item index="1">
            <el-icon slot="icon" :size="18">
              <Picture />
            </el-icon>
            照片墙
          </el-menu-item>
          <el-menu-item index="2">
            <el-icon slot="icon" :size="18">
              <Upload />
            </el-icon>
            上传作品
          </el-menu-item>
          <el-menu-item index="3">
            <el-icon slot="icon" :size="18">
              <Document />
            </el-icon>
            我的博客
          </el-menu-item>
          <el-menu-item index="4" v-show="userStore.isAdmin">
            <el-icon slot="icon" :size="18">
              <Setting />
            </el-icon>
            管理照片墙
          </el-menu-item>
        </el-menu>
        
        <!-- 标签过滤器 -->
        <div class="tag-filter-section">
          <div class="tag-filter-header">
            <div class="filter-title">
              <el-icon><Filter /></el-icon>
              <span>筛选标签</span>
            </div>
            <el-button 
              class="toggle-filter-btn" 
              @click="toggleTagFilter" 
              circle
              :class="{ 'active': showTagFilter }"
            >
              <el-icon>
                <ArrowUp v-if="showTagFilter" />
                <ArrowDown v-else />
              </el-icon>
            </el-button>
          </div>
          
          <transition name="slide-down">
            <div v-show="showTagFilter" class="tag-filter">
              <div class="tags-container">
                <el-tag
                  v-for="(tag, index) in sortedTags"
                  :key="tag.id"
                  :type="selectedTags.includes(tag.id) ? 'primary' : 'info'"
                  size="default"
                  :closable="selectedTags.includes(tag.id)"
                  @click="toggleTag(tag.id)"
                  @close="handleClose(index)"
                  class="tag-item"
                  clearable="false"
                >
                  {{ tag.name }}
                </el-tag>
              </div>
            </div>
          </transition>
        </div>
      </div>
      
      <!-- 底部功能区 -->
      <div class="sidebar-actions">
        <!-- 未登录状态 -->
        <template v-if="!userStore.isLoggedIn">
          <el-button class="login-button" @click="handleLogin">
            登录
          </el-button>
        </template>
        
        <!-- 已登录状态 -->
        <template v-else>
          <div class="user-avatar" @click="handleProfile">
            <img :src="userStore.avatar" alt="用户头像" v-if="userStore.avatar">
            <el-icon v-else><User /></el-icon>
          </div>
          <el-button class="logout-button" @click="handleLogout">
            退出
          </el-button>
        </template>
        
        <el-button class="theme-toggle-button" @click="toggleTheme" circle>
          <el-icon :size="18">
            <Moon v-if="userStore.theme === 'dark'" />
            <Sunny v-else />
          </el-icon>
        </el-button>
      </div>
    </div>
  </div>
  
  <!-- 遮罩层 - 模拟镜头渐变效果 -->
  <div class="overlay" :class="{ 'overlay-visible': sidebarOpen }" @click="closeSidebar"></div>
</template>

<script setup lang="ts">
import { ref, watchEffect, computed, onMounted, onUnmounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { useUserStore } from '@/store/store';
import api from '@/api/interceptor';
import { ENDPOINTS } from '@/api/api';
import { 
  Expand, Fold, Picture, Upload, Document, 
  Setting, User, Moon, Sunny, Filter, ArrowUp, ArrowDown, Search
} from '@element-plus/icons-vue';

// 标签接口定义
interface Tag {
  id: number | null;
  name: string;
}

const userStore = useUserStore();
const activeIndex = ref('1'); 
const router = useRouter();
const route = useRoute();
const searching = ref(false);
const searchInput = ref('');
const searchList = ref([]);
const sidebarOpen = ref(false);

// 标签相关状态
const tagsList = ref<Tag[]>([]);
const selectedTags = ref<number[]>([]);
const showTagFilter = ref(true);

// 汉堡按钮显示状态
const showHamburger = ref(true);
let inactivityTimer: number | null = null;
const INACTIVITY_TIMEOUT = 3000; // 3秒无交互后隐藏按钮

// 重置不活动计时器
const resetInactivityTimer = () => {
  // 清除之前的计时器
  if (inactivityTimer !== null) {
    clearTimeout(inactivityTimer);
  }
  
  // 显示汉堡按钮
  showHamburger.value = true;
  
  // 设置新的计时器
  inactivityTimer = window.setTimeout(() => {
    // 如果侧边栏没有打开，则隐藏汉堡按钮
    if (!sidebarOpen.value) {
      showHamburger.value = false;
    }
  }, INACTIVITY_TIMEOUT);
};

// 监听用户活动
const handleUserActivity = () => {
  resetInactivityTimer();
};

// 切换侧边栏
const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value;
  // 如果打开侧边栏，确保汉堡按钮可见
  if (sidebarOpen.value) {
    showHamburger.value = true;
  }
  resetInactivityTimer();
};

// 关闭侧边栏
const closeSidebar = () => {
  sidebarOpen.value = false;
  resetInactivityTimer();
};

// 监听主题状态变化
watchEffect(() => {
  const htmlClassList = document.documentElement.classList
  if (userStore.theme === 'dark') {
    htmlClassList.add('dark') 
  } else {
    htmlClassList.remove('dark')
  }
});

// 图集项接口定义
interface AtlasItem {
  id: number;
  path:string;
  author: string;
  title: string;
}

const routeMap = {
  '1': '/index',
  '2': '/upload',
  '3': '/blog',
  '4': '/admin',
};

// 处理导航选择事件
const handleSelect = (key) => {
  const route = routeMap[key];
  const pageName = route.slice(1);
  if (route) {
    router.push(route);
    userStore.changePage(pageName);
    closeSidebar();
  } else {
    console.warn(`没有找到路由: ${key}`);
  }
  resetInactivityTimer();
};

const handleLogin = () => {
  router.push('/login');
  closeSidebar();
  resetInactivityTimer();
};

// 处理用户资料点击
const handleProfile = () => {
  router.push('/profile');
  closeSidebar();
  resetInactivityTimer();
};

// 处理退出登录
const handleLogout = () => {
  userStore.logout();
  router.push('/index');
  closeSidebar();
  resetInactivityTimer();
};

// 切换主题
const toggleTheme = () => {
  userStore.changeTheme(userStore.theme === 'dark' ? 'light' : 'dark');
  resetInactivityTimer();
};

// 实时搜索
const searchItems = async () => {
  resetInactivityTimer();
  
  if (!searchInput.value.trim()) {
    searchList.value = [];
    searching.value = false;
    return;
  }
  
  searching.value = true; 
  try {
    searchList.value = await fetchSearchResults(searchInput.value);
  } catch (error) {
    console.error("搜索错误:", error.message);
    searchList.value = [];
  } finally {
    searching.value = false;
  }
};

// 搜索 API 请求
const fetchSearchResults = async (query: string) => {
  try {
    const response = await api.post(ENDPOINTS.SEARCH, { query });
    if (response.data.status === 830) {
      return Array.isArray(response.data.data) ? response.data.data : [];
    }
    return [];
  } catch (error) {
    console.error('获取图集数据失败:', error.message);
    return [];
  }
};

// 获取单个项目详细信息
const getItemInfo = (path: string) => {
  window.location.href = path;
  searching.value = false;
  searchList.value = [];
  closeSidebar();
  resetInactivityTimer();
};

// 处理输入框聚焦
const checkInput = () => {
  searching.value = true;
  resetInactivityTimer();
};

// 处理输入框失去焦点
let timeout;
const notSearching = () => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    searching.value = false;
    searchList.value = [];
  }, 200);
  resetInactivityTimer();
};

// 切换标签的选中状态
const toggleTag = (tagId) => {
  if (selectedTags.value.includes(tagId)) {
    selectedTags.value = selectedTags.value.filter((id) => id !== tagId);
  } else {
    selectedTags.value.push(tagId);
  }
  // 更新URL参数
  updateUrlTags();
  resetInactivityTimer();
};

// 关闭标签（仅限于选中的标签）
const handleClose = (index) => {
  selectedTags.value.splice(index, 1);
  // 更新URL参数
  updateUrlTags();
  resetInactivityTimer();
};

// 切换标签过滤器显示/隐藏
const toggleTagFilter = () => {
  showTagFilter.value = !showTagFilter.value;
  resetInactivityTimer();
};

// 更新URL中的标签参数
const updateUrlTags = () => {
  const currentQuery = { ...route.query };
  
  if (selectedTags.value.length > 0) {
    currentQuery.tags = selectedTags.value.join(',');
  } else {
    delete currentQuery.tags;
  }
  
  router.replace({ query: currentQuery });
};

// 获取标签列表
const getTag = async()=>{
  try {
    // 向后端请求图集数据
    const response = await api.get(ENDPOINTS.GETTAG);
    if (response.data.status === 830) {
      tagsList.value = response.data.data;
    }
  } catch (error) {
    console.error('获取标签列表失败:', error.message);
  }
}

// 组件挂载时执行
onMounted(async () => {
  // 获取标签列表
  getTag();
  // 从URL参数中获取选中的标签
  const tagsFromUrl = route.query.tags as string;
  if (tagsFromUrl) {
    selectedTags.value = tagsFromUrl.split(',').map(id => Number(id));
  }
  
  // 初始化不活动计时器
  resetInactivityTimer();
  
  // 添加全局事件监听器来跟踪用户活动
  document.addEventListener('mousemove', handleUserActivity);
  document.addEventListener('keypress', handleUserActivity);
  document.addEventListener('scroll', handleUserActivity);
  document.addEventListener('touchstart', handleUserActivity);
});

// 组件卸载时清理
onUnmounted(() => {
  // 清除计时器
  if (inactivityTimer !== null) {
    clearTimeout(inactivityTimer);
  }
  
  // 移除事件监听器
  document.removeEventListener('mousemove', handleUserActivity);
  document.removeEventListener('keypress', handleUserActivity);
  document.removeEventListener('scroll', handleUserActivity);
  document.removeEventListener('touchstart', handleUserActivity);
});

// 计算排序后的标签列表：选中的标签排在前面
const sortedTags = computed(() => {
  return [
    ...tagsList.value.filter((tag) => selectedTags.value.includes(tag.id)),
    ...tagsList.value.filter((tag) => !selectedTags.value.includes(tag.id)),
  ];
});
</script>

<style scoped>
/* 基础样式变量 */
:root {
  --photo-primary: #2d3436;
  --photo-secondary: #636e72;
  --photo-accent: #0984e3;
  --photo-light: #dfe6e9;
  --photo-dark: #2d3436;
  --transition-standard: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* 汉堡菜单按钮 - 相机快门风格 */
.hamburger-button {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1001;
  width: 44px;
  height: 44px;
  background-color: transparent;
  color: #111827;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border:none;
  opacity: 1;
  transform: scale(1);
}

.hamburger-button-hidden {
  opacity: 0;
  transform: scale(0.8);
  pointer-events: none;
}

.hamburger-button:hover {
  background-color: var(--photo-accent);
  transform: scale(1.05);
}

/* 侧边栏 - 相机控制面板风格 */
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  width: 280px;
  height: 100vh;
  background-color: #f8f9fa;
  transform: translateX(-100%);
  /* 更明确的过渡设置 */
  transition: transform 0.3s ease-in-out, opacity 0.3s ease-in-out;
  z-index: 1000;
  overflow-y: auto;
  box-shadow: 3px 0 15px rgba(0, 0, 0, 0.1);
  /* 添加初始透明度，使过渡更平滑 */
  opacity: 0;
}

.sidebar-open {
  transform: translateX(0);
  opacity: 1;
}

.sidebar-content {
  padding: 30px 20px;
  display: flex;
  flex-direction: column;
  gap: 15px; /* 减小了整体间距 */
  height: 100%;
  box-sizing: border-box;
}

/* 工作室标识 */
.studio-logo {
  text-align: center;
  padding: 10px 0 15px; /* 减小了底部间距 */
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.logo-text {
  font-size: 28px;
  font-weight: 900;
  letter-spacing: -1px;
  color: #1a1a1a;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.logo-subtext {
  display: block;
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 3px;
  color: #4a4a4a;
  margin-top: 5px;
}

/* 顶部导航区域 */
.top-navigation {
  display: flex;
  flex-direction: column;
  gap: 15px; /* 减小了间距 */
  flex-grow: 1;
}

/* 搜索区域 - 相机镜头风格 */
.search-container {
  position: relative;
  margin: 5px 0; /* 减小了上下边距 */
}

.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #4a4a4a;
  z-index: 1;
}

/* 移除输入框默认边框 */
:deep(.search-input .el-input__wrapper) {
  border: none !important;
  box-shadow: none !important;
  background-color: transparent !important;
}

/* 聚焦时也不显示边框（可选，根据需要保留或移除） */
:deep(.search-input .el-input__wrapper:focus-within) {
  border: none !important;
  box-shadow: none !important;
}
.search-input {
  padding-left: 40px !important;
  border-radius: 20px;
  background-color: #ffffff;
  border: 1px solid #d1d5db;
  transition: var(--transition-standard);
  height: 40px;
  color: #1a1a1a;
  font-weight: 500;
}

.search-input:focus {
  background-color: white;
  box-shadow: 0 0 0 1px rgba(9, 132, 227, 0.2);
  border-color: #0984e3;
}

/* 搜索建议 - 胶片风格 */
.search-suggestions {
  list-style: none;
  padding: 10px 0;
  margin: 5px 0 0; /* 减小了上边距 */
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  box-shadow: 0 5px 5px rgba(0, 0, 0, 0.1);
  max-height: 300px;
  overflow-y: auto;
  z-index: 10;
  border-radius: 8px;
  background-color: white;
  border: 1px solid #d1d5db;
}

.search-item {
  padding: 12px 15px;
  font-size: 14px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 12px;
  transition: var(--transition-standard);
}

.search-item:hover {
  background-color: #f6f8fa;
  transform: translateX(4px);
}

.item-thumbnail {
  width: 36px;
  height: 24px;
  background-color: #f1f2f6;
  border-radius: 3px;
  flex-shrink: 0;
  background-image: linear-gradient(45deg, #e2e2e2 25%, transparent 25%),
                    linear-gradient(-45deg, #e2e2e2 25%, transparent 25%),
                    linear-gradient(45deg, transparent 75%, #e2e2e2 75%),
                    linear-gradient(-45deg, transparent 75%, #e2e2e2 75%);
  background-size: 10px 10px;
  background-position: 0 0, 0 5px, 5px -5px, -5px 0px;
}

.item-title {
  font-weight: 500;
  color: #1a1a1a;
}

.item-author {
  font-size: 12px;
  color: #4a4a4a;
  margin-left: auto;
}

/* 标签过滤器区域 */
.tag-filter-section {
  margin: 5px 0; /* 减小了上下边距 */
}

.tag-filter-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px; /* 减小了底部间距 */
}

.filter-title {
  display: flex;
  align-items: center;
  font-weight: 500;
  font-size: 16px;
  color: #1a1a1a;
}

.filter-title .el-icon {
  margin-right: 8px;
  color: #0984e3;
}

.toggle-filter-btn {
  background-color: #f8f9fa;
  border: 1px solid #d1d5db;
  color: #4a4a4a;
  transition: var(--transition-standard);
}

.toggle-filter-btn:hover {
  border-color: #0984e3;
}


/* 标签选择器容器 */
.tag-filter {
  padding: 8px 0; /* 减小了上下内边距 */
}

/* 标签选择器样式 */
.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding: 6px 0; /* 减小了上下内边距 */
}

/* 单个标签样式 */
.tag-item {
  cursor: pointer;
  transition: var(--transition-standard);
  font-size: 14px;
  padding: 6px 12px;
  border-radius: 16px;
}
.tag-item:hover {
  transform: scale(1.05);
}
.tag-item.el-tag--primary {
  background-color: rgba(9, 132, 227, 0.1);
  color: #0984e3;
  border: 1px solid #0984e3;
}
.tag-item.el-tag--info {
  background-color: #f1f2f6;
  color: #1a1a1a;
  border: 1px solid #d1d5db;
}

/* 导航菜单 */
.sidebar-menu {
  border: none;
  background-color: transparent;
  flex-grow: 1;
}

.sidebar-menu .el-menu-item {
  height: 50px;
  line-height: 50px;
  font-size: 15px;
  margin: 6px 0; /* 减小了上下边距 */
  border-radius: 6px;
  color: #1a1a1a;
  transition: var(--transition-standard);
  padding-left: 20px !important;
  font-weight: 500;
}

.sidebar-menu .el-menu-item:hover,
.sidebar-menu .el-menu-item.is-active {
  background-color: rgba(9, 132, 227, 0.1);
  color: #0984e3;
}

/* 底部操作区 */
.sidebar-actions {
  display: flex;
  gap: 10px;
  margin-top: auto;
  padding-top: 15px; /* 减小了顶部内边距 */
  border-top: 1px solid rgba(0, 0, 0, 0.05);
  align-items: center;
}

.login-button {
  flex-grow: 1;
  border-radius: 20px;
  background-color: #1a1a1a;
  color: white;
  border: none;
  height: 40px;
  transition: var(--transition-standard);
  justify-content: center;
  gap: 8px;
  font-weight: 500;
}

.login-button:hover {
  background-color: #0984e3;
}

.logout-button {
  flex-grow: 1;
  border-radius: 20px;
  background-color: #f44336;
  color: white;
  border: none;
  height: 40px;
  transition: var(--transition-standard);
  justify-content: center;
  gap: 8px;
  font-weight: 500;
}

.logout-button:hover {
  background-color: #d32f2f;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f1f2f6;
  border: 1px solid #d1d5db;
  cursor: pointer;
  transition: var(--transition-standard);
  flex-shrink: 0;
}

.user-avatar:hover {
  border-color: #0984e3;
}

.user-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.user-avatar .el-icon {
  color: #4a4a4a;
  font-size: 20px;
}

.theme-toggle-button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #ffffff;
  color: #4a4a4a;
  border: 1px solid #d1d5db;
  transition: var(--transition-standard);
  flex-shrink: 0;
}

.theme-toggle-button:hover {
  background-color: #4a4a4a;
  color: white;
}

/* 遮罩层 - 渐变效果 */
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 30% 30%, rgba(0,0,0,0.2) 0%, rgba(0,0,0,0.5) 100%);
  z-index: 999;
  opacity: 0;
  visibility: hidden;
  transition: var(--transition-standard);
}

.overlay-visible {
  opacity: 1;
  visibility: visible;
}

/* 向下滑动动画 */
.slide-down-enter-active, .slide-down-leave-active {
  transition: all 0.3s ease;
  overflow: hidden;
}

.slide-down-enter-from, .slide-down-leave-to {
  max-height: 0;
  opacity: 0;
}

.slide-down-enter-to, .slide-down-leave-from {
  max-height: 300px;
  opacity: 1;
}

/* 暗黑主题 - 暗房风格 */
.dark {
  --photo-primary: #f1f2f6;
  --photo-secondary: #b2bec3;
  --photo-accent: #74b9ff;
  --photo-light: #2d3436;
  --photo-dark: #f8f9fa;
}

.dark .sidebar {
  background-color: #1a1a1a;
}

.dark .studio-logo {
  border-bottom-color: rgba(255, 255, 255, 0.05);
}

.dark .logo-text {
  color: #f1f2f6;
}

.dark .logo-subtext {
  color: #b2bec3;
}

.dark .search-input {
  background-color: #2d3436;
  color: #f1f2f6;
  border-color: #444;
}

.dark .search-input:focus {
  background-color: #2d3436;
  border-color: #74b9ff;
}

.dark .hamburger-button {
  color: black;
}

.dark .search-icon {
  color: #b2bec3;
}

.dark .search-suggestions {
  background-color: #2d3436;
  border-color: #444;
}

.dark .search-item:hover {
  background-color: #393e46;
}

.dark .item-title {
  color: #f1f2f6;
}

.dark .item-author {
  color: #b2bec3;
}

.dark .item-thumbnail {
  background-color: #393e46;
  background-image: linear-gradient(45deg, #4a4f57 25%, transparent 25%),
                    linear-gradient(-45deg, #4a4f57 25%, transparent 25%),
                    linear-gradient(45deg, transparent 75%, #4a4f57 75%),
                    linear-gradient(-45deg, transparent 75%, #4a4f57 75%);
}

.dark .filter-title {
  color: #f1f2f6;
}

.dark .toggle-filter-btn {
  background-color: #2d3436;
  border-color: #444;
  color: #b2bec3;
}

.dark .toggle-filter-btn:hover {
  border-color: #74b9ff;
  color: #74b9ff;
}

.dark .toggle-filter-btn.active {
  color: #74b9ff;
}

.dark .tag-item.el-tag--primary {
  background-color: rgba(116, 185, 255, 0.1);
  color: #74b9ff;
  border-color: #74b9ff;
}

.dark .tag-item.el-tag--info {
  background-color: #393e46;
  color: #f1f2f6;
  border-color: #444;
}

.dark .sidebar-menu .el-menu-item {
  color: #f1f2f6;
}

.dark .sidebar-menu .el-menu-item:hover,
.dark .sidebar-menu .el-menu-item.is-active {
  background-color: rgba(116, 185, 255, 0.1);
  color: #74b9ff;
}

.dark .sidebar-actions {
  border-top-color: rgba(255, 255, 255, 0.05);
}

.dark .login-button {
  background-color: #2d3436;
  color: #f1f2f6;
}

.dark .login-button:hover {
  background-color: #74b9ff;
}

.dark .logout-button {
  background-color: #c62828;
}

.dark .logout-button:hover {
  background-color: #b71c1c;
}

.dark .user-avatar {
  background-color: #2d3436;
  border-color: #444;
}

.dark .user-avatar:hover {
  border-color: #74b9ff;
}

.dark .user-avatar .el-icon {
  color: #b2bec3;
}

.dark .theme-toggle-button {
  background-color: #2d3436;
  border-color: #444;
  color: #b2bec3;
}

.dark .theme-toggle-button:hover {
  background-color: #b2bec3;
  color: #1a1a1a;
}

/* 响应式设计 */
@media screen and (max-width: 768px) {
  .sidebar {
    width: 85%;
    max-width: 300px;
  }
}
</style>