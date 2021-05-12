<template>
  <div class="main-div">
    <div class="Home">
      <div class="Login" v-if="hiddenReg">
        <Login/>
        <div class="a-anim">
          <p>Don't have an account? <a v-on:click="hiddenReg = false;">&nbsp;Sign up</a></p>
        </div>
      </div>
      <div class="Register" v-if="!hiddenReg">
        <Register/>
        <div class="a-anim">
          <p>I already have account. <a v-on:click="hiddenReg = true;">&nbsp;Log in</a></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import Login from '@/components/Login.vue'
import Register from '@/components/Register.vue'

export default {
  data()
  {
    return {
      hiddenReg: true,
    }
  },
  name: 'Home',
  components: {
    Login,
    Register
  }
}
</script>

<style>

.a-anim {
  display: block;
  margin: auto;
  text-align: center;
}
.a-anim p{
  font-size: 17px;
}
.Home{
  width: 33vw;
  height: 40vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(38, 35, 35, 0.7);
  font-family: 'Handlee', cursive;
  border-radius: 12%;
  box-shadow: 20px 20px 50px rgba(0,0,0,.5);
  backdrop-filter: blur(10px);
}
.Home input{
  width: 400px;
  height: 50px;
  text-align: center;
  margin-bottom: 30px;
  border-radius: 20px;
  border: 2px solid #c4221f;
  box-shadow: 0 0 5px #c4221f,
  0 0 15px #c4221f,
  0 0 20px #c4221f;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
  background-color: #ebebeb;
}
.Home input::placeholder{
  display: block;
  font-family: 'Handlee', cursive;
}
.Home button{
  width: 120px;
  height: 38px;
  font-size: 22px;
  font-weight: 600;
  border-radius: 8px;
}
.main-div{
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
