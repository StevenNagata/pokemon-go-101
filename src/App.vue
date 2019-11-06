<template>
  <div id="app" class="container-fluid p-2">
    <router-view ></router-view>
  </div>

</template>

<script>
import Home from './components/Home.vue'

export default {
  name: 'app',
  components: {
    Home
  }
}
</script>

<style>
#app {
background-color: rgb(139, 124, 146); 
    margin: 0;
    min-height: 100vh;
    padding: 0;
} 
h1 {
  text-align: center;
}
</style>
