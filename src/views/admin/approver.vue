<template>
  <DynamicForm
    ref="formDesign"
    :conf="mockData"
    tabName="formDesign" />
</template>

<script>
// @ is an alias to /src
import DynamicForm from "@/components/DynamicForm";
import { GET_MOCK_CONF } from '../../api'
const beforeUnload = function (e) {
  var confirmationMessage = '离开网站可能会丢失您编辑得内容';
  (e || window.event).returnValue = confirmationMessage;     // Gecko and Trident
  return confirmationMessage;                                // Gecko and WebKit
}

export default {
  name: "Home",
  props: {
    title: {
      type: String,
      default: '补卡申请'
    }
  },
  data() {
    return {
      mockData: null, // 可选择诸如 $route.param，Ajax获取数据等方式自行注入
      steps: [
        { label: "基础设置", key: "basicSetting" },
        { label: "表单设计", key: "formDesign" },
        { label: "流程设计", key: "processDesign" },
        { label: "高级设置", key: "advancedSetting" }
      ]
    };
  },
  beforeRouteEnter(to, from, next){
    window.addEventListener('beforeunload', beforeUnload)
    next()
  },
  beforeRouteLeave(to, from, next){
    window.removeEventListener('beforeunload', beforeUnload)
    next()
  },
  computed:{
    translateX () {
      return `translateX(${this.steps.findIndex(t => t.key === this.activeStep) * 100}%)`
    }
  },
  mounted() {
    GET_MOCK_CONF().then(data => {
      this.mockData = data.formData
    });

  },
  methods: {
    changeSteps(item) {
      this.activeStep = item.key;
    },
    publish() {
      const getCmpData = name => this.$refs[name].getData()
      // basicSetting  formDesign processDesign 返回的是Promise 因为要做校验
      // advancedSetting返回的就是值
      const p1 = getCmpData('formDesign')
      Promise.all([p1])
      .then(res => {
        const param = {
          formData: res[1].formData
        }
        this.sendToServer(param)
      })
      .catch(err => {
        err.target && (this.activeStep = err.target)
        err.msg && this.$message.warning(err.msg)
      })
    },
    sendToServer(param){
      this.$notify({
        title: '数据已整合完成',
        message: '请在控制台中查看数据输出',
        position: 'bottom-right'
      });
      console.log('配置数据', param)
    }
  },
  components: {
    DynamicForm
  }
};
</script>
<style lang="stylus" scoped>
$header-height = 0px;
.page {
  width: 100vw;
  height: 100vh;
  padding-top: $header-height;
  box-sizing: border-box;

  .page__header {
    width: 100%;
    height: $header-height;
    flex-center()
    justify-content: space-between;
    box-sizing: border-box;
    color: white;
    background: #3296fa;
    font-size: 14px;
    position: fixed;
    top: 0;

    .page-actions {
      height: 100%;
      text-align: center;
      line-height: $header-height;

      > div {
        height: 100%;
        padding-left: 20px;
        padding-right: 20px;
        display: inline-block;
      }

      i {
        font-size: 22px;
        vertical-align: middle;
      }
    }

    .step-tab {
      display: flex;
      justify-content: center;
      height: 100%;
      position: relative;

      >.step {
        width: 140px;
        line-height: $header-height;
        padding-left: 30px;
        padding-right: 30px;
        cursor: pointer;
        position: relative;

        &.ghost-step{
          position: absolute;
          height: $header-height;
          left: 0;
          z-index: -1;
          background: #4483f2;
          transition: transform .5s;

          &::after {
            content: '';
            border-width: 6px 6px 6px;
            border-style: solid;
            border-color: transparent transparent white;
            position: absolute;
            bottom: 0;
            left: 50%;
            margin-left: -6px;
          }
        }

        &.active >.step-index  {
          background: white;
          color: #4483f2;
        }

        >.step-index {
          display: inline-block;
          width: 18px;
          height: 18px;
          border: 1px solid #fff;
          border-radius: 8px;
          line-height: 18px;
          text-align: center;
          box-sizing: border-box;
        }
      }
    }
  }

  .page__content {
    width: 100%;
    height: 100%;
    overflow: hidden;
    box-sizing: border-box;
    background #f5f5f7
  }
}

.publish-btn {
  margin-right: 20px;
  color: #3296fa;
  padding: 7px 20px;
  border-radius: 4px;
  font-size: 14px;
}
</style>
