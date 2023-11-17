<template>
  <div class="options">
    <div class="option">
      <label for="numberOfColors">Number of Colors:</label>
      <input v-model="numberOfColors" type="number" min="1" id="numberOfColors" />
    </div>

    <div class="option">
      <label for="saturation">Saturation:</label>
      <input v-model="saturation" type="range" min="0" max="100" id="saturation" />
    </div>

    <div class="option">
      <label for="colorRange">Color Range:</label>
      <input v-model="colorRange" type="text" id="colorRange" />
    </div>

    <div class="option">
      <label>Color Constraint:</label>
      <label>
        <input type="radio" v-model="colorConstraint" value="warm" /> Warm
      </label>
      <label>
        <input type="radio" v-model="colorConstraint" value="cool" /> Cool
      </label>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
export default {
  setup() {
    const numberOfColors = ref(5);
    const saturation = ref(50);
    const colorRange = ref('0-360');
    const colorConstraint = ref(null);
    
    return {
      numberOfColors,
      saturation,
      colorRange,
      colorConstraint,
    };
  }
};
</script>

<style scoped>
/* Add styles if necessary */
</style>
