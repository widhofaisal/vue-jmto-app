<template>
    <div class="card p-6">
        <!-- 🧾 Form Input -->
        <form @submit.prevent="handleSubmit">
            <div class="row mb-3">
                <div class="col-md-4">
                    <label class="form-label">Nama</label>
                    <input v-model="form.nama" type="text" class="form-control" required />
                </div>

                <!-- <div class="col-md-4">
            <label class="form-label">Umur</label>
            <input v-model.number="form.umur" type="number" class="form-control" required />
          </div> -->

                <div class="col-md-4">
                    <label class="form-label">Shift</label>
                    <select v-model="form.shift" class="form-select" required>
                        <option value="">-- Pilih Shift --</option>
                        <option v-for="shift in shifts" :key="shift.id" :value="shift.name">
                            {{ shift.name }}
                        </option>
                    </select>
                </div>
            </div>

            <button type="submit" class="btn btn-primary">Tambah</button>
        </form>

        <!-- 📋 Tabel Data -->
        <div class="table-responsive mt-5">
            <table class="table table-bordered table-hover">
                <thead class="table-light">
                    <tr>
                        <th>#</th>
                        <th>Nama</th>
                        <th>Shift</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in dataList" :key="index">
                        <td>{{ index + 1 }}</td>
                        <td>{{ item.nama }}</td>
                        <td>{{ item.shift }}</td>
                    </tr>
                    <tr v-if="dataList.length === 0">
                        <td colspan="4" class="text-center">Belum ada data</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
const BASE_URL = import.meta.env.VITE_MOCK_API_URL

// State form dan list data
const form = reactive({
    nama: '',
    shift: ''
})

const dataList = ref([])

const shifts = ref([])

onMounted(async () => {
    try {
        const response = await fetch(`${BASE_URL}/shifts`)
        const data = await response.json()
        shifts.value = data
    } catch (err) {
        console.error('Failed to fetch shifts', err)
    }
})

// Handle submit
async function handleSubmit() {
    if (!form.nama || !form.shift) return
    try {
        const response = await fetch(`${BASE_URL}/shifts`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                name: form.nama,
                shift: form.shift
            })
        })
        if (!response.ok) throw new Error(`HTTP ${response.status}`)
        const created = await response.json()
        dataList.value.push(created)
        // Reset form
        form.nama = ''
        form.shift = ''
    } catch (err) {
        console.error('Failed to save shift', err)
    }
}
</script>