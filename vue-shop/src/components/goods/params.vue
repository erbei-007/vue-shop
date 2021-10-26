<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>
            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类:</span>
                    <el-cascader v-model="selectedKeys" :options="cateList" :props="cateProps" @change="handleChange"
                        clearable></el-cascader>
                </el-col>
            </el-row>
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">
                        添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTableData" border stripe>
                        <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" closable
                                    @close="handleClosed(index,scope.row)">
                                    {{item}}
                                </el-tag>
                                <!-- 输入文本框 -->
                                <el-input class="input-new-tag" v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue" ref="saveTagInput" size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)">
                                </el-input>
                                <!-- 添加的按钮 -->
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+
                                    New Tag
                                </el-button>
                            </template>
                        </el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini"
                                    @click="showEditDialog(scope.row.attr_id)">编辑
                                </el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini"
                                    @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态管理" name="only">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">
                        添加属性</el-button>
                    <!-- 静态属性 -->
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" closable
                                    @close="handleClosed(index,scope.row)">
                                    {{item}}
                                </el-tag>
                                <!-- 输入文本框 -->
                                <el-input class="input-new-tag" v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue" ref="saveTagInput" size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)">
                                </el-input>
                                <!-- 添加的按钮 -->
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+
                                    New Tag
                                </el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini"
                                    @click="showEditDialog(scope.row.attr_id)">编辑
                                </el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini"
                                    @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <!-- 添加参数对话框 -->
        <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改参数对话框 -->
        <el-dialog :title="'修改'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
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
                //商品分类列表
                cateList: [],
                //级联选择器
                cateProps: {
                    expandTrigger: 'hover',
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children',
                },
                //级联选择框双向绑定到的数组
                selectedKeys: [],
                //被激活的页签
                activeName: 'many',
                //动态参数数据
                manyTableData: [],
                // 静态属性的数据
                onlyTableData: [],
                //添加属性对话框是否显示
                addDialogVisible: false,
                //添加属性对话框表单内容
                addForm: {},
                addFormRules: {
                    attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
                },
                //编辑修改对话框
                editDialogVisible: false,
                editForm: {},
                editFormRules: {
                    attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
                },


            }

        },
        methods: {
            async getCateList() {
                const { data: res } = await this.$http.get('categories');
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.cateList = res.data;
            },
            //级联选择框选中项变化
            handleChange() {
                //console.log(this.selectedKeys)
                //根据下拉框选择的id获取内容
                this.getParamsData();

            },
            //获取参数
            async getParamsData() {
                if (this.selectedKeys.length !== 3) {
                    this.selectedKeys = [];
                    this.manyTableData = [];
                    this.onlyTableData = [];
                    return
                }
                const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } });
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                //console.log(res.data);
                res.data.forEach(item => {
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : [];
                    //控制文本框的显示与隐藏
                    item.inputVisible = false;
                    //文本框中输入的值
                    item.inputValue = '';
                })
                // console.log(res.data);
                if (this.activeName === 'many') {
                    this.manyTableData = res.data;
                } else {
                    this.onlyTableData = res.data;
                }
            },
            //tab点击事件
            handleTabClick() {
                //console.log(this.activeName)
                this.getParamsData()

            },
            //监听对话框的关闭时事件
            addDialogClosed() {
                this.$refs.addFormRef.resetFields();
            },
            //添加参数
            addParams() {
                this.$refs.addFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
                    if (res.meta.status !== 201) return this.$message.error(res.meta.msg);
                    this.$message.success('添加成功');
                    this.addDialogVisible = false;
                    this.getParamsData();

                })
            },
            //编辑对话框
            async showEditDialog(attr_id) {
                //获取参数信息渲染到表单中也就是双向数据绑定的表单中
                const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, { params: { attr_sel: this.activeName } })
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.editForm = res.data;
                this.editDialogVisible = true
            },
            //编辑按钮关闭事件 重置修改的
            editDialogClosed() {
                this.$refs.editFormRef.resetFields();
            },
            //点击编辑按钮修改参数
            editParams() {
                this.$refs.editFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName });
                    if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                    this.$message.success('修改成功');
                    this.getParamsData();
                    this.editDialogVisible = false;
                })
            },
            //删除参数
            async removeParams(attr_id) {
                const confirmRes = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err)
                if (confirmRes !== "confirm") {
                    return this.$message.info('你又不想删啦？')
                }
                const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`);
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.$message.success('删除成功');
                this.getParamsData();


            },
            //展开行下新建标签文本失去焦点或回车事件
            handleInputConfirm(row) {
                //如果输入的是空串直接制空
                if (row.inputValue.trim().length === 0) {
                    row.inputValue = "";
                    row.inputVisible = false;
                    return
                }
                row.attr_vals.push(row.inputValue.trim());
                row.inputValue = '';
                row.inputVisible = false;
                //发起请求
                this.saveAttrVals(row);

            },
            //点击按钮展示文本框
            showInput(row) {
                row.inputVisible = true;
                //$nextTick当页面上的元素被重新渲染后才会执行回调函数中的代码
                this.$nextTick(_ => {
                    //让文本框自动获得焦点
                    this.$refs.saveTagInput.$refs.input.focus();
                });
            },
            //对attr-val的操作保存到数据库
            async saveAttrVals(row) {
                //发起请求保存到数据库   
                const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
                    attr_name: row.attr_name,
                    attr_sel: row.attr_sel,
                    attr_vals: row.attr_vals.join(' ')
                });
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.$message.success('修改成功');
            },
            //删除标签
            handleClosed(index, row) {
                row.attr_vals.splice(index, 1);
                this.saveAttrVals(row);
            },
        },
        created() {
            this.getCateList();
        },
        computed: {
            //如果按钮要被禁用返回true 否则返回false
            isBtnDisabled() {
                if (this.selectedKeys.length !== 3) {
                    return true;
                }
                return false;
            },
            // 当前下拉框选中的id
            cateId() {
                if (this.selectedKeys.length === 3) {
                    return this.selectedKeys[2];
                }
                return null
            },
            //动态计算标题文本
            titleText() {
                if (this.activeName === "many") {
                    return "动态参数"
                }
                return "静态属性"
            },

        }
    }
</script>

<style scoped>
    .cat_opt {
        margin: 20px 5px;
    }

    .el-tag {
        margin: 10px
    }

    .input-new-tag {
        width: 120px
    }
</style>