<template>
    <div class="login-wrap">
        <div class="ms-title" style="font-size: 30px">
            我的收藏夹
        </div>
        <div class="button-container">
            <el-button type="primary" @click="sign_up()">注册</el-button>
            <el-button type="primary" @click="sign_in()">登录</el-button>
        </div>
        <el-dialog
            width="30%"
            title="注册账号"
            :visible.sync="dialogVisible">
            <div class="form-box">
                <el-form :model="form" :rules="rules" ref="form" label-width="150px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password1">
                        <el-input v-model="form.password1" type="password" placeholder="请输入密码"></el-input>
                    </el-form-item>
                    <el-form-item label="确认密码" prop="password2">
                        <el-input v-model="form.password2" type="password" placeholder="请确认密码"></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogVisible = false">取消</el-button>
                        <el-button type="primary" @click="sign_up_confirm()">确认</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import main from 'src/main';

export default {
    data: function () {
        const checkuser = (rule, value, callback) => {
            this.$http.get(main.url + '/users/list').then(
                response => {
                    const userList = response.data;
                    for (let i = 0; i < userList.length; i++) {
                        if (userList[i].username === value) {
                            this.form.ban = '1';
                            callback(new Error('该用户名已存在'));
                            return;
                        }
                    }
                    this.form.ban = '0';
                    callback();
                }
            );
        };
        const checkpasswd = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('请再次确认新密码'));
            } else if (value !== this.form.password1) {
                callback(new Error('两次输入的新密码不一致!'));
            } else {
                callback();
            }
        };
        return {
            dialogVisible: false,
            data: [],
            form: {
                id: '',
                username: '',
                password1: '',
                password2: '',
                role: 2,
                ban: '0' //重名检测，用户名是否已经存在，0存在1不存在
            },
            rules: {
                username: [
                    {required: true, message: '请输入用户名', trigger: 'blur'},
                    {validator: checkuser, trigger: 'change'}
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
            }
        }
    },
    created() {
        this.init();
    },
    methods: {
        init() {
            this.$http.get(main.url + "/favorite/list",
                {params: {owner: 13}}
            ).then(
                success => {
                    this.data = success.data;
                }
            );
            console.log(main.url + "/favorite/list")
        },
        sign_in() {
            this.$router.push({path: '/login'});
        },
        sign_up() {
            this.form = {
                id: '',
                username: '',
                password1: '',
                password2: '',
                role: 2,
                ban: '0'
            };
            this.dialogVisible = true
        },
        sign_up_confirm() {
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
                        'role': 2
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    () => {
                        this.$message({type: 'success', message: '注册成功'});
                        this.form = {
                            id: '',
                            username: '',
                            password1: '',
                            password2: '',
                            role: 0,
                            ban: '0'
                        };
                        this.dialogVisible = false;
                    }
                );
            }
        }
    }
}
</script>

<style scoped>
.button-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    text-align: center;
    line-height: 100px
}

.login-wrap {
    position: relative;
    background: url("/static/img/bg.jpg") no-repeat center;
    width: 100%;
    height: 100%;
}

.ms-title {
    position: absolute;
    top: 40%;
    width: 100%;
    margin-top: -50px;
    text-align: center;
    font-size: 14px;
    color: #fff;
    font-weight: bold;
}

.login-btn button {
    position: absolute;
    width: 40%;
    height: 35px;
    right: 10%;
    top: 10%;
}
</style>
