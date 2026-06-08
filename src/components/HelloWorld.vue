<template>
  <div class="cy-home">

    <!-- ===== NAVBAR ===== -->
    <nav class="navbar" :class="{ scrolled: isScrolled }">
      <div class="nav-inner">
        <div class="logo">
          <div class="logo-img">
            <img alt="誠揚智能科技 logo" class="logo" src="../assets/誠揚智能科技_logo_無字.png">
          </div>
          <div class="logo-text">
            <span class="logo-cn">誠揚智能科技</span>
            <span class="logo-en">CY Intelligent Technology</span>
          </div>
        </div>
        <ul class="nav-links" :class="{ open: menuOpen }">
          <li v-for="item in navItems" :key="item.label">
            <a :href="item.href" @click="menuOpen = false">{{ item.label }}</a>
          </li>
          <li><a href="#contact" class="nav-cta" @click="menuOpen = false">立即諮詢</a></li>
        </ul>
        <button class="hamburger" @click="menuOpen = !menuOpen" aria-label="menu">
          <span :class="{ active: menuOpen }"></span>
          <span :class="{ active: menuOpen }"></span>
          <span :class="{ active: menuOpen }"></span>
        </button>
      </div>
    </nav>

<!-- ===== VIDEO CAROUSEL ===== -->
<section id="showcase" class="vc-section">

  <!-- 輪播軌道 -->
  <div class="vc-track">
    <div
      v-for="(slide, i) in videoSlides"
      :key="slide.id"
      class="vc-slide"
      :class="{
        active:          currentSlide === i,
        prev:            prevSlide    === i,
        'dir-forward':   slideDir     === 1,
        'dir-backward':  slideDir     === -1,
      }"
    >
      <!-- YouTube iframe 容器：每個 slide 都有一個獨立的 div，供 YT.Player 掛載 -->
      <div
        :id="`yt-player-${i}`"
        class="vc-yt-container"
      ></div>

      <div class="vc-overlay"></div>
    </div>
  </div>

  <!-- 內容過場動畫 -->
  <div class="vc-content">
    <transition
      :name="slideDir >= 0 ? 'vc-slide-left' : 'vc-slide-right'"
      mode="out-in"
    ></transition>
  </div>

  <!-- 分頁圓點 -->
  <div class="vc-dots">
    <button
      v-for="(slide, i) in videoSlides"
      :key="slide.id"
      class="vc-dot"
      :class="{ active: currentSlide === i }"
      @click="goToSlide(i)"
      :aria-label="slide.title"
    >
      <svg class="vc-dot-ring" viewBox="0 0 36 36" fill="none">
        <circle
          class="vc-dot-ring-track"
          cx="18" cy="18" r="15"
          stroke-width="2"
        />
        <circle
          v-if="currentSlide === i"
          class="vc-dot-ring-progress"
          cx="18" cy="18" r="15"
          stroke-width="2"
          :style="{ animationDuration: slideDuration + 'ms' }"
        />
      </svg>
      <span class="vc-dot-inner"></span>
    </button>
  </div>

  <!-- 上一張按鈕 -->
  <button class="vc-arrow vc-arrow-prev" @click="prevSlideHandler" aria-label="previous">
    <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
      <path d="M13 4L7 10L13 16" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
    </svg>
  </button>

  <!-- 下一張按鈕 -->
  <button class="vc-arrow vc-arrow-next" @click="nextSlideHandler" aria-label="next">
    <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
      <path d="M7 4L13 10L7 16" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
    </svg>
  </button>

  <!-- 頁數計數器 -->
  <div class="vc-counter">
    <span class="vc-counter-cur">{{ String(currentSlide + 1).padStart(2, '0') }}</span>
    <span class="vc-counter-sep">/</span>
    <span class="vc-counter-total">{{ String(videoSlides.length).padStart(2, '0') }}</span>
  </div>

</section>


    <!-- ===== HERO SECTION ===== -->
    <section class="hero" ref="heroRef">

      <!-- ===== SVG MASK VISUAL ===== -->
      <!-- 🌟 修改點 1：將事件移到內層的 hotzone，外層保持乾淨 -->
      <div class="mask-stage" ref="maskStageRef">

<img
  ref="posterRef"
  class="mask-poster"
  :class="{ hidden: isHovering }"
  :src="heroPosterSrc"
  alt=""
  aria-hidden="true"
/>

<!-- 影片：同步 counter-scroll -->
<video
  ref="videoRef"
  class="mask-video"
  :class="{ playing: isHovering }"
  :src="heroVideoSrc"
  muted
  loop
  playsinline
  preload="auto"
></video>

        <!-- 漸層動畫替代影片 -->
        <div 
          class="mask-video-placeholder" 
          :class="{ playing: isHovering }" 
          ref="placeholderRef"
        ></div>

        <!-- SVG 遮罩層 -->
        <svg class="mask-svg" viewBox="0 0 600 700" preserveAspectRatio="xMidYMid slice" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <clipPath id="poly-clip" clipPathUnits="userSpaceOnUse"><path :d="polygonPath" fill-rule="evenodd" /></clipPath>
          </defs>
          <rect x="0" y="0" width="600" height="700" fill="var(--hero-bg)" fill-rule="evenodd" :clip-path="'url(#poly-clip)'" style="display:none" />
          <path :d="maskFillPath" fill="var(--hero-bg)" fill-rule="evenodd" class="mask-fill" />
          <polygon :points="polygonPoints" fill="none" stroke="var(--accent)" stroke-width="1.5" stroke-dasharray="6 4" class="poly-border" opacity="0.5" />
        </svg>

        <!-- 🌟 修改點 3：最高層透明熱區，負責接收事件 -->
        <svg  class="mask-hotzone"  viewBox="0 0 600 700"  preserveAspectRatio="xMidYMid slice"  xmlns="http://www.w3.org/2000/svg">  <polygon    :points="polygonPoints"    fill="transparent"    class="poly-hover-area"    @mouseenter="onPolyEnter"    @mouseleave="onPolyLeave"  /></svg>

        <!-- Hover 提示 -->
        <div class="hover-hint" :class="{ visible: isHovering }" aria-hidden="true">
          <div class="hint-dot"></div>
          <span>PLAY</span>
        </div>
      </div>

      <!-- ===== HERO CONTENT ===== -->
<div class="hero-content" :class="{ visible: heroVisible }">

  <div class="hero-eyebrow">
    <span class="eyebrow-line"></span>
    <span class="eyebrow-label">新創 科技夥伴</span>
  </div>

  <h1 class="hero-title">
    以誠入世<span class="title-dot">·</span>智慧揚帆
    <span class="hero-title-sub">
      為每一家企業量身打造<br class="br-desktop"/>數位轉型的最短路徑
    </span>
  </h1>

  <p class="hero-subtitle">
    我們不是傳統 IT 廠商——<br/>
    我們是與您並肩作戰的技術共同創辦人。<br class="br-desktop"/>
    以新創的速度、顧問的深度，不計成本全力以赴，為您的成長提速。
  </p>

  <div class="hero-pills">
    <span class="pill">快速交付</span>
    <span class="pill">彈性報價</span>
    <span class="pill">技術自主</span>
    <span class="pill">長期陪跑</span>
  </div>

  <!-- <div class="hero-stats">
    <div class="stat" v-for="s in stats" :key="s.label">
      <span class="stat-num">{{ s.value }}</span>
      <span class="stat-label">{{ s.label }}</span>
    </div>
  </div> -->

</div>

    </section>

    <!-- ===== SERVICES ===== -->
    <section id="services" class="section services-section">
      <div class="section-inner">
        <div class="section-header">
          <span class="section-badge">SERVICES</span>
          <h2 class="section-title">核心服務項目</h2>
          <p class="section-desc">以前瞻技術為基礎，提供全方位的企業智能化服務</p>
        </div>
        <div class="services-grid">
          <div v-for="(svc, i) in services" :key="svc.title" class="service-card" :style="{ '--delay': i * 0.1 + 's' }">
            <div class="card-bg" :style="getCardBgStyle(svc)"></div>
            <div class="card-overlay"></div>
            <div class="card-header-content"><h3 class="card-title">{{ svc.title }}</h3></div>
            <div class="card-hover-content">
              <h3 class="card-hover-title">{{ svc.title }}</h3>
              <p class="card-desc">{{ svc.desc }}</p>
              <div class="card-tags"><span v-for="tag in svc.tags" :key="tag" class="tag">{{ tag }}</span></div>
              <div class="card-cta"><span class="cta-text">了解更多</span><svg width="16" height="16" viewBox="0 0 18 18" fill="none"><path d="M4 9H14M14 9L10 5M14 9L10 13" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/></svg></div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ===== EXPERIENCES ===== -->
    <section id="experiences" class="section experiences-section">
      <div class="section-inner">
        <div class="section-header">
          <span class="section-badge">EXPERIENCES</span>
          <h2 class="section-title">服務足跡</h2>
          <p class="section-desc">誠揚核心團隊成員深耕產業多年，歷經金融科技、智慧製造、醫療健康與數位零售等多元領域的第一線淬煉。<br/>這些品牌印記，是我們對每一份合作承諾的最佳見證。</p>
        </div>
        <div class="marquee-bar">
          <div class="marquee-track">
            <span v-for="copyIdx in 3" :key="'copy-' + copyIdx" class="marquee-copy">
              <span v-for="tag in techTags" :key="tag.name + '-' + copyIdx" class="marquee-item">
                <span class="marquee-logo-wrap"><img :src="tag.logo" :alt="tag.name + ' logo'" class="marquee-logo-img" loading="lazy"/></span>
              </span>
            </span>
          </div>
        </div>
      </div>
    </section>

  </div>
  <!-- ===== FOOTER ===== -->
<footer class="footer" id="contact">
  <div class="footer-bar">

    <!-- 左側：Logo + 聯絡資訊 -->
    <div class="footer-left">
      <div class="footer-logo">
        <div class="footer-logo-img">
          <img src="../assets/誠揚智能科技_logo_無字.png" alt="誠揚智能科技" />
        </div>
        <div class="footer-logo-text">
          <span class="footer-logo-cn">誠揚智能科技</span>
          <span class="footer-logo-en">CY Intelligent Technology</span>
        </div>
      </div>
      <div class="footer-contact-row">
        <span class="footer-info-item">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="5" width="18" height="14" rx="2"/><path d="M3 7l9 6 9-6"/>
          </svg>
          jermaineching@gmail.com
        </span>
        <span class="footer-info-sep">|</span>
        <span class="footer-info-item">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/>
          </svg>
          高雄市楠梓區建楠路 236 號 3 樓（芯創商務共享中心）
        </span>
      </div>
    </div>

    <!-- 右側：社群 + 連結 + 版權 -->
    <div class="footer-right">
      <div class="footer-social">
        <a href="#" class="social-btn" aria-label="LinkedIn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
            <rect x="2" y="2" width="20" height="20" rx="4" stroke="currentColor" stroke-width="1.8"/>
            <path d="M7 10v7M7 7v.5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
            <path d="M11 17v-4a2 2 0 0 1 4 0v4M11 10v7" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
          </svg>
        </a>
        <a href="#" class="social-btn" aria-label="GitHub">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
            <path d="M9 19c-4.3 1.4-4.3-2.5-6-3m12 5v-3.5c0-1 .1-1.4-.5-2 2.8-.3 5.5-1.4 5.5-6a4.6 4.6 0 0 0-1.3-3.2 4.2 4.2 0 0 0-.1-3.2s-1-.3-3.3 1.3a11.5 11.5 0 0 0-6 0C6.3 2.8 5.3 3.1 5.3 3.1a4.2 4.2 0 0 0-.1 3.2A4.6 4.6 0 0 0 3.9 9.5c0 4.6 2.7 5.7 5.5 6-.6.6-.6 1.2-.5 2V21" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </a>
        <a href="mailto:jermainechiang@gmail.com" class="social-btn" aria-label="Email">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
            <rect x="3" y="5" width="18" height="14" rx="2" stroke="currentColor" stroke-width="1.8"/>
            <path d="M3 7l9 6 9-6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
          </svg>
        </a>
      </div>
      <div class="footer-nav-links">
        <a href="#services">網站地圖</a>
        <span class="footer-nav-sep">|</span>
        <a href="#">隱私權政策</a>
        <span class="footer-nav-sep">|</span>
        <a href="#">服務條款</a>
      </div>
      <p class="footer-copy">
        © 2026 誠揚智能科技股份有限公司 CY INTELLIGENT TECHNOLOGY. ALL RIGHTS RESERVED.
      </p>
    </div>

  </div>
</footer>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

// ── Hero 區塊（本機小檔，保留不動）─────────────────────────────
const posterRef     = ref(null)
const heroVideoSrc  = new URL('../assets/videos/cyit_logo_animation.mp4', import.meta.url).href
const heroPosterSrc = new URL('../assets/images/hero_poster.png', import.meta.url).href

// ── Navbar ────────────────────────────────────────────────────
const isScrolled = ref(false)
const menuOpen   = ref(false)
const navItems   = [
  { label: '服務項目', href: '#services' }
]

// ── Hero refs ─────────────────────────────────────────────────
const canvasRef      = ref(null)
const heroRef        = ref(null)
const maskStageRef   = ref(null)
const videoRef       = ref(null)
const placeholderRef = ref(null)
const heroVisible    = ref(false)
const isHovering     = ref(false)

const polygonPoints = '80,40 480,20 540,200 520,580 360,680 60,660 20,440 40,160'
const maskFillPath  = computed(() =>
  `M0,0 L600,0 L600,700 L0,700 Z M${polygonPoints.split(' ').join(' L')} Z`
)

// ── Video Carousel — YouTube IFrame API ───────────────────────
const videoSlides = [
  { id: 1, youtubeId: 'fo77RQlFo8M', link: '#services'  },
  { id: 2, youtubeId: 'VLit3RKCOaM', link: '#solutions' },
  { id: 3, youtubeId: 'YLCTkeShpJE', link: '#services'  },
  { id: 4, youtubeId: 'vJVbNM0J_ig', link: '#services'  },
]

const currentSlide  = ref(0)
const prevSlide     = ref(null)
const slideDir      = ref(1)
const slideDuration = ref(8000)            // 預設，待 onReady 後更新為真實秒數

const slideDurations = ref(videoSlides.map(() => 8000))  // 每支影片的真實毫秒
const ytPlayers      = ref([])                           // YT.Player 實例陣列
const playerReady    = ref(videoSlides.map(() => false)) // 各 player 是否就緒

let slideTimer = null

function clearTimer() {
  if (slideTimer) { clearTimeout(slideTimer); slideTimer = null }
}

// ── 載入 YouTube IFrame API ───────────────────────────────────
function loadYouTubeAPI() {
  return new Promise((resolve) => {
    // 已載入過
    if (window.YT && window.YT.Player) { resolve(); return }
    // script 已注入，等 callback
    if (document.getElementById('yt-iframe-api')) {
      window.onYouTubeIframeAPIReady = resolve; return
    }
    const tag   = document.createElement('script')
    tag.id      = 'yt-iframe-api'
    tag.src     = 'https://www.youtube.com/iframe_api'
    document.head.appendChild(tag)
    window.onYouTubeIframeAPIReady = resolve
  })
}

// ── 建立所有 YT.Player ────────────────────────────────────────
function initYTPlayers() {
  videoSlides.forEach((slide, i) => {
    const player = new window.YT.Player(`yt-player-${i}`, {
      videoId: slide.youtubeId,
      playerVars: {
        autoplay:       i === 0 ? 1 : 0, // 只有第一個自動播放
        mute:           1,                // 靜音（autoplay 必須）
        controls:       0,                // 隱藏控制列
        showinfo:       0,
        rel:            0,                // 不顯示相關影片
        modestbranding: 1,
        playsinline:    1,
        loop:           0,                // 不循環，靠 ENDED 事件推進
        fs:             0,                // 關閉全螢幕
        iv_load_policy: 3,                // 關閉資訊卡
        disablekb:      1,
        enablejsapi:    1,
      },
      events: {
        onReady:       (e) => onPlayerReady(e, i),
        onStateChange: (e) => onPlayerStateChange(e, i),
      },
    })
    ytPlayers.value[i] = player
  })
}

// ── Player 就緒 ───────────────────────────────────────────────
function onPlayerReady(event, index) {
  playerReady.value[index] = true

  // 取得 iframe 元素，直接用 JS 強制設定滿版樣式
  // （YouTube API 會寫死 width/height attribute，CSS 無法覆蓋，必須用 JS）
  const iframe = event.target.getIframe()
  if (iframe) {
    iframe.removeAttribute('width')
    iframe.removeAttribute('height')
    Object.assign(iframe.style, {
      position:       'absolute',
      top:            '50%',
      left:           '50%',
      width:          '177.78vh',   // 100vh × 16/9，橫向足夠寬
      height:         '100vh',
      minWidth:       '100%',
      minHeight:      '56.25vw',    // 100vw × 9/16，直向足夠高
      transform:      'translate(-50%, -50%)',
      border:         'none',
      pointerEvents:  'none',
    })
  }

  // 取得真實時長（秒）→ 毫秒
  const durationSec = event.target.getDuration()
  if (durationSec > 0) {
    slideDurations.value[index] = Math.round(durationSec * 1000)
  }

  // 若是當前 active slide，更新 dot ring 時長並確保播放
  if (index === currentSlide.value) {
    slideDuration.value = slideDurations.value[index]
    event.target.playVideo()
  }
}

// ── 偵測影片結束 → 自動推進 ──────────────────────────────────
function onPlayerStateChange(event, index) {
  if (
    index === currentSlide.value &&
    event.data === window.YT.PlayerState.ENDED
  ) {
    nextSlideHandler()
  }
}

// ── 輪播控制 ──────────────────────────────────────────────────
function goToSlide(index) {
  if (index === currentSlide.value) return

  slideDir.value  = index > currentSlide.value ? 1 : -1
  const leaving   = currentSlide.value
  prevSlide.value = leaving

  // 停止離開的 slide
  const leavingPlayer = ytPlayers.value[leaving]
  if (leavingPlayer && playerReady.value[leaving]) {
    leavingPlayer.stopVideo()
  }

  currentSlide.value = index

  // 更新 dot ring 時長
  slideDuration.value = slideDurations.value[index]

  // 播放新的 slide
  const enteringPlayer = ytPlayers.value[index]
  if (enteringPlayer && playerReady.value[index]) {
    enteringPlayer.seekTo(0, true)
    enteringPlayer.playVideo()
  }
  // 若尚未就緒，onPlayerReady 就緒時會自動播放
}

function nextSlideHandler() {
  goToSlide((currentSlide.value + 1) % videoSlides.length)
}
function prevSlideHandler() {
  goToSlide((currentSlide.value - 1 + videoSlides.length) % videoSlides.length)
}

// ── 其他資料 ──────────────────────────────────────────────────
const techTags = [
  { name: '優築網',               logo: new URL('../assets/images/experiences/unju.png', import.meta.url).href },
  { name: '無限創意科技',         logo: new URL('../assets/images/experiences/infiniteCreativity.png', import.meta.url).href },
  { name: '昕力資訊',             logo: new URL('../assets/images/experiences/thinkPower.png', import.meta.url).href },
  { name: '盛大資訊',             logo: new URL('../assets/images/experiences/shanda.png', import.meta.url).href },
  { name: '中冠資訊',             logo: new URL('../assets/images/experiences/infChamp.png', import.meta.url).href },
  { name: '昊聲訊息技術有限公司', logo: new URL('../assets/images/experiences/foreverWin.png', import.meta.url).href },
  { name: '巨力搬家大師',         logo: new URL('../assets/images/experiences/giantPower.png', import.meta.url).href },
]

const services = [
  { title: '資訊科技服務',      desc: 'Web / APP 開發 | 客製化資訊系統 | 企業平台整合',        tags: ['Consulting', 'ERP', 'Cloud'],   color: 'linear-gradient(135deg,#0EA5E9 0%,#6366F1 100%)', image: new URL('../assets/images/service1.jpg', import.meta.url).href },
  { title: '企業導入 AI 模型',  desc: 'AI x 企業數位轉型 | 機器學習、深度學習模型開發',        tags: ['ML', 'Deep Learning', 'NLP'],   color: 'linear-gradient(135deg,#6366F1 0%,#A855F7 100%)', image: new URL('../assets/images/service2.jpg', import.meta.url).href },
  { title: '互動科技與 AR / VR', desc: 'Unity 互動介面 | AR 擴增實境應用 | VR 沉浸式內容',   tags: ['AR', 'VR', 'Unity'],            color: 'linear-gradient(135deg,#0891B2 0%,#0EA5E9 100%)', image: new URL('../assets/images/service3.jpg', import.meta.url).href },
  { title: '影音與商業設計',    desc: '品牌識別（Logo / VI）| 平面設計、行銷視覺',             tags: ['Branding', 'Motion', 'Design'], color: 'linear-gradient(135deg,#059669 0%,#10B981 100%)', image: new URL('../assets/images/service4.jpg', import.meta.url).href },
]

let animId = null
function initCanvas() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  let W = canvas.width  = canvas.offsetWidth
  let H = canvas.height = canvas.offsetHeight
  const pts = Array.from({ length: 60 }, () => ({
    x: Math.random() * W, y: Math.random() * H,
    vx: (Math.random() - 0.5) * 0.4, vy: (Math.random() - 0.5) * 0.4,
    r: Math.random() * 1.5 + 0.5,
  }))
  function draw() {
    ctx.clearRect(0, 0, W, H)
    pts.forEach(p => {
      p.x += p.vx; p.y += p.vy
      if (p.x < 0 || p.x > W) p.vx *= -1
      if (p.y < 0 || p.y > H) p.vy *= -1
      ctx.beginPath(); ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
      ctx.fillStyle = 'rgba(14,165,233,0.5)'; ctx.fill()
    })
    pts.forEach((a, i) => pts.slice(i + 1).forEach(b => {
      const d = Math.hypot(a.x - b.x, a.y - b.y)
      if (d < 120) {
        ctx.beginPath(); ctx.moveTo(a.x, a.y); ctx.lineTo(b.x, b.y)
        ctx.strokeStyle = `rgba(14,165,233,${0.12 * (1 - d / 120)})`; ctx.stroke()
      }
    }))
    animId = requestAnimationFrame(draw)
  }
  draw()
  window.addEventListener('resize', () => {
    W = canvas.width  = canvas.offsetWidth
    H = canvas.height = canvas.offsetHeight
  })
}

function getCardBgStyle(svc) {
  if (svc.image) return { backgroundImage: `url(${svc.image})`, backgroundSize: 'cover', backgroundPosition: 'center' }
  return { background: svc.color }
}

// ── Hero hover ────────────────────────────────────────────────
let leaveTimeout = null

function onPolyEnter() {
  clearTimeout(leaveTimeout)
  isHovering.value = true
  if (videoRef.value) videoRef.value.play().catch(() => {})
}

function onPolyLeave() {
  leaveTimeout = setTimeout(() => {
    isHovering.value = false
    if (videoRef.value) videoRef.value.pause()
  }, 150)
}

// ── Scroll ────────────────────────────────────────────────────
let observer = null

const onScroll = () => {
  isScrolled.value = window.scrollY > 50
  if (maskStageRef.value) {
    const rect   = maskStageRef.value.getBoundingClientRect()
    const offset = -rect.top
    if (posterRef.value) posterRef.value.style.transform = `translateY(${offset}px)`
    if (videoRef.value)  videoRef.value.style.transform  = `translateY(${offset}px)`
  }
}

// ── 生命週期 ──────────────────────────────────────────────────
onMounted(async () => {
  setTimeout(() => { heroVisible.value = true }, 100)
  initCanvas()
  window.addEventListener('scroll', onScroll)

  observer = new IntersectionObserver(
    ([entry]) => { heroVisible.value = entry.isIntersecting },
    { threshold: 0.15 }
  )
  if (heroRef.value) observer.observe(heroRef.value)

  // 等 Vue DOM 渲染完，再初始化 YT Players
  await nextTick()
  await loadYouTubeAPI()
  initYTPlayers()
})

onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
  if (animId) cancelAnimationFrame(animId)
  clearTimer()
  clearTimeout(leaveTimeout)

  // 銷毀所有 YT Player，避免記憶體洩漏
  ytPlayers.value.forEach(player => {
    if (player && typeof player.destroy === 'function') player.destroy()
  })

  if (observer && heroRef.value) observer.unobserve(heroRef.value)
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;600;700&family=Noto+Serif+TC:wght@400;600;700&family=Syne:wght@600;700;800&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --navy:   #060F1E;
  --navy2:  #0B1829;
  --navy3:  #0F2236;
  --blue:   #0EA5E9;
  --blue2:  #38BDF8;
  --indigo: #6366F1;
  --white:  #F8FAFC;
  --gray:   #94A3B8;
  --border: rgba(14,165,233,0.15);
  --font-en: 'Syne', sans-serif;
  --font-tc: 'Noto Sans TC', sans-serif;
}

.cy-home { background: var(--navy); color: var(--white); font-family: var(--font-tc); overflow-x: hidden; }

/* ══ NAVBAR ══════════════════════════════════════════ */
.navbar { position: fixed; top: 0; left: 0; right: 0; z-index: 100; padding: 0.2rem 0; transition: all 0.3s ease; }
.navbar.scrolled { background: rgba(245,245,245,0.85); backdrop-filter: blur(20px); border-bottom: 1px solid var(--border); padding: 0.8rem 0; }
.nav-inner { max-width: 1200px; margin: 0 auto; padding: 0 2rem; display: flex; align-items: center; justify-content: space-between; }
.logo { display: flex; align-items: center; gap: 0.75rem; text-decoration: none; color: inherit; cursor: pointer; }
.logo-img { height: 52px; width: auto; display: flex; align-items: center; }
.logo-img img { height: 100%; width: auto; max-height: 52px; object-fit: contain; }
.logo-cn { font-size: 1.35rem; font-weight: 700; color: var(--white); letter-spacing: 0.08em; white-space: nowrap; line-height: 1.3; }
/* 放大：0.85rem -> 0.9rem */
.logo-en { font-size: 0.9rem; color: var(--blue2); font-family: var(--font-en); letter-spacing: 0.12em; white-space: nowrap; line-height: 1.3; margin-top: 2px; }
.navbar.scrolled .logo-img { height: 40px; max-height: 40px; }
.navbar.scrolled .logo-cn  { font-size: 1.15rem; }
/* 放大：0.8rem -> 0.85rem */
.navbar.scrolled .logo-en  { font-size: 0.85rem; }
.nav-links { display: flex; align-items: center; gap: 2.5rem; list-style: none; }
/* 放大：0.95rem -> 1rem */
.nav-links a { color: var(--gray); text-decoration: none; font-size: 1rem; font-weight: 500; transition: color 0.2s; }
.nav-links a:hover { color: var(--white); }
.nav-cta { background: linear-gradient(135deg,var(--blue),var(--indigo)); color: white !important; padding: 0.55rem 1.4rem; border-radius: 6px; font-weight: 600 !important; transition: opacity 0.2s, transform 0.2s !important; }
.nav-cta:hover { opacity: 0.88; transform: translateY(-1px) !important; }
.hamburger { display: none; flex-direction: column; gap: 5px; background: none; border: none; cursor: pointer; padding: 4px; }
.hamburger span { display: block; width: 24px; height: 2px; background: var(--white); border-radius: 2px; transition: all 0.3s; }

/* ══ VIDEO CAROUSEL ══════════════════════════════════ */
.vc-section { position: relative; width: 100%; height: 120svh; max-height: 900px; min-height: 560px; overflow: hidden; background: #000; }
.vc-track { position: absolute; inset: 0; }
.vc-slide { position: absolute; inset: 0; transform: translateX(100%); opacity: 0; transition: transform 0.85s cubic-bezier(0.77, 0, 0.175, 1), opacity 0.85s cubic-bezier(0.77, 0, 0.175, 1); pointer-events: none; will-change: transform, opacity; }
.vc-slide.dir-backward { transform: translateX(-100%); }
.vc-slide.active { transform: translateX(0); opacity: 1; pointer-events: auto; z-index: 2; }
.vc-slide.prev.dir-forward { transform: translateX(-100%); opacity: 0; z-index: 1; }
.vc-slide.prev.dir-backward { transform: translateX(100%); opacity: 0; z-index: 1; }
.vc-video { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; object-position: center; }
.vc-poster { position: absolute; inset: 0; background-size: cover; background-position: center; }
.vc-overlay { position: absolute; inset: 0; background: linear-gradient(135deg, rgba(6,15,30,0.70) 0%, rgba(6,15,30,0.30) 50%, rgba(6,15,30,0.55) 100%); z-index: 1; }
.vc-content { position: absolute; inset: 0; z-index: 10; display: flex; align-items: center; padding: 80px clamp(2rem,8vw,10rem) 0; max-width: 800px; }
.vc-dots { position: absolute; bottom: 2.5rem; left: 50%; transform: translateX(-50%); z-index: 20; display: flex; align-items: center; gap: 1.2rem; }
.vc-dot { position: relative; width: 36px; height: 36px; background: none; border: none; cursor: pointer; padding: 0; display: flex; align-items: center; justify-content: center; transform: scale(0.7); opacity: 0.45; transition: transform 0.35s cubic-bezier(0.34,1.56,0.64,1), opacity 0.3s; }
.vc-dot.active { transform: scale(1); opacity: 1; }
.vc-dot:hover:not(.active) { transform: scale(0.85); opacity: 0.75; }
.vc-dot-ring { position: absolute; inset: 0; width: 100%; height: 100%; transform: rotate(-90deg); }
.vc-dot-ring-track { stroke: rgba(255,255,255,0.2); }
.vc-dot-ring-progress { stroke: url(#dotGrad); stroke-dasharray: 94.25; stroke-dashoffset: 94.25; stroke-linecap: round; animation: vc-ring-progress linear forwards; }
@keyframes vc-ring-progress { from { stroke-dashoffset: 94.25; } to { stroke-dashoffset: 0; } }
.vc-dot-inner { width: 8px; height: 8px; border-radius: 50%; background: #ffffff; transition: transform 0.3s ease, background 0.3s; position: relative; z-index: 1; }
.vc-dot.active .vc-dot-inner { background: var(--blue2); box-shadow: 0 0 8px rgba(56,189,248,0.8); }
.vc-arrow { position: absolute; top: 50%; transform: translateY(-50%); z-index: 20; width: 48px; height: 48px; border-radius: 50%; border: 1px solid rgba(255,255,255,0.25); background: rgba(255,255,255,0.08); backdrop-filter: blur(8px); display: flex; align-items: center; justify-content: center; cursor: pointer; transition: background 0.25s, border-color 0.25s, transform 0.25s; }
.vc-arrow:hover { background: rgba(14,165,233,0.25); border-color: rgba(14,165,233,0.6); transform: translateY(-50%) scale(1.08); }
.vc-arrow-prev { left: 1.5rem; }
.vc-arrow-next { right: 1.5rem; }
.vc-counter { position: absolute; bottom: 2.75rem; right: 2rem; z-index: 20; display: flex; align-items: baseline; gap: 0.3rem; font-family: var(--font-en); }
.vc-counter-cur { font-size: 1.5rem; font-weight: 800; color: #fff; line-height: 1; }
.vc-counter-sep { font-size: 1rem; color: rgba(255,255,255,0.4); }
.vc-counter-total { font-size: 1rem; color: rgba(255,255,255,0.5); }
.vc-slide-left-enter-active { transition: opacity 0.55s ease, transform 0.55s cubic-bezier(0.22,1,0.36,1); }
.vc-slide-left-leave-active { transition: opacity 0.35s ease, transform 0.35s cubic-bezier(0.55,0,1,0.45); }
.vc-slide-left-enter-from { opacity: 0; transform: translateX(48px); }
.vc-slide-left-leave-to { opacity: 0; transform: translateX(-48px); }
.vc-slide-right-enter-active { transition: opacity 0.55s ease, transform 0.55s cubic-bezier(0.22,1,0.36,1); }
.vc-slide-right-leave-active { transition: opacity 0.35s ease, transform 0.35s cubic-bezier(0.55,0,1,0.45); }
.vc-slide-right-enter-from { opacity: 0; transform: translateX(-48px); }
.vc-slide-right-leave-to { opacity: 0; transform: translateX(48px); }

/* ══ HERO SECTION ════════════════════════════════════ */
.hero {
  --hero-bg:      #FAFBFF;
  --hero-bg2:     #F0F4FB;
  --accent:       #3b3fd8;
  --accent-light: #6366f1;
  --accent-sky:   #0EA5E9;
  --text-primary: #12172B;
  --text-muted:   #4A5568;
  --text-light:   #8896AA;
  position: relative; z-index: 1; min-height: 100vh;
  display: flex; align-items: stretch;
  background: #FAFBFF; overflow: hidden; padding: 0;
}
.hero::before {
  content: ''; position: absolute; inset: 0;
  background-image:
    linear-gradient(rgba(99,102,241,0.035) 1px, transparent 1px),
    linear-gradient(90deg, rgba(99,102,241,0.035) 1px, transparent 1px);
  background-size: 52px 52px; pointer-events: none; z-index: 0;
}
.hero::after {
  content: ''; position: absolute; top: -8%; right: -5%; width: 55%; height: 90%;
  background: radial-gradient(ellipse at top right, rgba(99,102,241,0.07) 0%, rgba(14,165,233,0.04) 35%, transparent 65%);
  pointer-events: none; z-index: 0;
}

/* ── mask-stage ── */
.mask-stage { position: relative; flex: 0 0 48%; min-height: 100vh; overflow: hidden; background: #EEF2FA; }
.mask-stage::before { display: none; }
.mask-stage::after  { display: none; }

.mask-bg {
  position: absolute; inset: 0; background-size: cover;
  background-position: center center; background-attachment: fixed;
  background-repeat: no-repeat; z-index: 1; opacity: 1;
  transition: opacity 0.7s cubic-bezier(0.22, 1, 0.36, 1); pointer-events: none;
}
.mask-bg.hidden { opacity: 0; }

.mask-poster {
  position: absolute; top: -20%; left: 0; width: 100%; height: 140%;
  object-fit: cover; object-position: center 30%; z-index: 1; opacity: 1;
  transition: opacity 0.6s cubic-bezier(0.22, 1, 0.36, 1);
  will-change: transform; pointer-events: none;
}
.mask-poster.hidden { opacity: 0; }

.mask-video {
  position: absolute; top: -20%; left: 0; width: 100%; height: 140%;
  object-fit: cover; object-position: center 30%; z-index: 2; opacity: 0;
  transition: opacity 0.6s cubic-bezier(0.22, 1, 0.36, 1);
  will-change: transform; pointer-events: none;
}
.mask-video.playing { opacity: 1; }
.mask-video-placeholder { display: none; }

.mask-svg {
  position: absolute; inset: 0; width: 100%; height: 100%; z-index: 3; pointer-events: none;
  filter: drop-shadow(0 0 20px rgba(99,102,241,0.14)) drop-shadow(0 0 6px rgba(14,165,233,0.10));
}
.mask-fill { transition: fill 0.4s ease; }
.poly-border {
  stroke: #3b3fd8; stroke-width: 1.2; stroke-dasharray: 10 6; opacity: 0.28;
  animation: dashMove 14s linear infinite; filter: drop-shadow(0 0 2px rgba(99,102,241,0.3));
}
@keyframes dashMove { to { stroke-dashoffset: -80; } }

.mask-hotzone { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 4; cursor: crosshair; }
.poly-hover-area { transition: fill 0.4s ease; }
.mask-hotzone:hover .poly-hover-area { fill: rgba(99,102,241,0.03); }

/* Hover 提示 */
.hover-hint {
  position: absolute; bottom: 16%; left: 42%;
  transform: translateX(-50%) scale(0.85);
  z-index: 5; display: flex; flex-direction: column; align-items: center; gap: 8px;
  opacity: 0; transition: opacity 0.4s ease, transform 0.4s cubic-bezier(0.34,1.56,0.64,1);
  pointer-events: none; color: rgba(59,63,216,0.8);
  font-family: 'Syne', sans-serif;
  font-size: 12px;
  letter-spacing: 0.3em; text-transform: uppercase;
}
.hover-hint.visible { opacity: 1; transform: translateX(-50%) scale(1); }
.hint-dot {
  width: 42px; height: 42px; border-radius: 50%;
  border: 1.5px solid rgba(59,63,216,0.35); background: rgba(99,102,241,0.06);
  backdrop-filter: blur(8px); position: relative; animation: hintPulse 2s ease-in-out infinite;
}
.hint-dot::after {
  content: ''; position: absolute; top: 50%; left: 55%; transform: translate(-50%,-50%);
  border-left: 10px solid rgba(59,63,216,0.8); border-top: 6px solid transparent; border-bottom: 6px solid transparent;
}
@keyframes hintPulse {
  0%,100% { box-shadow: 0 0 0 0 rgba(99,102,241,0.25); transform: scale(1); }
  50%      { box-shadow: 0 0 0 10px rgba(99,102,241,0); transform: scale(1.05); }
}

/* ── hero-content ── */
.hero-content {
  flex: 1;
  display: flex; flex-direction: column; justify-content: center;
  padding: clamp(4rem, 8vh, 6rem) clamp(2rem, 5vw, 4rem) clamp(4rem, 8vh, 6rem) clamp(1.5rem, 3vw, 3rem);
  position: relative; z-index: 2;
  opacity: 0; transform: translateY(28px);
  transition: opacity 0.9s cubic-bezier(0.22,1,0.36,1) 0.2s, transform 0.9s cubic-bezier(0.22,1,0.36,1) 0.2s;
}
.hero-content.visible { opacity: 1; transform: translateY(0); }

/* eyebrow 標籤 — 放大：13px -> 15px */
.hero-eyebrow { display: flex; align-items: center; gap: 12px; margin-bottom: 28px; }
.eyebrow-line { display: block; width: 32px; height: 1.5px; background: linear-gradient(90deg, #3b3fd8, #0EA5E9); border-radius: 2px; flex-shrink: 0; }
.eyebrow-label { font-family: 'Syne', sans-serif; font-size: 18px; font-weight: 700; letter-spacing: 0.20em; text-transform: uppercase; color: #3b3fd8; }

/* badge — 放大：13px -> 15px */
.hero-badge {
  display: inline-block; font-family: 'Syne', sans-serif;
  font-size: 15px;
  letter-spacing: 0.2em; text-transform: uppercase; color: #3b3fd8;
  border: 1px solid rgba(59,63,216,0.35); background: rgba(99,102,241,0.06);
  padding: 6px 16px; border-radius: 20px; margin-bottom: 28px;
}

/* 主標題 */
.hero-title {
  font-family: 'Noto Serif TC', 'Georgia', serif;
  font-size: clamp(2rem, 3.5vw, 3rem);
  font-weight: 700; color: #12172B; line-height: 1.2; margin: 0 0 16px; letter-spacing: 0.05em;
}
.title-dot { color: #3b3fd8; font-style: normal; }

/* 副主標 — 放大：clamp(1.05rem, 1.8vw, 1.35rem) -> clamp(1.1rem, 1.9vw, 1.4rem) */
.hero-title-sub {
  display: block; font-family: 'Noto Sans TC', sans-serif;
  font-size: clamp(1.1rem, 1.9vw, 1.4rem);
  font-weight: 400; color: #3b3fd8; letter-spacing: 0.03em; line-height: 1.6; margin-top: 10px;
  background: linear-gradient(130deg, #3b3fd8 0%, #0EA5E9 80%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}

/* 副標文案 — 放大：clamp(0.95rem, 1.4vw, 1.05rem) -> clamp(1rem, 1.55vw, 1.15rem) */
.hero-subtitle {
  font-family: 'Noto Sans TC', sans-serif;
  font-size: clamp(1rem, 1.55vw, 1.15rem);
  font-weight: 400; color: #4A5568; line-height: 2; margin: 20px 0 28px; letter-spacing: 0.02em;
}

/* Pills — 放大：13px -> 15px */
.hero-pills { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 36px; }
.pill {
  display: inline-flex; align-items: center; padding: 7px 18px; border-radius: 99px;
  font-family: 'Noto Sans TC', sans-serif;
  font-size: 18px;
  font-weight: 500; letter-spacing: 0.06em;
  color: #3b3fd8; background: rgba(59,63,216,0.06); border: 1px solid rgba(59,63,216,0.18);
  transition: background 0.2s, border-color 0.2s;
}
.pill:hover { background: rgba(59,63,216,0.10); border-color: rgba(59,63,216,0.35); }

/* 按鈕區 */
.hero-actions { display: flex; align-items: center; gap: 14px; flex-wrap: wrap; margin-bottom: 48px; }
.btn-primary {
  display: inline-flex; align-items: center; gap: 0.5rem;
  background: linear-gradient(135deg, #3b3fd8, #0EA5E9); color: white;
  padding: 0.8rem 2rem; border-radius: 8px;
  font-family: 'Noto Sans TC', sans-serif; font-weight: 600; font-size: 1rem;
  letter-spacing: 0.04em; text-decoration: none;
  transition: opacity 0.2s, transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 6px 28px rgba(59,63,216,0.22);
}
.btn-primary:hover { opacity: 0.88; transform: translateY(-2px); box-shadow: 0 10px 36px rgba(59,63,216,0.32); }
.btn-ghost {
  display: inline-flex; align-items: center; gap: 0.65rem; color: #12172B;
  padding: 0.8rem 1.6rem; border-radius: 8px;
  font-family: 'Noto Sans TC', sans-serif; font-weight: 500; font-size: 1rem;
  letter-spacing: 0.04em; text-decoration: none;
  border: 1px solid rgba(18,23,43,0.14);
  transition: border-color 0.2s, background 0.2s, color 0.2s;
}
.btn-ghost:hover { border-color: rgba(59,63,216,0.4); background: rgba(59,63,216,0.04); color: #3b3fd8; }
.play-icon {
  display: flex; align-items: center; justify-content: center;
  width: 28px; height: 28px; background: rgba(59,63,216,0.08);
  border-radius: 50%; border: 1px solid rgba(59,63,216,0.18); flex-shrink: 0;
}

/* Stats — stat-label 放大：0.8rem -> 0.9rem */
.hero-stats { display: flex; gap: 0; flex-wrap: wrap; padding-top: 28px; border-top: 1px solid rgba(18,23,43,0.08); }
.stat { display: flex; flex-direction: column; gap: 6px; padding-right: 36px; margin-right: 36px; border-right: 1px solid rgba(18,23,43,0.08); }
.stat:last-child { border-right: none; margin-right: 0; padding-right: 0; }
.stat-num {
  font-family: 'Syne', sans-serif; font-size: 1.9rem; font-weight: 800; line-height: 1;
  background: linear-gradient(135deg, #3b3fd8, #0EA5E9);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.stat-label { font-family: 'Noto Sans TC', sans-serif; font-size: 0.9rem; color: #8896AA; letter-spacing: 0.06em; white-space: nowrap; }

/* ══ SECTIONS ═══════════════════════════════════════ */
.section { padding: 6rem 2rem; }
.section-inner { max-width: 1200px; margin: 0 auto; }
.section-header { text-align: center; margin-bottom: 4rem; }
/* section-badge 放大：0.82rem -> 0.9rem */
.section-badge { display: inline-block; color: var(--blue); font-family: var(--font-en); font-size: 0.9rem; letter-spacing: 0.2em; font-weight: 600; margin-bottom: 0.75rem; text-transform: uppercase; }
.section-title { font-family: var(--font-tc); font-size: clamp(1.8rem,3vw,2.6rem); font-weight: 700; margin-bottom: 1rem; text-align: center; }
/* section-desc 放大：1.05rem -> 1.1rem；max-width 放寬讓中文段落更舒服 */
.section-desc { color: var(--gray); font-size: 1.4rem; max-width: 660px; margin: 0 auto; line-height: 1.9; }

/* ══ MARQUEE ════════════════════════════════════════ */
.marquee-bar { background: var(--navy3); border-top: 1px solid var(--border); border-bottom: 1px solid var(--border); padding: 1.5rem 0; overflow: hidden; width: 100%; }
.marquee-track { display: inline-flex; align-items: center; white-space: nowrap; animation: marquee-logo 28s linear infinite; will-change: transform; }
@keyframes marquee-logo { 0% { transform: translateX(0); } 100% { transform: translateX(-33.3333%); } }
.marquee-copy { display: inline-flex; align-items: center; flex-shrink: 0; }
.marquee-item { display: inline-flex; align-items: center; padding: 0; flex-shrink: 0; }
.marquee-bar:hover .marquee-track { animation-play-state: paused; }
.marquee-logo-wrap { width: 160px; height: 52px; padding: 0 2.4rem; flex-shrink: 0; box-sizing: content-box; }
.marquee-logo-img { width: 100%; height: 100%; object-fit: contain; object-position: center; opacity: 0.70; filter: grayscale(25%) brightness(1.15); display: block; transition: opacity 0.3s, filter 0.3s, transform 0.3s; }
.marquee-item:hover .marquee-logo-img { opacity: 1; filter: none; transform: scale(1.06); }

/* ══ SERVICES ═══════════════════════════════════════ */
.services-section { background: var(--navy2); }
.services-grid { display: grid; grid-template-columns: repeat(auto-fill,minmax(280px,1fr)); gap: 1.5rem; margin-top: 3rem; }
.service-card { position: relative; overflow: hidden; border-radius: 16px; min-height: 360px; cursor: pointer; animation: fadeSlideUp 0.6s ease both; animation-delay: var(--delay,0s); isolation: isolate; }
.card-bg { position: absolute; inset: 0; background-size: cover; background-position: center; transform: scale(1); transform-origin: bottom right; transition: transform 0.65s cubic-bezier(0.25,0.46,0.45,0.94); will-change: transform; z-index: 0; }
.service-card:hover .card-bg { transform: scale(1.12); }
.card-overlay { position: absolute; inset: 0; z-index: 1; background: linear-gradient(to bottom, rgba(0,0,0,0.18) 0%, rgba(0,0,0,0.45) 50%, rgba(0,0,0,0.75) 100%); transition: background 0.5s; }
.service-card:hover .card-overlay { background: linear-gradient(to bottom, rgba(0,0,0,0.30) 0%, rgba(0,0,0,0.60) 50%, rgba(0,0,0,0.88) 100%); }
.card-header-content { position: absolute; top: 0; left: 0; right: 0; padding: 1.75rem 1.75rem 0; z-index: 2; transition: transform 0.45s cubic-bezier(0.25,0.46,0.45,0.94), opacity 0.35s; }
.service-card:hover .card-header-content { transform: translateY(-8px); opacity: 0; }
/* card-title 放大：1.35rem -> 1.45rem */
.card-title { font-size: 1.45rem; font-weight: 700; color: #fff; line-height: 1.4; text-shadow: 0 2px 12px rgba(0,0,0,0.4); }
.card-hover-content { position: absolute; bottom: 0; left: 0; right: 0; padding: 1.75rem; z-index: 2; transform: translateY(32px); opacity: 0; transition: transform 0.50s cubic-bezier(0.25,0.46,0.45,0.94) 0.05s, opacity 0.40s ease 0.05s; }
.service-card:hover .card-hover-content { transform: translateY(0); opacity: 1; }
/* card-hover-title 放大：1.25rem -> 1.35rem */
.card-hover-title { font-size: 1.35rem; font-weight: 700; color: #fff; margin: 0 0 0.75rem; text-shadow: 0 1px 8px rgba(0,0,0,0.4); }
/* card-desc 放大：0.925rem -> 1rem */
.card-desc { font-size: 1rem; color: rgba(255,255,255,0.88); line-height: 1.75; margin: 0 0 1rem; }
.card-tags { display: flex; flex-wrap: wrap; gap: 0.4rem; margin-bottom: 1.25rem; }
/* tag 放大：0.8rem -> 0.875rem */
.tag { display: inline-block; padding: 0.3rem 0.75rem; border-radius: 99px; border: 1px solid rgba(255,255,255,0.40); font-size: 0.875rem; color: rgba(255,255,255,0.85); background: rgba(255,255,255,0.10); backdrop-filter: blur(4px); }
/* card-cta 放大：0.88rem -> 0.95rem */
.card-cta { display: inline-flex; align-items: center; gap: 0.4rem; font-size: 0.95rem; font-weight: 600; color: #fff; border-bottom: 1px solid rgba(255,255,255,0.50); padding-bottom: 2px; transition: border-color 0.2s, gap 0.2s; }
.card-cta:hover { border-color: #fff; gap: 0.65rem; }
.cta-text { line-height: 1; }
@keyframes fadeSlideUp { from { opacity: 0; transform: translateY(28px); } to { opacity: 1; transform: translateY(0); } }

/* ══ SOLUTIONS ══════════════════════════════════════ */
.solutions-section { background: var(--navy); }
.tab-nav { display: flex; gap: 0.5rem; margin-bottom: 3rem; flex-wrap: wrap; justify-content: center; }
.tab-btn { padding: 0.65rem 1.6rem; border-radius: 8px; border: 1px solid var(--border); background: transparent; color: var(--gray); font-size: 1rem; font-family: var(--font-tc); cursor: pointer; transition: all 0.25s; }
.tab-btn.active, .tab-btn:hover { background: linear-gradient(135deg,var(--blue),var(--indigo)); color: white; border-color: transparent; }
.tab-content { display: grid; grid-template-columns: 1fr 1fr; gap: 4rem; align-items: center; }
.tab-text h3 { font-size: 1.8rem; font-weight: 700; margin-bottom: 1rem; }
.tab-text p { color: var(--gray); line-height: 1.8; margin-bottom: 1.5rem; }
.tab-text ul { list-style: none; display: flex; flex-direction: column; gap: 0.75rem; }
.tab-text li { display: flex; align-items: center; gap: 0.75rem; font-size: 1rem; color: var(--gray); }
.tab-text li svg { flex-shrink: 0; }
.tab-visual { display: flex; justify-content: center; }
.visual-card { background: rgba(255,255,255,0.04); border: 1px solid var(--border); border-radius: 20px; padding: 2.5rem; width: 100%; max-width: 340px; text-align: center; }
.visual-icon { width: 96px; height: 96px; border-radius: 24px; margin: 0 auto 2rem; display: flex; align-items: center; justify-content: center; }
.visual-metrics { display: flex; gap: 2rem; justify-content: center; }
.metric { display: flex; flex-direction: column; align-items: center; gap: 0.3rem; }
.metric-val { font-family: var(--font-en); font-size: 2rem; font-weight: 800; color: var(--blue2); }
/* metric-lbl 放大：0.85rem -> 0.9rem */
.metric-lbl { font-size: 0.9rem; color: var(--gray); }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s, transform 0.3s; }
.fade-enter-from { opacity: 0; transform: translateY(10px); }
.fade-leave-to { opacity: 0; transform: translateY(-10px); }

/* ══ ABOUT ══════════════════════════════════════════ */
.about-section { background: var(--navy2); }
.about-inner { display: grid; grid-template-columns: 1fr 1fr; gap: 5rem; align-items: center; }
/* about-desc 放大：1rem -> 1.05rem */
.about-desc { color: var(--gray); line-height: 1.9; margin-bottom: 1.2rem; font-size: 1.05rem; }
.about-features { display: grid; grid-template-columns: 1fr 1fr; gap: 0.75rem; margin-top: 1.5rem; }
/* feature 放大：0.9rem -> 0.95rem */
.feature { display: flex; align-items: center; gap: 0.6rem; font-size: 0.95rem; color: var(--gray); }
.about-visual { position: relative; }
.about-card { background: rgba(255,255,255,0.04); border: 1px solid var(--border); border-radius: 16px; }
.main-card { padding: 1.5rem; }
.card-top { display: flex; gap: 6px; margin-bottom: 1.2rem; }
.dot { width: 12px; height: 12px; border-radius: 50%; }
.dot.green { background: #22C55E; }
.dot.yellow { background: #F59E0B; }
.dot.red { background: #EF4444; }
/* code-block 放大：0.88rem -> 0.92rem */
.code-block { font-family: 'Courier New', monospace; font-size: 0.92rem; line-height: 1.9; }
.code-line { white-space: nowrap; }
.code-line.indent { padding-left: 1.5rem; }
.kw { color: #7DD3FC; } .fn { color: #A5F3FC; } .str { color: #FDE68A; }
.val { color: #86EFAC; } .num { color: #FDA4AF; } .comment { color: #64748B; }
.blink-cursor::after { content: '_'; color: var(--blue); animation: blink 1s step-end infinite; }
@keyframes blink { 0%,100% { opacity: 1; } 50% { opacity: 0; } }
.float-card { position: absolute; bottom: -20px; right: -20px; padding: 1rem 1.5rem; display: flex; align-items: center; gap: 1rem; background: var(--navy); border-radius: 14px; }
.float-icon { width: 44px; height: 44px; border-radius: 10px; background: rgba(14,165,233,0.1); display: flex; align-items: center; justify-content: center; }
.float-val { font-family: var(--font-en); font-size: 1.4rem; font-weight: 800; color: var(--white); }
/* float-lbl 放大：0.82rem -> 0.88rem */
.float-lbl { font-size: 0.88rem; color: var(--gray); }

/* ══ CTA ════════════════════════════════════════════ */
.cta-section { background: var(--navy); }
.cta-inner { max-width: 700px; margin: 0 auto; text-align: center; position: relative; }
.cta-glow { position: absolute; top: 50%; left: 50%; transform: translate(-50%,-50%); width: 500px; height: 300px; background: radial-gradient(circle,rgba(14,165,233,0.12) 0%,transparent 70%); pointer-events: none; }
.cta-title { font-size: clamp(1.8rem,3vw,2.4rem); font-weight: 700; margin-bottom: 1rem; }
.cta-desc { color: var(--gray); font-size: 1.1rem; margin-bottom: 2.5rem; line-height: 1.9; }
.cta-actions { display: flex; gap: 1rem; justify-content: center; flex-wrap: wrap; }
.btn-primary { display: inline-flex; align-items: center; gap: 0.5rem; background: linear-gradient(135deg,var(--blue),var(--indigo)); color: white; padding: 0.85rem 2rem; border-radius: 8px; font-weight: 600; font-size: 1rem; text-decoration: none; transition: opacity 0.2s, transform 0.2s, box-shadow 0.2s; box-shadow: 0 4px 24px rgba(14,165,233,0.3); }
.btn-primary:hover { opacity: 0.88; transform: translateY(-2px); box-shadow: 0 8px 32px rgba(14,165,233,0.4); }
.btn-primary.large { padding: 1rem 2.4rem; font-size: 1.05rem; }
.btn-ghost { display: inline-flex; align-items: center; gap: 0.6rem; color: var(--white); padding: 0.85rem 1.8rem; border-radius: 8px; font-weight: 500; font-size: 1rem; text-decoration: none; border: 1px solid rgba(255,255,255,0.15); transition: border-color 0.2s, background 0.2s; }
.btn-ghost:hover { border-color: rgba(14,165,233,0.4); background: rgba(14,165,233,0.06); }
.btn-ghost.large { padding: 1rem 2.2rem; font-size: 1.05rem; }

/* ══ FOOTER ════════════════════════════════════════ */
.footer {  background: #2E3340;  border-top: 1px solid rgba(255, 255, 255, 0.08);}
/* ── 主橫條容器 ── */
.footer-bar {  max-width: 1300px;  margin: 0 auto;  padding: 2rem 2.5rem;  display: flex;  align-items: flex-end;      /* 底部對齊 → Email 與版權齊平 */  justify-content: space-between;  gap: 2rem;  flex-wrap: nowrap; }
/* ══ 左側：Logo + 聯絡資訊 ══════════════════════════════════════ */
.footer-left {  display: flex;  flex-direction: column;  gap: 0.65rem; }
/* Logo 區 */
.footer-logo {  display: flex;  align-items: center;  gap: 0.75rem;}
.footer-logo-img {  height: 60px;  width: auto;  display: flex;  align-items: center;  flex-shrink: 0;}
.footer-logo-img img {  height: 100%;  width: auto;  object-fit: contain;  filter: brightness(0) invert(1);}
.footer-logo-text {  display: flex;  flex-direction: column;  gap: 1px;}
.footer-logo-cn {  display: block;  font-size: 1.5rem;  font-weight: 700;  color: #FFFFFF;  letter-spacing: 0.1em;  line-height: 1.3;  font-family: var(--font-tc);}
.footer-logo-en {  display: block;  font-size: 1.2rem;  color: #A0AEC0;  font-family: var(--font-en);  letter-spacing: 0.12em;  line-height: 1.3;  text-transform: uppercase;}
/* 聯絡資訊橫排 */.footer-contact-row {  display: flex;  align-items: center;  flex-wrap: wrap;  gap: 0.4rem 0.75rem;}
.footer-info-item {  display: inline-flex;  align-items: center;  gap: 0.4rem;  color: #A0AEC0;  font-size: 1.25rem;  font-family: var(--font-tc);  letter-spacing: 0.02em;  line-height: 1;}
.footer-info-item svg {  flex-shrink: 0;  color: #718096;}
.footer-info-sep {  color: #4A5568;  font-size: 0.8rem;  line-height: 1;}
/* ══ 右側：社群 + 連結 + 版權 ══════════════════════════════════ */.footer-right {  display: flex;  flex-direction: column;  align-items: flex-end;  gap: 0.65rem;}
/* 社群按鈕 */.footer-social {  display: flex;  gap: 0.5rem;}
.social-btn {  width: 34px;  height: 34px;  border-radius: 6px;  border: 1px solid rgba(255, 255, 255, 0.2);  background: transparent;  color: #A0AEC0;  display: flex;  align-items: center;  justify-content: center;  text-decoration: none;}
/* 導覽連結 */.footer-nav-links {  display: flex;  align-items: center;  gap: 0.5rem;}
.footer-nav-links a {  color: #A0AEC0;  text-decoration: none;  font-size: 0.875rem;  font-family: var(--font-tc);  letter-spacing: 0.04em;}
.footer-nav-sep {  color: #4A5568;  font-size: 0.75rem;}
/* 版權文字 */
.footer-copy {  color: #718096;  font-size: 1.1rem;  font-family: var(--font-tc);  letter-spacing: 0.04em;  line-height: 1.5;  text-align: right;}

/* ── YouTube 容器 ── */
.vc-yt-container {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #000;
  pointer-events: none;
}

/* iframe 滿版覆蓋（以 16:9 比例補償任意視窗尺寸） */
.vc-yt-container :deep(iframe) {
  position: absolute !important;
  top: 50% !important;
  left: 50% !important;
  /* 以較大的那個維度為基準，確保不留黑邊 */
  width: max(100%, 177.78vh) !important;   /* 177.78vh = 100vh × 16/9 */
  height: max(100%, 56.25vw) !important;   /* 56.25vw  = 100vw × 9/16 */
  transform: translate(-50%, -50%) !important;
  pointer-events: none !important;
  border: none !important;
}

/* ══ RESPONSIVE ════════════════════════════════════ */
@media (max-width: 1024px) {  
.footer-inner {    grid-template-columns: 1fr;    gap: 3.5rem;  }  
.footer-cols {    grid-template-columns: repeat(2, 1fr);  }
}

@media (max-width: 900px) {
  .hero-visual { display: none; }
  .tab-content { grid-template-columns: 1fr; gap: 2rem; }
  .about-inner { grid-template-columns: 1fr; gap: 3rem; }
  .footer-inner { grid-template-columns: 1fr; gap: 2.5rem; }
  .float-card { right: 0; }
  .hero { flex-direction: column; min-height: auto; }
  .mask-stage { flex: none; width: 100%; height: 55vw; min-height: 300px; }
  .mask-bg { background-attachment: scroll; background-size: cover; background-position: center; }
  .hero-content { padding: 3rem 2rem 3.5rem; }
  .br-desktop { display: none; }
  .hero-title { font-size: 2rem; }
  .hero-title-sub { font-size: 1.1rem; }
  .footer-bar {    flex-direction: column;    align-items: flex-start;    padding: 2rem 1.5rem;    gap: 1.5rem;  }  
  .footer-right {    align-items: flex-start;    width: 100%;    padding-top: 1rem;    border-top: 1px solid rgba(255, 255, 255, 0.08);  }  
  .footer-copy {    text-align: left;  }
}
@media (max-width: 768px) {
  .nav-links { display: none; position: fixed; inset: 0; top: 70px; background: var(--navy2); flex-direction: column; align-items: center; justify-content: center; gap: 2rem; }
  .nav-links.open { display: flex; }
  .hamburger { display: flex; }
  .services-grid { grid-template-columns: 1fr 1fr; gap: 1rem; }
  .service-card { min-height: 280px; }
  .footer-cols { grid-template-columns: 1fr 1fr; }
  .about-features { grid-template-columns: 1fr; }
  .hero-stats { gap: 1.5rem; flex-wrap: wrap; }
  .vc-dots { gap: 0.8rem; bottom: 1.5rem; }
  .vc-dot { width: 28px; height: 28px; }
  .footer-cols { grid-template-columns: 1fr 1fr; }
}
@media (max-width: 640px) {  
.footer {    padding: 4rem 1.5rem 0;  }  
.footer-cols {    grid-template-columns: 1fr;    gap: 2.5rem;  }  
.footer-bottom {    flex-direction: column;    align-items: flex-start;    gap: 0.75rem;  } 
.footer-bottom-links {    gap: 2rem;  }
}
@media (max-width: 560px) {
  .mask-stage { height: 65vw; min-height: 240px; }
  .hero-actions { flex-direction: column; align-items: flex-start; }
  .hero-stats { gap: 1rem; }
  .stat { border-right: none; padding-right: 0; margin-right: 0; }
  .hero-pills { gap: 6px; }
  .hero-title { font-size: 1.75rem; }
  .hero-subtitle { font-size: 1rem; }

    .footer-contact-row {    flex-direction: column;    align-items: flex-start;    gap: 0.4rem;  }  
    .footer-info-sep {    display: none;  }  
    .footer-nav-links {    flex-wrap: wrap;  }
}
@media (max-width: 480px) {
  .footer-cols {    grid-template-columns: 1fr;  }  
  .footer-bottom {    flex-direction: column;    align-items: center;    text-align: center;  }  
  .footer-bottom-links {    justify-content: center;  }
}
</style>
