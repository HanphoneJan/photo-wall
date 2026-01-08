<template>
  <div class="admin-tag-container">
    <el-card shadow="never">
      <!-- 搜索和新建标签区域 -->
      <el-row :gutter="10" class="search-row">
        <el-col :xs="14" :sm="16" :md="18">
          <el-input 
            placeholder="请输入标签名称" 
            v-model="search" 
            @keyup.enter="searchTags"
          />
        </el-col>
        <el-col :xs="5" :sm="4" :md="3">
          <el-button type="primary" @click="searchTags" class="search-button">
            <el-icon><Search /></el-icon>
            <span style="margin-left: 5px;">搜索</span>
          </el-button>
        </el-col>
        <el-col :xs="5" :sm="4" :md="3">
          <el-button type="primary" @click="createTagDialogFormVisible = true" class="create-button">
            <el-icon><Plus /></el-icon>
            <span style="margin-left: 5px;">新建</span>
          </el-button>
        </el-col>
      </el-row>
      <!-- 标签卡片布局：根据屏幕宽度显示不同数量的列 -->
      <el-row :gutter="10" style="margin-top: 20px;">
        <el-col 
          :xs="12" 
          :sm="8" 
          :md="6" 
          :lg="4" 
          v-for="tag in filteredTagList" 
          :key="tag.id" 
          class="tag-col"
        >
          <el-card class="atlas-card" shadow="hover">
            <div class="op">
              <el-button @click="editTagById(tag.id)" :icon="Edit" style="color: #3a8ee6;margin-right: 5px"></el-button>
              <el-button @click="deleteTagById(tag.id)" :icon="Delete" style="color: red"></el-button>
            </div>
            <b> 标签名称 <span style="color: #3a8ee6">{{ tag.name }}</span></b> <br>
            <p style="margin-bottom: 0">图片数量 <span style="color: #2ac06d">{{ tag.number }}</span></p>
          </el-card>
        </el-col>
      </el-row>
      <!-- /**
       * @description 修改标签对话框组件
       * @component AdminTag
       * @property {Boolean} createTagDialogFormVisible - 控制对话框的显示与隐藏
       * @property {Object} createTagForm - 表单数据模型
       * @property {Object} createTagFormRules - 表单验证规则
       * @method backPage - 关闭对话框，返回上一页
       * @method createTag - 提交表单数据以创建新标签
       */ -->
      <el-dialog title="修改标签" v-model="createTagDialogFormVisible">
        <el-form style="text-align: left" ref="createTagFormRef" :model="createTagForm" :rules="createTagFormRules" label-width="80px">
          <el-form-item label="标签名称" prop="name">
            <el-input v-model="createTagForm.name"></el-input>
          </el-form-item>
        </el-form>
        <template #footer>
          <el-button @click="backPage">取消</el-button>
          <el-button type="primary" @click="createTag">提交</el-button>
        </template>
      </el-dialog>
    </el-card>
  </div>
</template>
  
<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import api from '@/api/interceptor';
import { ENDPOINTS } from '@/api/api';
import { ElMessage, ElMessageBox } from 'element-plus';
import { Delete, Edit, Search, Plus } from '@element-plus/icons-vue';
const tagList = ref([]);
const createTagDialogFormVisible = ref(false);
const createTagFormRef = ref(null);
const createTagForm = ref({ name: '' });
const createTagFormRules = {
    name: [{ required: true, message: '标签名称不能为空', trigger: 'blur' }],
};
const tag = ref({ id: null, name: '' });

// 搜索功能
const search = ref('');

/**
 * 计算属性，用于根据搜索关键字过滤标签列表。
 * 如果没有搜索关键字，则返回原始标签列表。
 * 否则，返回名称包含搜索关键字的标签列表。
 * @computed {Array} filteredTagList - 过滤后的标签列表
 */
const filteredTagList = computed(() => {
  if (!search.value) return tagList.value;
  const searchValue = search.value.toLowerCase();
  return tagList.value.filter(tag => tag.name.toLowerCase().includes(searchValue));
});

/**
 * 搜索标签的方法。
 * 该方法会根据输入框中的值过滤标签列表。
 */
const searchTags = () => {
  // 过滤逻辑由 computed 属性自动处理
};
  
 /**
  * 比较函数，用于根据指定属性的长度对两个对象进行排序。
  * @param {string} property - 要比较的属性名。
  * @returns {function} 返回一个比较函数，适用于数组的排序方法。
  */
const compare = (property) => (a, b) => b[property].length - a[property].length;
  
/**
 * 异步获取标签列表的函数。
 * 该函数通过发送网络请求获取标签数据，并根据响应状态更新标签列表。
 * 如果请求成功且状态为 830，则将返回的数据赋值给 tagList；否则，显示错误信息。
 * 在请求失败时，会捕获错误并显示相应的提示信息。
 * @async
 * @function getFullTagList
 * @returns {Promise<void>} 无返回值
 */
const getFullTagList = async () => {
  try {
    // 发送请求获取标签数据
    const { data: res } = await api.get(ENDPOINTS.GETADMINTAG);

    // 检查响应状态
    if (res.status === 830) {
      // 将返回的数据赋值给 tagList
      tagList.value = res.data;
      console.log('标签列表加载成功');
    } else {
      // 如果 status 不为 830，表示获取数据失败
      ElMessage.error('标签列表加载失败');
    }
  } catch (error) {
    // 捕获网络请求的错误
    console.error('获取标签列表失败:', error.message);
    ElMessage.error('获取标签列表失败，请稍后再试');
  }
};
  
const editTagById = (id) => {
  tag.value.id = id;
  createTagDialogFormVisible.value = true;
};
  
/**
 * 创建标签的异步函数。
 * 该函数首先验证表单，如果表单有效，则将标签名称设置为表单中的值。
 * 然后通过 POST 请求将标签数据发送到服务器。
 * 如果服务器返回状态为 830，表示创建成功，则关闭对话框，重置表单字段，并显示成功消息。
 * 最后，调用获取完整标签列表的函数以更新标签列表。
 */
const createTag = async () => {
    if (!createTagFormRef.value) return;
    createTagFormRef.value.validate(async (valid) => {
      if (!valid) return;
      tag.value.name = createTagForm.value.name;
      const { data: res } = await api.post(ENDPOINTS.CREATETAG, { tag: tag.value });
      if (res.status === 830) {
        // 关闭对话框，重置表单字段，显示成功消息
        createTagDialogFormVisible.value = false;
        tag.value.id = null;
        createTagFormRef.value.resetFields();
        ElMessage.success("创建成功");
        await getFullTagList();
      }
    });
};
  
/**
 * 根据标签 ID 删除标签。
 * @param {string} tagId - 要删除的标签的 ID。
 * @throws {Error} 如果用户取消操作，则抛出错误。
 */
const deleteTagById = async (tagId) => {
    try {
      await ElMessageBox.confirm('此操作将永久删除该标签, 是否继续', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      });
      const { data: res } = await api.post(ENDPOINTS.DELETETAG, { tagId });
      if (res.status === 830) {
        await getFullTagList();
      }
    } catch {
      ElMessage.info('已取消删除');
    }
};
  
const backPage = () => {
    createTagDialogFormVisible.value = false;
    createTagForm.value.name = '';
    createTagFormRef.value?.resetFields();
};

// 在组件挂载后获取完整标签列表
onMounted(getFullTagList);
</script>
  
<style scoped>
/* 统一字体样式 */
.admin-tag-container,
.admin-tag-container .el-input__inner,
.admin-tag-container .el-button,
.admin-tag-container .el-card,
.admin-tag-container .el-dialog,
.admin-tag-container .el-form,
.admin-tag-container .el-message-box {
  font-family: Courier,Kaiti, monospace,CustomFont;
}

.op:hover {
  cursor: pointer;
}

.op i:hover {
  font-weight: bold;
  font-size: large;
}

/* 响应式布局样式 */
.admin-tag-container {
  width: 100%;
  padding: 10px;
  max-width: 100%;
  overflow-x: hidden;
  box-sizing: border-box;
}

/* 防止 el-row gutter 导致溢出 */
.admin-tag-container .el-row {
  width: 100% !important;
  margin-left: 0 !important;
  margin-right: 0 !important;
}

.admin-tag-container .el-col {
  padding-left: 10px;
  padding-right: 10px;
}

/* 搜索区域样式 */
.search-row {
  margin-bottom: 0;
  display: flex;
  align-items: center;
  flex-wrap: nowrap;
}

.search-row .el-col {
  display: flex;
  flex-wrap: nowrap;
}

.search-button,
.create-button {
  width: 100%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 标签卡片样式 */
.tag-col {
  margin-bottom: 15px;
}

.atlas-card {
  margin-bottom: 10px;
  transition: all 0.3s ease;
  position: relative;
  text-align: left;
  height: 100%;
  min-height: 70px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 10px 12px;
  border-radius: 6px;
}

.atlas-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
}

/* 标签信息样式 */
.atlas-card b {
  font-size: 14px;
  margin-bottom: 4px;
  display: block;
  color: #303133;
}

.atlas-card p {
  font-size: 13px;
  margin-top: 3px;
  margin-bottom: 0;
  color: #303133;
}

/* 标签名称和数量样式 */
.atlas-card span {
  font-weight: normal;
}

/* 操作按钮样式 */
.op {
  position: absolute;
  right: 6px;
  top: 6px;
  display: flex;
  gap: 3px;
}

.op .el-button {
  padding: 0;
  min-width: auto;
  background: transparent;
  border: none;
}

/* 媒体查询：根据屏幕尺寸优化显示 */
@media screen and (max-width: 576px) {
  .admin-tag-container {
    padding: 5px;
  }

  .atlas-card {
    padding: 8px 10px;
    min-height: 65px;
  }

  .atlas-card b {
    font-size: 13px;
    margin-bottom: 3px;
  }

  .atlas-card p {
    font-size: 12px;
    margin-top: 2px;
  }

  .op {
    right: 4px;
    top: 4px;
  }

  .admin-tag-container .el-col {
    padding-left: 5px;
    padding-right: 5px;
  }
}
</style>
  