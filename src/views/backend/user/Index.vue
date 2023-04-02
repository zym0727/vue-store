<template>
  <div>
    <my-table-page
        :loading="loading"
        :columns="columns"
        :table-data="tableData"
        :current-page.sync="currentPage"
        :page-size.sync="pageSize"
        :total="total"
        :page-change="doSearch">
      <template #table-search>
        <el-form :model="searchModel" :inline="true" label-width="100px">
          <el-form-item prop="userName" label="用户姓名">
            <el-input v-model.trim="searchModel.userName" placeholder="请输入用户姓名" @keyup.enter.native="doSearch"/>
          </el-form-item>
          <el-form-item prop="userPhoneNumber" label="手机号">
            <el-input v-model.trim="searchModel.userPhoneNumber" placeholder="请输入手机号" @keyup.enter.native="doSearch"/>
          </el-form-item>
          <el-form-item prop="isAdmin" label="是否管理员">
            <el-select v-model="searchModel.isAdmin" placeholder="请选择" :clearable="true" @change="doSearch">
              <el-option :value="1" label="是"/>
              <el-option :value="0" label="否"/>
            </el-select>
          </el-form-item>
          <el-form-item style="margin-left: 30px">
            <el-button type="primary" @click="doSearch">搜索</el-button>
          </el-form-item>
        </el-form>
      </template>
      <template #table-action>
        <el-button type="primary" style="margin-left: auto" @click="addUser">添加用户</el-button>
      </template>
      <template #isAdmin="{isAdmin}">
        <i :class="isAdmin ? 'el-icon-success' : 'el-icon-error'" :style="{color: isAdmin ? 'green' : 'red', fontSize: '20px'}"/>
      </template>
      <template #operate="row">
        <el-button type="text" @click="editUser(row)">编辑</el-button>
        <el-button type="text" @click="deleteUser(row)" :disabled="!!row.isAdmin">删除</el-button>
      </template>
    </my-table-page>

    <el-dialog
        class="add-user-dialog"
        :title="dialogTitle"
        :visible.sync="dialogVisible"
        width="600px"
        :modal="false"
        center>
      <div>
        <el-form ref="userForm" :model="formModel" :rules="rules" label-position="top">
          <el-form-item prop="userName" label="用户姓名">
            <el-input v-model.trim="formModel.userName" placeholder="请输入用户姓名"/>
          </el-form-item>
          <el-form-item prop="password" label="用户密码">
            <el-input v-model.trim="formModel.password" placeholder="请输入用户密码"/>
          </el-form-item>
          <el-form-item prop="userPhoneNumber" label="手机号">
            <el-input v-model.trim="formModel.userPhoneNumber" placeholder="请输入手机号" maxlength="11"/>
          </el-form-item>
          <el-form-item prop="address" label="地址">
            <el-input v-model.trim="formModel.address"  type="textarea" :rows="2" placeholder="请输入" maxlength="50"/>
          </el-form-item>
          <el-form-item prop="isAdmin" label="是否管理员">
            <el-select v-model="formModel.isAdmin" placeholder="请选择">
              <el-option :value="1" label="是"/>
              <el-option :value="0" label="否"/>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirm">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import MyTablePage from "@/components/MyTablePage.vue";
import {validateName, validatePass} from "@/components/MyLogin.vue";

export let validatePhone = (rule, value, callback) => {
  if ( !value || /^1\d{10}$/.test(value)) {
    return callback();
  } else {
    return callback(new Error("手机号为11位数字"));
  }
};

export default {
  name: "User",
  components: {MyTablePage},
  data() {
    return {
      loading: false,
      tableData: [],
      columns: [
        {prop: 'user_id', label: '用户id', 'min-width': '100px'},
        {prop: 'userName', label: '用户姓名', 'min-width': '100px'},
        {prop: 'password', label: '用户密码', 'min-width': '100px'},
        {prop: 'userPhoneNumber', label: '手机号', 'min-width': '100px'},
        {prop: 'address', label: '地址', 'min-width': '100px'},
        {prop: 'isAdmin', label: '是否管理员', 'min-width': '100px'},
        {prop: 'operate', label: '操作', width: '200px'}
      ],
      currentPage: 1,
      total: 0,
      pageSize: 10,
      searchModel: {
        userName: '',
        userPhoneNumber: '',
        isAdmin: ''
      },
      formModel: {
        userName: '',
        password: '',
        userPhoneNumber: '',
        address: '',
        isAdmin: 0,
      },
      dialogVisible: false,
      action: '添加',
      rules: {
        userName: [{required: true, message: '请输入用户姓名', trigger: "blur"}, { validator: validateName, trigger: "blur" }],
        password: [{required: true, message: '请输入用户密码', trigger: "blur"}, { validator: validatePass, trigger: "blur" }],
        userPhoneNumber: [{ validator: validatePhone, trigger: "blur" }]
      }
    }
  },
  created() {
    this.doSearch()
  },
  computed: {
    dialogTitle() {
      return `${this.action}用户`
    }
  },
  methods: {
    doSearch() {
      this.loading = true
      const params = {pageNo: this.currentPage, pageSize: this.pageSize, ...this.searchModel}
      this.$axios.get('/api/user/queryUserList', { params }).then(res => {
        this.tableData = res.data.result
        this.total = res.data.total
      }).finally(() => {
        this.loading = false
      })
    },
    addUser() {
      this.resetForm()
      this.action = '添加'
      this.dialogVisible = true
    },
    editUser(row) {
      this.resetForm()
      this.action = '修改'
      this.dialogVisible = true
      this.$nextTick(() => {
        this.formModel = {...row}
      })
    },
    resetForm() {
      const form = this.$refs.userForm
      form && form.resetFields()
    },
    deleteUser({userName, user_id}) {
      this.$confirmTip(`确认删除用户 ${userName} ?`).then(() => {
        this.$axios.post('/api/user/deleteUser', { user_id }).then(this.commonHandle)
      }).catch(() => {})
    },
    confirm() {
      this.$refs.userForm.validate(valid => {
        if(valid) {
          const data = {...this.formModel}
          if(this.action === '添加') {
            this.$axios.post('/api/user/addUser', data).then(this.commonHandle)
          } else {
            this.$axios.post('/api/user/updateUser', data).then(this.commonHandle)
          }
        }
      })
    },
    commonHandle(res) {
      if(res.data.code === '001') {
        this.dialogVisible = false
        this.notifySucceed(res.data.msg)
        this.doSearch()
      } else {
        this.notifyError(res.data.msg)
      }
    }
  }
}
</script>

<style>
.add-user-dialog .el-dialog__title {
  font-size: 24px;
  font-weight: 700;
}

.add-user-dialog .el-form-item__label {
  margin-top: 10px;
  padding: 0;
  font-weight: 700;
}
</style>