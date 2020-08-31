<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图区 -->
        <el-card>
            <!-- 提示区域 -->
            <el-alert
                title="添加商品信息"
                type="info" center show-scon :closable="false">
            </el-alert>
            <!-- 步骤条 -->
            <el-steps :space="200" :active="activeIndex - 0"
            align-center finish-status="success">
                <el-step title="基本信息"></el-step>
                <el-step title="商品参数"></el-step>
                <el-step title="商品属性"></el-step>
                <el-step title="商品图片"></el-step>
                <el-step title="商品内容"></el-step>
                <el-step title="完成"></el-step>
            </el-steps>

            <!-- tab栏区域 -->
            <el-form :model="addForm" :rules="addFormRules"
            ref="addFormRef"
            label-width="100px" label-position="left">
                <el-tabs :before-leave="beforeTabLeave"
                v-model="activeIndex" :tab-position="'left'"
                @tab-click="tabClicked"
                >
                    <el-tab-pane label="基本信息" name="0">
                        <el-form-item label="商品名称" prop="goods_name">
                            <el-input v-model="addForm.goods_name"></el-input>
                        </el-form-item>
                        <el-form-item label="商品价格" prop="goods_price">
                            <el-input v-model="addForm.goods_price"
                            type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品重量" prop="goods_weight">
                            <el-input v-model="addForm.goods_weight"
                            type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品数量" prop="goods_number">
                            <el-input v-model="addForm.goods_number"
                            type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品分类" prop="goods_cat">
                            <el-cascader
                            v-model="addForm.goods_cat"
                            :options="catelist"
                            :props="cateProps"
                            @change="handleChange"></el-cascader>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品参数" name="1">
                      <!-- 渲染表单的item项 -->
                      <el-form-item :label="item.attr_name" v-for="item in manyTabledata"
                      :key="item.attr_id">
                      <!-- 复选框组 -->
                      <el-checkbox-group v-model="item.attr_vals">
                        <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i"></el-checkbox>
                      </el-checkbox-group>
                      </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品属性" name="2">
                      <el-form-item :label="item.attr_name" v-for="item in onlyTableData"
                      :key="item.attr_id">
                      <el-input v-model="item.attr_vals"></el-input>
                      </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品图片" name="3">
                      <!-- action 图片上传的后天API地址 -->
                      <el-upload
                      action="https://api.naccl.top/vue/shop/api/private/v1/upload"
                      :on-preview="handlePreview"
                      :on-remove="handleRemove"
                      list-type="picture"
                      :headers="headerObj">
                      <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                    </el-tab-pane>
                    <el-tab-pane label="商品内容" name="4">
                      <!-- 富文本编辑器组件 -->
                      <quill-editor
                      v-model="addForm.goods_introduce"></quill-editor>
                      <!-- 添加商品的按钮 -->
                      <el-button type="primary" class="btnAdd"
                      @click="add">添加商品</el-button>
                    </el-tab-pane>
                </el-tabs>
            </el-form>
        </el-card>
    </div>
</template>
<script>
import _ from 'lodash'

export default {
  data() {
    return {
      activeIndex: '0',
      // 添加商品的表单数据对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        // 商品的详情描述
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价值', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请输入商品分类', trigger: 'blur' }
        ]
      },
      // 商品分类列表
      catelist: [],
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 商品所属的分类数组
      selectedOptions2: [],
      // 动态参数列表数据
      manyTabledata: [],
      // 静态参数列表数据
      onlyTableData: [],
      // 上传图片的URL地址
      uploadURL: 'https://api.naccl.top/vue/shop/api/private/v1/upload',
      // 图片上传组件header请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类数据失败')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 这是级联选择器中变化会触发这个函数
    handleChange () {
      console.log(this.addForm.goods_cat)
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforeTabLeave (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类！')
        return false
      }
    //   console.log('即将离开的标签页名字是：' + oldActiveName,
    //     '即将进入的标签页名字是：' + activeName)
    //   return false
    },
    async tabClicked () {
    //   console.log(this.activeIndex)
      if (this.activeIndex === '1') {
        // console.log('动态参数列表')
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取动态参数列表失败')
        }
        console.log(res.data)
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTabledata = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取静态参数列表失败')
        }
        console.log(res.data)
        this.onlyTableData = res.data
      }
    },
    // 处理图片预览效果
    handlePreview () {},
    // 处理移除图片的操作
    handleRemove () {},
    add () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        // 执行添加的业务逻辑
        // lodash   cloneDeep(obj)
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTabledata.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          form.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          form.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)

        // 发起请求，添加商品
        // 商品的名称，必须是唯一
        const { data: res } = await this.$http.post('goods', form)

        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败！')
        }
        this.$message.success('添加商品成功！')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>
<style lang='less'>
.el-steps {
    margin: 15px 0;
}
.el-checkbox {
    margin: 0 15px 0 0 !important;
}
.btnAdd {
    margin-top: 15px !important;
}
</style>
