<template>
  <a-card class="content-bg-color" :bordered="false">
    <div class="header">
      <a-form layout="inline">
        <a-row>
          <!-- <a-col :md="12" :lg="8" :xl="10" :xxl="5">
            <a-form-item label="请选择考场计划">
              <a-select style="width: 200px" placeholder="请选择考试计划" v-model="ksdm" @change="ksmcChange">
                <a-select-option v-for="(role, index) in ksmcList" :key="index.toString()" :value="role.ksdm">
                  {{ role.ksmc }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col> -->
          <planSelect @ksdmChange="ksmcChange"></planSelect>
        </a-row>
      </a-form>
    </div>
    <div class="content-flex">
      <div class="content-left">
        <a-card :bordered="false">
          <div style="background: #fff;height: 100%; margin-top: 5px">
            <a-input-search @search="onSearch" style="width:55%;margin-right:5%" placeholder="请输入搜索关键字" />
            <a-radio-group style="width:40%" size="omitted" defaultValue="large" @change="handleSizeChange">
              <a-radio-button value="large">
                在线
              </a-radio-button>
              <a-radio-button value="default">
                全部
              </a-radio-button>
            </a-radio-group>
            <!-- 树-->

            <template v-if="userIdentity === '2' && departTree.length > 0">
              <!--组织机构-->
              <a-tree
                multiple
                @select="onSelect"
                @rightClick="rightHandle"
                :selectedKeys="selectedKeys"
                :checkedKeys="checkedKeys"
                :treeData="departTree"
                :checkStrictly="checkStrictly"
                :expandedKeys="iExpandedKeys"
                :autoExpandParent="autoExpandParent"
                @expand="onExpand"
              />
            </template>
            <div style="margin-top: 24px;" v-else-if="userIdentity === '2' && departTree.length == 0">
              <h3><span>您的部门下暂无有效部门信息</span></h3>
            </div>
            <div style="margin-top: 24px;" v-else><h3>普通员工暂此权限</h3></div>
          </div>
        </a-card>
      </div>
      <div class="content-right">
        <!-- <div class="content-right-header">
          <div class="header-left">
            <a-button type="primary" @click="videoPlaybackClick">录像回放</a-button>
          </div>
          <div class="header-right">
            <a-button icon="import" @click="downLoadClick">下载视频</a-button>
          </div>
        </div> -->
        <div class="content-right-header">
          <div class="header-left">
            <a-row>
              <a-col :span="4">
                回放设置
              </a-col></a-row
            >
            <a-row style="margin-left: 100px">
              <a-col :span="9">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="开始时间">
                  <a-date-picker show-time v-model="kssj" format="YYYY-MM-DD HH:mm:ss" />
                </a-form-item>
              </a-col>
              <a-col :span="9">
                <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="结束时间">
                  <a-date-picker v-model="jssj" show-time format="YYYY-MM-DD HH:mm:ss" />
                </a-form-item>
              </a-col>
            </a-row>
          </div>
          <div class="header-right">
            <a-button style="margin-right:10px" type="primary" @click="videoPlaybackClick(videoList[0], kssj, jssj)"
              >回放</a-button
            >
            <a-button @click="downloadFlv(videoList[0], kssj, jssj, url.exportUrl)">下载</a-button>
          </div>
        </div>

        <!-- <div class="player" style="width: 100%; height: 450px; background: #000000">
          <div id="a1"></div>
          <video :id="'videoElement' + 0" width="100%" height="100%" autoplay controls></video>
        </div> -->
        <div
          class="content-right-video"
          :style="{
            gridTemplateColumns: getSplitScreenColumesStyle(splitScreenStyle),
            gridTemplateRows: getSplitScreenRowsStyle(splitScreenStyle)
          }"
        >
          <div class="content-right-item" v-for="(item, index) in videoList">
            <div class="content-right-item-top">
              <div class="left">{{ item.record.bzhkcmc }}</div>
              <!-- <div class="right">全屏显示</div> -->
            </div>
            <div class="content-right-item-bottom">
              <div class="player" style="width: 100%; height: 450px; background: #000000">
                <keepAlive>
                  <video :id="'videoElement' + item.record.key" width="100%" height="100%" controls></video>
                </keepAlive>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </a-card>
</template>

<script>
import JDictSelectTag from '@/components/dict/JDictSelectTag'
import { queryMyDepartTreeList } from '@/api/api'
import { searchByKeywords } from '@/api/api'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { queryTCCourseInfoList, queryTree, queryKcBykd } from '@/api/common'
import JSelectDepart from '@/components/jeecgbiz/JSelectDepart'
import $ from 'jquery'
import flv from 'flv.js'
import CKobject from '@/utils/ckplayer.js'
import planSelect from '@/components/customComponents/planSelect'

export default {
  name: 'videoPlayback',
  components: { JDictSelectTag, JSelectDepart, planSelect },
  mixins: [JeecgListMixin],
  data() {
    return {
      kssj: null,
      jssj: null,
      ksmcList: [],
      currentDeptId: '',
      iExpandedKeys: [],
      loading: false,
      autoExpandParent: true,
      currFlowId: '',
      currFlowName: '',
      disable: true,
      treeData: [],
      visible: false,
      departTree: [],
      rightClickSelectedKey: '',
      hiding: true,
      model: {},
      dropTrigger: '',
      depart: {},
      checkedKeys: [],
      autoIncr: 1,
      currSelected: {},
      ksdm: '',

      disableSubmit: false,
      selectedKeys: [],
      autoIncr: 1,
      allTreeKeys: [],
      checkStrictly: true,
      form: this.$form.createForm(this),
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      graphDatasource: {
        nodes: [],
        edges: []
      },
      flvplayers: {},
      userIdentity: '2',
      splitScreenStyle: 1,
      videoList: [
        // { name: '省级机构数量', count: 30, backgroundImg: 'video-1.jpg' },
        // { name: '市级机构数量', count: 230, backgroundImg: 'video-2.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-3.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-4.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-3.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-4.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-3.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-4.jpg' },
        // { name: '区级机构数量', count: 310, backgroundImg: 'video-4.jpg' }
      ],
      url: {
        exportUrl: ''
      },
      btnList: [{ btnImg: 'one-video.png' }, { btnImg: 'four-video.png' }, { btnImg: 'nine-video.png' }]
    }
  },
  mounted: function() {
    // this.createPlayer(null, 1)
  },
  watch: {
    value(value) {
      console.log(value)
    }
  },
  created() {
    queryTCCourseInfoList(Object.assign({ status: 1 })).then(res => {
      if (res.success) {
        this.ksmcList = res.result.records
      }
    })
  },
  methods: {
    IsPC() {
      var userAgentInfo = navigator.userAgent
      var Agents = new Array('Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod')
      var flag = true
      for (var v = 0; v < Agents.length; v++) {
        if (userAgentInfo.indexOf(Agents[v]) > 0) {
          flag = false
          break
        }
      }
      return flag
    },
    createPlayer(item, index) {
      let that = this
      // var play_url = 'http://192.168.1.22:280/gb28181/25.flv?'
      var play_url = item.playUrl + '?'
      var videoElement = document.getElementById('videoElement' + index)
      if (flv.isSupported()) {
        // $('#a1').remove()
        this.flvplayers[index] = flv.createPlayer({
          type: 'flv',
          url: play_url
        })
        this.url.exportUrl = play_url
        this.flvplayers[index].attachMediaElement(videoElement)
        this.flvplayers[index].load()
        this.flvplayers[index].play()
      }
    },
    pausemix(flvPlayer) {
      if (flvPlayer) {
        // flvPlayer.pause()
        flvPlayer.unload()
        flvPlayer.detachMediaElement()
        flvPlayer.destroy()
        flvPlayer = null
      }
    },
    getImgUrl(name) {
      return require('@/assets/videoMonitoring/' + name)
    },

    ksmcChange(choiceItem) {
      this.ksdm = choiceItem.ksdm

      this.loadTree()
    },

    loadData() {
      // 不做处理
    },
    loadTree() {
      var that = this
      that.treeData = []
      that.departTree = []
      queryTree({ ksdm: this.ksdm }).then(res => {
        if (res.success) {
          //部门全选后，再添加部门，选中数量增多
          this.allTreeKeys = []
          for (let i = 0; i < res.result.length; i++) {
            let temp = res.result[i]
            that.departTree.push(that.initTreeData(temp))
            that.setThisExpandedKeys(temp)
          }
          this.loading = false
        }
      })
    },
    setThisExpandedKeys(node) {
      if (node.children && node.children.length > 0) {
        this.iExpandedKeys.push(node.key)
        for (let a = 0; a < node.children.length; a++) {
          this.setThisExpandedKeys(node.children[a])
        }
      }
    },
    initTreeData(node) {
      let temp1 = JSON.stringify(node)
      let temp2 = temp1.replace(/code/g, 'key')
      let temp3 = temp2.replace(/childrenList/g, 'children')
      let temp4 = temp3.replace(/name/g, 'title')
      let obj = JSON.parse(temp4)
      return obj
    },
    refresh() {
      this.loading = true
      this.loadTree()
    },
    // 右键操作方法
    rightHandle(node) {
      this.dropTrigger = 'contextmenu'
      console.log(node.node.eventKey)
      this.rightClickSelectedKey = node.node.eventKey
      this.rightClickSelectedOrgCode = node.node.dataRef.orgCode
    },
    onExpand(expandedKeys) {
      console.log('onExpand', expandedKeys)
      // if not set autoExpandParent to false, if children expanded, parent can not collapse.
      // or, you can remove all expanded children keys.
      this.iExpandedKeys = expandedKeys
      this.autoExpandParent = false
    },

    onSearch(value) {
      let that = this
      if (value) {
        searchByKeywords({ keyword: value, ksdm: this.ksdm }).then(res => {
          if (res.success) {
            that.departTree = []
            for (let i = 0; i < res.result.length; i++) {
              let temp = res.result[i]

              that.departTree.push(that.initTreeData(temp))
              that.setThisExpandedKeys(temp)
            }
          } else {
            that.$message.warning(res.message)
          }
        })
      }
    },

    onSelect(selectedKeys, e) {
      this.hiding = false
      let record = e.node.dataRef
      if (record.children != undefined) {
        return
      }

      let isPlaying = false
      for (let i = 0; i < this.videoList.length; i++) {
        if (this.videoList[i].record.key == record.key) {
          isPlaying = true
        }
      }
      if (isPlaying) {
        this.$message.warning('当前视频正在播放')
        return
      }
      this.currSelected = Object.assign({}, record)
      this.selectedKeys = [record.key]
      let that = this

      // let tempVideoList = that.videoList.slice(0)
      let play_url = record.playUrl
      // if (Math.floor(Math.random() * 10) % 2 == 0) {
      //   play_url = 'https://sf1-hscdn-tos.pstatp.com/obj/media-fe/xgplayer_doc_video/flv/xgplayer-demo-360p.flv?'
      // } else {
      //   play_url = 'http://1011.hlsplay.aodianyun.com/demo/game.flv?'
      // // }
      // play_url = 'https://sf1-hscdn-tos.pstatp.com/obj/media-fe/xgplayer_doc_video/flv/xgplayer-demo-360p.flv?'
      var flvplayer = flv.createPlayer({
        type: 'flv',
        url: play_url
      })
      let videoObj = { record: record, player: flvplayer, dom: null }
      // that.videoList.push(videoObj)

      that.videoList.splice(0, 1, videoObj)

      // that.$nextTick(() => {
      //   for (var i = 0; i < that.videoList.length; i++) {
      //     let item = that.videoList[i]

      //     var videoElement = document.getElementById('videoElement' + item.record.key)

      //     if (!item.player._mediaElement) {
      //       item.player.attachMediaElement(videoElement)
      //       item.player.load()
      //       item.player.play()
      //     } else {
      //       // item.player.pause()
      //     }
      //   }
      // })
    },

    getAllKeys(node) {
      // console.log('node',node);
      this.allTreeKeys.push(node.key)
      if (node.children && node.children.length > 0) {
        for (let a = 0; a < node.children.length; a++) {
          this.getAllKeys(node.children[a])
        }
      }
    },
    splitScreenClick(index) {
      if (this.videoList.length <= 4 && this.videoList.length >= 2) {
        if (index + 1 == 3) {
          this.$message.warning('无更多摄像头')
          return
        }
      }
      if (this.videoList.length < 2) {
        if (index + 1 == 2 || index + 1 == 3) {
          this.$message.warning('无更多摄像头')
          return
        }
      }

      this.splitScreenStyle = index + 1
    },
    videoPlaybackClick(video, kssj, jssj) {
      if (!video) {
        this.$warning({ title: '温馨提示', content: '请选择考场' })
        return
      }
      if (kssj && jssj) {
        if (kssj > jssj) {
          this.$message.warning('开始时间大于结束时间')
          return
        }
        let play_url =
          video.record.replayUrl +
          '&' +
          'starttime=' +
          kssj.format('YYYYMMDDHHmmss') +
          '&endtime=' +
          jssj.format('YYYYMMDDHHmmss')

        var flvplayer = flv.createPlayer({
          type: 'flv',
          url: play_url
        })
        this.url.exportUrl = play_url

        let videoObj = { record: video.record, player: flvplayer, dom: null }
        // that.videoList.push(videoObj)

        this.videoList.splice(0, 1, videoObj)

        this.$nextTick(() => {
          for (var i = 0; i < this.videoList.length; i++) {
            let item = this.videoList[i]

            var videoElement = document.getElementById('videoElement' + item.record.key)

            if (!item.player._mediaElement) {
              item.player.attachMediaElement(videoElement)
              item.player.load()
              item.player.play()
            } else {
              // item.player.pause()
            }
          }
        })
      } else {
        this.$warning({ title: '温馨提示', content: '请输入完整开始结束时间' })
      }
    },
    downloadFlv(video, kssj, jssj, filePath) {
      if (!video) {
        this.$warning({ title: '温馨提示', content: '请选择考场' })
        return
      }
      if (kssj && jssj) {
        if (kssj > jssj) {
          this.$message.warning('开始时间大于结束时间')
          return
        }
        fetch(filePath)
          .then(res => res.blob())
          .then(blob => {
            const a = document.createElement('a')
            document.body.appendChild(a)
            a.style.display = 'none'
            // 使用获取到的blob对象创建的url
            const url = window.URL.createObjectURL(blob)
            a.href = url
            // 指定下载的文件名
            a.download = '录像记录.flv'
            a.click()
            document.body.removeChild(a)
            // 移除blob对象的url
            window.URL.revokeObjectURL(url)
          })
      } else {
        this.$warning({ title: '温馨提示', content: '请输入完整开始结束时间' })
      }
    },
    getSplitScreenColumesStyle(index) {
      switch (this.splitScreenStyle) {
        case 1:
          return 'repeat(1, 1fr)'
        case 2:
          return 'repeat(2, 1fr)'
        case 3:
          return 'repeat(3, 1fr)'
      }
    },
    getSplitScreenRowsStyle(index) {
      switch (this.splitScreenStyle) {
        case 1:
          return 'repeat(1, 1fr)'
        case 2:
          return 'repeat(2, 1fr)'
        case 3:
          return 'repeat(3, 1fr)'
      }
    },
    handleSizeChange(value) {
      this.$warning({ title: '温馨提示', content: '功能开发中。。。' })
    }
  },
  computed: {},
  destroyed() {
    this.videoList.forEach(element => {
      this.pausemix(element.player)
    })
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
.content-bg-color {
  background-color: #f0f2f5;
  .header {
    padding: 10px 20px;
    margin-bottom: 20px;
  }
  .header-flex-box {
    display: flex;
    flex-direction: row;
    .header-flex-box-right {
      display: flex;
      width: 70%;
      flex-direction: row;
      float: right;
      justify-content: flex-end;
    }
    .header-flex-box-left {
      width: 30%;
      .flex-box {
        background-color: #ffffff;
        display: flex;
        margin: 20px 0px;
        height: 130px;
        width: 388px;
        .left-box {
          width: 70%;
          padding: 20px;
          :first-child {
            color: #333333;
            font-size: 15px;
          }
          :nth-child(2) {
            color: #014198;
            font-size: 40px;
          }
        }

        .right-box {
          width: 30%;
          height: 100%;
          background: url('../../assets/organizationManagement/right-bg.png') no-repeat;
          background-size: 100% 100%;
        }
      }
    }
  }
}
.content-flex {
  display: flex;
  flex-direction: row;
  width: 100%;
  .content-left {
    margin-right: 20px;
    width: 25%;
  }
  .content-right {
    padding: 20px;
    background-color: #ffffff;
    width: 75%;
    display: flex;
    flex-direction: column;
    .content-right-header {
      // padding-bottom: 10px;
      width: 100%;
      display: flex;
      justify-content: space-between;
      .header-left {
        flex-direction: row;
        justify-content: space-around;
        height: 100px;
        width: 80%;
        color: #353535;
        font-size: 16px;
      }
      .header-right {
        width: 80px;
        margin: 30px;
        display: flex;
        flex-direction: row;
        justify-content: space-around;
      }
    }
  }
  .content-right-video {
    width: 100%;
    // height: 1000px;
    display: grid;
    gap: 20px;
    // grid-template-columns: repeat(2, 1fr);
    grid-template-columns: repeat(3, 33.33%);
    grid-template-rows: repeat(3, 33.33%);
    .content-right-item {
      background-color: #e6f1ff;
      width: 100%;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      border-radius: 10px;
      .content-right-item-top {
        padding: 10px;
        display: flex;
        flex-direction: row;
        width: 100%;
        justify-content: space-between;
        .right {
          color: #7f7f7f;
          font-size: 12px;
        }
        .right::before {
          content: '☒';
          color: #7f7f7f;
        }
        .left {
          color: #014198;
          font-size: 14px;
        }
      }
      .content-right-item-bottom {
        padding: 10px;
        width: 100%;
        height: 100%;
        img {
          border-radius: 10px;
        }
      }
    }
  }
}
</style>
