<template>
  <div class="app-container">
    <el-form ref="form" :model="form" label-width="120px">
      <el-form-item label="企业名称" :span = "24">
        <el-input v-model="form.licenseBus.commpany"   />

      </el-form-item>
      <el-form-item label="用户注册量" :span="24">
        <el-col :span="11">
        <el-input  v-model="form.licenseUser.userResCount"   />
        </el-col>
        <el-col :span="2" class="line">用户在线</el-col>
        <el-col :span="11">
          <el-input  v-model="form.licenseUser.userOnCount"   />
        </el-col>
      </el-form-item>
      <el-form-item label="使用类型" :span = "24">
        <el-radio-group v-model="form.isFalg">
          <el-radio label="试用" value="1"/>
          <el-radio label="正式" value= "0"/>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="启用时间" :span = "24">
<!--        <el-col :span="7">-->
<!--          <el-date-picker v-model="form.factoryTime" type="datetime" placeholder="出厂时间" style="width: 100%;" />-->
<!--        </el-col>-->
        <el-col :span="11">
          <el-date-picker  format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"   v-model="form.makeTime" type="datetime" placeholder="启用时间" style="width: 100%;" />
        </el-col>
        <el-col :span="2" class="line">截止时间</el-col>
        <el-col :span="11">
          <el-date-picker  format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"    v-model="form.notAfterTime" type="datetime" placeholder="截止时间" style="width: 100%;" />
        </el-col>
      </el-form-item>
      <el-form-item label="授权菜单" >
        <el-button size="small" @click="expandHandle">{{
            expandAll ? "全部收起" : "全部展开"
          }}</el-button>
        <el-col :span = "24" style = "width:300px;height: 500px;overflow:scroll;display:flex">
           <el-tree
             ref="tree"
             node-key="ID_"
             :data="options"
             show-checkbox
             check-on-click-node
             :highlight-current="true"
             :default-expand-all="false"
             :expand-on-click-node="true"
             :props="defaultProps"
           ></el-tree>
        </el-col>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">提交</el-button>
        <el-button @click="onCancel" v-if = "false">关闭</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import request from "@/utils/request";

export default {
  data() {
    return {

      form: {
        licenseBus: {
          commpany: ''
        },
        licenseUser: {
          userResCount: 0,
          userOnCount: 0
        },
        menus: '',
        isFalg: true
      },
      options: [],
      defaultProps: { Id: 'ID_',
        label: 'name_',
        PId: 'PARENT_ID_'
      },
      expandAll: false
    }

  },
  methods: {
    onSubmit() {
      let that = this
      let menus = JSON.stringify(this.$refs.tree.getCheckedNodes(false))
      that.form.menus = menus
      request({
        url: '/auth/savePower',
        method: 'post',
        data:this.form
      }).then(response => {
        const data = response
        if (data.code == 20000) {
          that.$message({
            message: '注册成功',
            type: 'success'
          })
        } else {
          that.$message({
            message: '注册失败 , 请联系管理哦',
            type: 'error'
          })
        }
      })
    },
    onCancel() {
      this.$message({
        message: 'cancel!',
        type: 'warning'
      })
    },
    initMenus() {
      const this_ = this
      request({
        url: '/auth/getMenus',
        method: 'post'
      }).then(response => {
        const data = response.data
        this_.options = data
        this_.$nextTick(()=>{
          this_.expandHandle()
        })

      })
    },
    expandHandle() {
      this.expandAll = !this.expandAll
      this.expandNodes(this.$refs.tree.store.root)
    },
    // 遍历树形数据，设置每一项的expanded属性，实现展开收起
    expandNodes(node) {
      node.expanded = this.expandAll
      for (let i = 0; i < node.childNodes.length; i++) {
        node.childNodes[i].expanded = this.expandAll
        if (node.childNodes[i].childNodes.length > 0) {
          this.expandNodes(node.childNodes[i])
        }
      }
    }

  },
  mounted() {
    this.initMenus()
  }
}
</script>

<style scoped>
.line{
  text-align: center;
}
</style>

