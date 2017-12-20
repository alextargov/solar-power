<template>
  <div id="app">
    <Scene @generatedDivider="getDivider"></Scene>
    <appliances :divider="divider"></appliances>
  </div>
</template>

<script>
import Scene from './components/Scene.vue'
import Appliances from './components/Appliances.vue'
export default {
  name: 'app',
  data () {
    return {
      divider: ""
    }
  },
  methods: {
    getDivider(value) {
      this.divider = value;
    }

  },
  components: {
    Scene, Appliances
  }
}
</script>

<style>
#app {
  width: 960px;
  margin: 10px auto;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

}
</style>
