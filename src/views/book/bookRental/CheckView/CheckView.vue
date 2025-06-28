<template>
  <el-button
      type="primary"
      size="small"
      @click="checkView(props.rowData)"
  >
    查看
  </el-button>

  <el-dialog
      v-model="centerDialogVisible"
      width="700px"
      align-center
      append-to-body
      class="book-detail-dialog"
  >
    <template #header>
      <span style="font-weight: bold">图书详细信息</span>
    </template>
    <div class="book-detail-container">
      <!-- 封面图片 -->
      <div class="book-cover">
        <img
            :src="bookData.coverImagePath || 'https://via.placeholder.com/150x200?text=封面'"
            alt="图书封面"
        >
      </div>

      <!-- 详细信息表格（两列布局） -->
      <table class="server-style-table">
        <tbody>
        <tr>
          <td class="label">书名</td>
          <td class="value">{{ bookData.title }}</td>
        </tr>
        <tr>
          <td class="label">作者</td>
          <td class="value">{{ bookData.author }}</td>
        </tr>
        <tr>
          <td class="label">ISBN</td>
          <td class="value">{{ bookData.isbn }}</td>
        </tr>
        <tr>
          <td class="label">出版社</td>
          <td class="value">{{ bookData.publisher }}</td>
        </tr>
        <tr>
          <td class="label">出版年份</td>
          <td class="value">{{ bookData.publishYear }}</td>
        </tr>
        <tr>
          <td class="label">总数量</td>
          <td class="value">{{ bookData.totalCount }}</td>
        </tr>
        <tr>
          <td class="label">已借出</td>
          <td class="value">{{ bookData.borrowedCount }}</td>
        </tr>
        <tr>
          <td class="label">可借数量</td>
          <td class="value">{{ bookData.totalCount - bookData.borrowedCount }}</td>
        </tr>
        <tr>
          <td class="label">书架位置</td>
          <td class="value">{{ bookData.location }}</td>
        </tr>
        <tr>
          <td class="label">索书号</td>
          <td class="value">{{ bookData.callNumber }}</td>
        </tr>
        </tbody>
      </table>
    </div>

    <template #footer>
      <el-button @click="centerDialogVisible = false">关闭</el-button>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref } from 'vue'
import axios from "axios"

const bookData = ref({})
const centerDialogVisible = ref(false)

const props = defineProps({
  rowData: {
    type: Object,
    required: true
  }
})

const checkView = async (data) => {
  try {
    const params = data.bookId ? { bookId: data.bookId } : {}
    const response = await axios.get('/dev-api/api/student/book/bookInfo', { params })
    bookData.value = response.data.data
    centerDialogVisible.value = true
  } catch (error) {
    console.error("获取图书信息失败:", error)
  }
}
</script>

<style scoped>
.book-detail-dialog {
  text-align: center;
}

.book-detail-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.book-cover {
  margin-bottom: 20px;
}

.book-cover img {
  width: 150px;
  height: 200px;
  object-fit: cover;
}

.server-style-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  text-align: left;
}

.server-style-table td {
  padding: 12px 15px;
  border: 1px solid #e0e0e0;
}

.server-style-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.server-style-table tr:hover {
  background-color: #f5f5f5;
}


.label {
  width: 30%;
  font-weight: bold;
  color: #333;
  background-color: #f5f7fa;
}

.value {
  width: 70%;
  color: #555;
}

/*.el-button {*/
/*  margin-top: 20px;*/
/*}*/
</style>
