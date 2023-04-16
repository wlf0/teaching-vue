<template>
    <div id="main">
        <el-form  :rules="rules" :model="form" ref="form" label-width="110px">
            <el-form-item label="食品类型" prop="foodType">
                <el-select class="selec" v-model="form.foodType"  clearable placeholder="可选择分类查询">
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
            </el-form-item>

            <el-form-item label="指标" prop="quota">
                <el-select class="selec" v-model="form.quota" prop="quota" clearable placeholder="可选择指标查询">
                    <el-option value="quotaOne" label="指标1"></el-option>
                    <el-option value="quotaTwo" label="指标2"></el-option>
                    <el-option value="quotaThree" label="指标3"></el-option>
                    <el-option value="quotaFour" label="指标4"></el-option>
                    <el-option value="quotaFive" label="指标5"></el-option>
                    <el-option value="quotaSix" label="指标6"></el-option>
                    <el-option value="quotaSeven" label="指标7"></el-option>
                    <el-option value="quotaEight" label="指标8"></el-option>
                    <el-option value="quotaNine" label="指标9"></el-option>
                </el-select>
            </el-form-item>

            <el-form-item label="样品开始日期" required>
                <el-col :span="5">
                    <el-form-item prop="startDate">
                        <el-date-picker type="date" value-format="YYYY-MM-DD" placeholder="选择日期" v-model="form.startDate" style="width: 100%;"></el-date-picker>
                    </el-form-item>
                </el-col>
            </el-form-item>
            <el-form-item label="样品结束日期" required>
                <el-col :span="5">
                    <el-form-item prop="endDate">
                        <el-date-picker type="date" value-format="YYYY-MM-DD" placeholder="选择日期" v-model="form.endDate" style="width: 100%;"></el-date-picker>
                    </el-form-item>
                </el-col>
            </el-form-item>

            <el-form-item>
                <el-button type="primary" class="ml5" @click="load('form')">
                    <el-icon style="vertical-align: middle">
                        <Search />
                    </el-icon>  <span style="vertical-align: middle"> 搜索 </span>
                </el-button>
            </el-form-item>

        </el-form>


        <div id="myChart" :style="{ width: '700px', height: '400px' }"></div>
    </div>
</template>

<script>
    import { useRouter } from 'vue-router'
    import {useUserStore} from "@/stores/user";
    import request from "@/utils/request";
    import {ElMessage} from "element-plus";
    import router from "../router";
    import echarts from "@/echarts";
    import {ref} from "vue";
    export default {
        name: "testEcharts",
        data() {
            return {
                dateArr:[],
                valueArr:[],
                form:{},
                option: {},
                rules: {
                    foodType: [
                        { required: true, message: '请选择活动区域', trigger: 'change' }
                    ],
                    quota: [
                        { required: true, message: '请选择活动区域', trigger: 'change' }
                    ],
                    startDate: [
                        { required: true, message: '请选择日期', trigger: 'change' }
                    ],
                    endDate: [
                        { required: true, message: '请选择日期', trigger: 'change' }
                    ],
                    /* date2: [
                        { type: 'date', required: true, message: '请选择时间', trigger: 'change' }
                    ],
                    type: [
                        { type: 'array', required: true, message: '请至少选择一个活动性质', trigger: 'change' }
                    ],
                    resource: [
                        { required: true, message: '请选择活动资源', trigger: 'change' }
                    ],
                    desc: [
                        { required: true, message: '请填写活动形式', trigger: 'blur' }
                    ]*/
                },
            };
        },
        mounted() {

        },
        methods: {
            updateEcharts(){
                console.log("xAxis",this.dateArr)
                console.log("data",this.valueArr)
                //this.$root => app
                let myChart = echarts.init(document.getElementById("myChart"));
                // 绘制图表
                myChart.setOption({
                    title: { text: "总用户量" },
                    tooltip: {},
                    xAxis: {
                        data: this.dateArr,
                    },
                    yAxis: {},
                    series: [
                        {
                            name: "用户量",
                            type: "line",
                            data: this.valueArr,
                        },
                    ],
                });
            },
            load(formData){
                console.log("load:",this.form)
                this.$refs[formData].validate((valid) => {
                    console.log("valid",valid)
                    if(valid){
                        console.log("submit")
                        //submit
                        request.get('/foodRisk', {
                            params: {
                                foodType: this.form.foodType,
                                quota: this.form.quota,
                                startDate: this.form.startDate,
                                endDate: this.form.endDate,
                            }
                        }).then(res => {
                            if (res.code === '200') {
                                ElMessage.success('操作成功')
                                //更新图标
                                this.valueArr = res.data.map((x)=>{
                                    return x.value
                                })
                                this.dateArr = res.data.map((x) =>{
                                    return x.createDateStr
                                })
                                this.updateEcharts();
                            } else {
                                ElMessage.error(res.msg)
                            }
                        })
                    }else {
                        return false;
                    }
                })
            },
            initModel() {
                console.log("testEcharts.vue")
            },
        },
    };
</script>
<style>
    .ml5{
        margin-top: 4px;
    }
    .selec{
        margin: 4px;
    }
    .timeClass{
        margin-left: 4px;
    }
</style>