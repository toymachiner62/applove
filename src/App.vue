<template>
  <div id="app">
    <div id="header">
      <div class="pull-left">
        <h1>AppLoVe</h1>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.pull-left {
  float: left;
}

.pull-right {
  float: right;
}
</style>
