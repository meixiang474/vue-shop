<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button
            type="primary"
            @click="showAddCateDialog"
          >
            添加分类
          </el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        index-text="#"
        border
        :show-row-hover="false"
        class="tree-table"
      >
        <!-- 是否有效 -->
        <template
          slot="isok"
          slot-scope="scope"
        >
          <i
            v-if="scope.row.cat_deleted === false"
            class="el-icon-success"
            :style="{color: 'lightgreen'}"
          />
          <i
            v-else
            class="el-icon-error"
            style="color: red;"
          />
        </template>
        <!-- 排序 -->
        <template
          slot="order"
          slot-scope="scope"
        >
          <el-tag
            type="primary"
            v-if="scope.row.cat_level === 0"
            size="mini"
          >
            一级
          </el-tag>
          <el-tag
            type="success"
            v-if="scope.row.cat_level === 1"
            size="mini"
          >
            二级
          </el-tag>
          <el-tag
            type="warning"
            v-if="scope.row.cat_level === 2"
            size="mini"
          >
            三级
          </el-tag>
        </template>
        <!-- 操作 -->
        <template
          slot="opt"
          slot-scope="scope"
        >
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="openEidtCateDialog(scope.row.cat_id)"
          >
            编辑
          </el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteCate(scope.row.cat_id)"
          >
            删除
          </el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      />
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加商品分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item
          label="分类名称: "
          prop="cat_name"
        >
          <el-input v-model="addCateForm.cat_name" />
        </el-form-item>
        <el-form-item
          label="父级分类: "
        >
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ expandTrigger: 'hover', value: 'cat_id', children: 'children', label: 'cat_name', checkStrictly: true }"
            @change="parentCateChange"
            clearable
          />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="addCate"
        >确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑商品分类对话框 -->
    <el-dialog
      title="修改商品分类名称"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @click="closeEditCateDialog"
    >
      <el-form
        :model="editCateForm"
        :rules="editCateFormRules"
        ref="editCateFormRef"
        label-width="100px"
      >
        <el-form-item
          label="分类名称"
          prop="cat_name"
        >
          <el-input v-model="editCateForm.cat_name" />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="submitEditCateDialog"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类的数据列表
      catelist: [],
      //   查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //   总数据条数
      total: 0,
      //   为table指定列的定义
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        { label: '是否有效', type: 'template', template: 'isok' },
        { label: '排序', type: 'template', template: 'order' },
        { label: '操作', type: 'template', template: 'opt' }
      ],
      //   控制添加商品分类对话框的打卡与关闭
      addCateDialogVisible: false,
      addCateForm: {
        //   将要添加的分类名称
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      //   父级分类列表
      parentCateList: [],
      //   选中的父级id数组
      selectedKeys: [],
      //   控制编辑商品分类对话框的打开与关闭
      editCateDialogVisible: false,
      //   当前要编辑的商品分类id
      currentEditCateId: '',
      //   编辑商品分类表单数据
      editCateForm: {
        cat_name: ''
      },
      // 编辑商品分类表单验证规则
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请填写分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败')
      //   把数据列表赋值给catalist
      this.catelist = res.data.result
      //   为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 打开添加分类对话框
    showAddCateDialog () {
      // 获取父级分类
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类数据失败')
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChange () {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      }
    },
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error('添加商品分类失败')
        this.$message.success('添加商品分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
      console.log(this.addCateForm)
    },
    // 监听对话框的关闭事件
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 打开编辑商品分类名称对话框
    openEidtCateDialog (id) {
      this.currentEditCateId = id
      this.editCateDialogVisible = true
    },
    // 提交编辑商品分类数据
    async submitEditCateDialog () {
      const { data: res } = await this.$http.put('categories/' + this.currentEditCateId, this.editCateForm)
      if (res.meta.status !== 200) return this.$message.error('修改商品分类名称失败')
      this.$message.success('修改商品分类名称成功')
      this.getCateList()
      this.editCateDialogVisible = false
    },
    // 关闭编辑商品分类对话框
    closeEditCateDialog () {
      this.editCateDialogVisible = false
      this.editCateForm.cat_name = ''
    },
    // 删除商品分类
    async deleteCate (id) {
      const deleteRes = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (deleteRes !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除商品分类失败')
      this.$message.success('删除商品分类成功')
      this.getCateList()
    }

  }
}
</script>

<style lang="less" scoped>
.tree-table{
    margin-top: 15px;
}
.el-cascader{
    width: 100%;
}
</style>
