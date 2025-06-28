<template>
  <div class="app-container">
    <el-card style="height: 86vh;">
      <!--      头部-->
      <div class="header">
        <el-row>
          <el-col>
            <el-form :inline="true">
              <el-form-item prop="keyword" label="服务器名称">
                <el-input
                    v-model="searchForm.keyword"
                    style="width: 240px"
                    placeholder="请输入服务器名称"
                    clearable
                    @keyup.enter="handleSearch"
                />
              </el-form-item>
              <el-form-item prop="categories" label="服务器类型">
                <el-select
                    v-model="searchForm.categoryId"
                    placeholder="请选择服务器类型"
                    style="width: 240px"
                    clearable
                >
                  <el-option
                      v-for="item in options"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                      :disabled="item.disabled"
                  />
                </el-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" icon="Search" @click="handleSearch">搜索</el-button>
                <el-button icon="Refresh" @click="handleReset">重置</el-button>
              </el-form-item>
            </el-form>
          </el-col>
        </el-row>
      </div>

      <!-- 表格和分页 -->
      <div class="table-container">
        <el-table :data="visibleData" border style="width: 100%">
          <el-table-column prop="name" label="服务器名称" width="260" align="center" />
          <el-table-column prop="categories" align="center" label="服务器类型" width="240">
            <template #default="{row}">
              {{ row.categories }}
            </template>
          </el-table-column>
          <el-table-column prop="cpuCapacity" label="CPU核数" width="130" align="center"/>
          <el-table-column prop="memoryCapacity" label="内存(GB)" width="140" align="center"/>
          <el-table-column prop="purpose" label="用途" width="140" align="center">
            <template #default="{row}">
              <el-tag :type="getTagType(row.purpose)">{{ row.purpose }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="storageCapacity" label="硬盘容量" align="center"/>
          <el-table-column label="使用说明" align="center">
            <template #default="{row}">
              <el-button type="primary" prop="description" size="small" @click="handleView(row)">
                查看
              </el-button>
              <el-button type="success" size="small" @click="handleBorrow(row)">
                借阅
              </el-button>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页组件 -->
        <Pagination
            v-model:page="currentPage"
            v-model:limit="pageSize"
            :total="totalItems"
            @pagination="handlePagination"
        />
      </div>
    </el-card>

    <!-- 查看对话框 -->
    <el-dialog
        v-model="viewDialogVisible"
        title="服务器详情"
        width="50%"
    >
      <div v-if="currentServer" class="server-detail">
        <el-descriptions :column="1" border>
          <el-descriptions-item label="服务器名称">{{ currentServer.name }}</el-descriptions-item>
          <el-descriptions-item label="服务器类型">{{ currentServer.categories }}</el-descriptions-item>
          <el-descriptions-item label="CPU核数">{{ currentServer.cpuCapacity }}</el-descriptions-item>
          <el-descriptions-item label="内存(GB)">{{ currentServer.memoryCapacity }}</el-descriptions-item>
          <el-descriptions-item label="用途">
            <el-tag :type="getTagType(currentServer.purpose)">{{ currentServer.purpose }}</el-tag>
          </el-descriptions-item>
          <el-descriptions-item label="硬盘容量">{{ currentServer.storageCapacity }}</el-descriptions-item>
          <el-descriptions-item label="IP地址">192.168.1.100</el-descriptions-item>
          <el-descriptions-item label="操作系统">CentOS 7.9</el-descriptions-item>
          <el-descriptions-item label="使用说明">
            <el-input
                type="textarea"
                :rows="3"
                placeholder="请输入使用说明"
                v-model="usageInstructions"
            />
          </el-descriptions-item>
        </el-descriptions>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="viewDialogVisible = false">关闭</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 借阅抽屉 -->
    <el-drawer
        v-model="borrowDrawerVisible"
        title="借阅服务器"
        direction="rtl"
        size="40%"
    >
      <div v-if="currentServer" class="borrow-form">
        <h3>借阅服务器: {{ currentServer.name }}</h3>
        <el-form :model="borrowForm" label-width="120px">
          <el-form-item label="借阅人姓名">
            <el-input v-model="borrowForm.borrowerName" placeholder="请输入姓名" />
          </el-form-item>
          <el-form-item label="借阅用途">
            <el-input v-model="borrowForm.purpose" placeholder="请输入借阅用途" />
          </el-form-item>
          <el-form-item label="预计归还时间">
            <el-date-picker
                v-model="borrowForm.returnDate"
                type="datetime"
                placeholder="选择归还时间"
            />
          </el-form-item>
          <el-form-item label="备注">
            <el-input
                type="textarea"
                :rows="3"
                placeholder="请输入备注信息"
                v-model="borrowForm.remarks"
            />
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitBorrow">提交借阅</el-button>
            <el-button @click="borrowDrawerVisible = false">取消</el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-drawer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { ElMessage } from 'element-plus'
import Pagination from '@/components/Pagination/index.vue'

// 服务器类型选项
const options = ref([
  { value: 'GPU服务器', label: 'GPU服务器' },
  { value: '计算服务器', label: '计算服务器' },
  { value: '存储服务器', label: '存储服务器' },
  { value: '数据库服务器', label: '数据库服务器' },
])

// 表格原始数据
const originalData = ref([
  {name: '阿里云8核64g', categories:'GPU服务器',cpuCapacity: '8', memoryCapacity: '64', purpose: '开发', storageCapacity: '500GB'},
  {name: '华为云8核64g', categories:'计算服务器',cpuCapacity: '8', memoryCapacity: '64', purpose: '实验', storageCapacity: '500GB'},
  {name: '腾讯云16核128g', categories:'存储服务器',cpuCapacity: '16', memoryCapacity: '128', purpose: '测试', storageCapacity: '1TB'},
  {name: 'AWS 4核32g', categories:'数据库服务器',cpuCapacity: '4', memoryCapacity: '32', purpose: '实验', storageCapacity: '256GB'},
  {name: '阿里云32核256g', categories:'GPU服务器',cpuCapacity: '32', memoryCapacity: '256', purpose: '开发', storageCapacity: '2TB'},
  {name: '华为云16核64g', categories:'计算服务器',cpuCapacity: '16', memoryCapacity: '64', purpose: '测试', storageCapacity: '1TB'},
  {name: 'Azure 8核16g', categories:'GPU服务器',cpuCapacity: '8', memoryCapacity: '16', purpose: '开发', storageCapacity: '500GB'},
  {name: '腾讯云4核8g', categories:'存储服务器',cpuCapacity: '4', memoryCapacity: '8', purpose: '实验', storageCapacity: '128GB'},
  {name: '阿里云2核4g', categories:'数据库服务器',cpuCapacity: '2', memoryCapacity: '4', purpose: '开发', storageCapacity: '64GB'},
  {name: '华为云32核128g', categories:'GPU服务器',cpuCapacity: '32', memoryCapacity: '128', purpose: '测试', storageCapacity: '2TB'},
  {name: 'AWS 16核32g', categories:'GPU服务器',cpuCapacity: '16', memoryCapacity: '32', purpose: '开发', storageCapacity: '1TB'},
  {name: '腾讯云8核16g', categories:'GPU服务器',cpuCapacity: '8', memoryCapacity: '16', purpose: '实验', storageCapacity: '256GB'},
])

// 表格数据（会根据搜索条件变化）
const tableData = ref([...originalData.value])

// 根据用途返回不同的tag类型
const getTagType = (purpose) => {
  switch(purpose) {
    case '测试': return 'danger'
    case '实验': return 'success'
    case '开发': return 'warning'
    default: return 'info'
  }
}

// 分页相关数据
const currentPage = ref(1)
const pageSize = ref(10)
const totalItems = ref(tableData.value.length)

// 处理分页变化
function handlePagination({ page, limit }) {
  currentPage.value = page
  pageSize.value = limit
}

// 计算当前页显示的数据
const visibleData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return tableData.value.slice(start, end)
})

// 搜索表单数据
const searchForm = ref({
  keyword: '',
  categoryId: ''
})

// 搜索方法
const handleSearch = () => {
  currentPage.value = 1 // 搜索时重置到第一页

  // 过滤数据
  const filteredData = originalData.value.filter(item => {
    const matchesKeyword = item.name.includes(searchForm.value.keyword)
    const matchesCategory = !searchForm.value.categoryId ||
        item.categories === searchForm.value.categoryId
    return matchesKeyword && matchesCategory
  })

  tableData.value = filteredData
  totalItems.value = filteredData.length

  if (filteredData.length === 0) {
    ElMessage.info('未找到匹配的服务器')
  }
}

// 重置方法
const handleReset = () => {
  searchForm.value = {
    keyword: '',
    categoryId: ''
  }
  tableData.value = [...originalData.value]
  totalItems.value = originalData.value.length
  currentPage.value = 1
}

// 查看对话框相关
const viewDialogVisible = ref(false)
const currentServer = ref(null)
const usageInstructions = ref('')

const handleView = (row) => {
  currentServer.value = row
  viewDialogVisible.value = true
}

// 借阅抽屉相关
const borrowDrawerVisible = ref(false)
const borrowForm = ref({
  borrowerName: '',
  purpose: '',
  returnDate: '',
  remarks: ''
})

const handleBorrow = (row) => {
  currentServer.value = row
  // 重置表单
  borrowForm.value = {
    borrowerName: '',
    purpose: '',
    returnDate: '',
    remarks: ''
  }
  borrowDrawerVisible.value = true
}

// 提交借阅
const submitBorrow = () => {
  if (!currentServer.value) return

  // 这里可以添加借阅逻辑，比如调用API
  ElMessage.success(`借阅申请已提交，服务器: ${currentServer.value.name}`)
  borrowDrawerVisible.value = false
}
</script>

<style scoped lang="scss">
.app-container {
  .header{
    margin: 15px;
  }

  .server-detail {
    padding: 10px;

    .el-descriptions {
      margin-top: 10px;
    }
  }

  .borrow-form {
    padding: 20px;

    h3 {
      margin-bottom: 20px;
      text-align: center;
    }

    .el-form-item {
      margin-bottom: 20px;
    }
  }
}
</style>