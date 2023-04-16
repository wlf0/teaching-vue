<script setup>
import {onMounted, reactive, ref} from "vue"
import { User, Lock } from '@element-plus/icons-vue'
import request from "@/utils/request";
import {ElMessage} from "element-plus";
import { useUserStore } from "@/stores/user";
import router, {setRoutes} from "@/router";

  const loginData = reactive({})
  const rules = reactive({
    username: [
      { required: true, message: '请输入账号', trigger: 'blur' },
    ],
    password: [
      { required: true, message: '请输入密码', trigger: 'blur' },
      { min: 3, max: 20, message: '密码长度在3-20位之间', trigger: 'blur' },
    ],
  })

  const cache = reactive({
    rulesInDB:[]
  })


//拿权限信息
const load = () => {
  request.get('/role', {}).then(res => {
    console.log("res return",res.data)
    cache.rulesInDB = res.data
  })
}



load()  // 调用 load方法拿到后台数据

  const ruleFormRef = ref()
  const login = () => {
    ruleFormRef.value.validate(valid => {
      if (valid) {
        // 发送表单数据给后台

        request.post('/login', loginData).then(res => {
          if (res.code === '200') {
            ElMessage.success('登录成功')
            const userStore = useUserStore()
            userStore.setManagerInfo(res.data)
            router.push('/')
          } else {
            ElMessage.error(res.msg)
          }
        })
      }
    })
  }

const modelshow = () => {
  router.push('/modelshow')
  // window.open("/password")
}

</script>

<template>
  <div style="height: 100vh; overflow: hidden; background-color: aliceblue">
    <div style="width: 400px; margin: 150px auto; background-color: white; border: 1px solid #ddd; padding: 30px; border-radius: 10px">
      <el-form
          ref="ruleFormRef"
          :model="loginData"
          :rules="rules"
          size="large"
          status-icon
      >

        <div style="text-align: center; color: dodgerblue; font-size: 30px; font-weight: bold; margin-bottom: 30px">登 录</div>
        <el-form-item prop="username">
          <el-input v-model="loginData.username" placeholder="请输入账号"  :prefix-icon="User" />
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="loginData.password" show-password placeholder="请输入密码" :prefix-icon="Lock" />
        </el-form-item>
        <el-form-item label="角色">
          <el-radio-group v-model="loginData.role" size="medium">
<!--            <el-radio border label="ADMIN">管理员</el-radio>-->
            <el-radio v-for="item in cache.rulesInDB" border :label="item.flag" >{{item.name}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" style="width: 100%" @click="login">登 录</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="success" style="width: 100%" @click="modelshow">模型概览</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
