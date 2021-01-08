<template>
  <div id="app">
    <router-view class="container" />
  </div>
</template>

<script>
export default {
  name: 'App',
};

</script>

<style lang="scss">
  @import "./assets/scss/reset.css";
  @import "./assets/scss/color.scss";

  html,
  body {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
  }

  #app {
    box-sizing: border-box;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    width: 100%;
    height: 100%;
    padding: 10px;
    background-color: #fff;
    overflow-x: hidden;
  }

  .container {
    width: 100%;
  }

</style>
