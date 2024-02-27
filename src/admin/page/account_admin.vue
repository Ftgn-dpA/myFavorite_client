<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-setting"></i> 管理</el-breadcrumb-item>
                <el-breadcrumb-item>用户列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div>
            <el-table :data="data" border style="width: 70%" ref="multipleTable">
                <el-table-column label="ID" prop="id" min-width="80%"></el-table-column>
                <el-table-column label="用户名" prop="username" min-width="200%"></el-table-column>
                <el-table-column label="角色" prop="role" min-width="110%"></el-table-column>
                <el-table-column label="创建时间" prop="create_time" min-width="250%"></el-table-column>
                <el-table-column label="操作" min-width="110%">
                    <template v-slot="scope">
                        <el-button type="text" @click="openedit(scope.row)" v-if="check_role(scope.row)">修改
                        </el-button>
                        <el-button type="text" @click="del_user(scope.row)" v-if="check_role(scope.row)">删除
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div style="text-align: center">
                <el-button type="primary" @click="adduser()" style="margin-top: 100px">添加账号</el-button>
            </div>
        </div>
        <el-dialog
            width="30%"
            title="修改账号"
            :visible.sync="dialogFormVisibleed">
            <div>
                <el-form :model="form" :rules="rules" ref="form" label-width="150px">
                    <el-form-item label="新密码" prop="password1">
                        <el-input v-model="form.password1" type="password" placeholder="请输入新密码"></el-input>
                    </el-form-item>
                    <el-form-item label="确认新密码" prop="password2">
                        <el-input v-model="form.password2" type="password" placeholder="请确认新密码"></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button type="primary" @click="change_pass('form')">修改密码</el-button>
                    </el-form-item>
                    <el-form-item label="账号类型" prop="role">
                        <el-select v-model="form.role" placeholder="请选择账号类型">
                            <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button type="primary" @click="change_role('form')">修改账号类型</el-button>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed = false">取 消</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-dialog>
        <el-dialog
            width="30%"
            title="添加账号"
            :visible.sync="dialogFormVisibleed1">
            <div class="form-box">
                <el-form :model="form" :rules="rules1" ref="form" label-width="150px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password1">
                        <el-input v-model="form.password1" type="password" placeholder="请输入密码"></el-input>
                    </el-form-item>
                    <el-form-item label="确认密码" prop="password2">
                        <el-input v-model="form.password2" type="password" placeholder="请确认密码"></el-input>
                    </el-form-item>
                    <el-form-item label="账号类型" prop="role">
                        <el-select v-model="form.role" placeholder="请选择账号类型">
                            <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed1 = false">取 消</el-button>
                        <el-button type="primary" @click="adduser_submit(form)">添加</el-button>
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
        var checkpasswd = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('请再次确认新密码'));
            } else if (value !== this.form.password1) {
                callback(new Error('两次输入的新密码不一致!'));
            } else {
                callback();
            }
        };
        var checkuser = (rule, value, callback) => {
            for (let i = 0; i < this.data.length; i++) {
                if (this.data[i].username === value) {
                    this.form.ban = '1';
                    callback(new Error('该用户名已存在'));
                }
            }
        };
        return {
            data: [],
            dialogFormVisibleed: false,
            dialogFormVisibleed1: false,
            options: [{value: 0, label: "超级管理员"}, {value: 1, label: "管理员"}, {value: 2, label: "普通用户"}],
            form: {
                id: '',
                username: '',
                password1: '',
                password2: '',
                role: 0,
                ban: '0' //重名检测，用户名是否已经存在，0存在1不存在
            },
            rules: {
                password1: [
                    {required: true, message: '请输入新密码', trigger: 'blur'}
                ],
                password2: [
                    {required: true, message: '请确认新密码', trigger: 'blur'},
                    {validator: checkpasswd, trigger: 'blur'},
                ],
                role: [
                    {required: true, message: '请选择账号类型', trigger: 'blur'}
                ]
            },
            rules1: {
                username: [
                    {required: true, message: '请输入用户名', trigger: 'blur'},
                    {validator: checkuser, trigger: 'blur'}
                ],
                password1: [
                    {required: true, message: '请输入密码', trigger: 'blur'}
                ],
                password2: [
                    {required: true, message: '请确认密码', trigger: 'blur'},
                    {validator: checkpasswd, trigger: 'blur'},
                ],
                role: [
                    {required: true, message: '请选择账号类型', trigger: 'blur'}
                ]
            },
        }
    },
    created() {
        if (localStorage.getItem('username') === "") {
            this.$router.replace('/login');
        } else {
            this.init();
        }
    },
    methods: {
        init() {
            this.$http.get(main.url + "/users/list").then(
                success => {
                    this.data = success.data;
                    for (let i = 0; i < this.data.length; i++) {
                        if (this.data[i].role === 0)
                            this.data[i].role = "超级管理员";
                        else if (this.data[i].role === 1)
                            this.data[i].role = "管理员";
                        else
                            this.data[i].role = "普通用户";
                    }
                }
            )
        },
        check_role(row) {
            let roleMap = {
                '超级管理员': 0,
                '管理员': 1,
                '普通用户': 2
            };
            return localStorage.getItem('role') < roleMap[row.role];
        },
        openedit(row) {
            this.dialogFormVisibleed = true;
            this.form = {
                password1: '',
                password2: '',
                id: row.id,
                role: 0
            };
        },
        change_pass(form) { //修改密码
            if (this.form.password1 !== this.form.password2) {
                this.$message({type: 'error', message: '两次密码不一样！'});
            } else {
                let crypto = require('crypto');
                const md5 = crypto.createHash('md5');
                md5.update(this.form.password0);
                let md5password0 = md5.digest('hex');
                md5.update(this.form.password1);
                let md5password1 = md5.digest('hex');
                this.$http.post(main.url + "/users/alt_password",
                    {
                        'id': this.form.id,
                        'old_password': md5password0,
                        'new_password': md5password1,
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    success => {
                        this.$message({type: 'success', message: '修改成功'});
                        this.form = {
                            id: '',
                            username: '',
                            password1: '',
                            password2: '',
                            role: 0,
                            ban: '0'
                        };
                        this.dialogFormVisibleed = false;
                    }
                );
            }
        },
        adduser() {
            this.form = {
                id: '',
                username: '',
                password1: '',
                password2: '',
                role: 0,
                ban: '0'
            };
            this.dialogFormVisibleed1 = true
        },
        adduser_submit(form) { //添加新用户
            if (this.form.username === "")
                this.$message({type: 'error', message: '用户名不能为空！'});
            else if (this.form.password1 !== this.form.password2)
                this.$message({type: 'error', message: '两次密码不一样！'});
            else if (this.form.ban === "1")
                this.$message({type: 'error', message: '用户名已存在！'});
            else {
                let crypto = require('crypto');
                const md5 = crypto.createHash('md5');
                md5.update(this.form.password1);
                let md5password = md5.digest('hex');
                this.$http.post(main.url + "/users/register",
                    {
                        'username': this.form.username,
                        'password': md5password,
                        'role': this.form.role
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    success => {
                        this.$message({type: 'success', message: '添加成功'});
                        this.form = {
                            id: '',
                            username: '',
                            password1: '',
                            password2: '',
                            role: 0,
                            ban: '0'
                        };
                        this.init();
                    }
                );
                this.dialogFormVisibleed1 = false
            }
        },
        del_user(row) {  //删除用户
            if (row.username === localStorage.getItem('username'))
                this.$message({type: 'info', message: '不能删除自己！'});
            else if (row.id <= localStorage.getItem('role'))
                this.$message({type: 'info', message: '没有权限！'});
            else {
                this.$confirm('请确认是否要删除该用户！', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http.post(main.url + "/users/del",
                        {'uid': row.id,},
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
            }
        },
        change_role(form) { //修改账号类型
            this.$http.post(main.url + "/users/alt_role",
                {
                    'uid': this.form.id,
                    'role': this.form.role,
                },
                {
                    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                    emulateJSON: true
                }).then(
                success => {
                    this.$message({type: 'success', message: '修改成功'});
                    this.form = {
                        id: '',
                        username: '',
                        password1: '',
                        password2: '',
                        role: 0,
                        ban: '0'
                    };
                    this.init();
                }
            );
            this.dialogFormVisibleed = false;
        }
    }
}
</script>
