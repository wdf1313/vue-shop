<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>

      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom',i1 ===0 ? 'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过 for 循环 嵌套渲染二级权限 -->
                <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag v-for="(item3,i3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
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
        <!-- 操作按钮 -->
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                <!-- 删除按钮 -->
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
                <!-- 分配权限按钮 -->
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>

          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 内容主题区域 -->
      <el-form :model="addRole" :rules="addRoleRules" ref="addRoleRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRole.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">提 交</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户信息的对话框 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="editDialogVisible"
      width="50%" @close="editDialogClosed">
      <el-form :model="editRoles" :rules="addRoleRules" ref="editRolesRef" label-width="100px">
        <el-form-item label="角色名称">
          <el-input v-model="editRoles.roleName" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoles.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      addDialogVisible: false,
      // 添加用户的表单数据
      addRole: {
        roleName: '',
        roleDesc: ''
      },
      // 添加与修改的验证规则对象
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '角色名称的长度在3~10个字符之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 15, message: '角色描述的长度在6~15个字符之间', trigger: 'blur' }
        ]
      },
      editRoles: {},
      // 控制修改用户的显示与隐藏
      editDialogVisible: false,
      // 控制分配权限对话框显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件的绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id数值组
      defKeys: [105],
      roleId: ''

    }
  },
  created () {
    // 请求角色数据列表
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('角色数据请求失败')

      this.rolesList = res.data
    },
    // 监听添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addRoleRef.resetFields()
    },
    // 点击按钮 添加新用户
    addUser () {
      // 预校验
      this.$refs.addRoleRef.validate(async valid => {
        if (!valid) return Error()
        // 可以发起真正的添加请求
        const { data: res } = await this.$http.post('roles', this.addRole)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }

        this.$message.success('添加角色成功')
        this.addDialogVisible = false
        // 重新获取用户列表数据
        this.getRolesList()
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }

      this.editRoles = res.data
      this.editDialogVisible = true
    },
    // 修改角色并提交
    editUserInfo () {
      // 预验证
      this.$refs.editRolesRef.validate(async valid => {
        if (!valid) return Error()
        // 发起用户信息的修改请求
        const { data: res } = await this.$http.put('roles/' + this.editRoles.roleId, {
          roleName: this.editRoles.roleName,
          roleDesc: this.editRoles.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色失败!')
        }
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('修改角色成功')
      })
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed () {
      this.$refs.editRolesRef.resetFields()
    },
    // 根据Id删除对应的用户信息
    async removeUserById (id) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除该用户信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为 字符串 confirm
      // 如果用户取消删除，则返回值为 字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 删除操作
      const { data: res } = await this.$http.delete('roles/' + id)

      if (res.meta.status !== 200) {
        return this.$message.error('用户信息删除失败!')
      }
      this.$message.success('删除成功!')
      this.getRolesList()
    },
    // 根据id删除对应的权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除!')
      }

      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败!')
      }
      // this.getRolesList()
      role.children = res.data
    },
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取到所以权限的数据
      const { data: res } = await this.$http.get('rights/tree')

      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败!')
      }
      // 把获取到的权限数据保存到 data中
      this.rightslist = res.data

      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式 获取角色下所有三级权限的id,并保存到 defKeys 数组中
    getLeafKeys (node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }

      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }

  }
}
</script>

<style lang="less" scoped>
  .el-tag{
    margin: 7px;
  }

  .bdtop{
    border-top: 1px solid #eee;
  }
  .bdbottom{
    border-bottom: 1px solid #eee;
  }

  .vcenter{
    display: flex;
    align-items: center;
  }
</style>
