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
                checkable
                multiple
                @select="onSelect"
                @check="onCheck"
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
          <div class="header-left">视频监控 — 高校一号考场</div>
          <div class="header-right">
            <img v-for="item in btnList" :src="getImgUrl(item.btnImg)" />
          </div>
        </div>

        <div class="content-right-video">
          <div class="content-right-item" v-for="item in videoList">
            <div class="content-right-item-top">
              <div class="left">摄像头一号</div>
              <div class="right">全屏显示</div>
            </div>
            <div class="content-right-item-bottom">
              <img width="100%" height="100%" :src="getImgUrl(item.backgroundImg)" />
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
import { queryDepartTreeList, searchByKeywords, deleteByDepartId } from '@/api/api'

import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { queryTCCourseInfoList, queryTree } from '@/api/common'
import JSelectDepart from '@/components/jeecgbiz/JSelectDepart'

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
      disableSubmit: false,
      checkedKeys: [],
      selectedKeys: [],
      autoIncr: 1,
      currSelected: {},
      ksdm: '',
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
      userIdentity: '',

      videoList: [
        { name: '省级机构数量', count: 30, backgroundImg: 'video-1.jpg' },
        { name: '市级机构数量', count: 230, backgroundImg: 'video-2.jpg' },
        { name: '区级机构数量', count: 310, backgroundImg: 'video-3.jpg' },
        { name: '区级机构数量', count: 310, backgroundImg: 'video-4.jpg' }
      ],
      btnList: [{ btnImg: 'one-video.png' }, { btnImg: 'four-video.png' }, { btnImg: 'nine-video.png' }]
    }
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
    getImgUrl(name) {
      return require('@/assets/videoMonitoring/' + name)
    },
    callback(key) {
      //console.log(key)
    },
    loadData() {
      this.refresh()
    },
    clearSelectedDepartKeys() {
      this.checkedKeys = []
      this.selectedKeys = []
      this.currentDeptId = ''
      this.$refs.DeptUserInfo.currentDeptId = ''
      this.$refs.DeptRoleInfo.currentDeptId = ''
    },
    loadTree() {
      var that = this
      that.treeData = []
      that.departTree = []
      // queryMyDepartTreeList().then(res => {
      // queryTree({ ksdm: this.ksdm }).then(res => {
      //   if (res.success && res.result) {
      //     for (let i = 0; i < res.result.length; i++) {
      //       let temp = res.result[i]
      //       that.treeData.push(temp)
      //       that.departTree.push(temp)
      //       that.setThisExpandedKeys(temp)
      //       // console.log(temp.id)
      //     }
      //     this.loading = false
      //   }

      //   that.userIdentity = '2'
      // })

      queryTree({ ksdm: this.ksdm }).then(res => {
        if (res.success) {
          //部门全选后，再添加部门，选中数量增多
          this.allTreeKeys = []

          for (let i = 0; i < res.result.length; i++) {
            let temp = res.result[i]
            that.treeData.push(temp)
            that.departTree.push(temp)
            that.setThisExpandedKeys(temp)
            that.getAllKeys(temp)
            // console.log(temp.id)
          }
          this.loading = false
        }
      })
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
    setThisExpandedKeys(node) {
      if (node.childrenList && node.childrenList.length > 0) {
        this.iExpandedKeys.push(node.code)
        for (let a = 0; a < node.childrenList.length; a++) {
          this.setThisExpandedKeys(node.childrenList[a])
        }
      }
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
    onExpand(expandedKeys) {
      // console.log('onExpand', expandedKeys)
      // if not set autoExpandParent to false, if children expanded, parent can not collapse.
      // or, you can remove all expanded children keys.
      this.iExpandedKeys = expandedKeys
      this.autoExpandParent = false
    },

    onSearch(value) {
      let that = this
      if (value) {
        searchByKeywords({ keyWord: value, myDeptSearch: '1' }).then(res => {
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
    onCheck(checkedKeys, e) {
      let record = e.node.dataRef
      // console.log('onCheck', checkedKeys, e);
      this.checkedKeys = []
      // if (e.checked === true) {
      this.currentDeptId = record.id
      this.checkedKeys.push(record.id)

      this.$refs.DeptBaseInfo.open(record)
      this.$refs.DeptUserInfo.open(record)
      this.$refs.DeptRoleInfo.open(record)
      // }
      // else {
      //   this.checkedKeys = [];
      //   this.$refs.DeptBaseInfo.clearForm();
      //   this.$refs.DeptUserInfo.clearList();
      // }

      this.hiding = false
      // this.checkedKeys = checkedKeys.checked
    },
    onSelect(selectedKeys, e) {
      console.log('selected', selectedKeys, e)
      this.hiding = false
      let record = e.node.dataRef
      console.log('onSelect-record', record)
      this.currSelected = Object.assign({}, record)
      this.model = this.currSelected
      this.selectedKeys = [record.key]
      this.model.parentId = record.parentId
      this.setValuesToForm(record)
      this.$refs.departAuth.show(record.id)
    },
    onCheck(checkedKeys, info) {
      console.log('onCheck', checkedKeys, info)
      this.hiding = false
      //this.checkedKeys = checkedKeys.checked
      // <!---- author:os_chengtgen -- date:20190827 --  for:切换父子勾选模式 =======------>
      if (this.checkStrictly) {
        this.checkedKeys = checkedKeys.checked
      } else {
        this.checkedKeys = checkedKeys
      }
      // <!---- author:os_chengtgen -- date:20190827 --  for:切换父子勾选模式 =======------>
    },
    ksmcChange(val) {
      this.ksdm = val
      this.loadTree()
    }
  },
  computed: {}
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
        justify-content: space-between;
      }
    }
  }
  .content-right-video {
    width: 100%;
    height: 1000px;
    display: grid;
    gap: 20px;
    grid-template-columns: repeat(2, 1fr);
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
