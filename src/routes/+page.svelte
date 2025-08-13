<script>
  import { onMount } from 'svelte';
  
  let currentPalette = $state(['#6b7280', '#9ca3af', '#4b5563', '#d1d5db', '#374151']);
  let showToast = $state(false);
  let selectedColorTab = 'Generated';
  let showModal = $state(false);
  let selectedComponent = $state('');
  let componentCode = $state('');
  let toastMessage = $state('');
  
  // Generate random color
  function generateRandomColor() {
    const letters = '0123456789ABCDEF';
    let color = '#';
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }
  
  // Generate analogous color
  /**
	 * @param {string} hex
	 * @param {number} angle
	 */
  function generateAnalogousColor(hex, angle) {
    let [r, g, b] = hexToRgb(hex);
    let hsl = rgbToHsl(r, g, b);
    // @ts-ignore
    hsl[0] = (hsl[0] + angle/360) % 1;
    if (hsl[0] < 0) hsl[0] += 1;
    return hslToHex(...hsl);
  }
  
  // Generate tonal color (lighter/darker)
  // @ts-ignore
  function generateTonalColor(hex, percent) {
    let [r, g, b] = hexToRgb(hex);
    let hsl = rgbToHsl(r, g, b);
    // @ts-ignore
    hsl[2] = Math.min(1, Math.max(0, hsl[2] + percent/100));
    return hslToHex(...hsl);
  }
  
  // Color conversion utilities
  // @ts-ignore
  function hexToRgb(hex) {
    const r = parseInt(hex.slice(1, 3), 16);
    const g = parseInt(hex.slice(3, 5), 16);
    const b = parseInt(hex.slice(5, 7), 16);
    return [r, g, b];
  }
  
  // @ts-ignore
  function rgbToHsl(r, g, b) {
    r /= 255, g /= 255, b /= 255;
    const max = Math.max(r, g, b), min = Math.min(r, g, b);
    let h, s, l = (max + min) / 2;
  
    if (max === min) {
      h = s = 0; // achromatic
    } else {
      const d = max - min;
      s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
      switch (max) {
        case r: h = (g - b) / d + (g < b ? 6 : 0); break;
        case g: h = (b - r) / d + 2; break;
        case b: h = (r - g) / d + 4; break;
      }
      // @ts-ignore
      h /= 6;
    }
  
    return [h, s, l];
  }
  
  // @ts-ignore
  function hslToHex(h, s, l) {
    let r, g, b;
  
    if (s === 0) {
      r = g = b = l; // achromatic
    } else {
      // @ts-ignore
      const hue2rgb = (p, q, t) => {
        if (t < 0) t += 1;
        if (t > 1) t -= 1;
        if (t < 1/6) return p + (q - p) * 6 * t;
        if (t < 1/2) return q;
        if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
        return p;
      };
  
      const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
      const p = 2 * l - q;
  
      r = hue2rgb(p, q, h + 1/3);
      g = hue2rgb(p, q, h);
      b = hue2rgb(p, q, h - 1/3);
    }
  
    // @ts-ignore
    const toHex = x => {
      const hex = Math.round(x * 255).toString(16);
      return hex.length === 1 ? '0' + hex : hex;
    };
  
    return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
  }
  
  function generatePalette() {
    const baseColor = generateRandomColor();
    return [
      baseColor,
      generateAnalogousColor(baseColor, 25),
      generateAnalogousColor(baseColor, -25),
      generateTonalColor(baseColor, 25),
      generateTonalColor(baseColor, -25)
    ];
  }
  
  // Generate Tailwind config
  let tailwindConfig = $state('');
  
  function updateTailwindConfig() {
    if (currentPalette.length === 0) {
      tailwindConfig = '';
      return;
    }
    
    tailwindConfig = `// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: '${currentPalette[0]}',
          light: '${currentPalette[3]}',
          dark: '${currentPalette[4]}'
        },
        secondary: '${currentPalette[1]}',
        accent: '${currentPalette[2]}',
        success: '${currentPalette[1]}',
        warning: '${currentPalette[3]}',
        custom: {
          '50': '${currentPalette[3]}',
          '100': '${currentPalette[0]}',
          '200': '${currentPalette[1]}',
          '300': '${currentPalette[2]}',
          '400': '${currentPalette[3]}',
          '500': '${currentPalette[4]}'
        }
      }
    }
  }
}`;
  }
  
  // Copy to clipboard
  async function copyToClipboard() {
    try {
      await navigator.clipboard.writeText(tailwindConfig);
      showToast = true;
      setTimeout(() => {
        showToast = false;
      }, 3000);
    } catch (err) {
      console.error('Failed to copy: ', err);
    }
  }
  
  // Handle keyboard events
  // @ts-ignore
  function handleTouch(event) {
    // Prevent default touch behavior and generate new palette
    event.preventDefault();
    currentPalette = generatePalette();
    updateTailwindConfig();
  }

  // @ts-ignore
  function handleKeydown(event) {
    if (event.code === 'Space') {
      event.preventDefault();
      currentPalette = generatePalette();
      updateTailwindConfig();
    }
  }
  
  // Initialize on mount
  onMount(() => {
    updateTailwindConfig();
  });

// @ts-ignore
function generateComponentCode(componentType) {
  const componentCodes = {
    'stats-card': `<!-- Active Users Stats Card -->
<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-gray-600 transition-all duration-300 hover:shadow-xl cursor-pointer">
  <div class="flex items-center justify-between mb-4">
    <div class="p-3 rounded-lg" style="background-color: ${currentPalette[1]}20; color: ${currentPalette[1]}">
      <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z"></path>
      </svg>
    </div>
    <div class="text-right">
      <div class="text-sm text-gray-400 font-medium">Active Users</div>
    </div>
  </div>
  <div class="mb-4">
    <div class="text-3xl font-bold text-white mb-1">+2,350</div>
    <div class="flex items-center gap-2">
      <span class="text-sm font-medium px-2 py-1 rounded-full" style="background-color: ${currentPalette[2]}20; color: ${currentPalette[2]}">
        +180.1%
      </span>
      <span class="text-xs text-gray-400">from last month</span>
    </div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.border-gray-700 { border-color: #374151; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.rounded-xl { border-radius: 0.75rem; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.duration-300 { transition-duration: 300ms; }
.hover\\:shadow-xl:hover { box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1); }
</style>`,

    'sales-card': `<!-- Sales Stats Card -->
<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-gray-600 transition-all duration-300 hover:shadow-xl cursor-pointer">
  <div class="flex items-center justify-between mb-4">
    <div class="p-3 rounded-lg" style="background-color: ${currentPalette[2]}20; color: ${currentPalette[2]}">
      <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1"></path>
      </svg>
    </div>
    <div class="text-right">
      <div class="text-sm text-gray-400 font-medium">Sales</div>
    </div>
  </div>
  <div class="mb-4">
    <div class="text-3xl font-bold text-white mb-1">$12,234</div>
    <div class="flex items-center gap-2">
      <span class="text-sm font-medium px-2 py-1 rounded-full" style="background-color: ${currentPalette[0]}20; color: ${currentPalette[0]}">
        +19%
      </span>
      <span class="text-xs text-gray-400">from last month</span>
    </div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.border-gray-700 { border-color: #374151; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.rounded-xl { border-radius: 0.75rem; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.duration-300 { transition-duration: 300ms; }
.hover\\:shadow-xl:hover { box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1); }
</style>`,

    'conversion-card': `<!-- Conversion Rate Stats Card -->
<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-gray-600 transition-all duration-300 hover:shadow-xl cursor-pointer">
  <div class="flex items-center justify-between mb-4">
    <div class="p-3 rounded-lg" style="background-color: ${currentPalette[3]}20; color: ${currentPalette[3]}">
      <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path>
      </svg>
    </div>
    <div class="text-right">
      <div class="text-sm text-gray-400 font-medium">Conversion Rate</div>
    </div>
  </div>
  <div class="mb-4">
    <div class="text-3xl font-bold text-white mb-1">+573</div>
    <div class="flex items-center gap-2">
      <span class="text-sm font-medium px-2 py-1 rounded-full" style="background-color: ${currentPalette[1]}20; color: ${currentPalette[1]}">
        +201%
      </span>
      <span class="text-xs text-gray-400">from last month</span>
    </div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.border-gray-700 { border-color: #374151; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.rounded-xl { border-radius: 0.75rem; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.duration-300 { transition-duration: 300ms; }
.hover\\:shadow-xl:hover { box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1); }
</style>`,

    'calendar-widget': `<!-- Calendar Widget -->
<div class="bg-gray-800 rounded-xl p-4 md:p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all">
  <div class="flex justify-between items-center mb-4 md:mb-6">
    <h3 class="font-semibold text-base md:text-lg text-white">Calendar</h3>
    <div class="flex items-center gap-2 md:gap-3">
      <button class="text-gray-400 hover:text-white p-1 md:p-2 rounded-lg hover:bg-gray-700 transition-colors">
        <svg class="w-3 h-3 md:w-4 md:h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
        </svg>
      </button>
      <span class="font-medium text-white text-sm md:text-base">June 2025</span>
      <button class="text-gray-400 hover:text-white p-1 md:p-2 rounded-lg hover:bg-gray-700 transition-colors">
        <svg class="w-3 h-3 md:w-4 md:h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
        </svg>
      </button>
    </div>
  </div>
  <div class="grid grid-cols-7 gap-1 mb-2 md:mb-4">
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">Su</div>
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">Mo</div>
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">Tu</div>
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">We</div>
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">Th</div>
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">Fr</div>
    <div class="text-center text-xs font-medium text-gray-400 py-1 md:py-2">Sa</div>
  </div>
  <div class="grid grid-cols-7 gap-1">
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-500">1</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">2</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">3</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">4</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-white rounded shadow-lg cursor-pointer" style="background-color: ${currentPalette[0]}">5</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">6</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">7</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">8</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">9</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">10</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">11</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">12</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">13</div>
    <div class="text-center py-1 md:py-2 text-xs md:text-sm text-gray-300 hover:bg-gray-700 rounded cursor-pointer">14</div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.bg-gray-700 { background-color: #374151; }
.border-gray-700 { border-color: #374151; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.hover\\:bg-gray-700:hover { background-color: #374151; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.text-gray-300 { color: #d1d5db; }
.text-gray-500 { color: #6b7280; }
.rounded-xl { border-radius: 0.75rem; }
.rounded-lg { border-radius: 0.5rem; }
.rounded { border-radius: 0.25rem; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.transition-colors { transition-property: color, background-color, border-color, text-decoration-color, fill, stroke; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
</style>`,

    'activity-goal': `<!-- Activity Goal Widget -->
<div class="bg-gray-800 rounded-xl p-4 md:p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all">
  <div class="mb-4 md:mb-6">
    <h3 class="font-semibold text-base md:text-lg text-white mb-2">Move Goal</h3>
    <p class="text-xs md:text-sm text-gray-400">Set your daily activity goal.</p>
  </div>
  <div class="flex items-center justify-between mb-4 md:mb-6">
    <button class="w-10 h-10 md:w-12 md:h-12 rounded-full border-2 border-gray-600 flex items-center justify-center hover:border-gray-500 transition-colors hover:bg-gray-700">
      <svg class="w-4 h-4 md:w-5 md:h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4"></path>
      </svg>
    </button>
    <div class="text-center">
      <div class="text-3xl md:text-4xl font-bold text-white mb-1">350</div>
      <div class="text-xs text-gray-400 font-medium tracking-wider">CALORIES/DAY</div>
    </div>
    <button class="w-10 h-10 md:w-12 md:h-12 rounded-full border-2 border-gray-600 flex items-center justify-center hover:border-gray-500 transition-colors hover:bg-gray-700">
      <svg class="w-4 h-4 md:w-5 md:h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"></path>
      </svg>
    </button>
  </div>
  <div class="mb-4 md:mb-6">
    <div class="grid grid-cols-10 gap-1 h-12 md:h-16 items-end">
      <div class="rounded-sm" style="background-color: ${currentPalette[0]}; height: 60%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[1]}; height: 80%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[2]}; height: 45%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[0]}; height: 90%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[3]}; height: 70%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[1]}; height: 55%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[2]}; height: 85%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[0]}; height: 40%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[3]}; height: 75%"></div>
      <div class="rounded-sm" style="background-color: ${currentPalette[1]}; height: 65%"></div>
    </div>
  </div>
  <button class="w-full py-2 md:py-3 rounded-lg text-xs md:text-sm font-semibold transition-all hover:shadow-lg hover:scale-105 transform" style="background-color: ${currentPalette[1]}; color: white">
    Set Goal
  </button>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.bg-gray-700 { background-color: #374151; }
.border-gray-700 { border-color: #374151; }
.border-gray-600 { border-color: #4b5563; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.hover\\:border-gray-500:hover { border-color: #6b7280; }
.hover\\:bg-gray-700:hover { background-color: #374151; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.rounded-xl { border-radius: 0.75rem; }
.rounded-lg { border-radius: 0.5rem; }
.rounded-sm { border-radius: 0.125rem; }
.rounded-full { border-radius: 9999px; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.transition-colors { transition-property: color, background-color, border-color, text-decoration-color, fill, stroke; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.hover\\:shadow-lg:hover { box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1); }
.hover\\:scale-105:hover { transform: scale(1.05); }
.transform { transform: translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y)); }
</style>`,

    'team-activity': `<!-- Team Activity -->
<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all">
  <h3 class="font-semibold text-lg text-white mb-6">Team Activity</h3>
  <div class="space-y-4">
    <div class="flex items-center gap-4">
      <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold" style="background-color: ${currentPalette[0]}">
        JD
      </div>
      <div class="flex-1">
        <div class="text-sm text-white font-medium">John Doe</div>
        <div class="text-xs text-gray-400">Completed project milestone</div>
      </div>
      <div class="text-xs text-gray-500">2m ago</div>
    </div>
    <div class="flex items-center gap-4">
      <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold" style="background-color: ${currentPalette[1]}">
        SM
      </div>
      <div class="flex-1">
        <div class="text-sm text-white font-medium">Sarah Miller</div>
        <div class="text-xs text-gray-400">Updated design system</div>
      </div>
      <div class="text-xs text-gray-500">5m ago</div>
    </div>
    <div class="flex items-center gap-4">
      <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold" style="background-color: ${currentPalette[2]}">
        MJ
      </div>
      <div class="flex-1">
        <div class="text-sm text-white font-medium">Mike Johnson</div>
        <div class="text-xs text-gray-400">Deployed to production</div>
      </div>
      <div class="text-xs text-gray-500">12m ago</div>
    </div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.border-gray-700 { border-color: #374151; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.text-gray-500 { color: #6b7280; }
.rounded-xl { border-radius: 0.75rem; }
.rounded-full { border-radius: 9999px; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.space-y-4 > :not([hidden]) ~ :not([hidden]) { margin-top: 1rem; }
</style>`,

    'notifications': `<!-- Recent Notifications -->
<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all">
  <h3 class="font-semibold text-lg text-white mb-6">Recent Notifications</h3>
  <div class="space-y-4">
    <div class="flex items-start gap-4 p-3 rounded-lg border-l-4 bg-gray-700/50" style="border-left-color: ${currentPalette[0]}">
      <div class="w-2 h-2 rounded-full mt-2" style="background-color: ${currentPalette[0]}"></div>
      <div class="flex-1">
        <div class="text-sm text-white font-medium">New user registered</div>
        <div class="text-xs text-gray-400 mt-1">A new user has joined your platform</div>
        <div class="text-xs text-gray-500 mt-2">10 minutes ago</div>
      </div>
    </div>
    <div class="flex items-start gap-4 p-3 rounded-lg border-l-4 bg-gray-700/50" style="border-left-color: ${currentPalette[1]}">
      <div class="w-2 h-2 rounded-full mt-2" style="background-color: ${currentPalette[1]}"></div>
      <div class="flex-1">
        <div class="text-sm text-white font-medium">Payment received</div>
        <div class="text-xs text-gray-400 mt-1">$299.00 payment processed successfully</div>
        <div class="text-xs text-gray-500 mt-2">25 minutes ago</div>
      </div>
    </div>
    <div class="flex items-start gap-4 p-3 rounded-lg border-l-4 bg-gray-700/50" style="border-left-color: ${currentPalette[2]}">
      <div class="w-2 h-2 rounded-full mt-2" style="background-color: ${currentPalette[2]}"></div>
      <div class="flex-1">
        <div class="text-sm text-white font-medium">Server maintenance</div>
        <div class="text-xs text-gray-400 mt-1">Scheduled maintenance completed</div>
        <div class="text-xs text-gray-500 mt-2">1 hour ago</div>
      </div>
    </div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.bg-gray-700\\/50 { background-color: rgb(55 65 81 / 0.5); }
.border-gray-700 { border-color: #374151; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.text-gray-500 { color: #6b7280; }
.rounded-xl { border-radius: 0.75rem; }
.rounded-lg { border-radius: 0.5rem; }
.rounded-full { border-radius: 9999px; }
.border-l-4 { border-left-width: 4px; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.space-y-4 > :not([hidden]) ~ :not([hidden]) { margin-top: 1rem; }
</style>`,

    'revenue-analytics': `<!-- Revenue Analytics -->
<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all">
  <div class="flex justify-between items-center mb-6">
    <h3 class="font-semibold text-lg text-white">Revenue Analytics</h3>
    <button class="text-xs px-3 py-1 rounded-full border border-gray-600 text-gray-300 hover:bg-gray-700 transition-colors">
      View Details
    </button>
  </div>
  <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
    <div class="text-center">
      <div class="text-2xl font-bold text-white mb-1">$24.5K</div>
      <div class="text-xs text-gray-400">Monthly Revenue</div>
      <div class="text-xs font-medium mt-1" style="color: ${currentPalette[0]}">+12.5%</div>
    </div>
    <div class="text-center">
      <div class="text-2xl font-bold text-white mb-1">$89</div>
      <div class="text-xs text-gray-400">Avg Order Value</div>
      <div class="text-xs font-medium mt-1" style="color: ${currentPalette[1]}">+8.2%</div>
    </div>
    <div class="text-center">
      <div class="text-2xl font-bold text-white mb-1">1,234</div>
      <div class="text-xs text-gray-400">Total Orders</div>
      <div class="text-xs font-medium mt-1" style="color: ${currentPalette[2]}">+15.3%</div>
    </div>
    <div class="text-center">
      <div class="text-2xl font-bold text-white mb-1">2.1%</div>
      <div class="text-xs text-gray-400">Refund Rate</div>
      <div class="text-xs font-medium mt-1" style="color: ${currentPalette[3]}">-0.5%</div>
    </div>
  </div>
  <div class="h-32 flex items-end justify-between gap-2">
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[0]}, ${currentPalette[0]}80); height: 60%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[1]}, ${currentPalette[1]}80); height: 80%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[2]}, ${currentPalette[2]}80); height: 45%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[0]}, ${currentPalette[0]}80); height: 90%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[3]}, ${currentPalette[3]}80); height: 70%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[1]}, ${currentPalette[1]}80); height: 55%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[2]}, ${currentPalette[2]}80); height: 85%"></div>
    <div class="flex-1 rounded-t-sm" style="background: linear-gradient(to top, ${currentPalette[0]}, ${currentPalette[0]}80); height: 40%"></div>
  </div>
</div>

<style>
.bg-gray-800 { background-color: #1f2937; }
.bg-gray-700 { background-color: #374151; }
.border-gray-700 { border-color: #374151; }
.border-gray-600 { border-color: #4b5563; }
.hover\\:border-gray-600:hover { border-color: #4b5563; }
.hover\\:bg-gray-700:hover { background-color: #374151; }
.text-white { color: #ffffff; }
.text-gray-400 { color: #9ca3af; }
.text-gray-300 { color: #d1d5db; }
.rounded-xl { border-radius: 0.75rem; }
.rounded-full { border-radius: 9999px; }
.rounded-t-sm { border-top-left-radius: 0.125rem; border-top-right-radius: 0.125rem; }
.transition-all { transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.transition-colors { transition-property: color, background-color, border-color, text-decoration-color, fill, stroke; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms; }
.h-32 { height: 8rem; }
.flex-1 { flex: 1 1 0%; }
</style>`
  };
  
  // @ts-ignore
  return componentCodes[componentType] || '';
}

  // @ts-ignore

  function showComponentCode(componentType, componentName) {
    selectedComponent = componentName;
    componentCode = generateComponentCode(componentType);
    showModal = true;
  }

  async function copyComponentCode() {
    try {
      await navigator.clipboard.writeText(componentCode);
      showToast = true;
      toastMessage = 'Component code copied to clipboard!';
      setTimeout(() => {
        showToast = false;
      }, 3000);
    } catch (err) {
      console.error('Failed to copy: ', err);
      // Fallback for browsers that don't support clipboard API
      const textArea = document.createElement('textarea');
      textArea.value = componentCode;
      document.body.appendChild(textArea);
      textArea.select();
      document.execCommand('copy');
      document.body.removeChild(textArea);
      showToast = true;
      toastMessage = 'Component code copied to clipboard!';
      setTimeout(() => {
        showToast = false;
      }, 3000);
    }
  }
</script>

<!-- Added touch event listener for mobile -->
<svelte:window onkeydown={handleKeydown} ontouchstart={handleTouch} />

<div class="bg-gray-50 flex flex-col min-h-screen font-sans">
  <header class="py-6 md:py-8 px-4 text-center">
    <h1 class="text-3xl md:text-4xl font-bold text-gray-800 mb-2">Tailwind Palette Generator</h1>
    <!-- Updated instructions to include mobile touch functionality -->
    <p class="text-gray-600 max-w-2xl mx-auto text-sm md:text-base">
      Press spacebar or tap anywhere to generate new color palettes. When you find colors you like,
      copy the Tailwind CSS configuration to use in your projects.
    </p>
  </header>

  <main class="flex-grow flex flex-col items-center justify-center px-4">
    <div class="max-w-4xl w-full">
      <!-- Color Palette Display -->
      <!-- Improved mobile responsiveness with better grid layout -->
      <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-3 md:gap-4 mb-6 md:mb-8">
        {#each currentPalette as color, index}
          <div 
            class="color-swatch h-48 md:h-64 rounded-lg flex flex-col justify-between p-3 md:p-4 cursor-pointer transition-all duration-300 hover:-translate-y-1 hover:shadow-lg"
            style="background-color: {color}"
          >
            <div class="bg-black bg-opacity-20 text-white px-2 md:px-3 py-1 rounded-md text-xs md:text-sm font-mono">
              {color}
            </div>
            
            <button 
              aria-label="Copy color"
              onclick={() => { navigator.clipboard.writeText(color); showToast = true; setTimeout(() => { showToast = false; }, 3000); }}
              class="bg-white bg-opacity-20 text-white p-2 rounded-full hover:bg-opacity-30 transition-all self-end"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 md:h-5 md:w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M5 9V7a5 5 0 0110 0v2a2 2 0 012 2v5a2 2 0 01-2-2H5a2 2 0 01-2-2v-5a2 2 0 012-2zm8-2v2H7V7a3 3 0 016 0z" clip-rule="evenodd" />
              </svg>
            </button>
          </div>
        {/each}
      </div>
      
      <!-- Tailwind Config Output -->
      <!-- Improved mobile responsiveness for config section -->
      <div class="bg-white rounded-lg shadow-md p-4 md:p-6 mt-6 md:mt-8">
        <div class="flex flex-col sm:flex-row sm:justify-between sm:items-center mb-4 gap-3">
          <h2 class="text-lg md:text-xl font-semibold text-gray-800">Tailwind CSS Configuration</h2>
          <button 
            onclick={copyToClipboard}
            class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-md transition-colors text-sm md:text-base"
            aria-label="Copy Tailwind config"
          >Copy Code 
          </button>
        </div>
        <pre class="bg-gray-100 p-3 md:p-4 rounded-md text-xs md:text-sm text-gray-800 overflow-x-auto">{tailwindConfig}</pre>
      </div>

      <!-- Dashboard Preview Section -->
      <!-- Enhanced mobile responsiveness for dashboard preview -->
      <div class="bg-white rounded-lg shadow-md p-4 md:p-6 mt-6 md:mt-8">
        <div class="flex flex-col sm:flex-row sm:justify-between sm:items-center mb-4 md:mb-6 gap-3">
          <h2 class="text-lg md:text-xl font-semibold text-gray-800">Dashboard Preview</h2>
          <div class="text-xs md:text-sm text-gray-600">
            Click components to copy their code
          </div>
        </div>

        <!-- Improved mobile layout for dashboard -->
        <div class="bg-gray-900 rounded-xl p-4 md:p-8 text-white">
          <!-- Dashboard Header -->
          <div class="flex flex-col sm:flex-row sm:justify-between sm:items-center mb-6 md:mb-8 gap-4">
            <div>
              <h1 class="text-xl md:text-2xl font-bold text-white mb-1">Dashboard</h1>
              <p class="text-gray-400 text-xs md:text-sm">Welcome back! Here's what's happening.</p>
            </div>
            <div class="flex gap-2 md:gap-3">
              <button class="px-3 md:px-4 py-2 rounded-lg text-xs md:text-sm font-medium transition-all hover:shadow-lg" style="background-color: {currentPalette[0]}">
                Export Data
              </button>
              <button class="px-3 md:px-4 py-2 rounded-lg text-xs md:text-sm font-medium bg-gray-800 hover:bg-gray-700 transition-colors border border-gray-700">
                Settings
              </button>
            </div>
          </div>

          <!-- Stats Grid -->
          <!-- Improved mobile grid layout for stats -->
          <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-6 mb-6 md:mb-8">
            <!-- Active Users Card -->
            <div 
              class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-gray-600 transition-all duration-300 hover:shadow-xl cursor-pointer"
              role="button" tabindex="0"
              onclick={() => showComponentCode('stats-card', 'Stats Card')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('stats-card', 'Stats Card');
                }
              }}
            >
              <div class="flex items-center justify-between mb-4">
                <div class="p-3 rounded-lg" style="background-color: {currentPalette[1]}20; color: {currentPalette[1]}">
                  <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z"></path>
                  </svg>
                </div>
                <div class="text-right">
                  <div class="text-sm text-gray-400 font-medium">Active Users</div>
                </div>
              </div>
              <div class="mb-4">
                <div class="text-3xl font-bold text-white mb-1">+2,350</div>
                <div class="flex items-center gap-2">
                  <span class="text-sm font-medium px-2 py-1 rounded-full" style="background-color: {currentPalette[2]}20; color: {currentPalette[2]}">
                    +180.1%
                  </span>
                  <span class="text-xs text-gray-400">from last month</span>
                </div>
              </div>
              <!-- Enhanced wave chart -->
              <div class="h-16 relative">
                <svg viewBox="0 0 200 60" class="w-full h-full">
                  <defs>
                    <linearGradient id="waveGradient" x1="0%" y1="0%" x2="100%" y2="0%">
                      <stop offset="0%" style="stop-color:{currentPalette[1]};stop-opacity:1" />
                      <stop offset="50%" style="stop-color:{currentPalette[2]};stop-opacity:1" />
                      <stop offset="100%" style="stop-color:{currentPalette[0]};stop-opacity:1" />
                    </linearGradient>
                    <filter id="waveGlow">
                      <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
                      <feMerge> 
                        <feMergeNode in="coloredBlur"/>
                        <feMergeNode in="SourceGraphic"/>
                      </feMerge>
                  </filter>
                  </defs>
                  <path 
                    d="M0,40 Q25,20 50,25 T100,22 Q125,15 150,30 T200,28" 
                    stroke="url(#waveGradient)" 
                    stroke-width="3" 
                    fill="none"
                    filter="url(#waveGlow)"
                  />
                  <!-- Enhanced data points -->
                  {#each [25, 40, 30, 22, 35, 30, 28] as y, i}
                    <circle 
                      cx={i * 30 + 10} 
                      cy={y} 
                      r="4" 
                      fill={currentPalette[1]}
                      stroke={currentPalette[0]}
                      stroke-width="2"
                      class="drop-shadow-lg"
                    />
                  {/each}
                </svg>
              </div>
            </div>

            <!-- Sales Card -->
            <div 
              class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-gray-600 transition-all duration-300 hover:shadow-xl cursor-pointer"
              role="button" tabindex="0"
              onclick={() => showComponentCode('sales-card', 'Sales Card')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('sales-card', 'Sales Card');
                }
              }}
            >
              <div class="flex items-center justify-between mb-4">
                <div class="p-3 rounded-lg" style="background-color: {currentPalette[2]}20; color: {currentPalette[2]}">
                  <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 11V7a4 4 0 00-8 0v4M8 11v6a2 2 0 002 2h4a2 2 0 002-2v-6M8 11h8"></path>
                  </svg>
                </div>
                <div class="text-right">
                  <div class="text-sm text-gray-400 font-medium">Total Sales</div>
                </div>
              </div>
              <div class="mb-4">
                <div class="text-3xl font-bold text-white mb-1">12,234</div>
                <div class="flex items-center gap-2">
                  <span class="text-sm font-medium px-2 py-1 rounded-full" style="background-color: {currentPalette[0]}20; color: {currentPalette[0]}">
                    +19%
                  </span>
                  <span class="text-xs text-gray-400">from last month</span>
                </div>
              </div>
              <!-- Circular progress -->
              <div class="flex items-center justify-center h-16">
                <div class="relative w-16 h-16">
                  <svg class="w-16 h-16 transform -rotate-90" viewBox="0 0 64 64">
                    <circle cx="32" cy="32" r="28" stroke="#374151" stroke-width="4" fill="none"/>
                    <circle 
                      cx="32" 
                      cy="32" 
                      r="28" 
                      stroke={currentPalette[2]} 
                      stroke-width="4" 
                      fill="none"
                      stroke-dasharray="175.93"
                      stroke-dashoffset="52.78"
                      stroke-linecap="round"
                    />
                  </svg>
                  <div class="absolute inset-0 flex items-center justify-center">
                    <span class="text-sm font-bold" style="color: {currentPalette[2]}">70%</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Conversion Rate Card -->
            <div 
              class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-gray-600 transition-all duration-300 hover:shadow-xl cursor-pointer"
              role="button" tabindex="0"
              onclick={() => showComponentCode('conversion-card', 'Conversion Rate Card')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('conversion-card', 'Conversion Rate Card');
                }
              }}
            >
              <div class="flex items-center justify-between mb-4">
                <div class="p-3 rounded-lg" style="background-color: {currentPalette[3]}20; color: {currentPalette[3]}">
                  <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path>
                  </svg>
                </div>
                <div class="text-right">
                  <div class="text-sm text-gray-400 font-medium">Conversion</div>
                </div>
              </div>
              <div class="mb-4">
                <div class="text-3xl font-bold text-white mb-1">3.2%</div>
                <div class="flex items-center gap-2">
                  <span class="text-sm font-medium px-3 py-1 rounded-full" style="background-color: {currentPalette[3]}20; color: {currentPalette[3]}">
                    +0.5%
                  </span>
                  <span class="text-xs text-gray-400">from last month</span>
                </div>
              </div>
              <!-- Step progress -->
              <div class="flex items-center gap-2 h-16">
                {#each Array(5) as _, i}
                  <div class="flex-1 flex flex-col gap-1">
                    <div 
                      class="h-8 rounded-sm transition-all duration-300" 
                      style="background-color: {i < 3 ? currentPalette[3] : '#374151'}; opacity: {i < 3 ? 1 : 0.3}"
                    ></div>
                    <div 
                      class="h-6 rounded-sm transition-all duration-300" 
                      style="background-color: {i < 4 ? currentPalette[3] : '#374151'}; opacity: {i < 4 ? 0.7 : 0.2}"
                    ></div>
                  </div>
                {/each}
              </div>
            </div>
          </div>

          <!-- Enhanced mobile layout for bottom section -->
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 md:gap-8 h-full">
            <!-- Calendar Widget -->
            <div 
              class="bg-gray-800 rounded-xl p-4 md:p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all flex flex-col flex-1 min-h-[350px]"
              role="button" tabindex="0"
              onclick={() => showComponentCode('calendar-widget', 'Calendar Widget')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('calendar-widget', 'Calendar Widget');
                }
              }}
            >
              <div class="flex justify-between items-center mb-4 md:mb-6">
                <h3 class="font-semibold text-base md:text-lg text-white">Calendar</h3>
                <div class="flex items-center gap-2 md:gap-3">
                  <button class="text-gray-400 hover:text-white p-1 md:p-2 rounded-lg hover:bg-gray-700 transition-colors" aria-label="Previous Month">
                    <svg class="w-3 h-3 md:w-4 md:h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
                    </svg>
                  </button>
                  <span class="font-medium text-white text-sm md:text-base">June 2025</span>
                  <button class="text-gray-400 hover:text-white p-1 md:p-2 rounded-lg hover:bg-gray-700 transition-colors" aria-label="Next Month">
                    <svg class="w-3 h-3 md:w-4 md:h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                    </svg>
                  </button>
                </div>
              </div>
              <div class="grid grid-cols-7 gap-1 md:gap-2">
                {#each ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa'] as day}
                  <div class="text-center text-gray-400 py-2 md:py-3 text-xs md:text-sm font-medium">{day}</div>
                {/each}
                {#each Array(30) as _, i}
                  <div 
                    class="text-center py-2 md:py-3 rounded-lg cursor-pointer transition-all text-xs md:text-sm font-medium {i === 4 ? 'text-white shadow-lg' : 'text-gray-300 hover:text-white hover:bg-gray-700'}"
                    style="background-color: {i === 4 ? currentPalette[0] : 'transparent'}"
                  >
                    {i + 1}
                  </div>
                {/each}
              </div>
            </div>

            <!-- Activity Goal Widget -->
            <div 
              class="bg-gray-800 rounded-xl p-4 md:p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all flex flex-col flex-1 min-h-[350px]"
              role="button" tabindex="0"
              onclick={() => showComponentCode('activity-goal', 'Activity Goal Widget')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('activity-goal', 'Activity Goal Widget');
                }
              }}
            >
              <div class="mb-4 md:mb-6">
                <h3 class="font-semibold text-base md:text-lg text-white mb-2">Daily Goal</h3>
                <p class="text-xs md:text-sm text-gray-400">Track your daily activity progress</p>
              </div>
              <div class="flex items-center justify-between mb-4 md:mb-6">
                <button class="w-10 h-10 md:w-12 md:h-12 rounded-full border-2 border-gray-600 flex items-center justify-center hover:border-gray-500 transition-colors hover:bg-gray-700" aria-label="Decrease Goal">
                  <svg class="w-6 h-6 md:w-7 md:h-7 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <line x1="6" y1="12" x2="18" y2="12" stroke="currentColor" stroke-width="3" stroke-linecap="round" />
                  </svg>
                </button>
                <div class="text-center">
                  <div class="text-3xl md:text-4xl font-bold text-white mb-1">350</div>
                  <div class="text-xs text-gray-400 font-medium tracking-wider">CALORIES/DAY</div>
                </div>
                <button class="w-10 h-10 md:w-12 md:h-12 rounded-full border-2 border-gray-600 flex items-center justify-center hover:border-gray-500 transition-colors hover:bg-gray-700" aria-label="Increase Goal">
                  <svg class="w-6 h-6 md:w-7 md:h-7 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <line x1="12" y1="6" x2="12" y2="18" stroke="currentColor" stroke-width="3" stroke-linecap="round" />
                    <line x1="6" y1="12" x2="18" y2="12" stroke="currentColor" stroke-width="3" stroke-linecap="round" />
                  </svg>
                </button>
              </div>
              <!-- Enhanced progress visualization -->
              <div class="mb-4 md:mb-6">
                <div class="flex justify-between items-center mb-3">
                  <span class="text-xs md:text-sm text-gray-400">Progress</span>
                  <span class="text-xs md:text-sm font-medium" style="color: {currentPalette[0]}">67%</span>
                </div>
                <div class="w-full bg-gray-700 rounded-full h-2 md:h-3">
                  <div 
                    class="h-2 md:h-3 rounded-full transition-all duration-500" 
                    style="background: linear-gradient(to right, {currentPalette[0]}, {currentPalette[1]}); width: 67%"
                  ></div>
                </div>
              </div>
              <button 
                class="w-full py-2 md:py-3 rounded-lg text-xs md:text-sm font-semibold transition-all hover:shadow-lg hover:scale-105 transform"
                style="background-color: {currentPalette[1]}; color: white"
              >
                Update Goal
              </button>
            </div>
          </div>

          <!-- Team Activity & Recent Notifications Section -->
          <div class="mt-8 grid grid-cols-1 lg:grid-cols-2 gap-8">
            <!-- Team Activity -->
            <div 
              class="bg-gray-800 rounded-xl p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all"
              role="button" tabindex="0"
              onclick={() => showComponentCode('team-activity', 'Team Activity')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('team-activity', 'Team Activity');
                }
              }}
            >
              <div class="flex justify-between items-center mb-6">
                <h3 class="font-semibold text-lg text-white">Team Activity</h3>
                <button class="text-sm text-gray-400 hover:text-white transition-colors">View All</button>
              </div>
              <div class="space-y-4">
                <div class="flex items-center gap-4">
                  <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold text-sm" style="background-color: {currentPalette[0]}">
                    JD
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium">John Doe</div>
                    <div class="text-sm text-gray-400">Completed project milestone</div>
                  </div>
                  <div class="text-xs text-gray-500">2m ago</div>
                </div>
                <div class="flex items-center gap-4">
                  <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold text-sm" style="background-color: {currentPalette[1]}">
                    SM
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium">Sarah Miller</div>
                    <div class="text-sm text-gray-400">Updated design system</div>
                  </div>
                  <div class="text-xs text-gray-500">15m ago</div>
                </div>
                <div class="flex items-center gap-4">
                  <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold text-sm" style="background-color: {currentPalette[2]}">
                    MJ
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium">Mike Johnson</div>
                    <div class="text-sm text-gray-400">Deployed to production</div>
                  </div>
                  <div class="text-xs text-gray-500">1h ago</div>
                </div>
                <div class="flex items-center gap-4">
                  <div class="w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold text-sm" style="background-color: {currentPalette[3]}">
                    AL
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium">Anna Lee</div>
                    <div class="text-sm text-gray-400">Created new user story</div>
                  </div>
                  <div class="text-xs text-gray-500">3h ago</div>
                </div>
              </div>
            </div>

            <!-- Recent Notifications -->
            <div 
              class="bg-gray-800 rounded-xl p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all"
              role="button" tabindex="0"
              onclick={() => showComponentCode('notifications', 'Notifications Panel')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('notifications', 'Notifications Panel');
                }
              }}
            >
              <div class="flex justify-between items-center mb-6">
                <h3 class="font-semibold text-lg text-white">Recent Notifications</h3>
                <div class="flex items-center gap-2">
                  <div class="w-2 h-2 rounded-full" style="background-color: {currentPalette[0]}"></div>
                  <span class="text-xs text-gray-400">3 new</span>
                </div>
              </div>
              <div class="space-y-4">
                <div class="flex items-start gap-4 p-3 rounded-lg bg-gray-700 border-l-4" style="border-left-color: {currentPalette[0]}">
                  <div class="p-2 rounded-lg" style="background-color: {currentPalette[0]}20; color: {currentPalette[0]}">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium text-sm">System Alert</div>
                    <div class="text-xs text-gray-400 mt-1">Server maintenance scheduled for tonight</div>
                    <div class="text-xs text-gray-500 mt-2">5 minutes ago</div>
                  </div>
                </div>
                <div class="flex items-start gap-4 p-3 rounded-lg bg-gray-700 border-l-4" style="border-left-color: {currentPalette[1]}">
                  <div class="p-2 rounded-lg" style="background-color: {currentPalette[1]}20; color: {currentPalette[1]}">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium text-sm">Task Completed</div>
                    <div class="text-sm text-gray-400 mt-1">Database backup completed successfully</div>
                    <div class="text-xs text-gray-500 mt-2">1 hour ago</div>
                  </div>
                </div>
                <div class="flex items-start gap-4 p-3 rounded-lg bg-gray-700 border-l-4" style="border-left-color: {currentPalette[2]}">
                  <div class="p-2 rounded-lg" style="background-color: {currentPalette[2]}20; color: {currentPalette[2]}">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="text-white font-medium text-sm">New User</div>
                    <div class="text-sm text-gray-400 mt-1">3 new users registered today</div>
                    <div class="text-xs text-gray-500 mt-2">2 hours ago</div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Revenue Data Section -->
          <div class="mt-8">
            <div 
              class="bg-gray-800 rounded-xl p-6 border border-gray-700 cursor-pointer hover:border-gray-600 transition-all"
              role="button" tabindex="0"
              onclick={() => showComponentCode('revenue-analytics', 'Revenue Analytics')}
              onkeydown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                  showComponentCode('revenue-analytics', 'Revenue Analytics');
                }
              }}
            >
              <div class="flex justify-between items-center mb-6">
                <div>
                  <h3 class="font-semibold text-lg text-white mb-1">Revenue Analytics</h3>
                  <p class="text-sm text-gray-400">Monthly revenue breakdown and trends</p>
                </div>
                <div class="flex gap-2">
                  <button class="px-3 py-1 text-xs rounded-full border border-gray-600 text-gray-300 hover:text-white hover:border-gray-500 transition-colors">
                    7D
                  </button>
                  <button class="px-3 py-1 text-xs rounded-full text-white border-2" style="border-color: {currentPalette[0]}; background-color: {currentPalette[0]}20">
                    30D
                  </button>
                  <button class="px-3 py-1 text-xs rounded-full border border-gray-600 text-gray-300 hover:text-white hover:border-gray-500 transition-colors">
                    90D
                  </button>
                </div>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                <!-- Monthly Revenue -->
                <div class="text-center">
                  <div class="text-2xl font-bold text-white mb-1">$45,231</div>
                  <div class="text-sm text-gray-400 mb-2">Monthly Revenue</div>
                  <div class="flex items-center justify-center gap-1">
                    <svg class="w-4 h-4" style="color: {currentPalette[1]}" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M5.293 7.707a1 1 0 010-1.414l4-4a1 1 0 011.414 0l4 4a1 1 0 01-1.414 1.414L11 5.414V17a1 1 0 11-2 0V5.414L6.707 7.707a1 1 0 01-1.414 0z" clip-rule="evenodd"></path>
                    </svg>
                    <span class="text-sm font-medium" style="color: {currentPalette[1]}">+12.5%</span>
                  </div>
                </div>

                <!-- Average Order -->
                <div class="text-center">
                  <div class="text-2xl font-bold text-white mb-1">$127</div>
                  <div class="text-sm text-gray-400 mb-2">Avg Order Value</div>
                  <div class="flex items-center justify-center gap-1">
                    <svg class="w-4 h-4" style="color: {currentPalette[2]}" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M5.293 7.707a1 1 0 010-1.414l4-4a1 1 0 011.414 0l4 4a1 1 0 01-1.414 1.414L9 14.586V3a1 1 0 012 0v11.586l2.293-2.293a1 1 0 01.293.707V19a2 2 0 01-2 2z" clip-rule="evenodd"></path>
                    </svg>
                    <span class="text-sm font-medium" style="color: {currentPalette[2]}">+8.2%</span>
                  </div>
                </div>

                <!-- Total Orders -->
                <div class="text-center">
                  <div class="text-2xl font-bold text-white mb-1">1,429</div>
                  <div class="text-sm text-gray-400 mb-2">Total Orders</div>
                  <div class="flex items-center justify-center gap-1">
                    <svg class="w-4 h-4" style="color: {currentPalette[0]}" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M5.293 7.707a1 1 0 010-1.414l4-4a1 1 0 011.414 0l4 4a1 1 0 01-1.414 1.414L9 14.586V3a1 1 0 012 0v11.586l2.293-2.293a1 1 0 01.293.707V19a2 2 0 01-2 2z" clip-rule="evenodd"></path>
                    </svg>
                    <span class="text-sm font-medium" style="color: {currentPalette[0]}">+15.3%</span>
                  </div>
                </div>

                <!-- Refund Rate -->
                <div class="text-center">
                  <div class="text-2xl font-bold text-white mb-1">2.1%</div>
                  <div class="text-sm text-gray-400 mb-2">Refund Rate</div>
                  <div class="flex items-center justify-center gap-1">
                    <svg class="w-4 h-4" style="color: {currentPalette[3]}" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M14.707 12.293a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L9 14.586V3a1 1 0 012 0v11.586l2.293-2.293a1 1 0 01.293.707V19a2 2 0 01-2 2z" clip-rule="evenodd"></path>
                    </svg>
                    <span class="text-sm font-medium" style="color: {currentPalette[3]}">-0.8%</span>
                  </div>
                </div>
              </div>

              <!-- Revenue Chart -->
              <div class="h-64 relative">
                <div class="absolute top-0 left-0 text-sm text-gray-400 mb-4">Revenue Trend (Last 30 Days)</div>
                <svg viewBox="0 0 400 200" class="w-full h-full mt-6">
                  <defs>
                    <linearGradient id="revenueGradient" x1="0%" y1="0%" x2="0%" y2="100%">
                      <stop offset="0%" style="stop-color:{currentPalette[0]};stop-opacity:0.3" />
                      <stop offset="100%" style="stop-color:{currentPalette[0]};stop-opacity:0" />
                    </linearGradient>
                    <filter id="revenueGlow">
                      <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
                      <feMerge> 
                        <feMergeNode in="coloredBlur"/>
                        <feMergeNode in="SourceGraphic"/>
                      </feMerge>
                    </filter>
                  </defs>
                  
                  <!-- Grid lines -->
                  {#each Array(5) as _, i}
                    <line 
                      x1="40" 
                      y1={40 + i * 30} 
                      x2="380" 
                      y2={40 + i * 30} 
                      stroke="#374151" 
                      stroke-width="1"
                      opacity="0.3"
                    />
                  {/each}
                  
                  <!-- Revenue area -->
                  <path 
                    d="M40,160 Q80,140 120,130 T200,120 Q240,110 280,115 T360,100 L360,180 L40,180 Z" 
                    fill="url(#revenueGradient)"
                  />
                  
                  <!-- Revenue line -->
                  <path 
                    d="M40,160 Q80,140 120,130 T200,120 Q240,110 280,115 T360,100" 
                    stroke={currentPalette[0]} 
                    stroke-width="3" 
                    fill="none"
                    filter="url(#revenueGlow)"
                  />
                  
                  <!-- Data points -->
                  {#each [160, 140, 130, 120, 110, 115, 100] as y, i}
                    <circle 
                      cx={40 + i * 53.33} 
                      cy={y} 
                      r="5" 
                      fill={currentPalette[0]}
                      stroke="#1f2937"
                      stroke-width="3"
                      class="drop-shadow-lg"
                    />
                  {/each}
                  
                  <!-- Y-axis labels -->
                  <text x="30" y="45" fill="#9CA3AF" font-size="12" text-anchor="end">$50k</text>
                  <text x="30" y="75" fill="#9CA3AF" font-size="12" text-anchor="end">$40k</text>
                  <text x="30" y="105" fill="#9CA3AF" font-size="12" text-anchor="end">$30k</text>
                  <text x="30" y="135" fill="#9CA3AF" font-size="12" text-anchor="end">$20k</text>
                  <text x="30" y="165" fill="#9CA3AF" font-size="12" text-anchor="end">$10k</text>
                </svg>
              </div>

              <!-- Revenue Sources -->
              <div class="mt-8 grid grid-cols-1 md:grid-cols-3 gap-6">
                <div class="bg-gray-700 rounded-lg p-4">
                  <div class="flex items-center justify-between mb-3">
                    <span class="text-sm text-gray-300">Direct Sales</span>
                    <span class="text-sm font-medium" style="color: {currentPalette[0]}">68%</span>
                  </div>
                  <div class="w-full bg-gray-600 rounded-full h-2">
                    <div 
                      class="h-2 rounded-full transition-all duration-500" 
                      style="background-color: {currentPalette[0]}; width: 68%"
                    ></div>
                  </div>
                  <div class="text-lg font-semibold text-white mt-2">$30,757</div>
                </div>

                <div class="bg-gray-700 rounded-lg p-4">
                  <div class="flex items-center justify-between mb-3">
                    <span class="text-sm text-gray-300">Partnerships</span>
                    <span class="text-sm font-medium" style="color: {currentPalette[1]}">24%</span>
                  </div>
                  <div class="w-full bg-gray-600 rounded-full h-2">
                    <div 
                      class="h-2 rounded-full transition-all duration-500" 
                      style="background-color: {currentPalette[1]}; width: 24%"
                    ></div>
                  </div>
                  <div class="text-lg font-semibold text-white mt-2">$10,855</div>
                </div>

                <div class="bg-gray-700 rounded-lg p-4">
                  <div class="flex items-center justify-between mb-3">
                    <span class="text-sm text-gray-300">Referrals</span>
                    <span class="text-sm font-medium" style="color: {currentPalette[2]}">8%</span>
                  </div>
                  <div class="w-full bg-gray-600 rounded-full h-2">
                    <div 
                      class="h-2 rounded-full transition-all duration-500" 
                      style="background-color: {currentPalette[2]}; width: 8%"
                    ></div>
                  </div>
                  <div class="text-lg font-semibold text-white mt-2">$3,619</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>

  <footer class="py-6 px-4 text-center text-gray-500 text-sm">
    <p>Press spacebar anytime to generate a new palette</p>
  </footer>

  <!-- Enhanced modal responsiveness for mobile -->
  {#if showModal}
    <div 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-2 md:p-4 z-50" 
      role="button" 
      tabindex="0"
      onclick={() => showModal = false}
      onkeydown={(e) => {
        if (e.key === 'Escape' || e.key === 'Enter' || e.key === ' ') {
          showModal = false;
        }
      }}
      aria-label="Close modal"
    >
      <div 
        class="bg-white rounded-lg shadow-xl max-w-4xl w-full max-h-[90vh] md:max-h-[80vh] overflow-hidden" 
        role="dialog"
        tabindex="-1"
        aria-modal="true"
        aria-labelledby="modal-title"
        onclick={(e) => e.stopPropagation()}
        onkeydown={(e) => e.stopPropagation()}
      >
        <div class="flex flex-col sm:flex-row sm:justify-between sm:items-center p-4 md:p-6 border-b border-gray-200 gap-3">
          <h3 id="modal-title" class="text-lg md:text-xl font-semibold text-gray-800">{selectedComponent} - Component Code</h3>
          <div class="flex gap-2 md:gap-3">
            <button 
              onclick={copyComponentCode}
              class="bg-blue-600 hover:bg-blue-700 text-white px-3 md:px-4 py-2 rounded-md transition-colors text-xs md:text-sm"
            >
              Copy Code
            </button>
            <button 
              onclick={() => showModal = false}
              class="text-gray-400 hover:text-gray-600 p-2" aria-label="Close">
              <svg class="w-5 h-5 md:w-6 md:h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          </div>
        </div>
        <div class="p-4 md:p-6 overflow-y-auto max-h-[70vh] md:max-h-[60vh]">
          <pre class="bg-gray-100 p-3 md:p-4 rounded-md text-xs md:text-sm text-gray-800 overflow-x-auto whitespace-pre-wrap">{componentCode}</pre>
        </div>
      </div>
    </div>
  {/if}

  <!-- Toast Notification -->
  {#if showToast}
    <div class="fixed bottom-4 right-4 bg-green-600 text-white px-6 py-3 rounded-md shadow-lg animate-in slide-in-from-bottom-2 fade-in duration-300">
      Copied to clipboard!
    </div>
  {/if}
</div>

<style>
  :global(body) {
    font-family: 'Inter', sans-serif;
    margin: 0;
    padding: 0;
  }
  
  .color-swatch {
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
    animation: fadeIn 500ms ease forwards;
  }
  
  .color-swatch:hover {
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  }
  
  @keyframes fadeIn {
    from { 
      opacity: 0; 
      transform: translateY(10px); 
    }
    to { 
      opacity: 1; 
      transform: translateY(0); 
    }
  }
</style>
