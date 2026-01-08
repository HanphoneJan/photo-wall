<template>
  <div class="admin-atlas-container">
    <!-- 响应式布局容器 -->
    
    <!-- 大屏幕：表格布局 -->
    <div class="table-layout" v-if="isDesktop">
      <div class="table-container">
        <el-table :data="atlasData" class="responsive-table">
          <!-- /**
           * @description 表格列用于展示图片。
           * @component AdminAtlas.vue
           * @slot default - 自定义插槽，显示每行数据的图片。
           * @param {Object} scope - 当前行的数据上下文。
           * @param {string} scope.row.path - 图片的路径，用于显示和预览。
           */ -->
          <el-table-column label="图片">
            <template #default="scope">
              <el-image
                :src="scope.row.path"
                class="image-preview"
                :preview-src-list="[scope.row.path]"
                preview-teleported
              />
            </template>
          </el-table-column>
          <el-table-column prop="author" label="作者" width="90px" sortable></el-table-column>
          <el-table-column prop="title" label="标题"></el-table-column>
          <el-table-column prop="description" label="简介"></el-table-column>
          <el-table-column prop="upload_time" label="上传时间" sortable></el-table-column>
          <!-- 根据点赞数排序 -->
          <el-table-column prop="likes" label="点赞数" width="100px" :sortable="true" :sort-method="likesSort"></el-table-column>
          <!-- 根据审核状态排序 -->
          <!-- /**
           * @description 渲染审核状态列，使用 el-switch 组件来切换状态。
           * @param {Object} row - 当前行的数据对象。
           * @param {number} row.type - 当前行的审核状态，1 表示通过，0 表示不通过。
           * @event change - 当开关状态改变时触发，调用 photoStateChanged 方法。
           */ -->
          <el-table-column label="审核状态" width="110px" :sortable="true" :sort-method="typeSort">
            <template #default="{ row }">
              <el-switch
                :model-value="row.type"
                :active-value="1"
                :inactive-value="0"
                @change="(newValue) => photoStateChanged(row, newValue)"
              />
            </template>
          </el-table-column>
          <el-table-column label="标签" prop="tags">
            <template #default="scope">
              <!-- 显示已有标签 -->
              <el-tag size="default" v-for="(tag, i) in scope.row.tags" :key="tag.id" closable
                      @close="handleClose(i, scope.row)">
                {{ tag.name }}
              </el-tag>
              <!-- 新标签输入框 -->
              <el-input size="small" style="width: 100px;" class="input-new-tag" v-if="scope.row.inputVisible"
                        v-model="scope.row.inputValue" ref="saveTagInput"
                        @keyup.enter="handleInputConfirm(scope.row)"
                        @blur="handleInputConfirm(scope.row)">
              </el-input>
              <!-- 新标签按钮 -->
              <el-button v-else size="small" class="button-new-tag" @click="showInput(scope.row)">
                + 新标签
                </el-button>
                </template>
              </el-table-column>
          <!-- 操作列 -->
          <el-table-column label="操作">
            <template #default="scope">
              <div class="btn-group">
                <el-button @click="handleEdit(scope.row)" type="primary" size="small">编辑</el-button>
                <el-button @click="handleDelete(scope.row)" type="danger" size="small">删除</el-button>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
    
    <!-- 小屏幕：卡片布局 -->
    <div class="card-layout" v-else>
      <el-row :gutter="20">
        <el-col :xs="24" :sm="12" :md="8" v-for="item in atlasData" :key="item.id">
          <el-card class="atlas-card" shadow="hover">
            <!-- 图片 -->
            <template #header>
              <div class="card-header">
                <h3 class="card-title">{{ item.title }}</h3>
              </div>
            </template>
            <div class="card-image-container">
              <el-image
                :src="item.path"
                class="card-image"
                :preview-src-list="[item.path]"
                preview-teleported
              />
            </div>
            
            <!-- 卡片内容 -->
            <div class="card-content">
              <!-- 基本信息 -->
              <div class="card-info">
                <div class="info-item">
                  <span class="info-label">作者：</span>
                  <span class="info-value">{{ item.author }}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">上传时间：</span>
                  <span class="info-value">{{ item.upload_time }}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">点赞数：</span>
                  <span class="info-value">{{ item.likes }}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">审核状态：</span>
                  <span class="info-value status-value">
                    <el-switch
                      :model-value="item.type"
                      :active-value="1"
                      :inactive-value="0"
                      @change="(newValue) => photoStateChanged(item, newValue)"
                      size="small"
                    />
                  </span>
                </div>
              </div>
              
              <!-- 简介 -->
              <div class="card-description">
                <span class="info-label">简介：</span>
                <p class="description-text">{{ item.description }}</p>
              </div>
              
              <!-- 标签 -->
              <div class="card-tags">
                <span class="info-label">标签：</span>
                <div class="tags-container">
                  <el-tag size="small" v-for="(tag, i) in item.tags" :key="tag.id" closable
                          @close="handleClose(i, item)">
                    {{ tag.name }}
                  </el-tag>
                  <!-- 新标签输入框 -->
                  <el-input size="small" style="width: 100px;" class="input-new-tag" v-if="item.inputVisible"
                            v-model="item.inputValue" ref="saveTagInput"
                            @keyup.enter="handleInputConfirm(item)"
                            @blur="handleInputConfirm(item)">
                  </el-input>
                  <!-- 新标签按钮 -->
                  <el-button v-else size="small" class="button-new-tag" @click="showInput(item)">
                    + 新标签
                  </el-button>
                </div>
              </div>
            </div>
            
            <!-- 操作按钮 -->
            <div class="card-actions">
              <el-button @click="handleEdit(item)" type="primary" size="small">编辑</el-button>
              <el-button @click="handleDelete(item)" type="danger" size="small">删除</el-button>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>
    
    <!-- 编辑对话框 -->
    <el-dialog v-model="dialogVisible" title="编辑图集项" destroy-on-close>
      <el-form :model="currentItem">
        <el-form-item label="作者">
          <el-input v-model="currentItem.author"></el-input>
        </el-form-item>
        <el-form-item label="标题">
          <el-input v-model="currentItem.title"></el-input>
        </el-form-item>
        <el-form-item label="简介">
          <el-input v-model="currentItem.description"></el-input>
        </el-form-item>
        <el-form-item label="点赞量">
          <el-input v-model="currentItem.likes"></el-input>
        </el-form-item>
        <el-form-item label="审核状态">
          <el-switch
            v-model="currentItem.type"
            :active-value="1"
            :inactive-value="0"
          ></el-switch>
        </el-form-item>
        <el-form-item label="用户名">
          <el-input v-model="currentItem.username"></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="saveEdit">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref,  onMounted, nextTick, onUnmounted } from 'vue';
import { ENDPOINTS } from '@/api/api';
import { useUserStore } from '@/store/store';
import { ElMessage,ElMessageBox } from 'element-plus';
import api from '@/api/interceptor';

const userStore = useUserStore();
interface Tag {
  id: number | null;
  name: string;
}
// 图集项接口定义，描述每个图集项的结构
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
  inputVisible?: boolean;
  inputValue?: string;
}
// 存储从 API 获取的图集数据
const atlasData = ref<AtlasItem[]>([]);
const dialogVisible = ref(false);  // 是否显示编辑图集项的对话框
const currentItem = ref<AtlasItem | null>(null);
const saveTagInput = ref(null);  // 保存输入框的引用，以便在失去焦点时触发确认输入标签的事件

// 响应式布局判断
const isDesktop = ref(window.innerWidth >= 768);

// 监听窗口大小变化
const handleResize = () => {
  isDesktop.value = window.innerWidth >= 768;
};

// 加载图集数据
onMounted(async () => {
  try {
    const response = await api.get<{ data: AtlasItem[] }>(ENDPOINTS.ADMINSHOW, {
    });

    if (response.status === 200) {  
      // 确保type字段是数字类型
      atlasData.value = response.data.data.map(item => ({
        ...item,
        type: Number(item.type),
        inputVisible: false,
        inputValue: ''
      }));
    } else {
      console.error('错误的数据结构:', response.data);
    }
  } catch (error) {
    console.error('获取图集数据失败:', error.message);
  }
  
  // 添加窗口大小监听
  window.addEventListener('resize', handleResize);
});

// 组件卸载时移除监听
onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});

// 显示输入框以添加新标签
const showInput = (row) => {
      row.inputVisible = true; // 显示输入框
      nextTick(() => {
        if (saveTagInput.value) {
          saveTagInput.value.focus(); // 聚焦输入框
        }
      });
};

// 确认输入标签，失去焦点或按下回车键时触发
const handleInputConfirm = async (row) => {
  // 验证输入框是否为空
  if (!row.inputValue) {  
          row.inputValue = '';
          row.inputVisible = false;
          return
      }
  const inputValue = row.inputValue.trim();
  if (inputValue !== "") {
    // 添加标签到本地
    row.tags.push({ id: Date.now(), name: inputValue });
    // 发送请求到后台添加新标签
    try {
      const response = await api.post(ENDPOINTS.ADDTAG, {
        filesId: row.id,   // 图片的id
        tagName: inputValue // 新标签的name
      });
      row.inputValue = ""; // 清空输入框
      row.inputVisible = false; // 隐藏输入框，显示新标签按钮
      if (response.data.status === 830) {
        ElMessage.success('标签添加成功');
      } else {
        row.tags.pop(); // 删除新增失败的标签
        ElMessage.error('标签添加失败');
      }
    } catch (error) {
      row.inputValue = ""; // 清空输入框
      row.inputVisible = false; // 隐藏输入框，显示新标签按钮
      row.tags.pop(); // 删除新增失败的标签
      ElMessage.error('请求失败');
    }
  }
};

/**
 * 处理关闭标签的操作。
 * @param {number} index - 标签在 tags 数组中的索引。
 * @param {Object} row - 当前行的数据对象，包含标签信息。
 * @returns {Promise<void>} - 返回一个 Promise，表示操作的异步结果。
 * @throws {Error} - 如果删除标签的请求失败，将抛出错误。
 * @description 
 * 此函数会弹出确认对话框，询问用户是否确认删除标签。
 * 如果用户确认，发送删除请求到服务器，并根据响应结果更新本地标签列表。
 * 如果用户取消操作，则显示取消删除的提示信息。
 */
const handleClose = async (index, row) => {   
  try {
    const confirmResult = await ElMessageBox.confirm(
      '此操作将永久删除该标签, 是否继续',
      '提示',
      {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }
    );
    if(confirmResult === 'confirm') {
      const tag = row.tags[index];
    const response = await api.post(ENDPOINTS.DELETEPHOTOTAG, {
      filesId: row.id, // 图片的id
      tagId: tag.id // 标签的id
    });

    if (response.data.status === 830) {
      row.tags.splice(index, 1); // 删除本地标签
      ElMessage.success('标签删除成功');
    } else {
      ElMessage.error('标签删除失败');
    }
    }else{
      ElMessage.success('已取消删除');}
  } catch (error) {
        // 如果用户点击了取消或关闭对话框,则不弹出警告
        if (error !== 'cancel' && error !== 'close') {
      console.error('删除标签失败：', error.message);
      ElMessage.error('删除失败');
  }
  }
};

/**
 * 处理照片状态变化的异步函数。
 * @param {Object} row - 当前行的数据对象，包含照片信息。
 * @param {number} newValue - 新的照片类型值（1或0）。
 * 该函数首先保存旧的照片类型值，然后乐观更新 UI。接着，向服务器发送请求以更改照片类型。
 * 如果请求成功且状态为 830，显示成功消息；否则，恢复旧值并显示错误消息。
 * 在请求出错的情况下，也会恢复旧值并提示用户服务器繁忙。
 */
const photoStateChanged = async (row, newValue) => {
  // 保存旧值
  const oldValue = row.type;
  // 先乐观更新 UI
  row.type = Number(newValue);
  try {
    const { data: res } = await api.post(ENDPOINTS.CHANGEPHOTOTYPE, { photo: row });
    if (res.status !== 830) {
      // 审核失败，恢复旧值
      row.type = oldValue;
      ElMessage.error("审核失败");
    } else {
      ElMessage.success("审核成功");
    }
  } catch (error) {
    // 请求出错，恢复旧值
    row.type = oldValue;
    ElMessage.error("请求出错！服务器繁忙，请稍后再试。");
  }
};

// 自定义点赞数状态排序逻辑
const likesSort = (a: AtlasItem, b: AtlasItem) => {
  return Number(a.likes) - Number(b.likes); // 确保 likes 是数字进行排序
};
// 自定义审核状态排序逻辑
const typeSort = (a: AtlasItem, b: AtlasItem) => {
  return a.type - b.type; // type 已经是数字类型
};

// 编辑图集项
const handleEdit = async (item: AtlasItem) => {
  currentItem.value = { ...item };
  dialogVisible.value = true;
  await nextTick(); // 强制 DOM 更新，避免 ResizeObserver 报错
};

/**
 * 保存编辑后的项目数据。
 * 该函数会检查当前项目是否存在，如果存在，则通过 Axios 向服务器发送更新请求。
 * 请求成功后，会更新本地数据并关闭对话框，同时显示成功消息。
 * 如果请求失败，则会在控制台输出错误信息，并显示失败消息。
 * @async
 * @function saveEdit
 * @returns {Promise<void>} 无返回值
 */
const saveEdit = async () => {
  if (currentItem.value) {
    try {
      // 确保type是数字类型
      const itemToSave = {
        ...currentItem.value,
        type: Number(currentItem.value.type)
      };
      
      const response = await api.post(ENDPOINTS.ADMINUPDATE, itemToSave, {
        headers: {
          'Authorization': `Bearer ${userStore.token}`
        }
      });
      if (response.data.status === 830) {
        const index = atlasData.value.findIndex(item => item.id === currentItem.value!.id);
        if (index !== -1) {
          atlasData.value[index] = { ...itemToSave };
        }
        dialogVisible.value = false;
        ElMessage.success('更新成功');
      } else {
        console.error('图片数据更新失败:', response.data.message);
        ElMessage.error('更新失败');
      }
    } catch (error) {
      console.error('图片数据更新出错:', error.message);
      ElMessage.error('更新失败');
    }
  }
};

/**
 * 处理删除操作的函数 
 * @param {AtlasItem} item - 要删除的照片项
 * @returns {Promise<void>} - 无返回值
 * @throws {Error} - 如果删除操作失败，将抛出错误
 * 此函数会弹出确认对话框，询问用户是否确认删除照片。
 * 如果用户确认，将发送删除请求，并根据响应结果更新状态和提示用户。
 * 如果用户取消操作或关闭对话框，将不会执行删除操作。
 */
const handleDelete = async (item: AtlasItem) => {
  // 弹出对话框
  try {
    const confirmResult = await ElMessageBox.confirm(
      '此操作将永久删除该照片, 是否继续',
      '提示',
      {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }
    );
    // 如果用户点击了确定
    if (confirmResult === 'confirm') {
      const response = await api.post(ENDPOINTS.DELETEPHOTO,  {
      fileId: item.id, // 图片的id 
    },{
        headers: {
          Authorization: `Bearer ${userStore.token}`,
        },
      });

      if (response.data.status === 830) {
        atlasData.value = atlasData.value.filter((i) => i.id !== item.id);
        ElMessage.success('删除成功');
      } else {
        console.error('删除图片失败:', response.data.message);
        ElMessage.error('删除失败');
      }
    } else {
      ElMessage.info('已取消删除');
    }
  } catch (error) {
    // 如果用户点击了取消或关闭对话框,则不弹出警告
    if (error !== 'cancel' && error !== 'close') {
      console.error('删除失败:', error.message);
      ElMessage.error('删除失败');
    }
  }
};
</script>

<style scoped>
/* 统一字体样式 */
.admin-atlas-container,
.admin-atlas-container .el-input__inner,
.admin-atlas-container .el-input__inner::placeholder,
.admin-atlas-container .el-button,
.admin-atlas-container .el-card,
.admin-atlas-container .el-dialog,
.admin-atlas-container .el-form,
.admin-atlas-container .el-message-box {
  font-family: Courier,Kaiti, monospace,CustomFont;
}

/* 响应式布局样式 */
.admin-atlas-container {
  width: 100%;
  padding: 10px;
  max-width: 100%;
  overflow-x: hidden;
  box-sizing: border-box;
}

/* 防止 el-row gutter 导致溢出 */
.admin-atlas-container .el-row {
  width: 100% !important;
  margin-left: 0 !important;
  margin-right: 0 !important;
}

.admin-atlas-container .el-col {
  padding-left: 10px;
  padding-right: 10px;
}

/* 表格布局样式 */
.table-container {
  overflow-x: auto;
  font-family: Courier,Kaiti, monospace,CustomFont;
}

/* 让图片保持一定尺寸 */
.image-preview {
  height:120px;
  width: auto;
  min-width: 120px;
  border-radius: 6px;
  object-fit: cover;
}

/* 卡片布局样式 */
.card-layout {
  width: 100%;
}

.atlas-card {
  margin-bottom: 15px;
  transition: all 0.3s ease;
}

.atlas-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
}

.card-header {
  padding: 5px 0;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  margin: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.card-image-container {
  margin: 10px 0;
  border-radius: 6px;
  overflow: hidden;
}

.card-image {
  width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 6px;
}

.card-content {
  padding: 5px 0;
}

.card-info {
  margin-bottom: 10px;
}

.info-item {
  display: flex;
  align-items: center;
  margin-bottom: 6px;
  font-size: 13px;
}

.info-label {
  font-weight: 500;
  margin-right: 6px;
  color: #606266;
  min-width: 60px;
}

.info-value {
  color: #303133;
  flex: 1;
}

.status-value {
  display: flex;
  align-items: center;
}

.card-description {
  margin-bottom: 10px;
}

.description-text {
  margin: 3px 0 0 0;
  font-size: 13px;
  color: #303133;
  line-height: 1.3;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-box-orient: vertical;
}

.card-tags {
  margin-bottom: 10px;
}

.tags-container {
  margin-top: 3px;
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.card-actions {
  display: flex;
  gap: 8px;
  margin-top: 10px;
}

/* 媒体查询：根据屏幕尺寸显示不同布局 */
@media screen and (max-width: 768px) {
  .admin-atlas-container {
    padding: 5px;
  }
  
  .table-layout {
    display: none;
  }
  
  .card-layout {
    display: block;
  }
  
  .card-image {
    height: 160px;
  }
}

@media screen and (min-width: 769px) {
  .table-layout {
    display: block;
  }
  
  .card-layout {
    display: none;
  }
}
</style>