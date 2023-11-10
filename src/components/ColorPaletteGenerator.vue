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
                    <input type="radio" v-model="colorConstraint" value="warm" /> Warm
                </label>
                <label>
                    <input type="radio" v-model="colorConstraint" value="cool" /> Cool
                </label>
            </div>
        </div>

        <div class="mode-toggle">
            <label>Dark Mode</label>
            <input type="checkbox" v-model="darkMode" />
        </div>

        <div class="pattern-options">
            <label>Select Color Pattern:</label>
            <select v-model="selectedPattern">
                <option value="random">Random</option>
                <option value="complementary">Complementary</option>
                <option value="analogous">Analogous</option>
                <option value="triadic">Triadic</option>
            </select>
        </div>

        <div class="buttons">
            <button @click="generateColorPalette">Generate Color Palette</button>
            <button @click="randomizeColorPalette">Randomize Color Palette</button>
            <button @click="copyToClipboard">Copy to Clipboard</button>
            <button @click="resetSettings">Reset Settings</button>
        </div>

        <div class="color-palette">
            <div v-for="(color, index) in colorPalette" :key="index" :style="{ backgroundColor: color }" class="color-box">
            </div>
        </div>
    </div>
</template>
  
<script>
import chroma from 'chroma-js';

export default {
    data() {
        return {
            colorPalette: [],
            numberOfColors: 5,
            colorConstraint: null,
            saturation: 50,
            colorRange: '0-360',
            darkMode: false,
            selectedPattern: 'random',
        };
    },
    computed: {
        containerClass() {
            return {
                'color-palette-generator': true,
                'dark-mode': this.darkMode,
            };
        },
    },
    methods: {
        generateRandomColor() {
            return '#' + Math.floor(Math.random() * 16777215).toString(16);
        },
        generateColorPalette() {
            switch (this.selectedPattern) {
                case 'random':
                    this.colorPalette = Array.from({ length: this.numberOfColors }, this.generateRandomColor);
                    break;
                case 'complementary':
                    this.colorPalette = this.generateComplementaryPalette();
                    break;
                case 'analogous':
                    this.colorPalette = this.generateAnalogousPalette();
                    break;
                case 'triadic':
                    this.colorPalette = this.generateTriadicPalette();
                    break;
                default:
                    this.colorPalette = Array.from({ length: this.numberOfColors }, this.generateRandomColor);
            }
        },
        generateComplementaryPalette() {
            const baseColor = chroma(this.generateRandomColor());
            const complementaryColor = baseColor.saturate(2);
            return chroma.scale([baseColor, complementaryColor]).colors(this.numberOfColors);
        },
        generateAnalogousPalette() {
            const baseColor = chroma(this.generateRandomColor());
            const analogousColor1 = baseColor.saturate(2);
            const analogousColor2 = baseColor.saturate(-2);
            return chroma.scale([baseColor, analogousColor1, analogousColor2]).colors(this.numberOfColors);
        },
        generateTriadicPalette() {
            const baseColor = chroma(this.generateRandomColor());
            const triadicColor1 = baseColor.saturate(2).rotate(120);
            const triadicColor2 = baseColor.saturate(-2).rotate(-120);
            return chroma.scale([baseColor, triadicColor1, triadicColor2]).colors(this.numberOfColors);
        },
        copyToClipboard() {
            const colorsText = this.colorPalette.join(', ');
            const textarea = document.createElement('textarea');
            textarea.value = colorsText;
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand('copy');
            document.body.removeChild(textarea);
        },
        randomizeColorPalette() {
            this.colorPalette = Array.from({ length: this.numberOfColors }, this.generateRandomColor);
        },
        resetSettings() {
            this.numberOfColors = 5;
            this.colorConstraint = null;
            this.saturation = 50;
            this.colorRange = '0-360';
            this.darkMode = false;
            this.selectedPattern = 'random';
            this.generateColorPalette();
        },
    },
};
</script>
  
<style scoped>
.color-palette-generator {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 5px;
    background-color: #f9f9f9;
    transition: background-color 0.3s ease;
}

.dark-mode {
    background-color: #333;
    color: #fff;
}

.options {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    margin-bottom: 20px;
}

.option {
    flex: 1 1 48%;
    margin-bottom: 10px;
}

.mode-toggle {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
}

.mode-toggle label {
    margin-right: 10px;
}

.pattern-options {
    margin-bottom: 20px;
}

.buttons {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
}

.color-palette {
    display: flex;
    flex-wrap: wrap;
}

.color-box {
    flex: 0 0 calc(20% - 10px);
    height: 50px;
    margin: 0 5px 10px 0;
}
</style>
  