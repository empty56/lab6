<template>
  <div class="login-container">
    <input type="text" v-model="username" placeholder="Username" required/>
    <br/>
    <input type="password" v-model="password" placeholder="Password" required/>
    <br/>
    <button v-on:click="loginData">Log in</button>
    <br/>
  </div>
</template>

<script>
import gql from "graphql-tag"
import {onLogin} from "../vue-apollo";
const userData = gql`query login($username:String!,$password:String!) {
  login(username: $username, password: $password)
}`

export default {
  data()
  {
    return {
      username:"",
      password:"",
    };
  },
  name: "Login",
  methods: {
    async loginData() {
      const token = await this.getToken()
      onLogin(this.$apollo.provider.defaultClient, token);
      await this.$router.push('/chats')
    },
    async getToken() {
      const token = await this.$apollo.query({
        query: userData,
        variables: {
          username: this.username,
          password: this.password
        },
      })
      return token.data.login
    },
  }
}
</script>

<style scoped>
  .login-container input{
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
  .login-container input::placeholder{
    display: block;
    font-family: 'Handlee', cursive;
  }
  .login-container button{
    width: 120px;
    height: 38px;
    font-size: 22px;
    font-weight: 600;
    border-radius: 8px;
  }

</style>
