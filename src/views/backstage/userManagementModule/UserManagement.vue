<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>平台用户管理</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 搜索筛选 -->
    <el-form :inline="true" class="searchBox">
      <el-form-item label="名称">
        <el-input size="small" clearable style="width: 300px;" prefix-icon="el-icon-search" v-model="queryForm.nickname"
                  placeholder="请输入搜索的用户名称"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" icon="el-icon-search" @click="search()">&emsp;搜&emsp;索&emsp;</el-button>
      </el-form-item>
    </el-form>

    <!-- 数据表格-->
    <el-table :data="result" border style="width: 100%" height="450">
      <el-table-column type="index" label="序" :index="indexMethod" align="center">
      </el-table-column>
      <el-table-column prop="nickname" label="用户账号" min-width="30">
      </el-table-column>
      <el-table-column prop="phonenumber" label="电话号码" min-width="26">
      </el-table-column>
      <el-table-column prop="usertypeName" label="用户类型" min-width="26">
      </el-table-column>
      <el-table-column prop="headimage" label="用户头像" min-width="20">
      </el-table-column>
      <el-table-column prop="bitstateName" label="用户状态" min-width="20">
      </el-table-column>
      <el-table-column label="操作" min-width="25" align="center">
        <template slot-scope="scope">
          <el-button size="mini" icon="el-icon-edit" @click="handleEdit(scope.row)">编辑</el-button>&emsp;
          <el-button size="mini" icon="el-icon-delete" type="danger" @click="handleDelete(scope.$index, scope.row)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 完整分页-->
    <el-pagination style="margin-top: 15px;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                   :current-page="queryForm.current_page" :page-sizes="[10, 20, 30, 40]"
                   :page-size="queryForm.page_size" layout="total, sizes, prev, pager, next, jumper"
                   :total="queryForm.total_count">
    </el-pagination>


    <!-- 修改用户相关信息的操作 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" width="40%" @close="handleDialogClose">
      <el-form :model="mergeForm" :rules="mergeFormRules" ref="mergeForm" :label-position="labelPosition"
               label-width="80px">
        <el-form-item label="用户账号" prop="nickname">
          <el-input v-model="mergeForm.nickname" placeholder="请输入2-10的用户账号"></el-input>
        </el-form-item>
        <el-form-item label="用户密码" prop="userpassword">
          <el-input type="password" v-model="mergeForm.userpassword" placeholder="请输入6-10的用户密码" autocomplete="off"
                    show-password></el-input>
        </el-form-item>
        <el-form-item label="电话号码" prop="phonenumber">
          <el-input type="text" v-model="mergeForm.phonenumber" placeholder="请输入确认你输入的密码" autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="用户类型" prop="usertype">
          <div>
            <el-radio v-model="mergeForm.usertype" label="1" size="medium" border>普通用户</el-radio>
            <el-radio v-model="mergeForm.usertype" label="2" size="medium" border>投资者账户</el-radio>
          </div>
        </el-form-item>
        <el-form-item label="用户状态" prop="bitstate">
          <div>
            <el-radio v-model="mergeForm.bitstate" label="1" size="medium" border>可用</el-radio>
            <el-radio v-model="mergeForm.bitstate" label="0" size="medium" border>禁用</el-radio>
          </div>
        </el-form-item>
        <!--<el-form-item label="用户头像" prop="headimage">-->
          <!--<el-upload v-model="mergeForm.headimage" class="upload-demo" :limit="2"-->
                     <!--:auto-upload="false" multiple>-->
            <!--<el-button size="small" type="primary">选择头像文件</el-button>-->
          <!--</el-upload>-->
        <!--</el-form-item>-->
      </el-form>
      <div slot="footer" class="dialog-footer" align="center">
        <el-button style="width: 20%;" type="primary" @click="onSubmitMergeForm()">确 定</el-button>
        <el-button style="width: 20%;" @click="dialogVisible=false">取 消</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    data: function () {
      //校验手机号是否合法
      var validatePass02 = (rule, value, callback) => {
        const reg = /^(((13[0-9]{1})|(15[0-9]{1})|(18[0-9]{1}))+\d{8})$/;
        if (value == null) {
          callback(new Error('请输入你的电话号码！'));
        } else if (!reg.test(this.mergeForm.phonenumber)) {
          callback(new Error('你输入的电话号码不合法!'));
        } else {
          callback();
        }
      };

      return {
        queryForm: {
          nickname: null, //用户名称
          current_page: 1, //当前显示页码数
          page_size: 10, //每页显示的最大行数
          total_count: 0 //数据的总记录数
        },
        queryForm2: {
          nickname: null //用户名称
        },
        result: [], //返回多个用户结果集
        result2: [], //返回所有用户-角色结果集
        dialogVisible: false, //用户添加显示框
        dialogTitle: '用户修改', //用户显示框的题目（左上方）
        labelPosition: 'left', //文字在文本框左侧显示
        mergeForm: {
          userid: null, //用户ID
          nickname: null, //用户账号
          userpassword: null, //用户密码
          headimage: null,//用户头像
          usertype: null,//用户类型
          phonenumber: null,//电话号码
          bitstate: null, //用户状态
        },
        user_flagName: null, //用户标识的名称
        mergeFormRules: { //修改用户表单验证
          nickname: [{
            required: true,
            message: '请输入用户账号',
            trigger: 'blur'
          },
            {
              min: 2,
              max: 10,
              message: '长度在 2 到 10 个字符',
              trigger: 'blur'
            }
          ],
          userpassword: [{
            required: true,
            message: '请输入用户密码',
            trigger: 'blur'
          },
            {
              min: 2,
              max: 16,
              message: '长度在 2 到 16 个字符',
              trigger: 'blur'
            }
          ],
          bitstate: [{
            required: true,
            message: '请选择用户状态',
            trigger: 'change'
          }],
          phonenumber: [{
            required: true,
            validator: validatePass02,
            trigger: 'blur'
          }],
          headimage: [{
            required: true,
            message: '请选择头像文件',
            trigger: 'blur'
          }],
          usertype: [{
            required: true,
            message: '请选择用户类型',
            trigger: 'change'
          }]
        }
      };
    },
    created: function () {
      this.search();
    },
    methods: {
      //编辑文本（行）
      handleEdit: function (row) {
        this.mergeForm.userid = row.userid;
        this.mergeForm.nickname = row.nickname;
        this.mergeForm.userpassword = row.userpassword;
        this.mergeForm.headimage = row.headimage;
        this.mergeForm.usertype = row.usertype.toString();
        this.mergeForm.bitstate = row.bitstate.toString();
        this.mergeForm.phonenumber = row.phonenumber;
        this.dialogVisible = true;
      },
      //删除文本（行）
      handleDelete: function (index, row) {
        this.$confirm('你确定要删除这条记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let url = this.axios.urls.user_delUser;
          this.axios.post(url, {
            userid: row.userid
          }).then((resp) => {
            this.$message({
              message: resp.data.message,
              type: 'success'
            });
            this.search();
          }).catch((error) => {
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
      //序列显示方法
      indexMethod(index) {
        return (this.queryForm.current_page - 1) * this.queryForm.page_size + (index + 1);
      },
      //表单提交(添加或者修改的使用)
      onSubmitMergeForm: function () {
        this.$refs['mergeForm'].validate((valid) => {
          if (false === valid) {
            return false;
          }
          let url = this.axios.urls.user_editUser;
          this.axios.post(url, this.mergeForm).then((resp) => {
            if (0 == resp.data.code) {
              this.$message({
                message: resp.data.message,
                type: 'success'
              });
              if (null == this.mergeForm.userid) {
                this.handleDialogClose();
              }
              this.search();
            } else {
              this.$message.error(resp.data.message);
            }

          }).catch((error) => {
          });
        });
      },
      //dialog对话框的关闭事件
      handleDialogClose: function () {
        this.$refs['mergeForm'].resetFields(); //清空验证信息
        this.doClearMergeForm();
      },
      //清空表单中的数据
      doClearMergeForm: function () {
        this.mergeForm.userid = null; //清空后台提交表单数据
        this.mergeForm.nickname = null;
        this.mergeForm.userpassword = null;
        this.mergeForm.headimage = null;
        this.mergeForm.usertype = null;
        this.mergeForm.phonenumber = null;
        this.mergeForm.bitstate = null;
      },
      //更改每页显示行数
      handleSizeChange(rows) {
        this.queryForm.page_size = rows;
        this.queryForm.current_page = 1;
        this.search();
      },
      //更改当前页码数
      handleCurrentChange(page) {
        this.queryForm.current_page = page;
        this.search();
      },
      //提交头像文件相关
      // handleRemove(file, fileList) {
      //   console.log(file, fileList);
      // },
      // handlePreview(file) {
      //   console.log(file);
      // },
      // handleExceed(files, fileList) {
      //   this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
      // },
      // beforeRemove(file, fileList) {
      //   return this.$confirm(`确定移除 ${file.name}？`);
      // },
      //搜索
      search: function () {
        if (this.queryForm.nickname != this.queryForm2.nickname) {
          if (this.queryForm.nickname != null && this.queryForm.nickname != '') {
            this.queryForm.current_page = 1;
          }
        }
        this.queryForm2.nickname == this.queryForm.nickname

        let url = this.axios.urls.user_selectAllUser;
        this.axios.post(url, this.queryForm).then((resp) => {
          this.result = resp.data.result;
          this.queryForm.total_count = resp.data.total;
        }).catch((error) => {
        });

        let url2 = this.axios.urls.SYS_ROLE_LIST;
        this.axios.post(url2, {}).then((resp) => {
          this.result2 = resp.data.result;
        }).catch((error) => {
        });

      }

    }
  }
</script>


<style scoped>
  .searchBox {
    margin-top: 20px;
  }

  .userRole {
    width: 100%;
  }

  .width01 {
    min-width: 200px;
  }
</style>
