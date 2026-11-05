<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { login } from '../stores/session'
import { supabase } from '../utils/supabase' // Koneksi Supabase
import '../assets/css/login.css'

const router = useRouter()
const email = ref('')
const password = ref('')
const remember = ref(false)
const showPassword = ref(false)
const hasError = ref(false)
const isLoading = ref(false)
const isExiting = ref(false)

// State untuk modal kustom peringatan akun belum terdaftar
const showUnregisteredModal = ref(false)

onMounted(async () => {
  const remembered = localStorage.getItem('rememberedEmail')
  if (remembered) {
    email.value = remembered
    remember.value = true
  }

  // 🚀 PERIKSA SESI GOOGLE OAUTH SAAT REDIRECT KEMBALI KE HALAMAN LOGIN
  const { data: { session } } = await supabase.auth.getSession()
  if (session && session.user) {
    const googleEmail = session.user.email

    // Cek apakah email Google sudah terdaftar di tabel 'users' database lokal Anda
    const { data: dbUser } = await supabase
      .from('users')
      .select('*')
      .eq('email', googleEmail)
      .maybeSingle()

    if (!dbUser) {
      // Jika belum terdaftar, paksa keluar (sign out) dari sesi Google dan tampilkan modal
      await supabase.auth.signOut()
      showUnregisteredModal.value = true
    } else {
      // Jika sudah terdaftar, izinkan masuk ke dashboard
      login(dbUser.email)
      router.push('/dashboard')
    }
  }
})

function clearError() {
  hasError.value = false
}

// Login konvensional email & password via Supabase Database
async function handleLogin() {
  isLoading.value = true
  clearError()

  try {
    const { data, error } = await supabase
      .from('users')
      .select('*')
      .eq('email', email.value)
      .eq('password', password.value)
      .maybeSingle()

    if (error) throw error

    if (data) {
      login(data.email)
      isExiting.value = true
      setTimeout(() => {
        router.push('/dashboard')
      }, 300)
    } else {
      hasError.value = true
    }
  } catch (err) {
    console.error('Terjadi kesalahan:', err.message)
    hasError.value = true
  } finally {
    isLoading.value = false
  }
}

// 🚀 FUNGSI GOOGLE LOGIN: Mengarahkan kembali ke /login untuk divalidasi
async function handleGoogleLogin() {
  try {
    const { error } = await supabase.auth.signInWithOAuth({
      provider: 'google',
      options: {
        redirectTo: window.location.origin + '/login',
        queryParams: {
          prompt: 'select_account' // Memaksa Google menampilkan opsi pilihan akun setiap kali login ditekan
        }
      }
    })
    if (error) throw error
  } catch (err) {
    console.error('Gagal login dengan Google:', err.message)
    alert('Terjadi kesalahan saat menghubungkan ke Google Auth.')
  }
}

function handleForgotPassword() {
  alert('Silakan hubungi administrator kampus untuk pemulihan kata sandi.')
}

function redirectToRegister() {
  showUnregisteredModal.value = false
  router.push('/register')
}
</script>

<template>
  <div class="page-login" :class="{ 'fade-out': isExiting }">
    <!-- SISI KIRI: BRANDING & FITUR -->
    <div class="left-panel">
      <div class="brand">
        <div class="brand-icon" style="display: flex; align-items: center; color: #FFFFFF;">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
            <line x1="16" y1="2" x2="16" y2="6"></line>
            <line x1="8" y1="2" x2="8" y2="6"></line>
            <line x1="3" y1="10" x2="21" y2="10"></line>
          </svg>
        </div>
        <div class="brand-name">JadwalKu</div>
      </div>

      <div class="left-content">
        <div class="badge">
          <span>
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right: 4px;">
              <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"></path>
            </svg>
          </span>
          Kalender akademik pintar
        </div>
        <h1>Kuliah lebih teratur, tanpa jadwal bertabrakan.</h1>
        <p>Susun perkuliahan, rapat organisasi, dan tenggat tugas dalam satu kalender yang otomatis mendeteksi konflik.</p>

        <div class="feature-list">
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Deteksi konflik sebelum agenda disimpan</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Ringkasan beban akademik mingguan</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Pengingat tugas dan ujian tepat waktu</span>
          </div>
        </div>
      </div>

      <div class="testimonial-box">
        <p class="testimonial-text">"JadwalKu membantu saya menghindari rapat BEM yang bersamaan dengan kelas praktikum."</p>
        <div class="testimonial-author">Nadia • Mahasiswa Sistem Informasi</div>
      </div>
    </div>

    <!-- SISI KANAN: FORM LOGIN -->
    <div class="right-panel">
      <div class="login-card">
        <div class="mobile-brand">
          <div class="brand-icon" style="display: flex; align-items: center; color: #FFFFFF;">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
              <line x1="16" y1="2" x2="16" y2="6"></line>
              <line x1="8" y1="2" x2="8" y2="6"></line>
              <line x1="3" y1="10" x2="21" y2="10"></line>
            </svg>
          </div>
          <div class="brand-name">JadwalKu</div>
        </div>

        <div class="login-header">
          <div class="login-subtitle">Selamat datang kembali</div>
          <h2>Masuk ke akun mahasiswa</h2>
          <p>Gunakan akun Google atau email terdaftar untuk melanjutkan.</p>
        </div>

        <!-- TOMBOL GOOGLE SIGN-IN -->
        <button type="button" class="btn-google-login" @click="handleGoogleLogin" style="width: 100%; display: flex; align-items: center; justify-content: center; gap: 10px; background-color: #FFFFFF; color: #334155; border: 1px solid #CBD5E1; padding: 11px 16px; border-radius: 8px; font-weight: 600; font-size: 14px; cursor: pointer; transition: background 0.2s; margin-bottom: 20px;">
          <svg width="18" height="18" viewBox="0 0 24 24">
            <path fill="#4285F4" d="M23.745 12.27c0-.7-.06-1.4-.19-2.07H12v4.51h6.6c-.29 1.52-1.14 2.82-2.4 3.68v3.05h3.88c2.27-2.09 3.66-5.17 3.66-9.17z"/>
            <path fill="#34A853" d="M12 24c3.24 0 5.95-1.08 7.93-2.91l-3.88-3.05c-1.08.72-2.45 1.16-4.05 1.16-3.13 0-5.78-2.11-6.73-4.96H1.18v3.15C3.16 21.32 7.22 24 12 24z"/>
            <path fill="#FBBC05" d="M5.27 14.24c-.25-.72-.39-1.5-.39-2.24s.14-1.52.39-2.24V6.61H1.18C.43 8.13 0 9.84 0 12s.43 3.87 1.18 5.39l4.09-3.15z"/>
            <path fill="#EA4335" d="M12 4.75c1.77 0 3.35.61 4.6 1.8l3.42-3.42C17.95 1.19 15.24 0 12 0 7.22 0 3.16 2.68 1.18 6.61l4.09 3.15c.95-2.85 3.6-4.96 6.73-4.96z"/>
          </svg>
          Masuk dengan Google
        </button>

        <div style="display: flex; align-items: center; text-align: center; color: #94A3B8; margin-bottom: 20px; font-size: 13px;">
          <div style="flex: 1; border-bottom: 1px solid #E2E8F0;"></div>
          <span style="padding: 0 10px;">Atau dengan email</span>
          <div style="flex: 1; border-bottom: 1px solid #E2E8F0;"></div>
        </div>

        <div v-if="hasError" class="error-alert" role="alert">
          <div class="error-icon" style="display: flex; align-items: center; color: #DC2626;">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
              <line x1="12" y1="9" x2="12" y2="13"></line>
              <line x1="12" y1="17" x2="12.01" y2="17"></line>
            </svg>
          </div>
          <div>
            <div class="error-title">Email atau kata sandi belum tepat</div>
            <div class="error-desc">Periksa kembali email dan kata sandi, lalu coba lagi.</div>
          </div>
        </div>

        <form @submit.prevent="handleLogin">
          <div class="form-group">
            <label class="form-label" for="email">Email</label>
            <input
              id="email"
              v-model="email"
              type="email"
              class="form-input"
              :class="{ error: hasError }"
              placeholder="Masukkan Email (cth: nama@email.com)"
              required
              @input="clearError"
            >
          </div>

          <div class="form-group">
            <label class="form-label" for="password">Kata sandi</label>
            <div class="input-container">
              <input
                id="password"
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                class="form-input"
                :class="{ error: hasError }"
                placeholder="Masukkan Password"
                required
                @input="clearError"
              >
              <button
                type="button"
                class="password-toggle"
                :aria-label="showPassword ? 'Sembunyikan kata sandi' : 'Tampilkan kata sandi'"
                @click="showPassword = !showPassword"
              >
                <svg v-if="!showPassword" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path>
                  <circle cx="12" cy="12" r="3"></circle>
                </svg>
                <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path>
                  <line x1="1" y1="1" x2="23" y2="23"></line>
                </svg>
              </button>
            </div>
          </div>

          <div class="form-options">
            <label class="remember-me">
              <input v-model="remember" type="checkbox"> Ingat saya
            </label>
            <a href="#" class="forgot-password" @click.prevent="handleForgotPassword">Lupa kata sandi?</a>
          </div>

          <button type="submit" class="btn-submit" :disabled="isLoading">
            <span style="display: inline-flex; align-items: center;">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <line x1="5" y1="12" x2="19" y2="12"></line>
                <polyline points="12 5 19 12 12 19"></polyline>
              </svg>
            </span> 
            {{ isLoading ? 'Memeriksa Database...' : 'Masuk ke JadwalKu' }}
          </button>

          <div style="text-align: center; margin-top: 16px; font-size: 13px; color: #64748B;">
            Belum punya akun? 
            <router-link to="/register" style="color: #2563EB; font-weight: 600; text-decoration: none;">Daftar di sini</router-link>
          </div>
        </form>
      </div>
    </div>

    <!-- MODAL KUSTOM: AKUN BELUM TERDAFTAR DI HALAMAN LOGIN -->
    <div v-if="showUnregisteredModal" class="modal-overlay">
      <div class="modal-card" style="text-align: center; max-width: 400px; padding: 32px 24px;">
        <div style="display: flex; justify-content: center; margin-bottom: 16px;">
          <div style="width: 56px; height: 56px; background: #FEF3C7; color: #D97706; border-radius: 50%; display: flex; align-items: center; justify-content: center;">
            <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
              <line x1="12" y1="9" x2="12" y2="13"></line>
              <line x1="12" y1="17" x2="12.01" y2="17"></line>
            </svg>
          </div>
        </div>

        <h3 style="margin-bottom: 8px; font-size: 18px; color: #0F172A; font-weight: 700;">Akun Belum Terdaftar</h3>
        <p style="font-size: 13.5px; color: #64748B; margin-bottom: 24px; line-height: 1.5;">
          Akun Google ini belum terdaftar di sistem <strong>JadwalKu</strong>. Silakan lakukan pendaftaran terlebih dahulu untuk mengakses kalender akademik.
        </p>

        <div style="display: flex; gap: 10px;">
          <button type="button" class="btn-outline" style="flex: 1; padding: 11px;" @click="showUnregisteredModal = false">
            Tutup
          </button>
          <button type="button" class="btn-primary" style="flex: 1; padding: 11px; background-color: #2563EB;" @click="redirectToRegister">
            Daftar Sekarang
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
