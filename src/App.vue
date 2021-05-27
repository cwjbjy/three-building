<template>
  <div id="app">
    <nav>
      <router-link to="/" style="margin-right:20px">东方明珠页面</router-link>
      <router-link to="/about">工厂页面</router-link>
    </nav>
    <router-view />
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

body {
  margin: 0;
  padding: 0;
}

nav{
  position: absolute;
  left: 50%;
  color: red;
  background: oldlace;
}
</style>
