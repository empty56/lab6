<template>
  <div>
    <div>
      <particles-bg type="tadpole" color="#c4221f"  :bg="true" />
    </div>
    <router-view/>
  </div>
</template>


<style>
@import url('https://fonts.googleapis.com/css2?family=Handlee&display=swap');
body{
  background-image: url('~@/images/bg.jpeg');
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-position: center;
  background-size: cover;
  font-family: 'Handlee', cursive;
  color: #d9cbcb;
}
*{
  padding: 0;
  margin: 0;
}
input, button{
  outline: none;
}
button{
  display: block;
  margin: auto;
  border: solid #d9cbcb;
  color: #d9cbcb;
  text-align: center;
  background: #202020;
  font-family: 'Handlee', cursive;
}
button:hover {
  background: #3c3c3c;
}
a {
  background-image: linear-gradient(currentColor, currentColor);
  background-position: 0 100%;
  background-repeat: no-repeat;
  background-size: 0 2px;
  transition: background-size .3s;
  text-decoration: none;
  color: #d9cbcb;
}
a:hover{
  cursor: pointer;
  color: #ce2b29;
  background-size: 100% 2px;
}
</style>
