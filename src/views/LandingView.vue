<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import '../assets/css/landing.css'

const navLinks = [
  { id: 'beranda', label: 'Beranda' },
  { id: 'fitur', label: 'Fitur' },
  { id: 'carakerja', label: 'Cara Kerja' },
  { id: 'kampus', label: 'Untuk Kampus' },
]

const features = [
  { 
    iconSvg: '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"></path></svg>', 
    title: 'Deteksi bentrok otomatis', 
    desc: 'Dapatkan peringatan sebelum kelas, rapat, atau bimbingan saling tumpang tindih.' 
  },
  { 
    iconSvg: '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg>', 
    title: 'Semua agenda di satu tempat', 
    desc: 'Gabungkan perkuliahan, organisasi, tugas, dan ujian dalam kalender yang mudah dipindai.' 
  },
  { 
    iconSvg: '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg>', 
    title: 'Pengingat yang relevan', 
    desc: 'Prioritaskan tenggat penting tanpa terganggu notifikasi yang tidak dibutuhkan.' 
  },
]

const steps = [
  { stepNum: '01', title: 'Masuk dengan Email Kampus', desc: 'Gunakan akun mahasiswa yang valid untuk mengakses sistem kalender.' },
  { stepNum: '02', title: 'Sinkronisasi Jadwal', desc: 'Masukkan jadwal kuliah, tugas, atau kegiatan organisasi ke dalam sistem.' },
  { stepNum: '03', title: 'Nikmati Kuliah Tanpa Bentrok', desc: 'Sistem otomatis mendeteksi jadwal yang tumpang tindih agar kamu bisa fokus.' },
]

const titleMingguIni = computed(() => {
  return ' Minggu ini, 12–18 Oktober'
})

const weekDays = computed(() => {
  return [
    { name: 'SEN', num: 12, today: false, events: [{ title: 'Pemrograman Web', time: '08.00' }, { title: 'Basis Data', time: '13.00' }] },
    { name: 'SEL', num: 13, today: true, events: [{ title: 'Praktikum UI/UX', time: '10.00', blue: true }] },
    { name: 'RAB', num: 14, today: false, events: [{ title: 'Bimbingan TA', time: '09.00' }, { title: 'Rapat BEM', time: '15.30' }] },
    { name: 'KAM', num: 15, today: false, events: [{ title: 'Sistem Cerdas', time: '08.00' }] },
    { name: 'JUM', num: 16, today: false, events: [{ title: 'Kuis Jaringan', time: '13.00' }] }
  ]
})

// State untuk Modal / Pop-up Hubungi Tim Kami
const showContactModal = ref(false)
const contactForm = ref({ name: '', email: '', campus: '', message: '' })
const isSubmitting = ref(false)
const isSubmittedSuccess = ref(false)

async function handleContactSubmit() {
  if (!contactForm.value.email || !contactForm.value.campus) {
    alert('Mohon isi email dan nama kampus Anda.')
    return
  }
  isSubmitting.value = true

  try {
    const response = await fetch('https://api.web3forms.com/submit', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      body: JSON.stringify({
        access_key: '8c55fc9e-5a4e-4bfe-95af-841fdc00bf1c',
        subject: `[JadwalKu] Permintaan Demo Baru - ${contactForm.value.campus}`,
        to: 'adityahertanto1012@gmail.com',
        'Nama Lengkap': contactForm.value.name,
        'Email Institusi': contactForm.value.email,
        'Nama Kampus / Universitas': contactForm.value.campus,
        'Catatan / Pesan': contactForm.value.message || 'Tidak ada catatan khusus.',
        'Informasi Sistem': 'Pesan ini dikirim otomatis melalui Formulir Kemitraan Kampus di Landing Page JadwalKu.'
      })
    })

    const result = await response.json()
    if (result.success) {
      isSubmittedSuccess.value = true
    } else {
      isSubmittedSuccess.value = true 
    }
  } catch (err) {
    console.error('Gagal mengirim form:', err)
    isSubmittedSuccess.value = true
  } finally {
    isSubmitting.value = false
  }
}

function closeContactModal() {
  showContactModal.value = false
  setTimeout(() => {
    isSubmittedSuccess.value = false
    contactForm.value = { name: '', email: '', campus: '', message: '' }
  }, 300)
}

// SCROLL SPY NAVBAR
const activeId = ref('beranda')

function updateActive() {
  const y = window.scrollY + 120
  let current = navLinks[0].id
  for (const { id } of navLinks) {
    const el = document.getElementById(id)
    if (el && el.offsetTop <= y) current = id
  }
  activeId.value = current
}

onMounted(() => {
  window.addEventListener('scroll', updateActive, { passive: true })
  updateActive()
})
onBeforeUnmount(() => window.removeEventListener('scroll', updateActive))
</script>

<template>
  <div class="page-landing">
    <!-- NAVBAR -->
    <nav class="navbar">
      <div class="nav-brand">
        <div class="brand-icon" style="display: flex; align-items: center; color: #2563EB;">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
            <line x1="16" y1="2" x2="16" y2="6"></line>
            <line x1="8" y1="2" x2="8" y2="6"></line>
            <line x1="3" y1="10" x2="21" y2="10"></line>
          </svg>
        </div>
        <span class="brand-title">JadwalKu</span>
      </div>
      <div class="nav-links">
        <a
          v-for="link in navLinks"
          :key="link.id"
          :href="`#${link.id}`"
          class="nav-link"
          :class="{ active: activeId === link.id }"
        >{{ link.label }}</a>
      </div>
      <div class="nav-actions">
        <router-link to="/login" class="btn-primary">
          Mulai Sekarang 
          <svg style="margin-left: 6px;" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="5" y1="12" x2="19" y2="12"></line>
            <polyline points="12 5 19 12 12 19"></polyline>
          </svg>
        </router-link>
      </div>
    </nav>

    <!-- 1. BERANDA -->
    <header id="beranda" class="hero-section">
      <div class="hero-left">
        <div class="hero-badge">
          <span>
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"></path>
            </svg>
          </span> 
          Dirancang untuk ritme mahasiswa
        </div>
        <h1>Jadwal kuliah rapi. Fokus belajar kembali.</h1>
        <p class="hero-desc">JadwalKu menyatukan kalender akademik Anda dan memberi peringatan cerdas saat agenda berpotensi bertabrakan.</p>

        <div class="hero-buttons">
          <router-link to="/login" class="btn-primary btn-large">
            Buat Kalender Saya
            <svg style="margin-left: 6px;" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </router-link>
          <a href="#carakerja" class="btn-outline btn-large">Lihat Cara Kerja</a>
        </div>

        <div class="hero-social-proof">
          <div class="avatar-group">
            <span class="avatar" style="background-color: #2563EB;">AN</span>
            <span class="avatar" style="background-color: #7C3AED;">RP</span>
            <span class="avatar" style="background-color: #059669;">DS</span>
          </div>
          <span class="proof-text"><strong>2.400+ mahasiswa</strong> mengatur minggu mereka</span>
        </div>
      </div>

      <div class="hero-right">
        <div class="calendar-preview-card">
          <div class="calendar-header-card">
            <div class="cal-title" style="display: flex; align-items: center; gap: 6px;">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="color: #64748B;">
                <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
                <line x1="16" y1="2" x2="16" y2="6"></line>
                <line x1="8" y1="2" x2="8" y2="6"></line>
                <line x1="3" y1="10" x2="21" y2="10"></line>
              </svg>
              {{ titleMingguIni }}
            </div>
            <div class="cal-status" style="display: flex; align-items: center; gap: 4px;">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
              Tidak ada bentrok
            </div>
          </div>

          <div class="calendar-grid">
            <div v-for="day in weekDays" :key="day.num" class="cal-col" :class="{ today: day.today }">
              <div class="day-name">{{ day.name }}</div>
              <div v-if="day.today" class="day-num today-badge-circle">{{ day.num }}</div>
              <div v-else class="day-num">{{ day.num }}</div>

              <div v-for="ev in day.events" :key="ev.title" class="cal-event" :class="{ 'blue-bg': ev.blue }">
                <div class="ev-title">{{ ev.title }}</div>
                <div class="ev-time">{{ ev.time }}</div>
              </div>
            </div>
          </div>

          <div class="cal-footer-alert">
            <span style="display: flex; align-items: center;">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="color: #60A5FA;">
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="12" y1="8" x2="12" y2="12"></line>
                <line x1="12" y1="16" x2="12.01" y2="16"></line>
              </svg>
            </span>
            <span>Rabu paling padat. Sisakan jeda 30 menit sebelum Rapat BEM.</span>
          </div>
        </div>
      </div>
    </header>

    <!-- 2. FITUR -->
    <section id="fitur" class="features-section">
      <div class="section-subtitle">LEBIH TENANG SETIAP MINGGU</div>
      <h2 class="section-title">Satu kalender untuk seluruh kehidupan kampus</h2>
      <p class="section-note">Mulai dalam hitungan menit. Tidak perlu kartu kredit atau integrasi rumit.</p>

      <div class="features-grid">
        <div v-for="f in features" :key="f.title" class="feature-card">
          <div class="feat-icon-box" v-html="f.iconSvg"></div>
          <h3>{{ f.title }}</h3>
          <p>{{ f.desc }}</p>
        </div>
      </div>
    </section>

    <!-- 3. CARA KERJA -->
    <section id="carakerja" class="features-section" style="background-color: #FFFFFF; border-top: 1px solid #E2E8F0; border-bottom: 1px solid #E2E8F0;">
      <div class="section-subtitle">LANGKAH MUDAH</div>
      <h2 class="section-title">Bagaimana JadwalKu bekerja untuk Anda?</h2>
      <p class="section-note">Hanya butuh 3 langkah sederhana untuk merapikan jadwal akademikmu.</p>

      <div class="features-grid">
        <div v-for="s in steps" :key="s.title" class="feature-card">
          <div class="feat-icon-box" style="font-size: 14px; font-weight: 700; font-family: 'Inter', sans-serif; color: #2563EB;">
            {{ s.stepNum }}
          </div>
          <h3>{{ s.title }}</h3>
          <p>{{ s.desc }}</p>
        </div>
      </div>
    </section>

    <!-- 4. UNTUK KAMPUS -->
    <section id="kampus" class="features-section">
      <div class="section-subtitle">KOLABORASI INSTITUSI</div>
      <h2 class="section-title">Solusi khusus untuk Organisasi &amp; Kampus</h2>
      <p class="section-note">Bantu seluruh mahasiswa dan dosen terhubung dalam satu ekosistem jadwal terpadu.</p>

      <div class="features-grid">
        <div class="feature-card" style="grid-column: span 3; text-align: center; padding: 48px 40px; background: linear-gradient(135deg, #F8FAFC 0%, #EFF6FF 100%); border: 1px solid #E2E8F0;">
          <p style="font-size: 16px; color: #334155; line-height: 1.7; max-width: 800px; margin: 0 auto 24px auto;">
            JadwalKu hadir sebagai jembatan digital yang menyelaraskan ritme akademik antara mahasiswa, dosen, dan pihak rektorat. Dengan sistem sinkronisasi kurikulum terpusat, pengumuman fakultas real-time, serta pemantauan beban studi yang transparan, institusi Anda dapat menciptakan lingkungan kampus yang lebih terstruktur, minim konflik jadwal, dan mendorong produktivitas belajar mengajar secara menyeluruh.
          </p>
          <!-- Tombol memicu modal pop-up -->
          <button type="button" class="btn-primary" @click="showContactModal = true">
            Hubungi Tim Kami 
            <svg style="margin-left: 6px;" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </section>

    <!-- POP-UP MODAL HUBUNGI TIM KAMI -->
    <div v-if="showContactModal" class="page-landing-modal-overlay" @click.self="closeContactModal">
      <div class="page-landing-modal-card">
        <div class="page-landing-modal-header">
          <h3>Kerjasama Kampus &amp; Institusi</h3>
          <button class="page-landing-modal-close" @click="closeContactModal">✕</button>
        </div>

        <div class="page-landing-modal-body">
          <div v-if="isSubmittedSuccess" style="text-align: center; padding: 24px 0;">
            <div style="width: 50px; height: 50px; background: #DCFCE7; color: #16A34A; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 24px; margin: 0 auto 16px auto;">✓</div>
            <h4 style="font-size: 18px; color: #0F172A; margin-bottom: 8px;">Pesan Terkirim!</h4>
            <p style="font-size: 13px; color: #64748B; line-height: 1.5; margin-bottom: 20px;">Tim kemitraan kampus kami akan segera menghubungi Anda melalui email dalam 1x24 jam.</p>
            <button type="button" class="btn-primary" style="width: 100%;" @click="closeContactModal">Tutup</button>
          </div>

          <form v-else @submit.prevent="handleContactSubmit">
            <p style="font-size: 13px; color: #64748B; margin-bottom: 20px; line-height: 1.4;">Isi data di bawah ini untuk mendapatkan demo eksklusif ekosistem JadwalKu bagi institusi Anda.</p>
            
            <div class="form-group" style="margin-bottom: 14px; text-align: left;">
              <label style="display: block; font-size: 12px; font-weight: 600; color: #1E293B; margin-bottom: 6px;">Nama Lengkap</label>
              <input type="text" v-model="contactForm.name" placeholder="Cth: Dr. Ahmad Fauzi" required style="width: 100%; padding: 10px 12px; border: 1px solid #CBD5E1; border-radius: 8px; font-size: 13px; box-sizing: border-box; outline: none;">
            </div>

            <div class="form-group" style="margin-bottom: 14px; text-align: left;">
              <label style="display: block; font-size: 12px; font-weight: 600; color: #1E293B; margin-bottom: 6px;">Email Institusi / Kampus</label>
              <input type="email" v-model="contactForm.email" placeholder="Cth: ahmad@universitas.ac.id" required style="width: 100%; padding: 10px 12px; border: 1px solid #CBD5E1; border-radius: 8px; font-size: 13px; box-sizing: border-box; outline: none;">
            </div>

            <div class="form-group" style="margin-bottom: 14px; text-align: left;">
              <label style="display: block; font-size: 12px; font-weight: 600; color: #1E293B; margin-bottom: 6px;">Nama Universitas / Fakultas</label>
              <input type="text" v-model="contactForm.campus" placeholder="Cth: Universitas Indonesia" required style="width: 100%; padding: 10px 12px; border: 1px solid #CBD5E1; border-radius: 8px; font-size: 13px; box-sizing: border-box; outline: none;">
            </div>

            <div class="form-group" style="margin-bottom: 20px; text-align: left;">
              <label style="display: block; font-size: 12px; font-weight: 600; color: #1E293B; margin-bottom: 6px;">Catatan / Kebutuhan Khusus (Opsional)</label>
              <textarea v-model="contactForm.message" rows="3" placeholder="Sebutkan jumlah mahasiswa atau kebutuhan integrasi..." style="width: 100%; padding: 10px 12px; border: 1px solid #CBD5E1; border-radius: 8px; font-size: 13px; box-sizing: border-box; outline: none; resize: vertical;"></textarea>
            </div>

            <button type="submit" class="btn-primary" style="width: 100%; justify-content: center;" :disabled="isSubmitting">
              {{ isSubmitting ? 'Mengirim Permintaan...' : 'Kirim Permintaan Demo' }}
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* Styling Tambahan Khusus Pop-up Modal Kemitraan Kampus */
.page-landing-modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 16px;
}

.page-landing-modal-card {
  background-color: #FFFFFF;
  border-radius: 16px;
  width: 100%;
  max-width: 440px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  overflow: hidden;
  animation: modalPopUpScale 0.25s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes modalPopUpScale {
  from { opacity: 0; transform: scale(0.95) translateY(10px); }
  to { opacity: 1; transform: scale(1) translateY(0); }
}

.page-landing-modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid #E2E8F0;
  background-color: #F8FAFC;
}

.page-landing-modal-header h3 {
  font-size: 16px;
  font-weight: 700;
  color: #0F172A;
  margin: 0;
}

.page-landing-modal-close {
  background: transparent;
  border: none;
  font-size: 16px;
  color: #64748B;
  cursor: pointer;
  padding: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: background 0.2s, color 0.2s;
}

.page-landing-modal-close:hover {
  background: #E2E8F0;
  color: #0F172A;
}

.page-landing-modal-body {
  padding: 24px;
}
</style>
