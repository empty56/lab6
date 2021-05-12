<template>
  <div class="login">
    <input type="text" v-model="username" placeholder="Username" required/>
    <br/>
    <input type="password" v-model="password" placeholder="Password" required/>
    <br/>
    <input type="password" v-model="confirmPassword" placeholder="Confirm password" required/>
    <br/>
    <button v-on:click="register">Register</button>
    <br/>
  </div>
</template>

<script>

import gql from "graphql-tag"
import {onLogin} from "../vue-apollo";
const newUser = gql`query registration($username:String!,$password:String!) {
  register(username: $username, password: $password) {
    id
    username
    timestamp
  }
}`
const checkUser = gql`query checkUsername($username:String!) {
  user: usernameExists(username: $username)
}`
const userData = gql`query login($username:String!,$password:String!) {
  login(username: $username, password: $password)
}`
export default {
  data()
  {
    return {
      username:"",
      password:"",
      confirmPassword:"",
    };
  },
  name: "Register",
  methods: {
    async checkUser() {
      const user = await this.$apollo.query({
        query: checkUser,
        variables: {
          username: this.username
        },
      })
      return user.data.user
    },
    async register() {
      if (this.confirmPassword.length === 0 || this.password.length === 0 || this.password !== this.confirmPassword) {
        this.errorMes = "Incorrect password input"
        console.log("Incorrect password input.")
        return
      }
      if (await this.checkUser()) {
        console.log("User with such username already exists.")
        return
      }
      await this.$apollo.query({
        query: newUser,
        variables: {
          username: this.username,
          password: this.password,
        },
      })
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

</style>