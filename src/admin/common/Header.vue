<template>
    <div class="header">
        <div class="logo">{{ username }}的收藏夹</div>
        <div class="user-info">
            <el-dropdown trigger="click" @command="handleCommand">
                <span class="el-dropdown-link">
                    <img class="user-logo" src="../../../static/img/img.jpg" alt="">
                    {{ username }}
                </span>
                <el-dropdown-menu v-slot="dropdown">
                    <el-dropdown-item command="change_pass">修改密码</el-dropdown-item>
                    <el-dropdown-item command="logout">退出</el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </div>
        <el-dialog
            width="30%"
            title="修改密码"
            :visible.sync="dialogFormVisibleed1">
            <div class="form-box">
                <el-form :model="form1" :rules="rules1" ref="form1" label-width="150px">
                    <el-form-item label="旧密码" prop="name">
                        <el-input v-model="password0" type="password0" placeholder="请输入新密码"></el-input>
                    </el-form-item>
                    <el-form-item label="新密码" prop="name">
                        <el-input v-model="password1" type="password1" placeholder="请输入新密码"></el-input>
                    </el-form-item>
                    <el-form-item label="确认密码" prop="name">
                        <el-input v-model="password2" type="password2" placeholder="请输入确认密码"></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed1 = false">取消</el-button>
                        <el-button type="primary" @click="submit()">确认</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-dialog>
    </div>
</template>
<script>
import main from "../../main";

export default {
    data() {
        return {
            name: '用户名',
            dialogFormVisibleed1: false,
            password0: '',
            password1: '',
            password2: '',
            form1: {},
        }
    },
    computed: {
        username() {
            let username = localStorage.getItem('username');
            return username ? username : this.name;
        }
    },
    methods: {
        handleCommand(command) {
            if (command === 'logout') {
                localStorage.removeItem('username');
                localStorage.removeItem('role');
                localStorage.removeItem('id');
                this.$router.push('/');
            } else if (command === 'change_pass') {
                this.dialogFormVisibleed1 = true
            }
        },
        submit() {
            if (this.password1 === "")
                this.$message({type: 'info', message: '密码不能为空！'});
            else if (this.password1 !== this.password2)
                this.$message({type: 'info', message: '确认密码与新密码不一致！'});
            else {
                let crypto = require('crypto');
                const md5_0 = crypto.createHash('md5');
                const md5_1 = crypto.createHash('md5');
                md5_0.update(this.password0);
                let md5password0 = md5_0.digest('hex');
                md5_1.update(this.password1);
                let md5password1 = md5_1.digest('hex');
                this.$http.post(main.url + "/users/alt_password",
                    {
                        'id': localStorage.getItem('id'),
                        'old_password': md5password0,
                        'new_password': md5password1,
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    success => {
                        if (success.data === '0')
                            this.$message({type: 'success', message: '密码修改成功！'});
                        else
                            this.$message({type: 'error', message: '旧密码错误！'});
                    }
                );
                this.dialogFormVisibleed1 = false;
            }
        }
    }
}
</script>
<style scoped>
.header {
    position: relative;
    box-sizing: border-box;
    width: 100%;
    height: 70px;
    font-size: 22px;
    line-height: 70px;
    color: #fff;
}

.header .logo {
    float: left;
    width: 250px;
    text-align: center;
}

.user-info {
    float: right;
    padding-right: 50px;
    font-size: 16px;
    color: #fff;
}

.user-info .el-dropdown-link {
    position: relative;
    display: inline-block;
    padding-left: 50px;
    color: #fff;
    cursor: pointer;
    vertical-align: middle;
}

.user-info .user-logo {
    position: absolute;
    left: 0;
    top: 15px;
    width: 40px;
    height: 40px;
    border-radius: 50%;
}
</style>
