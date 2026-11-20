<template>
  <section class="about-page">
    <img src="/favicon.svg" alt="汇图壁纸网图标" />
    <span class="eyebrow">ABOUT HUITU</span>
    <h1>汇图壁纸网</h1>
    <p>为每一块屏幕，收藏值得停留的风景。</p>
    <router-link to="/">返回壁纸画廊</router-link>
  </section>
</template>

<style scoped>
.about-page { max-width: 700px; margin: 12vh auto; padding: 48px; text-align: center; background: white; border: 1px solid var(--brand-line); border-radius: 22px; }
.about-page img { display: block; width: 64px; height: 64px; margin: 0 auto 20px; }
.eyebrow { color: var(--brand-primary); font-size: 11px; letter-spacing: .16em; }
h1 { margin: 12px 0; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: 36px; }
p { margin-bottom: 24px; color: var(--brand-muted); }
@media (max-width: 600px) { .about-page { margin: 6vh auto; padding: 32px 20px; } }
</style>
