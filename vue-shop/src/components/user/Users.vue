<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card class="box-card">
            <el-row :gutter="20">
                <el-col :span="12">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserlist">
                        <el-button slot="append" icon="el-icon-search" @click="getUserlist"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 用户渲染区 -->
            <el-table :data="userList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="username" label="姓名"></el-table-column>
                <el-table-column prop="email" label="邮箱"></el-table-column>
                <el-table-column prop="mobile" label="电话"></el-table-column>
                <el-table-column prop="role_name" label="角色"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="190px">
                    <template slot-scope="scope">
                        <el-tooltip effect="dark" content="修改" placement="top" :enterable="false">
                            <!-- 修改按钮 -->
                            <el-button type="primary" icon="el-icon-edit" size="mini"
                                @click="showEditDialog(scope.row.id)">
                            </el-button>
                        </el-tooltip>
                        <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
                            <!-- 删除按钮 -->
                            <el-button type="danger" icon="el-icon-delete" size="mini"
                                @click="removeUserById(scope.row.id)"></el-button>
                        </el-tooltip>
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <!-- 分配角色按钮 -->
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)">
                            </el-button>
                        </el-tooltip>

                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加用户的对话框 -->
        <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
            <!-- 内容主体区域 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改用户对话框 -->
        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUserInfo">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 分配角色对话框 -->
        <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClosed">
            <div>
                <p>当前的用户：{{userInfo.username}}</p>
                <p>当前的角色：{{userInfo.role_name}}</p>
                <p>分配新角色：<el-select v-model="selctedRoleId" placeholder="请选择">
                        <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id">
                        </el-option>
                    </el-select>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
    export default {
        data() {
            //验证邮箱规则
            var checkEmaile = (rule, value, callback) => {
                const regEmaile = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/;
                if (regEmaile.test(value)) {
                    return callback();
                }
                callback(new Error('请输入合法的邮箱'));
            }
            //验证手机号规则
            return {
                queryInfo: {
                    query: '',
                    //当前页数
                    pagenum: 1,
                    //当前每页显示几条数据
                    pagesize: 2,
                },
                userList: [],
                total: 0,
                //控制添加对话框的显示与隐藏
                addDialogVisible: false,
                //添加用户的表单数据
                addForm: {
                    username: '',
                    password: '',
                    email: '',
                    telephone: '',

                },
                //添加表单规则验证
                addFormRules: {
                    username: [
                        { required: true, message: '请输入用户名字', trigger: 'blur' },
                        { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                    ],
                    password: [
                        { required: true, message: '请输入用户密码', trigger: 'blur' },
                        { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                    ],
                    email: [
                        { required: true, message: '请输入邮箱地址', trigger: 'blur' },
                        { validator: checkEmaile, trigger: 'blur' }
                    ],
                    mobile: [
                        { required: true, message: '请输入手机号', trigger: 'blur' },
                        { min: 11, max: 11, message: '请输入合法的手机号', trigger: 'blur' }
                    ]
                },
                //控制修改用户框的显示和隐藏
                editDialogVisible: false,
                //查询到的用户信息对象
                editForm: {},
                //控制分配角色的显示和隐藏
                setRoleDialogVisible: false,
                //需要被分配角色的用户信息
                userInfo: {},
                //所有角色
                rolesList: [],
                //选中的角色
                selctedRoleId: '',

            }
        },
        methods: {
            async getUserlist() {
                const { data: res } = await this.$http.get('users', { params: this.queryInfo });
                console.log(res);
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.userList = res.data.users;
                this.total = res.data.total;
            },
            //监听pagesize改变
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize;
                this.getUserlist();
            },
            //监听页码值改变的时间
            handleCurrentChange(newPage) {
                this.queryInfo.pagenum = newPage;
                this.getUserlist();
            },
            //监听用户状态的改变
            async userStateChange(userinfo) {
                const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`);
                //console.log(res);
                if (res.meta.status !== 200) {
                    userinfo.mg_state = !userinfo.mg_state;
                    return this.$message.error('更新出错啦');
                }
                this.$message.success('更新成功啦');

            },
            //监听添加用户框的关闭事件
            addDialogClose() {
                this.$refs.addFormRef.resetFields();
            },
            addUser() {
                this.$refs.addFormRef.validate(async valid => {
                    if (!valid) return;
                    //通过发起添加用户的网络
                    const { data: res } = await this.$http.post('users', this.addForm);
                    console.log(res);
                    if (res.meta.status !== 201) {
                        this.$message.error('添加用户失败');
                    }
                    this.$message.success('添加用户成功');
                    this.addDialogVisible = false;
                    this.getUserlist();
                })
            },
            //展示编辑用户 的对话框
            async showEditDialog(id) {
                this.editDialogVisible = true;
                const { data: res } = await this.$http.get('users/' + id);
                if (res.meta.status !== 200) {
                    return this.$message.error('查询失败');
                }
                this.editForm = res.data;
                console.log(res)
            },
            //监听修改用户对话框的关闭事件
            editDialogClosed() {
                this.$refs.editFormRef.resetFields();
            },
            //修改用户信息并提交
            editUserInfo() {
                this.$refs.editFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile });
                    if (res.meta.status !== 200) {
                        return this.$message.error('修改用户失败');
                    }
                    this.$message.success('修改用户成功');
                    this.editDialogVisible = false;
                    this.getUserlist();
                })
            },
            //删除用户信息
            async removeUserById(id) {
                const confirmRes = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err);
                if (confirmRes !== 'confirm') {
                    return this.$message.info('取消删除')
                }
                console.log('111')
                const { data: res } = await this.$http.delete('users/' + id);
                if (res.meta.status !== 200) {
                    return this.$message.error('删除失败');
                }
                this.$message.success('删除成功')
                this.getUserlist()
            },
            //展示分配角色的对话框
            async setRole(userInfo) {
                this.userInfo = userInfo;
                //在展示对话框之前获取所有的角色列表
                const { data: res } = await this.$http.get('roles');
                if (res.meta.status !== 200) {
                    return this.$message.error('获取角色列表失败');
                }
                this.rolesList = res.data
                this.setRoleDialogVisible = true;

            },
            //点击确定分配角色
            async saveRoleInfo() {
                if (!this.selctedRoleId) {
                    return this.$message.error('请选择要分配的角色！');
                }
                const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selctedRoleId })
                console.log(res);
                if (res.meta.status !== 200) {
                    return this.$message.error(res.meta.msg);
                }
                this.$message.success('更新角色列表成功')
                this.getUserlist();
                this.setRoleDialogVisible = false;
            },
            //分配角色对话框关闭
            setRoleDialogClosed() {
                this.selctedRoleId = '';
                this.userInfo = {};
            }
        },
        created() {
            this.getUserlist();
        },


    }
</script>

<style scoped>

</style>