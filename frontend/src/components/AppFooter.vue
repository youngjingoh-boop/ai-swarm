<template>
  <footer class="app-footer">
    <span class="footer-brand">{{ APP_NAME }} — {{ APP_TAGLINE }}</span>
    <span class="footer-credit">
      Built on <a :href="UPSTREAM_URL" target="_blank" rel="noopener">MiroFish</a>
      · Open source under <a href="https://www.gnu.org/licenses/agpl-3.0.html" target="_blank" rel="noopener">AGPL-3.0</a>
      · <a :href="SOURCE_REPO_URL" target="_blank" rel="noopener">Source</a>
    </span>
  </footer>
</template>

<script setup>
import { APP_NAME, APP_TAGLINE, UPSTREAM_URL, SOURCE_REPO_URL } from '../branding'
</script>

<style scoped>
.app-footer {
  display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 8px;
  padding: 20px 40px; border-top: 1px solid var(--border-light);
  font-size: 13px; color: var(--muted);
}
.app-footer a { color: var(--muted); text-decoration: underline; }
.app-footer a:hover { color: var(--accent); }
.footer-brand { font-weight: 600; color: var(--ink); }
@media (max-width: 640px) { .app-footer { flex-direction: column; align-items: flex-start; } }
</style>
