<template>
  <div class="container">
    <div class="page-title">
      <RouterLink to="/dashboard" class="btn-back">← Dashboard</RouterLink>
      <div class="title-row">
        <h1>🌟 Kelola Aktor</h1>
        <RouterLink to="/tambah-aktor" class="btn btn-primary">➕ Tambah Aktor</RouterLink>
      </div>
    </div>

    <div v-if="successMsg" class="alert alert-success">✅ {{ successMsg }}</div>
    <p v-if="loading" class="loading-text">⏳ Memuat data aktor...</p>

    <div v-else class="table-wrapper">
      <table class="film-table">
        <thead>
          <tr><th>No</th><th>Nama Aktor</th><th>Gender</th><th>Umur</th><th>Foto</th><th>Aksi</th></tr>
        </thead>
        <tbody>
          <tr v-if="aktors.length === 0">
            <td colspan="6" class="empty-row">Belum ada data aktor.</td>
          </tr>
          <tr v-for="(aktor, index) in aktors" :key="aktor.id">
            <td>{{ index + 1 }}</td>
            <td class="film-title-cell">{{ aktor.nama_aktor }}</td>
            <!-- Tampilkan teks lengkap, bukan kode L/P -->
            <td>{{ aktor.jenis_kelamin === 'laki-laki' ? 'Laki-laki' : 'Perempuan' }}</td>
            <td>{{ aktor.umur }}</td>
            <td><img :src="aktor.foto" :alt="aktor.nama_aktor" class="table-foto" /></td>
            <td>
              <div class="action-btns">
                <RouterLink :to="'/edit-aktor/' + aktor.id" class="btn-action btn-edit">✏️ Edit</RouterLink>
                <button @click="hapusAktor(aktor.id, aktor.nama_aktor)"
                  :disabled="deletingId === aktor.id" class="btn-action btn-delete">
                  <span v-if="deletingId === aktor.id">⏳</span>
                  <span v-else>🗑️ Hapus</span>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-box">
        <h3>⚠️ Konfirmasi Hapus</h3>
        <p>Yakin menghapus aktor: <strong>{{ aktorToDelete?.nama_aktor }}</strong>?</p>
        <p class="modal-warning">Tindakan ini tidak bisa dibatalkan!</p>
        <div class="modal-actions">
          <button @click="showModal = false" class="btn-modal-cancel">Batal</button>
          <button @click="konfirmasiHapus" class="btn-modal-delete">🗑️ Hapus</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted }        from 'vue'
import { RouterLink, useRouter } from 'vue-router'
import api                       from '../../../utils/api'

const router       = useRouter()
const aktors       = ref([])
const loading      = ref(true)
const successMsg   = ref('')
const deletingId   = ref(null)
const showModal    = ref(false)
const aktorToDelete = ref(null)

onMounted(async () => { await ambilAktor() })

const ambilAktor = async () => {
  try {
    loading.value = true
    const res = await api.get('/aktor')
    aktors.value = res.data.data
  } catch (err) { console.error(err) }
  finally { loading.value = false }
}

const hapusAktor = (id, nama_aktor) => {
  aktorToDelete.value = { id, nama_aktor }
  showModal.value     = true
}

const konfirmasiHapus = async () => {
  const id = aktorToDelete.value.id
  showModal.value = false
  try {
    deletingId.value = id
    await api.delete(`/aktor/${id}`)
    aktors.value = aktors.value.filter(a => a.id !== id)
    successMsg.value = `Aktor "${aktorToDelete.value.nama_aktor}" berhasil dihapus!`
    setTimeout(() => { successMsg.value = '' }, 3000)
  } catch (err) { alert('Gagal menghapus aktor!') }
  finally { deletingId.value = null; aktorToDelete.value = null }
}
</script>

<style scoped>
/* Reset & Dynamic Theme Variables */
:root {
  --primary: #4f46e5;
  --primary-hover: #4338ca;
  --danger: #ef4444;
  --danger-hover: #c54343;
  --success-bg: #79edb1;
  --success-text: #065f46;
  --bg-main: #f9fafb;
  --card-bg: #ffffff;
  --text-dark: #1f2937;
  --text-muted: #6b7280;
  --border-color: #e5e7eb;
  --radius: 10px;
  --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.container {
  max-width: 1100px;
  margin: 2rem auto;
  padding: 0 1.5rem;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: var(--text-dark);
}

/* Page Header */
.page-title {
  margin-bottom: 1.5rem;
}

.btn-back {
  display: inline-block;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 500;
  margin-bottom: 0.75rem;
  transition: color 0.2s ease;
}

.btn-back:hover {
  color: var(--primary);
}

.title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title-row h1 {
  font-size: 1.75rem;
  font-weight: 700;
  margin: 0;
  color: var(--text-dark);
}

/* Buttons */
.btn {
  padding: 0.6rem 1.2rem;
  border-radius: var(--radius);
  font-weight: 600;
  text-decoration: none;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  transition: all 0.2s ease;
  cursor: pointer;
  border: none;
}

.btn-primary {
  color: #312f2f;
}

.btn-primary:hover {
  background-color: var(--primary-hover);
  transform: translateY(-2px);
}

/* Alert Notification */
.alert-success {
  background-color: var(--success-bg);
  color: var(--success-text);
  padding: 1rem;
  border-radius: var(--radius);
  margin-bottom: 1.5rem;
  font-weight: 500;
  border: 1px solid #a7f3d0;
}

.loading-text {
  text-align: center;
  font-size: 1.1rem;
  color: var(--text-muted);
  padding: 3rem 0;
}

/* Table Styling */
.table-wrapper {
  background: var(--card-bg);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  overflow-x: auto;
  border: 1px solid var(--border-color);
}

.film-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

.film-table th {
  background-color: #f3f4f6;
  padding: 1rem;
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--text-muted);
  border-bottom: 1px solid var(--border-color);
}

.film-table td {
  padding: 1rem;
  border-bottom: 1px solid var(--border-color);
  vertical-align: middle;
  font-size: 0.95rem;
}

.film-table tbody tr:last-child td {
  border-bottom: none;
}

.film-table tbody tr:hover {
  background-color: #f9fafb;
}

.film-title-cell {
  font-weight: 600;
}

.empty-row {
  text-align: center;
  color: var(--text-muted);
  padding: 2rem !important;
}

.table-foto {
  width: 50px;
  height: 50px;
  object-fit: cover;
  border-radius: 50%;
  border: 2px solid var(--border-color);
}

/* Action Buttons Inside Table */
.action-btns {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.btn-action {
  padding: 0.4rem 0.8rem;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  border: 1px solid transparent;
  transition: all 0.2s ease;
}

.btn-edit {
  background-color: #f3f4f6;
  color: #374151;
  border-color: #d1d5db;
}

.btn-edit:hover {
  background-color: #e5e7eb;
}

.btn-delete {
  background-color: #fee2e2;
  color: var(--danger);
  border-color: #fca5a5;
}

.btn-delete:hover:not(:disabled) {
  background-color: #fecaca;
}

.btn-delete:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Modal Confirmation */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-box {
  background: var(--card-bg);
  padding: 1.75rem;
  border-radius: 12px;
  width: 90%;
  max-width: 420px;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
}

.modal-box h3 {
  margin-top: 0;
  margin-bottom: 0.75rem;
  font-size: 1.25rem;
  color: var(--text-dark);
}

.modal-box p {
  margin: 0.5rem 0;
  color: #4b5563;
}

.modal-warning {
  font-size: 0.85rem;
  color: var(--danger);
  font-weight: 500;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-top: 1.5rem;
}

.btn-modal-cancel {
  padding: 0.5rem 1rem;
  border-radius: 6px;
  border: 1px solid var(--border-color);
  background: white;
  color: var(--text-dark);
  font-weight: 500;
  cursor: pointer;
}

.btn-modal-cancel:hover {
  background-color: #f3f4f6;
}

.btn-modal-delete {
  padding: 0.5rem 1rem;
  border-radius: 6px;
  border: none;
  background-color: var(--danger);
  color: white;
  font-weight: 600;
  cursor: pointer;
}

.btn-modal-delete:hover {
  background-color: var(--danger-hover);
}
</style>