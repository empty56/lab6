<template>
  <div class="ChatBox">
    <h1 class ="welcome">Welcome to ChatBox</h1>
    <div class="Chats">
      <Chats/>
    </div>
  </div>
</template>

<script>

import Chats from '@/components/Chats.vue'

export default {
  data()
  {
    return {
      hiddenReg: true,
    }
  },
  name: 'ChatBox',
  components: {
    Chats
  }
}
</script>

<style>
.welcome{
  text-align: center;
  margin-top: 5vh;
  font-size: 50px;
}
</style>

