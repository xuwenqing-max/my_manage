<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 商品信息表
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
                <el-input v-model="query.name" placeholder="商品编号/商品类别/商品状态" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
                <template>
                    <el-button type="primary" icon="el-icon-lx-add">添加商品</el-button>
                </template>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="productNumber" label="商品编号" align="center"></el-table-column>
                <el-table-column prop="productName" label="商品名称" align="center"></el-table-column>
                <el-table-column prop="category" label="商品类别" align="center"></el-table-column>
                <el-table-column prop="stocks" label="商品库存" align="center"></el-table-column>
                <el-table-column label="商品图片" align="center" >
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.productPicture"
                            :preview-src-list="[scope.row.productPicture]"
                        ></el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="description"  label="商品简介" align="center" :show-overflow-tooltip="true"></el-table-column>
                <el-table-column label="商品单价" align="center">
                    <template slot-scope="scope">￥{{scope.row.price}}</template>
                </el-table-column>
                <el-table-column prop="zhuangtai" label="商品状态" align="center"></el-table-column>
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
                        <el-button v-if="scope.row.zhuangtai=='未上架'"
                            type="text"
                            class="blue"
                            @click="shangjia(scope.$index, scope.row)"
                        >上架</el-button>
                        <el-button v-else
                            type="text"
                            class="blue"
                            @click="xiajia(scope.$index, scope.row)"
                        >下架</el-button>
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
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="商品名称">
                    <el-input v-model="form.productName"></el-input>
                </el-form-item>
                <el-form-item label="商品库存">
                    <el-input v-model="form.stocks"></el-input>
                </el-form-item>
                <el-form-item label="商品图片">
                    <el-input v-model="form.productPicture"></el-input>
                </el-form-item>
                <el-form-item label="商品简介">
                    <el-input v-model="form.description"></el-input>
                </el-form-item>
                <el-form-item label="商品单价">
                    <el-input v-model="form.price"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 添加弹出框 -->
        <el-dialog title="添加" :visible.sync="addVisible" width="30%">
            <el-form ref="add_form" :model="add_form" label-width="70px">
                <el-form-item label="新闻编号">
                    <el-input v-model="form.num"></el-input>
                </el-form-item>
                <el-form-item label="新闻标题">
                    <el-input v-model="form.title"></el-input>
                </el-form-item>
                <el-form-item label="新闻专辑">
                    <el-input v-model="form.album"></el-input>
                </el-form-item>
                <el-form-item label="新闻等级">
                    <el-input v-model="form.grade"></el-input>
                </el-form-item>
                <el-form-item label="新闻地址">
                    <el-input v-model="form.herf"></el-input>
                </el-form-item> 
                <el-form-item label="新闻图片一">
                    <el-input v-model="form.img1"></el-input>
                </el-form-item>
                <el-form-item label="新闻图片二">
                    <el-input v-model="form.img2"></el-input>
                </el-form-item>
                <el-form-item label="新闻图片三">
                    <el-input v-model="form.img3"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveadd">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { fetchData } from '../../api/index';
export default {
    name: 'productInformation',
    data() {
        return {
            query: {
                name: '',
                pageIndex: 1,
                pageSize: 10
            },
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            addVisible: false,
            pageTotal: 0,
            form: {},
            idx: -1,
            id: -1
        };
    },
    mounted() {
        // this.getData();
        var that = this
        console.log('1111')
        this.axios.post('http://localhost:3001/goodsInfoMana',{
            data: {status:4}
        }).then(
            function(res){
                that.tableData = res.data
                that.pageTotal = res.data.length || 50;
                console.log(res.data)
                // console.log(that.pageTotal)
            },
            function(err){
                console.log(err)
            }
            )
    },
    methods: {
        shangjia(index,row) {
            this.util.axios.post('http://localhost:3001/shangjia',{productNumber:row.productNumber}).then((res)=>{
               console.log('上架成功')
               var a = this.tableData
               for(var i = 0;i<a.length;i++){
                   if(a[i].productNumber==row.productNumber){
                       a[i].zhuangtai = '已上架'
                   }
               }
               this.tableData = a
            },(err)=>{
            })
        },
        xiajia(index,row) {
            this.util.axios.post('http://localhost:3001/xiajia',{productNumber:row.productNumber}).then((res)=>{
               console.log('下架成功')
               var a = this.tableData
               for(var i = 0;i<a.length;i++){
                   if(a[i].productNumber==row.productNumber){
                       a[i].zhuangtai = '未上架'
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
                   if((b[i].productNumber.indexOf(this.query.name)!=-1)||(b[i].category.indexOf(this.query.name)!=-1)||(b[i].zhuangtai.indexOf(this.query.name)!=-1)){
                       arr.push(b[i])
                   }
               }
            console.log(arr)
            this.tableData = arr
        },
        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
                .then(() => {
                    this.$message.success('删除成功');
                    this.tableData.splice(index, 1);
                })
                .catch(() => {});
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            this.delList = this.delList.concat(this.multipleSelection);
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].name + ' ';
            }
            this.$message.error(`删除了${str}`);
            this.multipleSelection = [];
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 保存编辑
        saveEdit() {
            this.util.axios.post(this.util.lurl+'/xiushang',{a:this.form}).then((res) =>{
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
            this.$set(this.query, 'pageIndex', val);
            this.getData();
        }
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
