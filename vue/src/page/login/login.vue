<template>
  <a-form-model ref='form' :rules='rules' class='login-form' :wrapper-col="wrapperCol">
    <h2>登录</h2>
    <a-form-model-item prop='account'>
      <a-input placeholder='手机号' v-model='umUser.tel' />
    </a-form-model-item>
    <a-form-model-item prop='psw'>
      <a-input v-model='umUser.password' type='password' placeholder='密码' />
    </a-form-model-item>
    <a-form-model-item class='check'>
        <span @click='change("forget")'>忘记密码</span>
    </a-form-model-item>
    <a-form-model-item style='margin-bottom:0px'>
      <a-radio-group
        v-model="ident">
<!--        <a-radio :style="radioStyle" :value="1">用户(已停用)</a-radio>-->
        <a-radio :style="radioStyle" :value="2">管理员</a-radio>
        <a-radio :style="radioStyle" :value="3">用户</a-radio>
      </a-radio-group>
    </a-form-model-item>
    <a-form-model-item class='center' style="margin: 5px auto 29px auto">
      <a-button type="primary" block @click='userLogin()'>登录</a-button>
    </a-form-model-item>
  </a-form-model>
</template>

<script>
  export default {
    data () {
      return {
        ident: 3,
        wrapperCol: { span: 24 },
        umUser:{
          id:'',
          tel:'',
          password:'',
        },
        umLogin:{
          id:'',
          password:''
        },
        adminLogin:{
          id:'',
          password:''
        },
        salt:'',
        rules: {
          tel: {
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          },
          password: {
            required: true,
            message: '请输入登录密码',
            trigger: 'blur'
          }
        },
        radioStyle: {
          display: 'block',
          height: '30px',
          lineHeight: '30px'
        },
      }
    },
    mounted() {
      if(this.$cookies.isKey('vid') === true)
        this.$router.push('/visitor/center/info')
      if(this.$cookies.isKey('aid') === true)
        this.$router.push('/admin/list')
    },
    methods: {
      userLogin () {
        if(this.ident === 2){
          let _this = this
          axios.post('http://localhost:8181/admin/telToId/',this.umUser.tel).then(function (response){
            _this.umUser.id = response.data

            axios.post('http://localhost:8181/admin/querySalt/',_this.umUser.id).then(function (response){
              _this.salt = response.data
              _this.adminLogin.id = _this.umUser.id
              _this.adminLogin.password = _this.$sha256(_this.umUser.password + _this.salt)

              axios.put('http://localhost:8181/admin/login/',_this.adminLogin).then(function (response){
                if(!response.data)
                  alert('用户名或密码错误，请重新输入！')
                else{
                  // 路由注册
                  alert('登录成功！')
                  _this.$cookies.set('aid',_this.umUser.id)
                  _this.$router.push({
                    path: '/admin/list'
                  })
                }
              })
            })
          })
        }
        else if(this.ident === 3){
          //加密用户登录
          let _this = this
          axios.post('http://localhost:8181/umUser/telToId/',this.umUser.tel).then(function (response){
            _this.umUser.id = response.data

            axios.post('http://localhost:8181/umLogin/querySalt/',_this.umUser.id).then(function (response){
              _this.salt = response.data
              _this.umLogin.id = _this.umUser.id
              _this.umLogin.password = _this.$sha256(_this.umUser.password + _this.salt)

              axios.put('http://localhost:8181/umLogin/login/',_this.umLogin).then(function (response){
                if(!response.data)
                  alert('用户名或密码错误，请重新输入！')
                else{
                  // 路由注册
                  alert('登录成功！')
                  _this.$cookies.set('vid',_this.umUser.id)
                  _this.$router.push({
                    path: '/visitor/center/info'
                  })
                }
              })
            })
          })
        }

      },
      change (type) {
        this.$emit('changeType', type)
      }
    },
    submit () {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          console.log(valid)
          // 虚拟接口，需改成实际接口
          this.http.get('data.json').then(() => {
            // 路由注册
            this.$router.push({
              path: '/visitor/center/info'
            })
          })
        }
      })
    },
  }
</script>
