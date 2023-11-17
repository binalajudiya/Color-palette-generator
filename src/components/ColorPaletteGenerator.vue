<template>
  <div :class="containerClass">
    <h1>Color Palette Generator</h1>

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
          <input type="radio" v-model="state.colorConstraint" value="warm" /> Warm
        </label>
        <label>
          <input type="radio" v-model="state.colorConstraint" value="cool" /> Cool
        </label>
      </div>
    </div>

    <div class="mode-toggle">
      <label>Dark Mode</label>
      <input v-model="state.darkMode" type="checkbox" />
    </div>

    <div class="pattern-options">
      <label>Select Color Pattern:</label>
      <select v-model="state.selectedPattern" @change="updateHarmonyInformation">
        <option value="random">Random</option>
        <option value="complementary">Complementary</option>
        <option value="analogous">Analogous</option>
        <option value="triadic">Triadic</option>
      </select>
    </div>

    <div class="harmony-info" v-if="state.harmonyInformation">
      <p><strong>Color Harmony:</strong> {{ state.harmonyInformation }}</p>
    </div>

    <div class="palette-size">
      <label for="paletteSize">Palette Size:</label>
      <input v-model="state.paletteSize" type="number" min="1" id="paletteSize" @change="generateColorPalette" />
    </div>

    <div class="buttons">
      <button @click="generateColorPalette">Generate Color Palette</button>
      <button @click="randomizeColorPalette">Randomize Color Palette</button>
      <button @click="copyToClipboard">Copy to Clipboard</button>
      <button @click="resetSettings">Reset Settings</button>
    </div>

    <div class="color-palette" ref="colorPaletteElement">
      <div v-for="(color, index) in state.colorPalette" :key="index" :style="{ backgroundColor: color }" class="color-box">
      </div>
    </div>

    <div class="export-options">
      <button @click="exportToImage">Export to Image</button>
    </div>
  </div>
</template>

<script>
import { ref, reactive, watchEffect } from 'vue';
import chroma from 'chroma-js';
import domtoimage from 'dom-to-image';

export default {
  setup() {
    const state = reactive({
      colorPalette: [],
      colorConstraint: null,
      darkMode: false,
      selectedPattern: 'random',
      harmonyInformation: null,
      paletteSize: 5,
    });

    const colorPaletteElement = ref(null);
    const saturation = ref(50);
    const numberOfColors = ref(5);
    const colorRange = ref('0-360');

    const containerClass = ref({
      'color-palette-generator': true,
      'dark-mode': state.darkMode,
    });

    const updateHarmonyInformation = () => {
      switch (state.selectedPattern) {
        case 'complementary':
          state.harmonyInformation = 'Complementary';
          break;
        case 'analogous':
          state.harmonyInformation = 'Analogous';
          break;
        case 'triadic':
          state.harmonyInformation = 'Triadic';
          break;
        default:
          state.harmonyInformation = null;
      }
    };

    const generateRandomColor = () => {
      return '#' + Math.floor(Math.random() * 16777215).toString(16);
    };

    const generateColorPalette = () => {
      switch (state.selectedPattern) {
        case 'random':
          state.colorPalette = Array.from({ length: numberOfColors.value }, generateRandomColor);
          break;
        case 'complementary':
          state.colorPalette = generateComplementaryPalette();
          break;
        case 'analogous':
          state.colorPalette = generateAnalogousPalette();
          break;
        case 'triadic':
          state.colorPalette = generateTriadicPalette();
          break;
        default:
          // Replace 'baseColor' with a valid color variable or value
          state.colorPalette = chroma.scale([baseColor, colorRange.value]).mode('lab').colors(state.paletteSize);

      }
    };

    const generateComplementaryPalette = () => {
      const baseColor = chroma(generateRandomColor());
      const complementaryColor = baseColor.saturate(2);
      return chroma.scale([baseColor, complementaryColor]).colors(numberOfColors.value);
    };

    const generateAnalogousPalette = () => {
      const baseColor = chroma(generateRandomColor());
      const analogousColor1 = baseColor.saturate(2);
      const analogousColor2 = baseColor.saturate(-2);
      return chroma.scale([baseColor, analogousColor1, analogousColor2]).colors(numberOfColors.value);
    };

    const generateTriadicPalette = () => {
      const baseColor = chroma(generateRandomColor());
      const triadicColor1 = baseColor.saturate(2).rotate(120);
      const triadicColor2 = baseColor.saturate(-2).rotate(-120);
      return chroma.scale([baseColor, triadicColor1, triadicColor2]).colors(numberOfColors.value);
    };

    const copyToClipboard = () => {
      const colorsText = state.colorPalette.join(', ');
      const textarea = document.createElement('textarea');
      textarea.value = colorsText;
      document.body.appendChild(textarea);
      textarea.select();
      document.execCommand('copy');
      document.body.removeChild(textarea);
    };

    const randomizeColorPalette = () => {
      state.colorPalette = Array.from({ length: numberOfColors.value }, generateRandomColor);
    };

    const resetSettings = () => {
      numberOfColors.value = 5;
      state.colorConstraint = null;
      saturation.value = null;
      colorRange.value = '0-360';
      state.darkMode = false;
      state.selectedPattern = 'random';
      generateColorPalette();
    };

    const exportToImage = () => {
      // Access the DOM element using the ref
      const paletteElement = colorPaletteElement.value;

      if (!paletteElement) {
        console.error('Color palette element not found.');
        return;
      }

      domtoimage.toBlob(paletteElement)
        .then((blob) => {
          // Use createObjectURL to generate a URL for the blob
          const url = URL.createObjectURL(blob);

          // Create a link element and trigger a click event to download the image
          const link = document.createElement('a');
          link.href = url;
          link.download = 'color_palette.png';
          link.click();

          // Revoke the URL to free up resources
          URL.revokeObjectURL(url);
        })
        .catch((error) => {
          console.error('Error exporting to image:', error);
        });
    };

    watchEffect(() => {
      containerClass.value = {
        'color-palette-generator': true,
        'dark-mode': state.darkMode,
      };
    });

    return {
      state,
      containerClass,
      updateHarmonyInformation,
      generateColorPalette,
      colorPaletteElement,
      generateRandomColor,
      generateComplementaryPalette,
      generateAnalogousPalette,
      generateTriadicPalette,
      copyToClipboard,
      randomizeColorPalette,
      resetSettings,
      exportToImage,
      saturation,
      numberOfColors,
      colorRange
    };
  },
};
</script>
  
<style scoped>
/* Reset some default styles */
body, h1, h2, p, ul, li {
  margin: 0;
  padding: 0;
}

/* Apply a modern font */
body {
  font-family: 'Arial', sans-serif;
}

.containerClass {
  width: 80%;
  margin: 0 auto;
  font-family: 'Roboto', sans-serif;
  color: #333;
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

/* Main container styles */
.color-palette-generator {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 10px;
  background-color: #f5f5f5;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.dark-mode {
  background-color: #333;
  color:#fff;
}

/* Header styles */
.color-palette-generator h1 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

.dark-mode, .dark-mode h1 {
  background-color: #333;
  color: #fff;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

.options, .mode-toggle, .pattern-options, .palette-size, .buttons, .export-options {
  margin-bottom: 20px;
}

/* Options container styles */
.options {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-bottom: 20px;
}

.option {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.option label {
  margin-right: 10px;
}


.buttons {
  display: flex;
  justify-content: space-around;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #007BFF;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #0056b3;
}


/* Dark mode toggle styles */
.mode-toggle, .pattern-options, .palette-size {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.mode-toggle label {
  margin-right: 10px;
}

/* Pattern options, palette size, harmony info styles */
.pattern-options,
.palette-size,
.harmony-info {
  margin-bottom: 20px;
}

.dark-mode .harmony-info {
  color: #000;
}

/* Buttons container styles */
.buttons {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

/* Color palette container styles */
.color-palette {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 20px;
}

.color-box {
  width: 50px;
  height: 50px;
  margin: 5px;
  border-radius: 5px;
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.1);
}

.export-options {
  text-align: center;
  margin-top: 20px;
}

.harmony-info {
  background-color: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}
.color-box:hover {
  transform: scale(1.1);
}

/* Export options button styles */
.export-options button {
  padding: 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.export-options button:hover {
  background-color: #45a049;
}

</style>
  