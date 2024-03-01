<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item @click.native="returnHome()">
                    <i class="el-icon-setting"></i>
                    {{ username }}的收藏夹
                </el-breadcrumb-item>
                <el-breadcrumb-item>收藏列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div>
            <el-table :data="data" border style="width: 100%" ref="multipleTable">
                <el-table-column label="网站名称" prop="name" min-width="15%"></el-table-column>
                <el-table-column label="网站地址" prop="url" min-width="20%"></el-table-column>
                <el-table-column label="说明" prop="description" min-width="40%"></el-table-column>
                <el-table-column label="可见性" prop="visibility" min-width="10%"></el-table-column>
                <el-table-column label="操作" min-width="15%">
                    <template v-slot="scope">
                        <el-button type="text" @click="visitFavorite(scope.row)">访问</el-button>
                        <el-button type="text" @click="editFavorite(scope.row)">编辑</el-button>
                        <el-button type="text" @click="delFavorite(scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-button type="primary" @click="addFavorite()">添加新收藏</el-button>
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
                    <el-form-item label="可见性" prop="visibility">
                        <el-select v-model="form.new_visibility" placeholder="请设置可见性">
                            <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed = false">取消</el-button>
                        <el-button type="primary" @click="editFavorite_submit('form')">提交</el-button>
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
                    <el-form-item label="可见性" prop="visibility">
                        <el-select v-model="form.visibility" placeholder="请设置可见性">
                            <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button @click="dialogFormVisibleed1 = false">取消</el-button>
                        <el-button type="primary" @click="addFavorite_add(form)">添加</el-button>
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
            options: [{value: 0, label: "仅自己可见"}, {value: 1, label: "公开"}],
            dialogFormVisibleed: false,
            dialogFormVisibleed1: false,
            form: {
                id: '',
                name: '',
                url: '',
                visibility: 0,
                description: ''
            },
            rules: {
                name: [
                    {required: true, message: '请输入网站名称', trigger: 'blur'},
                ],
                url: [
                    {required: true, message: '请输入网站地址', trigger: 'blur'}
                ],
                visibility: [
                    {required: true, message: '请设置可见性', trigger: 'blur'}
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
    computed: {
        username() {
            let username = localStorage.getItem('username');
            return username ? username : this.name;
        }
    },
    methods: {
        init() {
            this.$http.get(main.url + "/favorite/list",
                {params: {'owner': localStorage.getItem('id')}}
            ).then(
                success => {
                    this.data = success.data;
                    let visibility_rule = {
                        0: "仅自己可见",
                        1: "公开"
                    }
                    for (let i = 0; i < this.data.length; i++)
                        this.data[i].visibility = visibility_rule[this.data[i].visibility];
                }
            );
        },
        returnHome() {
            this.$router.push({path: '/admin'})
        },
        visitFavorite(row) { //进入指定的网站
            window.open(row.url, "_blank");
        },
        editFavorite(row) { //修改收藏内容，按钮按下后
            this.dialogFormVisibleed = true;
            let visibilityDummy = {'仅自己可见': 0, '公开': 1};
            this.form = {
                fid: row.fid,
                new_name: row.name,
                new_url: row.url,
                new_visibility: visibilityDummy[row.visibility],
                new_description: row.description
            };
        },
        editFavorite_submit(form) { //修改收藏内容，提交
            if(this.form.new_url === '')
                this.$message({type: 'error', message: 'URL不能为空！'});
            else {
                this.$http.post(main.url + "/favorite/update",
                    {
                        'fid': this.form.fid,
                        'new_name': this.form.new_name,
                        'new_url': this.form.new_url,
                        'new_visibility': this.form.new_visibility,
                        'new_description': this.form.new_description
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    () => {
                        this.$message({type: 'success', message: '修改成功'});
                        this.form = {
                            fid: '',
                            new_name: '',
                            new_url: '',
                            new_visibility: 0,
                            new_description: ''
                        };
                        this.dialogFormVisibleed = false;
                        this.init();
                    }
                )
            }
        },
        addFavorite() {
            this.dialogFormVisibleed1 = true;
            this.form.visibility = 0;
        },
        addFavorite_add(form) { //添加新收藏
            if (this.form.url === '')
                this.$message({type: 'error', message: '网站地址不能为空！'});
            else {
                this.$http.post(main.url + "/favorite/add",
                    {
                        'owner': localStorage.getItem('id'),
                        'name': this.form.name,
                        'url': this.form.url,
                        'visibility': this.form.visibility,
                        'description': this.form.description
                    },
                    {
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        emulateJSON: true
                    }).then(
                    () => {
                        this.$message({type: 'success', message: '添加成功'});
                        this.form = {
                            owner: '',
                            name: '',
                            url: '',
                            visibility: 0,
                            description: ''
                        };
                        this.init();
                    }
                );
                this.dialogFormVisibleed1 = false;
            }
        },
        delFavorite(row) { //删除收藏记录
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
                    () => {
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
