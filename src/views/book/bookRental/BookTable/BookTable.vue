<template>
  <div>
    <search-and-filter  @search-result="handleSearchResult"></search-and-filter>
  </div>
  <div>
    <el-table :data="bookList" border v-loading="loading">
      <el-table-column prop="title" label="书名" />
      <el-table-column prop="author" label="作者" />
      <el-table-column prop="publisher" label="出版社" />
      <el-table-column prop="isbn" label="索书号" />
      <el-table-column label="操作" >
        <template #default="scope">

          <CheckView :row-data="scope.row"></CheckView>
<!--          借阅-->
          <Borrowing ref="borrowDrawer" />

          <el-button
              type="success"
              size="small"
              @click="openBorrowDrawer(scope.row)"
          >
            借阅
          </el-button>

        </template>
      </el-table-column>
    </el-table>
  </div>
  <div>
    <!-- 分页组件 -->
<!--    <Pagination-->
<!--        @pagechange="handpagechange"-->
<!--    />-->
      <!--  分页器-->
      <pagination
          v-show="total > 0"
          :total="total"
          v-model:page="queryParams.pageNum"
          v-model:limit="queryParams.pageSize"
          @pagination="getList"
      />
  </div>

</template>

<script setup>
import { ref, onMounted ,reactive} from 'vue'
import SearchAndFilter from '@/views/book/bookRental/SearchAndFilter/SearchAndFilter.vue'
// import Pagination from '@/views/book/bookRental/Pagination/Pagination.vue'
import CheckView from '@/views/book/bookRental/CheckView/CheckView.vue'
import Borrowing from '@/views/book/bookRental/Borrowing/Borrowing.vue'
import axios from 'axios'


//分页器数据
const total = ref(30)

const queryParams = reactive({
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    keyword: '',
  }
})
//结构赋值
// const {queryParams} = toRefs(data)
const bookList = ref([])
const loading = ref(false)
//借阅
const borrowDrawer = ref(null);
// 直接传入整个行数据对象
const openBorrowDrawer = (rowData) => {
  borrowDrawer.value.openDrawer(rowData);
};
//搜索框查到的数据
const handleSearchResult = (data) => {

  console.log("搜索查到的数据是"+data)
  bookList.value = data
}
//翻页获得的新数据
// const handpagechange = (data)=>{
//   console.log("搜索查到的数据是"+data)
//   bookList.value = data
// }
const getList = async () => {
  try {
    // 构造请求参数
    const params = {
      page: queryParams.pageNum ,
      pageSize: queryParams.pageSize
    };
    const response = await axios.get('/dev-api/api/student/book/bookList', {
      params: params
    });
    console.log('API返回数据:', response.data);
    console.log( response.data);
    bookList.value = response.data.data.data
  } catch (error) {
    console.error('请求失败:', error);
  }
};

// 获取图书数据
const fetchBooks = async () => {
  try {
    loading.value = true
    const response = await axios.get('/dev-api/api/student/book/bookList',{params:{
      page:1,pageSize:15
      }})
    bookList.value = response.data.data.data
    console.log("图书列表获取数据成功"+bookList.value)
  } catch (error) {
    console.error('获取图书数据失败:', error)
    ElMessage.error('获取图书数据失败')
  } finally {
    loading.value = false
  }
}


// 组件挂载时自动获取数据
onMounted(() => {
  fetchBooks()
})
</script>

<style scoped>

</style>
