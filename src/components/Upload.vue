<template>
  <!-- 上传页面 - 模拟暗房冲洗风格设计
    设计特点：
    - 暗房红色安全灯色调为灵感
    - 胶片边框与相纸纹理元素
    - 冲洗过程的视觉隐喻
    - 支持明暗主题自适应
  -->
  <div class="upload-wrapper">
    <!-- 顶部装饰条 - 模拟胶片胶片边缘 -->
    <div class="film-stripe"></div>
    
    <!-- 上传容器 - 模拟相纸尺寸 -->
    <div class="upload-container">
      <!-- 标题区域 - 暗房风格标识 -->
      <div class="upload-header">
        <h2 class="upload-title">
          <el-icon :size="24" class="title-icon"><Camera /></el-icon>
          照片上传
        </h2>
      </div>
      
      <!-- 表单滚动容器 -->
      <div class="form-scroll-container">
        <!-- 表单区域 - 模拟冲洗药液容器 -->
        <el-form 
          ref="uploadForm" 
          :model="form" 
          label-width="110px"
          class="upload-form"
        >
          <!-- 照片上传区 - 模拟胶片插槽 -->
          <el-form-item label="选择照片" class="form-item-photo">
            <el-upload
              list-type="picture-card"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              v-model:file-list="fileList"
              :before-upload="beforeUpload"
              :auto-upload="false"
              class="photo-uploader"
            >
              <div class="upload-plus">
                <el-icon :size="28"><Plus /></el-icon>
                <span class="upload-text">添加照片</span>
              </div>
            </el-upload>
            
            <!-- 图片预览组件 -->
            <el-image-viewer
              v-if="previewVisible"
              :url-list="previewUrlList"
              :initial-index="previewIndex"
              @close="previewVisible = false"
            />
            
            <p class="upload-hint">支持JPG、PNG格式，单张不超过100MB</p>
          </el-form-item>
          
          <!-- 照片信息输入区 - 模拟标签书写区 -->
          <el-form-item label="照片标题" class="form-item">
            <el-input 
              v-model="form.title" 
              placeholder="输入照片标题"
              class="custom-input"
            ></el-input>
          </el-form-item>
          
          <el-form-item label="拍摄作者" class="form-item">
            <el-input 
              v-model="form.author" 
              placeholder="输入作者名称"
              class="custom-input"
            ></el-input>
          </el-form-item>
          
          <el-form-item label="照片描述" class="form-item">
            <el-input 
              v-model="form.description" 
              placeholder="描述照片内容或故事"
              class="custom-input"
              type="textarea"
              :rows=3
            ></el-input>
          </el-form-item>
          
          <!-- 操作按钮区 - 模拟暗房操作按钮 -->
          <el-form-item class="form-actions">
            <el-button 
              @click="submitForm" 
              :loading="isUploading"
              class="submit-btn"
            >
            上传图片
              <el-icon :size="16" class="btn-icon"><Upload /></el-icon>
              
            </el-button>
            <el-button 
              @click="resetForm"
              class="reset-btn"
            >
            重置
              <el-icon :size="16" class="btn-icon"><Refresh /></el-icon>
              
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
    
    <!-- 底部装饰条 - 模拟胶片边缘 -->
    <div class="film-stripe bottom"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from 'vue';
import { useUserStore } from '@/store/store';
import api from '@/api/interceptor';
import { ElImageViewer, ElMessage } from 'element-plus';
import { ENDPOINTS } from '@/api/api';
import { Plus, Upload, Refresh, Camera } from '@element-plus/icons-vue';

const userStore = useUserStore();
const form = ref<{ title?: string; author?: string; description: string }>({
  title: '',
  author: '',
  description: '',
});
const fileList = ref<Array<any>>([]);  // 上传的文件列表
const uploadedUrls = ref<string[]>([]); // 存储已上传图片的URL
const isUploading = ref(false); // 上传状态

/**
 * 在上传文件之前进行验证。
 * @param {any} file - 要上传的文件对象。
 * @returns {boolean} - 如果文件是图片且大小不超过100MB，返回true；否则返回false。
 * @throws {ElMessage.error} - 如果文件类型不是图片，或文件大小超过限制，将显示错误消息。
 */
const beforeUpload = (file: any) => {
  const isImage = file.type.startsWith('image/');
  const isLtM = file.size / 1024 / 1024 < 100; // 限制文件大小

  if (!isImage) {
    ElMessage.error('只能上传图片文件!');
  }
  if (!isLtM) {
    ElMessage.error('文件大小不能超过100MB!');
  }
  return isImage && isLtM;
};

// 图片预览相关状态
const previewVisible = ref(false); // 是否显示预览
const previewUrlList = ref<string[]>([]); // 预览的图片 URL 列表
const previewIndex = ref(0); // 当前预览的图片索引

// 图片预览处理
/**
 * 处理文件预览的函数
 * @param {any} file - 需要预览的文件对象
 * 此函数会根据传入的文件对象设置预览的图片 URL 列表和当前预览的图片索引，并显示预览界面。
 */
const handlePreview = (file: any) => {
  // 设置预览的图片 URL 列表
  previewUrlList.value = fileList.value.map((f) => f.url || f.response?.url);
  // 设置当前预览的图片索引
  previewIndex.value = fileList.value.findIndex((f) => f.uid === file.uid);
  // 显示预览
  previewVisible.value = true;
};

/**
 * 处理文件移除操作
 * @param {any} file - 要移除的文件对象
 * @param {Array<any>} files - 当前文件列表
 * 从文件列表中移除指定的文件，并显示相应的提示信息。
 * 如果文件成功移除，将显示成功消息；如果未找到文件，将显示警告消息。
 */
const handleRemove = (file: any, files: Array<any>) => {
  // 从 fileList 中移除该文件
  const index = fileList.value.findIndex((f) => f.uid === file.uid);
  if (index !== -1) {
    fileList.value.splice(index, 1);
    // 同时从已上传URL列表中移除
    if (file.url) {
      const urlIndex = uploadedUrls.value.indexOf(file.url);
      if (urlIndex !== -1) {
        uploadedUrls.value.splice(urlIndex, 1);
      }
    }
    ElMessage.success('文件删除成功');
  } else {
    ElMessage.warning('文件未找到');
  }
};

/**
 * 上传单个文件并返回URL
 * @param {any} file - 要上传的文件对象
 * @returns {Promise<string>} - 返回上传后的URL
 */
const uploadSingleFile = async (file: any): Promise<string> => {
  return new Promise((resolve, reject) => {
    const formData = new FormData();
    formData.append('namespace', 'blog/atlas')
    formData.append('file', file.raw);
    
    api.post(ENDPOINTS.FILE.UPLOAD, formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    })
    .then(response => {
      if (response.data?.url) {
        resolve(response.data.url);
      } else {
        reject(new Error('上传失败，未返回URL'));
      }
    })
    .catch(error => {
      reject(error);
    });
  });
};

/**
 * 提交表单数据并上传文件
 * 此函数用于验证表单输入和文件列表，确保所有必填项均已填写，并且上传的文件数量在允许范围内。
 * 如果验证通过，将先逐张上传图片获取URL，然后再提交表单数据。
 */
const submitForm = async () => {
  // 验证表单输入和文件列表
  if (!form.value.title || !form.value.author || !form.value.description) {
    ElMessage.error('请填写所有必填项!');
    return;
  }
  if (fileList.value.length === 0) {
    ElMessage.error('请上传至少一张图片!');
    return;
  }
  if (fileList.value.length > 10) {
    ElMessage.error('一次性上传文件不能超过十张!');
    return;
  }
  
  isUploading.value = true;
  
  try {
    // 逐张上传图片
    const urls: string[] = [];
    
    for (const file of fileList.value) {
      try {
        const url = await uploadSingleFile(file);
        urls.push(url);
        // 更新文件列表中的URL，以便预览
        file.url = url;
      } catch (error) {
        ElMessage.error(`图片 ${file.name} 上传失败`);
        console.error('上传出错:', error);
        isUploading.value = false;
        return;
      }
    }
    
    // 保存上传的URL
    uploadedUrls.value = urls;
    
    // 构建提交数据
    const submitData = {
      title: form.value.title || '未命名',
      author: form.value.author || '佚名',
      description: form.value.description || '无',
      userId: userStore.userId || '1000',
      urls: urls // 使用urls[]而不是files[]
    };
    
    console.log('提交数据:', submitData);
    
    // 发送 POST 请求提交表单数据
    api.post(ENDPOINTS.UPLOAD, submitData)
      .then(response => {
        // 根据新的返回数据结构修改判断条件
        if (response.data.overallStatus === 830) {
          ElMessage.success('提交成功');
          resetForm();
        } else {
          ElMessage.error('提交失败');
          console.log('提交失败:' + response.data.message);
          
          // 如果有详细的结果信息，可以显示每个文件的上传状态
          if (response.data.results && response.data.results.length > 0) {
            response.data.results.forEach((result: any, index: number) => {
              if (result.status !== 830) {
                console.log(`文件 ${index + 1} 上传失败: ${result.message}`);
              }
            });
          }
        }
      })
      .catch(error => {
        ElMessage.error('提交出错');
        console.error('提交出错:', error.message);
      })
      .finally(() => {
        isUploading.value = false;
      });
  } catch (error) {
    ElMessage.error('上传过程中出错');
    console.error('上传过程中出错:', error);
    isUploading.value = false;
  }
};

/**
 * 重置表单字段和文件列表。
 * 将表单中的标题、作者和描述字段清空，并清空文件列表。
 */
const resetForm = () => {
  form.value.title = '';
  form.value.author = '';
  form.value.description = '';
  fileList.value = [];
  uploadedUrls.value = [];
};

// 监听 theme 状态变化，应用到 HTML 上
watchEffect(() => {
  const htmlClassList = document.documentElement.classList;
  if (userStore.theme === 'dark') {
    htmlClassList.add('dark-theme');
  } else {
    htmlClassList.remove('dark-theme');
  }
});
</script>

<style scoped>
/* 基础样式变量 - 暗房色调系统 */
:root {
  --darkroom-red: #9b2226;
  --darkroom-dark: #161a1d;
  --darkroom-light: #f5f3f4;
  --darkroom-gray: #666260;
  --darkroom-accent: #0984e3;
  --film-border: #d3d3d3;
  --transition-standard: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* 上传页面容器 - 模拟暗房环境 */
.upload-wrapper {
  height: 100vh;
  background-color: var(--darkroom-light);
  background-image: 
    radial-gradient(rgba(0, 0, 0, 0.03) 1px, transparent 1px),
    radial-gradient(rgba(0, 0, 0, 0.02) 1px, transparent 1px);
  background-size: 50px 50px;
  background-position: 0 0, 25px 25px;
  padding: 40px 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-sizing: border-box;
}

/* 胶片装饰条 */
.film-stripe {
  width: 100%;
  height: 24px;
  background-color: var(--film-border);
  background-image: 
    repeating-linear-gradient(90deg, 
      transparent, transparent 20px, 
      var(--darkroom-dark) 20px, var(--darkroom-dark) 24px
    );
  margin-bottom: 20px;
  border-radius: 4px;
}

.film-stripe.bottom {
  margin-top: 20px;
  margin-bottom: 0;
}

/* 上传容器 - 模拟相纸 */
.upload-container {
  width: 100%;
  max-width: 680px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 
    0 4px 20px rgba(0, 0, 0, 0.08),
    0 0 0 1px rgba(0, 0, 0, 0.05);
  overflow: hidden;
  transition: var(--transition-standard);
  /* 计算容器最大高度：视口高度减去上下装饰条和边距 */
  max-height: calc(100vh - 128px);
  display: flex;
  flex-direction: column;
}

/* 标题区域 */
.upload-header {
  padding: 14px 30px 6px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  text-align: center;
  flex-shrink: 0; /* 标题区域不收缩 */
}

.upload-title {
  font-size: 24px;
  font-weight: 700;
  color: var(--darkroom-dark);
  margin: 0 0 8px 0;
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.title-icon {
  color: var(--darkroom-red);
}

.upload-subtitle {
  font-size: 14px;
  color: var(--darkroom-gray);
  margin: 0;
}

/* 表单滚动容器 */
.form-scroll-container {
  flex-grow: 1; /* 占据剩余空间 */
  overflow-y: auto; /* 垂直滚动 */
  scrollbar-width: thin; /* 细滚动条 */
  scrollbar-color: var(--darkroom-gray) transparent; /* 滚动条颜色 */
}

/* 滚动条样式优化 */
.form-scroll-container::-webkit-scrollbar {
  width: 6px;
}

.form-scroll-container::-webkit-scrollbar-track {
  background: transparent;
}

.form-scroll-container::-webkit-scrollbar-thumb {
  background-color: var(--darkroom-gray);
  border-radius: 3px;
}

/* 表单样式 */
.upload-form {
  padding: 30px;
}

/* 表单项样式 */
.form-item-photo {
  margin-bottom: 25px !important;
}

.form-item {
  margin-bottom: 20px !important;
}

/* 上传组件样式 */
.photo-uploader {
  margin-bottom: 12px;
}

.upload-plus {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 140px;
  border: 2px dashed #d1d5db;
  border-radius: 6px;
  color: var(--darkroom-gray);
  transition: var(--transition-standard);
  cursor: pointer;
}

.upload-plus:hover {
  border-color: var(--darkroom-accent);
  color: var(--darkroom-accent);
  background-color: rgba(9, 132, 227, 0.03);
}

.upload-text {
  margin-top: 8px;
  font-size: 14px;
}

.upload-hint {
  margin: 0;
  font-size: 12px;
  color: var(--darkroom-gray);
  padding-left: 4px;
}

/* 自定义输入框样式 */
.custom-input {
  width: 100%;
  transition: var(--transition-standard);
}

:deep(.custom-input .el-input__wrapper) {
  border-radius: 6px;
  transition: var(--transition-standard);
}

:deep(.custom-input .el-input__wrapper:focus-within) {
  border-color: var(--darkroom-accent);
  box-shadow: 0 0 0 2px rgba(9, 132, 227, 0.2);
}

:deep(.el-form-item__label) {
  font-size: 14px;
  font-weight: 500;
  color: var(--darkroom-dark);
  padding-right: 12px;
}

/* 按钮样式 */
.form-actions {
  margin-top: 30px !important;
  display: flex;
  justify-content: center;
  gap: 16px;
  padding-bottom: 10px; /* 底部留出空间，避免被滚动条遮挡 */
}

.submit-btn {
  background-color: transparent;
  color: var(--darkroom-gray);
  border: 1px solid #d1d5db;
  padding: 10px 24px;
  border-radius: 6px;
  font-weight: 500;
  transition: var(--transition-standard);
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.submit-btn:hover {
  border-color: var(--darkroom-gray);
  background-color: rgba(102, 98, 96, 0.05);
  color: var(--darkroom-dark);
}

.reset-btn {
  background-color: transparent;
  color: var(--darkroom-gray);
  border: 1px solid #d1d5db;
  padding: 10px 24px;
  border-radius: 6px;
  font-weight: 500;
  transition: var(--transition-standard);
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.reset-btn:hover {
  border-color: var(--darkroom-gray);
  background-color: rgba(102, 98, 96, 0.05);
  color: var(--darkroom-dark);
}

.btn-icon{
  margin-left: 3px;
}

/* 深色主题样式 - 暗房红灯效果 */
.dark {
  --darkroom-light: #111827;
  --darkroom-dark: #f5f3f4;
  --film-border: #2d3436;
}

.dark .upload-wrapper {
  background-color: #111827;
  background-image: 
    radial-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
    radial-gradient(rgba(255, 255, 255, 0.02) 1px, transparent 1px);
}

.dark .upload-container {
  background-color: #161b22;
  box-shadow: 
    0 4px 20px rgba(0, 0, 0, 0.3),
    0 0 0 1px rgba(255, 255, 255, 0.05);
}

.dark .upload-header {
  border-bottom-color: rgba(255, 255, 255, 0.05);
}

.dark .upload-title {
  color: var(--darkroom-dark);
}

.dark .upload-subtitle {
  color: #8b949e;
}

.dark .upload-plus {
  border-color: #30363d;
  color: #8b949e;
}

.dark .upload-plus:hover {
  border-color: var(--darkroom-accent);
  background-color: rgba(9, 132, 227, 0.1);
}

.dark .upload-hint {
  color: #8b949e;
}

:deep(.dark .custom-input .el-input__wrapper) {
  border-color: #30363d;
  background-color: #0d1117;
}

:deep(.dark .el-input__inner) {
  color: #e6edf3;
}

:deep(.dark .el-form-item__label) {
  color: #e6edf3;
}

.dark .submit-btn {
  color: #e6edf3;
  border-color: #30363d;
}

.dark .reset-btn {
  color: #e6edf3;
  border-color: #30363d;
}

.dark .reset-btn:hover {
  background-color: rgba(139, 148, 158, 0.1);
  color: #e6edf3;
  border-color: #484f58;
}

/* 响应式设计 */
@media screen and (max-width: 768px) {
  .upload-wrapper {
    padding: 20px 10px;
  }
  
  .upload-container {
    max-width: 100%;
    max-height: calc(100vh - 88px); /* 移动端调整最大高度 */
  }
  
  .upload-header {
    padding: 18px 20px;
  }
  
  .upload-form {
    padding: 20px;
  }
  
  :deep(.el-form-item__label) {
    font-size: 13px;
    padding-right: 8px;
  }
  
  .form-actions {
    flex-direction: column;
    gap: 10px;
  }
  
  .submit-btn, .reset-btn {
    width: 100%;
    justify-content: center;
  }
}

@media screen and (max-width: 768px) {
  .upload-wrapper {
    padding: 20px 10px;
  }
  
  .upload-container {
    max-width: 100%;
    /* 取消表单移动端取消表单边界效果 */
    box-shadow: none;
    border: none;
  }
  
  .upload-header {
    padding: 18px 20px;
  }
  
  .upload-form {
    padding: 20px;
    /* 移除表单内边距和边框效果 */
    border: none;
  }
  
  :deep(.el-form-item) {
    /* 移除表单项的边框效果 */
    border: none !important;
    box-shadow: none !important;
  }
  
  :deep(.el-form-item__label) {
    font-size: 13px;
    padding-right: 8px;
  }
  
  /* 保持按钮并排显示 */
  .form-actions {
    flex-direction: row; /* 强制横向排列 */
    gap: 10px;
    width: 100%;
  }
  
  .submit-btn, .reset-btn {
    flex: 1; /* 按钮平均分配宽度 */
    width: auto; /* 取消100%宽度 */
    justify-content: center;
  }
}
</style>