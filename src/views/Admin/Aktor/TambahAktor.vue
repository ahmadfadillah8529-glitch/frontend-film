<template>
  <div class="container">
    <RouterLink to="/kelola-aktor" class="btn-back">← Kembali</RouterLink>
    <h1 style="margin: 12px 0 24px">➕ Tambah Aktor Baru</h1>

    <div class="form-wrapper">
      <form @submit.prevent="submitAktor">
        <div class="form-group">
          <label>Nama Aktor <span class="required">*</span></label>
          <input v-model="form.nama_aktor" type="text" placeholder="Contoh: Reza Rahadian" required class="form-input" />
        </div>

        <div class="form-group">
          <label>Gender <span class="required">*</span></label>
          <!-- Perhatikan: value pakai 'L' dan 'P' sesuai database ENUM -->
          <select v-model="form.jenis_kelamin" required class="form-input">
            <option value="" disabled>Pilih Gender</option>
            <option value="laki-laki">Laki-laki</option>
            <option value="perempuan">Perempuan</option>
          </select>
        </div>

        <div class="form-group">
          <label>Umur <span class="required">*</span></label>
          <input v-model="form.umur" type="text" placeholder="Contoh: 30" required class="form-input" />
        </div>

        <div class="form-group">
          <label>🖼️ URL Foto <span class="required">*</span></label>
          <input v-model="form.foto" type="text" placeholder="https://..." required class="form-input" />
          <!-- Preview gambar otomatis saat URL diisi -->
          <img v-if="form.foto" :src="form.foto" alt="Preview Poster" class="poster-preview" />
        </div>

        <button type="submit" :disabled="loading" class="btn-submit">
          <span v-if="loading">⏳ Menyimpan...</span>
          <span v-else>💾 Simpan Aktor</span>
        </button>
        <p v-if="errorMsg" class="error-msg">{{ errorMsg }}</p>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref }               from 'vue'
import { useRouter, RouterLink } from 'vue-router'
import api                   from '../../../utils/api'

const router = useRouter()
const form   = ref({ nama_aktor: '', jenis_kelamin: '', umur: '', foto: '' })
const loading  = ref(false)
const errorMsg = ref('')

const submitAktor = async () => {
  try {
    loading.value  = true
    errorMsg.value = ''
    await api.post('/aktor', form.value)
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
  max-width: 650px;
  margin: 2rem auto;
  padding: 0 1.5rem;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: var(--text-dark);
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

/* Image Preview */
.poster-preview {
  margin-top: 0.75rem;
  width: 110px;
  height: 110px;
  object-fit: cover;
  border-radius: var(--radius);
  border: 2px solid var(--primary);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

/* FIX WARNA BUTTON: Kontras Tegas & Clear Hover */
.btn-submit {
  width: 100%;
  margin-top: 1rem;
  padding: 0.85rem;
  background-color: #7670e9; /* Primary Solid */
  color: #ffffff !important;
  border: none;
  border-radius: var(--radius);
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.3);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-submit:hover:not(:disabled) {
  background-color: #3730a3; /* Darker Indigo pas Hover */
  box-shadow: 0 6px 16px rgba(55, 48, 163, 0.4);
  transform: translateY(-2px);
}

.btn-submit:active:not(:disabled) {
  transform: translateY(0);
}

.btn-submit:disabled {
  background-color: #9ca3af;
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