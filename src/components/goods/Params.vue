<template>
<div>
    <!--面包屑-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <!-- 警告区域 -->
        <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" show-icon closable>
        </el-alert>
        <!-- 选择商品分类区域 -->
        <el-row class="cat_opt">
            <el-col>
                <span>选择商品分类：</span>
                <!-- 级联选择框 -->
                <el-cascader v-model="selectedCateKeys" :options="cateList" :props="cateProps" @change="handleChange"></el-cascader>
            </el-col>
        </el-row>
        <!-- tab页签区域 -->
        <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <!-- 添加动态参数面板 -->
            <el-tab-pane label="用户管理" name="many">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>
                <el-table :data="manyTableData" border stripe>
                    <!-- 展开行 -->
                    <el-table-column type="expand">
                        <!-- 循环渲染tag标签 -->
                        <template v-slot="scope">
                            <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClosed(i, scope.row)">{{item}}</el-tag>
                            <!-- 输入文本框 -->
                            <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                            </el-input>
                            <!-- 添加tag的按钮 -->
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template v-slot="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row.attr_id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
            <!-- 添加静态属性的面板 -->
            <el-tab-pane label="配置管理" name="only">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
                <el-table :data="onlyTableData" border stripe>
                    <!-- 展开行 -->
                    <el-table-column type="expand">
                        <!-- 循环渲染tag标签 -->
                        <template v-slot="scope">
                            <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClosed(i, scope.row)">{{item}}</el-tag>
                            <!-- 输入文本框 -->
                            <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                            </el-input>
                            <!-- 添加tag的按钮 -->
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template v-slot="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row.attr_id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
        </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
        <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
    </el-dialog>
    <!-- 修改参数的对话框 -->
    <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form ref="editFormRef" :rules="editFormRules" :model="editForm" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="editForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    data() {
        return {
            // 商品分类列表
            cateList: [],
            // 级联选择框的配置对象
            cateProps: {
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            // 级联选择框双向绑定到的数组
            selectedCateKeys: [],
            // 被激活的页签的名称
            activeName: 'many',
            // 动态参数数据
            manyTableData: [],
            // 静态属性数据
            onlyTableData: [],
            // 控制添加对话框的显示与隐藏
            addDialogVisible: false,
            // 添加参数的表单数据对象
            addForm: {
                attr_name: ''
            },
            // 添加参数的验证规则
            addFormRules: {
                attr_name: [{
                    required: true,
                    message: '请输入参数名称',
                    trigger: 'blur'
                }]
            },
            // 控制修改对话框的显示与隐藏
            editDialogVisible: false,
            // 修改参数的表单数据对象 
            editForm: {
                attr_name: ''
            },
            // 修改参数的验证规则
            editFormRules: {
                attr_name: [{
                    required: true,
                    message: '请输入参数名称',
                    trigger: 'blur'
                }]
            }
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        // 获取所有的商品分类列表
        async getCateList() {
            const {
                data: res
            } = await this.$http.get('categories')
            if (res.meta.status !== 200) {
                this.$message.error('获取商品分类列表失败！')
            }
            this.cateList = res.data
        },
        // 级联选择框选中项变化时触发的函数
        handleChange() {
            this.getParamsData()
        },
        // tab页签点击事件
        handleTabClick() {
            this.getParamsData()
        },
        // 获取参数的列表数据
        async getParamsData() {
            if (this.selectedCateKeys.length !== 3) {
                this.$message.error('只能选择三级分类！')
                this.selectedCateKeys = []
                this.manyTableData = []
                this.onlyTableData = []
                return
            }
            // 根据所选分类的id和当前所处的面板获取对应的参数
            const {
                data: res
            } = await this.$http.get(`categories/${this.cateId}/attributes`, {
                params: {
                    sel: this.activeName
                }
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取参数失败！')
            }
            res.data.forEach(item => {
                item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                // 控制文本框的显示与隐藏
                item.inputVisible = false
                // 文本框中输入的值
                item.inputValue = ''
            })
            if (this.activeName === 'many') {
                this.manyTableData = res.data
            } else {
                this.onlyTableData = res.data
            }
        },
        // 监听对话框关闭事件
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()
        },
        // 点击按钮添加参数
        addParams() {
            this.$refs.addFormRef.validate(async valid => {
                if (!valid) return
                const {
                    data: res
                } = await this.$http.post(`categories/${this.cateId}/attributes`, {
                    attr_name: this.addForm.attr_name,
                    attr_sel: this.activeName
                })
                if (res.meta.status !== 201) {
                    return this.$message.error('添加参数失败！')
                }
                this.getParamsData()
                this.$message.success('添加参数成功！')
                this.addDialogVisible = false
            })
        },
        // 点击按钮展示修改的对话框
        async showEditDialog(id) {
            // 查询当前参数的信息
            const {
                data: res
            } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
                params: {
                    attr_sel: this.activeName
                }
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取参数信息失败！')
            }
            this.editForm = res.data
            this.editDialogVisible = true
        },
        // 修改参数对话框关闭事件
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        // 点击按钮修改参数
        editParams() {
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return
                const {
                    data: res
                } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
                    attr_name: this.editForm.attr_name,
                    attr_sel: this.activeName
                })
                if (res.meta.status !== 200) {
                    return this.$message.error('修改参数失败！')
                }
                this.$message.success('修改参数成功！')
                this.getParamsData()
                this.editDialogVisible = false
            })
        },
        // 删除参数
        async deleteParams(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该参数，是否继续？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)

            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除操作')
            }

            const {
                data: res
            } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
            if (res.meta.status !== 200) {
                return this.$message.error('删除参数失败！')
            }
            this.$message.success('删除参数成功！')
            this.getParamsData()
        },
        // 文本框失去焦点或enter后触发的函数
        async handleInputConfirm(row) {
            if (row.inputValue.trim().length === 0) {
                row.inputValue = ''
                row.inputVisible = false
                return
            }
            let newValue = row.inputValue.trim()
            row.attr_vals.push(newValue)
            row.inputValue = ''
            row.inputVisible = false
            // 发起请求保存操作到数据库
            this.saveAttrVals(row, 'add')
        },
        // 将对attr_vals的操作保存到数据库
        async saveAttrVals(row, opt) {
            let msg = opt === 'add' ? '添加' : '删除'
            const {
                data: res
            } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
                attr_name: row.attr_name,
                attr_sel: row.attr_sel,
                attr_vals: row.attr_vals.join(' ')
            })
            if (res.meta.status !== 200) {
                return this.$message.error(`${msg}参数项失败！`)
            }
            this.$message.success(`${msg}参数项成功！`)
        },
        // 点击显示文本框
        showInput(row) {
            row.inputVisible = true
            // 文本框自动获得焦点
            // $nextTick方法：当页面上元素被重新渲染之后，才会执行回调函数中的代码
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus()
            })
        },
        // 删除对应的参数和选项
        handleClosed(i, row) {
            row.attr_vals.splice(i, 1)
            this.saveAttrVals(row, 'delete')
        }
    },
    computed: {
        // 如果按钮需要被禁用， 则返回true， 否则返回false
        isBtnDisabled() {
            if (this.selectedCateKeys.length !== 3) {
                return true
            }
            return false
        },
        // 获得当前选中的三级分类id
        cateId() {
            if (this.selectedCateKeys.length === 3) {
                return this.selectedCateKeys[2]
            }
            return null
        },
        // 动态计算对话框标题的文本
        titleText() {
            if (this.activeName === 'many') {
                return '动态参数'
            } else {
                return '静态属性'
            }
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}

.el-tag {
    margin: 8px;
}

.input-new-tag {
    width: 120px;
}
</style>
