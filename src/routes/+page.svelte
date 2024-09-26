<script lang="ts">
  let hue = 120;
  let saturation = 100;
  let lightness = 50;
  let clickX = 0;
  let clickY = 0;
  let lastClick = false;

  function handleClick(event: MouseEvent) {
    const rect = (event.target as HTMLDivElement).getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;

    clickX = x;
    clickY = y;
    lastClick = true;

    saturation = Math.round(Math.min(100, Math.max(0, (x / 446) * 100)));

    if (y < 114) {
      lightness = Math.round(100 - (x / 446) * 50);
    } else {
      lightness = Math.round(50 - ((y - 114) / 114) * 50);
    }

    lightness = Math.min(100, Math.max(0, lightness)); // Limitar lightness para 0-100
  }

  function hslToHex(h: number, s: number, l: number) {
    l /= 100;
    const a = s * Math.min(l, 1 - l) / 100;
    const f = (n: number) => {
      const k = (n + h / 30) % 12;
      const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
      return Math.round(255 * color).toString(16).padStart(2, '0');
    };
    return `#${f(0)}${f(8)}${f(4)}`;
  }

  function hslToHsv(h: number, s: number, l: number) {
    const lDecimal = l / 100;
    const c = s / 100 * (1 - Math.abs(2 * lDecimal - 1)); // Chroma
    const v = lDecimal + c; // Value
    const newS = v ? Math.round((c / v) * 100) : 0; // Saturation

    return {
      h: Math.round(h) % 360, // Ajustar o hue para ficar entre 0 e 360
      s: Math.min(100, Math.max(0, newS)), // Limitar saturation para 0-100
      v: Math.min(100, Math.max(0, Math.round(v * 100))) // Limitar value para 0-100
    };
  }

  function hslToCmyk(h: number, s: number, l: number) {
    const { r, g, b } = hslToRgb(h, s, l);

    const c = 1 - r / 255;
    const m = 1 - g / 255;
    const y = 1 - b / 255;

    const k = Math.min(c, Math.min(m, y));
    
    return {
      c: Math.round(((c - k) / (1 - k)) * 100),
      m: Math.round(((m - k) / (1 - k)) * 100),
      y: Math.round(((y - k) / (1 - k)) * 100),
      k: Math.round(k * 100)
    };
  }

  function hslToRgb(h: number, s: number, l: number) {
    s /= 100;
    l /= 100;
    const a = s * Math.min(l, 1 - l);
    const f = (n: number) => {
      const k = (n + h / 30) % 12;
      const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
      return Math.round(255 * color);
    };
    return { r: f(0), g: f(8), b: f(4) };
  }

  function copyToClipboard(value: string) {
    navigator.clipboard.writeText(value).then(() => {
      console.log(`Copiado: ${value}`);
    });
  }

  $: color = `hsl(${hue}, ${saturation}%, ${lightness}%)`;
  $: hexColor = hslToHex(hue, saturation, lightness);
  $: hsv = hslToHsv(hue, saturation, lightness);
  $: cmyk = hslToCmyk(hue, saturation, lightness);
  $: rgb = hslToRgb(hue, saturation, lightness);
</script>

<div class="main" style="background-color: {color};">
  <h1 class="title">Color Picker</h1>

  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="container">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="color-selector"
      on:click={handleClick}
      style="background-image:
        linear-gradient(to bottom, transparent 0%, black 100%),
        linear-gradient(90deg, rgba(79, 70, 229, 0%) 0%, hsl({hue}, 100%, 50%) 100%),
        linear-gradient(0deg, #000000 0%, rgba(196, 196, 196, 0%) 100%);"
    >
      {#if lastClick}
        <div class="click-indicator" style="top: {clickY}px; left: {clickX}px;"></div>
      {/if}
    </div>

    <input type="range" min="0" max="360" bind:value={hue} class="slider" />

    <button class="btn" on:click={() => copyToClipboard(hexColor)}>
      <p>HEX: <span>{hexColor}</span></p>
      <svg height="16" viewBox="0 0 48 48" width="16" xmlns="http://www.w3.org/2000/svg"><path d="M0 0h48v48h-48z" fill="none"/><path d="M32 2h-24c-2.21 0-4 1.79-4 4v28h4v-28h24v-4zm6 8h-22c-2.21 0-4 1.79-4 4v28c0 2.21 1.79 4 4 4h22c2.21 0 4-1.79 4-4v-28c0-2.21-1.79-4-4-4zm0 32h-22v-28h22v28z"/></svg>
    </button>
    <div class="colors">
      <button class="btn" on:click={() => copyToClipboard(`rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`)}>
        <p>RGB: {rgb.r}, {rgb.g}, {rgb.b}</p>
        <svg height="16" viewBox="0 0 48 48" width="16" xmlns="http://www.w3.org/2000/svg"><path d="M0 0h48v48h-48z" fill="none"/><path d="M32 2h-24c-2.21 0-4 1.79-4 4v28h4v-28h24v-4zm6 8h-22c-2.21 0-4 1.79-4 4v28c0 2.21 1.79 4 4 4h22c2.21 0 4-1.79 4-4v-28c0-2.21-1.79-4-4-4zm0 32h-22v-28h22v28z"/></svg>
      </button>
      <button class="btn" on:click={() => copyToClipboard(`hsl(${hue}, ${saturation}%, ${lightness}%)`)}>
        <p>HSL: {hue}, {saturation}%, {lightness}%</p>
        <svg height="16" viewBox="0 0 48 48" width="16" xmlns="http://www.w3.org/2000/svg"><path d="M0 0h48v48h-48z" fill="none"/><path d="M32 2h-24c-2.21 0-4 1.79-4 4v28h4v-28h24v-4zm6 8h-22c-2.21 0-4 1.79-4 4v28c0 2.21 1.79 4 4 4h22c2.21 0 4-1.79 4-4v-28c0-2.21-1.79-4-4-4zm0 32h-22v-28h22v28z"/></svg>
      </button>
      <button class="btn" on:click={() => copyToClipboard(`hsv(${hsv.h}, ${hsv.s}%, ${hsv.v}%)`)}>
        <p>HSV: {hsv.h}, {hsv.s}%, {hsv.v}%</p>
        <svg height="16" viewBox="0 0 48 48" width="16" xmlns="http://www.w3.org/2000/svg"><path d="M0 0h48v48h-48z" fill="none"/><path d="M32 2h-24c-2.21 0-4 1.79-4 4v28h4v-28h24v-4zm6 8h-22c-2.21 0-4 1.79-4 4v28c0 2.21 1.79 4 4 4h22c2.21 0 4-1.79 4-4v-28c0-2.21-1.79-4-4-4zm0 32h-22v-28h22v28z"/></svg>
      </button>
      <button class="btn" on:click={() => copyToClipboard(`cmyk(${cmyk.c}%, ${cmyk.m}%, ${cmyk.y}%, ${cmyk.k}%)`)}>
        <p>CMYK: {cmyk.c}%, {cmyk.m}%, {cmyk.y}%, {cmyk.k}%</p>
        <svg height="16" viewBox="0 0 48 48" width="16" xmlns="http://www.w3.org/2000/svg"><path d="M0 0h48v48h-48z" fill="none"/><path d="M32 2h-24c-2.21 0-4 1.79-4 4v28h4v-28h24v-4zm6 8h-22c-2.21 0-4 1.79-4 4v28c0 2.21 1.79 4 4 4h22c2.21 0 4-1.79 4-4v-28c0-2.21-1.79-4-4-4zm0 32h-22v-28h22v28z"/></svg>
      </button>
    </div>
  </div>
</div>

<style>
  .main {
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .title {
    font-size: 5rem;
    margin-bottom: 1rem;
    font-family: 'Courier New', Courier, monospace;
  }

  .btn {
    background-color: #FFF;
    border: 1px solid #000;
    border-radius: 0.5rem;
    padding: 0.25rem 0.5rem;
    font-size: 0.75rem;
    margin: 0.5rem;
    cursor: pointer;
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: row;
  }

  .btn p {
    margin-right: 0.5rem;
  }

  .colors {
    display: flex;
    align-items: center;
  }

  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: #FFF;
    border-radius: 0.5rem;
    padding: 1rem;
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.1);
    width: 50rem;
    height: 30rem;
  }

  .color-selector {
    width: 446px;
    height: 228px;
    aspect-ratio: 1/1;
    position: relative;
    cursor: crosshair;
  }

  .click-indicator {
    position: absolute;
    width: 10px;
    height: 10px;
    background-color: white;
    border: 2px solid black;
    border-radius: 50%;
    transform: translate(-50%, -50%);
  }

  .slider {
    margin-top: 0.5rem;
    margin-bottom: 1rem;
    width: 100%;
  }

  input[type="range"] {
    -webkit-appearance: none;
    appearance: none;
    background: transparent;
    cursor: pointer;
    width: 25rem;
  }

  input[type="range"]:focus {
    outline: none;
  }

  input[type="range"]::-webkit-slider-runnable-track {
    background: linear-gradient(to right, red, yellow, lime, cyan, blue, magenta, red);
    border-radius: 0.5rem;
    height: 0.5rem;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    margin-top: -4px;
    background-color: #3d3846;
    border-radius: 0.5rem;
    height: 1rem;
    width: 1rem;
  }

  input[type="range"]:focus::-webkit-slider-thumb {
    outline: 3px solid #3d3846;
    outline-offset: 0.125rem;
  }

  input[type="range"]::-moz-range-track {
    background: linear-gradient(to right, red, yellow, lime, cyan, blue, magenta, red);
    border-radius: 0.5rem;
    height: 0.5rem;
  }

  input[type="range"]::-moz-range-thumb {
    background-color: #3d3846;
    border: none;
    border-radius: 0.5rem;
    height: 1rem;
    width: 1rem;
  }

  input[type="range"]:focus::-moz-range-thumb {
    outline: 3px solid #3d3846;
    outline-offset: 0.125rem;
  }
</style>