<template>
  <div>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item style="font-size: 24px;">系统账户流水查询</el-breadcrumb-item>
      </el-breadcrumb>

      <div style="text-align: right">
        <el-button-group>
          <el-button type="primary" icon="el-icon-question" @click="wantHelp()">帮助</el-button>
        </el-button-group>
        <!--<el-button-group>-->
        <!--<el-button type="primary" icon="el-icon-plus" @click="dialogVisible=true">添加</el-button>-->
        <!--</el-button-group>-->
      </div>
      <br/>

      <div>
        <el-form>
          <el-form-item>
            <el-input placeholder="请输入账户名" v-model="queryForm.accountName" style="width: 300px;" clearable></el-input>
            <el-select v-model="queryForm.actiontype" placeholder="请选择类型" key="-1" value="-1" clearable>
              <!--<el-option key="-1" index="-1" label="所有产类型" value=""></el-option>-->
              <el-option v-for="item in types" :label="item.label" :value="item.value" :key="item.value"></el-option>
            </el-select>
            <el-date-picker v-model="queryForm.tradetimeName" clearable align="right" type="date"
                            placeholder="选择交易日期" value-format="yyyy-MM-dd"></el-date-picker>
            <el-select v-model="queryForm.amount00" clearable placeholder="请选择交易价格段">
              <!--<el-option key="e" index="r" label="" value=""></el-option>-->
              <el-option v-for="(item,index) in this.list" :value="item" :index="index" :key="index" :label="item"
                         :disabled="item.disabled"></el-option>
            </el-select>

            <el-button type="primary" icon="el-icon-search" @click="loadAll()">搜索</el-button>
          </el-form-item>
        </el-form>
        <div>
          <p style="text-align: left;">平台账户当前金额:{{this.sumMoney}}￥</p>
        </div>
      </div>

      <!-- 数据表格 -->
      <el-table :data="dataResult" border style="width: 100%">
        <el-table-column type="index" label="序" :index="indexMethod" min-width="30" align="center"></el-table-column>

        <el-table-column prop="accountName" label="交易用户" width="200"></el-table-column>

        <el-table-column prop="amount" label="交易金额" width="200"></el-table-column>

        <el-table-column prop="tradeTimeCN" label="交易时间" width="200"></el-table-column>

        <!--<el-table-column prop="nummoney" label="平台账户当前金额" width="200"></el-table-column>-->

        <el-table-column prop="actionTypeCN" label="交易类型" width="200"></el-table-column>

        相关操作
        <el-table-column label="操作">
          <template label="操作" min-width="10px" slot-scope="scope">
            <el-button size="mini" type="warning" @click="handleByEdit(scope.row)" class="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" @click="handleByDel(scope.row)" class="el-icon-delete">删除</el-button>
            <!--<el-button size="mini" type="success" @click="handleByDetails(scope.row)" class="el-icon-info">详情-->
            <!--</el-button>-->
          </template>
        </el-table-column>

      </el-table>


      <!--完整分页-->
      <div class="block" style="padding-top: 20px; text-align: left;">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                       :current-page="queryForm.current_page"
                       :page-sizes="[10, 20, 30, 40]" :page-size="queryForm.page_size"
                       layout="total, sizes, prev, pager, next, jumper"
                       :total="queryForm.total_count">
        </el-pagination>
      </div>

      <!-- 添加和编辑操作 -->
      <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" width="30%" @close="handleDialogClose">
        <el-form :model="mergeForm" :rules="mergeFormRules" ref="mergeForm" :label-position="labelPosition"
                 label-width="80px">
          <el-form-item label="用户名称" prop="accountName">
            <el-input :readonly="diss" v-model="mergeForm.accountName"></el-input>
          </el-form-item>
          <el-form-item label="交易类型" prop="actiontype">
            <el-select v-model="mergeForm.actiontype" filterable placeholder="请选择">
              <el-option v-for="item in types" :label="item.label" :value="item.value" :key="item.value"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="交易金额" prop="amount">
            <el-input v-model="mergeForm.amount"></el-input>
          </el-form-item>
          <!--<el-form-item label="平台金额" prop="nummoney">-->
          <!--<el-input v-model="mergeForm.nummoney"></el-input>-->
          <!--</el-form-item>-->
          <el-form-item label="日期" prop="tradeTimeCN">
            <el-input :readonly="diss" v-model="mergeForm.tradeTimeCN"></el-input>
            <!--<el-date-picker-->
            <!--v-model="value2"-->
            <!--align="right"-->
            <!--type="date"-->
            <!--placeholder="选择日期"-->
            <!--:picker-options="pickerOptions">-->
            <!--</el-date-picker>-->
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible=false">取 消</el-button>
          <el-button type="primary" @click="onSubmitMergeForm()">确 定</el-button>
        </div>
      </el-dialog>

    </div>
  </div>

</template>

<script>
  // import moment from 'moment'

  export default {
    data: function () {
      return {
        // pickerOptions: {
        //   disabledDate(time) {
        //     return time.getTime() > Date.now();
        //   },
        //   shortcuts: [{
        //     text: '今天',
        //     onClick(picker) {
        //       picker.$emit('pick', new Date());
        //     }
        //   }, {
        //     text: '昨天',
        //     onClick(picker) {
        //       const date = new Date();
        //       date.setTime(date.getTime() - 3600 * 1000 * 24);
        //       picker.$emit('pick', date);
        //     }
        //   }, {
        //     text: '一周前',
        //     onClick(picker) {
        //       const date = new Date();
        //       date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
        //       picker.$emit('pick', date);
        //     }
        //   }]
        // },
        // value2: '',
        diss: null,
        dialogVisible: false,
        dialogTitle: '系统流水添加',
        labelPosition: 'left',
        dataResult: [],
        sumMoney: 0,
        queryForm: {
          current_page: 1, //当前页码数
          page_size: 10, //每页显示的最大记录数
          total_count: 0, //总记录数
          amount00: null,
          min: null,
          max: null
        },
        queryForm1: {
          accountName: null,
          amount00: null,
          tradetimeName: null,
          actiontype: null
        },

        amount: '',
        amounts: '',
        list: ["1-5000", "5001-10000", "10001-20000", "20001-50000", "50001-100000", "100001-200000", "200001-100000000"],
        mergeForm: {
          amount: null,
          tradetime: null,
          tradeTimeCN: null,
          nummoney: null,
          actiontype: null,
          actionTypeCN: null,
          accountId: null,
          accountName: null,
          accountflowid: null
        },
        types: [{
          value: '1',
          label: '充值'
        }, {
          value: '2',
          label: '提现'
        }, {
          value: '3',
          label: '还款'
        }, {
          value: '4',
          label: '获利'
        }, {
          value: '5',
          label: '逾期还款'
        }, {
          value: '6',
          label: '资金撤销'
        }],
        mergeFormRules: {
          // accountName: [{
          //   required: true,
          //   message: '请输入账号',
          //   trigger: 'blur'
          // }],
          // nummoney: [{
          //   required: true,
          //   message: '请输入服务费用',
          //   trigger: 'blur'
          // }],
          // actiontype: [{
          //   required: true,
          //   message: '请选择交易类型',
          //   trigger: 'blur'
          // }],
        }
      }
    },
    created: function () {
      // this.type();
      console.log(this.amounts);
      this.loadNummoney();
      this.loadAll();
    },
    methods: {
      // dateFormat: function (date) {
      //   var date = row[column.property];
      //   if (date == undefined) {
      //     return "";
      //   }
      //   return moment(date).format("YYYY-MM-DD HH:mm:ss");
      // },
      //查询的方法
      loadAll: function () {
        var min = null;
        var max = null;
        let str = this.queryForm.amount00;
        if (str != null && str.length > 0) {
          let str1 = str.split("-");
          min = str1[0];
          max = str1[1];
        }
        this.queryForm.min = min;
        this.queryForm.max = max;

        if (this.queryForm.actiontype == '') {
          this.queryForm.actiontype = null;
        }
        // console.log("最小值：", min);
        // console.log("最大值：", max);
        //将参数传到后台

        if (this.queryForm.accountName != this.queryForm1.accountName || this.queryForm1.actiontype != this.queryForm.actiontype || this.queryForm.tradetime != this.queryForm1.tradetime) {
          if (this.queryForm.accountName != null && this.queryForm.accountName != '') {
            this.queryForm.current_page = 1;
          }
          if (this.queryForm.actiontype != null && this.queryForm.actiontype != '') {
            this.queryForm.current_page = 1;
          }
          if (this.queryForm.tradetimeName != null && this.queryForm.tradetimeName != '') {
            this.queryForm.current_page = 1;
          }
        }
        this.queryForm1.accountName = this.queryForm.accountName;
        this.queryForm1.actiontype = this.queryForm.actiontype;
        this.queryForm1.tradetimeName = this.queryForm.tradetimeName;

        let url = this.axios.urls['QUERY_SYS_ACCOUNT_WATER'];

        if (this.queryForm.tradetimeName != null && this.queryForm.tradetimeName != '') {
          this.queryForm.tradetimeName = this.queryForm.tradetimeName.toString();
        }

        this.axios.post(url, this.queryForm).then((response) => {
          this.dataResult = response.data.result;
          this.queryForm.total_count = response.data.total;
        }).catch((response) => {
          console.log(Error);
        });
      },

      //账户当前的金额
      loadNummoney: function () {
        let url = this.axios.urls['QUERY_SYS_ACCOUNT_WATER'];
        this.axios.post(url, this.queryForm).then((response) => {
          this.results = response.data.result;
          this.queryForm.total_count = response.data.total;
          for (let i = 0; i < this.results.length; i++) {
            this.sumMoney += this.results[i].nummoney;
          }
        }).catch((response) => {
          console.log(Error);
        });
      },

      //绑定数据库的操作类型
      // type: function () {
      // let url = this.axios.urls['QUERY_TYPES'];
      // this.axios.post(url, this.queryForm).then((response) => {
      //   this.dataResult = response.data.result;
      // }).catch((response) => {
      //   console.log(Error);
      // });
      // },

      wantHelp: function () {
        this.$message({
          showClose: true,
          message: '这是系统账户流水查询界面，可进行查询、删除、修改等操作!'
        });
      },

      handleByDel: function (row) {
        this.$confirm('你确定要删除这条记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let url = this.axios.urls.DEL_SYS_ACCOUNT_FLOW;
          this.axios.post(url, {
            accountflowid: row.accountflowid
          }).then((response) => {
            this.$message({
              message: response.data.message,
              type: 'success'
            });
            this.loadAll();
          }).catch((error) => {
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },

      // handleByDetails: function () {
      //
      // },

      //提交表单
      onSubmitMergeForm: function () {
        this.$refs['mergeForm'].validate((valid) => {
          if (false === valid) {
            return false;
          }
          let url = this.axios.urls['EDIT_SYS_ACCOUNT_FLOW'];
          this.axios.post(url, {
              accountflowid: this.mergeForm.accountflowid,
              amount: this.mergeForm.amount,
              // servicefee: this.mergeForm.servicefee,
              actiontype: this.mergeForm.actiontype
            }
          ).then((response) => {
            this.$message({
              message: response.data.message,
              type: 'success'
            });
            if (null == this.mergeForm.accountflowid) {
              this.handleDialogClose();
            }
            this.loadAll();
          }).catch((error) => {
          });
        });
        // this.queryForm1.accountName = this.queryForm.accountName;
      },
      //修改
      handleByEdit: function (row) {
        this.mergeForm.amount = row.amount;
        this.mergeForm.actiontype = row.actiontype.toString();
        // this.mergeForm.actiontype = row.actionTypeCN;
        this.mergeForm.accountId = row.accountId;
        this.mergeForm.nummoney = row.nummoney;
        this.mergeForm.accountName = row.accountName;
        this.mergeForm.tradeTimeCN = row.tradeTimeCN;
        this.mergeForm.accountflowid = row.accountflowid;
        this.dialogVisible = true;
        this.diss = true;
        this.dialogTitle = '系统流水信息修改';
      },
      //更改每页显示行数
      handleSizeChange(total) {
        this.queryForm.page_size = total;
        this.queryForm.current_page = 1;
        this.loadAll();
      },
      //更改当前页码数
      handleCurrentChange(page_size) {
        this.queryForm.current_page = page_size;
        this.loadAll();
      },
      //dialog对话框的关闭事件
      handleDialogClose: function () {
        this.$refs['mergeForm'].resetFields(); //清空验证信息
        this.doClearMergeForm(); //调用方法清除表单中的数据
      },
      //清空表单中的数据
      doClearMergeForm: function () {
        // 清空后台提交表单数据
        this.mergeForm.accountId = null,
          this.mergeForm.actiontype = null,
          this.mergeForm.nummoney = null,
          this.mergeForm.amount = null,
          this.mergeForm.tradetime = null,
          // this.$refs['mergeForm'].resetFields(); //清空验证信息
          this.dialogTitle = '系统流水添加';
      },
      //序列显示方法
      indexMethod(index) {
        return (this.queryForm.current_page - 1) * this.queryForm.page_size + (index + 1);
      },

    },

  }
</script>

<style scoped>

</style>
