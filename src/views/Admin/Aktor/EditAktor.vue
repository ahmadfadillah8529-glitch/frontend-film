<template>
  <div class="container">
    <RouterLink to="/kelola-aktor" class="btn-back">← Kembali</RouterLink>
    <h1 style="margin: 12px 0 24px">✏️ Edit Aktor</h1>

    <p v-if="loadingData" class="loading-text">⏳ Memuat data...</p>

    <div v-else class="form-wrapper">
      <form @submit.prevent="submitAktor">
        <div class="form-group">
          <label>Nama Aktor <span class="required">*</span></label>
          <input v-model="form.nama_aktor" type="text" required class="form-input" />
        </div>

        <div class="form-group">
          <label>Gender <span class="required">*</span></label>
          <select v-model="form.jenis_kelamin" required class="form-input">
            <option value="" disabled>Pilih Gender</option>
            <option value="laki-laki">Laki-laki</option>
            <option value="perempuan">Perempuan</option>
          </select>
        </div>

        <div class="form-group">
          <label>Umur <span class="required">*</span></label>
          <input v-model="form.umur" type="text" required class="form-input" />
        </div>

        <div class="form-group">
          <label>🖼️ URL Foto <span class="required">*</span></label>
          <input v-model="form.foto" type="text" required class="form-input" placeholder="https://example.com/foto.jpg" />
          <img v-if="form.foto" :src="form.foto" alt="Preview" class="poster-preview" />
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
const aktorId = route.params.id

const form        = ref({ nama_aktor: '', jenis_kelamin: '', umur: '' })
const loadingData = ref(true)
const loading     = ref(false)
const errorMsg    = ref('')

onMounted(async () => {
  try {
    const res     = await api.get('/aktor')
    const current = res.data.data.find(a => a.id == aktorId)
    if (current) {
      form.value.nama_aktor    = current.nama_aktor
      form.value.jenis_kelamin        = current.jenis_kelamin
      form.value.umur = current.umur
      form.value.foto = current.foto
    } else { router.push('/kelola-aktor') }
  } catch (err) { alert('Gagal memuat data') }
  finally { loadingData.value = false }
})

const submitAktor = async () => {
  try {
    loading.value  = true
    errorMsg.value = ''
    await api.put(`/aktor/${aktorId}`, form.value)
    router.push('/kelola-aktor')
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
  --primary-hover: #4338ca;
  --danger: #ef4444;
  --danger-bg: #fef2f2;
  --bg-main: #f9fafb;
  --card-bg: #ffffff;
  --text-dark: #1f2937;
  --text-muted: #6b7280;
  --border-color: #e5e7eb;
  --radius: 10px;
  --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.container {
  max-width: 650px;
  margin: 2rem auto;
  padding: 0 1.5rem;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: var(--text-dark);
}

/* Header & Back Button */
.btn-back {
  display: inline-block;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 500;
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
  border: 1px solid var(--border-color);
}

/* Form Group Controls */
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
  padding: 0.65rem 0.85rem;
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
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.15);
}

/* Image Preview Handling */
.poster-preview {
  margin-top: 0.75rem;
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: var(--radius);
  border: 2px solid var(--border-color);
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

/* Actions & Feedback */
.btn-submit {
  width: 100%;
  margin-top: 0.5rem;
  padding: 0.75rem;
  color: #f43d3d;
  border: none;
  border-radius: var(--radius);
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-submit:hover:not(:disabled) {
  background-color: var(--primary-hover);
  transform: translateY(-1px);
}

.btn-submit:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.error-msg {
  margin-top: 1rem;
  padding: 0.75rem;
  background-color: var(--danger-bg);
  color: var(--danger);
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
  text-align: center;
  border: 1px solid #fca5a5;
}
</style>