<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
      <el-table
        :data="rightsList"
        border
        stripe
      >
        <el-table-column
          type="index"
          label="#"
        />
        <el-table-column
          prop="authName"
          label="权限名称"
        />
        <el-table-column
          prop="path"
          label="路径"
        />
        <el-table-column

          label="权限等级"
        >
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">
              一级
            </el-tag>
            <el-tag
              type="success"
              v-else-if="scope.row.level === '1'"
            >
              二级
            </el-tag>
            <el-tag
              type="warning"
              v-else
            >
              三级
            </el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有的权限列表
      rightsList: []
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    //   获取权限列表
    async getRightsList () {
      const { data: res } = await this.$http.get(`rights/${'list'}`)
      if (res.meta.status !== 200) return this.$message.error('权限列表请求失败')
      this.$message.success('权限列表请求成功')
      this.rightsList = res.data
    }
  }
}
</script>

<style lang="less" scoped>

</style>
