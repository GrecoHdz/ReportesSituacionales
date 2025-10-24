<template>
  <div id="app" class="bg-gray-50 dark:bg-gray-900">
    <!-- Header -->
    <header class="bg-white dark:bg-gray-800 shadow-lg border-b border-gray-200 dark:border-gray-700 no-print">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-16">
          <div class="flex items-center space-x-3">
            <div class="w-12 h-12 bg-gradient-to-br from-primary to-purple-600 rounded-xl flex items-center justify-center shadow-lg">
              <svg class="w-7 h-7 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v4a2 2 0 01-2 2h-2a2 2 0 01-2-2z"/>
              </svg>
            </div>
            <div>
              <h1 class="text-xl font-bold text-gray-900 dark:text-white">Centro de Monitoreo Avanzado</h1>
              <p class="text-sm text-gray-500 dark:text-gray-400">Sistema Integrado de Reportes Situacionales</p>
            </div>
          </div>

          <div class="flex items-center space-x-4">
            <div class="text-right">
              <p class="text-sm font-medium text-gray-900 dark:text-white">{{ currentDateTime }}</p>
              <p v-if="activeShift" class="text-xs text-green-600 dark:text-green-400 flex items-center">
                <span class="w-2 h-2 bg-green-500 rounded-full mr-1 animate-pulse"></span>
                Turno Activo - {{ activeShift.supervisor }}
              </p>
              <p v-else class="text-xs text-red-600 dark:text-red-400 flex items-center">
                <span class="w-2 h-2 bg-red-500 rounded-full mr-1"></span>
                Sin turno activo
              </p>
            </div>
            <div class="flex items-center space-x-2 bg-gray-100 dark:bg-gray-700 rounded-lg px-3 py-2">
              <div class="camera-status camera-online" title="Cámaras Online"></div>
              <span class="text-xs text-gray-600 dark:text-gray-400 font-medium">{{ getTotalOnlineCameras() }}/{{ getTotalCameras() }}</span>
            </div>
          </div>
        </div>
      </div>
    </header>

    <!-- Modales -->
    <!-- Modal de Código de Supervisor para Turno -->
    <div v-if="showTurnCodeModal" class="modal">
      <div class="modal-content p-6 w-96">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Código del Supervisor de Operaciones</h2>
        <form @submit.prevent="startTurn">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Ingrese su código para acceder al turno
            </label>
            <input v-model="turnCode" type="text" required
                   placeholder="Código del supervisor"
                   class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
            <p v-if="turnCodeError" class="text-red-500 text-sm mt-1">{{ turnCodeError }}</p>
          </div>
          <div class="flex justify-end space-x-3">
            <button type="button" @click="showTurnCodeModal = false" class="btn-secondary">
              Cancelar
            </button>
            <button type="submit" class="btn-primary">
              Iniciar Turno
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Cerrar Turno -->
    <div v-if="showEndShiftModal" class="modal">
      <div class="modal-content p-6 w-96">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Cerrar Turno</h2>
        <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">
          ¿Está seguro que desea cerrar el turno? No podrá registrar más eventos hasta que inicie un nuevo turno.
        </p>
        <div class="mb-4">
          <p class="text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">
            Turno iniciado por: {{ activeShift?.supervisor }}
          </p>
          <p class="text-sm font-medium text-gray-700 dark:text-gray-300">
            Eventos registrados: {{ events.length }}
          </p>
        </div>
        <div class="flex justify-end space-x-3">
          <button @click="showEndShiftModal = false" class="btn-secondary">
            Cancelar
          </button>
          <button @click="endShift" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg">
            Cerrar Turno
          </button>
        </div>
      </div>
    </div>

    <!-- Modal de Código de Agente -->
    <div v-if="showAgentCodeModal" class="modal">
      <div class="modal-content p-6 w-96">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Identificación del Agente</h2>
        <form @submit.prevent="submitEventWithAgent">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Código del Agente
            </label>
            <input v-model="agentCode" type="text" required
                   placeholder="Ingrese su código"
                   class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
            <p v-if="agentCodeError" class="text-red-500 text-sm mt-1">{{ agentCodeError }}</p>
          </div>
          <div class="flex justify-end space-x-3">
            <button type="button" @click="cancelEventSubmission" class="btn-secondary">
              Cancelar
            </button>
            <button type="submit" class="btn-primary">
              Registrar
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Editar Tiempo de Respuesta -->
    <div v-if="showEditResponseTimeModal" class="modal">
      <div class="modal-content p-6 w-96">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Editar Tiempo de Respuesta</h2>
        <form @submit.prevent="updateEventResponseTime">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Evento: {{ editingEvent?.code }}
            </label>
            <p class="text-sm text-gray-600 dark:text-gray-400 mb-3">{{ editingEvent?.description }}</p>
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Tiempo de Respuesta (minutos)
            </label>
            <input v-model.number="editingEventResponseTime" type="number" min="0" required
                   placeholder="Tiempo en minutos"
                   class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
          </div>
          <div class="flex justify-end space-x-3">
            <button type="button" @click="showEditResponseTimeModal = false" class="btn-secondary">
              Cancelar
            </button>
            <button type="submit" class="btn-primary">
              Actualizar
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Mapa -->
    <div v-if="showMapModal" class="modal">
      <div class="modal-content p-6" style="width: 800px;">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Seleccionar Ubicación en el Mapa</h2>
        <div id="modal-map" style="height: 500px; border-radius: 8px;" class="mb-4"></div>
        <div v-if="tempCoordinates.lat && tempCoordinates.lng" class="mb-4 text-sm text-gray-600 dark:text-gray-400">
          <strong>Coordenadas seleccionadas:</strong> {{ tempCoordinates.lat.toFixed(6) }}, {{ tempCoordinates.lng.toFixed(6) }}
        </div>
        <div class="flex justify-end space-x-3">
          <button @click="closeMapModal" class="btn-secondary">
            Cancelar
          </button>
          <button @click="confirmMapSelection" class="btn-primary">
            Confirmar
          </button>
        </div>
      </div>
    </div>

    <!-- Modal de Mapa para Reportes Manuales -->
    <div v-if="showManualReportMapModal" class="modal">
      <div class="modal-content p-6" style="width: 800px;">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Seleccionar Ubicación para el Reporte</h2>
        <div id="manual-report-map" style="height: 500px; border-radius: 8px;" class="mb-4"></div>
        <div v-if="tempManualReportCoordinates.lat && tempManualReportCoordinates.lng" class="mb-4 text-sm text-gray-600 dark:text-gray-400">
          <strong>Coordenadas seleccionadas:</strong> {{ tempManualReportCoordinates.lat.toFixed(6) }}, {{ tempManualReportCoordinates.lng.toFixed(6) }}
        </div>
        <div class="flex justify-end space-x-3">
          <button @click="closeManualReportMapModal" class="btn-secondary">
            Cancelar
          </button>
          <button @click="confirmManualReportMapSelection" class="btn-primary">
            Confirmar
          </button>
        </div>
      </div>
    </div>

    <!-- Modal para Ver Evento en Mapa -->
    <div v-if="showEventMapModal" class="modal">
      <div class="modal-content p-6" style="width: 800px;">
        <h2 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Ubicación del Evento: {{ selectedEvent?.code }}</h2>
        <div id="event-map" style="height: 500px; border-radius: 8px;" class="mb-4"></div>
        <div v-if="selectedEvent" class="mb-4 text-sm text-gray-600 dark:text-gray-400">
          <strong>Ubicación:</strong> {{ getFullLocationText(selectedEvent.location) }}<br>
          <strong>Coordenadas:</strong> {{ selectedEvent.coordinates.lat?.toFixed(6) }}, {{ selectedEvent.coordinates.lng?.toFixed(6) }}
        </div>
        <div class="flex justify-end">
          <button @click="closeEventMapModal" class="btn-primary">
            Cerrar
          </button>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">
      <!-- Navigation Tabs -->
      <div class="mb-6 no-print">
        <nav class="flex space-x-1 bg-gray-100 dark:bg-gray-800 rounded-xl p-1">
          <button v-for="tab in tabs" :key="tab.id"
                  @click="setActiveTab(tab.id)"
                  :class="[
                    'flex-1 py-3 px-4 rounded-lg font-medium text-sm transition-all duration-300',
                    activeTab === tab.id
                      ? 'bg-white dark:bg-gray-700 text-primary shadow-md transform scale-105'
                      : 'text-gray-500 hover:text-gray-700 hover:bg-gray-50 dark:text-gray-400 dark:hover:text-gray-300 dark:hover:bg-gray-700'
                  ]">
            {{ tab.name }}
          </button>
        </nav>
      </div>

      <!-- Dashboard Tab -->
      <div v-if="activeTab === 'dashboard'" class="space-y-6">
        <!-- Date Filter and Print Controls -->
        <div class="form-section no-print">
          <div class="flex flex-wrap items-center justify-between gap-4">
            <div class="flex items-center space-x-4">
              <div class="input-group">
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Fecha Inicio</label>
                <input v-model="dashboardStartDate" type="date"
                       @change="updateDashboard"
                       class="px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
              </div>
              <div class="input-group">
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Fecha Fin</label>
                <input v-model="dashboardEndDate" type="date"
                       @change="updateDashboard"
                       class="px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
              </div>
              <div class="flex items-end">
                <button @click="resetDashboardDates" class="btn-secondary">
                  🔄 Todos
                </button>
              </div>
            </div>
            <div>
              <button @click="printDashboard" class="bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-lg font-medium transition-all duration-300 flex items-center space-x-2">
                <span>🖨</span>
                <span>Imprimir Dashboard</span>
              </button>
            </div>
          </div>
        </div>

        <!-- KPIs Overview -->
        <div id="dashboard-content">
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-6">
            <div class="bg-gradient-to-br from-blue-50 to-blue-100 dark:from-blue-900 dark:to-blue-800 rounded-xl shadow-lg p-6 border border-blue-200 dark:border-blue-700">
              <div class="flex items-center justify-between">
                <div>
                  <p class="text-sm font-medium text-blue-600 dark:text-blue-300">Eventos en Rango</p>
                  <p class="text-3xl font-bold text-blue-900 dark:text-blue-100">{{ getFilteredEvents().length }}</p>
                </div>
                <div class="w-12 h-12 bg-blue-500 rounded-full flex items-center justify-center">
                  <svg class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
                  </svg>
                </div>
              </div>
            </div>

            <div class="bg-gradient-to-br from-green-50 to-green-100 dark:from-green-900 dark:to-green-800 rounded-xl shadow-lg p-6 border border-green-200 dark:border-green-700">
              <div class="flex items-center justify-between">
                <div>
                  <p class="text-sm font-medium text-green-600 dark:text-green-300">Cámaras Activas</p>
                  <p class="text-3xl font-bold text-green-900 dark:text-green-100">{{ getTotalOnlineCameras() }}/{{ getTotalCameras() }}</p>
                </div>
                <div class="w-12 h-12 bg-green-500 rounded-full flex items-center justify-center">
                  <svg class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z"/>
                  </svg>
                </div>
              </div>
            </div>

            <div class="bg-gradient-to-br from-yellow-50 to-yellow-100 dark:from-yellow-900 dark:to-yellow-800 rounded-xl shadow-lg p-6 border border-yellow-200 dark:border-yellow-700">
              <div class="flex items-center justify-between">
                <div>
                  <p class="text-sm font-medium text-yellow-600 dark:text-yellow-300">Recursos Desplegados</p>
                  <p class="text-3xl font-bold text-yellow-900 dark:text-yellow-100">{{ getTotalResourcesDeployedFiltered() }}</p>
                </div>
                <div class="w-12 h-12 bg-yellow-500 rounded-full flex items-center justify-center">
                  <svg class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M13 6a3 3 0 11-6 0 3 3 0 016 0zM18 8a2 2 0 11-4 0 2 2 0 014 0zM14 15a4 4 0 00-8 0v3h8v-3z"/>
                  </svg>
                </div>
              </div>
            </div>

            <div class="bg-gradient-to-br from-purple-50 to-purple-100 dark:from-purple-900 dark:to-purple-800 rounded-xl shadow-lg p-6 border border-purple-200 dark:border-purple-700">
              <div class="flex items-center justify-between">
                <div>
                  <p class="text-sm font-medium text-purple-600 dark:text-purple-300">Tiempo Resp. Prom</p>
                  <p class="text-3xl font-bold text-purple-900 dark:text-purple-100">{{ getAverageResponseTimeFiltered() }} min</p>
                </div>
                <div class="w-12 h-12 bg-purple-500 rounded-full flex items-center justify-center">
                  <svg class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-12a1 1 0 10-2 0v4a1 1 0 00.293.707l2.828 2.829a1 1 0 101.415-1.415L11 9.586V6z"/>
                  </svg>
                </div>
              </div>
            </div>
          </div>

          <!-- Charts Row 1 -->
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-6 border border-gray-200 dark:border-gray-700">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4 flex items-center">
                <span class="w-8 h-8 bg-blue-100 dark:bg-blue-900 rounded-lg flex items-center justify-center mr-3">📊</span>
                Eventos por Tipo
              </h3>
              <div class="chart-container">
                <canvas id="eventsTypeChart"></canvas>
              </div>
            </div>
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-6 border border-gray-200 dark:border-gray-700">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4 flex items-center">
                <span class="w-8 h-8 bg-green-100 dark:bg-green-900 rounded-lg flex items-center justify-center mr-3">🏢</span>
                Distribución por Institución
              </h3>
              <div class="chart-container">
                <canvas id="institutionsChart"></canvas>
              </div>
            </div>
          </div>

          <!-- Charts Row 2 -->
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-6 border border-gray-200 dark:border-gray-700">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4 flex items-center">
                <span class="w-8 h-8 bg-orange-100 dark:bg-orange-900 rounded-lg flex items-center justify-center mr-3">📈</span>
                Eventos por Hora
              </h3>
              <div class="chart-container">
                <canvas id="hourlyChart"></canvas>
              </div>
            </div>
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-6 border border-gray-200 dark:border-gray-700">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4 flex items-center">
                <span class="w-8 h-8 bg-purple-100 dark:bg-purple-900 rounded-lg flex items-center justify-center mr-3">📍</span>
                Distribución por Comunidad
              </h3>
              <div class="chart-container">
                <canvas id="communitiesChart"></canvas>
              </div>
            </div>
          </div>

          <!-- Additional Stats -->
          <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-6 border border-gray-200 dark:border-gray-700">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-8 h-8 bg-red-100 dark:bg-red-900 rounded-lg flex items-center justify-center mr-3">📋</span>
              Resumen por Prioridad
            </h3>
            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
              <div class="text-center p-6 bg-gradient-to-br from-red-50 to-red-100 dark:from-red-900 dark:to-red-800 rounded-xl border border-red-200 dark:border-red-700">
                <div class="text-4xl font-bold text-red-600 dark:text-red-400">{{ getEventsByPriorityFiltered('critica') }}</div>
                <div class="text-sm text-red-600 dark:text-red-400 mt-2 font-medium">🔴 Críticos</div>
              </div>
              <div class="text-center p-6 bg-gradient-to-br from-orange-50 to-orange-100 dark:from-orange-900 dark:to-orange-800 rounded-xl border border-orange-200 dark:border-orange-700">
                <div class="text-4xl font-bold text-orange-600 dark:text-orange-400">{{ getEventsByPriorityFiltered('alta') }}</div>
                <div class="text-sm text-orange-600 dark:text-orange-400 mt-2 font-medium">🟠 Altos</div>
              </div>
              <div class="text-center p-6 bg-gradient-to-br from-yellow-50 to-yellow-100 dark:from-yellow-900 dark:to-yellow-800 rounded-xl border border-yellow-200 dark:border-yellow-700">
                <div class="text-4xl font-bold text-yellow-600 dark:text-yellow-400">{{ getEventsByPriorityFiltered('media') }}</div>
                <div class="text-sm text-yellow-600 dark:text-yellow-400 mt-2 font-medium">🟡 Medios</div>
              </div>
              <div class="text-center p-6 bg-gradient-to-br from-green-50 to-green-100 dark:from-green-900 dark:to-green-800 rounded-xl border border-green-200 dark:border-green-700">
                <div class="text-4xl font-bold text-green-600 dark:text-green-400">{{ getEventsByPriorityFiltered('baja') }}</div>
                <div class="text-sm text-green-600 dark:text-green-400 mt-2 font-medium">🟢 Bajos</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Turno Tab -->
      <div v-if="activeTab === 'turno'" class="space-y-6">
        <!-- Configuración inicial antes de activar turno -->
        <div v-if="!activeShift" class="space-y-6">
          <!-- Alert que se debe configurar primero -->
          <div class="bg-gradient-to-r from-yellow-50 to-orange-50 dark:from-yellow-900 dark:to-orange-900 border border-yellow-200 dark:border-yellow-800 rounded-xl p-6">
            <div class="flex items-center">
              <span class="text-4xl mr-4">⚠️</span>
              <div>
                <p class="text-yellow-800 dark:text-yellow-200 font-bold text-lg">Complete la configuración antes de iniciar turno</p>
                <p class="text-yellow-700 dark:text-yellow-300">Debe configurar recursos institucionales y sistema de videovigilancia.</p>
              </div>
            </div>
          </div>

          <!-- Sistema de Videovigilancia -->
          <div class="form-section">
            <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-10 h-10 bg-blue-100 dark:bg-blue-900 rounded-xl flex items-center justify-center mr-3">📹</span>
              Sistema de Videovigilancia por Comunidad
            </h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div v-for="community in communities" :key="community.id"
                   class="input-group border-l-4 border-l-blue-500">
                <div class="flex justify-between items-center mb-4">
                  <h3 class="text-lg font-semibold text-gray-900 dark:text-white flex items-center">
                    <span class="text-2xl mr-2">🏘️</span>
                    {{ community.name }}
                  </h3>
                  <div class="flex items-center space-x-4">
                    <div class="text-sm text-gray-600 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">
                      <span class="camera-status camera-online"></span>
                      {{ community.cameras.online }}/{{ community.cameras.total }} activas
                    </div>
                  </div>
                </div>
                <div class="grid grid-cols-2 gap-4">
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Total de Cámaras</label>
                    <input v-model.number="community.cameras.total" type="number" min="0"
                           class="w-full px-4 py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Cámaras Activas</label>
                    <input v-model.number="community.cameras.online" type="number" min="0" :max="community.cameras.total"
                           class="w-full px-4 py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Recursos Institucionales -->
          <div class="form-section">
            <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-10 h-10 bg-green-100 dark:bg-green-900 rounded-xl flex items-center justify-center mr-3">🚓</span>
              Recursos Institucionales Disponibles
            </h2>
            <div class="space-y-6">
              <div v-for="institution in institutions" :key="institution.id"
                   class="input-group border-l-4 border-l-green-500">
                <div class="flex items-center mb-6">
                  <span class="text-3xl mr-4">{{ institution.icon }}</span>
                  <div class="flex-1">
                    <h3 class="text-xl font-semibold text-gray-900 dark:text-white">{{ institution.name }}</h3>
                    <p class="text-sm text-gray-600 dark:text-gray-400">Recursos disponibles para despliegue</p>
                  </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                  <div v-for="resource in institution.resources" :key="resource.id" class="bg-gray-50 dark:bg-gray-600 p-4 rounded-lg">
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">{{ resource.name }}</label>
                    <input v-model.number="resource.available" type="number" min="0"
                           placeholder="Disponibles"
                           class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Botón para iniciar turno -->
          <div class="form-section">
            <div class="flex justify-center">
              <button @click="validateAndInitiateTurn" :disabled="!canInitiateTurn()" 
                      :class="canInitiateTurn() ? 'btn-primary' : 'btn-secondary opacity-50 cursor-not-allowed'"
                      class="text-lg py-4 px-8">
                ▶ Iniciar Turno
              </button>
            </div>
            <div v-if="!canInitiateTurn()" class="text-center mt-4">
              <p class="text-red-600 dark:text-red-400 text-sm">
                Debe completar la configuración de recursos y cámaras antes de iniciar el turno
              </p>
            </div>
          </div>
        </div>

        <!-- Configuración del Turno (solo visible cuando hay turno activo) -->
        <div v-if="activeShift" class="space-y-6">
          <!-- Información del Turno Activo -->
          <div class="form-section">
            <div class="flex justify-between items-center mb-6">
              <h2 class="text-2xl font-bold text-gray-900 dark:text-white flex items-center">
                <span class="w-10 h-10 bg-green-100 dark:bg-green-900 rounded-xl flex items-center justify-center mr-3">👮</span>
                Turno Activo
              </h2>
              <button @click="showEndShiftModal = true" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg font-medium transition-all duration-300 flex items-center space-x-2">
                <span>⏹</span>
                <span>Cerrar Turno</span>
              </button>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
              <div class="bg-blue-50 dark:bg-blue-900/20 p-4 rounded-lg">
                <h4 class="font-bold text-blue-900 dark:text-blue-200 mb-2">👤 Supervisor</h4>
                <p class="text-blue-800 dark:text-blue-300">{{ activeShift.supervisor }}</p>
              </div>
              <div class="bg-green-50 dark:bg-green-900/20 p-4 rounded-lg">
                <h4 class="font-bold text-green-900 dark:text-green-200 mb-2">⏰ Inicio</h4>
                <p class="text-green-800 dark:text-green-300">{{ activeShift.startTime }}</p>
              </div>
              <div class="bg-purple-50 dark:bg-purple-900/20 p-4 rounded-lg">
                <h4 class="font-bold text-purple-900 dark:text-purple-200 mb-2">📊 Eventos</h4>
                <p class="text-purple-800 dark:text-purple-300">{{ events.length }}</p>
              </div>
            </div>
          </div>

          <!-- Sistema de Videovigilancia (modo visualización) -->
          <div class="form-section">
            <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-10 h-10 bg-blue-100 dark:bg-blue-900 rounded-xl flex items-center justify-center mr-3">📹</span>
              Sistema de Videovigilancia - Estado Actual
            </h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div v-for="community in communities" :key="community.id"
                   class="input-group border-l-4 border-l-blue-500">
                <div class="flex justify-between items-center mb-4">
                  <h3 class="text-lg font-semibold text-gray-900 dark:text-white flex items-center">
                    <span class="text-2xl mr-2">🏘️</span>
                    {{ community.name }}
                  </h3>
                  <div class="flex items-center space-x-4">
                    <div class="text-sm text-gray-600 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">
                      <span class="camera-status camera-online"></span>
                      {{ community.cameras.online }}/{{ community.cameras.total }} activas
                    </div>
                  </div>
                </div>
                <div class="grid grid-cols-2 gap-4">
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Total de Cámaras</label>
                    <input v-model.number="community.cameras.total" type="number" min="0"
                           class="w-full px-4 py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Cámaras Activas</label>
                    <input v-model.number="community.cameras.online" type="number" min="0" :max="community.cameras.total"
                           class="w-full px-4 py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Recursos Institucionales (modo visualización) -->
          <div class="form-section">
            <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-10 h-10 bg-green-100 dark:bg-green-900 rounded-xl flex items-center justify-center mr-3">🚓</span>
              Recursos Institucionales - Estado Actual
            </h2>
            <div class="space-y-6">
              <div v-for="institution in institutions" :key="institution.id"
                   class="input-group border-l-4 border-l-green-500">
                <div class="flex items-center mb-6">
                  <span class="text-3xl mr-4">{{ institution.icon }}</span>
                  <div class="flex-1">
                    <h3 class="text-xl font-semibold text-gray-900 dark:text-white">{{ institution.name }}</h3>
                    <p class="text-sm text-gray-600 dark:text-gray-400">Recursos disponibles para despliegue</p>
                  </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                  <div v-for="resource in institution.resources" :key="resource.id" class="bg-gray-50 dark:bg-gray-600 p-4 rounded-lg">
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">{{ resource.name }}</label>
                    <input v-model.number="resource.available" type="number" min="0"
                           placeholder="Disponibles"
                           class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Eventos Tab -->
      <div v-if="activeTab === 'events'" class="space-y-6">
        <!-- Alert if no active shift -->
        <div v-if="!activeShift" class="bg-gradient-to-r from-yellow-50 to-orange-50 dark:from-yellow-900 dark:to-orange-900 border border-yellow-200 dark:border-yellow-800 rounded-xl p-6">
          <div class="flex items-center">
            <span class="text-4xl mr-4">⚠️</span>
            <div>
              <p class="text-yellow-800 dark:text-yellow-200 font-bold text-lg">No hay un turno activo</p>
              <p class="text-yellow-700 dark:text-yellow-300">Debe iniciar un turno para registrar eventos.</p>
            </div>
          </div>
        </div>

        <!-- Agregar Evento -->
        <div v-else class="form-section">
          <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
            <span class="w-10 h-10 bg-primary rounded-xl flex items-center justify-center mr-3">📝</span>
            Registrar Nuevo Evento
          </h2>
          <form @submit.prevent="requestAgentCode" class="space-y-6">
            <!-- Información Básica -->
            <div class="input-group">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">ℹ️ Información Básica</h3>
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Evento</label>
                  <select v-model="newEvent.type" required @change="generateEventCode"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="">Seleccionar tipo</option>
                    <option v-for="type in eventTypes" :key="type.id" :value="type.id">{{ type.name }}</option>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Código (Automático)</label>
                  <input v-model="newEvent.code" type="text" readonly
                         class="w-full px-3 py-2 text-base bg-gray-100 dark:bg-gray-600 border border-gray-300 dark:border-gray-600 rounded-lg dark:text-white">
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Prioridad</label>
                  <select v-model="newEvent.priority"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="baja">🟢 Baja</option>
                    <option value="media">🟡 Media</option>
                    <option value="alta">🟠 Alta</option>
                    <option value="critica">🔴 Crítica</option>
                  </select>
                </div>
              </div>
            </div>

            <!-- Ubicación -->
            <div class="input-group">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">📍 Ubicación del Evento</h3>
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Comunidad</label>
                  <select v-model="newEvent.location.community" @change="onCommunityChange" required
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="">Seleccionar comunidad</option>
                    <option v-for="community in communities" :key="community.id" :value="community.id">{{ community.name }}</option>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Barrio/Colonia</label>
                  <select v-model="newEvent.location.neighborhood" :disabled="!newEvent.location.community"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white disabled:opacity-50">
                    <option value="">Seleccionar barrio</option>
                    <option v-for="neighborhood in availableNeighborhoods" :key="neighborhood" :value="neighborhood">{{ neighborhood }}</option>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Referencia Cercana</label>
                  <input v-model="newEvent.location.reference" type="text"
                         placeholder="Ej: Cerca del parque central"
                         class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                </div>
              </div>
              <div class="flex items-center justify-between">
                <button type="button" @click="openMapModal" class="btn-primary">
                  🗺 Seleccionar en el Mapa
                </button>
                <div v-if="newEvent.coordinates.lat && newEvent.coordinates.lng" class="text-sm text-gray-600 dark:text-gray-400 bg-green-100 dark:bg-green-900 px-3 py-2 rounded-lg">
                  <strong>Coordenadas:</strong> {{ newEvent.coordinates.lat.toFixed(6) }}, {{ newEvent.coordinates.lng.toFixed(6) }}
                </div>
              </div>
            </div>

            <!-- Instituciones y Recursos -->
            <div class="input-group">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">🚓 Instituciones y Recursos</h3>
              <div class="space-y-4">
                <div v-for="institution in institutions" :key="institution.id"
                     class="bg-gray-50 dark:bg-gray-600 rounded-lg p-4">
                  <div class="flex items-center justify-between mb-3">
                    <label class="text-lg font-medium text-gray-700 dark:text-gray-300 flex items-center cursor-pointer">
                      <input type="checkbox" v-model="newEvent.institutionsUsed[institution.id]"
                             class="mr-3 w-5 h-5 rounded border-gray-300 text-primary focus:ring-primary">
                      <span class="text-2xl mr-3">{{ institution.icon }}</span>
                      {{ institution.name }}
                    </label>
                    <div v-if="newEvent.institutionsUsed[institution.id]" class="text-sm text-gray-600 dark:text-gray-400">
                      <label class="block text-xs font-medium mb-1">T. Resp. Estimado (min)</label>
                      <input v-model.number="newEvent.responseTimeEstimated[institution.id]" type="number" min="0"
                             :placeholder="institution.responseTime"
                             class="w-20 px-2 py-1 text-base border border-gray-300 dark:border-gray-600 rounded focus:ring-1 focus:ring-primary dark:bg-gray-700 dark:text-white">
                    </div>
                  </div>
                  <div v-if="newEvent.institutionsUsed[institution.id]" class="ml-8">
                    <label class="block text-sm font-medium text-gray-600 dark:text-gray-400 mb-3">Recursos Utilizados:</label>
                    <div class="space-y-2">
                      <div v-for="resource in institution.resources" :key="resource.id" class="flex items-center space-x-3">
                        <input type="checkbox"
                               v-model="newEvent.resourcesUsed[institution.id + '_' + resource.id]"
                               :id="'res_' + institution.id + '_' + resource.id"
                               class="w-4 h-4 rounded border-gray-300 text-primary focus:ring-primary">
                        <label :for="'res_' + institution.id + '_' + resource.id" class="text-sm flex-1 font-medium">
                          {{ resource.name }}
                        </label>
                        <input v-if="newEvent.resourcesUsed[institution.id + '_' + resource.id]"
                               v-model="newEvent.resourceSpecifications[institution.id + '_' + resource.id]"
                               type="text"
                               placeholder="Especificación"
                               class="w-48 px-3 py-1 text-sm border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-1 focus:ring-primary dark:bg-gray-700 dark:text-white">
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Afectaciones -->
            <div class="input-group">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">📊 Afectaciones</h3>
              <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                <div v-for="impact in impactTypes" :key="impact.id" class="bg-gray-50 dark:bg-gray-600 p-3 rounded-lg">
                  <div class="flex items-center mb-2">
                    <input type="checkbox"
                           v-model="newEvent.impacts[impact.id]"
                           :id="'impact_' + impact.id"
                           class="mr-3 w-4 h-4 rounded border-gray-300 text-primary focus:ring-primary">
                    <label :for="'impact_' + impact.id" class="text-sm font-medium text-gray-700 dark:text-gray-300">
                      {{ impact.label }}
                    </label>
                  </div>
                  <input v-if="newEvent.impacts[impact.id]"
                         v-model="newEvent.impactDetails[impact.id]"
                         type="text"
                         :placeholder="impact.placeholder"
                         class="w-full px-3 py-2 text-sm border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                </div>
              </div>
            </div>

            <button type="submit" class="btn-primary w-full md:w-auto text-lg py-4 px-8">
              📝 Registrar Evento
            </button>
          </form>
        </div>

        <!-- Lista de Eventos del Turno Actual -->
        <div class="form-section">
          <div class="flex justify-between items-center mb-6">
            <h2 class="text-2xl font-bold text-gray-900 dark:text-white flex items-center">
              <span class="w-10 h-10 bg-orange-100 dark:bg-orange-900 rounded-xl flex items-center justify-center mr-3">📋</span>
              Eventos del Turno Actual
            </h2>
            <div class="text-sm text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-700 px-4 py-2 rounded-lg">
              Total: {{ currentShiftEvents.length }} eventos
            </div>
          </div>

          <div v-if="currentShiftEvents.length === 0" class="text-center py-12 text-gray-500 dark:text-gray-400">
            <div class="text-6xl mb-4">📊</div>
            <p class="text-lg">No hay eventos registrados en este turno</p>
            <p class="text-sm mt-2">Los eventos aparecerán aquí una vez que sean registrados</p>
          </div>

          <div v-else class="space-y-4">
            <div v-for="(event, index) in currentShiftEvents" :key="index"
                 class="bg-white dark:bg-gray-700 border border-gray-200 dark:border-gray-600 rounded-xl p-6 hover:shadow-lg transition-all duration-300">
              <div class="flex justify-between items-start mb-4">
                <div class="flex items-center space-x-4">
                  <span class="text-lg font-bold text-primary bg-blue-100 dark:bg-blue-900 px-3 py-1 rounded-lg">{{ event.code }}</span>
                  <span class="text-sm font-medium text-gray-900 dark:text-white bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ event.time }}</span>
                  <span :class="getPriorityBadgeClass(event.priority)"
                        class="px-3 py-1 text-sm font-medium rounded-lg">
                    {{ event.priority.toUpperCase() }}
                  </span>
                  <span class="text-sm text-gray-600 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ getEventTypeName(event.type) }}</span>
                </div>
                <div class="flex items-center space-x-3">
                  <button @click="editEventResponseTime(event)"
                          class="px-3 py-2 text-sm bg-blue-100 dark:bg-blue-900 text-blue-700 dark:text-blue-300 rounded-lg hover:bg-blue-200 dark:hover:bg-blue-800 transition-colors flex items-center space-x-2">
                    <span>⏱</span>
                    <span>T. Respuesta: {{ event.actualResponseTime || 'No definido' }} min</span>
                  </button>
                  <div class="text-xs text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-2 py-1 rounded">
                    Por: {{ event.registeredBy }}
                  </div>
                </div>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm text-gray-500 dark:text-gray-400">
                <div class="bg-gray-50 dark:bg-gray-600 p-3 rounded-lg">
                  <strong class="text-gray-700 dark:text-gray-300">📍 Ubicación:</strong><br>
                  {{ getFullLocationText(event.location) }}
                </div>
                <div v-if="hasImpacts(event)" class="bg-gray-50 dark:bg-gray-600 p-3 rounded-lg">
                  <strong class="text-gray-700 dark:text-gray-300">📊 Afectaciones:</strong><br>
                  {{ getImpactsText(event) }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Historial de Eventos Tab -->
      <div v-if="activeTab === 'historial'" class="space-y-6">
        <!-- Filtros -->
        <div class="form-section">
          <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
            <span class="w-10 h-10 bg-indigo-100 dark:bg-indigo-900 rounded-xl flex items-center justify-center mr-3">📚</span>
            Historial de Eventos
          </h2>
          
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Fecha Inicio</label>
              <input v-model="historialFilters.startDate" type="date"
                     class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Fecha Fin</label>
              <input v-model="historialFilters.endDate" type="date"
                     class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Evento</label>
              <select v-model="historialFilters.eventType"
                      class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                <option value="">Todos los tipos</option>
                <option v-for="type in eventTypes" :key="type.id" :value="type.id">{{ type.name }}</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Prioridad</label>
              <select v-model="historialFilters.priority"
                      class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                <option value="">Todas las prioridades</option>
                <option value="baja">🟢 Baja</option>
                <option value="media">🟡 Media</option>
                <option value="alta">🟠 Alta</option>
                <option value="critica">🔴 Crítica</option>
              </select>
            </div>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Comunidad</label>
              <select v-model="historialFilters.community"
                      class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                <option value="">Todas las comunidades</option>
                <option v-for="community in communities" :key="community.id" :value="community.id">{{ community.name }}</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Turno</label>
              <select v-model="historialFilters.shift"
                      class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                <option value="">Todos los turnos</option>
                <option v-if="activeShift" value="current">Turno Actual</option>
                <option v-for="(shift, index) in closedShifts" :key="index" :value="`shift_${index}`">
                  {{ shift.supervisor }} - {{ shift.startTime }}
                </option>
              </select>
            </div>
            <div class="flex items-end">
              <button @click="clearHistorialFilters" class="btn-secondary w-full">
                🔄 Limpiar Filtros
              </button>
            </div>
          </div>
        </div>

        <!-- Resumen de Filtros -->
        <div class="bg-blue-50 dark:bg-blue-900/20 rounded-lg p-4 border border-blue-200 dark:border-blue-800">
          <div class="flex justify-between items-center">
            <div>
              <h3 class="font-semibold text-blue-900 dark:text-blue-200">Resultados del Filtro</h3>
              <p class="text-sm text-blue-700 dark:text-blue-300">
                Mostrando {{ getFilteredHistorialEvents().length }} de {{ allEvents.length }} eventos totales
              </p>
            </div>
            <div class="flex space-x-4 text-sm">
              <div class="text-center">
                <p class="font-bold text-blue-900 dark:text-blue-200">{{ getFilteredEventsByPriority('critica') }}</p>
                <p class="text-blue-700 dark:text-blue-300">Críticos</p>
              </div>
              <div class="text-center">
                <p class="font-bold text-blue-900 dark:text-blue-200">{{ getFilteredEventsByPriority('alta') }}</p>
                <p class="text-blue-700 dark:text-blue-300">Altos</p>
              </div>
              <div class="text-center">
                <p class="font-bold text-blue-900 dark:text-blue-200">{{ getFilteredEventsByPriority('media') }}</p>
                <p class="text-blue-700 dark:text-blue-300">Medios</p>
              </div>
              <div class="text-center">
                <p class="font-bold text-blue-900 dark:text-blue-200">{{ getFilteredEventsByPriority('baja') }}</p>
                <p class="text-blue-700 dark:text-blue-300">Bajos</p>
              </div>
            </div>
          </div>
        </div>

        <!-- Lista de Eventos Filtrados -->
        <div class="form-section">
          <div v-if="getFilteredHistorialEvents().length === 0" class="text-center py-12 text-gray-500 dark:text-gray-400">
            <div class="text-6xl mb-4">🔍</div>
            <p class="text-lg">No se encontraron eventos con los filtros aplicados</p>
            <p class="text-sm mt-2">Intente ajustar los criterios de búsqueda</p>
          </div>

          <div v-else class="space-y-4">
            <div v-for="(event, index) in getFilteredHistorialEvents()" :key="`historial_${index}`"
                 class="bg-white dark:bg-gray-700 border border-gray-200 dark:border-gray-600 rounded-xl p-6 hover:shadow-lg transition-all duration-300">
              <div class="flex justify-between items-start mb-4">
                <div class="flex items-center space-x-4">
                  <span class="text-lg font-bold text-primary bg-blue-100 dark:bg-blue-900 px-3 py-1 rounded-lg">{{ event.code }}</span>
                  <span class="text-sm font-medium text-gray-900 dark:text-white bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ event.date }} - {{ event.time }}</span>
                  <span :class="getPriorityBadgeClass(event.priority)"
                        class="px-3 py-1 text-sm font-medium rounded-lg">
                    {{ event.priority.toUpperCase() }}
                  </span>
                  <span class="text-sm text-gray-600 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ getEventTypeName(event.type) }}</span>
                </div>
                <div class="flex items-center space-x-3">
                  <button v-if="event.coordinates.lat && event.coordinates.lng" @click="showEventOnMap(event)"
                          class="px-3 py-2 text-sm bg-green-100 dark:bg-green-900 text-green-700 dark:text-green-300 rounded-lg hover:bg-green-200 dark:hover:bg-green-800 transition-colors flex items-center space-x-2">
                    <span>🗺</span>
                    <span>Ver en Mapa</span>
                  </button>
                  <div class="text-sm text-blue-600 dark:text-blue-400 bg-blue-100 dark:bg-blue-900 px-3 py-1 rounded-lg">
                    T. Respuesta: {{ event.actualResponseTime || 'N/A' }} min
                  </div>
                  <div class="text-xs text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-2 py-1 rounded">
                    Por: {{ event.registeredBy }}
                  </div>
                  <div class="text-xs text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-2 py-1 rounded">
                    Turno: {{ event.shiftSupervisor || 'N/A' }}
                  </div>
                </div>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm text-gray-500 dark:text-gray-400">
                <div class="bg-gray-50 dark:bg-gray-600 p-3 rounded-lg">
                  <strong class="text-gray-700 dark:text-gray-300">📍 Ubicación:</strong><br>
                  {{ getFullLocationText(event.location) }}
                  <div v-if="event.coordinates.lat && event.coordinates.lng" class="mt-2 text-xs">
                    <strong>Coordenadas:</strong> {{ event.coordinates.lat.toFixed(6) }}, {{ event.coordinates.lng.toFixed(6) }}
                  </div>
                </div>
                <div v-if="hasImpacts(event)" class="bg-gray-50 dark:bg-gray-600 p-3 rounded-lg">
                  <strong class="text-gray-700 dark:text-gray-300">📊 Afectaciones:</strong><br>
                  {{ getImpactsText(event) }}
                </div>
              </div>

              <!-- Instituciones Participantes -->
              <div v-if="getEventInstitutions(event).length > 0" class="mt-4 pt-4 border-t border-gray-200 dark:border-gray-600">
                <strong class="text-gray-700 dark:text-gray-300 text-sm">🏢 Instituciones Participantes:</strong>
                <div class="flex flex-wrap gap-2 mt-2">
                  <span v-for="institution in getEventInstitutions(event)" :key="institution.id"
                        class="text-xs bg-purple-100 dark:bg-purple-900 text-purple-800 dark:text-purple-200 px-2 py-1 rounded-lg flex items-center">
                    <span class="mr-1">{{ institution.icon }}</span>
                    {{ institution.name }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Reportes Tab -->
      <div v-if="activeTab === 'reportes'" class="space-y-6">
        <!-- Selector de Tipo de Reporte -->
        <div class="form-section no-print">
          <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
            <span class="w-10 h-10 bg-purple-100 dark:bg-purple-900 rounded-xl flex items-center justify-center mr-3">📊</span>
            Generar Reportes
          </h2>

          <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
            <!-- Reportes Automáticos -->
            <div class="report-type-card" :class="{ active: reportCategory === 'automatic' }" @click="reportCategory = 'automatic'">
              <div class="text-center">
                <div class="text-4xl mb-3">🤖</div>
                <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-2">Reportes Automáticos</h3>
                <p class="text-gray-600 dark:text-gray-400 text-sm">Generados con datos de eventos del sistema</p>
                <ul class="mt-3 text-sm text-gray-500 dark:text-gray-500 text-left">
                  <li>• Cierre de turno</li>
                  <li>• Análisis distribucional</li>
                  <li>• Estadísticas operacionales</li>
                </ul>
              </div>
            </div>

            <!-- Reportes Manuales -->
            <div class="report-type-card" :class="{ active: reportCategory === 'manual' }" @click="reportCategory = 'manual'">
              <div class="text-center">
                <div class="text-4xl mb-3">📝</div>
                <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-2">Reportes Manuales</h3>
                <p class="text-gray-600 dark:text-gray-400 text-sm">Para comunicación con la población</p>
                <ul class="mt-3 text-sm text-gray-500 dark:text-gray-500 text-left">
                  <li>• Alertas meteorológicas</li>
                  <li>• Avisos de seguridad</li>
                  <li>• Comunicados oficiales</li>
                </ul>
              </div>
            </div>
          </div>

          <!-- Reportes Automáticos -->
          <div v-if="reportCategory === 'automatic'" class="input-group">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">🤖 Reportes Automáticos del Sistema</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Reporte</label>
                <select v-model="selectedReportType"
                        class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                  <option value="cierre">📊 Cierre de Turno</option>
                  <option value="distribucional">📋 Distribucional Territorial</option>
                </select>
              </div>
              <div class="flex items-end">
                <button @click="generateAutomaticReport" class="btn-primary w-full">
                  📊 Generar Reporte
                </button>
              </div>
            </div>
          </div>

          <!-- Reportes Manuales -->
          <div v-if="reportCategory === 'manual'" class="input-group">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">📝 Reportes Manuales para la Población</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Comunicado</label>
                <select v-model="selectedManualReportType"
                        class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                  <option value="weather">🌤 Alerta Meteorológica</option>
                  <option value="security">🚨 Aviso de Seguridad</option>
                  <option value="traffic">🚦 Aviso de Tránsito</option>
                  <option value="public_service">🔧 Interrupción de Servicio</option>
                </select>
              </div>
              <div class="flex items-end">
                <button @click="showManualReportForm = true" class="btn-primary w-full">
                  📝 Crear Comunicado
                </button>
              </div>
            </div>
          </div>

          <!-- Formulario de Reporte Manual -->
          <div v-if="showManualReportForm" class="input-group border-l-4 border-l-orange-500">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">
              📝 {{ getManualReportTitle() }}
            </h3>

           <!-- Formulario según tipo -->
<div v-if="selectedManualReportType === 'weather'" class="space-y-4">
  <!-- Tipo de alerta y riesgo -->
  <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
    <div>
      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Alerta</label>
      <select v-model="manualReport.weather.alertType"
              class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
        <option value="lluvia">🌧️ Lluvia</option>
        <option value="tormenta">⛈️ Tormenta Eléctrica</option>
        <option value="viento">💨 Vientos Fuertes</option>
        <option value="granizo">🧊 Granizo</option>
        <option value="calor">🌡️ Altas Temperaturas</option>
      </select>
    </div>
    <div>
      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Nivel de Riesgo</label>
      <select v-model="manualReport.weather.riskLevel"
              class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
        <option value="blanco">⚪ Sin Riesgo</option>
        <option value="verde">🟢 Verde - Riesgo Bajo</option>
        <option value="amarillo">🟡 Amarillo - Precaución</option> 
        <option value="rojo">🔴 Rojo - Alto Riesgo</option>
      </select>
    </div>
  </div>

  <!-- Zonas afectadas -->
  <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
    <div>
      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Zona Afectada</label>
      <div class="space-y-2">
        <div class="flex items-center">
          <input type="checkbox" value="municipio" v-model="manualReport.weather.affectedZones"
                 class="h-4 w-4 text-primary border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600" />
          <label class="ml-2 text-sm text-gray-700 dark:text-gray-300">Todo el municipio</label>
        </div>

        <!-- Comunidades: 2 por fila -->
        <div class="grid grid-cols-2 gap-x-4 gap-y-2">
          <div v-for="community in communities" :key="community.id" class="flex items-center">
            <input type="checkbox" :value="community.id" v-model="manualReport.weather.affectedZones"
                   class="h-4 w-4 text-primary border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600" />
            <label class="ml-2 text-sm text-gray-700 dark:text-gray-300">{{ community.name }}</label>
          </div>
        </div>
      </div>
    </div>

    <!-- Datos adicionales -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div>
        <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Duración Estimada</label>
        <input v-model="manualReport.weather.duration" type="text"
               placeholder="Ej: 2-4 horas"
               class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
      </div>

      <div v-if="manualReport.weather.alertType === 'lluvia'">
        <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Intensidad</label>
        <select v-model="manualReport.weather.intensity"
                class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
          <option value="ligera">Ligera (&lt; 10 mm/h)</option>
          <option value="moderada">Moderada (10–25 mm/h)</option>
          <option value="fuerte">Fuerte (25–50 mm/h)</option>
          <option value="intensa">Intensa (&gt; 50 mm/h)</option>
        </select>
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tiempo de Impacto Estimado</label>
        <input v-model="manualReport.weather.impactTime" type="text"
               placeholder="Ej: 30 minutos - 1 hora"
               class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
      </div> 

      <div>
        <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Probabilidad del Evento</label>
        <select v-model="manualReport.weather.probability"
                class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
          <option value="baja">Baja (≤30%)</option>
          <option value="media">Media (31–70%)</option>
          <option value="alta">Alta (&gt;70%)</option>
        </select>
      </div>
    </div>
  </div> 
</div>


            <!-- Otros tipos de reportes manuales -->
            <div v-if="selectedManualReportType === 'security'" class="space-y-4">
              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Aviso</label>
                  <select v-model="manualReport.security.alertType"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="preventivo">🛡️ Aviso Preventivo</option>
                    <option value="evacuacion">🚨 Evacuación</option>
                    <option value="restriccion">⛔ Restricción de Acceso</option>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Zona Afectada</label>
                  <select v-model="manualReport.security.affectedZone"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option v-for="community in communities" :key="community.id" :value="community.id">{{ community.name }}</option>
                  </select>
                </div>
              </div>
            </div>

            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Descripción Detallada</label>
                <textarea v-model="manualReport.description" rows="4"
                          placeholder="Describa la situación, medidas a tomar y recomendaciones..."
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white"></textarea>
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Recomendaciones para la Población</label>
                <textarea v-model="manualReport.recommendations" rows="3"
                          placeholder="Instrucciones específicas para los ciudadanos..."
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white"></textarea>
              </div> 

              <!-- 📸 Subir fotos -->
  <div>
    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Anexo</label>
    <input type="file" multiple accept="image/*" @change="handleImageUpload"
           class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer dark:text-gray-300 dark:bg-gray-700 dark:border-gray-600 focus:outline-none" />
    
    <!-- Vista previa -->
    <div v-if="manualReport.weather.images.length" class="mt-3 space-y-4">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
        <div v-for="(img, index) in manualReport.weather.images" :key="index" class="relative group">
          <div class="relative pb-[100%] h-0 overflow-hidden rounded-lg border-2 border-gray-200 dark:border-gray-600">
            <img :src="img.preview" 
                 class="absolute inset-0 w-full h-full object-cover transition-transform duration-300 group-hover:scale-105" />
          </div>
          <button type="button" 
                  @click="removeImage(index)"
                  class="absolute -top-3 -right-3 bg-red-600 hover:bg-red-700 text-white rounded-full w-8 h-8 flex items-center justify-center text-base transition-all duration-200 shadow-lg">
            ✕
          </button>
        </div>
      </div>
    </div>

  </div> 
  

<!-- Coordenadas para reportes manuales -->
              <div class="flex items-center justify-between">
                <button type="button" @click="openManualReportMapModal" class="btn-primary">🗺 Agregar Ubicación</button>
                <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng"
                     class="text-sm text-gray-600 dark:text-gray-400 bg-green-100 dark:bg-green-900 px-3 py-2 rounded-lg">
                  <strong>Coordenadas:</strong> {{ manualReport.coordinates.lat.toFixed(6) }}, {{ manualReport.coordinates.lng.toFixed(6) }}
                </div>
              </div>

              <!-- Fuentes de información -->
              <div class="space-y-2">
                <div class="flex items-center justify-between">
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Fuentes de información</label>
                  <button 
                    type="button" 
                    @click="addSource"
                    class="text-sm text-primary hover:text-primary-dark dark:text-blue-400 dark:hover:text-blue-300 flex items-center"
                  >
                    <span class="mr-1">+</span> Agregar fuente
                  </button>
                </div>
                
                <!-- Lista de fuentes agregadas -->
                <div v-if="sources.length > 0" class="mt-2 space-y-2">
                  <div v-for="(source, index) in sources" :key="index" class="flex items-center">
                    <a 
                      :href="source.url" 
                      target="_blank" 
                      rel="noopener noreferrer"
                      class="text-sm text-blue-600 dark:text-blue-400 hover:underline truncate flex-1"
                    >
                      {{ source.url }}
                    </a>
                    <button 
                      type="button" 
                      @click="removeSource(index)"
                      class="text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 ml-2"
                      title="Eliminar fuente"
                    >
                      ×
                    </button>
                  </div>
                </div>
                
                <!-- Input para agregar nueva fuente -->
                <div v-if="showSourceInput" class="mt-2 flex">
                  <input
                    ref="sourceInput"
                    v-model="newSource"
                    type="url"
                    placeholder="https://ejemplo.com"
                    class="flex-1 px-3 py-2 text-sm border border-gray-300 dark:border-gray-600 rounded-l-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white"
                    @keyup.enter="saveSource"
                  >
                  <button 
                    type="button" 
                    @click="saveSource"
                    class="px-3 py-2 bg-primary text-white rounded-r-lg hover:bg-primary-dark focus:outline-none focus:ring-2 focus:ring-primary focus:ring-opacity-50"
                  >
                    Guardar
                  </button>
                </div>
              </div>

              <div class="flex justify-end space-x-3">
                <button @click="showManualReportForm = false" class="btn-secondary">
                  Cancelar
                </button>
                <button @click="generateManualReport" class="btn-primary">
                  📝 Generar Comunicado
                </button>
              </div>
            </div>
          </div>

          <div v-if="showReport" class="flex justify-end mt-6">
            <button @click="printReport" class="bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-lg font-medium transition-all duration-300 flex items-center space-x-2">
              <span>🖨</span>
              <span>Imprimir PDF</span>
            </button>
          </div>
        </div>

        <!-- Vista Previa del Reporte -->
        <div v-if="showReport" id="report-content" class="bg-white dark:bg-gray-800 rounded-xl shadow-lg print:shadow-none border border-gray-200 dark:border-gray-700">
          
          <!-- Header del Reporte -->
          <div class="p-4 border-b border-gray-200 dark:border-gray-700 print:border-gray-300 print:py-2 print:px-0">
            <div class="flex justify-between items-center">
              <!-- Logo -->
              <div class="mr-4">
                <img src="./Escudo_de_Roatan.png" alt="Escudo de Roatan" class="h-16 w-auto print:h-12">
              </div>
              
              <div class="text-center flex-1">
                <h1 class="text-xl font-bold text-gray-900 dark:text-white print:text-black print:text-lg">Reporte Situacional</h1>
                <h2 class="text-base font-semibold text-gray-700 dark:text-gray-300 print:text-black print:text-sm">
                  {{ getReportTitle() }}
                </h2>
                <p class="text-sm text-gray-600 dark:text-gray-400 print:text-black print:text-xs">{{ getCurrentDateTime() }}</p>
              </div>
              <div v-if="reportCategory === 'manual' && selectedManualReportType === 'weather'" class="flex flex-col items-center justify-center text-center">
                <div class="text-3xl mb-1 print:text-2xl">⚠️</div>
                <h3 class="text-lg font-bold print:text-base print:text-black">{{ getRiskLevelText() }}</h3>
                <p class="text-base print:text-sm print:text-black">{{ getWeatherAlertTitle() }}</p>
              </div>
            </div>
          </div>

          <!-- Contenido del Reporte -->
          <div class="p-4 print:p-2">
            <!-- Reporte Manual -->
            <div v-if="reportCategory === 'manual' && selectedManualReportType">
              <!-- Alerta Meteorológica -->
              <div v-if="selectedManualReportType === 'weather'"> 

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6 print:grid-cols-2 print:gap-2 print:mb-4">
                  <!-- Información del Evento -->
                  <div class="bg-blue-50 dark:bg-blue-900/20 p-4 rounded-lg border border-gray-200 dark:border-gray-600 print:bg-white print:border-gray-300 print:shadow-none">
                    <h4 class="font-bold text-blue-900 dark:text-blue-200 mb-2 text-sm print:text-black print:text-xs print:font-bold print:mb-1">📊 INFORMACIÓN DEL EVENTO</h4>
                    <div class="space-y-1 text-xs text-gray-700 dark:text-gray-300 print:text-black print:text-[10px] print:leading-tight">
                      <p class="print:py-0.5"><strong class="font-semibold print:font-semibold">Tipo:</strong> {{ getWeatherAlertTitle() }}</p>
                      <p class="print:py-0.5"><strong class="font-semibold print:font-semibold">Zona:</strong> {{ getAffectedZoneText() }}</p>
                      <p class="print:py-0.5"><strong class="font-semibold print:font-semibold">Duración:</strong> {{ manualReport.weather.duration || 'Por determinar' }}</p>
                      <p v-if="manualReport.weather.rainfall" class="print:py-0.5"><strong class="font-semibold print:font-semibold">Precipitación:</strong> {{ manualReport.weather.rainfall }} mm</p>
                      <p v-if="manualReport.weather.intensity" class="print:py-0.5"><strong class="font-semibold print:font-semibold">Intensidad:</strong> {{ manualReport.weather.intensity }}</p>
                      <p v-if="manualReport.weather.impactTime" class="print:py-0.5"><strong class="font-semibold print:font-semibold">Impacto:</strong> {{ manualReport.weather.impactTime }}</p>
                      <p v-if="manualReport.weather.probability" class="print:py-0.5">
                        <strong class="font-semibold print:font-semibold">Probabilidad:</strong> 
                        {{ manualReport.weather.probability === 'baja' ? 'Baja (≤30%)' : 
                           manualReport.weather.probability === 'media' ? 'Media (31-70%)' : 
                           'Alta (>70%)' }}
                      </p> 
                      <p v-if="manualReport.weather.notes" class="print:py-0.5"><strong class="font-semibold print:font-semibold">Notas:</strong> {{ manualReport.weather.notes }}</p>
                    </div>
                  </div>

                  <!-- Nivel de Riesgo -->
                  <div class="bg-blue-50 dark:bg-blue-900/20 p-4 rounded-lg border border-gray-200 dark:border-gray-600 print:bg-white print:border-gray-300 print:shadow-none">
                    <h4 class="font-bold text-red-900 dark:text-red-200 mb-2 text-sm print:text-black print:text-xs print:font-bold print:mb-1">🚨 NIVEL DE RIESGO</h4>
                    <div class="text-center">
                      <div class="text-5xl mb-1 print:text-4xl">{{ getRiskIcon() }}</div>
                      <p class="font-bold text-base text-gray-900 dark:text-white print:text-black print:text-sm">{{ getRiskLevelText() }}</p>
                    </div>
                  </div>
                </div>

                <!-- Mapa -->
                <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">
                    🗺️ UBICACIÓN
                  </h3>
                  <div class="h-80 md:h-96">
                    <div id="report-map" style="height: 100%; border-radius: 8px;"></div>
                  </div>
                </div>
                
                <!-- Evidencia fotográfica -->
                <div v-if="manualReport.weather.images && manualReport.weather.images.length > 0" class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">
                    📸 ANEXOS
                  </h3>
                  <div class="h-80 md:h-96 overflow-y-auto">
                    <div class="grid grid-cols-2 gap-2 p-2 bg-gray-100 dark:bg-gray-700 rounded-lg">
                      <div v-for="(img, index) in manualReport.weather.images" :key="index" class="relative group">
                        <img :src="img.preview" class="w-full h-40 object-cover rounded-lg border border-gray-300 dark:border-gray-600" />
                      </div>
                    </div>
                  </div>
                </div>

                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">📝 DESCRIPCIÓN</h3>
                  <div class="bg-gray-50 dark:bg-gray-700 p-6 rounded-lg">
                    <p class="text-gray-700 dark:text-gray-300">{{ manualReport.description }}</p>
                  </div>
                </div>

                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">👥 RECOMENDACIONES PARA LA POBLACIÓN</h3>
                  <div class="bg-blue-50 dark:bg-blue-900/20 p-6 rounded-lg border border-blue-200 dark:border-blue-800">
                    <p class="text-blue-800 dark:text-blue-300">{{ manualReport.recommendations }}</p>
                  </div>
                </div>
              </div>

              <!-- Otros tipos de reportes manuales -->
              <div v-if="selectedManualReportType === 'security'">
                <div class="alert-banner text-center mb-8">
                  <div class="text-4xl mb-2">🚨</div>
                  <h3 class="text-2xl font-bold">AVISO DE SEGURIDAD</h3>
                  <p class="text-lg">{{ getSecurityAlertTitle() }}</p>
                </div>

                <!-- Mapa en el reporte si hay coordenadas -->
                <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">🗺️ UBICACIÓN</h3>
                  <div id="report-map" style="height: 400px; border-radius: 8px;" class="mb-4"></div>
                </div>

                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">📝 DESCRIPCIÓN</h3>
                  <div class="bg-gray-50 dark:bg-gray-700 p-6 rounded-lg">
                    <p class="text-gray-700 dark:text-gray-300">{{ manualReport.description }}</p>
                    <p v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="text-sm text-gray-600 dark:text-gray-400 mt-3">
                      <strong>Coordenadas del evento:</strong> {{ manualReport.coordinates.lat.toFixed(6) }}, {{ manualReport.coordinates.lng.toFixed(6) }}
                    </p>
                  </div>
                </div>

                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">👥 RECOMENDACIONES PARA LA POBLACIÓN</h3>
                  <div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-lg border border-red-200 dark:border-red-800">
                    <p class="text-red-800 dark:text-red-300">{{ manualReport.recommendations }}</p>
                  </div>
                </div>
                
              </div>
            </div>

            <!-- Reportes Automáticos -->
            <div v-if="reportCategory === 'automatic'">
              <!-- Reporte de Cierre de Turno -->
              <div v-if="selectedReportType === 'cierre'">
                <!-- Información del Turno -->
                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">👮 INFORMACIÓN DEL TURNO</h3>
                  <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="bg-blue-50 dark:bg-blue-900/20 p-6 rounded-lg">
                      <h4 class="font-bold text-blue-900 dark:text-blue-200 mb-3">📊 DATOS GENERALES</h4>
                      <div class="space-y-2 text-sm">
                        <p><strong>Supervisor:</strong> {{ activeShift?.supervisor || 'No disponible' }}</p>
                        <p><strong>Inicio:</strong> {{ activeShift?.startTime || 'No disponible' }}</p>
                        <p><strong>Cierre:</strong> {{ getCurrentDateTime() }}</p>
                        <p><strong>Total de Eventos:</strong> {{ events.length }}</p>
                      </div>
                    </div>

                    <div class="bg-green-50 dark:bg-green-900/20 p-6 rounded-lg">
                      <h4 class="font-bold text-green-900 dark:text-green-200 mb-3">📈 ESTADÍSTICAS</h4>
                      <div class="space-y-2 text-sm">
                        <p><strong>Tiempo Resp. Promedio:</strong> {{ getAverageResponseTimeFiltered() }} min</p>
                        <p><strong>Recursos Desplegados:</strong> {{ getTotalResourcesDeployedFiltered() }}</p>
                        <p><strong>Cámaras Activas:</strong> {{ getTotalOnlineCameras() }}/{{ getTotalCameras() }}</p>
                        <p><strong>Eventos Críticos:</strong> {{ getEventsByPriorityFiltered('critica') }}</p>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Resumen por Prioridad -->
                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">🚨 RESUMEN POR PRIORIDAD</h3>
                  <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                    <div class="text-center p-6 bg-gradient-to-br from-red-50 to-red-100 dark:from-red-900 dark:to-red-800 rounded-xl border border-red-200 dark:border-red-700">
                      <div class="text-4xl font-bold text-red-600 dark:text-red-400">{{ getEventsByPriorityFiltered('critica') }}</div>
                      <div class="text-sm text-red-600 dark:text-red-400 mt-2 font-medium">🔴 Críticos</div>
                    </div>
                    <div class="text-center p-6 bg-gradient-to-br from-orange-50 to-orange-100 dark:from-orange-900 dark:to-orange-800 rounded-xl border border-orange-200 dark:border-orange-700">
                      <div class="text-4xl font-bold text-orange-600 dark:text-orange-400">{{ getEventsByPriorityFiltered('alta') }}</div>
                      <div class="text-sm text-orange-600 dark:text-orange-400 mt-2 font-medium">🟠 Altos</div>
                    </div>
                    <div class="text-center p-6 bg-gradient-to-br from-yellow-50 to-yellow-100 dark:from-yellow-900 dark:to-yellow-800 rounded-xl border border-yellow-200 dark:border-yellow-700">
                      <div class="text-4xl font-bold text-yellow-600 dark:text-yellow-400">{{ getEventsByPriorityFiltered('media') }}</div>
                      <div class="text-sm text-yellow-600 dark:text-yellow-400 mt-2 font-medium">🟡 Medios</div>
                    </div>
                    <div class="text-center p-6 bg-gradient-to-br from-green-50 to-green-100 dark:from-green-900 dark:to-green-800 rounded-xl border border-green-200 dark:border-green-700">
                      <div class="text-4xl font-bold text-green-600 dark:text-green-400">{{ getEventsByPriorityFiltered('baja') }}</div>
                      <div class="text-sm text-green-600 dark:text-green-400 mt-2 font-medium">🟢 Bajos</div>
                    </div>
                  </div>
                </div>

                <!-- Detalle de Eventos -->
                <div class="mb-8" v-if="events.length > 0">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">📋 DETALLE DE EVENTOS</h3>
                  <div class="overflow-x-auto">
                    <table class="w-full border-collapse border border-gray-300 dark:border-gray-600">
                      <thead class="bg-gray-100 dark:bg-gray-700">
                        <tr>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">Código</th>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">Hora</th>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">Tipo</th>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">Prioridad</th>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">Ubicación</th>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">T.Resp</th>
                          <th class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-left">Agente</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="event in events" :key="event.code" class="hover:bg-gray-50 dark:hover:bg-gray-600">
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2 font-mono text-sm">{{ event.code }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2">{{ event.time }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2">{{ getEventTypeName(event.type) }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2">
                            <span :class="getPriorityBadgeClass(event.priority)" class="px-2 py-1 text-xs font-medium rounded">
                              {{ event.priority.toUpperCase() }}
                            </span>
                          </td>
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2 text-sm">{{ getShortLocationText(event.location) }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2">{{ event.actualResponseTime || 'N/A' }} min</td>
                          <td class="border border-gray-300 dark:border-gray-600 px-4 py-2">{{ event.registeredBy }}</td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>

                <!-- Distribución por Instituciones -->
                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">🏢 PARTICIPACIÓN INSTITUCIONAL</h3>
                  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <div v-for="institution in institutions" :key="institution.id"
                         class="bg-gray-50 dark:bg-gray-700 p-4 rounded-lg border border-gray-200 dark:border-gray-600">
                      <div class="flex items-center mb-2">
                        <span class="text-2xl mr-2">{{ institution.icon }}</span>
                        <h4 class="font-semibold text-gray-900 dark:text-white">{{ institution.name }}</h4>
                      </div>
                      <p class="text-lg font-bold text-primary">
                        {{ getEventsByInstitution(institution.id) }} eventos
                      </p>
                      <p class="text-sm text-gray-600 dark:text-gray-400">
                        Tiempo resp. prom: {{ getAvgResponseTimeByInstitution(institution.id) }} min
                      </p>
                    </div>
                  </div>
                </div>

                <!-- Observaciones y Recomendaciones -->
                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">📝 OBSERVACIONES Y RECOMENDACIONES</h3>
                  <div class="bg-yellow-50 dark:bg-yellow-900/20 p-6 rounded-lg border border-yellow-200 dark:border-yellow-800">
                    <div class="space-y-3">
                      <p class="text-yellow-800 dark:text-yellow-200">
                        • Se registraron {{ events.length }} eventos durante el turno
                      </p>
                      <p class="text-yellow-800 dark:text-yellow-200">
                        • Tiempo de respuesta promedio de {{ getAverageResponseTimeFiltered() }} minutos
                      </p>
                      <p class="text-yellow-800 dark:text-yellow-200">
                        • {{ getTotalOnlineCameras() }} de {{ getTotalCameras() }} cámaras operativas
                      </p>
                      <p class="text-yellow-800 dark:text-yellow-200" v-if="getEventsByPriorityFiltered('critica') > 0">
                        • Se atendieron {{ getEventsByPriorityFiltered('critica') }} eventos de prioridad crítica
                      </p>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Reporte Distribucional -->
              <div v-if="selectedReportType === 'distribucional'">
                <div class="mb-8">
                  <h3 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white border-b-2 border-primary pb-2">🗺️ DISTRIBUCIÓN TERRITORIAL</h3>
                  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <div v-for="community in communities" :key="community.id"
                         class="bg-white dark:bg-gray-700 p-6 rounded-lg border border-gray-200 dark:border-gray-600 shadow">
                      <h4 class="font-bold text-lg text-gray-900 dark:text-white mb-3 flex items-center">
                        <span class="text-2xl mr-2">🏘️</span>
                        {{ community.name }}
                      </h4>
                      <div class="space-y-2">
                        <p><strong>Población:</strong> {{ community.population.toLocaleString() }} hab.</p>
                        <p><strong>Eventos:</strong> {{ getEventsByCommunity(community.id) }}</p>
                        <p><strong>Cámaras:</strong> {{ community.cameras.online }}/{{ community.cameras.total }}</p>
                        <div class="mt-3">
                          <p class="text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Barrios:</p>
                          <p class="text-sm text-gray-600 dark:text-gray-400">{{ community.neighborhoods.join(', ') }}</p>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Footer del Reporte -->
           <div class="text-center text-sm text-gray-500 dark:text-gray-400 p-6 border-t border-gray-200 dark:border-gray-600 mt-8">
               <div class="max-w-3xl mx-auto">
                   <div v-if="sources.length > 0" class="flex flex-col items-center space-y-2 mb-4">
                       <span class="font-medium text-gray-500 dark:text-gray-400">🌐 Fuentes:</span>
                       <div class="flex flex-wrap justify-center items-center gap-2">
                           <template v-for="(source, index) in sources" :key="source.id">
                               <a :href="source.url" target="_blank" rel="noopener noreferrer" 
                                  class="text-blue-600 dark:text-blue-400 hover:underline whitespace-nowrap">
                                   {{ source.url }}
                               </a>
                               <span v-if="index < sources.length - 1" class="text-gray-400 dark:text-gray-500">|</span>
                           </template>
                       </div>
                   </div>
                   <p>📞 Emergencias: *1101</p>
                   <p class="mb-1"><strong>Centro de Monitoreo Roatán</strong></p> 
                   <p>Documento generado el {{ getCurrentDateTime() }}</p>
               </div>
           </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { createApp } from 'vue'

export default {
  name: 'CentroMonitoreo',
  data() {
    return {
      currentDateTime: '',
      activeTab: 'dashboard',
      sources: [],
      newSource: '',
      showSourceInput: false,
      reportSource: '',
      availableSources: [
        { id: 'social_media', name: 'Redes Sociales' },
        { id: 'citizen_report', name: 'Reporte Ciudadano' },
        { id: 'cctv', name: 'Cámaras de Seguridad' },
        { id: 'institution', name: 'Institución' },
        { id: 'other', name: 'Otra Fuente' }
      ],
      selectedSources: [],
      activeShift: null,
      closedShifts: [],
      showTurnCodeModal: false,
      showEndShiftModal: false,
      showAgentCodeModal: false,
      showEditResponseTimeModal: false,
      showMapModal: false,
      showEventMapModal: false,
      showManualReportMapModal: false,
      selectedEvent: null,
      turnCode: '',
      turnCodeError: '',
      agentCode: '',
      agentCodeError: '',
      editingEvent: null,
      editingEventResponseTime: null,
      tempCoordinates: { lat: null, lng: null },
      tempManualReportCoordinates: { lat: null, lng: null },
      modalMap: null,
      eventMap: null,
      manualReportMap: null,
      reportMap: null,
      
      // Reporte manual
      manualReport: {
        weather: {
          alertType: 'lluvia',
          riskLevel: 'amarillo',
          affectedZones: [],
          duration: '',
          intensity: 'moderada',
          impactTime: '',
          priorityLevel: '',
          probability: '',
          notes: '',
          images: [] // para guardar las fotos
        },
        security: {
          alertType: 'preventivo',
          affectedZone: ''
        },
        description: '',
        recommendations: '',
        coordinates: { lat: null, lng: null }
      },

      // Reports
      reportCategory: 'automatic',
      selectedReportType: 'cierre',
      selectedManualReportType: 'weather',
      showManualReportForm: false,
      showReport: false,
      
      // Charts
      eventsTypeChart: null,
      institutionsChart: null,
      hourlyChart: null,
      communitiesChart: null,
      
      tabs: [
        { id: 'dashboard', name: '📊 Dashboard' },
        { id: 'turno', name: '👮 Turno' },
        { id: 'events', name: '📝 Eventos' },
        { id: 'historial', name: '📚 Historial' },
        { id: 'reportes', name: '📋 Reportes' }
      ],
      
      communities: [
  {
    id: 'coxenhole',
    name: 'Coxen Hole',
    population: 18000,
    neighborhoods: [
      'Centro Histórico',
      'Colonia Divalle',
      'El Swampo',
      'Zona Comercial',
      'Barrio El Manantial',
      'Los Maestros',
      'Coxen Cove',
      'Bella Vista',
      'Franco Flat',
      'Mant Trap',
      'Cañaveral',
      'Zompopal',
      'Spanish Town',
      'Brass Hill',
      'La Punta',
      'Willie Warren',
      'Loma Linda',
      'Spring Garden 1',
      'Spring Garden 2',
      'Coconout Garden',
      'New York',
      'El Tiquet',
      'Nicaragua',
      'Palos Altos',
      'Calle 8/ El Mercado',
      'La Fe'
    ],
    cameras: { total: 12, online: 11 }
  },
  {
    id: 'frenchharbour',
    name: 'French Harbour',
    population: 14000,
    neighborhoods: [
      'Colonia Los Fuertes',
      'Spring Garden',
      'Zona Portuaria',
      'Área Industrial',
      'Barrio Bella Vista',
      'La Punta',
      'La Loma',
      'El Bajo',
      'Los Fuertes',
      'La Rampla',
      'La Bahia',
      'Monte Placentero'
    ],
    cameras: { total: 10, online: 9 }
  },
  {
    id: 'sandybay',
    name: 'Sandy Bay',
    population: 8000,
    neighborhoods: [
      'Área Turística',
      'Zona Costera',
      'Sector Educativo',
      'Barrio Loma Linda',
      'Policarpo Galindo',
      'Balfate',
      'Bellavista',
      'Gibson Bigh',
      'Las Gemelas',
      'Lawson Rock',
      "Antony's Key",
      'White Rock Hills',
      'La Uva'
    ],
    cameras: { total: 6, online: 6 }
  },
  {
    id: 'westend',
    name: 'West End',
    population: 6000,
    neighborhoods: [
      'Zona Hotelera',
      'Área Comercial',
      'Playa Pública',
      'Zona de Restaurantes',
      'El Berinche',
      'West End Village',
      'Half Moon Bay',
      'Mangrove Bight'
    ],
    cameras: { total: 5, online: 5 }
  },
  {
    id: 'westbay',
    name: 'West Bay',
    population: 5000,
    neighborhoods: [
      'Resorts',
      'Zona de Buceo',
      'Playas Privadas',
      'Área Residencial',
      'Colonia Trejo y El Barrial',
      'Villas Mackay',
      'Residencial Merendon Hills',
      'Infinity Bay Spa & Beach Resort',
      'West Bay Village',
      'Turtle Crossing',
      'The Turrets',
      'Lighthouse Estates'
    ],
    cameras: { total: 4, online: 4 }
  },
  {
    id: 'flowersbay',
    name: 'Flowers Bay',
    population: 4000,
    neighborhoods: [
      'Zona Residencial',
      'Área Costera',
      'Barrio Garífuna',
      'Sector Cultural',
      'First Bight'
    ],
    cameras: { total: 3, online: 3 }
  }
],
      
      institutions: [
        {
          id: 'policia',
          name: 'Policía',
          icon: '🚓',
          responseTime: 5,
          resources: [
            { id: 'patrullas', name: 'Patrullas', available: 12 },
            { id: 'motocicletas', name: 'Motocicletas', available: 8 },
            { id: 'agentes', name: 'Agentes', available: 25 }
          ]
        },
        {
          id: 'bomberos',
          name: 'Bomberos',
          icon: '🚒',
          responseTime: 8,
          resources: [
            { id: 'autobombas', name: 'Autobombas', available: 4 },
            { id: 'bomberos', name: 'Bomberos', available: 16 }
          ]
        },
        {
          id: 'copeco',
          name: 'Copeco (Ambulancia)',
          icon: '🚑',
          responseTime: 10,
          resources: [
            { id: 'ambulancias_copeco', name: 'Ambulancias', available: 3 },
            { id: 'paramedicos_copeco', name: 'Paramédicos', available: 12 }
          ]
        },
        {
          id: 'cruz_roja',
          name: 'Cruz Roja (Ambulancia)',
          icon: '🏥',
          responseTime: 12,
          resources: [
            { id: 'ambulancias_cruz_roja', name: 'Ambulancias', available: 2 },
            { id: 'paramedicos_cruz_roja', name: 'Paramédicos', available: 8 }
          ]
        }
      ],
      
      eventTypes: [
        { id: 'seguridad', name: 'Incidente de Seguridad', code: 'SEG' },
        { id: 'transito', name: 'Accidente de Tránsito', code: 'TRA' },
        { id: 'incendio', name: 'Incendio', code: 'INC' },
        { id: 'emergencia_medica', name: 'Emergencia Médica', code: 'MED' },
        { id: 'vandalismo', name: 'Vandalismo', code: 'VAN' },
        { id: 'disturbios', name: 'Disturbios Públicos', code: 'DIS' },
        { id: 'robo', name: 'Robo/Hurto', code: 'ROB' },
        { id: 'violencia', name: 'Violencia Doméstica', code: 'VDO' },
        { id: 'servicio_publico', name: 'Falla Servicio Público', code: 'SER' },
        { id: 'emergencia_ambiental', name: 'Emergencia Ambiental', code: 'AMB' }
      ],
      
      impactTypes: [
        { id: 'personas_afectadas', label: 'Personas Afectadas', placeholder: 'Número de personas' },
        { id: 'vehiculos_involucrados', label: 'Vehículos Involucrados', placeholder: 'Número de vehículos' },
        { id: 'calles_cerradas', label: 'Calles Cerradas', placeholder: 'Nombres de calles' },
        { id: 'servicios_interrumpidos', label: 'Servicios Interrumpidos', placeholder: 'Tipos de servicio' },
        { id: 'perdidas_materiales', label: 'Pérdidas Materiales', placeholder: 'Descripción de daños' },
        { id: 'tiempo_interrupcion', label: 'Tiempo de Interrupción', placeholder: 'Duración estimada' }
      ],
      
      newEvent: {
        type: '',
        code: '',
        priority: 'media',
        description: '',
        location: {
          community: '',
          neighborhood: '',
          reference: ''
        },
        coordinates: { lat: null, lng: null },
        institutionsUsed: {},
        resourcesUsed: {},
        resourceSpecifications: {},
        responseTimeEstimated: {},
        impacts: {},
        impactDetails: {}
      },
      
      pendingEvent: null,
      events: [],
      allEvents: [],
      availableNeighborhoods: [],
      dashboardStartDate: '',
      dashboardEndDate: '',
      
      // Filtros para historial
      historialFilters: {
        startDate: '',
        endDate: '',
        eventType: '',
        priority: '',
        community: '',
        shift: ''
      }
    }
  },
  
  computed: {
    currentShiftEvents() {
      if (!this.activeShift) return []
      return this.events.filter(event => event.shiftSupervisor === this.activeShift.supervisor)
    }
  },
  
  mounted() {
    this.updateDateTime()
    setInterval(this.updateDateTime, 1000)
    this.resetDashboardDates()
    this.updateDashboard()
    this.generateSampleData()
    this.initDarkMode()
    this.loadChartsLibraries()
  },
  
  methods: {
    loadChartsLibraries() {
      if (typeof Chart === 'undefined') {
        const script = document.createElement('script')
        script.src = 'https://cdn.jsdelivr.net/npm/chart.js'
        script.onload = () => {
          const plugin = document.createElement('script')
          plugin.src = 'https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2'
          plugin.onload = () => {
            Chart.register(ChartDataLabels)
            this.updateDashboard()
          }
          document.head.appendChild(plugin)
        }
        document.head.appendChild(script)
      }
    },

    initDarkMode() {
      if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
        document.documentElement.classList.add('dark')
      }
      window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
        if (event.matches) {
          document.documentElement.classList.add('dark')
        } else {
          document.documentElement.classList.remove('dark')
        }
      })
    },
    
    updateDateTime() {
      const now = new Date()
      this.currentDateTime = now.toLocaleString('es-ES', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      })
    },
    
    setActiveTab(tabId) {
      this.activeTab = tabId
    },

    // Validación para iniciar turno
    canInitiateTurn() {
      // Verificar que todas las comunidades tengan datos de cámaras
      const camerasConfigured = this.communities.every(community => 
        community.cameras.total > 0 && 
        community.cameras.online >= 0 && 
        community.cameras.online <= community.cameras.total
      )

      // Verificar que todas las instituciones tengan recursos configurados
      const resourcesConfigured = this.institutions.every(institution =>
        institution.resources.every(resource => resource.available >= 0)
      )

      return camerasConfigured && resourcesConfigured
    },

    validateAndInitiateTurn() {
      if (this.canInitiateTurn()) {
        this.showTurnCodeModal = true
      }
    },
    
    generateSampleData() {
      const sampleEvents = [
        {
          type: 'seguridad',
          code: 'SEG-240120-1430-01',
          priority: 'alta',
          description: 'Robo a transeúnte reportado en zona comercial',
          location: { community: 'coxenhole', neighborhood: 'Centro Histórico', reference: 'Cerca del banco principal' },
          coordinates: { lat: 16.3291, lng: -86.5392 },
          institutionsUsed: { policia: true },
          resourcesUsed: { 'policia_patrullas': true, 'policia_agentes': true },
          resourceSpecifications: { 'policia_patrullas': '2 unidades', 'policia_agentes': '4 efectivos' },
          responseTimeEstimated: { policia: 5 },
          impacts: { personas_afectadas: true },
          impactDetails: { personas_afectadas: '1 persona' },
          time: '14:30:00',
          date: new Date().toISOString().split('T')[0],
          registeredBy: 'AGT001',
          actualResponseTime: 7,
          shiftSupervisor: 'SUP001'
        }
      ]
      
      this.allEvents = [...sampleEvents]
    },
    
    // Shift Management
    startTurn() {
      if (this.turnCode.trim() !== 'SOGP') {
        this.turnCodeError = 'Código incorrecto. Debe ingresar "SOGP"'
        return
      }
      
      this.activeShift = {
        supervisor: this.turnCode.trim(),
        startTime: new Date().toLocaleString('es-ES'),
        events: []
      }
      
      this.showTurnCodeModal = false
      this.turnCode = ''
      this.turnCodeError = ''
      this.activeTab = 'turno'
    },
    
    endShift() {
      if (this.activeShift) {
        const closedShift = {
          ...this.activeShift,
          endTime: new Date().toLocaleString('es-ES'),
          totalEvents: this.events.length,
          events: [...this.events]
        }
        
        this.closedShifts.push(closedShift)
        
        this.events.forEach(event => {
          if (!this.allEvents.find(e => e.code === event.code)) {
            this.allEvents.push(event)
          }
        })
        
        this.activeShift = null
        this.events = []
      }
      this.showEndShiftModal = false
    },
    
    // Source Management
    addSource() {
      this.showSourceInput = true
      this.newSource = ''
      this.$nextTick(() => {
        this.$refs.sourceInput?.focus()
      })
    },
    
    saveSource() {
      if (!this.newSource.trim()) return
      
      // Asegurarse de que la URL tenga el protocolo http/https
      let url = this.newSource.trim()
      if (!url.match(/^https?:\/\//)) {
        url = 'https://' + url
      }
      
      this.sources.push({
        id: Date.now(),
        url: url
      })
      
      this.newSource = ''
      this.showSourceInput = false
    },
    
    removeSource(index) {
      this.sources.splice(index, 1)
    },
    
    // Event Management
    generateEventCode() {
      if (this.newEvent.type) {
        const eventType = this.eventTypes.find(t => t.id === this.newEvent.type)
        const date = new Date()
        const dateStr = date.toISOString().slice(2, 10).replace(/-/g, '')
        const timeStr = date.toTimeString().slice(0, 5).replace(':', '')
        const randomNum = Math.floor(Math.random() * 100).toString().padStart(2, '0')
        this.newEvent.code = `${eventType.code}-${dateStr}-${timeStr}-${randomNum}`
      }
    },
    
    onCommunityChange() {
      const community = this.communities.find(c => c.id === this.newEvent.location.community)
      this.availableNeighborhoods = community ? community.neighborhoods : []
      this.newEvent.location.neighborhood = ''
    },
    
    requestAgentCode() {
      this.pendingEvent = { ...this.newEvent }
      this.showAgentCodeModal = true
      this.agentCode = ''
      this.agentCodeError = ''
    },
    
    submitEventWithAgent() {
      if (this.agentCode.trim() === '') {
        this.agentCodeError = 'Debe ingresar un código de agente válido'
        return
      }
      
      const event = {
        ...this.pendingEvent,
        time: new Date().toLocaleString('es-ES', {
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit'
        }),
        date: new Date().toISOString().split('T')[0],
        registeredBy: this.agentCode.trim(),
        actualResponseTime: null,
        shiftSupervisor: this.activeShift?.supervisor
      }
      
      this.events.push(event)
      this.resetNewEvent()
      this.showAgentCodeModal = false
      this.pendingEvent = null
      
      this.updateDashboard()
    },
    
    cancelEventSubmission() {
      this.showAgentCodeModal = false
      this.pendingEvent = null
      this.agentCode = ''
      this.agentCodeError = ''
    },
    
    editEventResponseTime(event) {
      this.editingEvent = event
      this.editingEventResponseTime = event.actualResponseTime || 0
      this.showEditResponseTimeModal = true
    },
    
    updateEventResponseTime() {
      if (this.editingEvent) {
        this.editingEvent.actualResponseTime = this.editingEventResponseTime
        this.showEditResponseTimeModal = false
        this.editingEvent = null
        this.editingEventResponseTime = null
        this.updateDashboard()
      }
    },
    
    resetNewEvent() {
      this.newEvent = {
        type: '',
        code: '',
        priority: 'media',
        description: '',
        location: {
          community: '',
          neighborhood: '',
          reference: ''
        },
        coordinates: { lat: null, lng: null },
        institutionsUsed: {},
        resourcesUsed: {},
        resourceSpecifications: {},
        responseTimeEstimated: {},
        impacts: {},
        impactDetails: {}
      }
      this.availableNeighborhoods = []
    },
    
    // Map functions
    loadLeaflet() {
      return new Promise((resolve) => {
        if (typeof L !== 'undefined') {
          resolve()
          return
        }

        // Cargar CSS de Leaflet
        const link = document.createElement('link')
        link.rel = 'stylesheet'
        link.href = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css'
        document.head.appendChild(link)

        // Cargar JS de Leaflet
        const script = document.createElement('script')
        script.src = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js'
        script.onload = () => {
          resolve()
        }
        document.head.appendChild(script)
      })
    },

    openMapModal() {
      this.showMapModal = true
      this.tempCoordinates = { lat: null, lng: null }
      this.$nextTick(() => {
        this.initModalMap()
      })
    },
    
    closeMapModal() {
      this.showMapModal = false
      if (this.modalMap) {
        this.modalMap.remove()
        this.modalMap = null
      }
    },
    
    confirmMapSelection() {
      if (this.tempCoordinates.lat && this.tempCoordinates.lng) {
        this.newEvent.coordinates = { ...this.tempCoordinates }
      }
      this.closeMapModal()
    },

    openManualReportMapModal() {
      this.showManualReportMapModal = true
      this.tempManualReportCoordinates = { lat: null, lng: null }
      this.$nextTick(() => {
        this.initManualReportMap()
      })
    },

    closeManualReportMapModal() {
      this.showManualReportMapModal = false
      if (this.manualReportMap) {
        this.manualReportMap.remove()
        this.manualReportMap = null
      }
    },

    confirmManualReportMapSelection() {
      if (this.tempManualReportCoordinates.lat && this.tempManualReportCoordinates.lng) {
        this.manualReport.coordinates = { ...this.tempManualReportCoordinates }
      }
      this.closeManualReportMapModal()
    },

    initModalMap() {
      if (typeof L === 'undefined') {
        this.loadLeaflet().then(() => {
          this.createModalMap()
        })
        return
      }
      this.createModalMap()
    },

    createModalMap() {
      if (this.modalMap) {
        this.modalMap.remove()
      }
      
      this.modalMap = L.map('modal-map').setView([16.3291, -86.5392], 12)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(this.modalMap)
      
      let marker = null
      this.modalMap.on('click', (e) => {
        this.tempCoordinates = { lat: e.latlng.lat, lng: e.latlng.lng }
        if (marker) {
          this.modalMap.removeLayer(marker)
        }
        marker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(this.modalMap)
      })
    },

    initManualReportMap() {
      if (typeof L === 'undefined') {
        this.loadLeaflet().then(() => {
          this.createManualReportMap()
        })
        return
      }
      this.createManualReportMap()
    },

    createManualReportMap() {
      if (this.manualReportMap) {
        this.manualReportMap.remove()
      }
      
      this.manualReportMap = L.map('manual-report-map').setView([16.3291, -86.5392], 12)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(this.manualReportMap)
      
      let marker = null
      this.manualReportMap.on('click', (e) => {
        this.tempManualReportCoordinates = { lat: e.latlng.lat, lng: e.latlng.lng }
        if (marker) {
          this.manualReportMap.removeLayer(marker)
        }
        marker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(this.manualReportMap)
      })
    },
    
    showEventOnMap(event) {
      this.selectedEvent = event
      this.showEventMapModal = true
      this.$nextTick(() => {
        this.initEventMap()
      })
    },

    closeEventMapModal() {
      this.showEventMapModal = false
      if (this.eventMap) {
        this.eventMap.remove()
        this.eventMap = null
      }
    },

    initEventMap() {
      if (typeof L === 'undefined' || !this.selectedEvent) return
      
      if (this.eventMap) {
        this.eventMap.remove()
      }
      
      const lat = this.selectedEvent.coordinates.lat
      const lng = this.selectedEvent.coordinates.lng
      
      this.eventMap = L.map('event-map').setView([lat, lng], 15)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(this.eventMap)
      
      const marker = L.marker([lat, lng]).addTo(this.eventMap)
      marker.bindPopup(`
        <strong>${this.selectedEvent.code}</strong><br>
        ${this.getEventTypeName(this.selectedEvent.type)}<br>
        ${this.getFullLocationText(this.selectedEvent.location)}
      `).openPopup()
    },

    initReportMap() {
      if (typeof L === 'undefined' || !this.manualReport.coordinates.lat) return
      
      if (this.reportMap) {
        this.reportMap.remove()
      }
      
      const lat = this.manualReport.coordinates.lat
      const lng = this.manualReport.coordinates.lng
      
      this.reportMap = L.map('report-map').setView([lat, lng], 13)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(this.reportMap)
      
      const marker = L.marker([lat, lng]).addTo(this.reportMap)
      marker.bindPopup(`
        <strong>Ubicación del Evento</strong><br>
        Coordenadas: ${lat.toFixed(6)}, ${lng.toFixed(6)}
      `).openPopup()
    },
    
    // Utility functions
    getEventTypeName(typeId) {
      const type = this.eventTypes.find(t => t.id === typeId)
      return type ? type.name : 'Tipo desconocido'
    },
    
    getPriorityBadgeClass(priority) {
      const classes = {
        'baja': 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200',
        'media': 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200',
        'alta': 'bg-orange-100 text-orange-800 dark:bg-orange-900 dark:text-orange-200',
        'critica': 'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200'
      }
      return classes[priority] || classes['media']
    },
    
    getFullLocationText(location) {
      const community = this.communities.find(c => c.id === location.community)
      const communityName = community ? community.name : 'Comunidad desconocida'
      const parts = [communityName]
      
      if (location.neighborhood) parts.push(location.neighborhood)
      if (location.reference) parts.push(location.reference)
      
      return parts.join(', ')
    },

    getShortLocationText(location) {
      const community = this.communities.find(c => c.id === location.community)
      const communityName = community ? community.name : 'Desconocida'
      return `${communityName}${location.neighborhood ? ', ' + location.neighborhood : ''}`
    },
    
    hasImpacts(event) {
      return Object.values(event.impacts || {}).some(impact => impact)
    },
    
    getImpactsText(event) {
      const impacts = []
      if (event.impacts && event.impactDetails) {
        for (const [key, value] of Object.entries(event.impacts)) {
          if (value && event.impactDetails[key]) {
            const impactType = this.impactTypes.find(i => i.id === key)
            if (impactType) {
              impacts.push(`${impactType.label}: ${event.impactDetails[key]}`)
            }
          }
        }
      }
      return impacts.join('; ') || 'Sin afectaciones reportadas'
    },
    
    getEventInstitutions(event) {
      const institutions = []
      if (event.institutionsUsed) {
        for (const [institutionId, used] of Object.entries(event.institutionsUsed)) {
          if (used) {
            const institution = this.institutions.find(i => i.id === institutionId)
            if (institution) {
              institutions.push(institution)
            }
          }
        }
      }
      return institutions
    },
    
    // Dashboard functions
    resetDashboardDates() {
      const today = new Date()
      const lastWeek = new Date(today.getTime() - 7 * 24 * 60 * 60 * 1000)
      
      this.dashboardStartDate = lastWeek.toISOString().split('T')[0]
      this.dashboardEndDate = today.toISOString().split('T')[0]
    },
    
    updateDashboard() {
      this.$nextTick(() => {
        this.createCharts()
      })
    },
    
    getFilteredEvents() {
      if (!this.dashboardStartDate || !this.dashboardEndDate) {
        return this.allEvents
      }
      
      return this.allEvents.filter(event => {
        const eventDate = event.date
        return eventDate >= this.dashboardStartDate && eventDate <= this.dashboardEndDate
      })
    },
    
    getEventsByPriorityFiltered(priority) {
      return this.getFilteredEvents().filter(event => event.priority === priority).length
    },
    
    getTotalResourcesDeployedFiltered() {
      return this.getFilteredEvents().reduce((total, event) => {
        return total + Object.values(event.resourcesUsed || {}).filter(used => used).length
      }, 0)
    },
    
    getAverageResponseTimeFiltered() {
      const filteredEvents = this.getFilteredEvents()
      const eventsWithResponseTime = filteredEvents.filter(event => event.actualResponseTime && event.actualResponseTime > 0)
      if (eventsWithResponseTime.length === 0) return 0
      
      const total = eventsWithResponseTime.reduce((sum, event) => sum + event.actualResponseTime, 0)
      return Math.round(total / eventsWithResponseTime.length)
    },
    
    // Camera functions
    getTotalCameras() {
      return this.communities.reduce((total, community) => total + community.cameras.total, 0)
    },
    
    getTotalOnlineCameras() {
      return this.communities.reduce((total, community) => total + community.cameras.online, 0)
    },
    
    // Historial functions
    clearHistorialFilters() {
      this.historialFilters = {
        startDate: '',
        endDate: '',
        eventType: '',
        priority: '',
        community: '',
        shift: ''
      }
    },
    
    getFilteredHistorialEvents() {
      let filteredEvents = [...this.allEvents]
      
      if (this.activeShift) {
        filteredEvents = [...filteredEvents, ...this.events]
      }
      
      if (this.historialFilters.startDate) {
        filteredEvents = filteredEvents.filter(event => event.date >= this.historialFilters.startDate)
      }
      
      if (this.historialFilters.endDate) {
        filteredEvents = filteredEvents.filter(event => event.date <= this.historialFilters.endDate)
      }
      
      if (this.historialFilters.eventType) {
        filteredEvents = filteredEvents.filter(event => event.type === this.historialFilters.eventType)
      }
      
      if (this.historialFilters.priority) {
        filteredEvents = filteredEvents.filter(event => event.priority === this.historialFilters.priority)
      }
      
      if (this.historialFilters.community) {
        filteredEvents = filteredEvents.filter(event => event.location.community === this.historialFilters.community)
      }
      
      if (this.historialFilters.shift) {
        if (this.historialFilters.shift === 'current' && this.activeShift) {
          filteredEvents = filteredEvents.filter(event => event.shiftSupervisor === this.activeShift.supervisor)
        } else if (this.historialFilters.shift.startsWith('shift_')) {
          const shiftIndex = parseInt(this.historialFilters.shift.replace('shift_', ''))
          const shift = this.closedShifts[shiftIndex]
          if (shift) {
            filteredEvents = filteredEvents.filter(event => event.shiftSupervisor === shift.supervisor)
          }
        }
      }
      
      return filteredEvents.sort((a, b) => {
        const dateA = new Date(`${a.date} ${a.time}`)
        const dateB = new Date(`${b.date} ${b.time}`)
        return dateB - dateA
      })
    },
    
    getFilteredEventsByPriority(priority) {
      return this.getFilteredHistorialEvents().filter(event => event.priority === priority).length
    },

    // Report functions
    getCurrentDateTime() {
      return new Date().toLocaleString('es-ES', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
    },

    getReportTitle() {
      if (this.reportCategory === 'automatic') {
        return this.selectedReportType === 'cierre' ? 'REPORTE DE CIERRE DE TURNO' : 'REPORTE DISTRIBUCIONAL TERRITORIAL'
      } else {
        return this.getManualReportTitle()
      }
    },

    getManualReportTitle() {
      const titles = {
        weather: 'ALERTA METEOROLÓGICA',
        security: 'AVISO DE SEGURIDAD',
        traffic: 'AVISO DE TRÁNSITO',
        public_service: 'INTERRUPCIÓN DE SERVICIO PÚBLICO'
      }
      return titles[this.selectedManualReportType] || 'COMUNICADO OFICIAL'
    },

    generateAutomaticReport() {
      this.showReport = true
      this.$nextTick(() => {
        // Scroll to report
        document.getElementById('report-content').scrollIntoView({ behavior: 'smooth' })
      })
    },

    generateManualReport() {
      this.showReport = true
      this.showManualReportForm = false
      this.$nextTick(() => {
        // Inicializar mapa en el reporte si hay coordenadas
        if (this.manualReport.coordinates.lat && this.manualReport.coordinates.lng) {
          setTimeout(() => {
            this.initReportMap()
          }, 100)
        }
        // Scroll to report
        document.getElementById('report-content').scrollIntoView({ behavior: 'smooth' })
      })
    },

    getWeatherAlertTitle() {
      const titles = {
        lluvia: 'Alerta por Lluvia',
        tormenta: 'Alerta por Tormenta Eléctrica',
        viento: 'Alerta por Vientos Fuertes',
        granizo: 'Alerta por Granizo',
        calor: 'Alerta por Altas Temperaturas'
      }
      return titles[this.manualReport.weather.alertType] || 'Alerta Meteorológica'
    },

    getSecurityAlertTitle() {
      const titles = {
        preventivo: 'Aviso Preventivo',
        evacuacion: 'Orden de Evacuación',
        restriccion: 'Restricción de Acceso'
      }
      return titles[this.manualReport.security.alertType] || 'Aviso de Seguridad'
    },

    getRiskLevelText() {
      const levels = {
        blanco: 'SIN RIESGO',
        verde: 'RIESGO BAJO',
        amarillo: 'PRECAUCIÓN', 
        rojo: 'ALTO RIESGO'
      }
      return levels[this.manualReport.weather.riskLevel] || 'PRECAUCIÓN'
    },
handleImageUpload(event) {
  const files = event.target.files;
  if (!files.length) return;

  for (let i = 0; i < files.length; i++) {
    const file = files[i];
    if (file.type.startsWith('image/')) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.manualReport.weather.images.push({
          file: file,
          preview: e.target.result
        });
      };
      reader.readAsDataURL(file);
    }
  }
  event.target.value = ''; // Reset the input to allow selecting the same file again
},
    getRiskIcon() {
      const icons = {
        blanco: '⚪',
        verde: '🟢',
        amarillo: '🟡', 
        rojo: '🔴'
      }
      return icons[this.manualReport.weather.riskLevel] || '🟡'
    },

    getAffectedZoneText() {
      if (this.manualReport.weather.affectedZones.includes('municipio')) {
        return 'Todo el municipio'
      }
      
      const zones = this.manualReport.weather.affectedZones
        .filter(zone => zone !== 'municipio')
        .map(zoneId => {
          const community = this.communities.find(c => c.id === zoneId)
          return community ? community.name : zoneId
        })
      
      return zones.length > 0 ? zones.join(', ') : 'Zonas específicas'
    },

    getEventsByInstitution(institutionId) {
      return this.events.filter(event => event.institutionsUsed && event.institutionsUsed[institutionId]).length
    },

    getAvgResponseTimeByInstitution(institutionId) {
      const instEvents = this.events.filter(event => 
        event.institutionsUsed && 
        event.institutionsUsed[institutionId] && 
        event.actualResponseTime
      )
      
      if (instEvents.length === 0) return 'N/A'
      
      const total = instEvents.reduce((sum, event) => sum + event.actualResponseTime, 0)
      return Math.round(total / instEvents.length)
    },

    getEventsByCommunity(communityId) {
      return this.events.filter(event => event.location.community === communityId).length
    },

    printReport() {
      window.print()
    },
    
    // Chart functions
    createCharts() {
      if (typeof Chart === 'undefined') return
      
      this.createEventsTypeChart()
      this.createInstitutionsChart()
      this.createHourlyChart()
      this.createCommunitiesChart()
    },
    
    createEventsTypeChart() {
      const ctx = document.getElementById('eventsTypeChart')
      if (!ctx) return
      
      if (this.eventsTypeChart) {
        this.eventsTypeChart.destroy()
      }

      const events = this.getFilteredEvents()
      const typeData = this.eventTypes.map(type => ({
        label: type.name,
        count: events.filter(e => e.type === type.id).length
      })).filter(item => item.count > 0)

      this.eventsTypeChart = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels: typeData.map(item => item.label),
          datasets: [{
            data: typeData.map(item => item.count),
            backgroundColor: [
              '#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0',
              '#9966FF', '#FF9F40', '#FF6384', '#C9CBCF',
              '#4BC0C0', '#FF6384'
            ]
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            datalabels: {
              color: '#fff',
              font: {
                weight: 'bold'
              },
              formatter: (value, ctx) => {
                return value > 0 ? value : ''
              }
            },
            legend: {
              position: 'bottom'
            }
          }
        }
      })
    },

    createInstitutionsChart() {
      const ctx = document.getElementById('institutionsChart')
      if (!ctx) return
      
      if (this.institutionsChart) {
        this.institutionsChart.destroy()
      }

      const events = this.getFilteredEvents()
      const instData = this.institutions.map(inst => ({
        label: inst.name,
        count: events.filter(e => e.institutionsUsed && e.institutionsUsed[inst.id]).length
      })).filter(item => item.count > 0)

      this.institutionsChart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: instData.map(item => item.label),
          datasets: [{
            label: 'Eventos Atendidos',
            data: instData.map(item => item.count),
            backgroundColor: '#5D5CDE',
            borderColor: '#4F46E5',
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            datalabels: {
              anchor: 'end',
              align: 'top',
              color: '#374151',
              font: {
                weight: 'bold'
              }
            }
          },
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                stepSize: 1
              }
            }
          }
        }
      })
    },

    createHourlyChart() {
      const ctx = document.getElementById('hourlyChart')
      if (!ctx) return
      
      if (this.hourlyChart) {
        this.hourlyChart.destroy()
      }

      const events = this.getFilteredEvents()
      const hourlyData = Array(24).fill(0)
      
      events.forEach(event => {
        if (event.time) {
          const hour = parseInt(event.time.split(':')[0])
          hourlyData[hour]++
        }
      })

      this.hourlyChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: Array.from({length: 24}, (_, i) => `${i.toString().padStart(2, '0')}:00`),
          datasets: [{
            label: 'Eventos por Hora',
            data: hourlyData,
            borderColor: '#10B981',
            backgroundColor: 'rgba(16, 185, 129, 0.1)',
            tension: 0.4,
            fill: true
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            datalabels: {
              display: false
            }
          },
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                stepSize: 1
              }
            }
          }
        }
      })
    },

    createCommunitiesChart() {
      const ctx = document.getElementById('communitiesChart')
      if (!ctx) return
      
      if (this.communitiesChart) {
        this.communitiesChart.destroy()
      }

      const events = this.getFilteredEvents()
      const commData = this.communities.map(comm => ({
        label: comm.name,
        count: events.filter(e => e.location.community === comm.id).length
      })).filter(item => item.count > 0)

      this.communitiesChart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: commData.map(item => item.label),
          datasets: [{
            data: commData.map(item => item.count),
            backgroundColor: [
              '#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0',
              '#9966FF', '#FF9F40'
            ]
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            datalabels: {
              color: '#fff',
              font: {
                weight: 'bold'
              },
              formatter: (value, ctx) => {
                return value > 0 ? value : ''
              }
            },
            legend: {
              position: 'bottom'
            }
          }
        }
      })
    },
    
    printDashboard() {
      window.print()
    }
  }
}
</script>

<style>
@media print {
  @page { 
    size: auto;
    margin: 0mm 0mm 0mm 0mm;
  }
  
  body { 
    -webkit-print-color-adjust: exact; 
    print-color-adjust: exact;
    margin: 0;
    padding: 10mm;
  }
  
  .no-print, .no-print * {
    display: none !important;
  }
  
  /* Ocultar título y fecha en la impresión */
  body::before {
    display: none !important;
  }
}
</style>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');

* { 
  font-family: 'Inter', sans-serif; 
}

@media print {
  .no-print { 
    display: none !important; 
  }
  .print-page { 
    page-break-after: always; 
  }
  body { 
    font-size: 12px; 
  }
}

.camera-status {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 4px;
}

.camera-online { 
  background-color: #10b981; 
}

.camera-offline { 
  background-color: #ef4444; 
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 8px;
  max-width: 90%;
  max-height: 90%;
  overflow: auto;
}

.dark .modal-content {
  background: #1f2937;
}

.chart-container {
  position: relative;
  height: 300px;
  width: 100%;
}

.form-section {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border: 2px solid #e2e8f0;
  border-radius: 1rem;
  padding: 1.5rem;
  margin: 1rem 0;
  transition: all 0.3s ease;
}

.dark .form-section {
  background: linear-gradient(135deg, #374151 0%, #1f2937 100%);
  border-color: #4b5563;
}

.form-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}

.input-group {
  background: white;
  border-radius: 0.75rem;
  padding: 1rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  margin: 0.5rem 0;
}

.dark .input-group {
  background: #374151;
}

.btn-primary {
  background: linear-gradient(135deg, #5D5CDE 0%, #4F46E5 100%);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 0.75rem;
  font-weight: 600;
  transition: all 0.3s ease;
  border: none;
  cursor: pointer;
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 10px 25px rgba(93, 92, 222, 0.3);
}

.btn-secondary {
  background: linear-gradient(135deg, #64748b 0%, #475569 100%);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 0.75rem;
  font-weight: 600;
  transition: all 0.3s ease;
  border: none;
  cursor: pointer;
}

.btn-secondary:hover {
  transform: translateY(-1px);
  box-shadow: 0 10px 25px rgba(100, 116, 139, 0.3);
}

.report-type-card {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 1rem;
  padding: 1.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.report-type-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}

.report-type-card.active {
  border-color: #5D5CDE;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
}

.dark .report-type-card {
  background: #374151;
  border-color: #4b5563;
}

.dark .report-type-card.active {
  border-color: #5D5CDE;
  background: linear-gradient(135deg, #374151 0%, #1f2937 100%);
}

.alert-banner {
  background: linear-gradient(135deg, #fef3c7 0%, #fed7aa 100%);
  border: 2px solid #f59e0b;
  border-radius: 1rem;
  padding: 2rem;
  margin: 1rem 0;
}

.dark .alert-banner {
  background: linear-gradient(135deg, #451a03 0%, #7c2d12 100%);
  border-color: #ea580c;
}
</style>