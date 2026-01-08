<template>
  <!-- 管理员界面侧边导航 - 摄影工作室风格 -->
  
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
        <span class="logo-subtext">ADMIN</span>
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
          管理照片
        </el-menu-item>
        <el-menu-item index="2">
          <el-icon slot="icon" :size="18">
            <CollectionTag />
          </el-icon>
          管理标签
        </el-menu-item>
        <el-menu-item index="3">
          <el-icon slot="icon" :size="18">
            <UserFilled />
          </el-icon>
          管理用户
        </el-menu-item>
        <el-menu-item index="4">
          <el-icon slot="icon" :size="18">
            <Picture />
          </el-icon>
          照片墙
        </el-menu-item>
        <el-menu-item index="5">
          <el-icon slot="icon" :size="18">
            <Upload />
          </el-icon>
          上传
        </el-menu-item>
      </el-menu>
      
      <!-- 搜索框 - 模拟相机对焦环设计 -->
      <div class="search-container">
        <el-icon class="search-icon" :size="18">
          <Search />
        </el-icon>
        <el-input
          @focus="checkInput"
          @blur="notSearching()"
          class="search-input"
          placeholder="搜索内容..."
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
      
      <!-- 底部功能区 -->
      <div class="sidebar-actions">
        <el-button class="logout-button" @click="handleLogout">
          <el-icon :size="16" class="btn-icon">
            <SwitchButton />
          </el-icon>
        </el-button>
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
import { ref, watchEffect, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { useUserStore } from '@/store/store';
import api from '@/api/interceptor';
import { ENDPOINTS } from '@/api/api';
import { 
  Expand, Fold, Picture, Upload, UserFilled, CollectionTag, 
  SwitchButton, Moon, Sunny, Search
} from '@element-plus/icons-vue';

const userStore = useUserStore();
const activeIndex = ref('1');
const router = useRouter();
const searching = ref(false);
const searchInput = ref('');
const searchList = ref([]);
const sidebarOpen = ref(false);

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
  '1': '/admin/atlas',
  '2': '/admin/tag',
  '3': '/admin/user',
  '4': '/index',
  '5': '/upload',
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

// 处理用户登出操作
const handleLogout = () => {
  userStore.logout();
  router.push('/');
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

// 组件挂载时执行
onMounted(() => {
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
  transition: transform 0.3s ease-in-out, opacity 0.3s ease-in-out;
  z-index: 1000;
  overflow-y: auto;
  box-shadow: 3px 0 15px rgba(0, 0, 0, 0.1);
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
  gap: 20px;
  height: 100%;
  box-sizing: border-box;
}

/* 工作室标识 */
.studio-logo {
  text-align: center;
  padding: 10px 0 20px;
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
  margin: 8px 0;
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

/* 搜索区域 - 相机镜头风格 */
.search-container {
  position: relative;
  margin: 10px 0;
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
  margin: 8px 0 0;
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

.item-title {
  font-weight: 500;
  color: #1a1a1a;
}

.item-author {
  font-size: 12px;
  color: #4a4a4a;
  margin-left: auto;
}

/* 底部操作区 */
.sidebar-actions {
  display: flex;
  gap: 10px;
  margin-top: auto;
  padding-top: 20px;
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

.logout-button {
  flex-grow: 1;
  border-radius: 20px;
  background-color: #e74c3c;
  color: white;
  border: none;
  height: 40px;
  transition: var(--transition-standard);
  justify-content: center;
  gap: 8px;
  font-weight: 500;
}

.logout-button:hover {
  background-color: #c0392b;
}

.theme-toggle-button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #ffffff;
  color: #4a4a4a;
  border: 1px solid #d1d5db;
  transition: var(--transition-standard);
}

.theme-toggle-button:hover {
  background-color: #4a4a4a;
  color: white;
}

.btn-icon{
  margin-right:4px;
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
  color: #f1f2f6;
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

.dark .logout-button {
  background-color: #c0392b;
  color: #f1f2f6;
}

.dark .logout-button:hover {
  background-color: #e74c3c;
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