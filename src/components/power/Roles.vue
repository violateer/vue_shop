<template>
<div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <!-- 添加角色按钮 -->
        <el-row>
            <el-col>
                <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <!-- 角色列表 -->
        <el-table :data="rolesList" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template v-slot="scope">
                    <el-row :class="['bdbottom', 'vcenter', i1 === 0 ? 'bdtop' : '']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二级三级权限 -->
                        <el-col :span="19">
                            <!-- 二级权限 -->
                            <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <!-- 三级权限 -->
                                <el-col :span="18">
                                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                    <!-- <pre>{{scope.row}}</pre> -->
                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px">
                <template v-slot="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
                    <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
        <!-- 内容主题区 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="addForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="addForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addrole">确 定</el-button>
        </span>
    </el-dialog>
    <!-- 修改角色的对话框 -->
    <el-dialog title="添加角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <!-- 内容主题区 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="editForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="editForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editRoleInfo">确 定</el-button>
        </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
        <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    data() {
        return {
            // 所有角色列表
            rolesList: [],
            addDialogVisible: false,
            // 添加用户的表单数据
            addForm: {
                roleName: '',
                roleDesc: '',
            },
            // 添加表单的验证规则
            addFormRules: {
                roleName: [{
                        required: true,
                        message: '请输入角色名称',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 4,
                        message: '用户名的长度在2-4个字符之间',
                        trigger: 'blur'
                    }
                ],
                roleDesc: [{
                        required: true,
                        message: '请输入角色描述',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 15,
                        message: '密码的长度在2-15个字符之间',
                        trigger: 'blur'
                    }
                ]
            },
            // 控制修改用户对话框的显示与隐藏
            editDialogVisible: false,
            // 查询到的用户信息对象
            editForm: {},
            // 修改表单的验证规则
            editFormRules: {
                roleName: [{
                        required: true,
                        message: '请输入角色名称',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 4,
                        message: '用户名的长度在2-4个字符之间',
                        trigger: 'blur'
                    }
                ],
                roleDesc: [{
                        required: true,
                        message: '请输入角色描述',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 15,
                        message: '密码的长度在2-15个字符之间',
                        trigger: 'blur'
                    }
                ]
            },
            // 控制分配权限对话框的显示与隐藏
            setRightDialogVisible: false,
            // 所有权限的数据
            rightsList: [],
            // 树形控件的属性绑定对象
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            //默认选中的节点id值数组
            defKeys: [],
            // 当前即将分配权限额角色id
            roleId: ''
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        // 获取所有角色的列表
        async getRolesList() {
            const {
                data: res
            } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败！')
            }
            this.rolesList = res.data
        },
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()
        },
        // 点击按钮添加新角色
        addrole() {
            this.$refs.addFormRef.validate(async valid => {
                if (!valid) return
                // 发起添加角色的网络请求
                const {
                    data: res
                } = await this.$http.post('roles', this.addForm)
                if (res.meta.status != 201) {
                    this.$message.error('添加角色失败')
                }
                this.$message.success('添加角色成功')
                // console.log(res)
                // 隐藏对话框
                this.addDialogVisible = false
                // 重新获取角色列表数据
                this.getRolesList()
            })
        },
        // 展示编辑用户的对话框
        async showEditDialog(id) {
            const {
                data: res
            } = await this.$http.get('roles/' + id)
            if (res.meta.status != 200) {
                return this.$message.error('查询用户信息失败')
            }
            this.editForm = res.data
            this.editDialogVisible = true
        },
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        // 点击按钮修改新角色
        editRoleInfo() {
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) {
                    return this.$message.error('信息填写错误')
                }
                // 发起修改用户信息的数据请求
                const {
                    data: res
                } = await this.$http.put('roles/' + this.editForm.roleId, {
                    roleName: this.editForm.roleName,
                    roleDesc: this.editForm.roleDesc
                })
                if (res.meta.status !== 200) {
                    return this.$message.error('更新用户信息失败！')
                }
                this.editDialogVisible = false
                this.getRolesList()
                this.$message.success('更新用户信息成功！')
            })
        },
        // 根据角色ID删除角色
        async removeRoleById(id) {
            // 弹框询问用户是否删除
            const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            // 如果用户确认删除，返回字符串confirm, 取消删除则返回字符串cancel
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }

            const {
                data: res
            } = await this.$http.delete('roles/' + id)
            if (res.meta.status != 200) {
                return this.$message.error('删除角色失败！')
            }
            this.$message.success('删除角色成功！')
            this.getRolesList()
        },
        // 根据id删除对应的权限
        async removeRightById(role, rightId) {
            // 弹框提示用户是否删除
            const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            // 如果用户确认删除，返回字符串confirm, 取消删除则返回字符串cancel
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }

            const {
                data: res
            } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if (res.meta.status != 200) {
                return this.$message.error('删除权限失败！')
            }
            this.$message.success('删除权限成功！')
            role.children = res.data
        },
        // 展示分配权限的对话框
        async showSetRightDialog(role) {
            this.roleId = role.id
            // 获取所有权限的数据
            const {
                data: res
            } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限列表失败！')
            }
            this.rightsList = res.data
            // 递归获取三级节点id
            this.getLeafKeys(role, this.defKeys)
            this.$message.success('获取权限列表成功！')
            this.setRightDialogVisible = true
        },
        // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys数组中
        getLeafKeys(node, arr) {
            // 如果当前node节点不包含children属性， 则为三级节点
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => this.getLeafKeys(item, arr))
        },
        // 监听分配权限对话框的关闭事件
        setRightDialogClosed() {
            this.defKeys = []
        },
        // 点击为角色分配权限
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const {
                data: res
            } = await this.$http.post(`roles/${this.roleId}/rights`, {
                rids: idStr
            })
            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败！')
            }
            this.$message.success('分配权限成功！')
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.bdtop {
    border-top: 1px solid #eee
}

.bdbottom {
    border-bottom: 1px solid #eee
}

.vcenter {
    display: flex;
    align-items: center;
}
</style>
