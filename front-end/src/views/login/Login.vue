<template>
  <div class="login">
    <h2 class="active">Login</h2>
    <form @submit.prevent="onSubmit">
      <input class="text" v-model="loginForm.username" type="text"
             value="" id="login_username" required/>
      <span>username</span><span v-show="loginForm.usernameError"
                                 class="error-tip">{{ loginForm.usernameError }}</span>
      <br/>
      <input class="text" v-model="loginForm.password" type="password" value=""
             id="login_password" required minlength="8" />
      <span>password</span><span v-show="loginForm.passwordError"
                                 class="error-tip">{{ loginForm.passwordError }}</span>
      <br/>
      <input name="" type="checkbox" value="" id="checkbox-1-1" class="custom-checkbox" />
      <label for="checkbox-1-1">Keep me Signed in</label>
      <button class="sign_btn" type="submit">Submit</button>
    </form>
    <div class="select">
      <div>Register <i>----</i><router-link to="/register">click</router-link></div>
      <div>
        Forget Your Password?
        <router-link :to="{ name: 'ResetPasswordRequest' }">Click here to reset it.</router-link>
      </div>
    </div>
    <div class="tip">
      <p class="tip-text">
        *We promise not to disclose your personal information to other websites.
      </p>
    </div>
  </div>
</template>

<script>
import store from '../../store/store'

export default {
  name: 'Login',
  data () {
    return {
      sharedState: store.state,
      loginForm: {
        username: '',
        password: '',
        submitted: false,
        errors: 0,
        usernameError: null,
        passwordError: null
      }
    }
  },
  methods: {
    onSubmit (e) {
      this.loginForm.errors = 0 // 重置

      if (!this.loginForm.username) {
        this.loginForm.errors++
        this.loginForm.usernameError = 'Username required.'
      } else {
        this.loginForm.usernameError = null
      }

      if (!this.loginForm.password) {
        this.loginForm.errors++
        this.loginForm.passwordError = 'Password required.'
      } else {
        this.loginForm.passwordError = null
      }

      if (this.loginForm.errors > 0) {
        // 表单验证没通过时，不继续往下执行，即不会通过 axios 调用后端API
        return false
      }

      const path = '/tokens'
      // axios 实现Basic Auth需要在config中设置 auth 这个属性即可
      this.$axios.post(path, {}, {
        auth: {
          username: this.loginForm.username,
          password: this.loginForm.password
        }
      }).then((response) => {
        // handle success
        window.localStorage.setItem('madblog-token', response.data.token)
        store.loginAction()

        this.$toasted.success(`Welcome ${this.sharedState.user_name}!`, { icon: 'fingerprint' })
        if (typeof this.$route.query.redirect === 'undefined') {
          this.$router.push('/')
        } else {
          this.$router.push(this.$route.query.redirect)
        }
      })
        .catch((error) => {
          // handle error
          // console.log('failed', error.response);
          if (typeof error.response !== 'undefined') {
            // eslint-disable-next-line eqeqeq
            if (error.response.status === 401) {
              this.loginForm.usernameError = 'Invalid username or password.'
              this.loginForm.passwordError = 'Invalid username or password.'
            } else {
              console.log(error.response)
            }
          }
        })
    }
  }
}
</script>

<style scoped>
  .login {
    position: relative;
    height: 560px;
    width: 405px;
    padding: 40px 40px;
    margin: 20px auto;
  }
  form {
    padding-top: 30px;
  }

  .active {
    /*border-bottom: 2px solid #1161ed;*/
    border-bottom: 2px solid #928bad;
  }

  h2 {
    padding-left: 12px;
    font-size: 22px;
    text-transform: uppercase;
    padding-bottom: 5px;
    letter-spacing: 2px;
    display: inline-block;
    font-weight: 100;
  }

  h2:first-child {
    padding-left: 0px;
  }

  span {
    text-transform: uppercase;
    font-size: 12px;
    opacity: 0.8;
    display: inline-block;
    position: relative;
    top: -75px;
    transition: all 0.5s ease-in-out;
  }

  .text {
    width: 89%;
    padding: 10px 20px;
    display: block;
    height: 15px;
    border-radius: 20px;
    background: rgba(255, 255, 255, 0.1);
    border: 2px solid rgba(255, 255, 255, 0);
    overflow: hidden;
    margin-top: 15px;
    transition: all 0.5s ease-in-out;
  }

  .text:focus {
    outline: 0;
    border: 2px solid rgba(255, 255, 255, 0.5);
    border-radius: 20px;
    background: rgba(0, 0, 0, 0);
  }

  .text:focus + span {
    opacity: 0.6;
  }

  input[type="text"],
  input[type="password"] {
    font-family: 'Montserrat', sans-serif;
    color: #fff;
  }

  input {
    display: inline-block;
    padding-top: 20px;
    font-size: 14px;
  }

  h2,
  span,
  .custom-checkbox {
    margin-left: 20px;
  }

  .custom-checkbox {
    -webkit-appearance: none;
    background-color: rgba(255, 255, 255, 0.1);
    padding: 8px;
    border-radius: 2px;
    display: inline-block;
    position: relative;
    top: 6px;
  }

  .custom-checkbox:checked {
    background-color: #928bad;
  }

  .custom-checkbox:checked:after {
    content: '\2714';
    font-size: 10px;
    position: absolute;
    top: 1px;
    left: 4px;
    color: #fff;
  }

  .custom-checkbox:focus {
    outline: none;
  }

  label {
    display: inline-block;
    padding-top: 10px;
    padding-left: 5px;
  }

  .sign_btn {
    background-color: #928bad;
    font-family: 'Montserrat', sans-serif;
    color: #fff;
    width: 80%;
    padding: 10px 20px;
    display: block;
    height: 39px;
    border-radius: 20px;
    margin-top: 30px;
    transition: all 0.5s ease-in-out;
    border: none;
    text-transform: uppercase;
  }

  .sign_btn:hover {
    background: #948e99;
    box-shadow: 0px 4px 35px -5px #948e99;
    cursor: pointer;
  }

  .sign_btn:focus {
    outline: none;
  }

  hr {
    border: 1px solid rgba(255, 255, 255, 0.1);
    top: 85px;
    position: relative;
  }
  .select {
    font-size: 14px;
    margin-top: 50px;
    text-align: center;
    color: #eee;
    display: block;
  }
  .tip{
    position: fixed;
    right: 30px;
    bottom: 10px;
    width: 300px;
  }
  .tip-text {
    opacity: 0.8;
    color: #e8cbc0;
  }
</style>
