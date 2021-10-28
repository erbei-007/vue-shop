<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodList">
                        <el-button slot="append" icon="el-icon-search" @click="getGoodList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="goAddPage">添加商品</el-button>
                </el-col>
            </el-row>
            <el-table :data="goodsList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="商品名称" prop="goods_name"></el-table-column>
                <el-table-column label="商品价格" prop="goods_price" width="95px"></el-table-column>
                <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
                <el-table-column label="商品创建时间" width="140px">
                    <template slot-scope="scope">
                        {{scope.row.add_time | dateFormat}}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="130px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini"
                            @click="removeById(scope.row.goods_id)"></el-button>
                    </template>
                </el-table-column>

            </el-table>
            <!-- 分页区 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper" :total="total" background>
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                queryInfo: {
                    query: '',
                    pagenum: 1,
                    pagesize: 10
                },
                //商品列表
                goodsList: [],
                //总数据条数
                total: 0,
            }
        },
        methods: {
            async getGoodList() {
                const { data: res } = await this.$http.get('goods', { params: this.queryInfo });
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.$message.success('获取商品列表成功')
                console.log(res.data);
                this.goodsList = res.data.goods;
                this.total = res.data.total;
            },
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize;
                this.getGoodList();
            },
            handleCurrentChange(newPage) {
                this.queryInfo.pagenum = newPage;
                this.getGoodList();
            },
            //删除按钮
            async removeById(id) {
                const confirmRes = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err)
                if (confirmRes !== 'confirm') {
                    return this.$message.info('你又取消删除啦！')
                }
                const { data: res } = await this.$http.delete(`goods/${id}`);
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
                this.$message.success('删除成功');
                this.getGoodList();
            },
            //添加商品页面
            goAddPage() {
                this.$router.push('/goods/add');
            }

        },
        created() {
            this.getGoodList()
        }
    }
</script>

<style scoped>

</style>