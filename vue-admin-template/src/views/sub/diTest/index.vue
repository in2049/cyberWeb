<template>
  <div style="display: flex;flex-direction: column;">
    <el-form :inline="true" size="small" style="padding-left: 10%">
      <el-form-item label="数据采集时间">
        <el-input style="width: 300px"
                  class="filter-item"
                  v-model="dataTime"
                  disabled
        ></el-input>
      </el-form-item>

      <el-button type="primary" @click="handleRefresh" :loading="refreshLoading" icon="el-icon-refresh" size="small">
        刷新
      </el-button>
    </el-form>

    <div class="box-main">

      <div class="box-content">
        <div v-for="(item,index) in cpData" :key="index" class="box-item" :style="`width:${itemWidth}%;`">
          <el-checkbox :label="index+1" :key="index" class="check-list"></el-checkbox>
          <div class="info">
            <div class="info-item">
              <div class="info-text">门开：</div>
              <div :class="[isDoor ? 'red' : 'green']"></div>
            </div>
            <div class="info-item">
              <div class="info-text">烟感：</div>
              <div :class="[isSmoke ? 'red' : 'green']"></div>
            </div>
            <div class="info-item">
              <div class="info-text">短路：</div>
              <div :class="[isCurrent ? 'red' : 'green']"></div>
            </div>
          </div>

        </div>

      </div>
      <div class="water" v-if="hasWater">
        <div class="info-text">水浸：</div>
        <div :class="[isWater ? 'red' : 'green']"></div>
      </div>
    </div>
  </div>
</template>

<script>


import {commandResult, deviceData, refreshDevice} from "@/api/operation";
import {renderTime} from "@/utils";

export default {
  name: 'DiTest',
  data() {
    return {
      itemWidth: 100,
      deviceInfo: JSON.parse(sessionStorage.getItem('infoQuery')),
      dataTime: '',
      //格口数据
      cpData: undefined,
      infoSoc: [],
      isDoor: false,
      isSmoke: false,
      isCurrent: false,
      //柜子
      bigData: undefined,
      refreshLoading: false,
      isWater: false,
      hasWater: false
    }
  },
  created() {
    this.getList()
  },
  methods: {

    async getList() {
      const loading = this.$loading({
        lock: true,
        text: 'Loading',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      });
      let res = await deviceData(this.deviceInfo.deviceName)
      if (res.data.success && res.data.data !== null) {
        this.dataTime = renderTime(res.data.data.dataTime)
        this.hasWater = true
        //电池
        this.cpData = Object.values(res.data.data.cpAndBatteryData)
        this.bigData =  res.data.data.cabinetDataRecordVo
        this.itemWidth = 100 / (Math.ceil(this.cpData.length / 4))
        this.cpData.forEach((item, index, arr) => {
          if (item.batteryDataRecord !== null) {
            this.isSmoke = item.compartmentDataRecordVo.sdAlm === 1
            this.isDoor = item.compartmentDataRecordVo.cpOpen === 1
            this.isCurrent = item.compartmentDataRecordVo.batSwOff === 0

          }
        })
        console.log(this.bigData)
        this.isWater = this.bigData.p1Wat === 1
        loading.close();

      } else {
        loading.close();
        this.$message({
          showClose: true,
          message: '获取失败，请刷新重试: ' + res.data.message,
          type: 'error'
        })
      }


    },

    async handleRefresh() {
      let that = this
      that.refreshLoading = true
      let messageId = ''
      let i = 0
      let interval = null
      let cmdRes = null
      let idRes = await refreshDevice(this.deviceInfo.deviceName)
      if (idRes.data !== null) {
        if (idRes.data.success) {
          messageId = idRes.data.data.messageId

          interval = setInterval(function () {
            console.log(i)
            if (i === 4) {
              that.refreshLoading = false
              clearInterval(interval)
              that.$message({
                showClose: true,
                message: '刷新失败：' + cmdRes,
                type: 'error'
              })
            } else {
              commandResult(messageId).then(res => {
                if (res.data.data !== null) {
                  if (res.data.data.cmdStatus === '2') {
                    console.log(res.data.data.cmdStatus)
                    that.refreshLoading = false
                    clearInterval(interval)
                    that.getList()
                    that.$message({
                      showClose: true,
                      message: '刷新成功！',
                      type: 'success'
                    })
                  } else {
                    cmdRes = res.data.message
                    i++
                  }
                } else {
                  i++
                }
              })
            }

          }, 3000)

        } else {
          this.refreshLoading = false
          this.$message({
            showClose: true,
            message: '刷新失败：' + idRes.data.message,
            type: 'error'
          })
        }
      }
    },

  }

}
</script>

<style scoped lang="scss">
.info {
  display: flex;
  flex-direction: column;
  justify-content: start;
  padding: 10px;

  .info-item {
    display: flex;
    flex-direction: row;
    align-items: center;
    font-size: 14px;
    padding: 10px;
  }

}

.green {
  width: 16px;
  height: 16px;
  background-color: lightgreen;
}

.red {
  width: 16px;
  height: 16px;
  background-color: lightcoral;
}

.water {
  position: absolute;
  right: 5%;
  bottom: 6%;
  display: flex;
}
</style>
