<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i>商品订单表
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button
                    type="primary"
                    icon="el-icon-delete"
                    class="handle-del mr10"
                    @click="delAllSelection"
                >批量删除</el-button>
                <el-input v-model="query.name" placeholder="订单号/商品编号/订单状态" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
            </div>
            <el-table
                :data="tableData.slice((query.pageIndex-1)*query.pageSize,query.pageIndex*query.pageSize)"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <!-- <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column> -->
                <el-table-column prop="orderNumber" label="订单编号"></el-table-column>
                <el-table-column prop="userId" label="用户ID"></el-table-column>
                <el-table-column prop="shouperson" label="收货人"></el-table-column>
                <el-table-column prop="shouaddress" label="收货地址"></el-table-column>
                <el-table-column prop="shoutel" label="收货电话"></el-table-column>
                <el-table-column prop="productNumber" label="商品编号"></el-table-column>
                <el-table-column prop="productName" label="商品名称"></el-table-column>
                <el-table-column label="商品图片" align="center" >
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.productPicture"
                            :preview-src-list="[scope.row.productPicture]"
                        ></el-image>
                    </template>
                </el-table-column>
                <el-table-column label="商品单价">
                    <template slot-scope="scope">￥{{scope.row.price}}</template>
                </el-table-column>
                <el-table-column prop="shuliang" label="购买数量"></el-table-column>
                <el-table-column label="订单金额">
                    <template slot-scope="scope">￥{{scope.row.totalPrice}}</template>
                </el-table-column>
                <el-table-column prop="generationTime" label="订单生成时间"></el-table-column>
                <el-table-column prop="status" label="订单状态"></el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                            type="text"
                            icon="el-icon-edit"
                            @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
                        <el-button
                            type="text"
                            icon="el-icon-delete"
                            class="red"
                            @click="handleDelete(scope.$index, scope.row)"
                        >删除</el-button>
                        <el-button v-if="scope.row.status=='待发货'"
                            type="text"
                            class="blue"
                            @click="fahuo(scope.$index, scope.row)"
                        >发货</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="query.pageIndex"
                    :page-size="query.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>

                <!--  -->
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="收货人">
                    <el-input v-model="form.shouperson"></el-input>
                </el-form-item>
                <el-form-item label="收货地址">
                    <el-input v-model="form.shouaddress"></el-input>
                </el-form-item>
                <el-form-item label="收货电话">
                    <el-input v-model="form.shoutel"></el-input>
                </el-form-item>
                <el-form-item label="订单金额">
                    <el-input v-model="form.totalPrice"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { fetchData } from '../../api/index';
export default {
    name: 'orderForm',
    data() {
        return {
            query: {
                name: '',
                pageIndex: 1,
                pageSize: 6
            },
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            pageTotal: 0,
            form: {},
            idx: -1,
            id: -1,
            value: false
        };
    },
    created() {
        this.getData();
    },
    methods: {
        // 获取数据
        getData() {
            var that = this
            this.axios.post('http://localhost:3001/orderlist').then(
                function(res){
                    for(var i = 0;i<res.data.length;i++){
                        if(res.data[i].status=='0'){
                            res.data[i].status='待付款'
                        } else if(res.data[i].status=='1'){
                            res.data[i].status='待发货'
                        } else if(res.data[i].status=='2'){
                            res.data[i].status='待收货'
                        } else if(res.data[i].status=='3'){
                            res.data[i].status='待评价'
                        } else if(res.data[i].status=='4'){
                            res.data[i].status='已完成'
                        }
                    }
                    // res.data.length = 6
                    console.log(res.data)
                    that.tableData = res.data
                    that.pageTotal = res.data.length || 50;
                    // console.log(that.pageTotal)
                },
                function(err){
                    console.log(err)
                }
                )
        },
        // 发货
        fahuo (index,row) {
            // console.log(index)
            // console.log(row.orderNumber)
            this.util.axios.post('http://localhost:3001/fahuo',{orderNumber:row.orderNumber}).then((res)=>{
               console.log('发货成功')
               var a = this.tableData
               for(var i = 0;i<a.length;i++){
                   if(a[i].orderNumber==row.orderNumber){
                       a[i].status = '待收货'
                   }
               }
               this.tableData = a
            },(err)=>{

            })
        },
        // 触发搜索按钮
        handleSearch() {
            console.log(this.query.name)
            var arr = []
            var b = this.tableData
            for(var i = 0;i<b.length;i++){
                   if((b[i].orderNumber.indexOf(this.query.name)!=-1)||(b[i].productNumber.indexOf(this.query.name)!=-1)||(b[i].status.indexOf(this.query.name)!=-1)){
                       arr.push(b[i])
                   }
               }
            console.log(arr)
            this.tableData = arr
            // this.$set(this.query, 'pageIndex', 1);
            // this.getData();
        },
        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
                .then(() => {
                    this.util.axios.post('http://localhost:3001/delorder',{danhao:row.orderNumber}).then((res)=>{
                        console.log('删除成功')
                        this.$message.success('删除成功');
                        this.tableData.splice(index, 1);
                        },(err)=>{
                        })
                })
                .catch(() => {});
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        // delAllSelection() {
        //     const length = this.multipleSelection.length;
        //     let str = '';
        //     this.delList = this.delList.concat(this.multipleSelection);
        //     for (let i = 0; i < length; i++) {
        //         str += this.multipleSelection[i].name + ' ';
        //     }
        //     this.$message.error(`删除了${str}`);
        //     this.multipleSelection = [];
        // },
        delAllSelection() {
               // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
            .then(() => {
            const length = this.multipleSelection.length;
            let str = '';
            var arr = []
            this.delList = this.delList.concat(this.multipleSelection);
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].orderNumber + ' ';
                arr.push(this.multipleSelection[i].orderNumber)
            }
            console.log(arr)
            this.util.axios.post(this.util.lurl+'/manydelding',{id :JSON.stringify(arr)}).then((res)=>{
            var tmp_data = this.tableData
            console.log(tmp_data )
                for (var k = 0; k < arr.length; k++) {
                   for (var j = 0; j < tmp_data.length; j++) {
                       if(tmp_data[j].orderNumber==arr[k]){
                           tmp_data.splice(j, 1);
                       }
                   }
                }
            console.log(tmp_data )
            this.tableData = tmp_data
            this.$message.success(`批量删除成功`);
            // window.location.reload()
            this.multipleSelection = [];
            },(err)=>{
            console.log(err)
            this.$message.error(`删除失败`);
            })
            this.multipleSelection = [];
                })
                .catch(() => {});
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 保存编辑
        saveEdit() {
            this.util.axios.post(this.util.lurl+'/xiuding',{a:this.form}).then((res) =>{
            this.$set(this.tableData, this.idx, this.form);
            this.editVisible = false;
            this.$message.success(`修改成功`);
            // alert('修改成功')
            // window.location.reload()
            },(err) =>{
            this.$message.success(`修改失败`);
            }) 
        },
        // 分页导航
        handlePageChange(val) {
            // console.log("99999" , a , b , c)
            // <el-pagination
            //         background
            //         :hide-on-single-page="value"
            //         layout="total, prev, pager, next"
            //         :current-page="query.pageIndex"
            //         :page-size="query.pageSize"
            //         :total="pageTotal"
            //         @current-change="handlePageChange"
            //     ></el-pagination>


            
            this.$set(this.query, 'pageIndex', val);
            console.log(this.query, 'pageIndex', val)
            this.getData();
        },
        handleRemove(file) {
            console.log(file);
        },
        handlePictureCardPreview(file) {
            this.dialogImageUrl = file.url;
            this.dialogVisible = true;
        },
        handleDownload(file) {
            console.log(file);
        },
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
