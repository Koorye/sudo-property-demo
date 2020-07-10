<template>
  <div class="notice-push-container">

    <!-- 推送通知按钮 -->
    <template>
      <div class="btn-box">
        <el-tag class="btn-box-title" type="success">控制面板</el-tag>
        <el-button type="primary" size="small" @click="noticeVisible=true">推送通知</el-button>
      </div>

      <!-- 推送通知对话框 -->
      <el-dialog class="notice-dialog" title="推送通知" :visible.sync="noticeVisible">
        <el-input v-model="title" placeholder="请输入标题(必填)"></el-input>
        <div style="height: 10px"></div>
        <label>
          <textarea class="notice-box"
                    placeholder="请输入通知内容(必填)"
                    v-model="notice"
                    autocomplete="off">
          </textarea>
        </label>

        <!-- houseNo多选框 -->
        <span>请选择楼号：&nbsp;</span>
        <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选
        </el-checkbox>
        <div style="height: 10px"></div>
        <el-checkbox-group v-model="checkedHouseNoData" @change="handleCheckedHouseNoChange">
          <el-checkbox v-for="houseNo in houseNoData" :label="houseNo" :key="houseNo">{{houseNo}}</el-checkbox>
        </el-checkbox-group>
        <div style="height: 10px"></div>

        <!-- floor和roomNo输入框 -->
        <el-input v-model="floorQuery" placeholder="请输入楼层，以逗号分隔（为空则表示所有楼层）"></el-input>
        <div style="height: 10px"></div>
        <el-input v-model="roomNoQuery" placeholder="请输入房间号，以逗号分隔（为空则表示所有房间）"></el-input>

        <!-- 对话框页脚 -->
        <div slot="footer" class="dialog-footer">
          <span class="notice-length-box">{{ notice.length }}字</span>
          <el-button @click="noticeCancel">取 消</el-button>
          <el-button type="primary" @click="noticeSubmit">确 定</el-button>
        </div>
      </el-dialog>
    </template>

    <!-- 历史通知表格 -->
    <el-table :data="tableData.filter(data => !searchKey
          || data.title.toLowerCase().includes(searchKey.toLowerCase())
          || data.context.toLowerCase().includes(searchKey.toLowerCase())
          || data.time.toLowerCase().includes(searchKey.toLowerCase()))"
              style="width: 100%"
              height="500"
              :default-sort="{prop:'noticeId',order:'descending'}"
              sortable>
      <el-table-column prop="noticeId" label="ID" min-width="4%"/>
      <el-table-column prop="title" label="标题" min-width="16%"/>
      <el-table-column prop="context" label="内容" min-width="26%"/>
      <el-table-column prop="houseNo" label="楼号" min-width="8%"/>
      <el-table-column prop="floor" label="楼层号" min-width="8%"/>
      <el-table-column prop="roomNo" label="房间号" min-width="12%"/>
      <el-table-column prop="readNum" label="阅读次数" min-width="10%"/>
      <el-table-column prop="time" min-width="20%">
        <!-- 搜索框 -->
        <template slot="header" slot-scope="scope">
          <el-input v-model="searchKey" placeholder="输入关键词(标题/内容/日期)搜索"></el-input>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import request from '../../utils/request'
import qs from 'qs'

const houseNoOptions = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
export default {
  data() {
    return {
      title: '',
      notice: '',
      searchKey: '',
      noticeVisible: false,
      checkAll: true,
      checkedHouseNoData: houseNoOptions,
      houseNoData: houseNoOptions,
      isIndeterminate: true,
      floorQuery: '',
      roomNoQuery: '',
      tableData: []
    }
  },
  created() {
    this.getNotices()
  },
  methods: {
    getNotices() {
      request({
        baseURL: this.$global.baseURL,
        url: 'notices',
        method: 'get'
      }).then(resp => {
        console.log('[ INFO ] Get notices success!')
        console.log(resp.data)

        let array = []
        for (let index in resp.data) {
          array.push({
            'noticeId': resp.data[index].noticeId != null ? parseInt(resp.data[index].noticeId) : 'NULL',
            'time': resp.data[index].time != null ? resp.data[index].time : 'NULL',
            'title': resp.data[index].title != null ? resp.data[index].title : 'NULL',
            'context': resp.data[index].context != null ? resp.data[index].context : 'NULL',
            'houseNo': resp.data[index].houseNo != null ? this.arrayToQuery(resp.data[index].houseNo) : 'NULL',
            'floor': resp.data[index].floor != null ? this.arrayToQuery(resp.data[index].floor) : 'NULL',
            'roomNo': resp.data[index].roomNo != null ? this.arrayToQuery(resp.data[index].roomNo) : 'NULL',
            'readNum': resp.data[index].readNum != null ? resp.data[index].readNum : 'NULL'
          })
        }
        this.tableData = array
      }).catch(err => {
        console.log('[ ERROR ] Get notices failed!')
        console.log(err)
      })
    },
    handleCheckAllChange(val) {
      this.checkedHouseNoData = val ? houseNoOptions : []
      this.isIndeterminate = false
    },
    handleCheckedHouseNoChange(value) {
      let checkedCount = value.length
      this.checkAll = checkedCount === this.houseNoData.length
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.houseNoData.length
    },
    changeQuote(query) { // 将分割字符换成逗号
      return query.replace(/[，|\s]/, ',')
    },
    queryToArray(query) { // 切割后都可转化为整数则返回转化后的字符串数组，出错则返回null
      let str_array = query.split(/[,|，]/)
      let array = []
      for (let index in str_array) {
        if (/^\d+$/.test(str_array[index])) {
          array.push(parseInt(str_array[index]).toString())
        } else {
          return null
        }
      }
      return array
    },
    arrayToQuery(array) { // 将数组合并为字符串，以逗号分割
      let str = ''
      for (let i = 0; i < array.length; ++i) {
        if (array[i]) {
          if (i !== array.length - 1) {
            str += array[i] + ', '
          } else {
            str += array[i]
          }
        }
      }
      return str
    },
    noticeCancel() {
      this.$message({
        message: '通知推送取消',
        type: 'warning'
      })
      this.notice = ''
      this.noticeVisible = false
    },
    noticeSubmit() {
      // 通知为空
      if (this.title === '' || this.title.match('^\s+$')) {
        this.$message({
          message: '标题不能为空',
          type: 'error'
        })
        return
      }

      if (this.notice === '' || this.notice.match('^\s+$')) {
        this.$message({
          message: '正文内容不能为空',
          type: 'error'
        })
        return
      }

      // 转化楼层和房间号
      let floorData = this.queryToArray(this.floorQuery)
      let roomNoData = this.queryToArray(this.roomNoQuery)

      if (this.floorQuery !== '' && floorData == null) {
        this.$message({
          message: '楼层出现非法字符',
          type: 'error'
        })
        return
      }

      if (this.roomNoQuery !== '' && roomNoData == null) {
        this.$message({
          message: '房间号出现非法字符',
          type: 'error'
        })
        return
      }

      let houseNoArray = []
      if (this.checkAll) {  // 全选
        houseNoArray = null
      } else {
        if (!this.isIndeterminate) {  // 全不选
          this.$message({
            message: '请选择楼号',
            type: 'error'
          })
          return
        } else {
          houseNoArray = this.changeQuote(this.arrayToQuery(this.checkedHouseNoData))
        }
      }

      let that = this
      // 通知不为空，发送通知
      request({
        baseURL: this.$global.baseURL,
        url: 'notices',
        method: 'post',
        data: qs.stringify({
          title: that.title,
          context: that.notice,
          houseNo: houseNoArray,
          floor: that.floorQuery !== '' ? that.changeQuote(that.floorQuery) : null,
          roomNo: that.roomNoQuery !== '' ? that.changeQuote(that.roomNoQuery) : null
        })
      }).then(resp => {
        console.log('[ INFO ] Notice pushed success!')
        console.log(resp)
        this.$message({
          message: '通知推送成功：' + that.notice,
          type: 'success'
        })

        // 重新加载所有通知
        this.getNotices()
      }).catch(err => {
        console.log('[ ERROR ] Notice pushed failed!')
        console.log(err)
        this.$message({
          message: '通知推送失败！',
          type: 'error'
        })
      })
      this.title = ''
      this.floorQuery = ''
      this.roomNoQuery = ''
      this.notice = ''
      this.noticeVisible = false
    }
  }
}
</script>

<style scoped>
  .btn-box {
    padding: 20px;
  }

  .btn-box-title {
    margin: 10px;
  }

  .notice-dialog {
    width: 1300px;
  }

  .notice-box {
    width: 100%;
    height: 150px;
    padding: 8px 16px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }

  .notice-length-box {
    margin: 0 20px;
  }

  textarea::-webkit-input-placeholder {
    color: #ccc;
  }

  textarea:focus {
    border: 1px solid #1890FF;
    outline: none;
    transition: border 1s;
  }
</styles>
