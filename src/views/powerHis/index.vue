<template>
  <div class="app-container">
    <el-dialog  class="dialog1" style = "width: 80%;height:1000px"
               destroy-on-close
               title="待校验文件"
               :visible="powerStrShow"
               :before-close="() => powerStrShow=false"
    >
      <el-input  v-model="powerStr" type="textarea" style="height:500px"  height = "500px"></el-input>
      <div class="btn">
        <el-button  type="primary" @click = "clickPowerAuth()">确定</el-button>
        <el-button @click="powerStrShow = false" >取消</el-button>
      </div>
    </el-dialog>
    <el-dialog class="dialog2"
      destroy-on-close
      :visible="previewShow"
      :before-close="() => previewShow=false">
      <el-row>
        <el-col>
          <el-date-picker  format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"   v-model="authEndTime" type="datetime" placeholder="追加时间" style="width: 100%;" />
        </el-col>
      </el-row>
      <div class="btn">
          <el-button type="primary" @click = "clickAuth()">确定</el-button>
          <el-button   @click="previewShow = false">取消</el-button>
      </div>
    </el-dialog>
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="序号" width="50">
        <template slot-scope="scope">
          {{ scope.$index+1 }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="公司名称" width="250">
        <template slot-scope="scope">
          {{ scope.row.companyName }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="使用类型" width="100">
        <template slot-scope="scope">
          {{ scope.row.comType }}
        </template>
      </el-table-column>
      <el-table-column label="出厂时间" width="200" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.factoryTime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开始时间" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.startTime }}
        </template>
      </el-table-column>
      <el-table-column label="结束时间" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.endTime }}
        </template>
      </el-table-column>
      <el-table-column label="延期时间" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.endEndTime }}
        </template>
      </el-table-column>
      <el-table-column label="用户注册量" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.userResCount }}
        </template>
      </el-table-column>
      <el-table-column label="用户在线量" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.userOnLineCount }}
        </template>
        </el-table-column>
        <el-table-column label="操作"  align="center">
          <template slot-scope="scope">
            <el-dropdown split-button  @command="handleCommand">
               <i class="el-icon-view"></i> 预览
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item :command="{command:'downFile',id:scope.row.id}"><i class="el-icon-download"></i> 出厂文件</el-dropdown-item>
                <el-dropdown-item :command="{command:'showDialogAuth',id:scope.row.id}"><i class="el-icon-upload2"></i> 返厂授权</el-dropdown-item>
                <el-dropdown-item :command="{command:'downFileVer',id:scope.row.id}"><i class="el-icon-download"></i> 校验文件</el-dropdown-item>
                <el-dropdown-item :command="{command:'showDialogEndTime',id:scope.row.id}"><i class="el-icon-date"></i> 系统延期</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </template>
        </el-table-column>
    </el-table>
  </div>
</template>
<style>
.el-textarea__inner{
  height: 500px;
  overflow-y: auto;
  /* overflow-y: auto;兼容ie  */
}

</style>
<script>
import { getList } from '@/api/table'
import request, {download} from '@/utils/request'

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      powerStr: null,
      formId: null,
      authEndTime: null,
      previewShow: false,
      powerStrShow: false,
      list: null,
      listLoading: true
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    handleCommand(val) {
      this[val.command](val.id)
    },
    downFile(id) {
      const url = '/auth/downloadLicense/' + id
      window.open(url)
    },
    downFileVer(id){
      const url = '/auth/downFileVer/' + id
      window.open(url)
    },
    fetchData() {
      this.listLoading = true
      getList(null).then(response => {
        this.list = response.data
        this.listLoading = false
      })
    },
    showDialogEndTime(id) {
      this.formId = id
      this.previewShow = true
    },
    clickAuth() {
      if(!this.authEndTime) {
        this.$message({
            message: '请选择时间',
            type: 'error'
          })
          return
      }
      request({
        url: '/auth/addAuthEndTime?authEndTime=' + this.authEndTime + '&id=' + this.formId ,
        method: 'post' ,
      }).then(response => {
        const data = response
        this.$message({
          message: '延期设置成功',
          type: 'success'
        })
        this.previewShow = false
        this.fetchData()
      })
    },
    clickPowerAuth(){
      if(!this.powerStr) {
          this.$message({
            message: '请输入待校验文件',
            type: 'error'
          })
          return
      }
      request({
        url: '/auth/addAuthIpMAC/' + this.formId ,
        method: 'post' ,
        data:this.powerStr
      }).then(response => {
        if(!response.flag){
          this.$message({
            message: response.message,
            type: 'error'
          })
          return ;
        }
        const data = response
        this.$message({
          message: '返厂授权已生成',
          type: 'success'
        })
        this.powerStrShow = false
        this.fetchData()

      })
    },
    showDialogAuth(id) {
      this.formId = id
      this.powerStrShow = true
    }
  }
}
</script>
<style>
  .btn {
    margin-top: 10px;
    display: flex;
    justify-content: center;
  }
  .dialog2 .el-dialog{
    width: 500px;
  }
</style>
