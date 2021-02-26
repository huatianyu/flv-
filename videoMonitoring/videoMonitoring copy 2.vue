<template>
  <a-card class="content-bg-color" :bordered="false">
    <div class="header">
      <a-form layout="inline">
        <a-row>
          <a-col :md="12" :lg="8" :xl="10" :xxl="5">
            <a-form-item label="请选择考场计划">
              <a-select style="width: 200px" placeholder="请选择考试计划" v-model="ksdm" @change="ksmcChange">
                <a-select-option v-for="(role, index) in ksmcList" :key="index.toString()" :value="role.ksdm">
                  {{ role.ksmc }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <div class="content-flex">
      <div class="content-left">
        <a-card :bordered="false">
          <div style="background: #fff;height: 100%; margin-top: 5px">
            <a-input-search @search="onSearch" style="width:55%;margin-right:5%" placeholder="请输入搜索关键字" />
            <a-radio-group style="width:40%" size="omitted" @change="handleSizeChange">
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
        <div class="content-right-header">
          <div class="header-left"></div>
          <div class="header-right">
            <img @click="splitScreenClick(index)" v-for="(item, index) in btnList" :src="getImgUrl(item.btnImg)" />
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
          <div
            class="content-right-item"
            v-for="(item, index) in videoList.slice(0, splitScreenStyle * splitScreenStyle)"
          >
            <div class="content-right-item-top">
              <div class="left">{{ item.record.bzhkcmc }}</div>
              <div class="right">全屏显示</div>
            </div>
            <div class="content-right-item-bottom">
              <div class="player" style="width: 100%; height: 450px; background: #000000">
                <div id="a1"></div>
                <video
                  :id="'videoElement' + item.record.key"
                  :_data="getkey(item)"
                  width="100%"
                  height="100%"
                  autoplay
                  controls
                ></video>
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

export default {
  name: 'videoMonitoring',
  components: { JDictSelectTag, JSelectDepart },
  mixins: [JeecgListMixin],
  data() {
    return {
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
      btnList: [{ btnImg: 'one-video.png' }, { btnImg: 'four-video.png' }, { btnImg: 'nine-video.png' }]
    }
  },
  mounted: function() {
    // setInterval(function() {
    //   alert(1)
    // }, 1000)
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
    getkey(item) {
      this.$nextTick(() => {
        this.createPlayer(item.record, item.record.key)
      })
    },
    // alert(that.videoList.length)

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
    createPlayer(item, key) {
      let that = this
      var play_url = ''

      if (1) {
        play_url = 'https://sf1-hscdn-tos.pstatp.com/obj/media-fe/xgplayer_doc_video/flv/xgplayer-demo-360p.flv?'
      } else {
        play_url = 'http://1011.hlsplay.aodianyun.com/demo/game.flv?'
      }

      var videoElement = document.getElementById('videoElement' + key)
      if (false) {
        $('#a1').remove()
        videoElement.src = m3u8_play_url
        videoElement.play()
      } else {
        if (flv.isSupported()) {
          $('#a1').remove()
          var flvplayer = flv.createPlayer({
            type: 'flv',
            url: play_url
          })
          this.videoList.forEach(item => {
            if (item.record.key == key) {
              item.dom = videoElement
              item.player = flvplayer
            }
          })
          flvplayer.attachMediaElement(videoElement)
          flvplayer.load()
          flvplayer.play()
        } else {
          $('#videoElement' + key).remove()
          var flashvars = {
            f: play_url,
            c: 0,
            p: 1,
            lv: 1
          }
          var params = { bgcolor: '#FFF', allowFullScreen: true, allowScriptAccess: 'always', wmode: 'transparent' }
          CKobject.embedSWF(
            '/oadmin/static/tool/ckplayer/ckplayer.swf',
            'a1',
            'ckplayer_a1',
            '100%',
            '380',
            flashvars,
            params
          )
        }
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

    ksmcChange(val) {
      this.ksdm = val
      this.loadTree()
    },

    loadData() {
      this.refresh()
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
        searchByKeywords({ keyWord: value }).then(res => {
          if (res.success) {
            that.departTree = []
            for (let i = 0; i < res.result.length; i++) {
              let temp = res.result[i]
              that.departTree.push(temp)
            }
          } else {
            that.$message.warning(res.message)
          }
        })
      } else {
        that.loadTree()
      }
    },

    onSelect(selectedKeys, e) {
      this.hiding = false
      let record = e.node.dataRef
      if (record.children != undefined) {
        return
      }
      this.currSelected = Object.assign({}, record)
      this.selectedKeys = [record.key]
      let that = this

      if (that.videoList.length) {
        for (let i = 0; i < that.videoList.length; i++) {
          if (that.videoList[i].key != record.key) {
            if (that.splitScreenStyle * that.splitScreenStyle == that.videoList.length) {
              let videoObj = { record: record, player: null, dom: null }

              // that.videoList.unshift(videoObj)

              that.videoList.splice(0, 1, videoObj)
              // alert(that.videoList.length)
            } else if (that.splitScreenStyle * that.splitScreenStyle > that.videoList.length) {
              let videoObj = { record: record, player: null, dom: null }
              that.videoList.push(videoObj)
            }

            // that.$nextTick(() => {
            //   that.createPlayer(record, record.key)
            // })
          }
        }
      } else {
        let videoObj = { record: record, player: null, dom: null }

        that.videoList.push(videoObj)
        // that.$nextTick(() => {
        //   that.createPlayer(record, record.key)
        // })
      }
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
      // if (this.videoList.length <= 4 && this.videoList.length >= 2) {
      //   if (index + 1 == 3) {
      //     this.$message.warning('无更多摄像头')
      //     return
      //   }
      // }
      // if (this.videoList.length < 2) {
      //   if (index + 1 == 2 || index + 1 == 3) {
      //     this.$message.warning('无更多摄像头')
      //     return
      //   }
      // }

      this.splitScreenStyle = index + 1
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
      padding-bottom: 10px;
      width: 100%;
      display: flex;
      justify-content: space-between;
      .header-left {
        color: #353535;
        font-size: 16px;
      }
      .header-right {
        width: 80px;
        display: flex;
        box-sizing: border-box;
        justify-content: space-between;
        img:hover {
          border: 1px solid #c1c1c1;
        }
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
