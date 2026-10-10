<script setup>
import { computed, onMounted, ref } from 'vue'

const activePage = ref('grade')

// โปรแกรมตัดเกรด
const score = ref('')
const gradeMessage = ref('')
const grade = computed(() => {
  if (score.value === '' || score.value === null) return ''
  const number = Number(score.value)
  if (!Number.isFinite(number) || number < 0 || number > 100) return 'คะแนนไม่ถูกต้อง'
  if (number >= 80) return 'A'
  if (number >= 70) return 'B'
  if (number >= 60) return 'C'
  if (number >= 50) return 'D'
  return 'F'
})
const gradeDescription = computed(() => {
  const descriptions = {
    A: 'ดีเยี่ยม',
    B: 'ดีมาก',
    C: 'ดี',
    D: 'พอใช้',
    F: 'ไม่ผ่าน'
  }
  return descriptions[grade.value] || ''
})
function checkGrade() {
  if (score.value === '') {
    gradeMessage.value = 'กรุณากรอกคะแนนก่อน'
  } else if (Number(score.value) < 0 || Number(score.value) > 100 || !Number.isFinite(Number(score.value))) {
    gradeMessage.value = 'กรุณากรอกคะแนนตั้งแต่ 0 ถึง 100'
  } else {
    gradeMessage.value = `คะแนน ${Number(score.value)} คะแนน ได้เกรด ${grade.value} (${gradeDescription.value})`
  }
}

// โปรแกรมดึงราคาทองคำจาก API
const gold = ref(null)
const loading = ref(false)
const apiError = ref('')
const lastFetched = ref('')

const apiUrl = 'https://api.chnwt.dev/thai-gold-api/latest'

function formatPrice(value) {
  if (value === undefined || value === null || value === '') return 'ไม่มีข้อมูล'
  const parsed = Number(String(value).replaceAll(',', ''))
  return Number.isFinite(parsed)
    ? parsed.toLocaleString('th-TH', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
    : String(value)
}

async function fetchGoldPrice() {
  loading.value = true
  apiError.value = ''
  try {
    const response = await fetch(apiUrl)
    if (!response.ok) throw new Error(`เซิร์ฟเวอร์ตอบกลับสถานะ ${response.status}`)
    const data = await response.json()
    if (data.status !== 'success' || !data.response?.price) {
      throw new Error('รูปแบบข้อมูลจาก API ไม่ตรงตามที่คาดไว้')
    }
    gold.value = data.response
    lastFetched.value = new Date().toLocaleString('th-TH')
  } catch (error) {
    apiError.value = 'ไม่สามารถดึงราคาทองคำได้ในขณะนี้ กรุณาตรวจสอบอินเทอร์เน็ตแล้วกดลองใหม่อีกครั้ง'
    console.error('Gold API error:', error)
  } finally {
    loading.value = false
  }
}

onMounted(fetchGoldPrice)
</script>

<template>
  <main class="app-shell">
    <header class="hero">
      <div class="hero-badge">VUE.JS PRACTICE</div>
      <h1>แบบฝึกหัด Vue.js</h1>
      <p>โปรแกรมตัดเกรด และโปรแกรมดึงราคาทองคำผ่าน API</p>
      <div class="hero-line"></div>
    </header>

    <nav class="tabs" aria-label="เลือกโปรแกรม">
      <button :class="{ active: activePage === 'grade' }" @click="activePage = 'grade'">
        <span class="tab-icon">📘</span> โปรแกรมตัดเกรด
      </button>
      <button :class="{ active: activePage === 'gold' }" @click="activePage = 'gold'; fetchGoldPrice()">
        <span class="tab-icon">🪙</span> ราคาทองคำผ่าน API
      </button>
    </nav>

    <section v-if="activePage === 'grade'" class="panel">
      <div class="section-heading">
        <div class="heading-icon purple">A+</div>
        <div>
          <h2>โปรแกรมตัดเกรด</h2>
          <p>กรอกคะแนนระหว่าง 0–100 แล้วกดปุ่มเพื่อคำนวณเกรด</p>
        </div>
      </div>

      <form class="grade-form" @submit.prevent="checkGrade">
        <label for="score">คะแนนที่ได้</label>
        <div class="input-row">
          <input id="score" v-model="score" type="number" min="0" max="100" step="any"
                 placeholder="กรอกคะแนน เช่น 85" />
          <span class="input-suffix">/ 100</span>
        </div>
        <button class="primary-button" type="submit">คำนวณเกรด</button>
      </form>

      <div v-if="gradeMessage" class="result-box" :class="{ invalid: grade === 'คะแนนไม่ถูกต้อง' || score === '' || Number(score) < 0 || Number(score) > 100 }">
        <div class="result-label">ผลการประเมิน</div>
        <div class="grade-letter" v-if="grade && grade !== 'คะแนนไม่ถูกต้อง' && score !== '' && Number(score) >= 0 && Number(score) <= 100">{{ grade }}</div>
        <p>{{ gradeMessage }}</p>
      </div>

      <div class="criteria">
        <h3>เกณฑ์การตัดเกรด</h3>
        <div class="criteria-grid">
          <div><b class="grade-a">A</b><span>80–100</span></div>
          <div><b class="grade-b">B</b><span>70–79</span></div>
          <div><b class="grade-c">C</b><span>60–69</span></div>
          <div><b class="grade-d">D</b><span>50–59</span></div>
          <div><b class="grade-f">F</b><span>0–49</span></div>
        </div>
        <small>* เกณฑ์ตัวอย่าง สามารถปรับให้ตรงกับเกณฑ์ที่อาจารย์กำหนดได้</small>
      </div>
    </section>

    <section v-else class="panel">
      <div class="section-heading">
        <div class="heading-icon gold-icon">฿</div>
        <div>
          <h2>ราคาทองคำวันนี้</h2>
          <p>ข้อมูลจาก API ราคาทองคำของประเทศไทย</p>
        </div>
      </div>

      <div class="api-status">
        <span class="status-dot" :class="{ busy: loading, error: apiError }"></span>
        {{ loading ? 'กำลังดึงข้อมูล...' : apiError ? 'เชื่อมต่อไม่สำเร็จ' : 'ข้อมูลจาก API' }}
      </div>

      <div v-if="loading && !gold" class="loading-card">
        <div class="spinner"></div>
        <p>กำลังโหลดราคาทองคำ...</p>
      </div>

      <div v-if="apiError" class="error-card">
        <p>{{ apiError }}</p>
        <button class="primary-button" @click="fetchGoldPrice">ลองใหม่อีกครั้ง</button>
      </div>

      <template v-if="gold">
        <div class="update-info">
          <span>วันที่ประกาศ: <strong>{{ gold.update_date || 'ไม่ระบุ' }}</strong></span>
          <span>{{ gold.update_time || '' }}</span>
        </div>
        <div class="price-grid">
          <article class="price-card">
            <div class="price-card-top"><span>ทองคำแท่ง</span><span class="coin">▰</span></div>
            <div class="price-type">รับซื้อ</div>
            <div class="price">{{ formatPrice(gold.price?.gold_bar?.buy) }} <small>บาท</small></div>
            <div class="price-divider"></div>
            <div class="price-type">ขายออก</div>
            <div class="price">{{ formatPrice(gold.price?.gold_bar?.sell) }} <small>บาท</small></div>
          </article>
          <article class="price-card jewelry">
            <div class="price-card-top"><span>ทองรูปพรรณ</span><span class="coin">◇</span></div>
            <div class="price-type">รับซื้อ</div>
            <div class="price">{{ formatPrice(gold.price?.gold?.buy) }} <small>บาท</small></div>
            <div class="price-divider"></div>
            <div class="price-type">ขายออก</div>
            <div class="price">{{ formatPrice(gold.price?.gold?.sell) }} <small>บาท</small></div>
          </article>
        </div>
        <div class="source-note">
          <span>ⓘ</span>
          <p>ราคาที่แสดงเป็นข้อมูลอ้างอิงจาก API อาจมีการเปลี่ยนแปลง โปรดตรวจสอบราคากับร้านทองก่อนซื้อขาย</p>
        </div>
        <button class="refresh-button" :disabled="loading" @click="fetchGoldPrice">
          <span>↻</span> {{ loading ? 'กำลังอัปเดต...' : 'อัปเดตราคาล่าสุด' }}
        </button>
        <p v-if="lastFetched" class="last-fetched">ดึงข้อมูลล่าสุดเมื่อ {{ lastFetched }}</p>
      </template>
      <div class="api-footnote">API: api.chnwt.dev/thai-gold-api/latest</div>
    </section>

    <footer>
      <span>แบบฝึกหัดที่ 16</span>
      <span>Vue.js · HTML · CSS · API</span>
    </footer>
  </main>
</template>