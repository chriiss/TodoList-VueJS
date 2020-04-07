<template>
  <div id="app">
    <home></home>
  </div>
</template>

<script>
import Home from './components/Home.vue';

export default {
  name: 'App',
  components: {
    'home' : Home
  }
}
</script>

<style lang="scss">
@import './styles/Reset.scss';
</style>
