<template>
  <div class="enhance">
    <div v-if="!show">
      <a-icon class="icon" type="exclamation-circle" />
      <span>您还没有开通聊天侧边栏</span>
      <a-button @click="addShow = true" type="link">添加应用</a-button>
    </div>
    <div v-else>
      <a-icon class="icon success" type="check-circle" />
      <span>您已成功开通聊天侧边栏</span>
    </div>
    <div class="title">
      什么是聊天增强？
    </div>
    <div>
      <div>
        MoChat提供了高级快捷回复，帮助企业高效的与客户沟通，支持添加文字、图片、外链、图文链接、第三方链接等多类型消息，可对消息进行分类方便管理与查找，同时支持一键发送，帮助企业员工提高工作效率
      </div>
      <div class="title">
        如何使用
      </div>
      <div>
        开通成功后，登录企业微信，在与客户的聊天对话框右侧点击侧边栏的图标，点击开始使用后即可使用
        <img class="img" :src="require('@/assets/enhance.png')" alt="">
      </div>
      <div class="title">
        注意事项
      </div>
      <div class="bottom">
        因企业微信限制，侧边栏功能仅针对为该账号客户的外部联系人展示
      </div>
    </div>
    <settingSteps
      v-if="show"
      :chatTools="chatTools"
      :agents="agents"
      :whiteDomains="whiteDomains"
    />

    <a-modal
      title="添加应用"
      :visible="addShow"
      :confirm-loading="confirmLoading"
      :maskClosable="false"
      @ok="addApplication"
      @cancel="addShow = false"
    >
      <a-form-model
        :label-col="{ span: 4 }"
        :wrapper-col="{ span: 20}"
        :model="formData"
        :rules="rules"
        ref="ruleForm">
        <div class="name">
          <span class="star">*</span>
          <a-form-model-item class="form-item" label="应用ID" prop="wxAgentId">
            <a-input v-model="wxAgentId" class="input" placeholder="请输入应用ID"></a-input>
          </a-form-model-item>
        </div>
        <div class="name">
          <span class="star">*</span>
          <a-form-model-item class="form-item" label="应用secret" prop="wxSecret">
            <a-input v-model="wxSecret" class="input" placeholder="请输入应用secret"></a-input>
          </a-form-model-item>
        </div>
      </a-form-model>
    </a-modal>
  </div>
</template>

<script>
import settingSteps from './components/settingSteps'
import { createValidate, createFunc } from '@/utils/util'
import { addChatTool, chatTool } from '@/api/chatTool'
export default {
  components: {
    settingSteps
  },
  data () {
    const vwxAgentId = (rule, value, callback) => {
      value = this.wxAgentId
      createValidate(callback, value, '请输入应用ID')
    }
    const vwxSecret = (rule, value, callback) => {
      value = this.wxSecret
      createValidate(callback, value, '请输入应用secret')
    }
    return {
      show: false,
      addShow: false,
      formData: {},
      wxAgentId: '',
      wxSecret: '',
      confirmLoading: false,
      agents: [],
      chatTools: [],
      whiteDomains: [],
      rules: {
        wxAgentId: createFunc(vwxAgentId),
        wxSecret: createFunc(vwxSecret)
      }
    }
  },
  watch: {
    addShow (value) {
      if (!value) {
        this.$refs.ruleForm.resetFields()
        this.wxAgentId = ''
        this.wxSecret = ''
        this.confirmLoading = false
      }
    }
  },
  created () {
    this.getData()
  },
  methods: {
    async getData () {
      const { data: { agents, whiteDomains } } = await chatTool()
      if (agents) {
        this.agents = agents
        agents.forEach(item => {
          this.chatTools = this.chatTools.concat(item.chatTools)
        })
        this.show = true
      } else {
        this.show = false
      }
      this.whiteDomains = whiteDomains || []
    },
    addApplication () {
      this.$refs.ruleForm.validate(async valid => {
        if (valid) {
          this.confirmLoading = true
          try {
            const params = {
              wxAgentId: this.wxAgentId,
              wxSecret: this.wxSecret,
              type: 1
            }
            await addChatTool(params)
            this.confirmLoading = false
            this.addShow = false
            this.getData()
          } catch (e) {
            console.log(e)
            this.confirmLoading = false
          }
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.enhance {
  background-color: #fff;
  padding: 20px;
    .icon {
    color: #faad14;
    font-size: 20px;
    display: inline-block;
    margin-right: 10px;
  }
  .success {
    color: #52c41a;
  }
  .title {
    font-size: 17px;
    font-weight: 600;
    margin: 10px 0
  }
  .text {
  }
  .img {
    // height: 399px;
    width: 80%;
    margin: 20px 0
  }
  .bottom {
    width: 500px;
    padding: 2px;
    background: #fff4f4;
    color: rgba(199,56,56,.85);
    margin-bottom: 20px;
  }
}
.name {
  display: flex;
  .star {
    flex: 0 0 15px;
    // margin-right: -20px;
    font-size: 20px;
    color: red;
    padding-top: 8px;
  }
  .form-item {
    flex: 1
  }
  .input {
    max-width: 400px;
  }
}
</style>
