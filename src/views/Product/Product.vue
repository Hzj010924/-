<template>
    <div class="manage">
        <div class="manage-header">
            <el-button type="primary" @click="addUser()" style="margin-right: 35px;">+ 新增</el-button>
            <ProductForm :inline="inline" :formLabel="searchFormLabel" :form="searchFrom">
                <el-button type="primary" @click="config.page = 1; getList()">搜索</el-button>
            </ProductForm>
        </div>
        <CommonTable :tableData="tableData" :tableLabel="tableLabel" :config="config" @changePage="getList()"
            @edit="editUser" @del="delUser">
        </CommonTable>
    </div>
    <el-dialog :title="operateType === 'add' ? '新增商品' : '编辑商品'" v-model="isShow">
        <ProductForm :inline="inline" :formLabel="operateFormLabel" :form="operateForm" ref="form"></ProductForm>
        <template #footer>
            <el-button @click="isShow = false">取 消</el-button>
            <el-button type="primary" @click="confirm">确 定</el-button>
        </template>
    </el-dialog>
</template>

<script setup>
import http from "@/request/index.js"
import { ElMessage, ElMessageBox } from 'element-plus'
import { ref, reactive, onMounted } from 'vue'
import ProductForm from "../ProductForm.vue"
import CommonTable from "../CommonTable.vue"

const inline =true

const tableData = ref([])

const tableLabel = [
    {
        field: "name",
        label: "商品名",
    },
    {
        field: "age",
        label: "商品号",
    },
    {
        field: "sexLabel",
        label: "数量",
    },
    {
        field: "birth",
        label: "出产日期",
        width: 200,
    },
    {
        field: "addr",
        label: "出产地址",
        width: 320,
    },
]
const config = reactive({
    page: 1,
    total: 10, //总记录条数
    limit: 5,  //每页几条
    loading: false,
})
onMounted(() => { getList(); })
async function getList() {
    config.loading = true;
    // 搜索时，页码需要设置为1，才能正确返回数据，因为数据是从第一页开始返回的
    let name = searchFrom.value.keyword;
    console.log("name", name)
    // name ? (config.page = 1) : "";
    let res = await http
        .get("/api/product/getUser", {
            params: {
                page: config.page,
                limit: config.limit,
                name,
            },
        });
    console.log(res);
    tableData.value = res.list.map((item) => {
        item.sexLabel = item.sex 
        return item;
    });
    config.total = res.count;
    config.loading = false;
}

async function delUser(row) {
    console.log(row);
    try {
        let res = await ElMessageBox.confirm("此操作将永久删除该记录, 是否继续?", "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning",
        });
        console.log("***res=", res);
        let res2 = await http.get("/api/product/del?id=" + row.id)
        // let res2 = await http.get("/api/user/del", {
        //     params: {
        //         id: row.id,
        //     },
        // });
        console.log(res2);
        ElMessage({
            type: "success",
            message: "删除成功!",
        });
        getList();

    } catch (error) {
        console.log(error)
        ElMessage({
            type: "info",
            message: "已取消删除",
        });
    }
}

const searchFrom = ref({
    keyword: "",
})
const searchFormLabel = [
    {
        field: "keyword",
        label: "",
    },
]

const operateType = ref("add")
const isShow = ref(false)

const dataDefault = () => {
    return {
        name: "",
        addr: "",
        age: "",
        birth: "",
        sex: "",
    }
}
const operateForm = ref(dataDefault())

const operateFormLabel = [
    {
        field: "name",
        label: "商品名",
    },
    {
        field: "age",
        label: "商品号",
    },
    {
        field: "sex",
        label: "数量",
    },
    {
        field: "birth",
        label: "出产日期",
        type: "date",
    },
    {
        field: "addr",
        label: "出产地址",
    },
]

function addUser() {
    operateType.value = "add";
    // operateForm.value={};
    operateForm.value = dataDefault();
    isShow.value = true;
}

function editUser(row) {
    operateType.value = "edit";
    isShow.value = true;
    // operateForm.value = row;
    operateForm.value = { ...row }
}

/*新增或修改的确认 */
async function confirm() {
    let url = operateType.value === "edit" ? "/api/product/edit" : "/api/product/add"
    let res = await http.post(url, operateForm.value)
    console.log(res);
    isShow.value = false;
    getList();
    // form.resetFields();

}


</script>

<style lang="less" scoped>
.manage {
    padding-bottom: 20px;
    overflow: hidden;

    &-header {
        display: flex;
        // justify-content: space-between;
        align-items: flex-start;
    }
}
</style>