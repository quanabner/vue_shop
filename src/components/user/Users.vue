<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">
            添加用户
          </el-button>
        </el-col>
      </el-row>

      <!-- 用户列表区 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChange(scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
                @click="setRole(scope.row)"
              ></el-button>
            </el-tooltip>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeUser(scope.row.id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>

      <!-- 添加用户的对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClose"
      >
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">
            确 定
          </el-button>
        </span>
      </el-dialog>

      <!-- 修改用户的对话框 -->
      <el-dialog
        title="修改用户信息"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">
            确 定
          </el-button>
        </span>
      </el-dialog>

      <!-- 分配角色的对话框 -->
      <el-dialog
        title="分配角色"
        :visible.sync="setRoleDialogVisible"
        width="50%"
        @close="setRoleDialogClosed"
      >
        <div>
          <p>当前的用户: {{ userInfo.username }}</p>
          <p>当前的角色: {{ userInfo.role_name }}</p>
          <p>
            分配新角色:
            <el-select v-model="selectedRoleId" placeholder="请选择角色">
              <!-- :label 显示文本，:value 选中值 -->
              <el-option
                v-for="item in rolesList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id"
              ></el-option>
            </el-select>
          </p>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveRoleInfo">
            确 定
          </el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号码'))
    }
    return {
      // 获取用户列表数据的对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      addDialogVisible: false, // 控制添加用户对话框的显示与隐藏
      editDialogVisible: false, // 修改用户对话框
      setRoleDialogVisible: false, // 分配角色对话框
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      userInfo: {},
      rolesList: [],
      selectedRoleId: '',
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名长度在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '密码长度在6~15个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: ['blur', 'change']
          }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            validator: checkMobile,
            trigger: 'blur'
          }
        ]
      },
      editForm: {},
      editFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名长度在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入用户邮箱', trigger: 'blur' },
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '请输入用户手机号', trigger: 'blur' },
          {
            validator: checkMobile,
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },

    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },

    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },

    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },

    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$Message.error('请填写完整用户信息')
        // 发送请求完成添加用户的操作
        const { data: res } = await this.$http.post('users', this.addForm)
        // 判断如果添加失败，就做提示
        if (res.meta.status !== 201) {
          return this.$Message.error('添加用户失败')
        }
        // 添加成功的提示
        this.$Message.success('添加用户成功')
        // 关闭对话框
        this.addDialogVisible = false
        // 重新请求最新的数据
        this.getUserList()
      })
    },

    editUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$Message.error('请填写完整用户信息')
        const { data: res } = await this.$http.put(
          `users/${this.editForm.id}`,
          { email: this.editForm.email, mobile: this.editForm.mobile }
        )
        if (res.meta.status !== 200) {
          this.$Message.error('更新用户信息失败！')
        } else {
          this.editDialogVisible = false
          this.getUserList()
          this.$Message.success('更新用户信息成功')
        }
      })
    },

    removeUser(userId) {
      this.$Confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete('users/' + userId)
          if (res.meta.status !== 200) {
            return this.$Message.error('删除用户失败！')
          } else {
            this.$Message({
              type: 'success',
              message: '删除成功!'
            })
            this.getUserList()
          }
        })
        .catch(() => {
          this.$Message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },

    async showEditDialog(userId) {
      const { data: res } = await this.$http.get('users/' + userId)
      if (res.meta.status !== 200) {
        return this.$Message.error('查询用户信息失败！')
      } else {
        this.editForm = res.data
      }
      this.editDialogVisible = true
    },

    async userStateChange(userInfo) {
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        this.$Message.error('更新用户状态失败！')
      } else {
        this.$Message.success('更新用户状态成功')
      }
    },

    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$Message.error('获取用户列表失败！')
      } else {
        this.userList = res.data.users
        this.total = res.data.total
      }
    },

    async setRole(userInfo) {
      // 保存起来以供后续使用
      this.userInfo = userInfo
      // 获取所有的角色信息，以备下拉列表使用
      // 发送请求根据id完成删除操作
      const { data: res } = await this.$http.get('roles')
      // 判断如果删除失败，就做提示
      if (res.meta.status !== 200) { return this.$Message.error('获取角色列表失败') }

      this.rolesList = res.data
      // 展示分配角色对话框
      this.setRoleDialogVisible = true
    },

    async saveRoleInfo() {
      // 当用户点击确定按钮之后
      // 判断用户是否选择了需要分配的角色
      if (!this.selectedRoleId) {
        return this.$Message.error('请选择需要分配的角色')
      }
      // 发送请求完成分配角色的操作
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId })

      // 判断如果删除失败，就做提示
      if (res.meta.status !== 200) { return this.$Message.error('分配角色失败') }

      this.$Message.success('分配角色成功')
      this.getUserList()
      // 关闭对话框
      this.setRoleDialogVisible = false
    },

    setRoleDialogClosed() {
      // 当关闭对话框的时候，重置下拉框中的内容
      this.selectedRoleId = ''
      this.userInfo = {}
    }
  }
}
</script>

<style></style>
