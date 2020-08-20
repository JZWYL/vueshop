<template>
    <div>
      <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!-- 卡片视图 -->
      <el-card>
          <!-- 添加角色按钮区域 -->
          <el-row>
              <el-col>
                  <el-button type="primary" @click="addRoleShow">添加角色</el-button>
              </el-col>
          </el-row>
          <!-- 角色列表区域 -->
          <el-table :data="rolesList" border stripe>
              <!-- 权限展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                  <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '','vcenter']"  v-for="(item1,i1) in scope.row.children" :key="item1.id">
                    <!-- 渲染一级查询 -->
                    <el-col :span="5">
                        <el-tag closable  @close="removeRightsById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 渲染二级三级权限  -->
                    <el-col :span="19">
                        <!-- 渲染二级权限 -->
                        <el-row :class="[ i2 === 0 ? '' : 'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                          <el-col :span="6">
                            <el-tag type="success" closable  @close="removeRightsById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                          </el-col>
                          <!-- 三级 -->
                          <el-col :span="18">
                            <el-tag v-for="(item3) in item2.children" :key="item3.id" closable  @close="removeRightsById(scope.row,item3.id)" type="warning">{{item3.authName}}</el-tag>
                          </el-col>
                        </el-row>
                    </el-col>
                  </el-row>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editRoleshow(scope.row.id)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRole(scope.row.id)">删除</el-button>
                <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
              </template>
            </el-table-column>
          </el-table>
      </el-card>
      <!-- 分配权限 -->
      <el-dialog title="分配权限" :visible.sync="setRightDialogVisible"
      width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
        <el-tree :data="rightList" :props="treeProps" :default-checked-keys="defKeys"
        default-expand-all show-checkbox node-key="id" ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRight">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加角色对话框 -->
      <el-dialog title="添加角色" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClosed">
        <!-- 内容 -->
        <el-form :model="addRoleForm" :rules="addFormRules" ref="addRoleFormRef" label-width="80px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="addRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑角色对话框 -->
      <el-dialog title="编辑角色" :visible.sync="editRoleDialogVisible" width="50%">
        <!-- 内容 -->
        <el-form :model="editRoleForm" :rules="addFormRules" ref="editRoleFormRef" label-width="80px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="editRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRole">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      // 角色列表
      rolesList: [],
      // 控制权限分配对话框的显示隐藏
      setRightDialogVisible: false,
      // 权限列表
      rightList: [],
      // 树形控件的属性绑定
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点id
      defKeys: [],
      //  当前即将分配权限的id'
      roleId: '',
      // 添加角色对话框的显示隐藏
      setRoleDialogVisible: false,
      // 添加角色的表单数据
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ]
      },
      // 待编辑角色
      editRoleForm: '',
      editRoleDialogVisible: false
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表数据失败')
      }
      this.rolesList = res.data
    },
    // 删除角色权限
    async removeRightsById (role, rightId) {
      const confirmResult = await this.$confirm('是否删除该用户, 是否继续?', '删除用户', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(e => e)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除操作')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    // 展示分配权限对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      // 获取的权限数据存储到data
      this.rightList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归形式获取角色下所有三级id
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 添加权限对话框的关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRight () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    addRoleShow () {
      this.setRoleDialogVisible = true
    },
    // 添加角色
    addRole () {
      this.$refs.addRoleFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.setRoleDialogVisible = false
        this.getRolesList()
      })
    },
    setRoleDialogClosed () {
      this.$refs.addRoleFormRef.resetFields()
      this.addRoleForm.roleDesc = ''
    },
    // 获取当前编辑的角色并显示对话框
    async editRoleshow (id) {
      const { data: res } = await this.$http.get(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户角色信息失败')
      }
      this.editRoleForm = res.data
      this.editRoleDialogVisible = true
    },
    // 角色修改编辑
    editRole () {
      this.$refs.editRoleFormRef.validate(async valid => {
        const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, {
          roleName: this.editRoleForm.roleName,
          roleDesc: this.editRoleForm.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('角色修改失败')
        }
        this.editRoleDialogVisible = false
        this.getRolesList()
        this.$message.success('修改成功')
      })
    },
    // 删除角色
    async removeRole (id) {
      const confirmResult = await this.$confirm('是否删除该角色, 是否继续?', '删除用户', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(e => e)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除操作')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除用户成功')
      this.getRolesList()
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
