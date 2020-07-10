<template>
  <div class="info-verify-container">
    <!-- 信息表格 -->
    <el-table :data="tableData.filter(data => !searchKey
        || data.name.toLowerCase().includes(searchKey.toLowerCase())
        || data.address.toLowerCase().includes(searchKey.toLowerCase()))"
              :default-sort="{prop:'id',order:'descending'}"
              height="600"
              sortable
              style="width: 100%">
      <el-table-column prop="id" label="ID" min-width="4%"/>
      <el-table-column prop="name" label="用户名" min-width="10%"/>
      <el-table-column prop="address" label="地址" min-width="46%"/>
      <el-table-column prop="status" label="状态" min-width="8%"/>
      <el-table-column label="操作" min-width="22%">
        <template slot="header" slot-scope="scope">
          <el-input v-model="searchKey" placeholder="输入关键词(用户名/地址)搜索"></el-input>
        </template>
        <!-- 操作按钮组 -->
        <template slot-scope="scope">
          <div class="btn-box">
            <el-button type="success" size="small" @click="onPass(scope.row)">通过</el-button>
            <el-button type="danger" size="small" @click="onReject(scope.row)">驳回</el-button>
          </div>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import request from '../../utils/request'
import qs from 'qs'

export default {
  data() {
    return {
      searchKey: '',
      tableData: []
    }
  },
  created() { // 网页加载完成时加载待审核用户信息
    this.updateData()
  },
  methods: {
    updateData() {
      request({
        baseURL: this.$global.baseURL,
        url: 'auths',
        method: 'get'
      }).then(resp => {
        console.log('[ INFO ] Get user data success!')

        let data = []
        for (let i = 0; i < resp.data.length; ++i) {
          data[i] = {
            'id': resp.data[i].authId,
            'name': resp.data[i].username,
            'address': resp.data[i].province + '省 '
              + resp.data[i].city + '市 '
              + resp.data[i].district + ' '
              + resp.data[i].block + ' '
              + resp.data[i].houseNo + '号楼 '
              + resp.data[i].floor + '层 '
              + resp.data[i].roomNo + '号',
            'status': resp.data[i].state
          }
        }
        this.tableData = data
      })
    },
    onPass(row) {

      // 打开审核确认对话框
      this.$confirm('是否确认通过？', '用户名：' + row.name, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {

        // 审核通过
        request({
          baseURL: this.$global.baseURL,
          url: 'auths/' + row.id,
          method: 'put',
          params: {
            pass: true
          }
        }).then(resp => {
          console.log('[ INFO ] Pass the audit success!')
          console.log(resp)
          this.$message({
            type: 'success',
            message: '信息审核通过!'
          })

          // 审核成功通过，重新加载信息
          this.updateData()
        }).catch(err => {
          console.log('[ ERROR ] Pass the audit failed!')
          console.log(err)
          this.$message({
            type: 'error',
            message: '信息审核未能成功通过!'
          })
        })
      }).catch(() => {

          // 审核取消
          this.$message({
            type: 'warning',
            message: '信息审核取消'
          })
        }
      )
    },
    onReject(row) {

      // 打开审核确认对话框
      this.$confirm('是否确认驳回？', '用户名：' + row.name, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {

        // 审核驳回
        request({
          baseURL: this.$global.baseURL,
          url: 'auths/' + row.id,
          method: 'put',
          params: {
            pass: false
          }
        }).then(resp => {
          console.log('[ INFO ] Reject the audit success!')
          console.log(resp)
          this.$message({
            type: 'success',
            message: '信息审核驳回!'
          })

          // 审核成功驳回，重新加载信息
          this.updateData()
        }).catch(err => {
          console.log('[ ERROR ] Reject the audit failed!')
          console.log(err)
          this.$message({
            type: 'error',
            message: '信息审核未能成功驳回!'
          })
        })
      }).catch(() => {

        // 审核取消
        this.$message({
          type: 'warning',
          message: '信息审核取消'
        })
      })
    }
  }
}
</script>

<style scoped>
  .btn-box {
    display: flex;
    justify-content: space-around;
  }
</style>
