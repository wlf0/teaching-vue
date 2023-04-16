<script setup>
import { nextTick, reactive, ref } from "vue";
import request from "@/utils/request";
import {useRouter} from "vue-router/dist/vue-router";

const name = ref('')
const pageNum = ref(1)
const pageSize = ref(5)
const total = ref(0)

const state = reactive({
  tableData: [],
  form: {}
})

const load = () => {
  request.get('/model/page', {
    params: {
      name: name.value,
      pageNum: pageNum.value,
      pageSize: pageSize.value
    }
  }).then(res => {
    state.tableData = res.data.records
    total.value = res.data.total
  })
}
load()  // 调用 load方法拿到后台数据

const reset = () => {
  name.value = ''
  load()
}

// 查看
const router = useRouter()
const handleShow = (raw) => {
  router.push({path:'/model3d'})
}

</script>

<template>
  <div>
    <div style="margin: 10px 100px">
      <el-input v-model="name" placeholder="请输入名称" class="w300" />
      <el-button type="primary" class="ml5" @click="load">
        <el-icon style="vertical-align: middle">
          <Search />
        </el-icon>  <span style="vertical-align: middle"> 搜索 </span>
      </el-button>
      <el-button type="warning" class="ml5" @click="reset">
        <el-icon style="vertical-align: middle">
          <RefreshLeft />
        </el-icon>  <span style="vertical-align: middle"> 重置 </span>
      </el-button>
    </div>

    <div style="margin: 10px 100px">
      <el-table :data="state.tableData" stripe border  @selection-change="handleSelectionChange">
        <el-table-column prop="id" label="编号"></el-table-column>
        <el-table-column prop="name" label="名称"></el-table-column>
        <el-table-column prop="description" label="简介"></el-table-column>
        <el-table-column prop="avatar" label="缩略图">
          <template #default="scope">
            <el-image :src="scope.row.avatar" v-if="scope.row.avatar" style="width: 50px"></el-image>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="320">
          <template #default="scope">
            <el-button type="success" @click="handleShow(scope.row) ">查看</el-button><router-view></router-view>
                </template>
        </el-table-column>
      </el-table>
    </div>


    <div style="margin: 10px 100px">
      <el-pagination
          @current-change="load"
          @size-change="load"
          v-model:current-page="pageNum"
          v-model:page-size="pageSize"
          background
          :page-sizes="[2, 5, 10, 20]"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
      />
    </div>
  </div>
</template>
