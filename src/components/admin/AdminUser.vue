<template>
  <div class="admin-user-container">
    <el-card shadow="never">
      <!-- 搜索区域 -->
      <el-row :gutter="5" class="search-row">
        <el-col :xs="20" :sm="8" :md="6">
          <el-input placeholder="请输入用户名" v-model="search" />
        </el-col>
        <el-col :xs="4" :sm="3" :md="2">
          <el-button type="primary" @click="searchUser" class="search-button">  
            <el-icon><Search /></el-icon>
          </el-button>
        </el-col>
      </el-row>
      
      <!-- 用户卡片布局：根据屏幕宽度显示不同数量的列 -->
      <el-row :gutter="10">
        <el-col :xs="24" :sm="12" :md="8" :lg="6" v-for="user in paginatedUserList" :key="user.username">
          <el-card class="user-card" shadow="hover">
              <div class="user-card-content">
                <div class="user-info">
                  <div class="info-item">
                    <span class="info-label">用户名：</span>
                    <span class="info-value">{{ user.username }}</span>
                  </div>
                  <div class="info-item">
                    <span class="info-label">邮箱：</span>
                    <span class="info-value">{{ user.email }}</span>
                  </div>
                  <div class="info-item admin-status">
                    <span class="info-label">管理员：</span>
                    <span class="info-value">
                      <el-switch 
                        :model-value="user.type" 
                        :active-value="'1'" 
                        :inactive-value="'0'" 
                        :disabled="user.username === username"
                        @change="(newValue) => userStateChanged(user, newValue)"
                        size="small"
                      />
                    </span>
                  </div>
                </div>
                <div class="user-actions">
                  <el-button 
                    type="danger"
                    :icon="Delete"
                    @click="deleteUser(user.username)"
                    size="small"
                  >删除</el-button>
                </div>
              </div>
            </el-card>
          </el-col>
        </el-row>

      <!-- 分页组件 -->
      <el-pagination
        v-if="total > 0"
        :current-page="currentPage"
        :page-size="pageSize"
        :total="total"
        layout="total, prev, pager, next, jumper"
        @current-change="handlePageChange"
        class="pagination-center"
      />
    </el-card>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useUserStore } from '@/store/store';
import api from '@/api/interceptor';
import { ElMessage, ElMessageBox } from 'element-plus';
import { ENDPOINTS } from '@/api/api';
import { Delete,Search } from '@element-plus/icons-vue';

const userList = ref([]);
const search = ref('');
const username = useUserStore().username;
const currentPage = ref(1); // 当前页码
const pageSize = ref(10); // 每页显示条数
const total = ref(0); // 总条数

// 获取用户列表
const getUserList = async () => {
  try {
    const { data: res } = await api.get(ENDPOINTS.GETUSER);
    if (res.status === 830) {
      userList.value = res.data;
      total.value = res.data.length; // 获取总用户数
    } else {
      ElMessage.error("获取用户信息失败！");
    }
  } catch (error) {
    ElMessage.error("请求出错！");
  }
};

/**
 * 计算属性，用于根据搜索关键字过滤用户列表。
 * 如果没有搜索关键字，则返回原始用户列表。
 * 否则，使用正则表达式匹配用户的昵称或用户名，返回匹配的用户列表。
 * @computed {Array} filteredUserList - 过滤后的用户列表
 */
const filteredUserList = computed(() => {
  if (!search.value) return userList.value;
  const reg = new RegExp(search.value, 'i');
  return userList.value.filter(user => reg.test(user.nickname) || reg.test(user.username));
});

// 分页后的用户数据
const paginatedUserList = computed(() => {
  const startIndex = (currentPage.value - 1) * pageSize.value;
  return filteredUserList.value.slice(startIndex, startIndex + pageSize.value);
});

/**
 * 搜索用户并更新用户列表。
 * 当调用此函数时，将当前页重置为第一页，并根据输入的搜索条件过滤用户列表。
 * @function searchUser
 * @returns {void}
 */
const searchUser = () => {
  currentPage.value = 1; // 重置为第一页
  filteredUserList.value = userList.value.filter(user => user.username.includes(search.value));
};

const handlePageChange = (page) => {
  currentPage.value = page; // 更新当前页码
};

/**
 * 处理用户权限变更的异步函数。
 * @param {Object} row - 当前用户对象，包含用户信息。
 * @param {string} newValue - 新的用户类型，"1"为管理员，"0"为普通用户
 * @returns {Promise<void>} - 无返回值。
 * @throws {Error} - 如果请求失败，将会抛出错误。
 * 函数会尝试更新用户的类型，并根据响应结果显示相应的提示信息。
 * 如果请求失败或返回的状态不符合预期，将会恢复到旧的类型值。
 */
const userStateChanged = async (row, newValue) => {
  const oldValue = row.type;
  row.type = newValue;
  try {
    const { data: res } = await api.post(ENDPOINTS.CHANGEUSER, { user: row });
    if (res.status !== 830) {
      row.type = oldValue;
      ElMessage.error("修改用户类型失败");
    } else {
      ElMessage.success("修改用户类型成功");
    }
  } catch (error) {
    row.type = oldValue;
    ElMessage.error("请求出错！");
  }
};

/**
 * 删除指定用户名的用户。
 * 在确认后，向服务器发送删除请求。
 * 如果删除成功，显示成功消息并更新用户列表；如果失败，显示错误消息。
 * @param {string} username - 要删除的用户的用户名。
 * @returns {Promise<void>} - 无返回值。
 * @throws {Error} - 如果用户取消删除操作，将抛出错误。
 */
const deleteUser = async (username) => {
  try {
    await ElMessageBox.confirm('此操作将永久删除该用户, 是否继续', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
      center: true
    });

    const { data: res } = await api.post(ENDPOINTS.DELETEUSER, { username });
    if (res.code !== 200) {
      ElMessage.error('删除用户失败！');
    } else {
      ElMessage.success('删除用户成功！');
      getUserList();
    }
  } catch (error) {
    ElMessage.info('已取消删除');
  }
};

onMounted(() => {
  getUserList();
});
</script>

<style scoped>
.user-table {
  margin-top: 20px;
}

/* 添加分页居中样式 */
.pagination-center {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

/* 统一字体样式 */
.admin-user-container,
.admin-user-container .el-input__inner,
.admin-user-container .el-input__inner::placeholder,
.admin-user-container .el-button,
.admin-user-container .el-card,
.admin-user-container .el-dialog,
.admin-user-container .el-form,
.admin-user-container .el-message-box {
  font-family: Courier,Kaiti, monospace,CustomFont;
}

/* 响应式布局样式 */
.admin-user-container {
  width: 100%;
  padding: 10px;
  max-width: 100%;
  overflow-x: hidden;
  box-sizing: border-box;
}

/* 防止 el-row gutter 导致溢出 */
.admin-user-container .el-row {
  width: 100% !important;
  margin-left: 0 !important;
  margin-right: 0 !important;
}

.admin-user-container .el-col {
  padding-left: 5px;
  padding-right: 5px;
}

/* 搜索区域样式 */
.search-row {
  margin-bottom: 10px;
}

.search-button {
  width: 100%;
}

/* 表格布局样式 */
.card-layout {
  margin-top: 10px;
}

.user-card {
  margin-bottom: 10px;
  transition: all 0.3s ease;
}

.user-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
}

.user-card-content {
  padding: 8px 0;
}

.user-info {
  margin-bottom: 6px;
}

.info-item {
  display: flex;
  align-items: center;
  margin-bottom: 3px;
  font-size: 14px;
}

.info-label {
  font-weight: 500;
  margin-right: 4px;
  color: #606266;
  min-width: 55px;
}

.info-value {
  color: #303133;
  flex: 1;
}

.admin-status {
  margin-top: 3px;
}

.user-actions {
  display: flex;
  justify-content: flex-end;
  gap: 5px;
  margin-top: 6px;
}

/* 媒体查询：根据屏幕尺寸优化显示 */
@media screen and (max-width: 576px) {
  .admin-user-container {
    padding: 5px;
  }

  .user-card {
    padding: 8px;
  }

  .info-item {
    font-size: 13px;
    margin-bottom: 3px;
  }

  .info-label {
    min-width: 50px;
  }

  .user-actions {
    margin-top: 5px;
  }
}

@media screen and (min-width: 577px) and (max-width: 768px) {
  .admin-user-container {
    padding: 8px;
  }
}
</style>