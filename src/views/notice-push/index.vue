<template>
  <div class="notice-push-container">

    <!-- 地区三级联动选择器 -->
    <div class="region-box">
      <p>请选择省/市/区（县）：</p>
      <v-distpicker @province="onChangeProvince" @city="onChangeCity" @area="onChangeArea"/>
    </div>

    <!-- 信息表格 -->
    <el-table :data="tableData.filter(data=>data.name.toLowerCase().includes(searchKey.toLowerCase()))"
              style="width: 100%">
      <el-table-column prop="id" label="ID" min-width="8%"/>
      <el-table-column prop="name" label="社区" min-width="68%"/>
      <el-table-column min-width="24%">

        <!-- 搜索框 -->
        <template slot="header" slot-scope="scope">
          <el-input v-model="searchKey" placeholder="输入关键词搜索"></el-input>
        </template>

        <!-- 推送通知按钮 -->
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="noticeVisible=true">推送通知</el-button>

          <!-- 推送通知对话框 -->
          <el-dialog class="notice-dialog" :title=scope.row.name :visible.sync="noticeVisible">
            <label>
              <textarea class="notice-box" placeholder="请输入通知内容" v-model="notice" autocomplete="off"></textarea>
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
            <label>
              <el-input v-model="floorQuery" placeholder="请输入楼层，以逗号分隔（为空则表示所有楼层）"></el-input>
            </label>
            <div style="height: 10px"></div>
            <label>
              <el-input v-model="roomNoQuery" placeholder="请输入房间号，以逗号分隔（为空则表示所有房间）"></el-input>
            </label>

            <!-- 对话框页脚 -->
            <div slot="footer" class="dialog-footer">
              <span class="notice-length-box">{{ notice.length }}字</span>
              <el-button @click="noticeCancel">取 消</el-button>
              <el-button type="primary" @click="noticeSubmit">确 定</el-button>
            </div>

          </el-dialog>
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
      province: '',
      city: '',
      area: '',
      notice: '',
      searchKey: '',
      noticeVisible: false,
      checkAll: true,
      checkedHouseNoData: houseNoOptions,
      houseNoData: houseNoOptions,
      isIndeterminate: true,
      floorQuery: '',
      roomNoQuery: '',
      tableData: [{
        id: 1,
        name: '憨憨社区'
      }, {
        id: 2,
        name: '嘤嘤嘤社区'
      }, {
        id: 3,
        date: '2016-05-01',
        name: '狗狗社区'
      }, {
        id: 4,
        date: '2016-05-03',
        name: '猫猫社区'
      }]
    }
  },
  methods: {
    onChangeProvince(a) {
      this.province = a.value + '-'
    },
    onChangeCity(a) {
      this.city = a.value + '-'
    },
    onChangeArea(a) {
      this.area = a.value
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
    queryToArray(query) { // 切割后都可转化为整数则返回转化后的数组，出错则返回null
      let str_array = query.split(/[,|，]/)
      let array = []
      for (let index in str_array) {
        if (/^\d+$/.test(str_array[index])) {
          array.push(parseInt(str_array[index]))
        } else {
          return null
        }
      }
      return array
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
      if (this.notice === '' || this.notice.match('^\s+$')) {
        this.$message({
          message: '内容不能为空',
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

      let temp = this.notice
      // 通知不为空，发送通知
      request({
        baseURL: this.$global.baseURL,
        url: 'notices',
        method: 'post',
        data: qs.stringify({
          context: temp,
          houseNo: this.houseNoData,
          floor: this.floorQuery !== '' ? this.queryToArray(this.floorQuery) : null,
          roomNo: this.roomNoQuery !== '' ? this.queryToArray(this.roomNoQuery) : null
        })
      }).then(resp => {
        console.log('[ INFO ] Notice pushed success!')
        console.log(resp)
        this.$message({
          message: '通知推送成功：' + temp,
          type: 'success'
        })
      }).catch(err => {
        console.log('[ ERROR ] Notice pushed failed!')
        console.log(err)
        this.$message({
          message: '通知推送失败！',
          type: 'error'
        })
      })
      this.floorQuery = ''
      this.roomNoQuery = ''
      this.notice = ''
      this.noticeVisible = false
    }
  }
}
</script>

<style>
  .region-box {
    margin: 20px;
    width: 75%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 18px;
  }

  .notice-dialog {
    width: 1300px;
  }

  .notice-box {
    margin: 0 2%;
    width: 96%;
    height: 200px;
  }

  .notice-length-box {
    margin: 0 20px;
  }
</style>
