<template>
  <div class="app-container">
    <el-dialog style = "width: 80%;height:1000px"
               destroy-on-close
               title="待校验文件"
               :visible="powerStrShow"
               :before-close="() => powerStrShow=false"
    >
      <el-input  v-model="powerStr" type="textarea" style="height:500px"  height = "500px"></el-input>
      <el-button @click = "clickPowerAuth()">确定</el-button>

    </el-dialog>
    <el-dialog style = "width: 500px"
      destroy-on-close
      :visible="previewShow"
      :before-close="() => previewShow=false">
      <el-row>
        <el-col>
          <el-date-picker  format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"   v-model="authEndTime" type="datetime" placeholder="追加时间" style="width: 100%;" />
        </el-col>
      </el-row>
      <el-row>
        <el-col>
          <el-button @click = "clickAuth()">确定</el-button>
        </el-col>
      </el-row>
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
           <el-button-group>
             <el-button size="small"  icon = "el-icon-download"
                        @click="downFile(scope.row.id)"
             >出厂文件</el-button>
             <el-button size="small"
                        @click="showDialogAuth(scope.row.id)"  icon = "el-icon-upload2"
             >返厂授权</el-button>
             <el-button size="small" icon = "el-icon-download"
                        @click="downFileVer(scope.row.id)"
             >校验文件</el-button>
<!--             <el-button size="small"-->
<!--               @click="showDialogAuth(scope.row.id)"-->
<!--             >详情</el-button>-->
             <el-button size="small" icon = "el-icon-date"
               @click="showDialogEndTime(scope.row.id)"
             >系统延期</el-button>
           </el-button-group>

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
      request({
        url: '/auth/addAuthIpMAC?powerStr=' + this.powerStr + '&id=' + this.formId ,
        method: 'post' ,
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
