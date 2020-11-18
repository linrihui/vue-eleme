<template>
  <section class="loginContainer">
    <div class="loginInner">
      <div class="login_header">
        <h2 class="login_logo">Login</h2>
        <div class="login_header_title">
          <a href="javascript:;" :class="loginType === 'msg'?'on':''" @click="switchLogin('msg')">短信登录</a>
          <a href="javascript:;" :class="loginType === 'pwd'?'on':''" @click="switchLogin('pwd')">密码登录</a>
        </div>
      </div>
      <div class="login_content">
        <form @submit.prevent="login">
          <div :class="loginType === 'msg'?'on':''">
            <section class="login_message">
              <input type="tel" maxlength="11" placeholder="手机号" v-model="phone">
              <button :disabled="disabled" class="get_verification"
                      :class="{right_phone: rightPhone}" @click.prevent="getCode">{{computeTime>0 ? `已发送(${computeTime}s)` : '获取验证码'}}</button>
            </section>
            <section class="login_verification">
              <input type="tel" maxlength="8" placeholder="验证码" v-model="code">
            </section>
            <section class="login_hint">
              温馨提示：未注册饿了没外卖帐号的手机号，登录时将自动注册，且代表已同意
              <a href="javascript:;">《用户服务协议》</a>
            </section>
          </div>
          <div :class="loginType === 'pwd'?'on':''">
            <section>
              <section class="login_message">
                <input type="text" maxlength="11" placeholder="手机/邮箱/用户名" v-model="name">
              </section>
              <section class="login_verification">
                <input :type="showPwd?'text':'password'" maxlength="8" placeholder="密码" v-model="pwd">
                <div class="switch_button" @click="switchPwd" :class="showPwd?'on':'off'">
                  <div class="switch_circle" :class="{right:showPwd}"></div>
                  <span class="switch_text">...</span>
                </div>
              </section>
              <section class="login_message">
                <input type="text" maxlength="11" placeholder="验证码" v-model="captcha">
                <img ref="captcha" class="get_verification" @click="getCaptcha" src="http://localhost:4000/captcha" alt="captcha">
              </section>
            </section>
          </div>
          <button class="login_submit">登录</button>
        </form>
        <a href="javascript:;" class="about_us">关于我们</a>
      </div>
      <a href="javascript:" class="go_back" @click="$router.back()">
        <i class="iconfont icon-jiantou2"></i>
      </a>
    </div>
    <alert-tip :alertText="alertText" v-show="showAlert" @closeTip="closeTip"/>
  </section>
</template>

<script>
  import AlertTip from '../../components/AlertTip/AlertTip'
  import {reqSendCode, reqSmsLogin, reqPwdLogin} from '../../api'
  export default {
    data(){
      return {
        loginType:'msg',
        showPwd:false,
        computeTime: 0, //计时
        phone:'',
        code:'', //验证码
        name:'',  //登录账号
        pwd:'',
        captcha:'',
        alertText: '',
        showAlert: false,
      }
    },
    components:{
      AlertTip
    },
    methods:{
      switchLogin(type){
        this.loginType = type
      },
      switchPwd(){
        this.showPwd = !this.showPwd
      },
      /**
       * 获取验证码
       */
      async getCode(){
        // 如果当前未计时
        if(!this.computeTime){
          // 启动倒计时
          let codeText = ""
          this.computeTime = 30
          this.timer = setInterval(()=>{
            this.computeTime--
            if(this.computeTime<1){
              clearInterval(this.timer)
            }
          },1000)

          // 发送请求
          const result = await reqSendCode(this.phone)
          if(result.code === 1){
            this.popup(result.msg)
            // 停止计时
            if(this.computeTime){
              this.computeTime = 0
              clearInterval(this.timer)
            }
          }
        }
      },
      popup(alertText){
        this.showAlert = true
        this.alertText = alertText
      },
      /**
       * 登录
       */
      async login(){
        let result
        // 表单验证
        if(this.loginType === 'msg'){
          const {rightPhone,phone,code} = this
          if(!this.rightPhone){
            // 手机号不正确
            this.popup('手机号不正确')
            return
          }else if(!/^\d{6}$/.test(code)){
            // 验证码必须6位
            this.popup('验证码必须6位')
            return
          }
          // 短信登录
          result = await reqSmsLogin(phone,code);
        }else{
          const {name,pwd,captcha} = this
          if(!this.name){
            this.popup('账号不能为空')
            return
          }else if(!this.pwd){
            this.popup('密码不能为空')
            return
          }else if(!this.captcha) {
            this.popup('验证码必须4位')
            return
          }
          // 密码登录
          result = await reqPwdLogin({name,pwd,captcha});
        }

        // 停止计时
        if(this.computeTime){
          this.computeTime = 0
          clearInterval(this.timer)
        }
        // 根据结果数据处理
        console.log(result)
        if(result.code === 0){
          const user = result.data
          // 将user保存到vuex的state
          this.$store.dispatch('recordUser',user)
          // 去个人中心
          this.$router.replace('/profile')
        }else{
          // 刷新验证码
          this.getCaptcha()
          const msg = result.msg
          this.popup(msg)
        }

      },
      closeTip(){
        this.showAlert = false
        this.alertText = ''
      },
      getCaptcha(){
        this.$refs.captcha.src = 'http://localhost:4000/captcha?time='+Date.now()
      }
    },
    computed:{
      rightPhone(){
        const result = /^1\d{10}$/.test(this.phone)
        this.disabled = !result
        return result
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixins.styl"
  .loginContainer
    width 100%
    height 100%
    background #fff
    .loginInner
      padding-top 60px
      width 80%
      margin 0 auto
      .login_header
        .login_logo
          font-size 40px
          font-weight bold
          color #02a774
          text-align center
        .login_header_title
          padding-top 40px
          text-align center
          >a
            color #333
            font-size 14px
            padding-bottom 4px
            &:first-child
              margin-right 40px
            &.on
              color #02a774
              font-weight 700
              border-bottom 2px solid #02a774
      .login_content
        >form
          >div
            display none
            &.on
              display block
            input
              width 100%
              height 100%
              padding-left 10px
              box-sizing border-box
              border 1px solid #ddd
              border-radius 4px
              outline 0
              font 400 14px Arial
              &:focus
                border 1px solid #02a774
            .login_message
              position relative
              margin-top 16px
              height 48px
              font-size 14px
              background #fff
              .get_verification
                position absolute
                top 50%
                right 10px
                transform translateY(-50%)
                border 0
                color #ccc
                font-size 14px
                background transparent
                &.right_phone
                  color black
            .login_verification
              position relative
              margin-top 16px
              height 48px
              font-size 14px
              background #fff
              .switch_button
                font-size 12px
                border 1px solid #ddd
                border-radius 8px
                transition background-color .3s,border-color .3s
                padding 0 6px
                width 30px
                height 16px
                line-height 16px
                color #fff
                position absolute
                top 50%
                right 10px
                transform translateY(-50%)
                &.off
                  background #fff
                  .switch_text
                    float right
                    color #ddd
                &.on
                  background #02a774
                >.switch_circle
                  //transform translateX(27px)
                  position absolute
                  top -1px
                  left -1px
                  width 16px
                  height 16px
                  border 1px solid #ddd
                  border-radius 50%
                  background #fff
                  box-shadow 0 2px 4px 0 rgba(0,0,0,.1)
                  transition transform .3s
                  &.right
                    transform  translateX(30px)
            .login_hint
              margin-top 12px
              color #999
              font-size 14px
              line-height 20px
              >a
                color #02a774
          .login_submit
            display block
            width 100%
            height 42px
            margin-top 30px
            border-radius 4px
            background #4cd96f
            color #fff
            text-align center
            font-size 16px
            line-height 42px
            border 0
        .about_us
          display block
          font-size 12px
          margin-top 20px
          text-align center
          color #999
      .go_back
        position absolute
        top 5px
        left 5px
        width 30px
        height 30px
        >.iconfont
          font-size 20px
          color #999
</style>
