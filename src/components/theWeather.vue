```vue
<script setup>
import { ref, onMounted } from 'vue'

// =========================
// STATE
// =========================
const locationInput = ref('Makassar')
const locationName = ref('Makassar')
const currentTime = ref('--:--')

const temperature = ref('--')
const windSpeed = ref('--')
const rainChance = ref('--')
const currentIcon = ref('☀️')
const weatherDescription = ref('Memuat cuaca...')

const previousHours = ref([])
const futureHours = ref([])

const statusMessage = ref('')
const statusError = ref(false)
const isLoading = ref(false)


// =========================
// WEATHER CODE
// =========================
const codes = {
    0: ['☀️', 'Cerah'],
    1: ['🌤️', 'Cerah Berawan'],
    2: ['⛅', 'Berawan Sebagian'],
    3: ['☁️', 'Berawan'],

    45: ['🌫️', 'Berkabut'],
    48: ['🌫️', 'Kabut Tebal'],

    51: ['🌦️', 'Gerimis Ringan'],
    53: ['🌦️', 'Gerimis'],
    55: ['🌧️', 'Gerimis Lebat'],

    61: ['🌧️', 'Hujan Ringan'],
    63: ['🌧️', 'Hujan'],
    65: ['🌧️', 'Hujan Lebat'],

    71: ['🌨️', 'Salju Ringan'],
    73: ['🌨️', 'Salju'],
    75: ['❄️', 'Salju Lebat'],

    80: ['🌦️', 'Hujan Singkat'],
    81: ['🌦️', 'Hujan'],
    82: ['⛈️', 'Hujan Lebat'],

    95: ['⛈️', 'Badai Petir'],
    96: ['⛈️', 'Petir + Hujan Es'],
    99: ['⛈️', 'Badai Petir Lebat']
}


// =========================
// WEATHER INFO
// =========================
function getWeatherInfo(code) {
    return codes[code] || ['🌤️', 'Tidak diketahui']
}


// =========================
// FORMAT JAM
// =========================
function formatHour(time) {
    return new Date(time).toLocaleTimeString('id-ID', {
        hour: '2-digit',
        minute: '2-digit',
        hour12: false
    })
}


// =========================
// STATUS
// =========================
function setStatus(message, error = false) {
    statusMessage.value = message
    statusError.value = error
}


// =========================
// SEARCH LOCATION
// =========================
async function searchLocation(query) {
    const url =
        'https://geocoding-api.open-meteo.com/v1/search?' +
        `name=${encodeURIComponent(query)}` +
        '&count=1' +
        '&language=id' +
        '&format=json'

    const response = await fetch(url)

    if (!response.ok) {
        throw new Error('Gagal mencari lokasi.')
    }

    const data = await response.json()

    if (!data.results || data.results.length === 0) {
        throw new Error('Lokasi tidak ditemukan.')
    }

    return data.results[0]
}


// =========================
// GET FORECAST
// =========================
async function getForecast(latitude, longitude) {
    const params = new URLSearchParams({
        latitude,
        longitude,
        timezone: 'auto',
        forecast_days: '2',
        past_hours: '24',
        hourly:
            'temperature_2m,precipitation_probability,weather_code,wind_speed_10m'
    })

    const response = await fetch(
        `https://api.open-meteo.com/v1/forecast?${params}`
    )

    if (!response.ok) {
        throw new Error('Gagal mengambil data cuaca.')
    }

    return await response.json()
}


// =========================
// LOAD WEATHER
// =========================
async function loadWeather(query) {
    try {
        isLoading.value = true
        setStatus('Memuat data cuaca...')

        // Cari lokasi
        const location = await searchLocation(query)

        // Ambil data cuaca
        const data = await getForecast(
            location.latitude,
            location.longitude
        )

        const hourly = data.hourly

        const times = hourly.time
        const temperatures = hourly.temperature_2m
        const rain = hourly.precipitation_probability
        const weatherCodes = hourly.weather_code
        const winds = hourly.wind_speed_10m


        // =========================
        // CARI JAM SAAT INI
        // =========================

        const now = Date.now()

        let currentIndex = 0
        let smallestDifference = Infinity

        times.forEach((time, index) => {
            const difference =
                Math.abs(new Date(time).getTime() - now)

            if (difference < smallestDifference) {
                smallestDifference = difference
                currentIndex = index
            }
        })


        // =========================
        // CURRENT WEATHER
        // =========================

        const [icon, description] =
            getWeatherInfo(weatherCodes[currentIndex])

        locationName.value =
            location.name +
            (location.country_code
                ? `, ${location.country_code}`
                : '')

        currentTime.value =
            formatHour(times[currentIndex])

        currentIcon.value = icon

        temperature.value =
            Math.round(temperatures[currentIndex])

        weatherDescription.value =
            description

        windSpeed.value =
            Math.round(winds[currentIndex] ?? 0)

        rainChance.value =
            rain[currentIndex] ?? 0


        // =========================
        // PREVIOUS 24 HOURS
        // =========================

        const previous = []

        for (
            let i = currentIndex - 24;
            i < currentIndex;
            i++
        ) {
            if (i >= 0) {
                previous.push({
                    time: formatHour(times[i]),
                    temperature: Math.round(temperatures[i]),
                    rain: rain[i] ?? 0,
                    wind: Math.round(winds[i] ?? 0),
                    icon: getWeatherInfo(weatherCodes[i])[0],
                    description: getWeatherInfo(weatherCodes[i])[1]
                })
            }
        }

        previousHours.value = previous


        // =========================
        // NEXT 24 HOURS
        // =========================

        const future = []

        for (
            let i = currentIndex + 1;
            i <= currentIndex + 24 && i < times.length;
            i++
        ) {
            future.push({
                time: formatHour(times[i]),
                temperature: Math.round(temperatures[i]),
                rain: rain[i] ?? 0,
                wind: Math.round(winds[i] ?? 0),
                icon: getWeatherInfo(weatherCodes[i])[0],
                description: getWeatherInfo(weatherCodes[i])[1]
            })
        }

        futureHours.value = future


        setStatus(
            `Cuaca diperbarui untuk ${location.name}.`
        )

    } catch (error) {

        console.error(error)

        setStatus(
            error.message || 'Terjadi kesalahan.',
            true
        )

    } finally {

        isLoading.value = false
    }
}


// =========================
// SEARCH
// =========================
function handleSearch() {

    const query = locationInput.value.trim()

    if (!query) {
        setStatus(
            'Masukkan lokasi terlebih dahulu.',
            true
        )

        return
    }

    loadWeather(query)
}


// =========================
// REFRESH
// =========================
function refreshWeather() {

    const query =
        locationInput.value.trim() || 'Makassar'

    loadWeather(query)
}


// =========================
// LOAD AWAL
// =========================
onMounted(() => {
    loadWeather('Makassar')
})
</script>


<template>

    <main class="min-h-screen bg-sky-50 px-4 py-5 sm:px-6 lg:px-8">

        <div class="mx-auto max-w-6xl">

            <!-- HEADER -->
            <header class="mb-6 flex items-center justify-between">

                <div class="flex items-center gap-3">

                    <div
                        class="flex h-10 w-10 items-center justify-center rounded-2xl bg-sky-500 text-xl shadow-lg shadow-sky-200">
                        ☁️
                    </div>

                    <div>
                        <h1 class="text-xl font-bold text-slate-800">
                            SkyCast
                        </h1>

                        <p class="text-xs text-slate-500">
                            Simple Weather
                        </p>
                    </div>

                </div>


                <!-- REFRESH -->

                <button @click="refreshWeather" :disabled="isLoading"
                    class="flex h-10 w-10 items-center justify-center rounded-xl border border-sky-100 bg-white text-sky-600 shadow-sm transition hover:bg-sky-50 active:scale-95 disabled:opacity-50">

                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" :class="{ 'animate-spin': isLoading }"
                        fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">

                        <path stroke-linecap="round" stroke-linejoin="round"
                            d="M4 4v5h5M20 20v-5h-5M5.5 9A7.5 7.5 0 0118 6.5L20 9M18.5 15A7.5 7.5 0 016 17.5L4 15" />

                    </svg>

                </button>

            </header>


            <!-- SEARCH -->

            <section class="mb-5">

                <form @submit.prevent="handleSearch" class="flex flex-row gap-2">

                    <input v-model="locationInput" type="text" placeholder="Cari kota atau lokasi..."
                        class="h-12 min-w-0 flex-1 rounded-2xl border border-sky-100 bg-white px-4 text-sm leading-none outline-none transition focus:border-sky-400 focus:ring-4 focus:ring-sky-100" />

                    <button type="submit" :disabled="isLoading"
                        class="h-12 shrink-0 rounded-2xl bg-sky-500 px-5 font-semibold leading-none text-white shadow-lg shadow-sky-200 transition hover:bg-sky-600 disabled:opacity-50">
                        Cari
                    </button>

                </form>


                <!-- STATUS -->

                <p v-if="statusMessage" class="mt-2 text-sm" :class="statusError
                    ? 'text-red-500'
                    : 'text-slate-500'
                    ">
                    {{ statusMessage }}
                </p>

            </section>


            <!-- CURRENT WEATHER -->

            <section
                class="overflow-hidden rounded-3xl bg-linear-to-br from-sky-400 via-sky-500 to-cyan-500 p-5 text-white shadow-xl shadow-sky-100 sm:p-7">

                <div class="flex flex-col gap-7 sm:flex-row sm:items-center sm:justify-between">

                    <div>

                        <div class="mb-2 flex items-center gap-2 text-sm text-white/80">

                            <span>
                                {{ locationName }}
                            </span>

                            <span>•</span>

                            <span>
                                {{ currentTime }}
                            </span>

                        </div>


                        <div class="flex items-center gap-4">

                            <span class="text-6xl">
                                {{ currentIcon }}
                            </span>

                            <div>

                                <div class="flex items-start">

                                    <span class="text-6xl font-bold tracking-tight">
                                        {{ temperature }}
                                    </span>

                                    <span class="mt-2 text-2xl">
                                        °C
                                    </span>

                                </div>

                                <p class="mt-1 font-medium text-white/90">
                                    {{ weatherDescription }}
                                </p>

                            </div>

                        </div>

                    </div>


                    <!-- WEATHER DETAILS -->

                    <div class="grid grid-cols-2 gap-3 sm:min-w-[280px]">

                        <div class="rounded-2xl bg-white/15 p-4 backdrop-blur-sm">

                            <p class="text-xs text-white/70">
                                Angin
                            </p>

                            <p class="mt-1 text-lg font-semibold">
                                {{ windSpeed }} km/h
                            </p>

                        </div>


                        <div class="rounded-2xl bg-white/15 p-4 backdrop-blur-sm">

                            <p class="text-xs text-white/70">
                                Peluang hujan
                            </p>

                            <p class="mt-1 text-lg font-semibold">
                                {{ rainChance }}%
                            </p>

                        </div>

                    </div>

                </div>

            </section>


            <!-- PREVIOUS 24 HOURS -->

            <section class="mt-5 rounded-3xl border border-sky-100 bg-white p-5 shadow-sm sm:p-6">

                <div class="mb-4">

                    <p class="text-xs font-semibold uppercase tracking-wider text-sky-500">
                        History
                    </p>

                    <h2 class="mt-1 text-lg font-bold">
                        24 Jam Sebelumnya
                    </h2>

                </div>


                <div class="flex gap-3 overflow-x-auto pb-2">

                    <article v-for="weather in previousHours" :key="weather.time" :title="weather.description"
                        class="min-w-[100px] rounded-2xl border border-sky-50 bg-sky-50/70 p-3 text-center">

                        <p class="text-xs font-medium text-slate-500">
                            {{ weather.time }}
                        </p>

                        <div class="my-2 text-2xl">
                            {{ weather.icon }}
                        </div>

                        <p class="text-sm font-bold text-slate-700">
                            {{ weather.temperature }}°
                        </p>

                        <p class="mt-1 text-[11px] text-sky-600">
                            {{ weather.rain }}% hujan
                        </p>

                        <p class="mt-1 text-[10px] text-slate-400">
                            {{ weather.wind }} km/h
                        </p>

                    </article>

                </div>

            </section>


            <!-- NEXT 24 HOURS -->

            <section class="mt-5 rounded-3xl border border-sky-100 bg-white p-5 shadow-sm sm:p-6">

                <div class="mb-4">

                    <p class="text-xs font-semibold uppercase tracking-wider text-sky-500">
                        Forecast
                    </p>

                    <h2 class="mt-1 text-lg font-bold">
                        24 Jam Mendatang
                    </h2>

                </div>


                <div class="flex gap-3 overflow-x-auto pb-2">

                    <article v-for="weather in futureHours" :key="weather.time" :title="weather.description"
                        class="min-w-[100px] rounded-2xl border border-sky-50 bg-sky-50/70 p-3 text-center">

                        <p class="text-xs font-medium text-slate-500">
                            {{ weather.time }}
                        </p>

                        <div class="my-2 text-2xl">
                            {{ weather.icon }}
                        </div>

                        <p class="text-sm font-bold text-slate-700">
                            {{ weather.temperature }}°
                        </p>

                        <p class="mt-1 text-[11px] text-sky-600">
                            {{ weather.rain }}% hujan
                        </p>

                        <p class="mt-1 text-[10px] text-slate-400">
                            {{ weather.wind }} km/h
                        </p>

                    </article>

                </div>

            </section>


            <!-- FOOTER -->

            <footer class="py-7 text-center text-xs text-slate-400">
                Data cuaca dari Open-Meteo • SkyCast
            </footer>

        </div>

    </main>

</template>
