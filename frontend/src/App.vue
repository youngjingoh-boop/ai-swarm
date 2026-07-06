<template>
  <router-view />
</template>

<script setup>
// 使用 Vue Router 来管理页面
</script>

<style>
:root {
  --accent: #635bff;
  --accent-hover: #5249e6;
  --ink: #1a1f36;
  --muted: #6b7280;
  --border: #e2e4ea;
  --border-light: #edeef2;
  --surface: #ffffff;
  --surface-alt: #f7f8fa;
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Noto Sans SC', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
* { margin: 0; padding: 0; box-sizing: border-box; }
#app {
  font-family: var(--font-sans);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--ink);
  background-color: var(--surface);
}
::-webkit-scrollbar { width: 8px; height: 8px; }
::-webkit-scrollbar-track { background: var(--surface-alt); }
::-webkit-scrollbar-thumb { background: #c2c6d0; border-radius: 4px; }
::-webkit-scrollbar-thumb:hover { background: var(--muted); }
button { font-family: inherit; }
a { color: var(--accent); }
</style>
