<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>

      <!-- 表格 -->
        <tree-table :data="catalist" :columns="columns"
        :selection-type="false" :expand-type="false"
        show-index index-text="#" border
        :show-row-hover="false"
        class="treeTable">
            <!-- 是否有效 -->
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-error"
                v-if="scope.row.cat_daleted === false"
                style="color: red;"></i>
                <i class="el-icon-success" v-else
                style="color: lightgreen;"></i>
            </template>
            <!-- 排序 -->
            <template slot="order" slot-scope="scope">
                <el-tag  v-if="scope.row.cat_level === 0">一级</el-tag>
                <el-tag  v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
                <el-tag  v-else-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
            </template>
            <!-- 操作 -->
            <template slot="opt" slot-scope="">
                <el-button type="primary" icon="el-icon-edit"
                size="mini">
                编辑</el-button>
                <el-button type="danger" icon="el-icon-delete"
                size="mini">
                删除</el-button>
            </template>
        </tree-table>
      <!-- 分页区域 -->
      <span class="demonstration"></span>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormrules"
        ref="addCateFormref"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options 用来指定数据源 -->
          <!-- props 用啦指定配置对象 -->
          <el-cascader
            v-model="selectesKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            collapse-tags
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 查询数据条件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      catalist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义成模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          // 表示将当前列定义成模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列定义成模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分了的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类的等级，默认要添加的是1级分类
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCateFormrules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectesKeys: []
    }
  },
  created() {
    this.getCataList()
  },
  methods: {
    // 获取商品分类数据
    async getCataList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.catalist = res.data.result
      this.total = res.data.total
    },
    // 监听pagersize改变
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCataList()
    },
    // 监听pagenum改变
    handleCurrentChange(pagenum) {
      this.querInfo.pagenum = pagenum
      this.getCataList()
    },
    // 点击按钮，展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类的数据列表
      this.getParentCataList()
      // 展示出对话框
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCataList() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获去父级分类数据失败！')
      }
      console.log(res.data)
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChanged() {
      console.log(this.selectesKeys)
      if (this.selectesKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectesKeys[this.selectesKeys.length - 1]
        this.addCateForm.cat_level = this.selectesKeys.length
        return this.addCateForm.cat_level
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮，添加新的分类
    addCate () {
      this.$refs.addCateFormref.validate(async valid => {
        if (!valid) return valid
        const { data: res } = await this.$http.post('categories', this.addCateForm)

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCataList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateFormref.resetFields()
      this.selectesKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>
<style lang='less'>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
    width: 100%;
}
.el-cascader-panel{
    height: 200px;
}
</style>
