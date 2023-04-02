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
                    <el-form-item prop="product_id" label="商品id">
                        <el-input v-model.trim="searchModel.product_id" placeholder="请输入商品id" @keyup.enter.native="doSearch"/>
                    </el-form-item>
                    <el-form-item prop="product_name" label="商品名称">
                        <el-input v-model.trim="searchModel.product_name" placeholder="请输入商品名称" @keyup.enter.native="doSearch"/>
                    </el-form-item>
                    <el-form-item prop="category_id" label="商品类别">
                        <el-select v-model="searchModel.category_id" placeholder="请选择" :clearable="true" @change="doSearch">
                            <el-option v-for="({category_id, category_name}) in category " :key="category_id" :value="category_id" :label="category_name"/>
                        </el-select>
                    </el-form-item>
                    <el-form-item style="margin-left: 30px">
                        <el-button type="primary" @click="doSearch">搜索</el-button>
                    </el-form-item>
                </el-form>
            </template>
            <template #table-action>
                <el-button type="primary" style="margin-left: auto" @click="addProduct">添加商品</el-button>
            </template>
            <template #product_picture="{product_picture}">
                <el-image :src="$target + product_picture" :preview-src-list="[$target + product_picture]" alt="" class="row-product-img" :z-index="999999" @click="handleClickStop($event)"/>
            </template>
            <template #operate="row">
                <el-button type="text" @click="editProduct(row)">编辑</el-button>
                <el-button type="text" @click="deleteProduct(row)">删除</el-button>
            </template>
        </my-table-page>

        <el-dialog
            class="add-product-dialog"
            :title="dialogTitle"
            :visible.sync="dialogVisible"
            width="600px"
            :modal="false"
            center>
            <el-form ref="productForm" :model="formModel" :rules="rules" label-position="right" label-width="120px">
                <el-form-item prop="product_name" label="商品名称">
                    <el-input v-model.trim="formModel.product_name" placeholder="请输入商品名称" style="width: 400px"/>
                </el-form-item>
                <el-form-item prop="category_id" label="商品类别">
                    <el-select v-model="formModel.category_id" placeholder="请选择" :clearable="true"  style="width: 400px">
                        <el-option v-for="({category_id, category_name}) in category " :key="category_id" :value="category_id" :label="category_name"/>
                    </el-select>
                </el-form-item>
                <el-form-item prop="product_title" label="商品标题">
                    <el-input v-model.trim="formModel.product_title" placeholder="请输入商品标题" maxlength="30"  style="width: 400px"/>
                </el-form-item>
                <el-form-item prop="product_intro" label="商品介绍">
                    <el-input  v-model.trim="formModel.product_intro" type="textarea" :rows="3" placeholder="请输入"  style="width: 400px"/>
                </el-form-item>
                <el-form-item prop="files" label="商品图片">
                    <el-upload
                        :file-list="formModel.files"
                        :auto-upload="false"
                        :on-change="onchange"
                        :on-remove="onRemove"
                        :limit="7"
                        accept=".png,.jpg"
                        list-type="picture"
                        action="">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过2M,  第一张为主商品图，上传2-7张</div>
                    </el-upload>
                </el-form-item>
                <el-form-item prop="product_price" label="商品价格">
                    <el-input v-model.trim="formModel.product_price" placeholder="请输入商品价格" style="width: 400px"/>
                </el-form-item>
                <el-form-item prop="product_selling_price" label="商品降价价格">
                    <el-input v-model.trim="formModel.product_selling_price" placeholder="请输入商品降价价格" style="width: 400px"/>
                </el-form-item>
                <el-form-item prop="product_sales" label="商品销量">
                    <el-input v-model.trim="formModel.product_sales" placeholder="请输入商品销量" style="width: 400px"/>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirm">确 定</el-button>
      </span>
        </el-dialog>
    </div>
</template>

<script>
import MyTablePage from "@/components/MyTablePage.vue";

export const validatePrice = (rule, value, callback) => {
    if (/^(?!0{2,})\d+(\.\d{1,2})?$/.test(value)) {
        return callback();
    } else {
        return callback(new Error("请输入正确的价格(最多两位小数)"));
    }
};

export const validateNaturalNum = (rule, value, callback) => {
    if (/^(?!0{2,})\d+$/.test(value)) {
        return callback();
    } else {
        return callback(new Error("请输入自然数"));
    }
};

export default {
    name: "BackendProduct",
    components: {MyTablePage},
    data() {
        return {
            loading: false,
            tableData: [],
            columns: [
                {prop: 'product_id', label: '商品id', 'min-width': '100px'},
                {prop: 'product_name', label: '商品名称', 'min-width': '200px'},
                {prop: 'category_name', label: '商品类别', 'min-width': '100px'},
                {prop: 'product_title', label: '商品标题', 'min-width': '100px', 'show-overflow-tooltip': true},
                {prop: 'product_intro', label: '商品介绍', 'min-width': '100px','show-overflow-tooltip': true},
                {prop: 'product_picture', label: '商品缩略图', 'min-width': '100px'},
                {prop: 'product_price', label: '商品价格', 'min-width': '100px'},
                {prop: 'product_selling_price', label: '商品降价价格', 'min-width': '100px'},
                {prop: 'product_sales', label: '商品销量', 'min-width': '100px'},
                {prop: 'operate', label: '操作', width: '200px'}
            ],
            currentPage: 1,
            total: 0,
            pageSize: 10,
            searchModel: {
                product_id: '',
                product_name: '',
                category_id: ''
            },
            formModel: {
                product_name: '',
                category_id: '',
                product_title: '',
                product_intro: '',
                files: [],
                product_price: '',
                product_selling_price: '',
                product_sales: 0
            },
            dialogVisible: false,
            action: '添加',
            rules: {
                product_name: [{required: true, message: '请输入商品名称', trigger: "blur"}],
                category_id: [{required: true, message: '请选择商品类别', trigger: "change"}],
                product_title: [{required: true, message: '请输入商品标题', trigger: "blur"}],
                product_intro: [{required: true, message: '请输入商品介绍', trigger: "blur"}],
                files:[{required: true, message: '请上传商品图片', trigger: "change"}],
                product_price: [{required: true, message: '请输入商品价格', trigger: "blur"}, { validator: validatePrice, trigger: blur }],
                product_selling_price: [{required: true, message: '请输入商品降价价格', trigger: "blur"}, { validator: validatePrice, trigger: blur }],
                product_sales: [{required: true, message: '请输入商品销量', trigger: "blur"}, { validator: validateNaturalNum, trigger: blur }]
            },
            category: [],
            recordEditPictureIds: [],
            chooseRow:  {}
        }
    },
    created() {
        this.getAllCategory()
        this.doSearch()
    },
    computed: {
        dialogTitle() {
            return `${this.action}商品`
        }
    },
    methods: {
        getAllCategory() {
          this.$axios.post('/api/product/getCategory').then(res => {
              this.category = res.data.category
          })
        },
        doSearch() {
            this.loading = true
            const params = {pageNo: this.currentPage, pageSize: this.pageSize, ...this.searchModel}
            this.$axios.get('/api/user/product/queryProductList', { params }).then(res => {
                this.tableData = res.data.result
                this.total = res.data.total
            }).finally(() => {
                this.loading = false
            })
        },
        addProduct() {
            this.resetForm()
            this.action = '添加'
            this.dialogVisible = true
        },
        editProduct(row) {
            this.resetForm()
            this.action = '修改'
            this.dialogVisible = true
            this.recordEditPictureIds = []
            this.chooseRow =row
            this.$axios.post('/api/product/getDetailsPicture',{productID : row.product_id}).then(res => {
                this.formModel ={
                    product_name: row.product_name,
                    category_id: row.category_id,
                    product_title: row.product_title,
                    product_intro: row.product_intro,
                    product_price: row.product_price,
                    product_selling_price: row.product_selling_price,
                    product_sales: row.product_sales
                }
                const pictures = res.data.ProductPicture
                this.recordEditPictureIds = pictures.map(item => item.id)
                const files = [{product_id : row.product_id, product_picture: row.product_picture}, ...pictures]
                files.forEach((item, index) => {
                    item.name = index === 0 ? '商品主图' : `商品详情图${index}`
                    item.url = this.$target + item.product_picture
                })
                this.formModel.files = files
            })
        },
        resetForm() {
            const form = this.$refs.productForm
            form && form.resetFields()
        },
        deleteProduct({product_name, product_id}) {
            this.$confirmTip(`确认删除商品 ${product_name} ?`).then(() => {
                this.$axios.post('/api/user/product/deleteProduct', { product_id }).then(this.commonHandle)
            }).catch(() => {})
        },
        confirm() {
            this.$refs.productForm.validate(valid => {
                if(valid) {
                    if(this.formModel.files.length < 2 || this.formModel.files.length > 7) {
                        this.notifyError('需上传2-7张图片，1张商品主图和1-6张商品详情图')
                        return
                    }
                    if(Number(this.formModel.product_selling_price) > Number(this.formModel.product_price)) {
                        this.notifyError('降价价格不可能比原始价格高')
                        return
                    }
                    const formData = new FormData()
                    for(let key of Object.keys(this.formModel)){
                        if(key !== 'files') {
                            formData.append(key, this.formModel[key])
                        }
                    }
                    if(this.action === '添加') {
                      this.addProductRequest(formData)
                    } else {
                        this.uploadProductRequest(formData)
                    }
                }
            })
        },
        addProductRequest(formData) {
            const formFiles = this.formModel.files
            for (let i = 0; i < formFiles.length; i++) {
                formData.append('files', formFiles[i].raw);
            }
            this.$axios.post('/api/user/product/backAddProduct', formData).then(this.commonHandle)
        },
        uploadProductRequest(formData) {
            const formFiles = this.formModel.files
            const rawFiles = formFiles.filter(item => item.raw).map(item => item.raw)
            for (let i = 0; i < rawFiles.length; i++) {
                formData.append('files', rawFiles[i]);
            }
            let product_picture = this.chooseRow.product_picture
            if((formFiles.length > 0) && (formFiles[0].product_picture !== product_picture)) {
                product_picture = formFiles[0].product_picture || ''
            }
            formData.append('product_picture', product_picture)
            const deletePictureIds = []
            const tempFiles = formFiles.length > 1 ? formFiles.slice(1) : []
            this.recordEditPictureIds.forEach(id => {
                const file = tempFiles.find(item => item.id === id)
                if(!file) {
                    deletePictureIds.push(id)
                }
            })
            formData.append('deletePictureIds', deletePictureIds.join())
            formData.append('product_id', this.chooseRow.product_id)
            this.$axios.post('/api/user/product/updateProduct', formData).then(this.commonHandle)
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
        onchange(file, files){
            const index = files.findIndex(item => item.uid === file.uid)
            file.name = index === 0 ? '商品主图' : `商品详情图${index}`
            const temp = files.slice()
            temp.splice(index, 1)
            if(file.size > 2 * 1024 * 1024){   //验证文件大小
                this.$message.info(`上传文件不能大于2M`);
                this.formModel.files = temp
            } else if(['image/png', 'image/jpg', 'image/jpeg'].indexOf(file.raw.type) === -1){   //验证文件类型
                this.$message.info(`只能上传.png,.jpg文件`);
                this.formModel.files = temp
            } else {
                this.formModel.files = files
            }
        },
        onRemove(file, files) {
            files.forEach((item, index) => {
               item.name = index === 0 ? '商品主图' : `商品详情图${index}`
           })
            this.formModel.files = files
        },
        handleClickStop(event) {
            this.$nextTick(() => {
                const mask = event.target.nextElementSibling
                let domImageView = mask.querySelector(".el-image-viewer__mask"); // 获取遮罩层dom
                if (domImageView) {
                    domImageView.onclick = function () {
                        mask.querySelector(".el-image-viewer__close").click();
                    }
                }
            });
        }
    }
}
</script>

<style>
.add-product-dialog .el-dialog__title {
    font-size: 24px;
    font-weight: 700;
}

.add-product-dialog .el-form-item__label {
    font-weight: 700;
}

.el-image-viewer__close {
    color: white;
}
</style>

<style scoped>
.row-product-img {
    width: 36px;
    height: 36px;
    cursor: pointer;
}
</style>
