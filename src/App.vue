<template>
  <div id="app">
    <Body />
  </div>
</template>

<script>
import Body from './components/Body.vue';

export default {
  name: 'App',
  components: {
    Body
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
