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
          <el-form-item prop="order_id" label="订单编号">
            <el-input v-model.trim="searchModel.order_id" placeholder="请输入订单编号" @keyup.enter.native="doSearch"/>
          </el-form-item>
          <el-form-item prop="userName" label="用户名称">
            <el-input v-model.trim="searchModel.userName" placeholder="请输入用户名称" @keyup.enter.native="doSearch"/>
          </el-form-item>
          <el-form-item style="margin-left: 30px">
            <el-button type="primary" @click="doSearch">搜索</el-button>
          </el-form-item>
        </el-form>
      </template>
      <template #table-action>
        <el-button type="primary" style="margin-left: auto" @click="addOrder">添加订单</el-button>
      </template>
      <template #order_time="{order_time}">
        {{  order_time |  dateFormat}}
      </template>
      <template #productsDetail="{productsDetail}">
        <div v-for="({name, num, price}) in productsDetail" :key="name" class="product-line">
          <span>
            <el-tag type="info">{{ name }}</el-tag>
            <el-tag type="success" style="margin-left: 20px">{{ num }} 个</el-tag>
            <el-tag type="warning"  style="margin-left: 20px">{{price}} 元/个</el-tag>
          </span>
        </div>
      </template>
      <template #price="{price}">
        {{  Number(price.toFixed(2))}}
      </template>
        <template #order_status="{order_status}">
            <span :class="order_status === 0 ? 'unSend': 'send'">●</span>
            {{  order_status === 0 ? '未发货' : '已发货'}}
        </template>
      <template #operate="row">
        <el-button type="text" @click="editOrder(row)">编辑</el-button>
        <el-button type="text" @click="seeDetail(row)">详情</el-button>
        <el-button type="text" @click="deleteOrder(row)">删除</el-button>
      </template>
    </my-table-page>

    <el-dialog
        class="add-order-dialog"
        :title="dialogTitle"
        :visible.sync="dialogVisible"
        width="600px"
        :modal="false"
        center>
      <div>
        <el-form ref="orderForm" :model="formModel" :rules="rules" label-position="right" label-width="100px">
          <el-form-item v-if="action === '修改'" prop="order_id" label="订单编号">
            <el-input v-model.trim="formModel.order_id" disabled/>
          </el-form-item>
          <el-form-item prop="orderUserId" label="用户姓名">
            <el-select v-model="formModel.orderUserId" filterable placeholder="请选择">
              <el-option
                  v-for="item in users"
                  :key="item.user_id"
                  :label="item.userName"
                  :value="item.user_id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="productsDetail" label="商品详情">
            <div v-for="(ele, index) in formModel.productsDetail" :key="ele.key" class="dialog-product-line">
              <el-select v-model="ele.productID" filterable placeholder="请选择" @change="changeProduct(ele, $event)" style="width: 200px">
                <el-option
                    v-for="item in products"
                    :key="item.product_id"
                    :label="item.product_name"
                    :value="item.product_id">
                </el-option>
              </el-select>
              <el-input-number v-model="ele.num" :min="1" :max="100" style="width: 140px; margin-left: 10px"/>
              <i v-if="index === formModel.productsDetail.length -1" class="el-icon-circle-plus add-icon" @click="addProduct"/>
            </div>
          </el-form-item>
          <el-form-item prop="order_status" label="发货状态">
              <el-select v-model="formModel.order_status" filterable placeholder="请选择">
                  <el-option label="未发货" :value="0"/>
                  <el-option label="已发货" :value="1"/>
              </el-select>
          </el-form-item>
          <el-form-item prop="order_time" label="购买时间">
            <el-date-picker
                v-model="formModel.order_time"
                type="datetime"
                placeholder="选择日期时间">
            </el-date-picker>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirm">确 定</el-button>
      </span>
    </el-dialog>

      <el-drawer
        title="订单详情"
        :modal="false"
        :visible.sync="drawVisible">
          <div class="order-detail-container">
              <div class="detail-line">
                  <span>订单编号：</span>
                  <span>{{detailData.order_id || '-'}}</span>
              </div>
              <div class="detail-line">
                  <span>用户姓名：</span>
                  <span>{{detailData.userName || '-'}}</span>
              </div>
              <div class="detail-line">
                  <span>购买商品详情：</span>
                  <span>
                       <div v-for="({name, num, price}) in detailData.productsDetail" :key="name" class="detail-product-line">
                          <span>
                            <el-tag type="info">{{ name }}</el-tag>
                            <el-tag type="success" style="margin-left: 20px">{{ num }} 个</el-tag>
                            <el-tag type="warning"  style="margin-left: 20px">{{price}} 元/个</el-tag>
                          </span>
                        </div>
                  </span>
              </div>
              <div class="detail-line">
                  <span>总价：</span>
                  <span>{{detailData.price || '-'}}</span>
              </div>
              <div class="detail-line">
                  <span>发货状态：</span>
                  <span>{{detailData.order_status === 0 ? '未发货' : '已发货'}}</span>
              </div>
              <div class="detail-line">
                  <span>购买时间：</span>
                  <span>{{detailData.order_time || '-'}}</span>
              </div>
              <div class="detail-line">
                  <span>收货人手机号码：</span>
                  <span>{{detailData.userPhoneNumber || '-'}}</span>
              </div>
              <div class="detail-line">
                  <span>收货人具体地址：</span>
                  <span>{{detailData.address || '-'}}</span>
              </div>
              <div class="detail-line">
                  <span>配送方式：</span>
                  <span>包邮</span>
              </div>
              <div class="detail-line">
                  <span>发票：</span>
                  <span>电子发票</span>
              </div>
          </div>
      </el-drawer>
  </div>
</template>

<script>
import MyTablePage from "@/components/MyTablePage.vue";

export default {
  name: "BackendOrder",
  components: {MyTablePage},
  data() {
    return {
      loading: false,
      tableData: [],
      columns: [
        {prop: 'order_id', label: '订单编号', 'min-width': '100px'},
        {prop: 'userName', label: '用户姓名', 'min-width': '100px'},
        {prop: 'productsDetail', label: '购买商品详情', 'min-width': '300px'},
        {prop: 'price', label: '总价', 'min-width': '100px'},
        {prop: 'order_status', label: '发货状态', 'min-width': '100px'},
        {prop: 'order_time', label: '购买时间', 'min-width': '100px'},
        {prop: 'operate', label: '操作', width: '200px'}
      ],
      currentPage: 1,
      total: 0,
      pageSize: 10,
      searchModel: {
        order_id: '',
        userName: ''
      },
      formModel: {
        order_id: '',
        orderUserId: '',
        productsDetail: [{key: 0}],
        order_status: 0,
        order_time: '',
      },
      dialogVisible: false,
      action: '添加',
      users: [],
      products: [],
      rules: {
        orderUserId: [{required: true, message: '请选择用户', trigger: "change"}],
        order_time: [{required: true, message: '请选择购买时间', trigger: "change"}]
      },
      key: 9,
     detailData: {},
     drawVisible: false
    }
  },
  created() {
    this.doSearch()
    this.getUsers()
    this.getProducts()
  },
  computed: {
    dialogTitle() {
      return `${this.action}订单`
    }
  },
  methods: {
    doSearch() {
      this.loading = true
      const params = {pageNo: this.currentPage, pageSize: this.pageSize, ...this.searchModel}
      this.$axios.get('/api/user/order/queryOrderList', { params }).then(res => {
        this.tableData = res.data.result
        this.total = res.data.total
      }).finally(() => {
        this.loading = false
      })
    },
    getUsers() {
      this.$axios.get('/api/user/getNoPagedUsers').then(res => {
        this.users = res.data.result
      })
    },
    getProducts() {
      this.$axios.get('/api/user/product/getNoPagedProducts').then(res => {
        this.products = res.data.result
      })
    },
    addOrder() {
      this.resetForm()
      this.action = '添加'
      this.dialogVisible = true
    },
    editOrder(row) {
      this.resetForm()
      this.action = '修改'
      this.dialogVisible = true
      this.$nextTick(() => {
        this.formModel = {...row}
        this.formModel.productsDetail = [...this.formModel.productsDetail]
      })
    },
    seeDetail(row) {
      this.drawVisible = true
      this.detailData = row
      this.$axios.post('/api/user/getUserById', {user_id: row.orderUserId}).then(res => {
          this.detailData = {...row, ...res.data.user}
      })
    },
    resetForm() {
      const form = this.$refs.orderForm
      form && form.resetFields()
      this.formModel.productsDetail = [{key: this.key++}]
    },
    deleteOrder({order_id}) {
      this.$confirmTip(`确认删除订单 ${order_id}?`).then(() => {
        this.$axios.post('/api/user/order/deleteOrder', { order_id }).then(this.commonHandle)
      }).catch(() => {})
    },
    confirm() {
     this.$refs.orderForm.validate(valid => {
       if(valid) {
         const {order_id, orderUserId, productsDetail, order_status, order_time} = this.formModel
         const data = {
           user_id: this.$store.getters.getUser.user_id,
           orderUserId,
           order_status,
           order_time: new Date(order_time).getTime()
         }
         data.products = productsDetail.filter(item => item.productID !==undefined && item.num !==undefined && item.price !==undefined).slice()
         if(!data.products.length) {
           this.notifyError('请至少选一个商品')
           return
         }
         if(this.action === '添加') {
           this.$axios.post('/api/user/order/backAddOrder', data).then(this.commonHandle)
         } else {
           data.order_id = order_id
           this.$axios.post('/api/user/order/updateOrder', data).then(this.commonHandle)
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
    },
    changeProduct(item, id) {
      const product = this.products.find(ele => ele.product_id === id)
      if(product) {
        item.price = product.product_selling_price
      } else {
        item.price = 0
      }
    },
    addProduct() {
      this.formModel.productsDetail.push({key: this.key++})
    }
  }
}
</script>

<style>
.add-order-dialog .el-dialog__title {
  font-size: 24px;
  font-weight: 700;
}

.add-order-dialog .el-form {
 padding-left: 70px;
}

.add-order-dialog .el-form-item__label {
  font-weight: 700;
}

.unSend {
    margin-right: 8px;
    color: #E65D6E;
}

.send {
    margin-right: 8px;
    color: #67c23a;
}
</style>

<style scoped>
.product-line:not(:last-child){
  margin-bottom: 20px;
}

.dialog-product-line {
  display: flex;
}

.dialog-product-line:not(:last-child){
  margin-bottom: 8px;
}

.add-icon {
  margin-left: 10px;
  color: #1ab91d;
  align-self: center;
  font-size: 16px;
}

.order-detail-container {
    padding: 0 50px 20px 20px;
}

.order-detail-container .detail-line {
    display: flex;
    line-height: 60px;
    font-size: 18px;
}

.order-detail-container .detail-line >span:first-child {
    width: 200px;
    font-weight: 700;
    margin-right: 50px;
}

.order-detail-container .detail-line .detail-product-line {
    display: flex;
}
</style>
