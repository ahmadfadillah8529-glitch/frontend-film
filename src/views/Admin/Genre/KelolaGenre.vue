<template>
  <div class="container">
    <div class="page-title">
      <RouterLink to="/dashboard" class="btn-back">← Dashboard</RouterLink>
      <div class="title-row">
        <h1>🎭 Kelola Genre</h1>
        <RouterLink to="/tambah-genre" class="btn btn-primary">➕ Tambah Genre</RouterLink>
      </div>
    </div>

    <div v-if="successMsg" class="alert alert-success">✅ {{ successMsg }}</div>
    <p v-if="loading" class="loading-text">⏳ Memuat data genre...</p>

    <div v-else class="table-wrapper">
      <table class="film-table">
        <thead><tr><th>No</th><th>Nama Genre</th><th>Aksi</th></tr></thead>
        <tbody>
          <tr v-if="genres.length === 0">
            <td colspan="3" class="empty-row">Belum ada data genre.</td>
          </tr>
          <tr v-for="(genre, index) in genres" :key="genre.id">
            <td>{{ index + 1 }}</td>
            <td class="film-title-cell">{{ genre.nama_genre }}</td>
            <td>
              <div class="action-btns">
                <RouterLink :to="'/edit-genre/' + genre.id" class="btn-action btn-edit">✏️ Edit</RouterLink>
                <button @click="hapusGenre(genre.id, genre.nama_genre)"
                  :disabled="deletingId === genre.id" class="btn-action btn-delete">
                  <span v-if="deletingId === genre.id">⏳</span>
                  <span v-else>🗑️ Hapus</span>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal Hapus -->
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-box">
        <h3>⚠️ Konfirmasi Hapus</h3>
        <p>Yakin menghapus genre: <strong>{{ genreToDelete?.nama_genre }}</strong>?</p>
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

const router        = useRouter()
const genres        = ref([])
const loading       = ref(true)
const successMsg    = ref('')
const deletingId    = ref(null)
const showModal     = ref(false)
const genreToDelete = ref(null)

onMounted(async () => { await ambilGenre() })

const ambilGenre = async () => {
  try {
    loading.value = true
    const res = await api.get('/genre')
    genres.value = res.data.data
  } catch (err) { console.error(err) }
  finally { loading.value = false }
}

const hapusGenre = (id, nama_genre) => {
  genreToDelete.value = { id, nama_genre }
  showModal.value     = true
}

const konfirmasiHapus = async () => {
  const id = genreToDelete.value.id
  showModal.value = false
  try {
    deletingId.value = id
    await api.delete(`/genre/${id}`)
    genres.value = genres.value.filter(g => g.id !== id)
    successMsg.value = `Genre "${genreToDelete.value.nama_genre}" berhasil dihapus!`
    setTimeout(() => { successMsg.value = '' }, 3000)
  } catch (err) {
    alert('Gagal menghapus genre!')
  } finally {
    deletingId.value = null
    genreToDelete.value = null
  }
}
</script>

<style scoped>
:root {
  --primary: #4f46e5;
  --primary-hover: #3730a3;
  --danger: #ef4444;
  --danger-hover: #dc2626;
  --success-bg: #d1fae5;
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
  max-width: 900px;
  margin: 2rem auto;
  padding: 0 1.5rem;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: var(--text-dark);
}

.page-title {
  margin-bottom: 1.5rem;
}

.btn-back {
  display: inline-block;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 600;
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

/* FIX TOMBOL TAMBAH GENRE BIAR KETARA BANGET */
.btn-primary {
  background-color: #4f46e5 !important;
  color: #ffffff !important;
  padding: 0.65rem 1.25rem;
  border-radius: var(--radius);
  font-weight: 700;
  font-size: 0.95rem;
  text-decoration: none !important;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  border: none;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.35);
  transition: all 0.2s ease;
}

.btn-primary:hover {
  background-color: #3730a3 !important;
  color: #ffffff !important;
  box-shadow: 0 6px 16px rgba(55, 48, 163, 0.5);
  transform: translateY(-2px);
}

.alert-success {
  background-color: var(--success-bg);
  color: var(--success-text);
  padding: 1rem;
  border-radius: var(--radius);
  margin-bottom: 1.5rem;
  font-weight: 600;
  border: 1px solid #a7f3d0;
}

.loading-text {
  text-align: center;
  font-size: 1.1rem;
  color: var(--text-muted);
  padding: 3rem 0;
}

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

.film-table th:nth-child(1) { width: 80px; }
.film-table th:nth-child(3) { width: 180px; }

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
  padding: 2.5rem !important;
}

.action-btns {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.btn-action {
  padding: 0.45rem 0.85rem;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 600;
  text-decoration: none;
  cursor: pointer;
  border: 1px solid transparent;
  transition: all 0.2s ease;
}

.btn-edit {
  background-color: #f3f4f6;
  color: #374151 !important;
  border-color: #d1d5db;
}

.btn-edit:hover {
  background-color: #e5e7eb;
}

.btn-delete {
  background-color: #fee2e2;
  color: var(--danger) !important;
  border-color: #fca5a5;
}

.btn-delete:hover:not(:disabled) {
  background-color: #fecaca;
}

.btn-delete:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

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
  font-weight: 600;
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
  font-weight: 600;
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
  color: white !important;
  font-weight: 600;
  cursor: pointer;
}

.btn-modal-delete:hover {
  background-color: var(--danger-hover);
}
</style>