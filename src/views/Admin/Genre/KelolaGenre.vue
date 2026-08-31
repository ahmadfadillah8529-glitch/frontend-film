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