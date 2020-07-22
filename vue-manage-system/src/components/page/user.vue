<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i>用户信息表
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
                <el-input v-model="query.name" placeholder="用户账号/违规次数" class="handle-input mr10"></el-input>
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
                <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="userId" label="用户账号" width="110"></el-table-column>
                <el-table-column label="头像" align="center">
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.avatar"
                            :preview-src-list="[scope.row.avatar]"
                        ></el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="telnumber" label="手机号"></el-table-column>
                <el-table-column prop="nickname" label="昵称"></el-table-column>
                <el-table-column prop="password" label="密码"></el-table-column>
                <el-table-column prop="mailbox" label="邮箱"></el-table-column>
                <el-table-column prop="weigui" label="违规次数"></el-table-column>
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
                        <el-button v-if="scope.row.zhuang=='解冻'"
                            type="text"
                            class="red"
                            @click="dong(scope.$index, scope.row)"
                        >冻结</el-button>
                        <el-button v-else
                            type="text"
                            class="red"
                            @click="jie(scope.$index, scope.row)"
                        >解冻</el-button>
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
                <el-form-item label="头像">
                    <el-input v-model="form.avatar"></el-input>
                </el-form-item>
                <el-form-item label="昵称">
                    <el-input v-model="form.nickname"></el-input>
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
    name: 'user',
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
            pageTotal: 0,
            form: {},
            idx: -1,
            id: -1
        };
    },
    created() {
        this.getData();
    },
    methods: {
        // 获取 easy-mock 的模拟数据
        getData() {
            var that = this
            console.log('1111')
            this.axios.post('http://localhost:3001/usersdata').then(
                function(res){
                    // that.$set(that.my_data , "tableData" , res.data.reverse())
                    that.pageTotal = res.data.length || 10;
                    console.log(res.data)
                    var a = res.data
                    for(var i=0;i<a.length;i++){
                        a[i].id=i+1
                    }
                    that.tableData=a 
                    // console.log(that.pageTotal)
                },
                function(err){
                    console.log(err)
                }
                )
        },
        dong(index,row){
            this.util.axios.post('http://localhost:3001/dong',{userId:row.userId,weigui:row.weigui}).then((res)=>{
               console.log('冻结成功')
               var a = this.tableData
               for(var i=0;i<a.length;i++){
                   if(a[i].userId==row.userId){
                       a[i].zhuang='冻结'
                       a[i].weigui= a[i].weigui-0+1
                   }
               }
               this.tableData=a
               this.$message.success(`冻结成功`);
            },(err)=>{

            })
        },
        jie(index,row){
            this.util.axios.post('http://localhost:3001/jie',{userId:row.userId}).then((res)=>{
               console.log('解冻成功')
               var a = this.tableData
               for(var i=0;i<a.length;i++){
                   if(a[i].userId==row.userId){
                       a[i].zhuang='解冻'
                   }
               }
               this.tableData=a
               this.$message.success(`解冻成功`);
            },(err)=>{

            })
        },
        handleSearch() {
            console.log(this.query.name)
            var arr = []
            var b = this.tableData
            for(var i = 0;i<b.length;i++){
                   if((b[i].userId.indexOf(this.query.name)!=-1)||(b[i].weigui.indexOf(this.query.name)!=-1)){
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
                    this.util.axios.post('http://localhost:3001/deluser',{userId:row.userId}).then((res)=>{
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
            this.util.axios.post(this.util.lurl+'/xiuuser',{a:this.form}).then((res) =>{
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
