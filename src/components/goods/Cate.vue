<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">
            添加分类
          </el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <tree-table
        :data="cateList"
        :columns="cateColumns"
        :selection-type="false"
        :expand-type="false"
        :show-row-hover="false"
        show-index
        index-text="#"
        border
        class="treeTable"
      >
        <!-- 是否有效区域， 设置对应的模板列： slot="isok"(与columns中设置的template一致) -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            size="mini"
            type="success"
            v-else-if="scope.row.cat_level === 1"
          >
            二级
          </el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="showEditCateDialog(scope.row.cat_id)"
          >
            编辑
          </el-button>
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
            @click="removeCate(scope.row.cat_id)"
          >
            删除
          </el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>

      <!-- 添加分类对话框 -->
      <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClosed"
      >
        <el-form
          :model="addCateForm"
          :rules="addCateFormRules"
          ref="addCateFormRef"
          label-width="100px"
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类" prop="cat_pid">
            <el-cascader
              expandTrigger="hover"
              v-model="selectedKeys"
              :options="parentCateList"
              :props="cascaderProps"
              @change="parentCateChange"
              clearable
              change-on-select
            ></el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 编辑分类对话框 -->
      <el-dialog
        title="编辑分类"
        :visible.sync="editCateDialogVisible"
        width="50%"
        @close="editCateDialogClosed"
      >
        <el-form
          :model="editCateForm"
          :rules="editCateFormRules"
          ref="editCateFormRef"
          label-width="100px"
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="editCateForm.cat_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editCate">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      cateColumns: [
        { label: '分类名称', prop: 'cat_name' },
        // type:'template'(将该列设置为模板列)，template:'isok'(设置该列模板的名称为isok)
        { label: '是否有效', prop: '', type: 'template', template: 'isok' },
        { label: '排序', prop: '', type: 'template', template: 'order' },
        { label: '操作', prop: '', type: 'template', template: 'opt' }
      ],
      addCateDialogVisible: false,
      editCateDialogVisible: false,
      addCateForm: {
        // 分类名称
        cat_name: '',
        // 添加分类的父级id，0则表示父级为0.添加一级分类
        cat_pid: 0,
        // 添加分类的等级，0则表示添加一级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      editCateForm: {
        cat_id: '',
        cat_name: ''
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 保存1,2级父级分类的列表
      parentCateList: [],
      // 配置级联菜单中数据如何展示
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 绑定用户选择的分类值
      selectedKeys: []
    }
  },

  created() {
    this.getCateList()
  },

  methods: {
    handleSizeChange(newSize) {
      // 当pagesize发生改变时触发
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },

    handleCurrentChange(newPage) {
      // 当pagenum发生改变时触发
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },

    showAddCateDialog() {
      // 调用getParentCateList获取分类列表
      this.getParentCateList()
      // 显示添加分类对话框
      this.addCateDialogVisible = true
    },

    async showEditCateDialog(cateId) {
      const { data: res } = await this.$http.get(`categories/${cateId}`)
      if (res.meta.status !== 200) {
        return this.$Message.error('获取分类失败！')
      }
      this.editCateForm.cat_name = res.data.cat_name
      this.editCateForm.cat_id = res.data.cat_id
      this.editCateDialogVisible = true
    },

    async editCate() {
      const { data: res } = await this.$http.put(
        `categories/${this.editCateForm.cat_id}`,
        {
          cat_name: this.editCateForm.cat_name
        }
      )

      if (res.meta.status !== 200) {
        return this.$Message.error('编辑提交分类失败')
      }
      this.editCateDialogVisible = false
      this.getCateList()
    },

    removeCate(cateId) {
      this.$Confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete('categories/' + cateId)
          if (res.meta.status !== 200) {
            return this.$Message.error('删除分类失败！')
          } else {
            this.$Message({
              type: 'success',
              message: '删除分类成功!'
            })
            this.getCateList()
          }
        })
        .catch(() => {
          this.$Message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },

    async getParentCateList() {
      // 获取父级分类数据列表
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })

      if (res.meta.status !== 200) {
        return this.$Message.error('获取商品分类列表数据失败')
      }
      this.parentCateList = res.data
    },

    parentCateChange() {
      // 如果用户选择了父级分类
      if (this.selectedKeys.length > 0) {
        // 则将数组中的最后一项设置为父级分类
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // level也要跟着发生变化
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },

    addCateDialogClosed() {
      // 当关闭添加分类对话框时，重置表单
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },

    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },

    addCate() {
      // 点击确定，完成添加分类
      console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        // 发送请求完成添加分类
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )

        if (res.meta.status !== 201) {
          return this.$Message.error('添加分类失败')
        }

        this.$Message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },

    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })

      if (res.meta.status !== 200) {
        return this.$Message.error('获取商品分类失败！')
      } else {
        this.cateList = res.data.result
        this.total = res.data.total
      }
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
