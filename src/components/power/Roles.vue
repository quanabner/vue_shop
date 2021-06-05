<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">
            添加角色
          </el-button>
        </el-col>
      </el-row>

      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            >
              编辑
            </el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRole(scope.row.id)"
            >
              删除
            </el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSettingRightDialog(scope.row)"
            >
              分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 添加角色的对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClose"
      >
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="90px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">
            确 定
          </el-button>
        </span>
      </el-dialog>

      <!-- 编辑角色的对话框 -->
      <el-dialog
        title="修改角色信息"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="90px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRole">
            确 定
          </el-button>
        </span>
      </el-dialog>

      <!-- 分配权限的对话框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="settingRightDialogVisible"
        width="50%"
        @close="settingRightDialogClosed"
      >
        <el-tree
          :data="rightsList"
          :props="treeProps"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="defaultKeys"
          ref="treeRef"
        ></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="settingRightDialogVisible = false">
            取 消
          </el-button>
          <el-button type="primary" @click="settingRights">
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
    return {
      rolesList: [],
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {},
      editForm: {},
      addFormRules: {
        roleName: { required: true, message: '请输入角色名', trigger: 'blur' }
      },
      editFormRules: {
        roleName: { required: true, message: '请输入角色名', trigger: 'blur' }
      },
      settingRightDialogVisible: false,
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defaultKeys: [],
      roleId: '' // 当前即将分配权限的角色id
    }
  },

  created() {
    this.getRolesList()
  },

  methods: {
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },

    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },

    async showSettingRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$Message.error('获取权限数据失败！')
      } else {
        this.rightsList = res.data
      }

      this.getLeafKeys(role, this.defaultKeys)
      this.settingRightDialogVisible = true
    },

    settingRightDialogClosed() {
      this.defaultKeys = []
    },

    async settingRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedNodes()
      ]
      const idsStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idsStr })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$Message.error('分配权限失败！')
      } else {
        this.$Message.success('分配权限成功')
        this.getRolesList()
      }
      this.settingRightDialogVisible = false
    },

    async getRolesList() {
      const { data: res } = await this.$http.get('roles')

      if (res.meta.status !== 200) {
        return this.$Message.error('获取角色列表失败')
      } else {
        this.rolesList = res.data
      }
    },

    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$Message.error('请填写完整角色信息')
        // 发送请求完成添加角色的操作
        const { data: res } = await this.$http.post('roles', this.addForm)
        // 判断如果添加失败，就做提示
        if (res.meta.status !== 201) {
          return this.$Message.error('添加角色失败')
        }
        // 添加成功的提示
        this.$Message.success('添加角色成功')
        // 关闭对话框
        this.addDialogVisible = false
        // 重新请求最新的数据
        this.getRolesList()
      })
    },

    editRole() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$Message.error('请填写完整角色信息')
        const { data: res } = await this.$http.put(
          `roles/${this.editForm.roleId}`,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          }
        )
        if (res.meta.status !== 200) {
          this.$Message.error('更新角色信息失败！')
        } else {
          this.editDialogVisible = false
          this.getRolesList()
          this.$Message.success('更新角色信息成功')
        }
      })
    },

    async showEditDialog(roleId) {
      const { data: res } = await this.$http.get('roles/' + roleId)
      if (res.meta.status !== 200) {
        return this.$Message.error('查询角色信息失败！')
      } else {
        this.editForm = res.data
      }
      this.editDialogVisible = true
    },

    removeRole(roleId) {
      this.$Confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete('roles/' + roleId)
          if (res.meta.status !== 200) {
            return this.$Message.error('删除角色失败！')
          } else {
            this.$Message({
              type: 'success',
              message: '删除角色成功!'
            })
            this.getRolesList()
          }
        })
        .catch(() => {
          this.$Message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },

    removeRightById(role, rightId) {
      this.$Confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(
            `roles/${role.id}/rights/${rightId}`
          )
          if (res.meta.status !== 200) {
            return this.$Message.error('删除权限失败！')
          } else {
            this.$Message({
              type: 'success',
              message: '删除权限成功!'
            })
            // this.getRolesList()
            role.children = res.data
          }
        })
        .catch(() => {
          this.$Message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },

    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
