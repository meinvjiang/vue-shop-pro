<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card class="box-card">
      <!-- 搜索框和添加用户 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入搜索内容"
            v-model="queryParams.query"
            :clearable="true"
            @clear="getUserInfos"
            @keyup.enter.native="getUserInfos"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfos"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%"
        top="185px"
        @close="addDiglogClose"
      >
        <el-form
          :rules="addFromRules"
          ref="addFromRef"
          :model="addFrom"
          label-width="80px"
          class="demo-ruleForm"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addFrom.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input type="password" v-model="addFrom.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addFrom.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="addFrom.mobile"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </div>
      </el-dialog>
<!-- @close="editDiglogClose" -->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        top="185px"
        
      >
        <el-form
          :rules="editFromRules"
          ref="addFromRef"
          :model="editFrom"
          label-width="80px"
          class="demo-ruleForm"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editFrom.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editFrom.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="editFrom.mobile"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser" >确 定</el-button>
        </div>
        <!-- @click="editUser" -->
      </el-dialog>
      <!-- 数据列表展示 -->
      <el-table :data="userInfos" border style="width: 100%">
        <el-table-column type="index" label="序号" width="50"></el-table-column>
        <el-table-column prop="username" label="用户名" width="160"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="160"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="160"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="160"></el-table-column>
        <el-table-column label="状态" width="80">
          <el-switch
            v-model="infos.row.mg_state"
            slot-scope="infos"
            @change="changeState(infos.row.id,infos.row.mg_state)"
          ></el-switch>
          <!-- <span slot-scope="infos">{{infos.row.username}}</span> -->
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="info">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(info.row.id)"></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deLuser(info.row.id)"
            ></el-button>
            <el-tooltip content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 数据分页展示区域 -->
      <!-- size-change每页信息条数变化的回调函数处理 -->
      <!-- current-change当前页码变化的回调处理 -->
      <!-- current-page当前默认第几页 -->
      <!-- page-sizes 每页显示数据-->
      <!-- page-size 默认每页记录条数-->
      <!-- total 总记录条数 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryParams.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryParams.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="queryParams.total"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getUserInfos()
  },
  methods: {
    //修改用户信息
   async showEditDialog(id){
      this.editDialogVisible = true
      //把被修改用户的信息展示到form表单中
      //@param id:被修改的用户
      // 根据id查询被修改的信息
      const {data:res} = await this.$http.get('users/'+id)
      if(res.meta.status !== 200){
        return this.$message({
          message:'获取用户信息失败！',
          type:'error',
          duration:1000
        })
      }
      this.editFrom = res.data 
      this.editDialogVisible = true
    },
    //接受信息用户
   async editUser(id){
    const {data:res} = await this.$http.put('users/'+this.editFrom.id,this.editFrom)
    if(res.meta.status !== 200){return this.$message.error(res.meta.msg)}
    this.editDialogVisible = false
    this.$message.success(res.meta.msg)
    this.getUserInfos()

    },
    // 删除用户
    deLuser(id) {
      this.$confirm('确定要删除该会员吗, 是否继续?', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          console.log(res + '12121212121')
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        })
        .catch(() => {})
    },
    // 添加用户相关
    //收集用户信息存储到后端api的数据中
    addUser() {
      this.$refs.addFromRef.validate(async vaild => {
        if (vaild) {
          const { data: res } = await this.$http.post('users', this.addFrom)
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }
          this.addDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    addDiglogClose() {
      this.$refs.addFromRef.resetFields()
    },
    async changeState(id, state) {
      const { data: res } = await this.$http.put(`users/${id}/state/${state}`)
      if (res.meta.status !== 200) {
        return this.$message.error('修改用户状态！')
      }
      this.$message({
        message: '修改状态成功',
        type: 'success',
        duration: 1500
      })
    },
    // 数据分页相关
    handleSizeChange(val) {
      // val:代表变化后的信息条数值
      this.queryParams.pagesize = val
      //根据变化后的显示条数从新获得数据
      this.getUserInfos()
    },
    // 当前页码回调函数的处理
    handleCurrentChange(val) {
      // val:代表变化后的页码
      this.queryParams.pagenum = val
      this.getUserInfos()
    },
    // 获得收评数据
    async getUserInfos() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryParams
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.queryParams.total = res.data.total
      //把获得的用户数据 赋予给userInfos
      this.userInfos = res.data.users
    }
  },
  data() {
    
    // 自定义校验器
    //callback 回调函数
    // value 被校验的数据
    var checkMobile = (rule, value, callback) => {
      // 校验手机号码规则
      if (!/^1[359789]\d{9}$/.test(value)) {
        return callback(new Error('手机号码格式不正确'))
      }
      callback()
    }
    return {
      keywords: '',
      //接收用户数据
      userInfos: [],
      //定义获取用户数据时用到的查询条件参数
      queryParams: {
        query: '', //用于关键字搜素使用
        pagenum: 1, //默认显示第几页
        pagesize: 3, //每行显示条数
        total: 0 //记录总条数
      },
      addDialogVisible: false,
      addFrom: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },

      addFromRules: {
        username: [
          { required: true, message: '用户名必填！', trigger: 'blur' }
        ],
        password: [{ required: true, message: '密码必填！', trigger: 'blur' }],
        mobile: [
          { required: true, message: '手机号码必填！', trigger: 'blur' },
          //自定义校验器
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 修改用户
      editDialogVisible: false,
      editFrom: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },

      editFromRules: {
        mobile: [
          { required: true, message: '手机号码必填！', trigger: 'blur' },
          //自定义校验器
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
    }
  }
}
</script>

<style lang="less" scoped>
</style>

