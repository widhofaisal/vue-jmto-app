<template>
    <div class="card p-6">
        <form @submit.prevent="handleSubmit">
            <!-- Tanggal & Shift -->
            <div class="row mb-3">
                <div class="col-md-3">
                    <label class="form-label">Tanggal</label>
                    <input v-model="form.tanggal" type="date" class="form-control" required />
                </div>
                <div class="col-md-3">
                    <label class="form-label">Shift</label>
                    <select v-model="form.shift" class="form-select" required>
                        <option value="">-- Pilih Shift --</option>
                        <option v-for="shift in shifts" :key="shift.id" :value="shift.name">{{ shift.name }}</option>
                    </select>
                </div>
            </div>

            <!-- CS + Gardu + CSS in One Row -->
            <div v-for="i in Math.max(form.pasangan.length, form.css.length)" :key="i" class="row mb-2 align-items-end">
                <!-- CS -->
                <div class="col-md-3">
                    <label v-if="i === 1" class="form-label">Nama CS</label>
                    <select v-if="form.pasangan[i - 1]" v-model="form.pasangan[i - 1].cs" class="form-select">
                        <option value="">-- Pilih CS --</option>
                        <option v-for="cs in csList" :key="cs.id" :value="cs.name">{{ cs.name }}</option>
                    </select>
                </div>

                <!-- Gardu -->
                <div class="col-md-3">
                    <label v-if="i === 1" class="form-label">Gardu</label>
                    <select v-if="form.pasangan[i - 1]" v-model="form.pasangan[i - 1].gardu" class="form-select">
                        <option value="">-- Pilih Gardu --</option>
                        <option v-for="gardu in garduList" :key="gardu.id" :value="gardu.name">{{ gardu.name }}</option>
                    </select>
                </div>

                <!-- Tombol + Pasangan -->
                <div class="col-md-1">
                    <button v-if="i === form.pasangan.length" type="button" class="btn btn-sm btn-secondary"
                        @click="tambahPasangan">
                        ➕
                    </button>
                </div>

                <!-- CSS -->
                <div class="col-md-3">
                    <label v-if="i === 1" class="form-label">Nama CSS</label>
                    <select v-if="form.css[i - 1]" v-model="form.css[i - 1].css" class="form-select">
                        <option value="">-- Pilih CSS --</option>
                        <option v-for="css in cssList" :key="css.id" :value="css.name">{{ css.name }}</option>
                    </select>
                </div>

                <!-- Tombol + CSS -->
                <div class="col-md-1">
                    <button v-if="i === form.css.length" type="button" class="btn btn-sm btn-secondary"
                        @click="tambahCss">
                        ➕
                    </button>
                </div>
            </div>

            <!-- Tombol Submit -->
            <button type="submit" class="btn btn-primary mt-3">LOCK</button>
        </form>

        <!-- 📋 Tabel -->
        <div class="table-responsive mt-5">
            <table class="table table-bordered table-hover">
                <thead class="table-light">
                    <tr>
                        <th>#</th>
                        <th>Tanggal</th>
                        <th>Shift</th>
                        <th>Pasangan CS & Gardu</th>
                        <th>CSS</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in dataList" :key="index">
                        <td>{{ index + 1 }}</td>
                        <td>{{ item.tanggal }}</td>
                        <td>{{ item.shift }}</td>
                        <td>
                            <ul>
                                <li v-for="(pair, i) in item.pasangan" :key="i">{{ pair.cs }} - {{ pair.gardu }}</li>
                            </ul>
                        </td>
                        <td>
                            <ul>
                                <li v-for="(cssItem, j) in item.css" :key="j">CSS: {{ cssItem.css }}</li>
                            </ul>
                        </td>
                    </tr>
                    <tr v-if="dataList.length === 0">
                        <td colspan="5" class="text-center">Belum ada data</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'

const BASE_URL = import.meta.env.VITE_MOCK_API_URL

const form = reactive({
    tanggal: '',
    shift: '',
    pasangan: [{ cs: '', gardu: '' }],
    css: [{ css: '' }]
})

const dataList = ref([])
const shifts = ref([])
const csList = ref([])
const cssList = ref([])
const garduList = ref([])

onMounted(async () => {
    try {
        const [shiftRes, csRes, garduRes, cssRes] = await Promise.all([
            fetch(`${BASE_URL}/shifts`),
            fetch(`${BASE_URL}/cs`),
            fetch(`${BASE_URL}/gardu`),
            fetch(`${BASE_URL}/css`)
        ])
        shifts.value = await shiftRes.json()
        csList.value = await csRes.json()
        garduList.value = await garduRes.json()
        cssList.value = await cssRes.json()
    } catch (err) {
        console.error('Failed to fetch data', err)
    }
})

function tambahPasangan() {
    form.pasangan.push({ cs: '', gardu: '' })
}

function tambahCss() {
    form.css.push({ css: '' })
}

async function handleSubmit() {
    if (!form.tanggal || !form.shift || form.pasangan.length === 0) return
    try {
        const response = await fetch(`${BASE_URL}/shifts`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                tanggal: form.tanggal,
                shift: form.shift,
                pasangan: form.pasangan,
                css: form.css
            })
        })
        if (!response.ok) throw new Error(`HTTP ${response.status}`)
        const created = await response.json()
        dataList.value.push(created)

        form.tanggal = ''
        form.shift = ''
        form.pasangan = [{ cs: '', gardu: '' }]
        form.css = [{ css: '' }]
    } catch (err) {
        console.error('Failed to save data', err)
    }
}
</script>