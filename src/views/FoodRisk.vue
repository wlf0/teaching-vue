<script setup>
import { nextTick, reactive, ref } from "vue";
import request from "@/utils/request";
import {ElMessage} from "element-plus";
import config from "../../config";
import {useUserStore} from "@/stores/user";
import router from "../router";

const name = ref('')
const choice = ref('')
const pageNum = ref(1)
const pageSize = ref(5)
const total = ref(0)

const state = reactive({
  tableData: [],
  form: {}
})
const multipleSelection = ref([])

// 批量删除
const handleSelectionChange = (val) => {
  multipleSelection.value = val
}

const confirmDelBatch = () => {
  if (!multipleSelection.value || !multipleSelection.value.length) {
    ElMessage.warning("请选择数据")
    return
  }
  const idArr = multipleSelection.value.map(v => v.id)
  request.post('/foodRisk/del/batch', idArr).then(res => {
    if (res.code === '200') {
      ElMessage.success('操作成功')
      load()  // 刷新表格数据
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const load = () => {
  request.get('/foodRisk/page', {
    params: {
      name: name.value,
      foodType: choice.value,
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

const dialogFormVisible = ref(false)

const rules = reactive({
  name: [
    { required: true, message: '请输入名称', trigger: 'blur' },
  ]
})
const ruleFormRef = ref()

// 新增
const handleAdd = () => {
  dialogFormVisible.value = true
  nextTick(() => {
    ruleFormRef.value.resetFields()
    state.form = {}
  })
}

// 保存
const save = () => {
  ruleFormRef.value.validate(valid => {   // valid就是校验的结果
    if (valid) {
      request.request({
        url: '/foodRisk',
        method: state.form.id ? 'put' : 'post',
        data: state.form
      }).then(res => {
        if (res.code === '200') {
          ElMessage.success('保存成功')
          dialogFormVisible.value = false
          load()  // 刷新表格数据
        } else {
          ElMessage.error(res.msg)
        }
      })
    }
  })
}

// 编辑
const handleEdit = (raw) => {
  dialogFormVisible.value = true
  nextTick(() => {
    ruleFormRef.value.resetFields()
    state.form = JSON.parse(JSON.stringify(raw))
  })
}

// 删除
const del = (id) => {
  request.delete('/foodRisk/' + id).then(res => {
    if (res.code === '200') {
      ElMessage.success('操作成功')
      load()  // 刷新表格数据
    } else {
      ElMessage.error(res.msg)
    }
  })
}

// 导出接口
const exportData = () => {
  window.open(`http://${config.serverUrl}/foodRisk/export`)
}

const userStore = useUserStore()
const token = userStore.getBearerToken
const auths =  userStore.getAuths

const handleImportSuccess = () => {
  // 刷新表格
  load()
  ElMessage.success("导入成功")
}

const handleFileUploadSuccess = (res) => {
  state.form.file = res.data
  ElMessage.success('上传成功')
}
const handleImgUploadSuccess = (res) => {
  state.form.img = res.data
  ElMessage.success('上传成功')
}

const dateFormat = (raw) => {
  if(!raw || !raw.createTime){
    return '';
  }
  const data = new Date(raw.createTime);
  // 封装年份时间
  const y = data.getFullYear();
  // 封装月份时间
  const m = data.getMonth();
  // 封装日期时间
  const d = data.getDate();
  // 转换详细时分
  const hours =
          data.getHours() < 10 ? "0" + data.getHours() : data.getHours();
  const minutes =
          data.getMinutes() < 10 ? "0" + data.getMinutes() : data.getMinutes();
  // 输出转换后时间节点
  return `${y}年${m+1}月${d}日 ${hours}:${minutes}`;
}

const handleColumnStyle = (data) =>{
    console.log("handleColumnStyle:data",data)
  if(data > 1){
    return {'color':'red'}
  }else if(data > 0.5){
    return {'color':'#000'}
  }else if(data > 0){
    return {'color':'#13ce66'}
  }
}
const testEcharts = () =>{
  console.log("testEcharts")
  router.push({
    path:`/testEcharts`})
}
</script>

<template>
  <div>
    <div>
      <el-select v-model="choice" clearable placeholder="可选择分类查询">
        <el-option label="配方奶粉" value="配方奶粉"></el-option>
        <el-option label="米粉" value="米粉"></el-option>
        <el-option label="面条" value="面条"></el-option>
        <el-option label="饼干" value="饼干"></el-option>
        <el-option label="馒头" value="馒头"></el-option>
        <el-option label="大米" value="大米"></el-option>
        <el-option label="小米" value="小米"></el-option>
        <el-option label="果泥" value="果泥"></el-option>
        <el-option label="果酱" value="果酱"></el-option>
      </el-select>
      <el-input v-model="name" placeholder="请输入食品标识" class="w300" />
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

      <el-button type="warning" class="ml5" @click="testEcharts()">
        <el-icon style="vertical-align: middle">
          <RefreshLeft />
        </el-icon>  <span style="vertical-align: middle"> 测试 </span>
      </el-button>

    </div>

    <div style="margin: 10px 0">
      <el-button type="success" @click="handleAdd" v-if="auths.includes('foodRisk.add')">
        <el-icon style="vertical-align: middle">
          <Plus />
        </el-icon>  <span style="vertical-align: middle"> 新增 </span>
      </el-button>
      <el-upload
          v-if="auths.includes('foodRisk.import')"
          class="ml5"
          :show-file-list="false"
          style="display: inline-block; position: relative; top: 3px"
          :action='`http://${config.serverUrl}/foodRisk/import`'
          :on-success="handleImportSuccess"
          :headers="{ Authorization: token}"
      >
        <el-button type="primary">
          <el-icon style="vertical-align: middle">
            <Bottom />
          </el-icon>  <span style="vertical-align: middle"> 导入 </span>
        </el-button>
      </el-upload>
      <el-button type="primary" @click="exportData" class="ml5" v-if="auths.includes('foodRisk.export')">
        <el-icon style="vertical-align: middle">
          <Top />
        </el-icon>  <span style="vertical-align: middle"> 导出 </span>
      </el-button>
      <el-popconfirm title="您确定删除吗？" @confirm="confirmDelBatch" v-if="auths.includes('foodRisk.deleteBatch')">
        <template #reference>
          <el-button type="danger" style="margin-left: 5px">
            <el-icon style="vertical-align: middle">
              <Remove />
            </el-icon>  <span style="vertical-align: middle"> 批量删除 </span>
          </el-button>
        </template>
      </el-popconfirm>
    </div>

    <div style="margin: 10px 0">
      <el-table :data="state.tableData" stripe border  @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="95" />
      <el-table-column prop="id"  width="95"  label="唯一标识"></el-table-column>
      <el-table-column prop="foodLabel" width="95" label="食物标志"></el-table-column>
      <el-table-column prop="foodType" width="95" label="食物类型"></el-table-column>
      <el-table-column prop="quotaOne"  label="指标1">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaOne)">
            {{scope.row.quotaOne}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaTwo" label="指标2">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaTwo)">
            {{scope.row.quotaTwo}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaThree" label="指标3">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaThree)">
            {{scope.row.quotaThree}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaFour" label="指标4">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaFour)">
            {{scope.row.quotaFour}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaFive" label="指标5">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaFive)">
            {{scope.row.quotaFive}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaSix" label="指标6">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaSix)">
            {{scope.row.quotaSix}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaSeven" label="指标7">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaSeven)">
            {{scope.row.quotaSeven}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaEight" label="指标8">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaEight)">
            {{scope.row.quotaEight}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="quotaNine" label="指标9">
        <template #default="scope">
          <b :style="handleColumnStyle(scope.row.quotaNine)">
            {{scope.row.quotaNine}}
          </b>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" width="170" :formatter="dateFormat"  label="抽样时间"></el-table-column>
      <el-table-column prop="location" label="所在地"></el-table-column>
      <el-table-column prop="creator" label="调查员"></el-table-column>

        <el-table-column label="操作" width="180">
          <template #default="scope">
            <el-button type="primary" @click="handleEdit(scope.row)" v-if="auths.includes('foodRisk.edit')">编辑</el-button>
            <el-popconfirm title="您确定删除吗？" @confirm="del(scope.row.id)" v-if="auths.includes('foodRisk.delete')">
              <template #reference>
                <el-button type="danger">删除</el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div style="margin: 10px 0">
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

    <el-dialog v-model="dialogFormVisible" title="动态信息" width="40%">
      <el-form ref="ruleFormRef" :rules="rules" :model="state.form" label-width="80px" style="padding: 0 20px" status-icon>
        <el-form-item prop="foodLabel" label="食物标志">
          <el-input v-model="state.form.foodLabel" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="foodType" label="食物类型">
          <el-input v-model="state.form.foodType" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaOne" label="指标1">
          <el-input v-model="state.form.quotaOne" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaTwo" label="指标2">
          <el-input v-model="state.form.quotaTwo" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaThree" label="指标3">
          <el-input v-model="state.form.quotaThree" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaFour" label="指标4">
          <el-input v-model="state.form.quotaFour" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaFive" label="指标5">
          <el-input v-model="state.form.quotaFive" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaSix" label="指标6">
          <el-input v-model="state.form.quotaSix" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaSeven" label="指标7">
          <el-input v-model="state.form.quotaSeven" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaEight" label="指标8">
          <el-input v-model="state.form.quotaEight" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="quotaNine" label="指标9">
          <el-input v-model="state.form.quotaNine" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="location" label="所在地">
          <el-input v-model="state.form.location" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item prop="creator" label="调查员">
          <el-input v-model="state.form.creator" autocomplete="off"></el-input>
        </el-form-item>

      </el-form>
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button type="primary" @click="save">
          保存
        </el-button>
      </span>
      </template>
    </el-dialog>
  </div>
</template>