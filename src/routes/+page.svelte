<script lang="ts">
  let hue = 120;
  let saturation = 100;
  let lightness = 50;

  function handleClick(event: MouseEvent) {
    const rect = (event.target as HTMLDivElement).getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;

    saturation = Math.round((x / 446) * 100);

    if (y < 114) {
      lightness = Math.round(100 - (x / 446) * 50);
    } else {
      lightness = Math.round(50 - ((y - 114) / 114) * 50);
    }
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
    const v = (l + (s * Math.min(l, 1 - l)) / 100);
    const newS = v ? (2 - (2 * l) / v) * 100 : 0;
    return { h, s: newS, v: v * 100 };
  }

  function hslToCmyk(h: number, s: number, l: number) {
    const { r, g, b } = hslToRgb(h, s, l);

    const c = 1 - r;
    const m = 1 - g;
    const y = 1 - b;
    
    const k = Math.min(c, Math.min(m, y));
    
    return {
      c: (c - k) / (1 - k) * 100,
      m: (m - k) / (1 - k) * 100,
      y: (y - k) / (1 - k) * 100,
      k: k * 100
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
      alert(`Copiado: ${value}`);
    });
  }

  $: color = `hsl(${hue}, ${saturation}%, ${lightness}%)`;
  $: hexColor = hslToHex(hue, saturation, lightness);
  $: hsv = hslToHsv(hue, saturation, lightness);
  $: cmyk = hslToCmyk(hue, saturation, lightness);
  $: rgb = hslToRgb(hue, saturation, lightness);
</script>

<style>
  .main {
    height: 100vh;
    display: flex;
    justify-content: center;
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
    width: 40rem;
    height: 30rem;
  }

  .color-selector {
    width: 446px;
    height: 228px;
    aspect-ratio: 1/1;
    position: relative;
    cursor: crosshair;
  }

  .slider {
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

  input[type="range"]:focus::-moz-range-thumb{
    outline: 3px solid #3d3846;
    outline-offset: 0.125rem;
  }
</style>

<div class="main" style="background-color: {color};">
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="container">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="color-selector"
      on:click={handleClick}
      style="background-image:
        linear-gradient(to bottom, transparent 0%, black 100%),
        linear-gradient(90deg, rgba(79, 70, 229, 0%) 0%, hsl({hue}, 100%, 50%) 100%),
        linear-gradient(0deg, #000000 0%, rgba(196, 196, 196, 0%) 100%);">
    </div>

    <input type="range" min="0" max="360" bind:value={hue} class="slider" />

    <p>RGB: {rgb.r}, {rgb.g}, {rgb.b}</p>
    <p>HSL: {hue}, {saturation}%, {lightness}%</p>
    <p>HEX: <span>{hexColor}</span></p>
    <p>HSV: {hsv.h}, {hsv.s}%, {hsv.v}%</p>
    <p>CMYK: {cmyk.c.toFixed(2)}%, {cmyk.m.toFixed(2)}%, {cmyk.y.toFixed(2)}%, {cmyk.k.toFixed(2)}%</p>
    
  </div>
</div>
