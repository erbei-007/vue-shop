<template>
    <div>
        <!-- 面包屑导航栏 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片 -->
        <el-card>
            <!-- 添加角色按钮 -->
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 数据渲染区 -->
            <el-table :data="rolesList" style="width: 100%" border stripe>
                <el-table-column type='expand'>
                    <template slot-scope="scope">
                        <el-row v-for="(item1,index1) in scope.row.children" :key="item1.id"
                            :class="['bdbuttom',index1 === 0 ? 'bdtop':'','vcenter']">
                            <!-- 一级权限 -->
                            <el-col :span=" 4">
                                <el-tag closable @close="removeRightById(scope.row,item1.id)">
                                    {{item1.authName}}
                                </el-tag>
                                <i class="el-icon-caret-right
                                    "></i>
                            </el-col>
                            <!-- 二级、三级权限 -->
                            <el-col :span="20">
                                <el-row v-for="(item2,index2) in item1.children" :key="item2.id"
                                    :class="[index2 === 0 ? '':'bdtop','vcenter']">
                                    <el-col :span="5">
                                        <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">
                                            {{item2.authName}}
                                        </el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="19">
                                        <el-tag type="warning" v-for="(item3,index3) in item2.children" :key="item3.id"
                                            closable @close="removeRightById(scope.row,item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>

                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type='index'>
                </el-table-column>
                <el-table-column prop="roleName" label="角色名称">
                </el-table-column>
                <el-table-column prop="roleDesc" label="角色描述">
                </el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini"
                            @click="showSetRightDoialog(scope.row)">分配权限
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 分配权限 -->
        <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
            <!-- 树形控件 -->
            <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all
                :default-checked-keys="defKeys" ref="treeRef"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        created() {
            this.getRolesList();
        },
        data() {
            return {
                //所有角色的列表
                rolesList: [],
                //控制分配权限的对话框的显示与隐藏
                setRightDialogVisible: false,
                //所有权限的数据
                rightsList: [],
                //树形控件的属性绑定对象
                treeProps: {
                    children: 'children',
                    label: 'authName'
                },
                //默认选中的节点id值数组
                defKeys: [],
                //当前即将分配角色的id
                roleId: '',
            }
        },
        methods: {
            //获取所有角色的列表
            async getRolesList() {
                const { data: res } = await this.$http.get('roles');
                console.log(res);
                if (res.meta.status !== 200) return this.$message.error('请求失败')
                this.rolesList = res.data;
            },
            //根据id删除三级标签下的属性
            async removeRightById(role, id) {
                //弹框提示用户是否删除
                const confirmRes = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err)

                if (confirmRes !== 'confirm') {
                    return this.$message.info('已取消删除')
                }
                const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${id}`);
                if (res.meta.status !== 200) {
                    return this.$message.error('删除失败');
                }
                role.children = res.data;
            },
            //展示分配权限的对话框
            async showSetRightDoialog(role) {
                this.roleId = role.id;
                const { data: res } = await this.$http.get('rights/tree');
                if (res.meta.status !== 200) {
                    return this.$message.error('获取失败');
                }
                this.rightsList = res.data;
                //展示的时候就吧默认的权限勾选
                this.getLeafKeys(role, this.defKeys)
                //console.log(this.defKeys)
                this.setRightDialogVisible = true;
            },
            //获取三级节点的id
            getLeafKeys(node, arr) {
                if (!node.children) {
                    return arr.push(node.id)
                }
                node.children.forEach(item => {
                    this.getLeafKeys(item, arr)
                })
            },
            //监听分配权限对话框的关闭时间
            setRightDialogClosed() {
                this.defKeys = [];
            },
            //保存分配角色
            async allotRights() {
                const keys = [
                    ...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()
                ];
                const idStr = keys.join(",");
                const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr });
                if (res.meta.status !== 200) {
                    return this.$message.error('分配权限失败');
                }
                this.$message.success('分配权限成功');
                this.getRolesList();
                this.setRightDialogVisible = false;
            },
        },
    }
</script>

<style scoped>
    .el-tag {
        margin: 7px;
    }

    .bdtop {
        border-top: 1px solid #eee;
    }

    .bdbuttom {
        border-bottom: 1px solid #eee;
    }

    .vcenter {
        display: flex;
        align-items: center;
    }
</style>