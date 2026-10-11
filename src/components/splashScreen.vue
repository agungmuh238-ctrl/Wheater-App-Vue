<template>

    <body class="min-h-screen overflow-hidden bg-linear-to-b from-sky-400 via-sky-500 to-blue-600">

        <main class="relative flex min-h-screen items-center justify-center px-6">
            <!-- Decorative background -->
            <div class="absolute -left-20 top-20 h-40 w-40 rounded-full bg-white/10 blur-3xl"></div>
            <div class="absolute -right-20 bottom-20 h-56 w-56 rounded-full bg-cyan-300/10 blur-3xl"></div>

            <section class="relative flex w-full max-w-sm flex-col items-center text-center">

                <!-- Cloud illustration -->
                <div class="relative mb-8 h-40 w-40">
                    <div class="glow absolute inset-2 rounded-full bg-white/30 blur-2xl"></div>

                    <div class="cloud-float relative flex h-full w-full items-center justify-center">
                        <svg viewBox="0 0 200 150" class="h-36 w-36 drop-shadow-[0_12px_18px_rgba(0,70,130,0.18)]"
                            aria-label="Ilustrasi awan" role="img">
                            <defs>
                                <linearGradient id="cloudGradient" x1="0" y1="0" x2="0" y2="1">
                                    <stop offset="0%" stop-color="#ffffff" />
                                    <stop offset="100%" stop-color="#dff6ff" />
                                </linearGradient>
                            </defs>

                            <path
                                d="M42 111h113c18 0 32-13 32-30s-14-30-32-30c-3 0-6 0-9 1C140 31 123 18 101 18c-27 0-49 19-54 44C23 64 8 78 8 96c0 8 3 15 8 20 6 6 15 9 26 9z"
                                fill="url(#cloudGradient)" />

                            <!-- small rain drops -->
                            <path d="M65 125l-6 12" stroke="#bdeeff" stroke-width="5" stroke-linecap="round" />
                            <path d="M101 125l-6 12" stroke="#bdeeff" stroke-width="5" stroke-linecap="round" />
                            <path d="M137 125l-6 12" stroke="#bdeeff" stroke-width="5" stroke-linecap="round" />
                        </svg>
                    </div>
                </div>

                <!-- App name -->
                <div class="fade-up">
                    <h1 class="text-3xl font-bold tracking-tight text-white">Weather App</h1>
                    <p class="mt-2 text-sm font-medium text-white/75">
                        Pantau Cuaca, Nikmati Hari yang Indah.
                    </p>
                </div>

                <!-- Loading -->
                <div class="fade-up mt-10 w-36 overflow-hidden rounded-full bg-white/20" style="animation-delay:.2s">
                    <div class="loading-bar h-1.5 w-1/2 rounded-full bg-white"></div>
                </div>

                <p class="fade-up mt-3 text-xs text-white/60" style="animation-delay:.35s">
                    Memuat data cuaca...
                </p>
            </section>
        </main>

    </body>
</template>

<style scoped>
@keyframes floatCloud {

    0%,
    100% {
        transform: translateY(0) translateX(0);
    }

    50% {
        transform: translateY(-10px) translateX(4px);
    }
}

@keyframes pulseGlow {

    0%,
    100% {
        transform: scale(1);
        opacity: .35;
    }

    50% {
        transform: scale(1.12);
        opacity: .55;
    }
}

@keyframes fadeUp {
    from {
        opacity: 0;
        transform: translateY(14px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes loading {
    0% {
        transform: translateX(-100%);
    }

    100% {
        transform: translateX(250%);
    }
}

.cloud-float {
    animation: floatCloud 3.5s ease-in-out infinite;
}

.glow {
    animation: pulseGlow 3s ease-in-out infinite;
}

.fade-up {
    animation: fadeUp .8s ease-out both;
}

.loading-bar {
    animation: loading 1.7s ease-in-out infinite;
}

@media (prefers-reduced-motion: reduce) {

    .cloud-float,
    .glow,
    .fade-up,
    .loading-bar {
        animation: none;
    }
}
</style>