<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from 'vue'

const navItems = [
  { label: '소개', href: '#about' },
  { label: '포트폴리오', href: '#portfolio' },
  { label: '영상', href: '#media' },
  { label: '문의', href: '#contact' },
]

const servicePoints = [
  {
    title: '벽면 임대 매칭',
    description: '상권, 유동 인구, 노출 각도를 기준으로 건물 외벽과 브랜드를 연결합니다.',
  },
  {
    title: '아트/광고 통합 기획',
    description: '벽화, 래핑, 미디어파사드까지 현장 성격에 맞는 크리에이티브를 제안합니다.',
  },
  {
    title: '현장 실행 원스톱',
    description: '디자인 시안, 시공, 운영, 유지보수까지 한 팀에서 끝까지 책임집니다.',
  },
]

const stats = [
  { value: '120+', label: '운영 벽면 수' },
  { value: '340K', label: '월 평균 노출량' },
  { value: '98%', label: '프로젝트 완료율' },
  { value: '24/7', label: '현장 모니터링' },
]

const journeyEntries = [
  {
    year: '2020',
    items: ['청년창업사관학교 10기', 'WallD 서비스 런칭', '서울특별시 로컬크리에이터 선정', 'WALLD X CORP. 서비스 런칭'],
  },
  {
    year: '2021',
    items: ['WALLD X -KPOP STAR 서비스 런칭', '한국관광벤처기업 선정', '교보 임팩트업 최종 선정', 'MG희망나눔재단 소셜성장지원사업 3기 우수상'],
  },
  {
    year: '2022',
    items: ['디자인 스타트업 데모데이 2위', 'D.CAMP D.DAY 우승', 'WALLD 미술건축 서비스 런칭', 'SEED 7억원 투자유치'],
  },
  { year: '2023', items: ['창업도약패키지 선정'] },
  { year: '2024', items: ['GSA 파트너십'] },
  { year: '2025', items: ['예술경영지원센터 창업도약 패키지 선정', 'WALLD 벽 시리즈 1권 공급'] },
]

const mediaVideoModules = import.meta.glob('./assets/image/*.{mp4,webm,mov,m4v}', {
  eager: true,
  import: 'default',
})

const mediaImageModules = import.meta.glob('./assets/image/*.{jpg,jpeg,png,webp,avif}', {
  eager: true,
  import: 'default',
})

const getFileName = (path) => {
  return path.split('/').pop() ?? path
}

const mediaDisplayTitles = [
  '메가그래픽',
  '메가 인스톨',
  '미디어 아트',
  '스트릿 아트',
  '시트 래핑',
]

const mediaPosterColors = [
  ['#0f1219', '#2f436e'],
  ['#10141c', '#41566e'],
  ['#10151e', '#334f63'],
  ['#12151d', '#4b3f65'],
  ['#11161f', '#3b5f5b'],
]

const buildMediaPoster = (title, index) => {
  const [startColor, endColor] = mediaPosterColors[index % mediaPosterColors.length]
  const svg = `
<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 1200 1500'>
  <defs>
    <linearGradient id='posterGradient' x1='0' y1='0' x2='1' y2='1'>
      <stop offset='0%' stop-color='${startColor}' />
      <stop offset='100%' stop-color='${endColor}' />
    </linearGradient>
  </defs>
  <rect width='1200' height='1500' fill='url(#posterGradient)' />
  <rect x='52' y='52' width='1096' height='1396' fill='none' stroke='rgba(255,255,255,0.18)' stroke-width='3' />
  <text x='94' y='1406' fill='rgba(255,255,255,0.88)' font-size='62' font-family='Arial, sans-serif' letter-spacing='2'>${title}</text>
</svg>
  `.trim()

  return `data:image/svg+xml,${encodeURIComponent(svg)}`
}

const sortedVideoItems = Object.entries(mediaVideoModules)
  .map(([path, video]) => {
    return {
      id: path,
      video,
      fileName: getFileName(path),
    }
  })
  .sort((a, b) => a.fileName.localeCompare(b.fileName, 'en'))

if (sortedVideoItems.length > mediaDisplayTitles.length) {
  const mediaLastIndex = mediaDisplayTitles.length - 1
  const footerIndex = mediaDisplayTitles.length
  const mediaLastItem = sortedVideoItems[mediaLastIndex]
  sortedVideoItems[mediaLastIndex] = sortedVideoItems[footerIndex]
  sortedVideoItems[footerIndex] = mediaLastItem
}

const allVideoItems = sortedVideoItems.map((item, index) => {
  const title = mediaDisplayTitles[index] ?? `영상 ${String(index + 1).padStart(2, '0')}`

  return {
    ...item,
    title,
    poster: buildMediaPoster(title, index),
    preload: index < 2 ? 'auto' : 'metadata',
  }
})

const mediaItems = allVideoItems.slice(0, mediaDisplayTitles.length)

const imagePool = Object.entries(mediaImageModules)
  .map(([path, image]) => {
    const fileName = getFileName(path)
    return {
      id: path,
      image,
      fileName,
    }
  })
  .sort((a, b) => a.fileName.localeCompare(b.fileName, 'en'))

const categoryPool = [
  'Mega Graphic',
  'Street Art',
  'Media Art',
  'Facade Art',
  'Commercial Wrap',
  'Artist Collaboration',
  'Campaign Art',
  'Seasonal Campaign',
  'Public Art',
  'Campaign Branding',
  'Night Campaign',
  'Commercial Art',
  'Entertainment',
  'Fandom Campaign',
  'Detail Shot',
  'Media Campaign',
  'Commercial Wrap',
  'Art Detail',
]

const featuredNamePool = [
  '도심 시그니처 월',
  '랜드마크 브랜드 캔버스',
  '나이트 라이트 파사드',
]

const featuredDescriptionPool = [
  '브랜드 핵심 메시지를 대형 외벽에 집약해 첫 시선을 잡는 대표 시안',
  '유동 인구가 높은 동선에 맞춰 강한 인지와 체류를 만든 메인 비주얼',
  '야간 조도 환경까지 고려해 입체감과 집중도를 높인 파사드 연출',
]

const portfolioNamePool = [
  '블루 코너 래핑',
  '선셋 타이포 월',
  '시티 리듬 파사드',
  '크림 톤 메가월',
  '네온 포인트 래핑',
  '아티스트 콜라주 월',
  '브랜드 모노그램 빌딩',
  '캠페인 포토 스팟 월',
  '컬러 블록 익스텐션',
  '스트리트 아트 캔버스',
  '하이 콘트라스트 파사드',
  '그래픽 스트라이프 월',
  '웨이브 패턴 미디어월',
  '메인 스트리트 브랜딩',
  '아이덴티티 타워 래핑',
  '다이내믹 캐릭터 월',
  '파노라마 아트 파사드',
  '그라데이션 임팩트 월',
  '화이트 라인 시그니처',
  '딥톤 커머셜 래핑',
  '센터포인트 비주얼 월',
  '시즌 키비주얼 파사드',
  '아트웍 익스프레션 월',
  '컬러 스플래시 브랜딩',
]

const portfolioDescriptionPool = [
  '현장 가시성과 보행 동선을 반영해 브랜드 노출을 극대화한 외벽 시안',
  '건물 비율에 맞춘 그래픽 밸런스로 멀리서도 식별되는 브랜딩 구성',
  '공간 분위기와 상권 성격을 고려해 메시지 전달력을 높인 벽면 연출',
  '색 대비와 타이포 리듬을 조합해 체류 시간을 늘린 시각 집중형 디자인',
]

const featuredProjects = computed(() =>
  imagePool.slice(0, 3).map((item, index) => ({
    title: featuredNamePool[index] ?? `시그니처 월 ${String(index + 1).padStart(2, '0')}`,
    category: 'Featured Project',
    image: item.image,
    description:
      featuredDescriptionPool[index] ??
      featuredDescriptionPool[index % featuredDescriptionPool.length],
  })),
)

const basePortfolioItems = computed(() =>
  imagePool.slice(3).map((item, index) => ({
    title: portfolioNamePool[index] ?? `WallD 아카이브 ${String(index + 1).padStart(2, '0')}`,
    category: categoryPool[index % categoryPool.length],
    image: item.image,
    description: portfolioDescriptionPool[index % portfolioDescriptionPool.length],
  })),
)

const portfolioInitialCount = 9
const portfolioExpandStep = 3
const portfolioVisibleCount = ref(portfolioInitialCount)

const allPortfolioItems = computed(() => {
  if (basePortfolioItems.value.length > 0) {
    return basePortfolioItems.value
  }

  return featuredProjects.value
})

const visiblePortfolioItems = computed(() =>
  allPortfolioItems.value.slice(0, portfolioVisibleCount.value),
)

const hasMorePortfolioItems = computed(
  () => portfolioVisibleCount.value < allPortfolioItems.value.length,
)

const showMorePortfolioItems = () => {
  portfolioVisibleCount.value = Math.min(
    portfolioVisibleCount.value + portfolioExpandStep,
    allPortfolioItems.value.length,
  )
}

const footerVideo = computed(() => {
  if (!allVideoItems.length) {
    return null
  }

  return allVideoItems[mediaDisplayTitles.length] ?? allVideoItems[0]
})

const revealObservers = new WeakMap()
const mediaSectionRef = ref(null)
const mediaViewportRef = ref(null)
const mediaTrackRef = ref(null)
const pageProgress = ref(0)
const mediaProgress = ref(0)
const mediaScrollDistance = ref(0)
const isMobileMediaMode = ref(false)
const videoRefs = ref([])

let scrollTicking = false
let rafId = null
let mediaModeQuery = null
let mediaModeQueryListener = null

const clamp = (value, min = 0, max = 1) => Math.min(max, Math.max(min, value))

function primeVideoFrame(index) {
  const target = videoRefs.value[index]
  if (!target) {
    return
  }

  if (target.dataset.primed === '1') {
    return
  }

  const previewTime = Number.isFinite(target.duration) && target.duration > 0.2 ? 0.2 : 0.01

  try {
    target.currentTime = previewTime
    target.dataset.primed = '1'
  } catch {
    // Ignore browsers that block seeking before full metadata sync.
  }
}

const setVideoRef = (el, index) => {
  if (!el) {
    return
  }

  videoRefs.value[index] = el
}

const playHoveredVideo = async (index) => {
  videoRefs.value.forEach((video, videoIndex) => {
    if (!video || videoIndex === index) {
      return
    }

    video.pause()
  })

  const target = videoRefs.value[index]
  if (!target) {
    return
  }

  try {
    await target.play()
  } catch {
    // Ignore autoplay interruption in strict browser settings.
  }
}

const pauseHoveredVideo = (index) => {
  const target = videoRefs.value[index]
  if (!target) {
    return
  }

  target.pause()
}

const resetVideoToStart = (index) => {
  const target = videoRefs.value[index]
  if (!target) {
    return
  }

  target.pause()

  try {
    target.currentTime = 0
  } catch {
    // Ignore browsers that block seeking before full metadata sync.
  }
}

const handleVideoTouchStart = (index) => {
  playHoveredVideo(index)
}

const handleVideoTouchEnd = (index) => {
  resetVideoToStart(index)
}

const updateMediaMetrics = () => {
  if (isMobileMediaMode.value) {
    mediaScrollDistance.value = 0
    return
  }

  const viewport = mediaViewportRef.value
  const track = mediaTrackRef.value

  if (!viewport || !track) {
    mediaScrollDistance.value = 0
    return
  }

  const distance = track.scrollWidth - viewport.clientWidth
  mediaScrollDistance.value = distance > 0 ? distance : 0
}

const updateMediaProgress = () => {
  if (isMobileMediaMode.value) {
    mediaProgress.value = 0
    return
  }

  const section = mediaSectionRef.value
  if (!section || typeof window === 'undefined') {
    return
  }

  const maxScroll = section.offsetHeight - window.innerHeight
  if (maxScroll <= 0) {
    mediaProgress.value = 0
    return
  }

  const scrolled = -section.getBoundingClientRect().top
  mediaProgress.value = clamp(scrolled / maxScroll)
}

const updatePageProgress = () => {
  if (typeof window === 'undefined') {
    return
  }

  const maxScroll = document.documentElement.scrollHeight - window.innerHeight
  pageProgress.value = maxScroll > 0 ? clamp(window.scrollY / maxScroll) : 0
}

const updateScrollState = () => {
  updateMediaProgress()
  updatePageProgress()
}

const scheduleScrollUpdate = () => {
  if (scrollTicking || typeof window === 'undefined') {
    return
  }

  scrollTicking = true
  rafId = window.requestAnimationFrame(() => {
    updateScrollState()
    scrollTicking = false
  })
}

const handleResize = () => {
  updateMediaMetrics()
  scheduleScrollUpdate()
}

const updateMediaMode = () => {
  if (typeof window === 'undefined') {
    return
  }

  isMobileMediaMode.value = window.matchMedia('(max-width: 760px)').matches
}

const headerStyle = computed(() => {
  const progress = pageProgress.value
  return {
    backgroundColor: `rgba(8, 10, 13, ${(0.42 + progress * 0.42).toFixed(3)})`,
    borderBottomColor: `rgba(255, 255, 255, ${(0.08 + progress * 0.16).toFixed(3)})`,
  }
})

const mediaSectionStyle = computed(() =>
  isMobileMediaMode.value
    ? {}
    : {
        height: `calc(100vh + ${mediaScrollDistance.value.toFixed(2)}px)`,
      },
)

const mediaTrackStyle = computed(() =>
  isMobileMediaMode.value
    ? {}
    : {
        transform: `translate3d(-${(mediaProgress.value * mediaScrollDistance.value).toFixed(2)}px, 0, 0)`,
      },
)

const vReveal = {
  mounted(el, binding) {
    const variant = typeof binding.value === 'string' ? binding.value : 'up'
    el.classList.add('reveal', `reveal_${variant}`)

    if (typeof window === 'undefined') {
      return
    }

    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
      el.classList.add('is_visible')
      return
    }

    const observer = new IntersectionObserver(
      (entries, currentObserver) => {
        entries.forEach((entry) => {
          if (!entry.isIntersecting) {
            return
          }

          entry.target.classList.add('is_visible')
          currentObserver.unobserve(entry.target)
        })
      },
      {
        threshold: 0.15,
        rootMargin: '0px 0px -10% 0px',
      },
    )

    observer.observe(el)
    revealObservers.set(el, observer)
  },
  unmounted(el) {
    const observer = revealObservers.get(el)
    if (!observer) {
      return
    }

    observer.unobserve(el)
    observer.disconnect()
    revealObservers.delete(el)
  },
}

onMounted(() => {
  updateMediaMode()
  updateMediaMetrics()
  updateScrollState()
  window.addEventListener('scroll', scheduleScrollUpdate, { passive: true })
  window.addEventListener('resize', handleResize)

  mediaModeQuery = window.matchMedia('(max-width: 760px)')
  mediaModeQueryListener = (event) => {
    isMobileMediaMode.value = event.matches
    handleResize()
  }

  if (typeof mediaModeQuery.addEventListener === 'function') {
    mediaModeQuery.addEventListener('change', mediaModeQueryListener)
  } else if (typeof mediaModeQuery.addListener === 'function') {
    mediaModeQuery.addListener(mediaModeQueryListener)
  }

  nextTick(() => {
    updateMediaMetrics()
    updateScrollState()
  })
})

onBeforeUnmount(() => {
  if (typeof window !== 'undefined') {
    if (rafId !== null) {
      window.cancelAnimationFrame(rafId)
    }

    window.removeEventListener('scroll', scheduleScrollUpdate)
    window.removeEventListener('resize', handleResize)
  }

  if (mediaModeQuery && mediaModeQueryListener) {
    if (typeof mediaModeQuery.removeEventListener === 'function') {
      mediaModeQuery.removeEventListener('change', mediaModeQueryListener)
    } else if (typeof mediaModeQuery.removeListener === 'function') {
      mediaModeQuery.removeListener(mediaModeQueryListener)
    }
  }

  mediaModeQuery = null
  mediaModeQueryListener = null

  videoRefs.value.forEach((target) => {
    if (!target) {
      return
    }

    target.pause()
  })
})
</script>

<template>
  <div class="page">
    <header class="site_header" :style="headerStyle">
      <div class="container header_inner">
        <a href="#media" class="logo_link" aria-label="walld 홈">
          <span class="logo_text">wallD</span>
        </a>

        <nav class="nav" aria-label="주요 메뉴">
          <a v-for="item in navItems" :key="item.href" :href="item.href">{{ item.label }}</a>
        </nav>
      </div>
    </header>

    <section id="media" class="media_section" ref="mediaSectionRef" :style="mediaSectionStyle">
      <div class="media_sticky">
        <div class="media_viewport" ref="mediaViewportRef">
          <div v-if="mediaItems.length" class="video_track" ref="mediaTrackRef" :style="mediaTrackStyle">
            <div
              class="video_item"
              v-for="(item, index) in mediaItems"
              :key="item.id"
              tabindex="0"
              @mouseenter="playHoveredVideo(index)"
              @mouseleave="pauseHoveredVideo(index)"
              @focusin="playHoveredVideo(index)"
              @focusout="pauseHoveredVideo(index)"
              @touchstart.passive="handleVideoTouchStart(index)"
              @touchend="handleVideoTouchEnd(index)"
              @touchcancel="handleVideoTouchEnd(index)"
            >
              <div class="video_frame">
                <video
                  :ref="(el) => setVideoRef(el, index)"
                  :src="item.video"
                  :poster="item.poster"
                  muted
                  loop
                  playsinline
                  :preload="item.preload"
                  @loadedmetadata="handleResize"
                  @loadeddata="primeVideoFrame(index)"
                ></video>
              </div>
              <p class="video_title">{{ item.title }}</p>
            </div>
          </div>
          <div v-else class="media_empty">`src/assets/image`에 영상 파일을 넣어주세요.</div>
        </div>
      </div>
    </section>

    <main>
      <section id="about" class="section about_section">
        <div class="container">
          <div class="section_head" v-reveal="'up'">
            <p class="section_kicker">ABOUT WALLD</p>
            <h2>브랜드 메시지가 보이는 벽을 만듭니다</h2>
          </div>

          <div class="about_grid">
            <article
              class="about_card"
              v-for="(item, index) in servicePoints"
              :key="item.title"
              v-reveal="'up'"
              :style="{ '--reveal_delay': `${index * 80}ms` }"
            >
              <h3>{{ item.title }}</h3>
              <p>{{ item.description }}</p>
            </article>
          </div>

          <div class="stats_grid">
            <article
              class="stat"
              v-for="(item, index) in stats"
              :key="item.label"
              v-reveal="'zoom'"
              :style="{ '--reveal_delay': `${index * 70}ms` }"
            >
              <strong>{{ item.value }}</strong>
              <span>{{ item.label }}</span>
            </article>
          </div>
        </div>
      </section>

      <section id="journey" class="section journey_section">
        <div class="container journey_layout">
          <div class="journey_intro" v-reveal="'up'">
            <p class="journey_kicker">✦ Our Journey</p>
            <h2>WallD가<br />지나온 길</h2>
          </div>

          <div class="journey_timeline" v-reveal="'up'">
            <p class="journey_range">(©2020-25)</p>

            <article class="journey_row" v-for="entry in journeyEntries" :key="entry.year">
              <strong>{{ entry.year }}</strong>
              <ul>
                <li v-for="item in entry.items" :key="item">{{ item }}</li>
              </ul>
            </article>
          </div>
        </div>
      </section>

      <section id="portfolio" class="section portfolio_section">
        <div class="container">
          <div class="section_head" v-reveal="'up'">
            <p class="section_kicker">PORTFOLIO</p>
            <h2>고정 비율로 구성한 실제 시공 포트폴리오</h2>
          </div>

          <div class="portfolio_featured" v-if="featuredProjects.length">
            <article
              class="portfolio_feature_card"
              v-for="(item, index) in featuredProjects"
              :key="item.title"
              v-reveal="'up'"
              :style="{ '--reveal_delay': `${index * 90}ms` }"
            >
              <div class="media_frame featured_media">
                <img :src="item.image" :alt="item.title" class="media_fluid" loading="lazy" />
              </div>
              <div class="portfolio_copy">
                <p>{{ item.category }}</p>
                <h3>{{ item.title }}</h3>
                <span>{{ item.description }}</span>
              </div>
            </article>
          </div>

          <div class="portfolio_grid">
            <article
              class="portfolio_card"
              v-for="(item, index) in visiblePortfolioItems"
              :key="`${item.title}_${index}`"
              v-reveal="'up'"
              :style="{ '--reveal_delay': `${index * 65}ms` }"
            >
              <div class="media_frame card_media">
                <img :src="item.image" :alt="item.title" class="media_fluid" loading="lazy" />
              </div>
              <div class="portfolio_copy">
                <p>{{ item.category }}</p>
                <h3>{{ item.title }}</h3>
                <span>{{ item.description }}</span>
              </div>
            </article>
          </div>

          <div class="portfolio_more_wrap" v-if="hasMorePortfolioItems">
            <button type="button" class="portfolio_more_button" @click="showMorePortfolioItems">
              더보기
            </button>
          </div>
        </div>
      </section>

      <section id="contact" class="section contact_section">
        <div class="container contact_shell">
          <div class="contact_topline" v-reveal="'up'">
            <span>WallD</span>
            <span>Wall Advertising</span>
            <span>Contact</span>
          </div>

          <div class="contact_hero" v-reveal="'up'">
            <h2>Contact.</h2>
            <p>Transforming spaces with creativity. From concept to perfection, we make it seamless.</p>
          </div>

          <div class="contact_layout">
            <aside class="contact_side" v-reveal="'left'">
              <span class="contact_chip">Contact now</span>
              <h3>벽 임대 및<br />솔루션 요청</h3>
              <p>wallD는 기업의 광고 아이디어를 현실에 구현해 드립니다.</p>
              <div class="contact_divider"></div>
              <ul class="contact_meta">
                <li>서울특별시 성동구 성수일로3길 5</li>
                <li>Monday - Friday / 10am - 6pm</li>
                <li>T: 02-6215-2027</li>
                <li>E: leonardo@wall-d.com</li>
              </ul>
              <small class="contact_legal">© IFBI corp. All Right Reserved</small>
            </aside>

            <div class="contact_card" v-reveal="'up'">
              <h3>Let's Talk</h3>
              <form class="contact_form" @submit.prevent>
                <label>
                  이름 *
                  <input type="text" placeholder="문의자 이름/소속을 입력해주세요" />
                </label>

                <label>
                  회사명 *
                  <input type="text" placeholder="회사명을 입력해주세요" />
                </label>

                <div class="form_grid">
                  <label>
                    Email *
                    <input type="email" placeholder="abcd@walld.com" />
                  </label>

                  <label>
                    연락처 *
                    <input type="tel" placeholder="01012345678" />
                  </label>
                </div>

                <button type="submit">Submit Now</button>
              </form>
            </div>
          </div>
        </div>
      </section>

      <section class="section footer_banner_section">
        <div class="footer_banner" v-reveal="'up'">
          <div class="footer_banner_text">
            <h2>WALLD</h2>
          </div>

          <div class="footer_banner_video" v-if="footerVideo">
            <video :src="footerVideo.video" autoplay muted loop playsinline preload="metadata"></video>
          </div>
        </div>
      </section>
    </main>

    <footer class="site_footer">
      <div class="container footer_inner">
        <p>© 2026 walld. All rights reserved.</p>
        <div>
          <a href="#">개인정보처리방침</a>
          <a href="#">이용약관</a>
          <a href="#">문의하기</a>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.page {
  --bg: #050608;
  --surface: #101217;
  --surface_2: #151a20;
  --line: #2c323a;
  --text: #f3f4f6;
  --muted: #a7b0ba;
  min-height: 100vh;
  background: radial-gradient(circle at 80% 20%, rgba(62, 76, 117, 0.16), transparent 30%), var(--bg);
  color: var(--text);
}

.container {
  width: min(1240px, calc(100% - 2.8rem));
  margin: 0 auto;
}

.site_header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 70;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
}

.header_inner {
  min-height: 74px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.logo_link {
  display: inline-flex;
  align-items: center;
}

.logo_text {
  font-size: 1.48rem;
  font-weight: 800;
  letter-spacing: -0.03em;
  color: #f4f7fb;
}

.nav {
  display: flex;
  gap: 1.35rem;
  font-size: 0.92rem;
  color: var(--muted);
}

.nav a:hover {
  color: var(--text);
}

.media_frame {
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 14px;
  background: #090b0e;
  overflow: hidden;
}

.media_fluid {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.section {
  position: relative;
  padding: 5.4rem 0;
  border-top: 1px solid var(--line);
}

.section_head {
  margin-bottom: 1.2rem;
}

.section_kicker {
  margin: 0;
  font-size: 0.76rem;
  color: #98a3b0;
  letter-spacing: 0.08em;
}

h2 {
  margin: 0.62rem 0 0;
  font-size: clamp(1.6rem, 3vw, 2.6rem);
  line-height: 1.2;
}

.about_grid {
  display: grid;
  gap: 0.8rem;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

.about_card {
  border: 1px solid var(--line);
  border-radius: 12px;
  padding: 1rem;
  background: var(--surface);
}

.about_card h3 {
  margin: 0;
  font-size: 1.03rem;
}

.about_card p {
  margin: 0.55rem 0 0;
  color: var(--muted);
  line-height: 1.68;
  font-size: 0.92rem;
}

.stats_grid {
  margin-top: 0.8rem;
  display: grid;
  gap: 0.8rem;
  grid-template-columns: repeat(4, minmax(0, 1fr));
}

.stat {
  border: 1px solid var(--line);
  border-radius: 12px;
  padding: 0.9rem;
  background: var(--surface_2);
}

.stat strong {
  display: block;
  font-size: 1.55rem;
}

.stat span {
  margin-top: 0.3rem;
  display: block;
  color: var(--muted);
  font-size: 0.84rem;
}

.media_section {
  border-top: 0;
  position: relative;
  background: #000;
}

.media_sticky {
  position: sticky;
  top: 0;
  height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: flex-end;
  padding: 5.8rem 0 1.2rem;
  background: #000;
}

.media_viewport {
  width: 100%;
  overflow: hidden;
  background: #000;
}

.video_track {
  display: flex;
  align-items: end;
  gap: 0.9rem;
  padding: 0 2.8rem;
  will-change: transform;
  background: #000;
}

.video_item {
  flex: 0 0 min(52vw, 760px);
  display: grid;
  gap: 0.45rem;
  outline: none;
  background: #000;
}

.video_item:focus-visible {
  box-shadow: 0 0 0 2px rgba(238, 242, 248, 0.75);
}

.video_frame {
  aspect-ratio: 4 / 5;
  background: #000;
  overflow: hidden;
}

.video_frame video {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: contain;
  background: #000;
  pointer-events: none;
}

.video_title {
  margin: 0;
  color: #d6dde7;
  font-size: 0.82rem;
  letter-spacing: 0.02em;
  text-align: center;
}

.media_empty {
  width: 100%;
  min-height: 220px;
  display: grid;
  place-items: center;
  color: #aeb8c4;
  font-size: 0.95rem;
}

.journey_section {
  background: #000;
}

.journey_layout {
  display: grid;
  grid-template-columns: minmax(220px, 0.72fr) minmax(0, 1.28fr);
  gap: 2rem;
}

.journey_kicker {
  margin: 0;
  color: #d5dce5;
  font-size: 1.1rem;
}

.journey_intro h2 {
  margin: 1rem 0 0;
  font-size: clamp(2.4rem, 5.4vw, 5.5rem);
  line-height: 0.95;
  letter-spacing: -0.03em;
}

.journey_range {
  margin: 0;
  padding-bottom: 0.85rem;
  border-bottom: 1px solid #1e232b;
  color: #788291;
  font-size: 0.88rem;
}

.journey_row {
  display: grid;
  grid-template-columns: 92px 1fr;
  gap: 1rem;
  padding: 1rem 0;
  border-bottom: 1px solid #1e232b;
}

.journey_row strong {
  font-size: 1.8rem;
  line-height: 1;
  letter-spacing: -0.01em;
}

.journey_row ul {
  margin: 0;
  padding-left: 1.1rem;
  display: grid;
  gap: 0.42rem;
  color: #ccd3dd;
  font-size: 1.15rem;
  line-height: 1.45;
}

.portfolio_section {
  background: linear-gradient(180deg, rgba(17, 20, 25, 0.46), rgba(17, 20, 25, 0));
}

.portfolio_note {
  margin: 0.42rem 0 0;
  color: #95a2b1;
  font-size: 0.87rem;
}

.portfolio_featured {
  display: grid;
  gap: 0.95rem;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

.portfolio_feature_card,
.portfolio_card {
  border: 1px solid var(--line);
  border-radius: 12px;
  background: rgba(12, 15, 20, 0.9);
  overflow: hidden;
}

.featured_media {
  aspect-ratio: 4 / 5;
}

.portfolio_grid {
  margin-top: 0.95rem;
  display: grid;
  gap: 0.9rem;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

.card_media {
  aspect-ratio: 4 / 5;
}

.portfolio_copy {
  padding: 1rem;
  min-height: 136px;
  display: grid;
  align-content: start;
  gap: 0.42rem;
}

.portfolio_copy p {
  margin: 0;
  color: #98a5b3;
  font-size: 0.76rem;
  letter-spacing: 0.06em;
}

.portfolio_copy h3 {
  margin: 0;
  font-size: 1.06rem;
}

.portfolio_copy span {
  margin: 0;
  display: -webkit-box;
  color: var(--muted);
  line-height: 1.58;
  font-size: 0.88rem;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.portfolio_more_wrap {
  margin-top: 1.25rem;
  display: flex;
  justify-content: center;
}

.portfolio_more_button {
  min-width: 148px;
  min-height: 48px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.26);
  background: rgba(255, 255, 255, 0.04);
  color: #eef2f8;
  font: inherit;
  font-size: 0.92rem;
  font-weight: 700;
  cursor: pointer;
  transition: background-color 180ms ease, border-color 180ms ease;
}

.portfolio_more_button:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.46);
}

.contact_section {
  overflow: hidden;
  background: #000;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
}

.contact_shell {
  display: grid;
  gap: 1.25rem;
}

.contact_topline {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  align-items: center;
  gap: 0.75rem;
  padding-bottom: 0.62rem;
  border-bottom: 1px solid #1f242b;
  color: #7b8593;
  font-size: 0.8rem;
}

.contact_topline span:nth-child(2) {
  text-align: center;
}

.contact_topline span:nth-child(3) {
  text-align: right;
}

.contact_hero {
  display: grid;
  grid-template-columns: 1fr minmax(260px, 420px);
  align-items: end;
  gap: 1rem;
}

.contact_hero h2 {
  margin: 0;
  font-size: clamp(2.9rem, 8.2vw, 7.1rem);
  line-height: 0.9;
  letter-spacing: -0.03em;
}

.contact_hero p {
  margin: 0;
  color: #cad2dc;
  font-size: clamp(1.1rem, 2.4vw, 2.05rem);
  line-height: 1.08;
}

.contact_layout {
  display: grid;
  grid-template-columns: minmax(220px, 0.58fr) minmax(0, 1.42fr);
  gap: 1.1rem;
}

.contact_side {
  display: grid;
  align-content: start;
  gap: 0.95rem;
}

.contact_chip {
  width: max-content;
  padding: 0.34rem 0.62rem;
  border-radius: 2px;
  border: 1px solid #bec4cb;
  background: #f2f4f7;
  color: #0b0f14;
  font-size: 0.82rem;
  font-weight: 600;
}

.contact_side h3 {
  margin: 0;
  font-size: clamp(2.2rem, 4.2vw, 4rem);
  line-height: 1.05;
  letter-spacing: -0.02em;
}

.contact_side > p {
  margin: 0;
  color: #8f9aa7;
  font-size: 0.96rem;
  line-height: 1.7;
}

.contact_divider {
  height: 1px;
  background: #232a32;
}

.contact_meta {
  margin: 0;
  padding: 0;
  list-style: none;
  display: grid;
  gap: 0.42rem;
  color: #b0bac7;
  font-size: 0.88rem;
}

.contact_legal {
  color: #67717f;
  font-size: 0.78rem;
}

.contact_card {
  border: 1px solid #242a33;
  border-radius: 2px;
  background: linear-gradient(180deg, #111318, #0d0f13);
  padding: 1.5rem;
}

.contact_card h3 {
  margin: 0 0 1.15rem;
  font-size: clamp(1.75rem, 3vw, 2.15rem);
  letter-spacing: -0.01em;
}

.contact_form {
  display: grid;
  gap: 0.85rem;
}

.contact_form label {
  display: grid;
  gap: 0.36rem;
  font-size: 0.82rem;
  color: #d8e0ea;
}

.contact_form input,
.contact_form textarea {
  width: 100%;
  min-width: 0;
  min-height: 58px;
  border: 1px solid #313845;
  border-radius: 8px;
  background: #23262e;
  color: #f0f3f7;
  font: inherit;
  font-size: 0.9rem;
  padding: 0.78rem 0.88rem;
}

.contact_form input::placeholder,
.contact_form textarea::placeholder {
  color: #838b97;
}

.form_grid {
  display: grid;
  gap: 0.72rem;
  grid-template-columns: 1fr 1fr;
}

.contact_form button {
  margin-top: 0.18rem;
  border: 0;
  border-radius: 8px;
  min-height: 58px;
  background: #f1f3f6;
  color: #0e1116;
  font: inherit;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
}

.footer_banner_section {
  background: #000;
  border-top: 1px solid #1f242b;
}

.footer_banner {
  position: relative;
  width: 100%;
  border: 1px solid #1f242b;
  border-radius: 2px;
  min-height: clamp(250px, 36vw, 520px);
  background: radial-gradient(circle at 80% 0%, rgba(255, 255, 255, 0.06), transparent 28%),
    repeating-linear-gradient(-35deg, rgba(255, 255, 255, 0.022) 0px, rgba(255, 255, 255, 0.022) 1px, transparent 1px, transparent 6px),
    linear-gradient(180deg, #15171c, #111318);
  overflow: hidden;
}

.footer_banner_text {
  width: calc(100% - min(288px, 23vw));
  height: 100%;
  display: grid;
  align-items: center;
  padding: clamp(0.65rem, 1.7vw, 1.35rem);
}

.footer_banner_text h2 {
  margin: 0;
  color: #eceff3;
  font-size: clamp(5.8rem, 22vw, 21rem);
  letter-spacing: -0.01em;
  line-height: 0.78;
  white-space: nowrap;
}

.footer_banner_video {
  position: absolute;
  right: clamp(0.85rem, 1.8vw, 1.6rem);
  top: 50%;
  transform: translateY(-50%);
  width: min(288px, 23vw);
  aspect-ratio: 9 / 16;
  border-radius: 18px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: #000;
  overflow: hidden;
}

.footer_banner_video video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.site_footer {
  border-top: 1px solid var(--line);
  background: #090b0d;
}

.footer_inner {
  min-height: 68px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.9rem;
  color: #97a3b1;
  font-size: 0.82rem;
}

.footer_inner div {
  display: flex;
  gap: 0.72rem;
  flex-wrap: wrap;
}

.footer_inner a {
  color: #aab6c3;
}

.reveal {
  --reveal_x: 0px;
  --reveal_y: 28px;
  --reveal_scale: 1;
  opacity: 0;
  transform: translate3d(var(--reveal_x), var(--reveal_y), 0) scale(var(--reveal_scale));
  transition: opacity 820ms cubic-bezier(0.22, 1, 0.36, 1),
    transform 820ms cubic-bezier(0.22, 1, 0.36, 1),
    filter 820ms cubic-bezier(0.22, 1, 0.36, 1);
  transition-delay: var(--reveal_delay, 0ms);
  filter: blur(1px);
}

.reveal_up {
  --reveal_y: 28px;
}

.reveal_left {
  --reveal_x: -34px;
  --reveal_y: 0px;
}

.reveal_right {
  --reveal_x: 34px;
  --reveal_y: 0px;
}

.reveal_zoom {
  --reveal_y: 12px;
  --reveal_scale: 0.94;
}

.reveal.is_visible {
  opacity: 1;
  transform: translate3d(0, 0, 0) scale(1);
  filter: blur(0);
}

@media (max-width: 1180px) {
  .about_grid,
  .stats_grid,
  .portfolio_featured,
  .portfolio_grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .about_grid > :last-child:nth-child(odd),
  .portfolio_featured > :last-child:nth-child(odd),
  .portfolio_grid > :last-child:nth-child(odd) {
    grid-column: 1 / -1;
  }

  .journey_layout {
    grid-template-columns: 1fr;
    gap: 1.2rem;
  }
}

@media (max-width: 760px) {
  .container {
    width: min(1240px, calc(100% - 1.2rem));
  }

  .header_inner {
    min-height: 62px;
  }

  .nav {
    gap: 0.8rem;
    font-size: 0.8rem;
  }

  .media_sticky {
    position: static;
    height: auto;
    align-items: stretch;
    padding-top: 4.9rem;
    padding-bottom: 0.8rem;
  }

  .media_viewport {
    overflow: visible;
  }

  .video_track {
    display: grid;
    grid-template-columns: 1fr;
    gap: 0.62rem;
    padding: 0 1.2rem;
  }

  .video_item {
    flex: initial;
  }

  .section {
    padding: 4.6rem 0;
  }

  .journey_kicker {
    font-size: 0.95rem;
  }

  .journey_intro h2 {
    font-size: clamp(2.2rem, 11vw, 3.6rem);
  }

  .journey_row {
    grid-template-columns: 72px 1fr;
    gap: 0.72rem;
    padding: 0.9rem 0;
  }

  .journey_row strong {
    font-size: 1.35rem;
  }

  .journey_row ul {
    font-size: 0.93rem;
    line-height: 1.55;
  }

  .contact_topline {
    grid-template-columns: 1fr;
    gap: 0.2rem;
  }

  .contact_topline span:nth-child(2),
  .contact_topline span:nth-child(3) {
    text-align: left;
  }

  .contact_hero {
    grid-template-columns: 1fr;
    gap: 0.7rem;
  }

  .contact_hero p {
    max-width: 460px;
    font-size: 1.1rem;
  }

  .contact_layout {
    grid-template-columns: 1fr;
  }

  .footer_banner {
    min-height: auto;
    padding: 1rem;
    display: grid;
    gap: 0.85rem;
  }

  .footer_banner_text {
    width: 100%;
    padding: 0;
  }

  .footer_banner_text h2 {
    font-size: clamp(3.6rem, 20vw, 7rem);
    line-height: 0.82;
  }

  .footer_banner_video {
    position: relative;
    right: auto;
    top: auto;
    transform: none;
    width: min(320px, 72vw);
    justify-self: end;
  }

  .portfolio_grid,
  .portfolio_featured,
  .about_grid,
  .stats_grid {
    grid-template-columns: 1fr;
  }

  .form_grid {
    grid-template-columns: 1fr;
  }

  .footer_inner {
    min-height: auto;
    padding: 1rem 0;
    flex-direction: column;
    align-items: flex-start;
  }
}

@media (max-width: 560px) {
  .nav {
    display: none;
  }
}

@media (prefers-reduced-motion: reduce) {
  .reveal {
    opacity: 1;
    transform: none;
    filter: none;
    transition: none;
  }
}
</style>
