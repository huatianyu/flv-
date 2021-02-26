<template>
  <j-modal
    :title="title"
    :width="500"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="取消"
    okText="操作"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="回放日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-date-picker show-time placeholder="回放日期" format="YYYY-MM-DD HH:mm:ss" />
        </a-form-item>
        <div v-for="(item, index) in batchList" :key="index">
          <a-form-item :label="item.title" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-date-picker
              show-time
              placeholder="开始时间"
              :value="item.lzkssj ? moment(item.lzkssj, 'YYYY-MM-DD HH:mm:ss') : null"
              format="YYYY-MM-DD HH:mm:ss"
              @change="(time, value) => (item.lzkssj = value)"
            />
            -
            <a-date-picker
              show-time
              placeholder="结束时间"
              :value="item.lzjssj ? moment(item.lzjssj, 'YYYY-MM-DD HH:mm:ss') : null"
              format="YYYY-MM-DD HH:mm:ss"
              @change="(time, value) => (item.lzjssj = value)"
            />
          </a-form-item>
        </div>
      </a-form>
    </a-spin>
  </j-modal>
</template>

<script>
import { httpAction, postAction } from '@/api/manage'
import pick from 'lodash.pick'
import moment from 'moment'

import { querySelectList } from '@/api/common'

export default {
  name: 'videoArchiveModal',
  props: ['ksccList', 'choiceItem'],
  data() {
    return {
      title: '操作',
      visible: false,
      model: {},
      labelCol: {
        span: 24
      },
      wrapperCol: {
        span: 24
      },
      batchList: [{ title: '回放时间段', lzkssj: null, lzjssj: null }],
      confirmLoading: false,
      form: this.$form.createForm(this),
      validatorRules: {
        lzkssj: {
          rules: [{ required: true, message: '请输入机构标识编码!' }]
        }
      },
      url: {
        add: '/inspectMan/tDConfigIssueEntity/addBatch',
        edit: '/inspectMan/tDConfigIssueEntity/selectList ',
        issueExamConfig: '/inspectMan/inspect/issueExamConfig'
      }
    }
  },
  created() {},

  computed: {},
  methods: {
    moment,
    getlzkssj(value) {
      return value
    },
    add() {
      this.edit({})
    },

    edit(record) {
      this.visible = true
      // querySelectList(record).then(res => {
      //   if (res.success) {
      //     this.batchList = res.result

      //     if (this.batchList.length == 0) {
      //       this.batchList.push({
      //         lzkssj: null,
      //         lzjssj: null
      //       })
      //     }
      //   } else {
      //     console.log(res.message)
      //   }
      // })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      this.$confirm({
        title: '提示',
        content: '是否同步至设备？',
        okText: '同步至设备',
        cancelText: '仅保存配置',
        onOk: () => {
          const that = this
          // 触发表单验证
          this.form.validateFields((err, values) => {
            if (!err) {
              that.confirmLoading = true
              let httpurl = ''
              let method = ''

              httpurl += this.url.add
              method = 'post'

              //          ksdm;   考试代码
              // ksmc;    考试名称
              // kskssj;    考试考试时间
              // ksjssj;     考试结束时间
              // childs：  List集合
              // ccbh;     场次编号
              // ccmc;    场次名称
              // lzkssj;    录制开始时间
              // lzjssj;     录制结束时间

              let formData = {
                ksdm: this.choiceItem[0].ksdm,
                list: this.batchList
              }
              let timeIsTrue = true
              this.batchList.forEach((item, index) => {
                var lzkssj = new Date(item.lzkssj)
                var lzjssj = new Date(item.lzjssj)
                if (lzkssj > lzjssj) {
                  timeIsTrue = false
                }
              })
              if (!timeIsTrue) {
                this.$message.warning('开始时间不得大于结束时间，请校验')
                this.confirmLoading = false
                return
              }
              // //时间格式化
              // formData.lzkssj = formData.lzkssj ? formData.lzkssj.format('YYYY-MM-DD HH:mm:ss') : null
              // formData.lzjssj = formData.lzjssj ? formData.lzjssj.format('YYYY-MM-DD HH:mm:ss') : null

              httpAction(httpurl, formData, method)
                .then(res => {
                  if (res.success) {
                    that.$message.success(res.message)

                    postAction(this.url.issueExamConfig, {
                      ksdm: this.choiceItem[0].ksdm
                    }).then(res => {
                      if (res.success) {
                        // this.$message.success('下发成功')
                      }
                    })
                    that.$emit('ok')
                  } else {
                    that.$message.warning(res.message)
                  }
                })
                .finally(() => {
                  that.confirmLoading = false
                  that.close()
                })
            }
          })
        },
        onCancel: () => {
          let formData = {
            ksdm: this.choiceItem[0].ksdm,
            list: this.batchList
          }
          let method = 'post'
          httpAction(this.url.add, formData, method)
            .then(res => {
              if (res.success) {
                this.close()
                this.$message.success(res.message)
              } else {
                this.$message.warning(res.message)
              }
            })
            .finally(() => {
              this.confirmLoading = false
              this.close()
            })
        }
      })
      // -------
    },
    handleCancel() {
      this.close()
    },
    ksccChange($event) {
      this.ksmc = val
    },
    addBatch() {
      this.batchList.push({
        kscc: this.choiceItem[0].ccmc,
        lzkssj: '',
        lzjssj: ''
      })
    },

    removeBatch(val) {
      this.batchList = this.batchList.filter((item, index) => {
        return val != index
      })
    }
  }
}
</script>

<style lang="less" scoped>
.bottom-icon {
  display: flex;
  flex-direction: row;
}
</style>
