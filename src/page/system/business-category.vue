<template>
  <div class="business-category">
      <i-layout :head="true">
        <div slot="tableHead">
          <el-button size="small" @click="createType">新建</el-button>
          <el-button size="small" @click="edit">修改</el-button>
          <el-button size="small" @click="remove">删除</el-button>
        </div>
        <el-tree
          v-loading="loading"
          :data="treeData"
          show-checkbox
          default-expand-all
          :check-strictly="true"
          node-key="systemUniqueId"
          ref="tree"
          highlight-current
          :props="defaultProps">
        </el-tree>
      </i-layout>
      <el-dialog :title="title" center :visible.sync="dialogVisible" width="500px" :before-close="dialogClose">
        <el-form :model="form" :rules="rules" ref="form" label-width="100px" size="mini">
          <el-form-item label="父节点名称" v-if="parentNode !== ''">
          <!-- <el-form-item label="手机号"> -->
            <el-input  type="text" v-model="parentNode" maxlength="20" disabled></el-input>
          </el-form-item>
          <el-form-item label="节点名称" prop="typeName">
          <!-- <el-form-item label="手机号"> -->
            <el-input  type="text" v-model="form.typeName" maxlength="20"></el-input>
          </el-form-item>
          <!-- <el-form-item label="节点组类" prop="typeGroup">
            <el-input v-model="form.typeGroup" maxlength="20"></el-input>
          </el-form-item> -->
          <el-form-item label="节点代号" prop="typeCode" v-if="lastFlag">
            <el-input v-model="form.typeCode" maxlength="20"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogCancel">取 消</el-button>
          <el-button type="primary" @click="editQuery">确 定</el-button>
        </div>
      </el-dialog>
  </div>
</template>
<script>
import req from '../../api/system'
export default {
  data () {
    return {
      treeData: [],
      defaultProps: {
        children: 'childList',
        label: 'labelName'
      },
      dialogVisible: false,
      form: {
        systemUniqueId: '',
        typeCode: '',
        typeGroup: '',
        typeName: ''
      },
      rules: {
        typeCode: [
          { required: true, message: '请输入节点代号', trigger: 'blur' }
        ],
        typeGroup: [
          { required: true, message: '请输入节点组类', trigger: 'blur' }
        ],
        typeName: [
          { required: true, message: '请输入节点名称', trigger: 'blur' }
        ]
      },
      createFlag: false,
      lastFlag: false,
      parentNode: '',
      title: '新建类别',
      loading: true
    }
  },
  created () {
    this.getData()
  },
  methods: {
    getData () {
      // this.$ajax.get('/dataManage/getSystemTypeList.json')
      req('getSystemTypeList')
        .then(res => {
          this.loading = false
          if (res.data.code === '00000') {
            this.treeData = res.data.data
            this.jointStr(this.treeData)
          } else {
            this.$message({
              type: 'error',
              message: res.data.msg
            })
          }
        })
    },
    jointStr (data) {
      data.map(val => {
        val.dataStatus
          ? val['labelName'] = val['typeName'] + ' — [ ' + val['typeGroup'] + ' - ' + val['typeCode'] + ' ]'
          : val['labelName'] = val['typeName'] + ' — [ ' + val['typeGroup'] + ' - ' + val['typeCode'] + ' ]' + ' - ' + '无效'
        val.childList.length && this.jointStr(val.childList)
      })
    },
    createType () {
      Object.keys(this.form).map(key => {
        this.form[key] = ''
      })
      if (this.$refs.tree.getCheckedNodes().length > 1) {
        this.$message({
          type: 'warning',
          message: '不能同时新增超过1个的类别的子类别'
        })
      } else {
        this.createFlag = true
        this.lastFlag = true
        this.dialogVisible = true
        this.title = '新建类别'
        this.$refs.tree.getCheckedNodes().length
          ? this.form.typeGroup = this.$refs.tree.getCheckedNodes()[0].typeCode
          : this.form.typeGroup = 'group'
        this.$refs.tree.getCheckedNodes().length
          ? this.parentNode = this.$refs.tree.getCheckedNodes()[0].labelName
          : this.parentNode = ''
      }
    },
    edit () {
      this.parentNode = ''
      if (!this.$refs.tree.getCheckedNodes().length) {
        this.$message({
          type: 'warning',
          message: '请勾选需要编辑的类别'
        })
      } else if (this.$refs.tree.getCheckedNodes().length > 1) {
        this.$message({
          type: 'warning',
          message: '不能同时编辑超过1个的类别'
        })
      } else {
        this.createFlag = false
        this.dialogVisible = true
        this.title = '修改类别'
        Object.keys(this.form).map(key => {
          this.form[key] = this.$refs.tree.getCheckedNodes()[0][key]
        })
        this.$refs.tree.getCheckedNodes()[0].childList.length ? this.lastFlag = false : this.lastFlag = true
      }
    },
    dialogCancel () {
      Object.keys(this.form).map(key => {
        this.form[key] = ''
      })
      this.dialogVisible = false
      this.$refs['form'].resetFields()
    },
    dialogClose (done) {
      done()
      this.$refs['form'].resetFields()
    },
    editQuery () {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          if (!this.createFlag) {
            // this.$ajax.post('/dataManage/modifySystemType.json', this.form)
            req('modifySystemType', this.form)
              .then(res => {
                if (res.data.code === '00000') {
                  this.dialogVisible = false
                  this.getData()
                  this.$message({
                    type: 'success',
                    message: '修改类别信息成功!'
                  })
                } else {
                  this.$message({
                    type: 'error',
                    message: res.data.msg
                  })
                }
              })
          } else {
            // this.$ajax.post('/dataManage/addSystemType.json', this.form)
            req('addSystemType', this.form)
              .then(res => {
                if (res.data.code === '00000') {
                  this.dialogVisible = false
                  this.getData()
                  this.$message({
                    type: 'success',
                    message: '新增类别信息成功!'
                  })
                } else {
                  this.$message({
                    type: 'error',
                    message: res.data.msg
                  })
                }
              })
          }
        } else {
          return false
        }
      })
    },
    remove () {
      if (!this.$refs.tree.getCheckedNodes().length) {
        this.$message({
          type: 'warning',
          message: '请勾选需要删除的类别'
        })
      } else if (this.$refs.tree.getCheckedNodes().length > 1) {
        this.$message({
          type: 'warning',
          message: '不能同时删除超过1个的类别'
        })
      } else {
        if (this.$refs.tree.getCheckedNodes()[0].childList.length) {
          this.$message({
            type: 'warning',
            message: '此类别包含有子类别，不能直接删除'
          })
        } else {
          this.$confirm('将此类别进行删除操作, 请确认是否删除?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            // this.$ajax.post('/dataManage/deleteSystemTypeBySystemUniqueId.json', {systemUniqueId: this.$refs.tree.getCheckedNodes()[0].systemUniqueId})
            req('deleteSystemType', {systemUniqueId: this.$refs.tree.getCheckedNodes()[0].systemUniqueId})
              .then(res => {
                if (res.data.code === '00000') {
                  this.getData()
                  this.$message({
                    type: 'success',
                    message: '删除成功!'
                  })
                } else {
                  this.$message({
                    type: 'error',
                    message: res.data.msg
                  })
                }
              })
          }).catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
        }
      }
    }
  }
}
</script>
<style lang="less">
.business-category{
  .el-input__inner::-webkit-outer-spin-button,
  .el-input__inner::-webkit-inner-spin-button {
    -webkit-appearance: none !important;
    margin: 0;
  }
  .el-input__inner[type="number"]{
    -moz-appearance: textfield;
  }
}
</style>
