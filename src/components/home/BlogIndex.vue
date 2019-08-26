<template>
  <div>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.name" placeholder="姓名"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="getUsers">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="info" @click="addUser">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>


    <el-table :data="userInfoList" style="width: 100%">
      <el-table-column prop="name" label="姓名" width="180"></el-table-column>
      <el-table-column prop="sex" label="性别" width="180">
        <template slot-scope="scope">
          {{scope.row.sex == false ? '男' : '女'}}
        </template>
      </el-table-column>
      <el-table-column prop="age" label="年龄" width="180"></el-table-column>
      <el-table-column prop="address" label="地址" width="180"></el-table-column>
      <el-table-column prop="email" label="邮件" width="180"></el-table-column>

      <!--第二步  开始进行修改和查询操作-->
      <el-table-column label="操作" align="center" min-width="100">
        <template slot-scope="scope">
          <el-button type="text" @click="checkDetail(scope.row)">查看详情</el-button>
          <el-button type="info" @click="modifyUser(scope.row)">修改</el-button>
          <el-button type="info" @click="deleteUser(scope.row)">删除</el-button>
        </template>
      </el-table-column>
      <!--编辑与增加的页面-->
    </el-table>

    <!--新增界面-->
    <el-dialog title="记录" :visible.sync="dialogVisible" width="40%" :close-on-click-modal="false">
      <el-form :model="addFormData" :rules="rules2" ref="addFormData" label-width="80px" class="demo-ruleForm login-container">
        <el-form-item label="姓名" prop="name">
          <el-input type="text" v-model="addFormData.name"></el-input>
        </el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="addFormData.sex">
            <el-radio :label="false">男</el-radio>
            <el-radio :label="true">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="年龄" prop="age">
          <el-input type="text" v-model="addFormData.age"></el-input>
        </el-form-item>
        <el-form-item label="地址" prop="address">
          <el-input type="text" v-model="addFormData.address"></el-input>
        </el-form-item>
        <el-form-item label="邮件" prop="email">
          <el-input type="text" v-model="addFormData.email"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
                <el-button @click.native="dialogVisible = false,addFormData={name:'',sex:0,age:'',address:'',email:''}">取 消</el-button>
                <el-button v-if="isView" type="primary" @click.native="addSubmit">确 定</el-button>
            </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: 'home',
    data() {
      return {
        userInfoList: [],
        addFormReadOnly: true,
        dialogVisible: false,
        isView: true,
        addFormData: {
          name: '',
          sex: 0,
          age:'',
          address:'',
          email:''
        },
        rules2: {
          name: [{
            required: true,
            message: '用户名不能为空',
            trigger: 'blur'
          }],
          age: [{
            required: true,
            message: '年龄不能为空',
            trigger: 'blur'
          }],
          address: [{
            required: true,
            message: '地址不能为空',
            trigger: 'blur'
          }],
          email: [{
            required: true,
            message: '邮件不能为空',
            trigger: 'blur'
          }]
        },
        filters: {
          name: ''
        }
      }
    },
    mounted: function () {
      this.loadData();
    },
    methods: {
      loadData() {
        let param = {filter : this.filters.name};
        this.$axios.get('/user/userlist',{params: param}).then((result) => {
          var _data = result.data;
          this.userInfoList = _data;
        });
      },
      getUsers() {
        this.loadData();
      },
      addUser() {
        this.addFormData = {
          name: '',
          sex: 0,
          age:'',
          address:'',
          email:''
        };
        this.isView = true;
        this.dialogVisible = true;
        //    this.addFormReadOnly = false;
      },
      checkDetail(rowData) {
        this.addFormData = Object.assign({}, rowData);
        this.isView = false;
        this.dialogVisible = true;
        //    this.addFormReadOnly = true;
      },
      modifyUser(rowData) {
        this.addFormData = Object.assign({}, rowData);
        this.isView = true;
        this.dialogVisible = true;
        //    this.addFormReadOnly = false;
      },
      deleteUser(rowData) {
        this.$alert('是否删除这条记录', '信息删除', {
          confirmButtonText: '确定',
          callback: action => {
            var param = {
              userId: rowData.id
            };
            this.$axios.delete("/user/delete", {params: param}).then((result) => {
              console.info(result);
              if (result.data.success) {
                this.$message({
                  type: 'info',
                  message: `已删除`
                });
              } else {
                this.$message({
                  type: 'info',
                  message: `删除失败`
                });

              }
              this.loadData();
            });

          }
        });

      },
      //增加一条记录
      addSubmit() {
        //先判断表单是否通过了判断
        this.$refs.addFormData.validate((valid) => {
          //代表通过验证 ,将参数传回后台
          if (valid) {
            let param = Object.assign({}, this.addFormData);
            this.$axios.post("/user/submit", param).then((result) => {
              if (result.data.success) {
                this.$message({
                  type: 'info',
                  message: result.data.msg,
                });
                this.loadData();
              } else {
                this.$message({
                  type: 'info',
                  message: '添加失败',
                });
              }
              this.dialogVisible = false;
            });
          }

        });
      }

    }

  }
</script>

<style>
  body {
    background: #DFE9FB;
  }
</style>
