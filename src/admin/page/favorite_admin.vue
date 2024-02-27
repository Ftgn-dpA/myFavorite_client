<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-setting"></i> 管理</el-breadcrumb-item>
                <el-breadcrumb-item>收藏列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div>
            <el-table :data="data" border style="width: 90%" ref="multipleTable">
                <el-table-column label="网站名称" prop="name" width="250px"></el-table-column>
                <el-table-column label="网站地址" prop="url" width="300px"></el-table-column>
                <el-table-column label="说明" prop="description" width="328px"></el-table-column>
                <el-table-column label="操作" width="150px">
                    <template slot-scope="scope" width="100px">
                        <el-button type="text" @click="gotourl(scope.row)">进入</el-button>
                        <el-button type="text" @click="openedit(scope.row)">修改</el-button>
                        <el-button type="text" @click="del_fav(scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-button type="primary" @click="addfavorite()" align="center">添加新收藏</el-button>
        </div>
        <el-dialog
            width="30%"
            title="修改"
            :visible.sync="dialogFormVisibleed">
            <div>
                <el-form :model="form" :rules="rules" ref="form" label-width="150px">
                    <el-form-item label="网站名称" prop="name">
                        <el-input v-model="form.new_name" placeholder="请输入网站名称"></el-input>
                    </el-form-item>
                    <el-form-item label="网站地址" prop="url">
                        <el-input v-model="form.new_url" placeholder="请输入网站地址"></el-input>
                    </el-form-item>
                    <el-form-item label="说明" prop="description">
                        <el-input v-model="form.new_description" placeholder="请输入说明"></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed = false">取消</el-button>
                        <el-button type="primary" @click="submit('form')">修改</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-dialog>
        <el-dialog
            width="30%"
            title="添加收藏"
            :visible.sync="dialogFormVisibleed1">
            <div class="form-box">
                <el-form :model="form" :rules="rules" ref="form" label-width="150px">
                    <el-form-item label="网站名称" prop="name">
                        <el-input v-model="form.name" placeholder="请输入网站名称"></el-input>
                    </el-form-item>
                    <el-form-item label="网站地址" prop="url">
                        <el-input v-model="form.url" placeholder="请输入网站地址"></el-input>
                    </el-form-item>
                    <el-form-item label="说明" prop="description">
                        <el-input v-model="form.description" placeholder="请输入说明"></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed1 = false">取消</el-button>
                        <el-button type="primary" @click="addnew(form)">添加</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import main from "../../main";

export default {
    data: function () {
        return {
            data: [],
            dialogFormVisibleed: false,
            dialogFormVisibleed1: false,
            form: {
                id: '',
                name: '',
                url: '',
                description: ''
            },
            rules: {
                name: [
                    {required: true, message: '请输入网站名称', trigger: 'blur'}
                ],
                url: [
                    {required: true, message: '请输入网站地址', trigger: 'blur'}
                ],
                description: [
                    {required: false, message: '请输入说明', trigger: 'blur'}
                ]
            }
        }
    },
    created() {
        if (localStorage.getItem('username') === "") {
            this.$router.replace('/login')
        } else {
            this.init();
        }
    },
    methods: {
        init() {
            this.$http.get(main.url + "/favorite/list",
                {params: {'owner': localStorage.getItem('id')}}
            ).then(
                success => {
                    this.data = success.data;
                }
            );
        },
        gotourl(row) { //进入指定的网站
            window.open(row.url, "_blank");
        },
        openedit(row) { //修改收藏内容，按钮按下后
            this.dialogFormVisibleed = true;
            this.form = {
                fid: row.fid,
                new_name: row.name,
                new_url: row.url,
                new_description: row.description
            };
        },
        submit(form) { //修改收藏内容，提交
            this.$http.post(main.url + "/favorite/update",
                {
                    'fid': this.form.fid,
                    'new_name': this.form.new_name,
                    'new_url': this.form.new_url,
                    'new_description': this.form.new_description
                },
                {
                    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                    emulateJSON: true
                }).then(
                success => {
                    this.$message({type: 'success', message: '修改成功'});
                    this.form = {
                        fid: '',
                        new_name: '',
                        new_url: '',
                        new_description: ''
                    };
                    this.dialogFormVisibleed = false;
                    this.init();
                }
            )
        },
        addfavorite() {
            this.dialogFormVisibleed1 = true
        },
        addnew(form) { //添加新收藏
            if (this.form.name === "")
                this.$message({type: 'error', message: '网站名称！'});
            else if (this.form.url === "")
                this.$message({type: 'error', message: '网站地址不能为空！'});
            else {
                this.$http.post(main.url + "/favorite/add",
                    {
                        'owner': localStorage.getItem('id'),
                        'name': this.form.name,
                        'url': this.form.url,
                        'description': this.form.description
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    success => {
                        this.$message({type: 'success', message: '添加成功'});
                        this.form = {
                            owner: '',
                            name: '',
                            url: '',
                            description: ''
                        };
                        this.init();
                    }
                );
                this.dialogFormVisibleed1 = false;
            }
        },
        del_fav(row) { //删除收藏记录
            this.$confirm('请确认是否要删除该收藏记录！', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http.post(main.url + "/favorite/del",
                    {'fid': row.fid,},
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    success => {
                        this.$message({type: 'success', message: '已删除'});
                        this.init();
                    }
                );
            }).catch(() => {
                this.$message({type: 'info', message: '已取消'});
            });
        },
    }
}
</script>
