<template>
    <div class="categories">
        <el-row>
           <el-col :span='24'>
               <el-breadcrumb separator="/">
                <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
                <el-breadcrumb-item>商品管理</el-breadcrumb-item>
                <el-breadcrumb-item ><span class="currentpage">商品分类</span></el-breadcrumb-item>               
               </el-breadcrumb>
           </el-col>
       </el-row>
       <el-row>
            <el-button type="success" plain @click="addCategories">添加分类</el-button>
       </el-row>
       <tree-grid 
        v-loading='loading'
        :treeStructure="true"
        :columns="columns"
        :data-source="dataSource"
        @deleteCate="deleteCategory"
        @editCate="editCategory">
        </tree-grid>
        <div class="page">
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="1"
            :page-sizes="[10, 20, 30, 40]"
            :page-size="pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total='total'>
            </el-pagination>
        </div>
        <!-- 添加分类对话框 -->
        <el-dialog title="添加分类" :visible.sync="addDialogFormVisible" class="addForm">
            <el-form :model='addForm' :rules='rules' ref='addFormName' v-loading='loading'>
                <el-form-item label="分类名称" prop="cat_addname" >
                    <el-input v-model="addForm.cat_addname" auto-complete="off" class="input"></el-input>
                </el-form-item>
                <el-form-item label="父级名称" >
                    <el-cascader                       
                        :options="options"
                        v-model="selectedOptions"
                        change-on-select
                        :props="props"
                        @change="handleChange">
                    </el-cascader>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="addDialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCategorySubmit">确 定</el-button>
            </div>
        </el-dialog>
        <!-- 编辑分类对话框 -->
        <el-dialog title="修改分类" :visible.sync="editDialogFormVisible">
            <el-form :model="editForm" :rules='rules' ref="editFormDom">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="editForm.cat_name" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="editDialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="editCategorySubmit">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
import TreeGrid from '@/components/TreeGrid/TreeGrid'
import {getCategories, addCategories, deleteCategoried, getCategoriesId, editCategories} from '@/api'
export default {
    data () {
        return {
            dataSource: [],
            loading: true,
            columns: [{
                text: '分类名称',
                dataIndex: 'cat_name',
                width: ''
                }, {
                text: '是否有效',
                dataIndex: 'cat_deleted',
                width: ''
                }, {
                text: '排序',
                dataIndex: 'cat_level',
                width: ''
            }],
            pagesize: 10,
            pagenum: 1,
            total: 1,
            addDialogFormVisible: false,
            options: [], // 级联选择器的数据源
            selectedOptions: [], // 级联选择器选中后的数据
            addForm: { 
               cat_level: 0,
               cat_name: '',
               cat_pid: 0
            },
            props: { // 表示配置级联选择器展示的字段
                value: 'cat_id',
                label: 'cat_name'
            },
            rules: {
                cat_addname: [
                    { required: true, message: '请输入分类名称', trigger: 'blur' }
                ],
                cat_name: [
                    { required: true, message: '请输入分类名称1', trigger: 'blur' }
                ]
            },         
            editDialogFormVisible: false,
            editForm: {
                cat_name: '',
            }
        }
    },
    components: {
        TreeGrid
    },
    created () {
        this.initList()
    },
    methods: {
        // TreeGrid 删除商品分类
        deleteCategory (cid) {
            console.log(cid)
            this.$confirm('是否删除该分类？', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                deleteCategoried({id: cid}).then(res => {
                    if (res.meta.status === 200) {
                    this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                    this.initList()
                }
                
             })    
               
           }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
           });
            
        },
        // TreeGrid 修改商品分类
        editCategory (cid) {
            this.editDialogFormVisible = true
            getCategoriesId({id: cid}).then(res => {
                if (res.data.meta.status === 200) {
                    this.editForm.cat_name = res.data.data.cat_name
                    this.editForm.id = res.data.data.cat_id
                }                
            })
        },
        // 修改商品分类提交
        editCategorySubmit () {
            this.$refs.editFormDom.validate(valide => {
                if (valide) {
                    editCategories(this.editForm).then(res => {
                        if (res.meta.status === 200) {                       
                        this.$message({
                            type: 'success',
                            message: res.meta.msg
                        })
                        this.editDialogFormVisible = false
                        this.initList()
                        this.editForm.cat_name = ''
                      } else {
                          this.$message({
                              type: 'error',
                              message: res.meta.msg
                          })
                       }
                    })
                }
            })
        },
        // 分页
        handleSizeChange(val) {
            this.pagesize = val
            this.initList()
        },
        handleCurrentChange(val) {
            this.pagenum = val
            this.initList()
        },
        // 获取产品分类列表
        initList () {
            getCategories({type: '3',pagesize: this.pagesize, pagenum: this.pagenum}).then(res => {
               if (res.meta.status === 200) {
                   this.total = res.data.total
                   this.dataSource = res.data.result
                   this.loading = false
               }
            })
        },
        // 点击添加商品
        addCategories () {
            this.addDialogFormVisible =  true
            getCategories({type: 2}).then(res => {
                if (res.meta.status === 200) {
                    this.options = res.data
                    this.loading = false
                }
            })
        },
        // 确定添加商品分类
        addCategorySubmit () {
            this.$refs.addFormName.validate(valide => {
                if (valide) {
                   if (this.selectedOptions.length === 0) {
                        this.addForm.cat_pid = 0
                        this.addForm.cat_level = 0
                    } else if (this.selectedOptions.length === 1) {
                        this.addForm.cat_pid = this.selectedOptions[this.selectedOptions.length - 1]
                        this.addForm.cat_level = 1
                    } else {
                        this.addForm.cat_pid = this.selectedOptions[this.selectedOptions.length - 1]
                        this.addForm.cat_level = 2
                    }
                    addCategories(this.addForm).then(res => {
                        if (res.meta.status === 201) {
                        this.addDialogFormVisible = false
                        this.initList()
                        this.$message({
                            type: 'success',
                            message: res.meta.msg
                        })
                        this.selectedOptions.length = 0
                        this.addForm.cat_addname = ''
                      } else {
                          this.$message({
                              type: 'error',
                              message: res.meta.msg
                          })
                      }
                    })
                }
            })
        },
        handleChange (value) {
        console.log(value);
      }
    }
}
</script>
<style lang="scss" scoped>
.addForm {

 .input {
     width: 60%;
 }
}
</style>
