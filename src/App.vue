<template>
  <div id="app">
    <ColorPaletteGenerator />
  </div>
</template>

<script>
import ColorPaletteGenerator from './components/ColorPaletteGenerator.vue';

export default {
  components: {
    ColorPaletteGenerator,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1 {
  font-size: 2em;
}
</style>
