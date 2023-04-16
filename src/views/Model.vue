<script setup lang="ts">
import { nextTick, reactive, ref } from "vue";
import request from "@/utils/request";
import {ElMessage} from "element-plus";
import config from "../../config";
import {useUserStore} from "@/stores/user";
import { genFileId } from 'element-plus'
import type { UploadInstance, UploadProps, UploadRawFile } from 'element-plus'
import { useRouter } from 'vue-router'

const name = ref('')
const pageNum = ref(1)
const pageSize = ref(10)
const total = ref(0)
const upload = ref<UploadInstance>()

const handleExceed: UploadProps['onExceed'] = (files) => {
  upload.value!.clearFiles()
  const file = files[0] as UploadRawFile
  file.uid = genFileId()
  upload.value!.handleStart(file)
}

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
  request.post('/model/del/batch', idArr).then(res => {
    if (res.code === '200') {
      ElMessage.success('操作成功')
      load()  // 刷新表格数据
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const classifyManage = () =>{
  alert("分类管理")
}

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
        url: '/model',
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
// 查看
const router = useRouter()
const handleShow = (raw) => {
  const paramsinfo = {
    obj: raw.path,
    mtl: raw.avatar
  }
  console.log("handleShow",raw)
  console.log("handleShow",raw.path)
  console.log("handleShow",raw.avatar)
  console.log("paramsinfo:",paramsinfo)
  router.push({
    path:`/model3d`,
    query: paramsinfo
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
  request.delete('/model/' + id).then(res => {
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
  window.open(`http://${config.serverUrl}/model/export`)
}

const userStore = useUserStore()
const token = userStore.getBearerToken
const auths =  userStore.getAuths
console.log("auths:",auths)

const handleImportSuccess = () => {
  // 刷新表格
  load()
  ElMessage.success("导入成功")
}
const handleModelImportSuccess = (res) => {
  state.form.path = res.data
  ElMessage.success('模型上传成功')
}


const handleAvatarImportSuccess = (res) => {
  state.form.avatar = res.data
  ElMessage.success("上传缩略图成功")
}
</script>

<template>
  <div>
    <div>
      <el-input v-model="name" placeholder="请输入模型名称" class="w300" />
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

    <div style="margin: 10px 0">
      <el-button type="success" @click="handleAdd" v-if="auths.includes('model.add')">
        <el-icon style="vertical-align: middle">
          <Plus />
        </el-icon>  <span style="vertical-align: middle"> 新增 </span>
      </el-button>
      <el-upload
          v-if="auths.includes('model.import')"
          class="ml5"
          :show-file-list="false"
          style="display: inline-block; position: relative; top: 3px"
          :action='`http://${config.serverUrl}/model/import`'
          :on-success="handleImportSuccess"
          :headers="{ Authorization: token}"
      >
        <el-button type="primary">
          <el-icon style="vertical-align: middle">
            <Bottom />
          </el-icon>  <span style="vertical-align: middle"> 导入 </span>
        </el-button>
      </el-upload>
      <el-button type="primary" @click="exportData" class="ml5" v-if="auths.includes('model.export')">
        <el-icon style="vertical-align: middle">
          <Top />
        </el-icon>  <span style="vertical-align: middle"> 导出 </span>
      </el-button>
      <el-button type="primary" @click="classifyManage" class="ml5 classic" style="margin-left: 5px" v-if="auths.includes('model.classic')">
        <el-icon style="vertical-align: middle">
          <Top />
        </el-icon>  <span style="vertical-align: middle"> 分类管理 </span>
      </el-button>
      <el-popconfirm title="您确定删除吗？" @confirm="confirmDelBatch" v-if="auths.includes('model.deleteBatch')">
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
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="编号"></el-table-column>
        <el-table-column prop="name" label="名称"></el-table-column>
<!--        v-if="false"-->
        <el-table-column prop="avatar" label="mtl路径" ></el-table-column>
          <!--<template #default="scope">
            <el-image :src="scope.row.avatar" v-if="scope.row.avatar" style="width: 50px"></el-image>
          </template>
        </el-table-column>-->
        <el-table-column prop="description" label="简介"></el-table-column>
        <el-table-column prop="path" label="obj路径" ></el-table-column>
        <el-table-column prop="classify" label="分类"></el-table-column>
        <el-table-column prop="publisher" label="发布者"></el-table-column>
        <el-table-column label="操作" width="320">
          <template #default="scope">
            <el-button type="success" @click="handleShow(scope.row) ">查看</el-button><router-view></router-view>
            <el-button type="primary" @click="handleEdit(scope.row)" v-if="auths.includes('model.edit')">编辑</el-button>
            <el-popconfirm title="您确定删除吗？" @confirm="del(scope.row.id)" v-if="auths.includes('model.delete')">
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

    <el-dialog v-model="dialogFormVisible" title="模型信息" width="40%">
      <el-form ref="ruleFormRef" :rules="rules" :model="state.form" label-width="80px" style="padding: 0 20px" status-icon>
        <el-form-item prop="name" label="名称" >
          <el-input v-model="state.form.name" autocomplete="off" />
        </el-form-item>
        <!--<el-form-item prop="avatar" label="缩略图" >
          <el-upload
              class="avatar-uploader"
              :show-file-list="true"
              :action='`http://${config.serverUrl}/file/upload`'
              :on-success="handleAvatarImportSuccess"
              :headers="{ Authorization: token}"
          >
            <img v-if="state.form.avatar" :src="state.form.avatar" class="avatar" />
            <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
          </el-upload>
        </el-form-item>-->
        <el-form-item prop="description" label="简介" >
          <el-input v-model="state.form.description" autocomplete="off" />
        </el-form-item>
        <el-form-item prop="description" label="分类" >
          <el-input v-model="state.form.classify" autocomplete="off" />
        </el-form-item>
        <el-form-item prop="description" label="发布者" >
          <el-input v-model="state.form.publisher" autocomplete="off" />
        </el-form-item>
        <el-form-item prop="path" label="obj上传" >
          <el-upload
              ref="upload"
              class="upload-demo"
              :action='`http://${config.serverUrl}/file/upload`'
              :limit="1"
              :on-exceed="handleExceed"
              :on-success="handleModelImportSuccess"
              :headers="{ Authorization: token}"
              accept=".obj"
          >
            <template #trigger>
              <el-button type="primary">选择模型</el-button>
            </template>

            <template #tip>
              <div class="el-upload__tip">
                仅能选择一个文件, 新的文件将会复制原有文件。
              </div>
            </template>
          </el-upload>

        </el-form-item>
        <el-form-item prop="avatar" label="mtl上传" >
          <el-upload
                  ref="upload"
                  class="upload-demo"
                  :action='`http://${config.serverUrl}/file/upload`'
                  :limit="1"
                  :on-exceed="handleExceed"
                  :on-success="handleAvatarImportSuccess"
                  :headers="{ Authorization: token}"
                  accept=".mtl"
          >
            <template #trigger>
              <el-button type="primary">选择模型</el-button>
            </template>

            <template #tip>
              <div class="el-upload__tip">
                仅能选择一个文件, 新的文件将会复制原有文件。
              </div>
            </template>
          </el-upload>

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
<style>
.avatar-uploader .avatar {
  width: 120px;
  height: 120px;
  display: block;
}

.avatar-uploader .el-upload {
  border: 1px dashed #ccc;
  /*border-radius: 50%;*/
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #ccc;
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 120px;
  height: 120px;
  text-align: center;
}
</style>
