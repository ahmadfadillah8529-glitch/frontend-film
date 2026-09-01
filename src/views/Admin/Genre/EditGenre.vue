<template>
  <div class="container">
    <RouterLink to="/kelola-genre" class="btn-back">← Kembali</RouterLink>
    <h1>✏️ Edit Genre</h1>

    <p v-if="loadingData" class="loading-text">⏳ Memuat data...</p>

    <div v-else class="form-wrapper">
      <form @submit.prevent="submitGenre">
        <div class="form-group">
          <label>Nama Genre <span class="required">*</span></label>
          <input v-model="form.nama_genre" type="text" required class="form-input" />
        </div>
        <button type="submit" :disabled="loading" class="btn-submit">
          <span v-if="loading">⏳ Menyimpan...</span>
          <span v-else>💾 Simpan Perubahan</span>
        </button>
        <p v-if="errorMsg" class="error-msg">{{ errorMsg }}</p>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted }            from 'vue'
import { useRouter, useRoute, RouterLink } from 'vue-router'
import api                           from '../../../utils/api'

const router  = useRouter()
const route   = useRoute()
const genreId = route.params.id

const form        = ref({ nama_genre: '' })
const loadingData = ref(true)
const loading     = ref(false)
const errorMsg    = ref('')

onMounted(async () => {
  try {
    // Karena tidak ada GET /genre/:id, ambil semua lalu filter
    const res     = await api.get('/genre')
    const current = res.data.data.find(g => g.id == genreId)
    if (current) { form.value.nama_genre = current.nama_genre }
    else { router.push('/kelola-genre') }
  } catch (err) { alert('Gagal memuat data') }
  finally { loadingData.value = false }
})

const submitGenre = async () => {
  try {
    loading.value  = true
    errorMsg.value = ''
    await api.put(`/genre/${genreId}`, form.value)
    router.push('/kelola-genre')
  } catch (err) {
    errorMsg.value = err.response?.data?.message || 'Terjadi kesalahan.'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
:root {
  --primary: #4f46e5;
  --primary-hover: #3730a3;
  --danger: #ef4444;
  --danger-bg: #fef2f2;
  --bg-main: #f9fafb;
  --card-bg: #ffffff;
  --text-dark: #1f2937;
  --text-muted: #6b7280;
  --border-color: #d1d5db;
  --radius: 10px;
  --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.container {
  max-width: 550px;
  margin: 2rem auto;
  padding: 0 1.5rem;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: var(--text-dark);
}

.container h1 {
  font-size: 1.75rem;
  font-weight: 700;
  margin: 12px 0 24px;
}

/* Header & Back Link */
.btn-back {
  display: inline-block;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  transition: color 0.2s ease;
}

.btn-back:hover {
  color: var(--primary);
}

.loading-text {
  text-align: center;
  font-size: 1.1rem;
  color: var(--text-muted);
  padding: 3rem 0;
}

/* Card Form Wrapper */
.form-wrapper {
  background: var(--card-bg);
  padding: 2rem;
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  border: 1px solid #e5e7eb;
}

/* Form Controls */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1.25rem;
}

.form-group label {
  font-size: 0.9rem;
  font-weight: 600;
  color: #374151;
}

.required {
  color: var(--danger);
}

.form-input {
  width: 100%;
  padding: 0.75rem 0.9rem;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  font-size: 0.95rem;
  color: var(--text-dark);
  background-color: #ffffff;
  box-sizing: border-box;
  transition: all 0.2s ease;
}

.form-input:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.2);
}

/* Tombol Submit Warna Solid & Kontras */
.btn-submit {
  width: 100%;
  margin-top: 0.5rem;
  padding: 0.85rem;
  background-color: #4f46e5 !important;
  color: #ffffff !important;
  border: none !important;
  border-radius: var(--radius);
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 14px rgba(79, 70, 229, 0.4);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-submit:hover:not(:disabled) {
  background-color: #3730a3 !important;
  color: #ffffff !important;
  box-shadow: 0 6px 18px rgba(55, 48, 163, 0.5);
  transform: translateY(-2px);
}

.btn-submit:active:not(:disabled) {
  transform: translateY(0);
}

.btn-submit:disabled {
  background-color: #9ca3af !important;
  color: #ffffff !important;
  box-shadow: none;
  cursor: not-allowed;
  opacity: 0.7;
}

/* Feedback Error */
.error-msg {
  margin-top: 1rem;
  padding: 0.75rem;
  background-color: var(--danger-bg);
  color: var(--danger);
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 600;
  text-align: center;
  border: 1px solid #fca5a5;
}
</style>