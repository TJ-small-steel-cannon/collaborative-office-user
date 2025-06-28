<template>
  <el-drawer v-model="drawerVisible" :direction="direction" append-to-body>
    <template #header>
      <h1>图书借阅</h1>
    </template>
    <template #default>
      <div class="book-info">
        <el-descriptions :column="1" border>
          <el-descriptions-item label="书名">{{ bookData.title }}</el-descriptions-item>
          <el-descriptions-item label="作者">{{ bookData.author }}</el-descriptions-item>
          <el-descriptions-item label="出版社">{{ bookData.publisher }}</el-descriptions-item>
          <el-descriptions-item label="ISBN">{{ bookData.isbn }}</el-descriptions-item>
          <el-descriptions-item label="出版年份">{{ bookData.publishYear }}</el-descriptions-item>
          <el-descriptions-item label="馆藏位置">{{ bookData.location }}</el-descriptions-item>
          <el-descriptions-item label="总数量">{{ bookData.totalCount }}</el-descriptions-item>
          <el-descriptions-item label="已借出">{{ bookData.borrowedCount }}</el-descriptions-item>
          <el-descriptions-item label="可借数量">{{ (bookData.totalCount || 0) - (bookData.value?.borrowedCount || 0) }}</el-descriptions-item>
        </el-descriptions>
      </div>

      <el-divider />

      <el-form :model="borrowForm" label-width="120px" label-position="left">
        <el-form-item>
          <template #label>
            <div style="display: flex; align-items: center;">
              <el-icon size="24"><Calendar /></el-icon>
              <span>归还日期</span>
            </div>
          </template>
          <el-date-picker
              v-model="borrowForm.expectedReturnDate"
              type="date"
              placeholder="选择归还日期"
              format="YYYY-MM-DD"
              value-format="YYYY-MM-DD"
              :disabled-date="disabledDate"
          />
        </el-form-item>
      </el-form>
    </template>
    <template #footer>
      <el-button @click="drawerVisible = false">取消</el-button>
      <el-button type="primary" @click="handleBorrow" :loading="loading">确认借阅</el-button>
    </template>
  </el-drawer>
</template>

<script setup>
import { ref } from 'vue';
import { ElMessage } from 'element-plus';
import axios from 'axios';

const drawerVisible = ref(false);
const direction = 'rtl';
const loading = ref(false);
const bookData = ref({});
const borrowForm = ref({
  expectedReturnDate: ''
});

const disabledDate = (time) => {
  return time.getTime() < Date.now() - 8.64e7;
};

const openDrawer = (rowData) => {
  borrowForm.value.expectedReturnDate = '';
  bookData.value = rowData;
  console.log(bookData.value);
  drawerVisible.value = true;
};

const handleBorrow = async () => {
  if (!borrowForm.value.expectedReturnDate) {
    ElMessage.warning('请选择归还日期');
    return;
  }

  loading.value = true;
  try {
    const params = {
      bookId: bookData.value.bookId, // 虽然不显示但仍用于请求
      expectedReturnDate: borrowForm.value.expectedReturnDate
    };

    const response = await axios.get('/dev-api/api/student/book/borrowBook', { params });

    if (response.data.code === 200) {
      ElMessage.success('借阅成功');
      drawerVisible.value = false;
    } else {
      ElMessage.error(response.data.msg || '借阅失败');
    }
  } catch (error) {
    ElMessage.error('请求失败: ' + error.message);
  } finally {
    loading.value = false;
  }
};

defineExpose({ openDrawer });
</script>

<style scoped>
.el-date-picker {
  width: 100%;
}

.book-info {
  margin-bottom: 20px;
}

.el-descriptions {
  margin-top: 10px;
}

.el-divider {
  margin: 20px 0;
}
</style>
