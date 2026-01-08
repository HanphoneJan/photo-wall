<template>
<!-- 欢迎页面内容 -->
<transition name="slide-left" mode="out-in">
<div v-if="showWelcome" key="welcome-page" class="welcome-container">
<el-row style="height: 100vh" class="welcome-row">
<el-col :span="24" style="height: 100%">
<el-card shadow="hover" class="welcome">
<h1 class="tit">
{{ welcomeTitle }}
<div class="border"></div>
</h1>
<h2 class="intro">{{ intro }}</h2>
<!-- 点击箭头触发 startRead 函数 -->
<div class="bounce down" @click="handleClick">
<el-icon style="color:white;"><ArrowRightBold /></el-icon>
</div>
</el-card>
</el-col>
</el-row>
</div>
</transition>

<div class="atlas-container" id="atlas-container">
<div class="masonry-columns">
<div
v-for="item in filteredAtlasData"
:key="item.id"
class="grid-item"
>
<el-card :body-style="{ padding: '0px' }" class="photo-card">
<div class="photo-wrapper">
<!-- 移除了 preview-teleported 和 preview-src-list 属性 -->
<el-image
:src="item.path"
class="image"
lazy
ref="imageRef"
/>
<div class="photo-overlay">
<div class="photo-info-overlay">
<!-- 紧凑布局：标题和作者同一行 -->
<div class="title-author-row">
<div class="photo-title">{{ item.title }}</div>
<div class="photo-author">@{{ item.author }}</div>
</div>

<!-- 描述保持两行限制 -->
<div class="photo-description">{{ item.description }}</div>

<!-- 标签和操作按钮放在同一行 -->
<div class="tags-actions-row">
<div class="photo-tags">
<el-tag
v-for="tag in item.tags"
:key="tag.id"
size="small"
type="info"
class="photo-tag"
>
#{{ tag.name }}
</el-tag>
</div>

<div class="photo-actions">
<el-icon class="action-icon" @click="zoomImage(item)">
<ZoomIn :style="{ color: 'white' }" />
</el-icon>
<el-icon class="action-icon" @click="downloadImage(item)">
<Download :style="{ color: 'white' }" />
</el-icon>
<el-icon class="action-icon" @click="handleLikes(item)">
<StarFilled v-if="item.isLiked" :style="{ color: 'yellow' }" />
<Star v-else :style="{ color: 'white' }" />
</el-icon>
<span class="likes-count">{{ item.likes }}</span>
</div>
</div>
</div>
</div>
</div>
</el-card>
</div>
</div>
</div>
</template>

<script setup lang="ts">
// 脚本部分 - 引入 Element Plus 图标替代 SVG
import { ref, onMounted, computed, watchEffect, watch } from 'vue';
import { useRoute } from 'vue-router';
import api from '@/api/interceptor';
import { ENDPOINTS } from '@/api/api';
// 移除 SVG 导入，改为引入 Element Plus 图标
import { ArrowRightBold, Star, StarFilled, ZoomIn, Download } from '@element-plus/icons-vue';

interface Tag {
id: number | null;
name: string;
}

interface AtlasItem {
id: number;
path: string;
author: string;
title: string;
description: string;
likes: number;
type: number;
upload_time: string;
isLiked: boolean;
tags:Tag[];
username: string;
}

const route = useRoute();

const tagsList = ref<Tag[]>([]);
const atlasData = ref<AtlasItem[]>([]);
const sortedAtlasData = ref<AtlasItem[]>([]);
const selectedTags = ref<number[]>([]);
const showWelcome = ref(true);
const imageRefs = ref<(HTMLElement | null)[]>([]);
let timerId: ReturnType<typeof setTimeout> | undefined = undefined;
const intro = ref('生活の瞬間を捉え、感動を記録する');
const welcomeTitle = ref('寒楓のフォトギャラリー');

onMounted(async () => {
    const bgImage = new Image();
  bgImage.src = 'https://hanphone.top/blog/atlas/background.jpeg';
  
getTag();
atlasShow();
startReadTimeout();
});

watch(() => route.query.tags, (newTags) => {
if (newTags) {
selectedTags.value = (newTags as string).split(',').map(id => Number(id));
} else {
selectedTags.value = [];
}
}, { immediate: true });

const filteredAtlasData = computed(() => {
const rawSelectedTags = selectedTags.value.map(id => Number(id));
if (rawSelectedTags.length === 0) {
return sortedAtlasData.value;
}
const result = sortedAtlasData.value.filter(item => {
const isMatch = rawSelectedTags.every(tagId =>
item.tags.some(tag => tag.id !== null && Number(tag.id) === tagId)
);
return isMatch;
});
return result;
});

const startRead = () => {
showWelcome.value = false;
};

const handleClick = () => {
clearTimeout(timerId);
startRead();
};

const startReadTimeout = () => {
timerId = setTimeout(() => {
startRead();
}, 3000);
};

const atlasShow = async ()=>{
try {
const response = await api.get(ENDPOINTS.SHOW);
if (response.data.status === 830) {
atlasData.value = response.data.data;
sortedAtlasData.value = [...atlasData.value].sort((b, a) => a.likes - b.likes);
}
} catch (error) {
console.error('获取图集数据失败：', error.message);
}
}

const getTag = async()=>{
try {
const response = await api.get(ENDPOINTS.GETTAG);
if (response.data.status === 830) {
tagsList.value = response.data.data;
}
} catch (error) {
console.error('获取标签列表失败:', error.message);
}
}

const handleLikes = (item: AtlasItem) => {
if (item.isLiked) {
return;
}
api.post(ENDPOINTS.LIKES, { id: item.id }).then(response => {
if (response.data.status === 830) {
item.isLiked = true;
item.likes++;
console.log(response.data.message);
} else {
console.error('点赞失败:', response.data.message);
}
}).catch(error => {
console.error('点赞失败:', error.message);
});
};

// 放大查看图片 - 修改为使用自定义预览
const zoomImage = (item: AtlasItem) => {
// 创建一个自定义的图片预览组件
const previewContainer = document.createElement('div');
previewContainer.style.position = 'fixed';
previewContainer.style.top = '0';
previewContainer.style.left = '0';
previewContainer.style.width = '100%';
previewContainer.style.height = '100%';
previewContainer.style.backgroundColor = 'rgba(0, 0, 0, 0.8)';
previewContainer.style.display = 'flex';
previewContainer.style.justifyContent = 'center';
previewContainer.style.alignItems = 'center';
previewContainer.style.zIndex = '9999';
previewContainer.style.cursor = 'zoom-out';

const previewImage = document.createElement('img');
previewImage.src = item.path;
previewImage.style.maxWidth = '90%';
previewImage.style.maxHeight = '90%';
previewImage.style.objectFit = 'contain';

previewContainer.appendChild(previewImage);
document.body.appendChild(previewContainer);

// 点击关闭预览
previewContainer.addEventListener('click', () => {
document.body.removeChild(previewContainer);
});
};

// 下载图片
const downloadImage = (item: AtlasItem) => {
// 处理URL，添加download=1参数
const url = new URL(item.path);
url.searchParams.append('download', '1'); // 无论原URL是否有参数，都会正确添加

const link = document.createElement('a');
link.target = '_blank';
link.href = url.toString(); // 使用添加参数后的URL
link.download = `${item.title}_${item.author}.jpg`;
document.body.appendChild(link);
link.click();
// 延迟移除避免部分浏览器问题
setTimeout(() => {
document.body.removeChild(link);
}, 0);
};
watchEffect(() => {
const handleResize = () => {
if (window.innerWidth < 768) {
welcomeTitle.value = '寒楓の照片墙';
intro.value = '生活の瞬間を捉え';
} else {
welcomeTitle.value = '寒楓のフォトギャラリー';
}
};
handleResize();
window.addEventListener('resize', handleResize);
return () => {
window.removeEventListener('resize', handleResize);
};
});
</script>

<style scoped>
/* 基础样式变量 - 黑白简约风格 */
:root {
--primary-color: #007bff;
--text-color: #212529;
--text-secondary: #6c757d;
--bg-color: #ffffff;
--bg-secondary: #f8f9fa;
--shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
--transition: all 0.3s ease;
}

/* 欢迎页面容器 */
.welcome-container {
position: fixed;
top: 0;
left: 0;
width: 100%;
height: 100vh;
z-index: 1000;
}

.welcome-row {
background-color: #f8fafc;
background-image: url('https://hanphone.top/blog/atlas/background.jpeg');
background-size: cover;
background-position: center;
height: 100%;
display: flex;
justify-content: center;
align-items: center;
}

.welcome {
background-color: rgba(0, 0, 0, 0);
border: none;
height: 100%;
position: relative;
}

.border {
width: 812px;
height: 112px;
position: absolute;
top: -6px;
left: -6px;
border: 3px solid white;
box-sizing: border-box;
animation: clipMe 5s linear infinite;
}

.tit {
box-sizing: border-box;
position: relative;
width: 800px;
height: 100px;
line-height: 100px;
box-shadow: inset 0 0 0 1px white;
margin: 40px auto;
margin-top: 80px;
color: rgb(77, 77, 77,1);
text-align: center;
font-size: 50px;
font-weight: normal;
letter-spacing: 10px;
font-family:Courier,Kaiti, monospace,CustomFont;
}

.intro {
letter-spacing: 5px;
line-height: 50px;
width: 80%;
margin: 0 auto;
text-align: center;
font-weight: normal;
color: rgba(77,77,77,1);
font-family:Courier,Kaiti, monospace,CustomFont;
}

.down {
animation: bounce 2s infinite;
animation-duration: 3s;
font-size: 25px;
position: absolute;
bottom: 8vmin;
left: 50%;
transform: translateX(-50%);
display: flex;
justify-content: center;
align-items: center;
width: 50px;
height: 50px;
border-radius: 50%;
border: 2px solid #fff;
}

.down:hover {
animation: none;
cursor: pointer;
box-shadow: 0 0 20px 0 white;
transition: all .2s;
}

.atlas-container {
padding: 0;
width: 100vw;
margin: 0 auto;
position: relative;
background: #f8fafc;
min-height: 100vh;
}

.atlas-container::before {
content: "";
position: absolute;
top: 0;
left: 0;
right: 0;
bottom: 0;
background-image:
radial-gradient(circle at 10% 20%, rgba(0, 123, 255, 0.05) 0%, transparent 20%),
radial-gradient(circle at 90% 80%, rgba(0, 123, 255, 0.05) 0%, transparent 20%),
radial-gradient(circle at 50% 50%, rgba(0, 123, 255, 0.03) 0%, transparent 50%);
z-index: -1;
}

.atlas-container::after {
content: "";
position: absolute;
top: 0;
left: 0;
right: 0;
bottom: 0;
background-image:
linear-gradient(rgba(0, 0, 0, 0.02) 1px, transparent 1px),
linear-gradient(90deg, rgba(0, 0, 0, 0.02) 1px, transparent 1px);
background-size: 20px 20px;
z-index: -1;
}

.masonry-columns {
column-count: 4;
column-gap: 0;
position: relative;
z-index: 1;
}

@media (max-width: 1200px) {
.masonry-columns { column-count: 3; }
}
@media (max-width: 992px) {
.masonry-columns { column-count: 2; }
}
@media (max-width: 768px) {
.masonry-columns { column-count: 2; }
}
@media (max-width: 480px) {
.masonry-columns { column-count: 2; }
.atlas-container {
padding: 0;
}
}

.grid-item {
break-inside: avoid;
margin-bottom: 0;
}

.photo-card {
border-radius: 0;
overflow: hidden;
box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
transition: var(--transition);
margin-bottom: 0;
background-color: var(--bg-color);
/* 必须要有边框，否则el样式异常 */
border:1px solid gray;
backdrop-filter: blur(5px);
}

.photo-card:hover {
box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
}

.photo-wrapper {
position: relative;
overflow: hidden;
}

.image {
width: 100%;
height: auto;
display: block;
transition: transform 0.3s;
}

.photo-card:hover .image {
transform: scale(1.03);
}

.photo-overlay {
position: absolute;
top: 0;
left: 0;
right: 0;
bottom: 0;
background: linear-gradient(to bottom, rgba(0,0,0,0.05) 0%, rgba(0,0,0,0.2) 50%);
opacity: 0;
transition: opacity 0.3s;
display: flex;
flex-direction: column;
justify-content: flex-end;
padding: 8px; /* 减少内边距 */
color: white;
}

.photo-card:hover .photo-overlay {
opacity: 1;
}

.photo-info-overlay {
width: 100%;
gap: 6px; /* 减少信息块之间的间距 */
display: flex;
flex-direction: column;
}

/* 紧凑布局样式 */
.title-author-row {
display: flex;
justify-content: space-between;
align-items: center;
width: 100%;
margin-bottom: 4px; /* 减少间距 */
}

.tags-actions-row {
display: flex;
justify-content: space-between;
align-items: center;
width: 100%;
margin-top: 4px; /* 减少间距 */
}

.photo-title {
font-weight: 500;
font-size: 14px; /* 缩小字体 */
color: white;
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-box-orient: vertical;
flex: 1;
margin-right: 8px;
}

.photo-author {
font-size: 11px; /* 缩小字体 */
color: rgba(255, 255, 255, 0.8);
white-space: nowrap;
}

.photo-description {
font-size: 12px; /* 缩小字体 */
color: rgba(255, 255, 255, 0.9);
margin-bottom: 4px; /* 减少间距 */
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-box-orient: vertical;
line-height: 1.3; /* 缩小行高 */
}

.photo-tags {
display: flex;
flex-wrap: wrap;
gap: 2px; /* 减少标签间距 */
flex: 1;
overflow: hidden;
}

.photo-tag {
font-size: 10px; /* 缩小标签字体 */
padding: 1px 4px; /* 减少标签内边距 */
border-radius: 3px;
background-color: rgba(255, 255, 255, 0.2);
color: white;
border: none;
}

.photo-actions {
display: flex;
align-items: center;
margin-left: 6px; /* 增加与标签的间距 */
white-space: nowrap;
gap: 4px; /* 添加按钮之间的间距 */
}

.action-icon {
cursor: pointer;
color: white;
margin-left: 8px;
font-size: 1.2em; /* 调整图标大小 */
}

.likes-count {
color: #fff;
font-size: 12px; /* 缩小点赞数字体 */
font-weight: 500;
}

/* 动画过渡效果 */
.fade-enter, .fade-leave-to {
opacity: 0;
}

.fade-enter-active, .fade-leave-active {
transition: opacity 1s;
}

.slide-left-enter-active {
transition: all 0.5s ease-out;
}

.slide-left-leave-active {
transition: all 0.5s ease-in;
}

.slide-left-enter-from {
transform: translateX(-100%);
}

.slide-left-leave-to {
transform: translateX(-100%);
}

@keyframes clipMe {
0%, 100% {
clip: rect(0px, 806px, 6px, 0px);
}

25% {
clip: rect(0px, 6px, 112px, 0px);
}

50% {
clip: rect(112px, 812px, 112px, 0px);
}

75% {
clip: rect(0px, 812px, 112px, 806px);
}
}

@keyframes bounce {
0%, 20%, 50%, 80%, 100% {
transform: translate(-50%, 0);
}
40% {
transform: translate(-50%, -5px);
}
60% {
transform: translate(-50%, -3px);
}
}

@media screen and (max-width: 768px) {
.welcome-row {
background-image: url('https://hanphone.top/blog/atlas/background.jpeg');
}

.welcome {
width: 100%;

.border {
display: none;
}

.tit {
font-size: 2rem;
width: 100%;
line-height: 50px;
letter-spacing: 2px;
height: auto;
}

.intro {
font-size: 1rem;
line-height: 30px;
}
}

/* 移动端进一步紧凑处理 */
.photo-overlay {
padding: 8px;
}

.photo-title {
font-size: 13px;
}

.photo-description {
font-size: 11px;
}

.photo-actions {
gap: 2px; /* 移动端减少按钮间距 */
}
}

/* 暗黑模式样式 */
.dark {
--primary-color: #4dabf7;
--text-color: #f8f9fa;
--text-secondary: #adb5bd;
--bg-color: #111827;
--bg-secondary: #343a40;
--shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.dark .welcome-row{
background-color: #111827;
background-image: url('https://hanphone.top/blog/atlas/background.jpeg');
}
.dark .welcome{
background-color: rgba(0, 0, 0, 0.1);
}
.dark .tit{
color: white;
}
.dark .intro{
color: white;
}
.dark .photo-card {
background-color: var(--bg-color);
border:1px solid #111827;
}
.dark .photo-title {
color: var(--text-color);
}
.dark .photo-author {
color: var(--text-secondary);
}
.dark .photo-description {
color: var(--text-secondary);
}
.dark .photo-tag {
background-color: var(--bg-secondary);
color: var(--text-secondary);
}

.dark .atlas-container {
background: linear-gradient(135deg, #1a1d23 0%, #2c313a 100%);
}

.dark .atlas-container::before {
background-image:
radial-gradient(circle at 10% 20%, rgba(77, 171, 247, 0.05) 0%, transparent 20%),
radial-gradient(circle at 90% 80%, rgba(77, 171, 247, 0.05) 0%, transparent 20%),
radial-gradient(circle at 50% 50%, rgba(77, 171, 247, 0.03) 0%, transparent 50%);
}

.dark .atlas-container::after {
background-image:
linear-gradient(rgba(255, 255, 255, 0.02) 1px, transparent 1px),
linear-gradient(90deg, rgba(255, 255, 255, 0.02) 1px, transparent 1px);
}

/* 在组件的style scoped部分 */
.atlas-container {
  overflow: hidden;
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE and Edge */
}

.atlas-container::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}
</style>