<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tablero de Seguimiento Territorial | IA & DB</title>
    <script src="https://cdn.tailwindcss.com?plugins=typography"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; color: #1e293b; }
        
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #f1f5f9; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        
        .tab-btn.active { border-bottom: 2px solid #0f172a; color: #0f172a; font-weight: 600; }
        
        /* Animación IA global */
        @keyframes pulse-border {
            0% { box-shadow: 0 0 0 0 rgba(99, 102, 241, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(99, 102, 241, 0); }
            100% { box-shadow: 0 0 0 0 rgba(99, 102, 241, 0); }
        }
        .ai-btn { animation: pulse-border 2s infinite; }
        .ai-btn:hover { animation: none; }
        
        #loader { transition: opacity 0.3s ease; }

        /* Estrellas interactivas */
        .star-rating span { cursor: pointer; transition: color 0.2s; }
        .star-rating span:hover, .star-rating span:hover ~ span { color: #fbbf24; } /* Amber-400 */
        .star-active { color: #f59e0b; } /* Amber-500 */
        .star-inactive { color: #e2e8f0; } /* Slate-200 */
    </style>
</head>
<body class="antialiased min-h-screen flex flex-col relative">

    <!-- Pantalla de carga inicial (Base de datos) -->
    <div id="loader" class="fixed inset-0 bg-slate-900 z-[100] flex flex-col items-center justify-center text-white">
        <span class="text-6xl mb-4 animate-bounce">📍</span>
        <h2 class="text-xl font-bold mb-2">Conectando a la Base de Datos...</h2>
        <p class="text-slate-400 text-sm">Sincronizando información territorial</p>
    </div>

    <!-- Header -->
    <header class="bg-white shadow-sm sticky top-0 z-40 border-b border-slate-200">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between items-center h-16">
                <div class="flex items-center">
                    <span class="text-2xl mr-2">📍</span>
                    <h1 class="font-bold text-xl text-slate-800 tracking-tight">GeoTrack Territorial</h1>
                </div>
                <div class="flex items-center gap-3">
                    <span id="db-status" class="text-xs font-bold text-emerald-500 bg-emerald-50 px-2 py-1 rounded-full border border-emerald-200 hidden sm:inline-block">🟢 DB Online</span>
                    <button onclick="window.abrirModalAsistenteIA()" class="ai-btn bg-gradient-to-r from-purple-600 to-indigo-600 text-white px-4 py-2 rounded-lg text-sm font-bold hover:from-purple-700 hover:to-indigo-700 transition shadow-md flex items-center">
                        <span class="mr-2 text-lg">✨</span> <span class="hidden sm:inline">IA Assistant</span>
                    </button>
                </div>
            </div>
            <!-- Navigation Tabs -->
            <nav class="flex space-x-4 sm:space-x-8 mt-2 overflow-x-auto" aria-label="Tabs">
                <button onclick="window.switchTab('dashboard')" id="tab-dashboard" class="tab-btn active pb-3 text-sm sm:text-base text-slate-500 hover:text-slate-800 transition-colors whitespace-nowrap">📊 Panel General</button>
                <button onclick="window.switchTab('directorio')" id="tab-directorio" class="tab-btn pb-3 text-sm sm:text-base text-slate-500 hover:text-slate-800 transition-colors whitespace-nowrap">📂 Expedientes</button>
                <button onclick="window.switchTab('agenda')" id="tab-agenda" class="tab-btn pb-3 text-sm sm:text-base text-slate-500 hover:text-slate-800 transition-colors whitespace-nowrap">📅 Agenda</button>
                <button onclick="window.switchTab('eventos')" id="tab-eventos" class="tab-btn pb-3 text-sm sm:text-base text-slate-500 hover:text-slate-800 transition-colors whitespace-nowrap">🌟 Eventos CABA</button>
            </nav>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-grow max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8 w-full">

        <!-- SECTION 1: DASHBOARD -->
        <section id="sec-dashboard" class="space-y-6 block">
            <div class="bg-gradient-to-r from-slate-800 to-slate-900 p-8 rounded-2xl shadow-lg text-white flex justify-between items-center flex-wrap gap-4">
                <div>
                    <h2 class="text-3xl font-bold mb-2">Panel General</h2>
                    <p class="text-slate-300">Monitoreo en tiempo real de la base de datos cloud.</p>
                </div>
            </div>
            <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-center items-center text-center">
                    <span class="text-4xl mb-2">👥</span>
                    <h3 class="text-sm font-semibold text-slate-500 uppercase tracking-wider">Militancia Total</h3>
                    <p id="kpi-wp" class="text-5xl font-bold text-slate-800 mt-2">0</p>
                </div>
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-center items-center text-center">
                    <span class="text-4xl mb-2">🔥</span>
                    <h3 class="text-sm font-semibold text-slate-500 uppercase tracking-wider">Activos</h3>
                    <p id="kpi-activos" class="text-5xl font-bold text-indigo-600 mt-2">0</p>
                </div>
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-center items-center text-center sm:col-span-2 md:col-span-1">
                    <span class="text-4xl mb-2">🌟</span>
                    <h3 class="text-sm font-semibold text-slate-500 uppercase tracking-wider">Eventos CABA</h3>
                    <p id="kpi-eventos-grales" class="text-5xl font-bold text-emerald-600 mt-2">0</p>
                </div>
            </div>
        </section>

        <!-- SECTION 2: DIRECTORIO DE COMUNAS -->
        <section id="sec-directorio" class="hidden space-y-6">
            <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100 flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Expedientes por Comuna</h2>
                    <p class="text-slate-600 text-sm">Seleccione una comuna para gestionar sus datos y evaluar su compromiso.</p>
                </div>
                <div class="w-full md:w-auto min-w-[250px]">
                    <select id="comuna-selector" onchange="window.actualizarVistaDetalle()" class="block w-full rounded-md border-slate-300 border p-3 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm bg-slate-50 font-medium">
                        <option value="">-- Seleccione una Comuna --</option>
                    </select>
                </div>
            </div>

            <!-- Detalle de la comuna -->
            <div id="comuna-detalle" class="hidden flex flex-col lg:flex-row gap-6">
                <!-- Columna Izquierda: Datos -->
                <div class="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden flex-grow relative" id="detalle-card">
                    <div class="p-6 border-b border-slate-100 bg-slate-50 flex flex-col sm:flex-row justify-between items-start gap-4">
                        <div>
                            <h3 id="det-nombre" class="text-3xl font-bold text-slate-800">Comuna --</h3>
                            <div class="mt-2 flex flex-wrap items-center gap-2 text-sm">
                                <span id="det-estado" class="font-medium px-3 py-1 rounded-full bg-white border shadow-sm">--</span>
                                <span class="text-slate-600 bg-white px-3 py-1 rounded-full border shadow-sm flex items-center">
                                    Compromiso Promedio: <span id="det-compromiso" class="ml-2 font-bold text-lg text-amber-500">--</span>
                                </span>
                            </div>
                        </div>
                        <button onclick="window.abrirModalEdicion()" class="bg-slate-800 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-slate-700 transition shadow-sm flex items-center">
                            📝 Editar Datos
                        </button>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-3 divide-y md:divide-y-0 md:divide-x divide-slate-100">
                        <div class="p-6 space-y-6">
                            <h4 class="font-bold text-slate-400 uppercase tracking-widest text-xs">Termómetro</h4>
                            <div class="flex justify-between items-center"><span class="text-slate-600">Padrón Gral</span><span id="det-wp" class="font-bold text-xl">0</span></div>
                            <div class="flex justify-between items-center"><span class="text-slate-600">Activos</span><span id="det-activos" class="font-bold text-xl text-indigo-600">0</span></div>
                            <div class="flex justify-between items-center"><span class="text-slate-600">Mesa Chica</span><span id="det-equipo" class="font-bold text-xl">0</span></div>
                        </div>
                        <div class="p-6 md:col-span-2 space-y-6">
                            <div>
                                <h4 class="font-bold text-slate-400 uppercase tracking-widest text-xs mb-2">Delegados / Estructura Barrial</h4>
                                <p class="text-slate-800 bg-slate-50 p-3 rounded border border-slate-100"><strong id="det-delegados" class="text-slate-700 font-normal">--</strong></p>
                            </div>
                            <div>
                                <h4 class="font-bold text-slate-400 uppercase tracking-widest text-xs mb-2">Notas Territoriales</h4>
                                <div class="bg-amber-50 p-4 rounded border border-amber-100">
                                    <p id="det-planes" class="text-slate-700 whitespace-pre-wrap">"--"</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            
            <div id="comuna-vacia" class="text-center p-12 bg-white rounded-xl shadow-sm border border-slate-100 text-slate-400">
                <span class="text-4xl block mb-4">🔍</span> Selecciona una comuna para ver los datos de la base.
            </div>
        </section>

        <!-- SECTION 3: AGENDA -->
        <section id="sec-agenda" class="hidden space-y-6">
            <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100 flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Agenda Inter-Comunal</h2>
                    <p class="text-slate-600 text-sm">Calendario de reuniones y actividades específicas.</p>
                </div>
                <button onclick="window.abrirModalAgenda()" class="bg-indigo-600 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-indigo-700 shadow-sm whitespace-nowrap">
                    + Nueva Reunión
                </button>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6" id="agenda-container"></div>
            <div id="agenda-vacia" class="hidden text-center p-12 bg-white rounded-xl border border-slate-100 text-slate-400">📭 No hay reuniones programadas.</div>
        </section>

        <!-- SECTION 4: EVENTOS GENERALES CABA -->
        <section id="sec-eventos" class="hidden space-y-6">
            <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100 flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
                <div>
                    <h2 class="text-2xl font-bold mb-2 text-indigo-900 flex items-center"><span class="mr-2">🌟</span> Eventos Generales (CABA)</h2>
                    <p class="text-slate-600 text-sm">Evaluación de asistencia real vs esperada y compromiso de delegados.</p>
                </div>
                <button onclick="window.abrirModalEventoGeneral()" class="bg-emerald-600 text-white px-4 py-2 rounded-lg text-sm font-bold hover:bg-emerald-700 shadow-md whitespace-nowrap">
                    + Crear Evento CABA
                </button>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6" id="eventos-grales-container"></div>
            <div id="eventos-grales-vacia" class="hidden text-center p-12 bg-white rounded-xl border border-slate-100 text-slate-400">📭 No hay eventos generales registrados.</div>
        </section>

    </main>

    <!-- ================= MODALES ================= -->

    <!-- Modal Edición Manual de Comuna -->
    <div id="modal-editar" class="hidden fixed inset-0 bg-slate-900 bg-opacity-60 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
        <div class="bg-white rounded-xl shadow-2xl max-w-xl w-full">
            <div class="p-5 border-b border-slate-100 flex justify-between items-center"><h3 class="text-lg font-bold">Editar Datos de Comuna</h3><button onclick="window.cerrarModalEdicion()" class="text-slate-400 text-xl hover:text-slate-800">✖</button></div>
            <form id="form-editar" onsubmit="window.guardarRegistro(event)" class="p-6 space-y-4">
                <div class="bg-blue-50 text-blue-800 text-xs p-3 rounded-lg border border-blue-100 flex items-start gap-2">
                    <span class="text-lg leading-none">ℹ️</span> 
                    <p>El <strong>Compromiso</strong> no se edita aquí. Se calcula automáticamente según la participación de la comuna en la pestaña de <strong>Eventos CABA</strong>.</p>
                </div>

                <div class="grid grid-cols-3 gap-4">
                    <div><label class="text-xs font-bold text-slate-600">Padrón</label><input type="number" id="edit-wp" class="w-full border p-2 rounded focus:ring-indigo-500 focus:border-indigo-500 outline-none"></div>
                    <div><label class="text-xs font-bold text-slate-600">Activos</label><input type="number" id="edit-activos" class="w-full border p-2 rounded focus:ring-indigo-500 focus:border-indigo-500 outline-none"></div>
                    <div><label class="text-xs font-bold text-slate-600">Mesa Chica</label><input type="number" id="edit-equipo" class="w-full border p-2 rounded focus:ring-indigo-500 focus:border-indigo-500 outline-none"></div>
                </div>
                <div><label class="text-xs font-bold text-slate-600">Estado Territorial</label><select id="edit-estado" class="w-full border p-2 rounded focus:ring-indigo-500 focus:border-indigo-500 outline-none"><option value="Muy Activo">Muy Activo</option><option value="Cuello Botella">Cuello Botella</option><option value="Recién empezando">Recién empezando</option><option value="Desorganizado">Desorganizado</option><option value="Sin Datos">Sin Datos</option></select></div>
                <div>
                    <label class="text-xs font-bold text-slate-600">Delegados / Referentes</label>
                    <input type="text" id="edit-delegados" placeholder="Nombres de los encargados..." class="w-full border p-2 rounded focus:ring-indigo-500 focus:border-indigo-500 outline-none">
                </div>
                <div><label class="text-xs font-bold text-slate-600">Notas Adicionales</label><textarea id="edit-planes" rows="3" class="w-full border p-2 rounded focus:ring-indigo-500 focus:border-indigo-500 outline-none"></textarea></div>
                
                <div class="flex justify-end gap-2 pt-2"><button type="button" onclick="window.cerrarModalEdicion()" class="px-4 py-2 bg-slate-100 text-slate-600 rounded-lg hover:bg-slate-200">Cancelar</button><button type="submit" class="px-4 py-2 bg-slate-800 text-white rounded-lg hover:bg-slate-700 font-bold">Guardar en DB</button></div>
            </form>
        </div>
    </div>

    <!-- Modal Agenda Intercomunal -->
    <div id="modal-agenda" class="hidden fixed inset-0 z-50 flex items-center justify-center bg-slate-900 bg-opacity-60 p-4 backdrop-blur-sm">
        <div class="bg-white p-6 rounded-xl w-full max-w-md shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h3 class="font-bold text-lg">Nueva Reunión / Actividad</h3>
                <button onclick="window.cerrarModalAgenda()" class="text-slate-400 hover:text-slate-800">✖</button>
            </div>
            <form id="form-agenda" onsubmit="window.guardarEventoAgenda(event)">
                <input type="text" id="ev-nombre" placeholder="Título de la reunión" class="w-full border border-slate-300 p-2 mb-3 rounded outline-none focus:border-indigo-500" required>
                <input type="date" id="ev-fecha" class="w-full border border-slate-300 p-2 mb-3 rounded outline-none focus:border-indigo-500" required>
                <div class="mb-4">
                    <label class="text-xs font-bold text-slate-600 mb-1 block">Comunas involucradas (separar por comas, ej: C1, C12)</label>
                    <input type="text" id="ev-comunas" placeholder="C1, C2" class="w-full border border-slate-300 p-2 rounded outline-none focus:border-indigo-500" required>
                </div>
                <div class="flex justify-end gap-2"><button type="button" onclick="window.cerrarModalAgenda()" class="px-4 py-2 bg-slate-100 rounded-lg">Cancelar</button><button type="submit" class="px-4 py-2 bg-indigo-600 text-white rounded-lg font-bold">Agendar</button></div>
            </form>
        </div>
    </div>

    <!-- Modal Evento General CABA -->
    <div id="modal-evento-general" class="hidden fixed inset-0 z-50 flex items-center justify-center bg-slate-900 bg-opacity-70 p-4 backdrop-blur-sm">
        <div class="bg-white rounded-xl w-full max-w-3xl shadow-2xl max-h-[95vh] flex flex-col">
            <div class="p-5 border-b border-slate-100 flex justify-between items-center bg-emerald-50 rounded-t-xl">
                <div>
                    <h3 class="font-bold text-xl text-emerald-900 flex items-center"><span class="mr-2">🌟</span> Gestión de Evento CABA</h3>
                    <p class="text-xs text-emerald-700">Compara asistencia deseada vs real y califica territorios.</p>
                </div>
                <button onclick="window.cerrarModalEventoGeneral()" class="text-emerald-700 hover:text-emerald-900 text-2xl leading-none">✖</button>
            </div>
            
            <div class="p-6 overflow-y-auto flex-grow" id="form-evento-gral-container">
                <!-- Se inyecta dinámicamente vía JS -->
            </div>
            
            <div class="p-4 border-t border-slate-100 bg-slate-50 rounded-b-xl flex justify-between items-center" id="footer-evento-gral">
                <!-- Botones se inyectan vía JS -->
            </div>
        </div>
    </div>

    <!-- Modal Confirmación Realizado -->
    <div id="modal-confirmar-realizado" class="hidden fixed inset-0 z-[60] flex items-center justify-center bg-slate-900 bg-opacity-80 p-4">
        <div class="bg-white p-6 rounded-xl w-full max-w-sm text-center shadow-2xl border-2 border-emerald-500">
            <span class="text-5xl block mb-4">✅</span>
            <h3 class="font-bold text-lg mb-2 text-slate-800">¿Confirmar ejecución?</h3>
            <p class="text-sm text-slate-600 mb-6">Al marcar el evento como <strong>Realizado</strong>, se habilitará la pestaña para ingresar los <strong>Asistentes Reales</strong> y calificar el compromiso de cada delegado. Esta acción no se puede deshacer.</p>
            <div class="flex justify-center gap-3">
                <button onclick="document.getElementById('modal-confirmar-realizado').classList.add('hidden')" class="px-4 py-2 bg-slate-100 text-slate-700 rounded-lg hover:bg-slate-200">Cancelar</button>
                <button onclick="window.ejecutarMarcarRealizado()" class="px-4 py-2 bg-emerald-600 text-white rounded-lg font-bold hover:bg-emerald-700 shadow-md">Sí, confirmar</button>
            </div>
        </div>
    </div>

    <!-- Modal IA -->
    <div id="modal-asistente-ia" class="hidden fixed inset-0 bg-slate-900 bg-opacity-70 z-50 flex items-center justify-center p-4 backdrop-blur-sm transition-opacity">
        <div class="bg-white rounded-2xl shadow-2xl max-w-3xl w-full flex flex-col max-h-[90vh]">
            <div class="p-6 border-b border-slate-100 bg-gradient-to-r from-indigo-900 to-purple-900 rounded-t-2xl flex justify-between items-center text-white">
                <div>
                    <h3 class="text-xl font-bold flex items-center"><span class="mr-2 text-2xl">✨</span> Cerebro IA Territorial</h3>
                    <p class="text-indigo-200 text-xs mt-1">Capaz de actualizar padrones y agendar reuniones al mismo tiempo.</p>
                </div>
                <button onclick="window.cerrarModalAsistenteIA()" class="text-indigo-200 hover:text-white font-bold text-2xl leading-none">✖</button>
            </div>
            <div class="p-6 flex-grow overflow-y-auto">
                <div class="bg-indigo-50 border border-indigo-100 rounded-lg p-4 mb-4 text-sm text-indigo-900">
                    <strong>Pruébame:</strong> <em>"En la comuna 15 sumamos 5 militantes más, y agendá una mateada para el sábado."</em>
                </div>
                <textarea id="ia-raw-text" rows="5" class="w-full rounded-xl border-slate-300 border p-4 text-slate-700 focus:ring-indigo-500 focus:border-indigo-500 shadow-sm text-lg outline-none" placeholder="Escribe tu mensaje de voz transcrito o reporte aquí..."></textarea>
                <div id="ia-error-msg" class="hidden mt-3 text-sm text-red-600 bg-red-50 p-3 rounded-lg border border-red-200"></div>
                <div id="ia-success-msg" class="hidden mt-3 text-sm text-emerald-700 bg-emerald-50 p-4 rounded-lg border border-emerald-200">
                    <strong class="block mb-1">✅ Acción completada:</strong><ul id="ia-resumen" class="list-disc pl-5 space-y-1"></ul>
                </div>
            </div>
            <div class="p-5 border-t border-slate-100 bg-slate-50 rounded-b-2xl flex justify-end gap-3">
                <button type="button" onclick="window.cerrarModalAsistenteIA()" class="px-5 py-2.5 rounded-lg text-slate-600 hover:bg-slate-200 font-medium">Cerrar</button>
                <button id="btn-procesar-ia" type="button" onclick="window.procesarTextoConIA()" class="px-6 py-2.5 rounded-lg font-bold text-white bg-indigo-600 hover:bg-indigo-700 shadow-md flex items-center">Procesar con IA</button>
            </div>
        </div>
    </div>


    <!-- ================= FIREBASE MODULE ================= -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, onSnapshot, getDocs, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // CONFIGURACIÓN FIREBASE
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : { projectId: "demo-project" };
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'geotrack-app';
        
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        // ESTADO GLOBAL
        window.bd_comunas = [];
        window.bd_actividades = [];
        window.bd_eventos_generales = [];
        window.currentCommuneId = null;
        window.currentEventGralEdit = null; 
        let currentUser = null;

        const apiKey = ""; 

        // INICIALIZACIÓN
        async function initApp() {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        currentUser = user;
                        await setupDatabaseListeners();
                        document.getElementById('loader').style.opacity = '0';
                        setTimeout(() => document.getElementById('loader').style.display = 'none', 300);
                    }
                });
            } catch (error) {
                console.error("Error inicializando Firebase:", error);
                document.getElementById('loader').innerHTML = `<span class="text-4xl mb-4">❌</span><h2>Error de conexión</h2><p class="text-sm">${error.message}</p>`;
            }
        }

        // LISTENERS
        async function setupDatabaseListeners() {
            if (!currentUser) return;

            const comunasRef = collection(db, 'artifacts', appId, 'public', 'data', 'comunas');
            const agendaRef = collection(db, 'artifacts', appId, 'public', 'data', 'actividades');
            const eventosGralesRef = collection(db, 'artifacts', appId, 'public', 'data', 'eventos_generales');

            // Semilla inicial comunas
            const snapshot = await getDocs(comunasRef);
            if (snapshot.empty) {
                for(let i=1; i<=15; i++) {
                    await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'comunas', `C${i}`), {
                        id: `C${i}`, nombre: `Comuna ${i}`, estado_raw: "Sin Datos", estado: "⚪ Sin Datos",
                        delegados: "Desconocido", wp: 0, activos: 0, equipo: 0, planes: "Sin registros."
                    });
                }
            }

            onSnapshot(eventosGralesRef, (snap) => {
                window.bd_eventos_generales = [];
                snap.forEach(doc => window.bd_eventos_generales.push(doc.data()));
                window.renderEventosGenerales();
                if(window.bd_comunas.length > 0) actualizarUI();
            });

            onSnapshot(comunasRef, (snap) => {
                window.bd_comunas = [];
                snap.forEach(doc => window.bd_comunas.push(doc.data()));
                window.bd_comunas.sort((a,b) => parseInt(a.id.substring(1)) - parseInt(b.id.substring(1)));
                actualizarUI(); 
                if(window.currentCommuneId) window.actualizarVistaDetalle(); 
            });

            onSnapshot(agendaRef, (snap) => {
                window.bd_actividades = [];
                snap.forEach(doc => window.bd_actividades.push(doc.data()));
                window.renderAgenda();
            });
        }

        function calcularCompromisoComuna(comunaId) {
            let totalEstrellas = 0;
            let eventosParticipados = 0;
            
            window.bd_eventos_generales.forEach(ev => {
                if(ev.estado === "Realizado" && ev.participacion && ev.participacion[comunaId] !== undefined) {
                    if(ev.participacion[comunaId] > 0) { 
                        totalEstrellas += ev.participacion[comunaId];
                        eventosParticipados++;
                    }
                }
            });

            if(eventosParticipados === 0) return { promedio: 0, html: "<span class='text-slate-400'>Sin evaluación</span>" };
            
            const promedio = Math.round(totalEstrellas / eventosParticipados);
            let estrellasHTML = "";
            for(let i=1; i<=5; i++) {
                estrellasHTML += `<span class="${i <= promedio ? 'text-amber-500' : 'text-slate-300'}">★</span>`;
            }
            return { promedio, html: estrellasHTML };
        }

        function actualizarUI() {
            let activos = 0, wp = 0;
            const select = document.getElementById('comuna-selector');
            const prevSelect = select.value;
            select.innerHTML = '<option value="">-- Seleccione una Comuna --</option>';

            window.bd_comunas.forEach(c => {
                activos += c.activos || 0; wp += c.wp || 0;
                select.innerHTML += `<option value="${c.id}">${c.estado_raw === "Sin Datos" ? "⚪" : "✅"} ${c.nombre}</option>`;
            });
            select.value = prevSelect;

            document.getElementById('kpi-activos').innerText = activos;
            document.getElementById('kpi-wp').innerText = wp;
        }

        window.switchTab = (tabId) => {
            document.querySelectorAll('main > section').forEach(sec => sec.classList.add('hidden'));
            document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active', 'border-b-[2px]', 'text-slate-800'));
            document.getElementById('sec-' + tabId).classList.remove('hidden');
            document.getElementById('tab-' + tabId).classList.add('active');
        };

        window.actualizarVistaDetalle = () => {
            const val = document.getElementById('comuna-selector').value;
            window.currentCommuneId = val;
            if(!val) {
                document.getElementById('comuna-detalle').classList.add('hidden');
                document.getElementById('comuna-vacia').classList.remove('hidden');
                return;
            }
            const data = window.bd_comunas.find(c => c.id === val);
            if(!data) return;

            document.getElementById('comuna-vacia').classList.add('hidden');
            document.getElementById('comuna-detalle').classList.remove('hidden');
            document.getElementById('det-nombre').innerText = data.nombre;
            
            let colorClass = "bg-slate-100 text-slate-800";
            if(data.estado.includes("🟢")) colorClass = "bg-emerald-50 text-emerald-700 border-emerald-200";
            if(data.estado.includes("🟡")) colorClass = "bg-amber-50 text-amber-700 border-amber-200";
            if(data.estado.includes("🔴")) colorClass = "bg-rose-50 text-rose-700 border-rose-200";
            
            document.getElementById('det-estado').innerHTML = `<span class="inline-block px-2 py-1 rounded-md text-sm border ${colorClass}">${data.estado}</span>`;
            
            const compromiso = calcularCompromisoComuna(data.id);
            document.getElementById('det-compromiso').innerHTML = compromiso.html;
            
            document.getElementById('det-wp').innerText = data.wp || 0;
            document.getElementById('det-activos').innerText = data.activos || 0;
            document.getElementById('det-equipo').innerText = data.equipo || 0;
            document.getElementById('det-delegados').innerText = data.delegados || "Sin asignar";
            document.getElementById('det-planes').innerText = data.planes || "Sin notas.";
        };

        window.abrirModalEdicion = () => {
            const d = window.bd_comunas.find(c => c.id === window.currentCommuneId);
            document.getElementById('edit-wp').value = d.wp || 0; 
            document.getElementById('edit-activos').value = d.activos || 0; 
            document.getElementById('edit-equipo').value = d.equipo || 0;
            document.getElementById('edit-estado').value = d.estado_raw || "Sin Datos"; 
            document.getElementById('edit-planes').value = d.planes || "";
            document.getElementById('edit-delegados').value = d.delegados || ""; 
            document.getElementById('modal-editar').classList.remove('hidden');
        };
        window.cerrarModalEdicion = () => document.getElementById('modal-editar').classList.add('hidden');
        
        window.guardarRegistro = async (e) => {
            e.preventDefault();
            if(!currentUser || !window.currentCommuneId) return;
            const d = window.bd_comunas.find(c => c.id === window.currentCommuneId);
            const nEst = document.getElementById('edit-estado').value;
            const sMap = {"Muy Activo":"🟢", "Cuello Botella":"🟡", "Recién empezando":"🟡", "Desorganizado":"🔴", "Sin Datos":"⚪"};
            
            const updatedData = {
                ...d,
                wp: parseInt(document.getElementById('edit-wp').value)||0,
                activos: parseInt(document.getElementById('edit-activos').value)||0,
                equipo: parseInt(document.getElementById('edit-equipo').value)||0,
                estado_raw: nEst,
                estado: `${sMap[nEst]} ${nEst}`,
                delegados: document.getElementById('edit-delegados').value,
                planes: document.getElementById('edit-planes').value
            };
            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'comunas', d.id), updatedData);
            window.cerrarModalEdicion();
        };

        window.renderAgenda = () => {
            const cont = document.getElementById('agenda-container');
            cont.innerHTML = '';
            if(window.bd_actividades.length === 0) {
                cont.classList.add('hidden'); document.getElementById('agenda-vacia').classList.remove('hidden'); return;
            }
            cont.classList.remove('hidden'); document.getElementById('agenda-vacia').classList.add('hidden');
            const ordenadas = [...window.bd_actividades].sort((a,b) => new Date(a.fecha) - new Date(b.fecha));
            ordenadas.forEach(a => {
                cont.innerHTML += `
                    <div class="bg-white p-5 rounded-xl border border-slate-200 shadow-sm relative group">
                        <span class="absolute top-4 right-4 text-xs px-2 py-1 rounded font-bold border bg-blue-50 text-blue-700 border-blue-200">${a.estado}</span>
                        <h3 class="font-bold text-lg text-slate-800 mt-1 mb-1 pr-24">${a.nombre}</h3>
                        <p class="text-sm text-indigo-600 font-medium mb-3 flex items-center"><span class="mr-1">🗓️</span> ${a.fecha}</p>
                        <div class="text-xs text-slate-500 bg-slate-50 p-2 rounded"><strong>Comunas:</strong> ${a.comunas.join(', ')}</div>
                        <button onclick="window.eliminarEventoAgenda('${a.id}')" class="mt-3 text-xs text-red-500 hover:underline opacity-0 group-hover:opacity-100 transition">Eliminar</button>
                    </div>`;
            });
        };
        window.abrirModalAgenda = () => document.getElementById('modal-agenda').classList.remove('hidden');
        window.cerrarModalAgenda = () => document.getElementById('modal-agenda').classList.add('hidden');
        window.guardarEventoAgenda = async (e) => {
            e.preventDefault();
            if(!currentUser) return;
            const id = Date.now().toString();
            const comArr = document.getElementById('ev-comunas').value.split(',').map(s=>s.trim().toUpperCase());
            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'actividades', id), {
                id, nombre: document.getElementById('ev-nombre').value, fecha: document.getElementById('ev-fecha').value, estado: "💡 Propuesta", comunas: comArr
            });
            window.cerrarModalAgenda();
        };
        window.eliminarEventoAgenda = async (id) => {
            if(!currentUser) return;
            if(confirm("¿Eliminar reunión?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'actividades', id));
        };


        // ==============================================================================
        // --- LOGICA EVENTOS GENERALES CABA (NUEVO FLUJO CON ASISTENCIA) ---
        // ==============================================================================

        window.renderEventosGenerales = () => {
            const cont = document.getElementById('eventos-grales-container');
            cont.innerHTML = '';
            document.getElementById('kpi-eventos-grales').innerText = window.bd_eventos_generales.length;

            if(window.bd_eventos_generales.length === 0) {
                cont.classList.add('hidden'); document.getElementById('eventos-grales-vacia').classList.remove('hidden'); return;
            }
            cont.classList.remove('hidden'); document.getElementById('eventos-grales-vacia').classList.add('hidden');
            
            const ordenadas = [...window.bd_eventos_generales].sort((a,b) => new Date(b.fecha) - new Date(a.fecha));

            ordenadas.forEach(ev => {
                const isDone = ev.estado === "Realizado";
                const stClass = isDone ? "bg-emerald-100 text-emerald-800 border-emerald-300" : "bg-amber-100 text-amber-800 border-amber-300";
                
                // Fallback de retrocompatibilidad si existen datos viejos
                const esperados = ev.asistentes_esperados !== undefined ? ev.asistentes_esperados : (ev.asistentes_totales || 0);
                const reales = ev.asistentes_reales || 0;

                let performanceHTML = `<p class="text-sm text-slate-500 mt-2">👥 Meta: ${esperados} asistentes</p>`;

                if(isDone) {
                    const stars = ev.calificacion_general || 0;
                    const porcentaje = esperados > 0 ? Math.round((reales / esperados) * 100) : (reales > 0 ? 100 : 0);
                    
                    let barColor = "bg-rose-500";
                    let textColor = "text-rose-600";
                    if(porcentaje >= 100) { barColor = "bg-emerald-500"; textColor = "text-emerald-600"; }
                    else if(porcentaje >= 75) { barColor = "bg-amber-400"; textColor = "text-amber-600"; }

                    performanceHTML = `
                        <div class="mt-3 p-3 bg-slate-50 rounded-lg border border-slate-100">
                            <div class="flex justify-between items-end mb-1 text-xs font-bold">
                                <span class="text-slate-500">Asistencia Real vs Esperada</span>
                                <span class="${textColor}">${reales} / ${esperados} (${porcentaje}%)</span>
                            </div>
                            <div class="w-full bg-slate-200 rounded-full h-2 mb-2">
                                <div class="${barColor} h-2 rounded-full transition-all duration-500" style="width: ${Math.min(porcentaje, 100)}%"></div>
                            </div>
                            <div class="text-lg flex justify-end">${'⭐'.repeat(stars)}${'✩'.repeat(5-stars)}</div>
                        </div>
                    `;
                }

                cont.innerHTML += `
                    <div class="bg-white p-5 rounded-xl border border-slate-200 shadow-sm relative hover:shadow-md transition cursor-pointer flex flex-col" onclick="window.abrirModalEventoGeneral('${ev.id}')">
                        <span class="absolute top-4 right-4 text-xs px-2 py-1 rounded-full font-bold border ${stClass}">${isDone ? '✅ Realizado' : '⏳ Pendiente'}</span>
                        <h3 class="font-bold text-lg text-slate-800 mt-1 mb-1 pr-24 truncate">${ev.nombre}</h3>
                        <p class="text-sm text-slate-500 font-medium flex items-center"><span class="mr-1">🗓️</span> ${ev.fecha}</p>
                        <div class="mt-auto pt-2">
                            ${performanceHTML}
                        </div>
                    </div>`;
            });
        };

        function getStarsHTML(idPrefix, currentRating, isInteractive) {
            let html = `<div class="flex text-2xl tracking-widest star-rating" data-prefix="${idPrefix}">`;
            for(let i=1; i<=5; i++) {
                const active = i <= currentRating ? 'star-active' : 'star-inactive';
                const clickAction = isInteractive ? `onclick="window.setLocalRating('${idPrefix}', ${i})"` : "";
                html += `<span class="${active}" data-val="${i}" ${clickAction}>★</span>`;
            }
            html += `</div>`;
            return html;
        }

        window.abrirModalEventoGeneral = (eventId = null) => {
            let ev;
            if(eventId) {
                ev = {...window.bd_eventos_generales.find(e => e.id === eventId)};
            } else {
                ev = { id: Date.now().toString(), nombre: "", fecha: "", estado: "Pendiente", asistentes_esperados: 0, asistentes_reales: 0, notas: "", calificacion_general: 0, participacion: {} };
            }
            window.currentEventGralEdit = ev;
            
            renderFormularioEventoGeneral();
            document.getElementById('modal-evento-general').classList.remove('hidden');
        };

        window.cerrarModalEventoGeneral = () => {
            document.getElementById('modal-evento-general').classList.add('hidden');
            window.currentEventGralEdit = null;
        };

        function renderFormularioEventoGeneral() {
            const ev = window.currentEventGralEdit;
            const container = document.getElementById('form-evento-gral-container');
            const footer = document.getElementById('footer-evento-gral');
            const isDone = ev.estado === "Realizado";

            // Fallback para propiedades antiguas
            const esperados = ev.asistentes_esperados !== undefined ? ev.asistentes_esperados : (ev.asistentes_totales || 0);

            let html = `
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 mb-6">
                    <div class="lg:col-span-2"><label class="text-xs font-bold text-slate-500">Nombre del Evento General</label>
                    <input type="text" id="evg-nombre" value="${ev.nombre}" class="w-full border p-2 rounded bg-slate-50 focus:bg-white outline-none focus:border-indigo-500" ${isDone?'readonly':''}></div>
                    
                    <div><label class="text-xs font-bold text-slate-500">Fecha</label>
                    <input type="date" id="evg-fecha" value="${ev.fecha}" class="w-full border p-2 rounded bg-slate-50 focus:bg-white outline-none focus:border-indigo-500" ${isDone?'readonly':''}></div>
                    
                    <div><label class="text-xs font-bold text-slate-500">Asistentes Esperados (Meta)</label>
                    <input type="number" id="evg-esperados" value="${esperados}" class="w-full border p-2 rounded ${isDone ? 'bg-slate-100 text-slate-500' : 'bg-white focus:border-indigo-500'} outline-none" ${isDone?'readonly':''}></div>
            `;

            if(isDone) {
                html += `
                    <div><label class="text-xs font-bold text-indigo-700">Asistentes Reales (Final)</label>
                    <input type="number" id="evg-reales" value="${ev.asistentes_reales || 0}" class="w-full border-2 border-indigo-300 p-2 rounded bg-indigo-50 focus:bg-white focus:border-indigo-600 outline-none text-indigo-900 font-bold"></div>
                `;
            }

            html += `
                    <div class="${isDone ? 'lg:col-span-1' : 'md:col-span-2 lg:col-span-3'}"><label class="text-xs font-bold text-slate-500">Notas / Resumen</label>
                    <textarea id="evg-notas" rows="1" class="w-full border p-2 rounded bg-slate-50 focus:bg-white outline-none focus:border-indigo-500">${ev.notas||""}</textarea></div>
                </div>
            `;

            if (isDone) {
                html += `
                <div class="border-t border-slate-200 pt-6 mt-2">
                    <div class="flex justify-between items-center mb-4">
                        <h4 class="font-bold text-lg text-emerald-900">✅ Calificación de Comunas</h4>
                        <div class="text-right">
                            <span class="text-xs font-bold text-slate-500 block">Calificación General del Evento</span>
                            ${getStarsHTML('gral', ev.calificacion_general || 0, true)}
                        </div>
                    </div>
                    <div class="bg-slate-50 border border-slate-200 rounded-lg overflow-hidden">
                        <div class="max-h-64 overflow-y-auto p-4 space-y-3">
                `;
                
                window.bd_comunas.forEach(c => {
                    const rating = ev.participacion[c.id] || 0;
                    const isChecked = rating > 0;
                    html += `
                        <div class="flex items-center justify-between p-3 bg-white border rounded shadow-sm ${isChecked ? 'border-indigo-200' : 'border-slate-100 opacity-60'}">
                            <div class="flex items-center gap-3">
                                <input type="checkbox" id="chk-${c.id}" ${isChecked?'checked':''} onchange="window.toggleParticipacion('${c.id}')" class="w-5 h-5 text-indigo-600 rounded">
                                <div>
                                    <span class="font-bold text-slate-800">${c.id}</span>
                                    <span class="text-xs text-slate-500 block">Delegado: ${c.delegados || 'S/D'}</span>
                                </div>
                            </div>
                            <div class="${isChecked ? 'opacity-100' : 'opacity-0 pointer-events-none'} transition-opacity">
                                ${getStarsHTML(`com-${c.id}`, rating, true)}
                            </div>
                        </div>
                    `;
                });

                html += `</div></div></div>`;
            }

            container.innerHTML = html;

            if (!isDone) {
                footer.innerHTML = `
                    <button type="button" onclick="window.confirmarMarcarRealizado()" class="px-5 py-2.5 bg-emerald-100 text-emerald-800 font-bold rounded-lg hover:bg-emerald-200 border border-emerald-300 flex items-center transition">
                        ✅ Marcar como Realizado y Calificar
                    </button>
                    <div class="flex gap-2">
                        <button onclick="window.cerrarModalEventoGeneral()" class="px-4 py-2 bg-slate-200 text-slate-700 rounded-lg hover:bg-slate-300 transition">Cerrar</button>
                        <button onclick="window.guardarEventoGeneral()" class="px-4 py-2 bg-indigo-600 text-white rounded-lg font-bold shadow-md hover:bg-indigo-700 transition">Guardar Evento</button>
                    </div>
                `;
            } else {
                footer.innerHTML = `
                    <span class="text-xs text-emerald-600 font-bold bg-emerald-100 px-3 py-1 rounded-full">Modo Evaluación Activo</span>
                    <div class="flex gap-2">
                        <button onclick="window.cerrarModalEventoGeneral()" class="px-4 py-2 bg-slate-200 text-slate-700 rounded-lg hover:bg-slate-300 transition">Cerrar</button>
                        <button onclick="window.guardarEventoGeneral()" class="px-4 py-2 bg-emerald-600 text-white rounded-lg font-bold shadow-md hover:bg-emerald-700 transition">Guardar Evaluaciones</button>
                    </div>
                `;
            }
        }

        window.setLocalRating = (prefix, val) => {
            if(!window.currentEventGralEdit) return;
            if(prefix === 'gral') {
                window.currentEventGralEdit.calificacion_general = val;
            } else {
                const comunaId = prefix.replace('com-', '');
                window.currentEventGralEdit.participacion[comunaId] = val;
            }
            renderFormularioEventoGeneral(); 
        };

        window.toggleParticipacion = (comunaId) => {
            const isChecked = document.getElementById(`chk-${comunaId}`).checked;
            if(isChecked) {
                window.currentEventGralEdit.participacion[comunaId] = 3; 
            } else {
                delete window.currentEventGralEdit.participacion[comunaId]; 
            }
            renderFormularioEventoGeneral();
        };

        window.confirmarMarcarRealizado = () => {
            if(!document.getElementById('evg-nombre').value) return alert("Ponle nombre al evento primero.");
            if(!document.getElementById('evg-esperados').value || document.getElementById('evg-esperados').value == 0) return alert("Por favor, ingresa una meta de 'Asistentes Esperados' antes de continuar.");
            document.getElementById('modal-confirmar-realizado').classList.remove('hidden');
        };

        window.ejecutarMarcarRealizado = () => {
            window.currentEventGralEdit.estado = "Realizado";
            document.getElementById('modal-confirmar-realizado').classList.add('hidden');
            renderFormularioEventoGeneral();
        };

        window.guardarEventoGeneral = async () => {
            if(!currentUser) return;
            const ev = window.currentEventGralEdit;
            
            ev.nombre = document.getElementById('evg-nombre').value || "Evento Sin Nombre";
            ev.fecha = document.getElementById('evg-fecha').value || new Date().toISOString().split('T')[0];
            ev.asistentes_esperados = parseInt(document.getElementById('evg-esperados').value) || 0;
            
            if(document.getElementById('evg-reales')) {
                ev.asistentes_reales = parseInt(document.getElementById('evg-reales').value) || 0;
            }

            ev.notas = document.getElementById('evg-notas').value || "";

            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'eventos_generales', ev.id), ev);
            window.cerrarModalEventoGeneral();
        };

        // ==============================================================================
        // --- ASISTENTE IA ---
        // ==============================================================================

        window.abrirModalAsistenteIA = () => {
            document.getElementById('ia-raw-text').value = '';
            document.getElementById('ia-error-msg').classList.add('hidden');
            document.getElementById('ia-success-msg').classList.add('hidden');
            document.getElementById('modal-asistente-ia').classList.remove('hidden');
        };
        window.cerrarModalAsistenteIA = () => document.getElementById('modal-asistente-ia').classList.add('hidden');

        async function fetchGeminiWithRetry(prompt, systemPrompt) {
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = { contents: [{ parts: [{ text: prompt }] }], systemInstruction: { parts: [{ text: systemPrompt }] }, generationConfig: { responseMimeType: "application/json" } };
            for (let i = 0; i < 3; i++) {
                try {
                    const response = await fetch(url, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                    const result = await response.json();
                    return result.candidates?.[0]?.content?.parts?.[0]?.text || null;
                } catch (error) { if (i === 2) throw error; await new Promise(r => setTimeout(r, 1000)); }
            }
        }

        window.procesarTextoConIA = async () => {
            const rawText = document.getElementById('ia-raw-text').value.trim();
            const btn = document.getElementById('btn-procesar-ia');
            if(!rawText) return;
            document.getElementById('ia-error-msg').classList.add('hidden');
            btn.disabled = true; btn.innerHTML = `Analizando...`;

            const listadoComunas = window.bd_comunas.map(c => `${c.id}: WP=${c.wp}, Activos=${c.activos}`).join(" | ");
            const fechaHoy = new Date().toISOString().split('T')[0];

            const systemPrompt = `Cerebro IA App Seguimiento Territorial. Genera JSON con acciones. Fecha hoy: ${fechaHoy}.
1. Actualizar comunas. Interpreta números relativos o absolutos.
2. Agendar reuniones (actividades_agenda).

Estructura estricta:
{
  "actualizaciones_comunas": [ {"id": "C15", "wp": 40, "activos": 15, "estado_raw": "Muy Activo"} ],
  "nuevos_eventos": [ {"nombre": "Mateada", "fecha": "2026-03-10", "comunas": ["C15"]} ]
}`;

            try {
                const response = await fetchGeminiWithRetry(`Contexto: ${listadoComunas}\nTexto: "${rawText}"`, systemPrompt);
                let jsonParsed = JSON.parse(response.replace(/```json/g, '').replace(/```/g, '').trim());
                let accionesUI = [];

                if(jsonParsed.actualizaciones_comunas) {
                    for(const update of jsonParsed.actualizaciones_comunas) {
                        const target = window.bd_comunas.find(c => c.id === update.id);
                        if(target) {
                            const nData = {...target, wp: update.wp ?? target.wp, activos: update.activos ?? target.activos};
                            if(update.estado_raw) {
                                const sMap = {"Muy Activo":"🟢", "Cuello Botella":"🟡", "Recién empezando":"🟡", "Desorganizado":"🔴", "Sin Datos":"⚪"};
                                nData.estado_raw = update.estado_raw; nData.estado = `${sMap[update.estado_raw]||"⚪"} ${update.estado_raw}`;
                            }
                            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'comunas', update.id), nData);
                            accionesUI.push(`Comuna ${update.id} actualizada.`);
                        }
                    }
                }

                if(jsonParsed.nuevos_eventos) {
                    for(const ev of jsonParsed.nuevos_eventos) {
                        const evId = Date.now().toString() + Math.random().toString(36).substring(7);
                        await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'actividades', evId), {
                            id: evId, nombre: ev.nombre, fecha: ev.fecha, estado: "💡 Propuesta", comunas: ev.comunas || []
                        });
                        accionesUI.push(`Reunión agendada: ${ev.nombre}.`);
                    }
                }

                document.getElementById('ia-resumen').innerHTML = accionesUI.map(a => `<li>${a}</li>`).join('');
                document.getElementById('ia-success-msg').classList.remove('hidden');
                document.getElementById('ia-raw-text').value = '';
            } catch (error) {
                document.getElementById('ia-error-msg').innerText = "Error en formato devuelto por IA. Intenta de nuevo.";
                document.getElementById('ia-error-msg').classList.remove('hidden');
            } finally {
                btn.disabled = false; btn.innerHTML = `Procesar con IA`;
            }
        };

        initApp();
    </script>
</body>
</html>
