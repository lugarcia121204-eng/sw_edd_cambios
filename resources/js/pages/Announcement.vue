<script setup>
import { Head, Link } from '@inertiajs/vue3';
import LandingLayout from '@/layouts/LandingLayout.vue';
import { ref, computed } from 'vue';
import { Users, ClipboardCheck, CalendarDays, FileText, ArrowRight, ArrowUpRight, Clock3 } from 'lucide-vue-next';

const props = defineProps({
    announcement: Object, // Changed from convocatoria to match web.php
});

const showModal = ref(false);
const currentPdfUrl = ref('');
const currentPdfTitle = ref('');

const openPdfModal = (url, title) => {
    if (!url) return;
    currentPdfUrl.value = url;
    currentPdfTitle.value = title;
    showModal.value = true;
};

const closeModal = () => {
    showModal.value = false;
    currentPdfUrl.value = '';
    currentPdfTitle.value = '';
};

const isPdfResource = (url) => /\.pdf($|\?)/i.test(url || '');
const isImageResource = (url) => /\.(png|jpe?g|gif|webp|bmp|svg)($|\?)/i.test(url || '');

const currentPreviewIsPdf = computed(() => isPdfResource(currentPdfUrl.value));
const currentPreviewIsImage = computed(() => isImageResource(currentPdfUrl.value));

// Acceder a los datos correctamente (handling Resource wrapper)
const announcementData = computed(() => {
    return props.announcement?.data || props.announcement;
});

// Helper para parsear fecha "YYYY-MM-DD", "YYYY-MM-DD HH:MM:SS" o ISO "YYYY-MM-DDTHH:mm:ss"
const parseDateLocal = (dateString) => {
    if (!dateString) return null;
    // Tomamos los primeros 10 caracteres (YYYY-MM-DD)
    const datePart = dateString.substring(0, 10);
    const [year, month, day] = datePart.split('-').map(Number);
    
    if (!year || !month || !day) return null;

    return new Date(year, month - 1, day);
};

const etapas = computed(() => {
    const announcement = announcementData.value;
    const stages = [
        {
            id: 1,
            titulo: 'Publicación de Convocatoria',
            dateStartKey: 'publication_start',
            color: '#1B396A'
        },
        {
            id: 2,
            titulo: 'Registro y Carga de Documentos',
            dateStartKey: 'registration_start',
            dateEndKey: 'registration_end',
            color: '#10A558'
        },
        {
            id: 3,
            titulo: 'Evaluación y Dictaminación',
            dateStartKey: 'evaluation_start',
            dateEndKey: 'evaluation_end',
            color: '#E9C81F'
        },
        {
            id: 4,
            titulo: 'Publicación de Resultados',
            dateStartKey: 'results_start',
            dateEndKey: 'results_end',
            color: '#2B6CB0'
        }
    ];

    if (!announcement || !announcement.calendar) {
        return stages.map(s => ({
            ...s,
            fechas: 'Por definir'
        }));
    }

    return stages.map(stage => {
        const start = parseDateLocal(announcement.calendar[stage.dateStartKey]);
        let fechasStr = start ? start.toLocaleDateString('es-MX', { day: '2-digit', month: 'short' }) : 'Por definir';
        
        if (stage.dateEndKey && announcement.calendar[stage.dateEndKey]) {
            const end = parseDateLocal(announcement.calendar[stage.dateEndKey]);
            if (end) {
                fechasStr += ` - ${end.toLocaleDateString('es-MX', { day: '2-digit', month: 'short', year: 'numeric' })}`;
            }
        } else if (start) {
             fechasStr += ` ${start.getFullYear()}`;
        }

        return {
            ...stage,
            fechas: fechasStr
        };
    });
});
</script>

<template>
    <LandingLayout>

        <Head title="Convocatoria" />

        <div class="announcement-page">
            <section class="announcement-hero">
                <div class="announcement-container">
                    <nav class="announcement-breadcrumb" aria-label="Ruta de navegación">
                        <Link :href="route('inicio')">Inicio</Link><span aria-hidden="true">/</span><span>Convocatoria</span>
                    </nav>
                    <div class="hero-heading">
                        <div>
                            <span class="hero-eyebrow">PROGRAMA DE ESTÍMULOS DOCENTES</span>
                            <h1>Información de Convocatoria</h1>
                            <p>Detalles y requisitos para participar en el proceso actual para los Institutos Tecnológicos Federales y Centros</p>
                        </div>
                        <div class="hero-mark" aria-hidden="true"><FileText :size="44" :stroke-width="1.4" /></div>
                    </div>
                    <nav class="section-navigation" aria-label="Secciones de la convocatoria">
                        <a href="#participantes"><Users :size="16" /> ¿A quién va dirigido?</a>
                        <a href="#bases"><ClipboardCheck :size="16" /> Bases de Participación</a>
                        <a href="#calendario"><CalendarDays :size="16" /> Calendario de Actividades</a>
                    </nav>
                </div>
            </section>

            <section class="announcement-content announcement-container">
                <div class="details-column">
                    <article id="participantes" class="information-card">
                        <div class="card-heading"><span class="heading-icon"><Users :size="23" /></span><h2>¿A quién va dirigido?</h2></div>
                        <p class="audience-copy">Personal docente de tiempo completo (40 hrs) adscrito a los Institutos y Centros Federales del TecNM que imparten el 80% del año lectivo 2024 y cumplan con el 80% durante el primer semestre de 2025.</p>
                    </article>

                    <article id="bases" class="information-card requirements-card">
                        <div class="card-heading"><span class="heading-icon"><ClipboardCheck :size="23" /></span><h2>Bases de Participación</h2></div>
                        <ol class="requirements-list">
                            <li><span class="requirement-number">01</span><p>Haber cubierto la carga horaria reglamentaria frente a grupo durante los dos semestres de 2024 y en el primer semestre de 2026.</p></li>
                            <li><span class="requirement-number">02</span><p>Personal docente titular en subdirección, jefatura de departamento o área administrativa (50% docencia, 50% funciones directivas).</p></li>
                            <li><span class="requirement-number">03</span><p>Personal docente titular en Docencia e Investigación con mínimo 20 horas-semana en los últimos dos semestres de 2024 y primer semestre de 2026.</p></li>
                            <li><span class="requirement-number">04</span><p>Personal docente que haya disfrutado licencia sin goce de sueldo según Artículo 12 de los Lineamientos del Programa.</p></li>
                            <li><span class="requirement-number">05</span><p>Personal con nombramiento de tiempo parcial no podrá participar.</p></li>
                            <li><span class="requirement-number">06</span><p>Integrar expediente virtual completo según período establecido.</p></li>
                        </ol>
                    </article>

                    <article class="participation-card">
                        <div><h2>¿Listo para participar?</h2><p>Si cumples con los requisitos, inicia tu trámite hoy mismo.</p></div>
                        <Link :href="route('login')" class="participation-button">Iniciar Sesión <ArrowRight :size="18" /></Link>
                    </article>
                </div>

                <aside class="calendar-column" aria-label="Documento y calendario">
                    <article v-if="announcementData" class="featured-announcement">
                        <div class="featured-top"><span class="document-icon"><FileText :size="24" /></span><span class="featured-label">CONVOCATORIA</span></div>
                        <h2>{{ announcementData.name }}</h2>
                        <p class="announcement-description">{{ announcementData.description || 'Sin descripción disponible.' }}</p>
                        <button v-if="announcementData.file_url && announcementData.status !== 'pendiente'" @click="openPdfModal(route('announcement.preview', announcementData.id), announcementData.name)" class="document-button"><FileText :size="18" /> Ver Convocatoria PDF <ArrowUpRight :size="18" /></button>
                        <div v-else-if="announcementData.status === 'pendiente'" class="document-notice"><Clock3 :size="18" /> Convocatoria Próximamente</div>
                        <div v-else class="document-notice"><FileText :size="18" /> Sin PDF disponible</div>
                    </article>
                    <article v-else class="information-card empty-announcement"><FileText :size="32" /><h2>No hay convocatorias activas</h2><p>Por favor, vuelve más tarde.</p></article>

                    <article id="calendario" class="information-card calendar-card">
                        <div class="card-heading"><span class="heading-icon"><CalendarDays :size="23" /></span><h2>Calendario de Actividades</h2></div>
                        <ol class="stages-list">
                            <li v-for="etapa in etapas" :key="etapa.id" class="stage-item" :style="{ '--stage-color': etapa.color }">
                                <span class="stage-number">{{ etapa.id }}</span>
                                <div><span class="stage-label">FASE {{ etapa.id }}</span><h3>{{ etapa.titulo }}</h3><p class="stage-date"><CalendarDays :size="14" /><span>{{ etapa.fechas }}</span></p></div>
                            </li>
                        </ol>
                    </article>
                </aside>
            </section>
        </div>

        <!-- PDF Modal -->
        <Teleport to="body">
            <Transition name="modal">
                <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center p-4">
                    <!-- Backdrop -->
                    <div class="absolute inset-0 bg-black/60 backdrop-blur-sm"></div>
                    
                    <!-- Modal Content -->
                    <div class="relative bg-white rounded-2xl shadow-2xl w-full max-w-6xl h-[88vh] sm:h-[90vh] flex flex-col">
                        <!-- Modal Header -->
                        <div class="flex items-start sm:items-center justify-between gap-3 p-4 sm:p-6 border-b border-gray-200">
                            <h2 class="text-lg font-semibold text-gray-900">{{ currentPdfTitle }}</h2>
                            <button @click="closeModal" class="text-gray-400 hover:text-gray-800 transition-colors cursor-pointer">
                                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                                </svg>
                            </button>
                        </div>
                        
                        <!-- Document Viewer -->
                        <div class="flex-1 overflow-hidden bg-gray-100">
                            <iframe
                                v-if="currentPreviewIsPdf"
                                :src="currentPdfUrl"
                                class="w-full h-full"
                                frameborder="0"
                            ></iframe>

                            <div
                                v-else-if="currentPreviewIsImage"
                                class="w-full h-full overflow-auto flex items-center justify-center p-4"
                            >
                                <img
                                    :src="currentPdfUrl"
                                    :alt="currentPdfTitle"
                                    class="max-w-full max-h-full object-contain mx-auto"
                                />
                            </div>

                            <div v-else class="w-full h-full flex items-center justify-center p-8">
                                <a
                                    :href="currentPdfUrl"
                                    target="_blank"
                                    class="px-4 py-2 rounded-md bg-[var(--institutional-primary)] text-white font-medium hover:bg-[var(--institutional-primary-dark)]"
                                >
                                    Abrir documento
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </Transition>
        </Teleport>
    </LandingLayout>
</template>

<style scoped>
.announcement-page { --page-surface:#ffffff; --page-background:#f5f7fb; --page-text:#142c4d; --page-muted:#56677d; --page-border:#e2e8f0; background:var(--page-background); color:var(--page-text); }
.announcement-container { max-width:1280px; margin:0 auto; padding-left:24px; padding-right:24px; }
.announcement-hero { background:linear-gradient(115deg,var(--institutional-primary),var(--institutional-primary-dark)); color:white; padding:26px 0 0; }
.announcement-breadcrumb { display:flex; gap:12px; align-items:center; color:#c8d6e9; font-size:12px; }
.announcement-breadcrumb a:hover { color:white; }
.hero-heading { display:flex; align-items:center; justify-content:space-between; gap:32px; padding:32px 0; }
.hero-eyebrow { font-size:11px; font-weight:700; letter-spacing:.13em; color:var(--institutional-gold); }
.hero-heading h1 { font-size:clamp(28px,3.4vw,40px); font-weight:750; line-height:1.2; letter-spacing:-.025em; margin:12px 0; }
.hero-heading p { max-width:700px; color:#d8e5f6; font-size:15px; line-height:1.7; }
.hero-mark { padding:22px; border:1px solid #ffffff25; background:#ffffff08; border-radius:24px; color:#d5e3f6; }
.section-navigation { display:flex; gap:30px; border-top:1px solid #ffffff20; padding:19px 0; flex-wrap:wrap; }
.section-navigation a { display:inline-flex; align-items:center; gap:9px; font-size:13px; color:#edf4ff; }
.section-navigation a:hover { color:var(--institutional-gold); }
.announcement-content { display:grid; grid-template-columns:minmax(0,1.35fr) minmax(0,1fr); align-items:start; gap:28px; padding-top:36px; padding-bottom:64px; }
.details-column,.calendar-column { display:grid; gap:24px; min-width:0; }
.information-card { background:var(--page-surface); border:1px solid var(--page-border); border-radius:18px; padding:28px; box-shadow:0 4px 18px #10234404; scroll-margin-top:120px; }
.card-heading { display:flex; align-items:center; gap:13px; margin-bottom:20px; }
.heading-icon { display:flex; align-items:center; justify-content:center; width:44px; height:44px; border-radius:12px; background:#1b396a0a; color:var(--institutional-primary); flex-shrink:0; }
.card-heading h2 { font-size:19px; font-weight:700; line-height:1.35; letter-spacing:-.02em; }
.audience-copy { font-size:15px; line-height:1.85; color:var(--page-muted); }
.requirements-list { list-style:none; margin:0; padding:0; }
.requirements-list li { display:flex; gap:16px; padding:19px 0; border-bottom:1px solid var(--page-border); align-items:flex-start; }
.requirements-list li:first-child { padding-top:3px; }
.requirements-list li:last-child { padding-bottom:0; border-bottom:0; }
.requirement-number { flex-shrink:0; font-size:12px; font-weight:750; color:var(--institutional-primary); background:#1b396a0a; border-radius:8px; width:32px; height:32px; display:flex; align-items:center; justify-content:center; }
.requirements-list p { font-size:14px; color:var(--page-muted); line-height:1.8; }
.participation-card { display:flex; justify-content:space-between; align-items:center; gap:20px; border:1px solid var(--page-border); border-radius:18px; padding:26px 28px; background:var(--page-surface); }
.participation-card h2 { font-size:18px; font-weight:700; margin-bottom:6px; }
.participation-card p { font-size:13px; color:var(--page-muted); line-height:1.65; }
.participation-button { background:var(--institutional-primary); color:white; border-radius:10px; padding:12px 16px; display:inline-flex; align-items:center; justify-content:center; gap:12px; font-size:13px; font-weight:650; flex-shrink:0; }
.participation-button:hover { background:var(--institutional-primary-dark); }
.featured-announcement { background:linear-gradient(135deg,var(--institutional-primary),var(--institutional-primary-dark)); color:white; border-radius:18px; padding:28px; border-top:3px solid var(--institutional-gold); }
.featured-top { display:flex; align-items:center; gap:12px; margin-bottom:22px; }
.document-icon { padding:10px; display:flex; border:1px solid #ffffff25; border-radius:12px; background:#ffffff08; }
.featured-label { color:#cadcf4; font-size:11px; letter-spacing:.12em; font-weight:700; }
.featured-announcement h2 { font-size:26px; line-height:1.3; font-weight:750; letter-spacing:-.02em; overflow-wrap:anywhere; margin-bottom:14px; }
.announcement-description { font-size:14px; line-height:1.8; color:#d6e4f8; white-space:pre-line; overflow-wrap:anywhere; margin-bottom:26px; }
.document-button { width:100%; display:flex; gap:10px; align-items:center; justify-content:center; background:white; color:var(--institutional-primary); font-size:13px; font-weight:700; padding:14px; border-radius:10px; cursor:pointer; transition:background .2s; }
.document-button:hover { background:#e8eef7; }
.document-notice { display:flex; gap:9px; align-items:center; font-size:13px; background:#ffffff0d; border:1px solid #ffffff20; padding:14px; border-radius:10px; }
.empty-announcement { text-align:center; padding:36px 24px; }
.empty-announcement svg { margin:0 auto 16px; color:var(--page-muted); }
.empty-announcement h2 { font-size:19px; font-weight:700; margin-bottom:8px; }
.empty-announcement p { color:var(--page-muted); font-size:14px; }
.stages-list { list-style:none; padding:4px 0 0; margin:0; }
.stage-item { position:relative; display:grid; grid-template-columns:36px minmax(0,1fr); gap:16px; padding-bottom:30px; }
.stage-item:last-child { padding-bottom:0; }
.stage-item:not(:last-child)::before { content:''; position:absolute; top:38px; bottom:2px; left:17px; width:1px; background:var(--page-border); }
.stage-number { width:36px; height:36px; display:flex; justify-content:center; align-items:center; border-radius:11px; color:var(--page-text); background:var(--page-background); border:1px solid var(--page-border); border-top:3px solid var(--stage-color); font-size:13px; font-weight:750; }
.stage-label { font-size:10px; color:var(--page-muted); letter-spacing:.09em; font-weight:650; }
.stage-item h3 { font-size:14px; font-weight:700; line-height:1.5; margin:4px 0 8px; }
.stage-date { display:flex; align-items:center; gap:7px; color:var(--page-muted); font-size:12px; line-height:1.6; }
.stage-date svg { flex-shrink:0; }
.announcement-page a:focus-visible,.announcement-page button:focus-visible { outline:3px solid var(--institutional-gold); outline-offset:4px; }
:global(.dark .announcement-page) { --page-background:var(--institutional-dark-background); --page-surface:var(--institutional-dark-surface); --page-text:var(--institutional-dark-text); --page-muted:#bbc8dd; --page-border:#ffffff18; }
:global(.dark .announcement-page .heading-icon),:global(.dark .announcement-page .requirement-number) { background:#ffffff08; color:#b8d8ff; }
@media(max-width:1023px) { .announcement-content { grid-template-columns:1fr; } .calendar-column { grid-template-columns:1fr 1fr; align-items:start; } .participation-card { flex-wrap:wrap; } }
@media(max-width:639px) { .announcement-container { padding-left:16px; padding-right:16px; } .hero-heading { padding:26px 0; } .hero-mark { display:none; } .hero-heading p { font-size:14px; } .section-navigation { gap:14px 20px; } .section-navigation a { font-size:12px; } .announcement-content { padding-top:24px; padding-bottom:40px; gap:20px; } .calendar-column { grid-template-columns:1fr; } .information-card,.featured-announcement,.participation-card { padding:22px; } .card-heading h2 { font-size:17px; } .participation-button { width:100%; } }
.modal-enter-active,
.modal-leave-active {
    transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
    opacity: 0;
}

.modal-enter-active .relative,
.modal-leave-active .relative {
    transition: transform 0.3s ease;
}

.modal-enter-from .relative,
.modal-leave-to .relative {
    transform: scale(0.95);
}
</style>
