<template>
    <el-card class="box-card">
        <!-- 卡片顶部添加品牌按钮 -->
        <el-button type="primary" size="default" icon="Plus" @click="addTrademark">添加品牌</el-button>
        <!-- 表格组件：用于展示已有的平台数据 -->
        <el-table style="margin: 10px 0px" border :data="trademarkArr">
            <el-table-column label="序号" width="80px" align="center" type="index"/>
            <!-- table-column：默认展示数据用div -->
            <!-- <el-table-column label="品牌名称" prop="tmName" style="color: red"/> -->
            <!-- 当然也可以用插槽 -->
            <el-table-column label="品牌名称">
                <template #="{row,$index}">
                    <pre style="color:brown">{{ row.tmName }}</pre>
                </template>
            </el-table-column>
            <el-table-column label="品牌LOGO">
                <template #="{ row,$index }">
                    <img :src="row.logUrl" alt="没有图片" style="width: 100px;height: 100px;">
                </template>
            </el-table-column>
            <el-table-column label="品牌操作">
                <template #="{row,$index}">
                    <el-button type="primary" size="small" icon="Edit" @click="$event=>updateTrademark(row)"></el-button>
                    <el-button type="danger" size="small" icon="Delete"></el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页器组件
        v-model:current-page    设置分页器当前页码
        v-model:page-size       设置每一个展示数据条数
        page-sizes              用于设置下拉菜单数据
        background              设置分页器按钮的背景颜色
        layout                  可以设置分页器下的子组件的布局调整
         -->
        <el-pagination
            @size-change="sizeChange"
            @current-change="getHasTrademark"
            v-model:current-page="pageNo"
            v-model:page-size="limit"
            :page-sizes="[3, 5, 7, 9]"
            :size="size"
            :pager-count="9"
            :background="true"
            layout="prev, pager, next, jumper ,->, total, sizes"
            :total="total"
        />
    </el-card>
    <!-- 对话框组件：在添加或修改已有品牌的时候使用的结构 -->
     <!--
        v-model: 该属性用于用户控制对话框的显示或隐藏，true显示，false隐藏 
        title：设置对话框左上角标题    
    -->
    <el-dialog v-model="dialogFormVisible" :title="trademarkParams.id?'修改品牌':'添加品牌'" width="500">
        <el-form style="width: 80%;">
            <el-form-item label="品牌名称" label-width="80px">
                <el-input placeholder="请您输入品牌名称" v-model="trademarkParams.tmName"></el-input>
            </el-form-item>
            <el-form-item label="品牌LOGO" label-width="80px">
                <!-- upload组件属性： action图片上传路径，要加/api，否则代理服务器不发送这次post请求 -->
                <el-upload
                    class="avatar-uploader"
                    action="/api/admin/product/Trademark/imgupload"
                    :show-file-list="false"
                    :on-success="handleAvatarSuccess"
                    :before-upload="beforeAvatarUpload"
                >
                    <img v-if="trademarkParams.logUrl" :src="trademarkParams.logUrl" class="avatar" />
                    <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
                </el-upload>
            </el-form-item>
        </el-form>
        <template #footer>
            <el-button type="primary">主要按钮</el-button>
            <el-button type="primary"  @click="cancle">取消</el-button>
            <el-button type="primary"  @click="confirm">确定</el-button>
        </template>
    </el-dialog> 
</template>

<script setup lang="ts">
import type { UploadProps } from 'element-plus';
import { ElMessage } from 'element-plus';
import 'element-plus/theme-chalk/el-message.css';
// 引入组合式API函数ref
import { ref,onMounted,reactive } from 'vue';
import { reqHasTrademark,reqAddOrUpdateTrademark } from '../../../api/product/trademark/';
import type { Records,TradeMarkResponseData,TradeMark } from '../../../api/product/trademark/type'
// 当前页码,默认第1页
let pageNo = ref<number>(1);
// 每一页展示多少条数据
let limit = ref<number>(3);
// 存储已有品牌数据总数
let total = ref<number>(0);
// 存储已有品牌的数据
let trademarkArr = ref<Records>([]);
// 控制对话框显示与隐藏
let dialogFormVisible = ref<boolean>(false);
// 定义收集新增品牌数据
let trademarkParams = reactive<TradeMark>({
    tmName: '',
    logUrl: ''
});
// 获取已有品牌的接口封装为一个函数：在任何情况下想获取数据，调用此函数即可
const getHasTrademark = async (pager = 1) => {
    // 当前页码
    pageNo.value = pager;
    let result:TradeMarkResponseData = await reqHasTrademark(pageNo.value,limit.value);
    if(result.code == 200) {
        // 给已有品牌的总数重新赋值
        total.value = result.data.total;
        trademarkArr.value = result.data.records;
    }
}
// 组件挂载完毕钩子 --- 发一次请求，获取第一页、一页10条数据
onMounted(() => {
    getHasTrademark();
})

// 当前分页器页码发生变化的时候会触发
// 对应当前页码发生变化自定义事件，组件pagenation父组件回传了数据(当前页码
// const changePageNo = (a) => {
//     console.log(a)
//     // 当前页面发生变化的时候，再次发请求获取对应已有品牌的数据并展示
//     getHasTrademark();
// }

// 当下拉菜单发生变化的时候触发此方法
// 这个自定义事件，分页器组件会将下拉菜单选中的数值返回
const sizeChange = () => {
    // 当前每一页的数据量发生变化时，让当前页码归1
    getHasTrademark();
}

// 添加品牌按钮的回调
const addTrademark = () => {
    // 对话框显示
    dialogFormVisible.value = true;
    // 清空对话框的上一次的数据
    trademarkParams.id = 0;
    trademarkParams.tmName = '';
    trademarkParams.logUrl = '';
}

// 修改已有品牌按钮的回调
// row：即为当前已有的品牌
const updateTrademark = (row:TradeMark) => {
    // 对话框显示
    dialogFormVisible.value = true;
    // ES6语法：合并对象，可以用于代替其下的三行，效果是一样的
    Object.assign(trademarkParams,row);
    // trademarkParams.id = row.id;
    // // 修改之前先展示要修改的品牌的信息 
    // trademarkParams.tmName = row.tmName;
    // trademarkParams.logUrl = row.logUrl;
}

// 对话框底部取消按钮
const cancle = () => {
    dialogFormVisible.value = false;
}
// 对话框底部确定按钮
const confirm = async () => {
    let result:any = await reqAddOrUpdateTrademark(trademarkParams);
    // 添加|修改已有品牌
    if(result.code == 200) {
        // 关闭对话框
        dialogFormVisible.value =false;
        // 弹出提示信息
        ElMessage({
            type:"success",
            message: trademarkParams.id?'修改品牌成功':'添加品牌成功'
        });
        // 再次发请求获取已有全部品牌的数据
        getHasTrademark(trademarkParams.id?pageNo.value:1);
        // 添加品牌失败
    } else {
        dialogFormVisible.value =false;
        ElMessage({
            type:"error",
            message: trademarkParams.id?'修改品牌失败':"添加品牌失败"
        });
        // 关闭对话框
        dialogFormVisible.value =false;
    }
}
// 上传图片组件：上传图片之前触发的钩子
const beforeAvatarUpload: UploadProps['beforeUpload'] = (rawFile) => {
  // 钩子是在图片上传成功之前触发，上传文件之前进行一些约束，比如文件类型、大小等
 //  举例：要求文件类型为jpg、png、gif，文件大小不超过 4M   
    if (rawFile.type !== 'image/jpeg') {
       ElMessage.error('Avatar picture must be JPG format!')
       return false
     } else if (rawFile.size / 1024 / 1024 > 2) {
       ElMessage.error('Avatar picture size can not exceed 2MB!')
       return false
     }
     return true
}
// 图片上传成功的钩子
const handleAvatarSuccess: UploadProps['onSuccess'] = (response,uploadFile) => {
    // response：当前这次上传图片服务器返回的响应数据
    // 收集上传图片的地址，添加一个新品牌的时候带给服务器
    trademarkParams.logUrl = response.data;
}
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
.dialog-cancle{
    background-color: blue;
}
</style>