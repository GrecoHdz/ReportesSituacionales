<template>
  <div id="app" class="bg-gray-50 dark:bg-gray-900 min-h-screen">
    
    <!-- Header del Dashboard para Impresión (solo para dashboard) -->
    <div v-if="activeTab === 'dashboard'" class="dashboard-print-header print:block hidden" style="display: none;">
      <div class="flex items-center justify-center text-center relative print:items-start">
        <!-- Logo -->
        <div class="absolute left-0 flex items-center justify-center print:relative print:left-auto print:w-auto print:flex-shrink-0">
          <img src="./Escudo_de_Roatan.png" alt="Escudo de Roatan" class="h-10 w-auto sm:h-12 md:h-14 print:h-10 print:w-10">
        </div>

        <!-- Título del Reporte (centrado) -->
        <div class="flex-1 print:flex-grow print:text-center print:mx-0">
          <h1 class="text-sm sm:text-base md:text-xl font-bold text-gray-900 dark:text-white print:text-black print:text-lg">
            Centro de Monitoreo Avanzado
          </h1>
          <h2 class="text-[10px] sm:text-xs md:text-lg font-semibold text-gray-700 dark:text-gray-300 print:text-black print:text-sm">
            Dashboard de Estadísticas
          </h2>
          <p class="text-xs text-gray-500 dark:text-gray-400 print:text-black print:text-sm">
            {{ getCurrentDateTime() }}
          </p>
        </div>
      </div>
    </div>

    <!-- Header de la App (mejorado para móviles) -->
    <header class="bg-white dark:bg-gray-800 shadow-lg border-b border-gray-200 dark:border-gray-700 no-print">
      <div class="max-w-7xl mx-auto px-3 sm:px-6 lg:px-8">
        <div class="flex flex-col sm:flex-row justify-between items-center h-auto sm:h-16 py-3 sm:py-0">
          <div class="flex items-center space-x-2 sm:space-x-3 mb-3 sm:mb-0">
            <div class="w-8 h-8 sm:w-12 sm:h-12 rounded-lg sm:rounded-xl flex items-center justify-center shadow-lg">
              <img src="./Escudo_de_Roatan.png" 
                     alt="Escudo de Roatan" 
                     class="h-10 w-auto sm:h-12 md:h-14 print:h-10 print:w-10">
            </div>
            <div class="text-center sm:text-left">
              <h1 class="text-base sm:text-xl font-bold text-gray-900 dark:text-white">Centro de Monitoreo Avanzado</h1>
              <p class="text-xs sm:text-sm text-gray-500 dark:text-gray-400">Sistema Integrado de Reportes Situacionales</p>
            </div>
          </div>

          <div class="hidden sm:flex flex-col sm:flex-row items-center space-y-2 sm:space-y-0 sm:space-x-4">
            <div class="text-center sm:text-right">
              <p class="text-xs sm:text-sm font-medium text-gray-900 dark:text-white">{{ currentDateTime }}</p>
              <p v-if="activeShift" class="text-xs text-green-600 dark:text-green-400 flex items-center justify-center sm:justify-end">
                <span class="w-2 h-2 bg-green-500 rounded-full mr-1 animate-pulse"></span>
                Turno Activo - {{ activeShift.supervisor }}
              </p>
              <p v-else class="text-xs text-red-600 dark:text-red-400 items-center justify-center sm:justify-end hidden sm:flex">
                <span class="w-2 h-2 bg-red-500 rounded-full mr-1"></span>
                Sin turno activo
              </p>
            </div>
          </div>
          <!-- Versión móvil que solo muestra cuando hay turno activo -->
          <div v-if="activeShift" class="sm:hidden flex flex-col items-center space-y-2">
            <div class="text-center">
              <p class="text-xs font-medium text-gray-900 dark:text-white">{{ currentDateTime }}</p>
              <p class="text-xs text-green-600 dark:text-green-400 flex items-center justify-center">
                <span class="w-2 h-2 bg-green-500 rounded-full mr-1 animate-pulse"></span>
                Turno Activo - {{ activeShift.supervisor }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </header>

    <!-- Modales -->
    <!-- Modal de Código de Supervisor para Turno -->
    <div v-if="showTurnCodeModal" class="modal">
      <div class="modal-content p-4 sm:p-6">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Código del Supervisor de Operaciones</h2>
        <form @submit.prevent="startTurn">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Ingrese el código para iniciar turno
            </label>
            <input v-model="turnCode" type="text" required
                   placeholder="Código del supervisor"
                   class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
            <p v-if="turnCodeError" class="text-red-500 text-sm mt-1">{{ turnCodeError }}</p>
          </div>
          <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
            <button type="button" @click="showTurnCodeModal = false" class="btn-secondary w-full sm:w-auto">
              Cancelar
            </button>
            <button type="submit" class="btn-primary w-full sm:w-auto">
              Iniciar Turno
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Cerrar Turno -->
    <div v-if="showEndShiftModal" class="modal">
      <div class="modal-content p-4 sm:p-6">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Cerrar Turno</h2>
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
        <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
          <button @click="showEndShiftModal = false" class="btn-secondary w-full sm:w-auto">
            Cancelar
          </button>
          <button @click="endShift" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg w-full sm:w-auto">
            Cerrar Turno
          </button>
        </div>
      </div>
    </div>

    <!-- Modal de Código de Agente -->
    <div v-if="showAgentCodeModal" class="modal">
      <div class="modal-content p-4 sm:p-6">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Identificación del Agente</h2>
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
          <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
            <button type="button" @click="cancelEventSubmission" class="btn-secondary w-full sm:w-auto">
              Cancelar
            </button>
            <button type="submit" class="btn-primary w-full sm:w-auto">
              Registrar
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Estado del Evento -->
    <div v-if="showEventStatusModal" class="modal">
      <div class="modal-content p-4 sm:p-6">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Actualizar Estado del Evento</h2>
        <form @submit.prevent="updateEventStatus">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Evento: {{ editingEvent?.code }}
            </label>
            <p class="text-sm text-gray-600 dark:text-gray-400 mb-3">{{ editingEvent?.description }}</p>
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Estado
            </label>
            <select v-model="editingEventStatus" required
                    class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
              <option value="en_progreso">🟡 En Progreso</option>
              <option value="resuelto">🟢 Resuelto</option>
              <option value="cancelado">🔴 Cancelado</option>
            </select>
          </div>
          <div v-if="editingEventStatus === 'resuelto'" class="mb-4">
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
              Tiempo de Respuesta (minutos)
            </label>
            <input v-model.number="editingEventResponseTime" type="number" min="0" required
                   placeholder="Tiempo en minutos"
                   class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-md focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
          </div>
          <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
            <button type="button" @click="showEventStatusModal = false" class="btn-secondary w-full sm:w-auto">
              Cancelar
            </button>
            <button type="submit" class="btn-primary w-full sm:w-auto">
              Actualizar
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Editar Evento -->
    <div v-if="showEditEventModal" class="modal">
      <div class="modal-content p-4 sm:p-6 max-w-4xl">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Editar Evento: {{ editingEvent?.code }}</h2>
        <form @submit.prevent="updateEvent" class="space-y-6">
          <!-- Información Básica -->
          <div class="input-group">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">ℹ️ Información Básica</h3>
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Evento</label>
                <select v-model="editingEventData.type" required
                        class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                  <option value="">Seleccionar tipo</option>
                  <option v-for="type in eventTypes" :key="type.id" :value="type.id">{{ type.name }}</option>
                </select>
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Código</label>
                <input v-model="editingEventData.code" type="text" readonly
                       class="w-full px-3 py-2 text-base bg-gray-100 dark:bg-gray-600 border border-gray-300 dark:border-gray-600 rounded-lg dark:text-white">
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Prioridad</label>
                <select v-model="editingEventData.priority"
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
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 mb-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Comunidad</label>
                <select v-model="editingEventData.location.community" @change="onEditCommunityChange" 
                        class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                  <option value="">Seleccionar comunidad</option>
                  <option v-for="community in communities" :key="community.id" :value="community.id">{{ community.name }}</option>
                </select>
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Barrio/Colonia</label>
                <select v-model="editingEventData.location.neighborhood" :disabled="!editingEventData.location.community"
                        class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white disabled:opacity-50">
                  <option value="">Seleccionar barrio</option>
                  <option v-for="neighborhood in editAvailableNeighborhoods" :key="neighborhood" :value="neighborhood">{{ neighborhood }}</option>
                </select>
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Referencia Cercana</label>
                <input v-model="editingEventData.location.reference" type="text"
                       placeholder="Ej: Cerca del parque central"
                       class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
              </div>
            </div>
          </div>

          <!-- Afectaciones -->
          <div class="input-group">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">📊 Afectaciones</h3>
            <div class="space-y-4">
              <div v-for="category in impactCategories" :key="category.id" 
                   class="bg-gray-50 dark:bg-gray-600 border-2 border-gray-200 dark:border-gray-600 rounded-lg overflow-hidden"
                   :class="category.bgClass">
                
                <!-- Category Header -->
                <div class="px-4 py-3 bg-white dark:bg-gray-600">
                  <div class="flex items-center justify-between">
                    <div class="flex items-center gap-3">
                      <input type="checkbox"
                             :id="'edit_category_' + category.id"
                             @change="toggleEditCategoryImpacts(category)"
                             :checked="isEditCategorySelected(category)"
                             class="w-5 h-5 rounded border-gray-300 text-primary focus:ring-primary">
                      <label :for="'edit_category_' + category.id" class="flex items-center gap-2 cursor-pointer">
                        <span class="text-xl">{{ category.icon }}</span>
                        <span class="font-semibold text-gray-900 dark:text-white">
                          {{ category.label }}
                        </span>
                      </label>
                      <span v-if="getEditCategoryActiveCount(category) > 0" 
                            class="px-2 py-1 text-xs font-bold rounded-full bg-green-500 text-white">
                        {{ getEditCategoryActiveCount(category) }}
                      </span>
                    </div>
                    <button @click="toggleEditCategory(category.id)"
                            type="button"
                            class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300">
                      <svg class="w-5 h-5 transition-transform" 
                           :class="{ 'rotate-180': editExpandedCategories[category.id] }"
                           fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
                      </svg>
                    </button>
                  </div>
                </div>

                <!-- Category Content -->
                <div v-show="editExpandedCategories[category.id]" 
                     class="px-4 py-4 bg-white dark:bg-gray-600 border-t border-gray-200 dark:border-gray-700">
                  <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
                    <div v-for="impact in category.impacts" :key="impact.id" 
                         class="bg-gray-50 dark:bg-gray-700 p-3 rounded-lg">
                      <div class="flex items-center mb-2">
                        <input type="checkbox"
                               v-model="editingEventData.impacts[impact.id]"
                               :id="'edit_impact_' + impact.id"
                               :disabled="!editExpandedCategories[category.id]"
                               class="mr-3 w-4 h-4 rounded border-gray-300 text-primary focus:ring-primary disabled:opacity-50">
                        <label :for="'edit_impact_' + impact.id" 
                               class="text-sm font-medium text-gray-700 dark:text-gray-300 disabled:opacity-50">
                          {{ impact.label }}
                        </label>
                      </div>
                      <input v-if="editingEventData.impacts[impact.id]"
                             v-model="editingEventData.impactDetails[impact.id]"
                             type="text"
                             :placeholder="impact.placeholder"
                             class="w-full px-3 py-2 text-sm border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Instituciones y Recursos -->
          <div class="input-group">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">🚓 Instituciones y Recursos</h3>
            <div class="space-y-4 print:space-y-2">
              <div v-for="institution in institutions" :key="institution.id"
                   class="bg-gray-50 dark:bg-gray-600 rounded-lg p-4">
                <div class="flex flex-col lg:flex-row items-start lg:items-center justify-between mb-3">
                  <label class="text-base sm:text-lg font-medium text-gray-700 dark:text-gray-300 flex items-center cursor-pointer mb-2 lg:mb-0">
                    <input type="checkbox" v-model="editingEventData.institutionsUsed[institution.id]"
                           class="mr-3 w-4 h-4 sm:w-5 sm:h-5 rounded border-gray-300 text-primary focus:ring-primary">
                    <span class="text-xl sm:text-2xl mr-3">{{ institution.icon }}</span>
                    {{ institution.name }}
                  </label>
                  <div v-if="editingEventData.institutionsUsed[institution.id]" class="text-sm text-gray-600 dark:text-gray-400">
                    <label class="block text-xs font-medium mb-1">T. Resp. Estimado (min)</label>
                    <input v-model.number="editingEventData.responseTimeEstimated[institution.id]" type="number" min="0"
                           :placeholder="institution.responseTime"
                           class="w-16 sm:w-20 px-2 py-1 text-base border border-gray-300 dark:border-gray-600 rounded focus:ring-1 focus:ring-primary dark:bg-gray-700 dark:text-white">
                  </div>
                </div>
                <div v-if="editingEventData.institutionsUsed[institution.id]" class="ml-0 lg:ml-8">
                  <label class="block text-sm font-medium text-gray-600 dark:text-gray-400 mb-3">Recursos Utilizados:</label>
                  <div class="space-y-2">
                    <div v-for="resource in institution.resources" :key="resource.id" class="flex flex-col lg:flex-row items-start lg:items-center space-y-2 lg:space-y-0 lg:space-x-3">
                      <div class="flex items-center">
                        <input type="checkbox"
                               v-model="editingEventData.resourcesUsed[institution.id + '_' + resource.id]"
                               :id="'edit_res_' + institution.id + '_' + resource.id"
                               class="w-4 h-4 rounded border-gray-300 text-primary focus:ring-primary">
                        <label :for="'edit_res_' + institution.id + '_' + resource.id" class="text-sm flex-1 font-medium ml-2">
                          {{ resource.name }}
                        </label>
                      </div>
                      <input v-if="editingEventData.resourcesUsed[institution.id + '_' + resource.id]"
                             v-model.number="editingEventData.resourceSpecifications[institution.id + '_' + resource.id]"
                             type="number"
                             min="0"
                             placeholder="Cantidad"
                             class="w-16 sm:w-20 px-2 py-1 text-base border border-gray-300 dark:border-gray-600 rounded focus:ring-1 focus:ring-primary dark:bg-gray-700 dark:text-white">
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
            <button type="button" @click="showEditEventModal = false" class="btn-secondary w-full sm:w-auto">
              Cancelar
            </button>
            <button type="submit" class="btn-primary w-full sm:w-auto">
              Guardar Cambios
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de Confirmar Eliminación -->
    <div v-if="showDeleteEventModal" class="modal">
      <div class="modal-content p-4 sm:p-6">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Confirmar Eliminación</h2>
        <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">
          ¿Está seguro que desea eliminar el evento <strong>{{ eventToDelete?.code }}</strong>?
        </p>
        <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">
          Esta acción no se puede deshacer.
        </p>
        <div class="bg-yellow-50 dark:bg-yellow-900/20 border border-yellow-200 dark:border-yellow-800 rounded-lg p-3 mb-4">
          <div class="flex items-start">
            <span class="text-yellow-500 mr-2">⚠️</span>
            <div>
              <p class="text-sm text-yellow-800 dark:text-yellow-200 font-medium">Información del evento:</p>
              <p class="text-sm text-yellow-700 dark:text-yellow-300">{{ getEventTypeName(eventToDelete?.type) }}</p>
              <p class="text-sm text-yellow-700 dark:text-yellow-300">Prioridad: {{ eventToDelete?.priority?.toUpperCase() }}</p>
              <p class="text-sm text-yellow-700 dark:text-yellow-300">Registrado por: {{ eventToDelete?.registeredBy }}</p>
            </div>
          </div>
        </div>
        <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
          <button @click="showDeleteEventModal = false" class="btn-secondary w-full sm:w-auto">
            Cancelar
          </button>
          <button @click="deleteEvent" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg w-full sm:w-auto">
            Eliminar Evento
          </button>
        </div>
      </div>
    </div>

    <!-- Modal de Mapa -->
    <div v-if="showMapModal" class="modal">
      <div class="modal-content modal-map-content p-4 sm:p-6 w-full max-w-3xl">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Seleccionar Ubicación en el Mapa</h2>
        <div id="modal-map" class="h-64 sm:h-96 lg:h-[500px] border-2 border-gray-200 dark:border-gray-600 rounded-lg mb-4"></div>
        <div v-if="tempCoordinates.lat && tempCoordinates.lng" class="mb-4 text-sm text-gray-600 dark:text-gray-400">
          <strong>Coordenadas seleccionadas:</strong> {{ tempCoordinates.lat.toFixed(6) }}, {{ tempCoordinates.lng.toFixed(6) }}
        </div>
        <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
          <button @click="closeMapModal" class="btn-secondary w-full sm:w-auto">
            Cancelar
          </button>
          <button @click="confirmMapSelection" class="btn-primary w-full sm:w-auto">
            Confirmar
          </button>
        </div>
      </div>
    </div>

    <!-- Modal de Mapa para Reportes Manuales -->
    <div v-if="showManualReportMapModal" class="modal">
      <div class="modal-content modal-map-content p-4 sm:p-6 w-full max-w-3xl">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Seleccionar Ubicación para el Reporte</h2>
        <div id="manual-report-map" class="h-64 sm:h-96 lg:h-[500px] border-2 border-gray-200 dark:border-gray-600 rounded-lg mb-4"></div>
        <div v-if="tempManualReportCoordinates.lat && tempManualReportCoordinates.lng" class="mb-4 text-sm text-gray-600 dark:text-gray-400">
          <strong>Coordenadas seleccionadas:</strong> {{ tempManualReportCoordinates.lat.toFixed(6) }}, {{ tempManualReportCoordinates.lng.toFixed(6) }}
        </div>
        <div class="flex flex-col sm:flex-row justify-end space-y-2 sm:space-y-0 sm:space-x-3">
          <button @click="closeManualReportMapModal" class="btn-secondary w-full sm:w-auto">
            Cancelar
          </button>
          <button @click="confirmManualReportMapSelection" class="btn-primary w-full sm:w-auto">
            Confirmar
          </button>
        </div>
      </div>
    </div>

    <!-- Modal para Ver Evento en Mapa -->
    <div v-if="showEventMapModal" class="modal">
      <div class="modal-content modal-map-content p-4 sm:p-6">
        <h2 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-4">Ubicación del Evento: {{ selectedEvent?.code }}</h2>
        <div id="event-map" class="h-64 sm:h-96 lg:h-[500px] border-2 border-gray-200 dark:border-gray-600 rounded-lg mb-4"></div>
        <div v-if="selectedEvent" class="mb-4 text-sm text-gray-600 dark:text-gray-400">
          <strong>Ubicación:</strong> {{ getFullLocationText(selectedEvent.location) }}<br>
          <strong>Coordenadas:</strong> {{ selectedEvent.coordinates.lat?.toFixed(6) }}, {{ selectedEvent.coordinates.lng?.toFixed(6) }}
        </div>
        <div class="flex justify-end">
          <button @click="closeEventMapModal" class="btn-primary w-full sm:w-auto">
            Cerrar
          </button>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 sm:py-6">
    
      <!-- Navigation Tabs (mejorado para móviles) -->
      <div class="mb-6 no-print">
        <nav class="flex flex-wrap gap-1 bg-gray-100 dark:bg-gray-800 rounded-xl p-1">
          <button v-for="tab in tabs" :key="tab.id"
                  @click="setActiveTab(tab.id)"
                  :class="[
                    'flex-1 min-w-0 py-2 px-2 sm:px-4 rounded-lg font-medium text-[10px] sm:text-sm transition-all duration-300',
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
  <div class="grid grid-cols-2 gap-3 items-end w-full">

    <!-- Fecha Inicio -->
    <div class="input-group w-full">
      <label class="block text-[8px] sm:text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">
        Fecha Inicio
      </label>
      <input v-model="dashboardStartDate" type="date"
             @change="updateDashboard"
             class="w-full px-2 py-1 text-[8px] sm:text-sm border border-gray-300 dark:border-gray-600 rounded-md 
                    focus:ring-2 focus:ring-primary focus:border-primary 
                    dark:bg-gray-700 dark:text-white">
    </div>

    <!-- Fecha Fin -->
    <div class="input-group w-full">
      <label class="block text-[8px] sm:text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">
        Fecha Fin
      </label>
      <input v-model="dashboardEndDate" type="date"
             @change="updateDashboard"
             class="w-full px-2 py-1 text-[8px] sm:text-sm border border-gray-300 dark:border-gray-600 rounded-md 
                    focus:ring-2 focus:ring-primary focus:border-primary 
                    dark:bg-gray-700 dark:text-white">
    </div>

    <!-- Botón "Todos" -->
<div class="flex items-end w-full">
  <button @click="resetDashboardDates"
          class="w-full flex justify-center items-center gap-1 
                 px-3 py-2 rounded-lg font-medium transition-all duration-300 
                 text-[10px] sm:text-sm 
                 bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 
                 text-gray-900 dark:text-white">
    🔄 Todos
  </button>
</div>

<!-- Botón "Imprimir" -->
<div class="w-full">
  <button @click="printDashboard"
          class="w-full flex justify-center items-center gap-1 
                 px-3 py-2 rounded-lg font-medium transition-all duration-300 
                 text-[10px] sm:text-sm 
                 bg-green-600 hover:bg-green-700 text-white">
    🖨 Imprimir
  </button>
</div>


  </div>
</div>


        <!-- KPIs Overview -->
        <div id="dashboard-content"> 
           
         <div class="grid grid-cols-2 xl:grid-cols-4 gap-4 sm:gap-6 mb-6 print:grid-cols-2 print:gap-4 print:mb-4 print:max-w-2xl print:mx-auto">
          
           <div class="bg-blue-50 dark:bg-blue-800 print:bg-blue-50 print:dark:bg-blue-50 p-4 sm:p-6 rounded-lg print:border print:border-0 print:ring-1 print:ring-gray-100">
  <div class="flex flex-col items-center text-center">
    <p class="text-sm font-medium text-blue-600 dark:text-white print:text-black mb-2 w-full">Eventos en Rango</p>
    <div class="flex items-center justify-center">
      <p class="text-2xl sm:text-3xl font-bold text-blue-900 dark:text-blue-100 print:text-black">{{ getFilteredEvents().length }}</p>
      <div class="w-8 h-8 sm:w-10 sm:h-10 bg-blue-500 rounded-full flex items-center justify-center ml-2 print:hidden">
        <svg class="w-4 h-4 sm:w-5 sm:h-5 text-white print:text-gray-600" fill="currentColor" viewBox="0 0 20 20">
          <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
      </div>
    </div>
  </div>
</div>
           
<!-- Cámaras Activas -->
<div class="bg-green-50 dark:bg-green-800 print:bg-green-50 print:dark:bg-green-50 p-4 sm:p-6 rounded-lg print:border print:border-0 print:ring-1 print:ring-gray-100">
  <div class="flex flex-col items-center text-center">
    <p class="text-sm font-medium text-green-600 dark:text-white print:text-black mb-2 w-full">Cámaras Activas</p>
    <div class="flex items-center justify-center">
      <p class="text-2xl sm:text-3xl font-bold text-green-900 dark:text-green-100 print:text-black">{{ getTotalOnlineCameras() }}/{{ getTotalCameras() }}</p>
      <div class="w-8 h-8 sm:w-10 sm:h-10 bg-green-500 rounded-full flex items-center justify-center ml-2 print:hidden">
        <svg class="w-4 h-4 sm:w-5 sm:h-5 text-white" fill="currentColor" viewBox="0 0 20 20">
          <path d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z"/>
        </svg>
      </div>
    </div>
  </div>
</div>

<!-- Recursos Desplegados -->
<div class="bg-yellow-50 dark:bg-yellow-800 print:bg-yellow-50 print:dark:bg-yellow-50 p-4 sm:p-6 rounded-lg print:border print:border-0 print:ring-1 print:ring-gray-100">
  <div class="flex flex-col items-center text-center">
    <p class="text-sm font-medium text-yellow-600 dark:text-white print:text-black mb-2 w-full">Recursos Desplegados</p>
    <div class="flex items-center justify-center">
      <p class="text-2xl sm:text-3xl font-bold text-yellow-900 dark:text-yellow-100 print:text-black">{{ getTotalResourcesDeployedFiltered() }}</p>
      <div class="w-8 h-8 sm:w-10 sm:h-10 bg-yellow-500 rounded-full flex items-center justify-center ml-2 print:hidden">
        <svg class="w-4 h-4 sm:w-5 sm:h-5 text-white" fill="currentColor" viewBox="0 0 20 20">
          <path d="M13 6a3 3 0 11-6 0 3 3 0 016 0zM18 8a2 2 0 11-4 0 2 2 0 014 0zM14 15a4 4 0 00-8 0v3h8v-3z"/>
        </svg>
      </div>
    </div>
  </div>
</div>

<!-- Tiempo Resp. Prom -->
<div class="bg-purple-50 dark:bg-purple-800 print:bg-purple-50 print:dark:bg-purple-50 p-4 sm:p-6 rounded-lg print:border print:border-0 print:ring-1 print:ring-gray-100">
  <div class="flex flex-col items-center text-center">
    <p class="text-sm font-medium text-purple-600 dark:text-white print:text-black mb-2 w-full">Tiempo Resp. Prom</p>
    <div class="flex items-center justify-center">
      <p class="text-2xl sm:text-3xl font-bold text-purple-900 dark:text-purple-100 print:text-black">{{ getAverageResponseTimeFiltered() }} min</p>
      <div class="w-8 h-8 sm:w-10 sm:h-10 bg-purple-500 rounded-full flex items-center justify-center ml-2 print:hidden">
        <svg class="w-4 h-4 sm:w-5 sm:h-5 text-white" fill="currentColor" viewBox="0 0 20 20">
          <path d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-12a1 1 0 10-2 0v4a1 1 0 00.293.707l2.828 2.829a1 1 0 101.415-1.415L11 9.586V6z"/>
        </svg>
      </div>
    </div>
  </div>
</div>
</div>

<!-- Additional Stats - Resumen por Prioridad (SOLO, A LO ANCHO) -->
<div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-2 sm:p-6 border border-gray-200 dark:border-gray-700 mb-6 sm:mb-10 print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
  <h3 class="text-sm sm:text-lg font-semibold text-gray-900 dark:text-white print:text-black mb-3 sm:mb-6 flex items-center justify-center sm:justify-start">
    <span class="w-4 h-4 sm:w-8 sm:h-8 bg-red-100 dark:bg-red-900 rounded-lg flex items-center justify-center mr-2 sm:mr-3 print:bg-gray-300">📋</span>
    <span>Resumen por Prioridad</span>
  </h3> 
  <!-- Grid de 4 columnas incluso en móvil -->
  <div class="grid grid-cols-4 gap-1 sm:gap-4 text-center">
    
    <!-- Críticos -->
<div class="p-1 sm:p-4 bg-red-50 dark:bg-red-800 rounded-xl border border-red-200 dark:border-red-700 
            flex flex-col items-center justify-center 
            print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
  <div class="text-[14px] sm:text-4xl font-bold text-red-600 dark:text-white print:text-black">
    {{ getEventsByPriorityFiltered('critica') }}
  </div>
  <div class="text-[6px] sm:text-sm text-red-600 dark:text-white print:text-black mt-1 sm:mt-2 font-medium">
    🔴 Críticos
  </div>
</div>

<!-- Altos -->
<div class="p-1 sm:p-4 bg-orange-50 dark:bg-orange-800 rounded-xl border border-orange-200 dark:border-orange-700 
            flex flex-col items-center justify-center 
            print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
  <div class="text-[14px] sm:text-4xl font-bold text-orange-600 dark:text-white print:text-black">
    {{ getEventsByPriorityFiltered('alta') }}
  </div>
  <div class="text-[6px] sm:text-sm text-orange-600 dark:text-white print:text-black mt-1 sm:mt-2 font-medium">
    🟠 Altos
  </div>
</div>

<!-- Medios -->
<div class="p-1 sm:p-4 bg-yellow-50 dark:bg-yellow-800 rounded-xl border border-yellow-200 dark:border-yellow-700 
            flex flex-col items-center justify-center 
            print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
  <div class="text-[14px] sm:text-4xl font-bold text-yellow-600 dark:text-white print:text-black">
    {{ getEventsByPriorityFiltered('media') }}
  </div>
  <div class="text-[6px] sm:text-sm text-yellow-600 dark:text-white print:text-black mt-1 sm:mt-2 font-medium">
    🟡 Medios
  </div>
</div>

<!-- Bajos -->
<div class="p-1 sm:p-4 bg-green-50 dark:bg-green-800 rounded-xl border border-green-200 dark:border-green-700 
            flex flex-col items-center justify-center 
            print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
  <div class="text-[14px] sm:text-4xl font-bold text-green-600 dark:text-white print:text-black">
    {{ getEventsByPriorityFiltered('baja') }}
  </div>
  <div class="text-[6px] sm:text-sm text-green-600 dark:text-white print:text-black mt-1 sm:mt-2 font-medium">
    🟢 Bajos
  </div>
</div>


  </div>
</div>
          
          <!-- Charts Row 1 -->
          <div class="grid grid-cols-1 xl:grid-cols-2 print:grid-cols-2 gap-4 sm:gap-6 mb-6">
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-4 sm:p-6 border border-gray-200 dark:border-gray-700 print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white print:text-black mb-4 flex items-center">
                <span class="w-6 h-6 sm:w-8 sm:h-8 bg-blue-100 dark:bg-blue-900 rounded-lg flex items-center justify-center mr-3 print:bg-gray-300">📊</span>
                Eventos por Tipo
              </h3>
              <div class="relative h-48 sm:h-64 print:h-40 w-full">
                <canvas id="eventsTypeChart" class="w-full h-full"></canvas>
              </div>
            </div>
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-4 sm:p-6 border border-gray-200 dark:border-gray-700 print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white print:text-black mb-4 flex items-center">
                <span class="w-6 h-6 sm:w-8 sm:h-8 bg-green-100 dark:bg-green-900 rounded-lg flex items-center justify-center mr-3 print:bg-gray-300">🏢</span>
                Distribución por Institución
              </h3>
              <div class="relative h-48 sm:h-64 print:h-40 w-full">
                <canvas id="institutionsChart" class="w-full h-full"></canvas>
              </div>
            </div>
          </div>

          <!-- Charts Row 2 -->
          <div class="grid grid-cols-1 xl:grid-cols-2 print:grid-cols-2 gap-4 sm:gap-6 mb-6">
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-4 sm:p-6 border border-gray-200 dark:border-gray-700 print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white print:text-black mb-4 flex items-center">
                <span class="w-6 h-6 sm:w-8 sm:h-8 bg-orange-100 dark:bg-orange-900 rounded-lg flex items-center justify-center mr-3 print:bg-gray-300">📈</span>
                Eventos por Hora
              </h3>
              <div class="relative h-48 sm:h-64 print:h-40 w-full">
                <canvas id="hourlyChart" class="w-full h-full"></canvas>
              </div>
            </div>
            <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-4 sm:p-6 border border-gray-200 dark:border-gray-700 print:bg-white print:border-0 print:ring-1 print:ring-gray-100">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white print:text-black mb-4 flex items-center">
                <span class="w-6 h-6 sm:w-8 sm:h-8 bg-purple-100 dark:bg-purple-900 rounded-lg flex items-center justify-center mr-3 print:bg-gray-300">📍</span>
                Distribución por Comunidad
              </h3>
              <div class="relative h-48 sm:h-64 print:h-40 w-full">
                <canvas id="communitiesChart" class="w-full h-full"></canvas>
              </div>
            </div>
          </div>

        </div>

<!-- Footer del Dashboard para Impresión -->
<div class="dashboard-print-footer print:block hidden">
  <p>Municipalidad de Roatán</p>
  <p>📞 Número de Emergencias: *1101</p>
  <p><strong>Centro de Monitoreo Joseph Solomon</strong></p>
</div>

      </div>

      <!-- Turno Tab -->
      <div v-if="activeTab === 'turno'" class="space-y-6">
        <!-- Configuración inicial antes de activar turno -->
        <div v-if="!activeShift" class="space-y-6">
          <!-- Alert que se debe configurar primero -->
          <div class="bg-gradient-to-r from-yellow-50 to-orange-50 dark:from-yellow-900 dark:to-orange-900 border border-yellow-200 dark:border-yellow-800 rounded-xl p-4 sm:p-6">
            <div class="flex items-center">
              <span class="text-2xl sm:text-4xl mr-4">⚠️</span>
              <div>
                <p class="text-yellow-800 dark:text-yellow-200 font-bold text-base sm:text-lg">Complete la configuración antes de iniciar turno</p>
                <p class="text-yellow-700 dark:text-yellow-300 text-sm sm:text-base">Debe configurar recursos institucionales y sistema de videovigilancia.</p>
              </div>
            </div>
          </div>

          <!-- Botón para iniciar turno -->
          <div class="form-section">
            <div class="flex justify-center">
              <button @click="validateAndInitiateTurn" :disabled="!canInitiateTurn()" 
                      :class="canInitiateTurn() ? 'btn-primary' : 'btn-secondary opacity-50 cursor-not-allowed'"
                      class="text-base sm:text-lg py-3 sm:py-4 px-6 sm:px-8 w-full sm:w-auto">
                ▶ Iniciar Turno
              </button>
            </div>
            <div v-if="!canInitiateTurn()" class="text-center mt-4">
              <p class="text-red-600 dark:text-red-400 text-sm">
                Debe completar la configuración de recursos y cámaras antes de iniciar el turno
              </p>
            </div>
          </div>

          <!-- Sistema de Videovigilancia -->
          <div class="form-section">
            <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-8 h-8 sm:w-10 sm:h-10 bg-blue-100 dark:bg-blue-900 rounded-xl flex items-center justify-center mr-3">📹</span>
              Sistema de Videovigilancia por Comunidad
            </h2>
            <div class="grid grid-cols-1 xl:grid-cols-2 gap-4 sm:gap-6">
              <div v-for="community in communities" :key="community.id"
                   class="input-group border-l-4 border-l-blue-500">
                <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center mb-4">
                  <h3 class="text-lg font-semibold text-gray-900 dark:text-white flex items-center mb-2 lg:mb-0">
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
                <!-- Cambio 5: En vista móvil, poner Total de cámaras y cámaras activas en una misma fila -->
                <div class="grid grid-cols-2 gap-4">
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Total de Cámaras</label>
                    <input v-model.number="community.cameras.total" type="number" min="0"
                           class="w-full px-3 sm:px-4 py-2 sm:py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Cámaras Activas</label>
                    <input v-model.number="community.cameras.online" type="number" min="0" :max="community.cameras.total"
                           class="w-full px-3 sm:px-4 py-2 sm:py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Recursos Institucionales -->
<div class="form-section">
  <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
    <span class="w-8 h-8 sm:w-10 sm:h-10 bg-green-100 dark:bg-green-900 rounded-xl flex items-center justify-center mr-3">
      🚓
    </span>
    Recursos Institucionales Disponibles
  </h2>

  <div class="space-y-6">
    <div 
      v-for="institution in institutions" 
      :key="institution.id"
      class="input-group border-l-4 border-l-green-500 p-4 rounded-lg bg-gray-50 dark:bg-gray-800"
    >
      <div class="flex items-center mb-6">
        <span class="text-2xl sm:text-3xl mr-4">{{ institution.icon }}</span>
        <div class="flex-1">
          <h3 class="text-lg sm:text-xl font-semibold text-gray-900 dark:text-white">
            {{ institution.name }}
          </h3>
          <p class="text-sm text-gray-600 dark:text-gray-400">
            Recursos disponibles para despliegue
          </p>
        </div>
      </div>

      <!-- 🟢 2 recursos por fila -->
      <div class="grid grid-cols-2 gap-4">
        <div 
          v-for="resource in institution.resources" 
          :key="resource.id" 
          class="bg-white dark:bg-gray-700 p-4 rounded-lg border border-gray-200 dark:border-gray-600"
        >
          <label class="block text-xs sm:text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
            {{ resource.name }}
          </label>
          <input 
            v-model.number="resource.available" 
            type="number" 
            min="0"
            placeholder="Disponibles"
            class="w-full px-3 py-2 text-sm border border-gray-300 dark:border-gray-600 
                   rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500 
                   dark:bg-gray-800 dark:text-white transition-all duration-300"
          />
        </div>
      </div>
    </div>
  </div>
</div>

        </div>

        <!-- Configuración del Turno (solo visible cuando hay turno activo) -->
        <div v-if="activeShift" class="space-y-6">
          <!-- Información del Turno Activo -->
          <div class="form-section">
            <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center mb-6">
              <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white flex items-center mb-4 lg:mb-0">
                <span class="w-8 h-8 sm:w-10 sm:h-10 bg-green-100 dark:bg-green-900 rounded-xl flex items-center justify-center mr-3">👮</span>
                Turno Activo
              </h2>
              <button @click="showEndShiftModal = true" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg font-medium transition-all duration-300 flex items-center space-x-2 w-full lg:w-auto justify-center">
                <span>⏹</span>
                <span>Cerrar Turno</span>
              </button>
            </div>
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
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
            <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-8 h-8 sm:w-10 sm:h-10 bg-blue-100 dark:bg-blue-900 rounded-xl flex items-center justify-center mr-3">📹</span>
              Sistema de Videovigilancia - Estado Actual
            </h2>
            <div class="grid grid-cols-1 xl:grid-cols-2 gap-4 sm:gap-6">
              <div v-for="community in communities" :key="community.id"
                   class="input-group border-l-4 border-l-blue-500">
                <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center mb-4">
                  <h3 class="text-lg font-semibold text-gray-900 dark:text-white flex items-center mb-2 lg:mb-0">
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
                           class="w-full px-3 sm:px-4 py-2 sm:py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                  <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Cámaras Activas</label>
                    <input v-model.number="community.cameras.online" type="number" min="0" :max="community.cameras.total"
                           class="w-full px-3 sm:px-4 py-2 sm:py-3 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white transition-all duration-300">
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Recursos Institucionales (modo visualización) -->
          <div class="form-section">
            <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
              <span class="w-8 h-8 sm:w-10 sm:h-10 bg-green-100 dark:bg-green-900 rounded-xl flex items-center justify-center mr-3">🚓</span>
              Recursos Institucionales - Estado Actual
            </h2>
            <div class="space-y-6">
              <div v-for="institution in institutions" :key="institution.id"
                   class="input-group border-l-4 border-l-green-500">
                <div class="flex items-center mb-6">
                  <span class="text-2xl sm:text-3xl mr-4">{{ institution.icon }}</span>
                  <div class="flex-1">
                    <h3 class="text-lg sm:text-xl font-semibold text-gray-900 dark:text-white">{{ institution.name }}</h3>
                    <p class="text-sm text-gray-600 dark:text-gray-400">Recursos disponibles para despliegue</p>
                  </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-4">
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
        <div v-if="!activeShift" class="bg-gradient-to-r from-yellow-50 to-orange-50 dark:from-yellow-900 dark:to-orange-900 border border-yellow-200 dark:border-yellow-800 rounded-xl p-4 sm:p-6">
          <div class="flex items-center">
            <span class="text-2xl sm:text-4xl mr-4">⚠️</span>
            <div>
              <p class="text-yellow-800 dark:text-yellow-200 font-bold text-base sm:text-lg">No hay un turno activo</p>
              <p class="text-yellow-700 dark:text-yellow-300 text-sm sm:text-base">Debe iniciar un turno para registrar eventos.</p>
            </div>
          </div>
        </div>

        <!-- Agregar Evento -->
        <div v-else class="form-section">
          <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
            <span class="w-8 h-8 sm:w-10 sm:h-10 bg-primary rounded-xl flex items-center justify-center mr-3">📝</span>
            Registrar Nuevo Evento
          </h2>
          <form @submit.prevent="requestAgentCode" class="space-y-6">
            <!-- Información Básica -->
            <div class="input-group">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">ℹ️ Información Básica</h3>
              <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
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
              <div v-if="newEvent.type === 'llamada_falsa'" class="mb-4 p-3 bg-yellow-50 dark:bg-yellow-900/20 border border-yellow-200 dark:border-yellow-800 rounded-lg">
                <p class="text-sm text-yellow-800 dark:text-yellow-200">
                  ⚠️ La ubicación es opcional para llamadas falsas
                </p>
              </div>
              <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 mb-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Comunidad</label>
                  <select v-model="newEvent.location.community" @change="onCommunityChange" :required="newEvent.type !== 'llamada_falsa'"
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
              <div class="flex flex-col lg:flex-row items-start lg:items-center justify-between space-y-2 lg:space-y-0">
                <button type="button" @click="openMapModal" class="btn-primary w-full lg:w-auto">
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
              <div class="space-y-4 print:space-y-2">
                <div v-for="institution in institutions" :key="institution.id"
                     class="bg-gray-50 dark:bg-gray-600 rounded-lg p-4">
                  <div class="flex flex-col lg:flex-row items-start lg:items-center justify-between mb-3">
                    <label class="text-base sm:text-lg font-medium text-gray-700 dark:text-gray-300 flex items-center cursor-pointer mb-2 lg:mb-0">
                      <input type="checkbox" v-model="newEvent.institutionsUsed[institution.id]"
                             class="mr-3 w-4 h-4 sm:w-5 sm:h-5 rounded border-gray-300 text-primary focus:ring-primary">
                      <span class="text-xl sm:text-2xl mr-3">{{ institution.icon }}</span>
                      {{ institution.name }}
                    </label>
                    <div v-if="newEvent.institutionsUsed[institution.id]" class="text-sm text-gray-600 dark:text-gray-400">
                      <label class="block text-xs font-medium mb-1">T. Resp. Estimado (min)</label>
                      <input v-model.number="newEvent.responseTimeEstimated[institution.id]" type="number" min="0"
                             :placeholder="institution.responseTime"
                             class="w-16 sm:w-20 px-2 py-1 text-base border border-gray-300 dark:border-gray-600 rounded focus:ring-1 focus:ring-primary dark:bg-gray-700 dark:text-white">
                    </div>
                  </div>
                  <div v-if="newEvent.institutionsUsed[institution.id]" class="ml-0 lg:ml-8">
                    <label class="block text-sm font-medium text-gray-600 dark:text-gray-400 mb-3">Recursos Utilizados:</label>
                    <div class="space-y-2">
                      <div v-for="resource in institution.resources" :key="resource.id" class="flex flex-col lg:flex-row items-start lg:items-center space-y-2 lg:space-y-0 lg:space-x-3">
                        <div class="flex items-center">
                          <input type="checkbox"
                                 v-model="newEvent.resourcesUsed[institution.id + '_' + resource.id]"
                                 :id="'res_' + institution.id + '_' + resource.id"
                                 class="w-4 h-4 rounded border-gray-300 text-primary focus:ring-primary">
                          <label :for="'res_' + institution.id + '_' + resource.id" class="text-sm flex-1 font-medium ml-2">
                            {{ resource.name }}
                          </label>
                        </div>
                        <input v-if="newEvent.resourcesUsed[institution.id + '_' + resource.id]"
                               v-model.number="newEvent.resourceSpecifications[institution.id + '_' + resource.id]"
                               type="number"
                               min="0"
                               placeholder="Cantidad"
                               class="w-16 sm:w-20 px-2 py-1 text-base border border-gray-300 dark:border-gray-600 rounded focus:ring-1 focus:ring-primary dark:bg-gray-700 dark:text-white">
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Afectaciones -->
            <div class="input-group">
              <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">📊 Afectaciones</h3>
              <div class="space-y-4">
                <div v-for="category in impactCategories" :key="category.id" 
                     class="bg-gray-50 dark:bg-gray-600 border-2 border-gray-200 dark:border-gray-600 rounded-lg overflow-hidden"
                     :class="category.bgClass">
                  
                  <!-- Category Header -->
                  <div class="px-4 py-3 bg-white dark:bg-gray-600">
                    <div class="flex items-center justify-between">
                      <div class="flex items-center gap-3">
                        <input type="checkbox"
                               :id="'category_' + category.id"
                               @change="toggleCategoryImpacts(category)"
                               :checked="isCategorySelected(category)"
                               class="w-5 h-5 rounded border-gray-300 text-primary focus:ring-primary">
                        <label :for="'category_' + category.id" class="flex items-center gap-2 cursor-pointer">
                          <span class="text-xl">{{ category.icon }}</span>
                          <span class="font-semibold text-gray-900 dark:text-white">
                            {{ category.label }}
                          </span>
                        </label>
                        <span v-if="getCategoryActiveCount(category) > 0" 
                              class="px-2 py-1 text-xs font-bold rounded-full bg-green-500 text-white">
                          {{ getCategoryActiveCount(category) }}
                        </span>
                      </div>
                      <button @click="toggleCategory(category.id)"
                              type="button"
                              class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300">
                        <svg class="w-5 h-5 transition-transform" 
                             :class="{ 'rotate-180': expandedCategories[category.id] }"
                             fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
                        </svg>
                      </button>
                    </div>
                  </div>

                  <!-- Category Content -->
                  <div v-show="expandedCategories[category.id]" 
                       class="px-4 py-4 bg-white dark:bg-gray-600 border-t border-gray-200 dark:border-gray-700">
                    <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
                      <div v-for="impact in category.impacts" :key="impact.id" 
                           class="bg-gray-50 dark:bg-gray-700 p-3 rounded-lg">
                        <div class="flex items-center mb-2">
                          <input type="checkbox"
                                 v-model="newEvent.impacts[impact.id]"
                                 :id="'impact_' + impact.id"
                                 :disabled="!expandedCategories[category.id]"
                                 class="mr-3 w-4 h-4 rounded border-gray-300 text-primary focus:ring-primary disabled:opacity-50">
                          <label :for="'impact_' + impact.id" 
                                 class="text-sm font-medium text-gray-700 dark:text-gray-300 disabled:opacity-50">
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
                </div>
              </div>
            </div>

            <button type="submit" class="btn-primary w-full text-base sm:text-lg py-3 sm:py-4 px-6 sm:px-8">
              📝 Registrar Evento
            </button>
          </form>
        </div>

        <!-- Lista de Eventos del Turno Actual -->
        <div class="form-section">
          <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center mb-6">
            <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white flex items-center mb-4 lg:mb-0">
              <span class="w-8 h-8 sm:w-10 sm:h-10 bg-orange-100 dark:bg-orange-900 rounded-xl flex items-center justify-center mr-3">📋</span>
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
                 class="bg-white dark:bg-gray-700 border border-gray-200 dark:border-gray-600 rounded-xl p-4 sm:p-6 hover:shadow-lg transition-all duration-300">
              <div class="flex flex-col xl:flex-row justify-between items-start xl:items-center mb-4 space-y-2 xl:space-y-0">
                <div class="flex flex-wrap items-center gap-2 print:gap-1">
                  <span class="text-base font-bold text-primary bg-blue-100 dark:bg-blue-900 px-3 py-1 rounded-lg">{{ event.code }}</span>
                  <span class="text-sm font-medium text-gray-900 dark:text-white bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ event.time }}</span>
                  <span :class="getPriorityBadgeClass(event.priority)"
                        class="px-3 py-1 text-sm font-medium rounded-lg">
                    {{ event.priority.toUpperCase() }}
                  </span>
                  <span class="text-sm text-gray-600 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ getEventTypeName(event.type) }}</span>
                  <span :class="getStatusBadgeClass(event.status)"
                        class="px-3 py-1 text-sm font-medium rounded-lg">
                    {{ getStatusText(event.status) }}
                  </span>
                </div>
                <div class="flex flex-wrap items-center gap-2 print:gap-1">
                  <button @click="editEventStatus(event)"
                          class="px-3 py-2 text-sm bg-blue-100 dark:bg-blue-900 text-blue-700 dark:text-blue-300 rounded-lg hover:bg-blue-200 dark:hover:bg-blue-800 transition-colors flex items-center space-x-2">
                    <span>⚙️</span>
                  </button>
                  <button @click="editEvent(event)"
                          class="px-3 py-2 text-sm bg-green-100 dark:bg-green-900 text-green-700 dark:text-green-300 rounded-lg hover:bg-green-200 dark:hover:bg-green-800 transition-colors flex items-center space-x-2">
                    <span>✏️</span>
                  </button>
                  <button @click="confirmDeleteEvent(event)"
                          class="px-3 py-2 text-sm bg-red-100 dark:bg-red-900 text-red-700 dark:text-red-300 rounded-lg hover:bg-red-200 dark:hover:bg-red-800 transition-colors flex items-center space-x-2">
                    <span>🗑️</span>
                  </button>
                  <div class="text-xs text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-2 py-1 rounded">
                    Por: {{ event.registeredBy }}
                  </div>
                  <div v-if="event.actualResponseTime" class="text-xs text-green-600 dark:text-green-400 bg-green-100 dark:bg-green-900 px-2 py-1 rounded">
                    T. Resp: {{ event.actualResponseTime }} min
                  </div>
                </div>
              </div>

              <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 text-sm text-gray-500 dark:text-gray-400">
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
          <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
            <span class="w-8 h-8 sm:w-10 sm:h-10 bg-indigo-100 dark:bg-indigo-900 rounded-xl flex items-center justify-center mr-3">📚</span>
            Historial de Eventos
          </h2>
          
          <!-- Cambio 5: En vista móvil, poner 2 filtros por línea -->
          <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
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

          <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 mb-6">
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
          <div class="flex flex-col xl:flex-row justify-between items-start xl:items-center space-y-4 xl:space-y-0">
            <div>
              <h3 class="font-semibold text-blue-900 dark:text-blue-200">Resultados del Filtro</h3>
              <p class="text-sm text-blue-700 dark:text-blue-300">
                Mostrando {{ getFilteredHistorialEvents().length }} de {{ allEvents.length }} eventos totales
              </p>
            </div>
            <div class="flex flex-wrap gap-4 text-sm">
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
                 class="bg-white dark:bg-gray-700 border border-gray-200 dark:border-gray-600 rounded-xl p-4 sm:p-6 hover:shadow-lg transition-all duration-300">
              <div class="flex flex-col xl:flex-row justify-between items-start xl:items-center mb-4 space-y-2 xl:space-y-0">
                <div class="flex flex-wrap items-center gap-2 print:gap-1">
                  <span class="text-base font-bold text-primary bg-blue-100 dark:bg-blue-900 px-3 py-1 rounded-lg">{{ event.code }}</span>
                  <span class="text-sm font-medium text-gray-900 dark:text-white bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ event.date }} - {{ event.time }}</span>
                  <span :class="getPriorityBadgeClass(event.priority)"
                        class="px-3 py-1 text-sm font-medium rounded-lg">
                    {{ event.priority.toUpperCase() }}
                  </span>
                  <span class="text-sm text-gray-600 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-3 py-1 rounded-lg">{{ getEventTypeName(event.type) }}</span>
                  <span :class="getStatusBadgeClass(event.status)"
                        class="px-3 py-1 text-sm font-medium rounded-lg">
                    {{ getStatusText(event.status) }}
                  </span>
                </div>
                <div class="flex flex-wrap items-center gap-2 print:gap-1">
                  <button v-if="event.coordinates.lat && event.coordinates.lng" @click="showEventOnMap(event)"
                          class="px-3 py-2 text-sm bg-green-100 dark:bg-green-900 text-green-700 dark:text-green-300 rounded-lg hover:bg-green-200 dark:hover:bg-green-800 transition-colors flex items-center space-x-2">
                    <span>🗺</span>
                    <span>Ver en Mapa</span>
                  </button>
                  <div v-if="event.actualResponseTime" class="text-sm text-blue-600 dark:text-blue-400 bg-blue-100 dark:bg-blue-900 px-3 py-1 rounded-lg">
                    T. Respuesta: {{ event.actualResponseTime }} min
                  </div>
                  <div class="text-xs text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-2 py-1 rounded">
                    Por: {{ event.registeredBy }}
                  </div>
                  <div class="text-xs text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-600 px-2 py-1 rounded">
                    Turno: {{ event.shiftSupervisor || 'N/A' }}
                  </div>
                </div>
              </div>

              <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 text-sm text-gray-500 dark:text-gray-400">
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
          <h2 class="text-xl sm:text-2xl font-bold text-gray-900 dark:text-white mb-6 flex items-center">
            <span class="w-8 h-8 sm:w-10 sm:h-10 bg-purple-100 dark:bg-purple-900 rounded-xl flex items-center justify-center mr-3">📊</span>
            Generar Reportes
          </h2>

          <div class="grid grid-cols-1 xl:grid-cols-2 gap-4 sm:gap-6 mb-8">
            <!-- Reportes Automáticos -->
            <div class="report-type-card" :class="{ active: reportCategory === 'automatic' }" @click="reportCategory = 'automatic'">
              <div class="text-center">
                <div class="text-3xl sm:text-4xl mb-3">🤖</div>
                <h3 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-2">Reportes Automáticos</h3>
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
                <div class="text-3xl sm:text-4xl mb-3">📝</div>
                <h3 class="text-lg sm:text-xl font-bold text-gray-900 dark:text-white mb-2">Reportes Manuales</h3>
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
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 mb-6">
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
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 mb-6">
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

            <!-- Campos específicos según tipo -->
            <div v-if="selectedManualReportType === 'weather'" class="space-y-4">
              <!-- Tipo de alerta y riesgo -->
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Alerta</label>
                  <select v-model="manualReport.weather.alertType"
                    class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="" disabled selected class="text-gray-400 dark:text-gray-500">
                      Seleccione un tipo de alerta
                    </option>
                    <optgroup label="-Meteorológicos-">
                      <option value="lluvia">🌧️ Lluvias intensas</option>
                      <option value="tormenta">⛈️ Tormenta eléctrica</option>
                      <option value="calor">🌡️ Altas temperaturas</option>
                      <option value="frio">🧊 Bajas temperaturas</option>
                      <option value="neblina">🌫️ Neblina densa</option>
                    </optgroup>
                    <optgroup label="-Hidrológicos-">
                      <option value="inundacion">🌊 Inundaciones</option>
                      <option value="huracan">🌀 Huracán / ciclón</option>
                      <option value="mar">🌊 Condiciones marinas</option>
                    </optgroup>
                    <optgroup label="-Geológicos-">
                      <option value="sismo">🌎 Sismo/Terremoto</option>
                      <option value="tsunami">🌊 Tsunami</option>
                    </optgroup>
                    <optgroup label="-Ambientales y otros-">
                      <option value="contaminacion">☣️ Contaminación</option>
                      <option value="sargazo">🌿 Afluencia de sargazo</option>
                      <option value="epidemia">🦠 Emergencia sanitaria</option>
                    </optgroup>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Fenómeno</label>
                  <select v-model="manualReport.weather.eventType"
                          :disabled="!manualReport.weather.alertType"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="" disabled selected class="text-gray-400 dark:text-gray-500">
                      {{ manualReport.weather.alertType ? 'Seleccione un fenómeno' : 'Seleccione primero el tipo de alerta' }}
                    </option>
                    
                    <!-- Fenómenos de Lluvia -->
                    <template v-if="manualReport.weather.alertType === 'lluvia'">
                      <option value="depresion">🌧️ Depresión Tropical</option>
                      <option value="tormenta">🌪️ Tormenta Tropical</option> 
                      <option value="post_ciclone">🌪️ Post-Ciclón Tropical</option>
                      <option value="onda">🌬️ Onda Tropical</option>
                      <option value="frio">🧊 Frente Frío</option>
                    </template>
                    
                    <!-- Fenómenos de Tormenta -->
                    <template v-else-if="manualReport.weather.alertType === 'tormenta'">
                      <option value="electrica">⚡ Tormenta Eléctrica</option>
                      <option value="frio">🧊 Frente Frío</option>
                    </template>
                    
                    <!-- Sargazo -->
                    <template v-else-if="manualReport.weather.alertType === 'sargazo'">
                      <option value="sargazo_leve">🌿 Afluencia Leve</option>
                      <option value="sargazo_moderado">🌿🌿 Afluencia Moderada</option>
                      <option value="sargazo_masivo">🌿🌿🌿 Afluencia Masiva</option>
                    </template>

                    <!-- Fenómenos de Calor -->
                    <template v-else-if="manualReport.weather.alertType === 'calor'">
                      <option value="ola_calor">🌡️ Ola de Calor</option>
                    </template>
                    
                    <!-- Fenómenos de Frío -->
                    <template v-else-if="manualReport.weather.alertType === 'frio'">
                      <option value="ola_frio">❄️ Ola de Frío</option>
                      <option value="frente_frio">❄️ Frente de Frío</option>
                    </template>
                    
                    <!-- Otros fenómenos meteorológicos -->
                    <template v-else-if="manualReport.weather.alertType === 'neblina'">
                      <option value="neblina_espesa">🌫️ Niebla Densa</option>
                      <option value="bruma">🌁 Bruma</option>
                    </template>
                    
                    <!-- Fenómenos Hidrológicos -->
                    <template v-else-if="manualReport.weather.alertType === 'inundacion'">
                      <option value="inundacion_repentina">🌊 Inundación Repentina</option>
                      <option value="inundacion_fluvial">🌊 Inundación Fluvial</option>
                      <option value="inundacion_pluvial">🌊 Inundación Pluvial</option>
                    </template>
                    
                    <template v-else-if="manualReport.weather.alertType === 'huracan'">
                      <option value="huracan_cat1">🌀 Huracán Categoría 1-2</option>
                      <option value="huracan_cat3">🌀🌀 Huracán Categoría 3-4</option>
                      <option value="huracan_cat5">🌀🌀🌀 Huracán Categoría 5</option>
                    </template>
                    
                    <!-- Fenómenos Geológicos -->
                    <template v-else-if="manualReport.weather.alertType === 'sismo'">
                      <option value="temblor">🌎 Temblor</option>
                      <option value="terremoto">🌍 Terremoto</option>
                      <option value="replica">🔄 Réplica Sísmica</option>
                    </template> 

                    <template v-else-if="manualReport.weather.alertType === 'tsunami'">
                      <option value="alerta_tsunami">🌊 Alerta de Tsunami</option>
                      <option value="advertencia_tsunami">⚠️ Advertencia de Tsunami</option>
                    </template> 

                    <template v-else-if="manualReport.weather.alertType === 'contaminacion'">
                      <option value="contaminacion_aire">💨 Contaminación del Aire</option>
                      <option value="derrame_quimico">☣️ Derrame Químico</option>
                      <option value="derrame_petroleo">🛢️ Derrame de Petróleo</option>
                    </template>

                    <!-- Condiciones Marinas -->
                    <template v-else-if="manualReport.weather.alertType === 'mar'">
                      <option value="corrientes_fuertes">🌊 Corrientes Fuertes</option>
                      <option value="marejada">🌊 Marejada</option>
                      <option value="mar_picado">🌊🌊 Mar Picado</option>
                      <option value="mar_arriba">🌊🌊🌊 Mar Arriba (Peligroso)</option>
                    </template>

                    
                    <template v-else-if="manualReport.weather.alertType === 'epidemia'">
                      <option value="brote">🦠 Brote Epidemiológico</option>
                      <option value="epidemia">🌍 Epidemia</option>
                      <option value="pandemia">🌐 Pandemia</option>
                    </template>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Nivel de Alerta</label>
                  <select v-model="manualReport.weather.riskLevel"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="" disabled selected class="text-gray-400 dark:text-gray-500">Seleccione un nivel</option>
                    <option value="blanco">⚪ Sin Alerta</option>
                    <option value="verde">🟢 Verde - Alerta Baja</option>
                    <option value="amarillo">🟡 Amarillo - Alerta Media</option> 
                    <option value="rojo">🔴 Rojo - Alerta Alta</option>
                  </select>
                </div>
              </div>

              <!-- Sección de dos columnas: Campos a la izquierda, comunidades a la derecha -->
              <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Columna izquierda - Campos del formulario -->
                <div class="lg:col-span-2 space-y-4">
                  <div class="grid grid-cols-1 md:grid-cols-2 gap-4">

                    <div>
                      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Probabilidad de Impacto en Roatán</label>
                      <select v-model="manualReport.weather.probability"
                              class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                        <option value="" disabled selected class="text-gray-400 dark:text-gray-500">Seleccione una probabilidad</option>
                        <option value="baja">Baja probabilidad (<30%)</option>
                        <option value="media">Media probabilidad (31–70%)</option>
                        <option value="alta">Alta probabilidad (>70%)</option>
                        <option value="impacto">Ya impactó</option>
                      </select>
                    </div>
                    
                    <div v-if="['media', 'alta', 'impacto'].includes(manualReport.weather.probability)">
                      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
                        {{ manualReport.weather.probability === 'impacto' ? 'Tiempo de Impacto' : 'Tiempo de Impacto Estimado' }}
                      </label>
                      <input v-model="manualReport.weather.impactTime" type="text"
                            :placeholder="manualReport.weather.probability === 'impacto' ? 'Ej: 12:00 PM - 24/10/2023' : 'Ej: 1 hora - 3 días'"
                            class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    </div>

                    <div v-if="['lluvia', 'tormenta', 'huracan', 'inundacion'].includes(manualReport.weather.alertType)">
                      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Intensidad</label>
                      <select v-model="manualReport.weather.intensity"
                              class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                        <option value="" disabled selected class="text-gray-400 dark:text-gray-500">Seleccione una intensidad</option>
                        <option value="ligera">Ligera (&lt; 10 mm/h)</option>
                        <option value="moderada">Moderada (10–25 mm/h)</option>
                        <option value="fuerte">Fuerte (25–50 mm/h)</option>
                        <option value="intensa">Intensa (&gt; 50 mm/h)</option>
                      </select>
                    </div>

                    <div v-if="manualReport.weather.probability === 'impacto'">
                      <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
                        Duración Estimada
                        <span class="text-red-500">*</span>
                      </label>
                      <input v-model="manualReport.weather.duration" type="text"
                            placeholder="Ej: 2 horas"
                            class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary bg-white dark:bg-gray-700 text-gray-900 dark:text-white">
                    </div>

                  </div>
                </div>
                
                <!-- Columna derecha - Zonas afectadas -->
                  <div v-if="['media', 'alta', 'impacto'].includes(manualReport.weather.probability)" class="lg:border-l lg:pl-6 lg:border-gray-200 dark:lg:border-gray-700">
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
                    Posibles Zonas Afectadas
                    <span v-if="manualReport.weather.probability === 'impacto'" class="text-red-500">*</span>
                  </label>
                  <div class="space-y-2">
                    <div class="flex items-center">
                      <input type="checkbox" value="municipio" v-model="manualReport.weather.affectedZones"
                            :disabled="manualReport.weather.probability == 'baja'"
                            :class="[
                              'h-4 w-4 text-primary border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600',
                              manualReport.weather.probability == 'baja' ? 'opacity-50 cursor-not-allowed' : ''
                            ]" />
                      <label :class="[
                        'ml-2 text-sm',
                        manualReport.weather.probability == 'baja' ? 'text-gray-400 dark:text-gray-500' : 'text-gray-700 dark:text-gray-300'
                      ]">
                        Todo el municipio
                      </label>
                    </div>

                    <!-- Comunidades en una cuadrícula de 2 columnas -->
                    <div class="grid grid-cols-2 gap-x-4 gap-y-2 max-h-60 overflow-y-auto p-1">
                      <div v-for="community in communities" :key="community.id" class="flex items-center">
                        <input type="checkbox" :value="community.id" v-model="manualReport.weather.affectedZones"
                              :disabled="manualReport.weather.probability == 'baja'"
                              :class="[
                                'h-4 w-4 text-primary border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600',
                                manualReport.weather.probability == 'baja' ? 'opacity-50 cursor-not-allowed' : ''
                              ]" />
                        <label :class="[
                          'ml-2 text-sm',
                          manualReport.weather.probability == 'baja' ? 'text-gray-400 dark:text-gray-500' : 'text-gray-700 dark:text-gray-300'
                        ]">
                          {{ community.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div v-if="selectedManualReportType === 'security'" class="space-y-4">
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
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

            <div v-if="selectedManualReportType === 'traffic'" class="space-y-4">
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Aviso</label>
                  <select v-model="manualReport.traffic.alertType"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="cierre_vial">🚫 Cierre Vial</option>
                    <option value="desvio">↗️ Desvío de Ruta</option>
                    <option value="obras">🚧 Obras en Progreso</option>
                    <option value="accidente">🚗 Accidente de Tránsito</option>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Duración Estimada</label>
                  <input v-model="manualReport.traffic.duration" type="text"
                         placeholder="Ej: 2 horas, Todo el día"
                         class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                </div>
              </div>
            </div>

            <div v-if="selectedManualReportType === 'public_service'" class="space-y-4">
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Tipo de Servicio</label>
                  <select v-model="manualReport.publicService.serviceType"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="energia">⚡ Energía Eléctrica</option>
                    <option value="agua">💧 Agua Potable</option>
                    <option value="internet">🌐 Internet/Telecomunicaciones</option>
                    <option value="gas">🔥 Gas</option>
                    <option value="transporte">🚌 Transporte Público</option>
                  </select>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Zona Afectada</label>
                  <select v-model="manualReport.publicService.affectedZone"
                          class="w-full px-3 py-2 text-base border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-primary focus:border-primary dark:bg-gray-700 dark:text-white">
                    <option value="municipio">Todo el municipio</option>
                    <option v-for="community in communities" :key="community.id" :value="community.id">{{ community.name }}</option>
                  </select>
                </div>
              </div>
            </div>

            <div class="space-y-4 print:space-y-2">
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Descripción Detallada</label>
                <textarea v-model="manualReport.description" rows="4"
                          placeholder="Describa la situación, medidas a tomar y recomendaciones..."
                          class="w-full px-3 py-2 text-base border-0 border-b-2 border-white rounded-none focus:ring-2 focus:ring-0 focus:border-white dark:bg-gray-700 dark:text-white"></textarea>
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Recomendaciones para la Población</label>
                <textarea v-model="manualReport.recommendations" rows="3"
                          placeholder="Instrucciones específicas para los ciudadanos..."
                          class="w-full px-3 py-2 text-base border-0 border-b-2 border-white rounded-none focus:ring-2 focus:ring-0 focus:border-white dark:bg-gray-700 dark:text-white"></textarea>
              </div>

              <!-- 📸 Subir fotos -->
              <div>
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
                  Anexos
                </label>

                <input 
                  type="file" 
                  multiple 
                  accept="image/*" 
                  @change="handleImageUpload"
                  class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer 
                  dark:text-gray-300 dark:bg-gray-700 dark:border-gray-600 focus:outline-none" 
                />

                <!-- Vista previa cuadrada -->
                <div v-if="uploadedImages && uploadedImages.length" class="mt-4">
                  <div class="flex flex-wrap gap-3">
                    <div 
                      v-for="(img, index) in uploadedImages" 
                      :key="index" 
                      class="relative group w-24 h-24 sm:w-32 sm:h-32 lg:w-[14rem] lg:h-[14rem]" 
                    >
                      <!-- Imagen cuadrada -->
                      <img 
                        :src="img.preview" 
                        class="w-full h-full object-cover rounded-md border border-gray-200 dark:border-gray-600 
                        shadow-sm transition-transform duration-200 group-hover:scale-105"
                        :alt="`Imagen ${index + 1}`"
                      />

                      <!-- Botón eliminar -->
                      <button 
                        type="button" 
                        @click.stop="removeImage(index)"
                        class="absolute -top-1.5 -right-1.5 bg-red-600 hover:bg-red-700 text-white rounded-full w-5 h-5 
                        flex items-center justify-center text-xs transition-all duration-200 shadow-md opacity-0 
                        group-hover:opacity-100 hover:scale-110"
                        title="Eliminar imagen"
                      >
                        ×
                      </button>
                    </div>
                  </div>
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
                    class="text-blue-400 px-3 py-2 bg-primary text-dark rounded-r-lg hover:bg-primary-dark focus:outline-none focus:ring-2 focus:ring-primary focus:ring-opacity-50"
                  >
                    Guardar
                  </button>
                </div>
              </div>

              <!-- Coordenadas para reportes manuales -->
              <div class="flex flex-col lg:flex-row items-start lg:items-center justify-between space-y-2 lg:space-y-0">
                <button type="button" @click="openManualReportMapModal" class="btn-primary w-full lg:w-auto">🗺 Agregar Ubicación</button>
                <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng"
                     class="text-sm text-gray-600 dark:text-gray-400 bg-green-100 dark:bg-green-900 px-3 py-2 rounded-lg">
                  <strong>Coordenadas:</strong> {{ manualReport.coordinates.lat.toFixed(6) }}, {{ manualReport.coordinates.lng.toFixed(6) }}
                </div>
              </div>

              <div class="flex flex-col lg:flex-row justify-end space-y-2 lg:space-y-0 lg:space-x-3">
                <button @click="showManualReportForm = false" class="btn-secondary w-full lg:w-auto">
                  Cancelar
                </button>
                <button @click="generateManualReport" class="btn-primary w-full lg:w-auto">
                  📝 Generar Comunicado
                </button>
              </div>
            </div>
          </div>

          <div v-if="showReport" class="flex justify-end mt-6">
            <button @click="printReport" class="bg-green-600 hover:bg-green-700 text-white px-4 sm:px-6 py-2 sm:py-3 rounded-lg font-medium transition-all duration-300 flex items-center space-x-2 w-full lg:w-auto justify-center">
              <span>🖨</span>
              <span>Imprimir PDF</span>
            </button>
          </div>
        </div>

        <!-- Vista Previa del Reporte -->
        <div v-if="showReport" id="report-content" class="print:shadow-none">
          
          <!-- Header del Reporte -->
          <div class="p-0 print:p-0">
            <div class="flex items-center justify-center text-center relative print:items-start">

              <!-- Logo -->
              <div class="absolute left-0 flex items-center justify-center 
                print:relative print:left-auto print:w-auto print:flex-shrink-0">
                <img src="./Escudo_de_Roatan.png" 
                     alt="Escudo de Roatan" 
                     class="h-10 w-auto sm:h-12 md:h-14 print:h-10 print:w-10">
              </div>

              <!-- Título del Reporte (centrado) -->
              <div class="flex-1 print:flex-grow print:text-center print:mx-0">
                <h1 class="text-sm sm:text-base md:text-xl font-bold 
                 text-gray-900 dark:text-white print:text-black print:text-lg">
                  Reporte Situacional
                </h1>
                <h2 class="text-[10px] sm:text-xs md:text-lg font-semibold 
                 text-gray-700 dark:text-gray-300 print:text-black print:text-sm">
                  {{ getReportTitle() }}
                </h2>
                <p class="text-[9px] sm:text-xs md:text-base 
                text-gray-600 dark:text-gray-400 print:text-black print:text-xs">
                  {{ getCurrentDateTime() }}
                </p>
              </div>
              
            </div>
          </div>



          <!-- Contenido del Reporte -->
          <div class="p-0 print:p-0">
           
           <!-- Reporte Manual -->
            <div v-if="reportCategory === 'manual' && selectedManualReportType">
  
            <!-- Alerta Meteorológica -->
            <div v-if="selectedManualReportType === 'weather'"> 
    
            <!-- Información del Evento -->
            <div class="mt-2 mb-2">
              <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
                INFORMACIÓN DEL EVENTO
              </h3>
       
              <!-- 📘 Sección: Datos del Evento y Nivel de Alerta -->
<div class="flex flex-row gap-2 sm:gap-4 seccion-datos">

  <!-- 🟦 Datos del Evento -->
  <div class="bg-blue-50 dark:bg-blue-900/20 print:bg-blue-50 print:dark:bg-blue-50 
              p-2 sm:p-4 md:p-6 rounded-lg 
              print:border print:border-0 print:ring-1 print:ring-gray-100 flex-1 min-w-[200px]">

    <h4 class="font-bold text-blue-900 dark:text-blue-200 print:text-black 
               mb-2 sm:mb-3 text-xs sm:text-sm md:text-base print:text-[12px]">
      📊 DATOS DEL EVENTO
    </h4>

    <div class="space-y-1 text-black dark:text-white text-xs sm:text-sm md:text-base print:text-[12px] print:space-y-2">
      <p class="print:text-black"><strong>Tipo:</strong> {{ getWeatherAlertTitle() }}</p>
      <p v-if="manualReport.weather.affectedZones?.length" class="print:text-black">
        <strong>Zona:</strong> {{ getAffectedZoneText() }}
      </p>
      <p v-if="manualReport.weather.duration" class="print:text-black">
        <strong>Duración:</strong> {{ manualReport.weather.duration }}
      </p>
      <p v-if="manualReport.weather.probability" class="print:text-black">
        <strong>Probabilidad:</strong>
        {{ manualReport.weather.probability === 'baja' ? 'Baja (≤30%)' : 
          manualReport.weather.probability === 'media' ? 'Media (31-70%)' : 
          'Alta (>70%)' }}
      </p>
      <p v-if="manualReport.weather.rainfall" class="print:text-black">
        <strong>Precipitación:</strong> {{ manualReport.weather.rainfall }} mm
      </p>
      <p v-if="manualReport.weather.intensity" class="print:text-black">
        <strong>Intensidad:</strong> {{ manualReport.weather.intensity }}
      </p>
      <p v-if="manualReport.weather.impactTime" class="print:text-black">
        <strong>Impacto:</strong> {{ manualReport.weather.impactTime }}
      </p>
    </div>
  </div>

  <!-- 🚨 Nivel de Alerta -->
  <div
    :class="[
      'p-2 sm:p-4 md:p-6 rounded-lg print:border print:border-0 print:ring-1 print:ring-gray-100 flex-1 min-w-[150px]',
      manualReport.weather.riskLevel === 'blanco' ? 'bg-white-50 dark:bg-white-900/20 print:bg-white-50 print:dark:bg-green-50 text-green-900 dark:text-green-200 print:text-black' :
      manualReport.weather.riskLevel === 'verde' ? 'bg-green-50 dark:bg-green-900/20 print:bg-green-50 print:dark:bg-green-50 text-green-900 dark:text-green-200 print:text-black' :
      manualReport.weather.riskLevel === 'amarillo' ? 'bg-yellow-50 dark:bg-yellow-900/20 print:bg-yellow-50 print:dark:bg-yellow-50 text-yellow-900 dark:text-yellow-200 print:text-black' :
      manualReport.weather.riskLevel === 'rojo' ? 'bg-red-50 dark:bg-red-900/20 print:bg-red-50 print:dark:bg-red-50 text-red-900 dark:text-red-200 print:text-black' : ''
    ]">         

    <h4 class="font-bold text-dark-900 dark:text-dark-200 print:text-black 
               mb-2 sm:mb-3 text-xs sm:text-sm md:text-base print:text-[12px]">
      🚨 NIVEL DE ALERTA
    </h4>

    <div class="text-center space-y-1 text-white text-xs sm:text-sm md:text-base print:text-[12px]">
      <div class="text-3xl sm:text-4xl md:text-5xl mb-1 sm:mb-2 print:text-[24px]">
        {{ getRiskIcon() }}
      </div>
      <p 
        :class="[
          'font-bold',
          manualReport.weather.riskLevel === 'blanco' ? 'text-green-900 dark:text-green-200 print:text-black print:text-[12px]' :
          manualReport.weather.riskLevel === 'verde' ? 'text-green-900 dark:text-green-200 print:text-black print:text-[12px]' :
          manualReport.weather.riskLevel === 'amarillo' ? 'text-yellow-900 dark:text-yellow-200 print:text-black print:text-[12px]' :
          manualReport.weather.riskLevel === 'rojo' ? 'text-red-900 dark:text-red-200 print:text-black print:text-[12px]' : ''
        ]"
      >
        {{ getRiskLevelText() }}
      </p>
    </div>
  </div>
</div>


          </div>     <!-- Ubicación -->
     <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        UBICACIÓN
      </h3>
      <div class="print-map-container h-80 md:h-96">
        <div id="report-map" style="height: 100%; border-radius: 8px;" class="print-visible-map"></div>
      </div>
     </div>

     <!-- Descripción -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        DESCRIPCIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.description }}
        </p>
      </div>
     </div>

     <!-- Recomendaciones -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        RECOMENDACIONES PARA LA POBLACIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.recommendations }}
        </p>
      </div>
     </div>
   </div>     

        <!-- Secciones de Ubicación, Descripción y Recomendaciones eliminadas de la vista general
             ya que se muestran en las secciones específicas de cada tipo de reporte -->

        <!-- Aviso de Seguridad -->
        <div v-if="selectedManualReportType === 'security'"> 
          <!-- Información del Evento -->
          <div class="mb-8">
            <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
              INFORMACIÓN DEL EVENTO
            </h3>
      
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 print:grid-cols-2 print:gap-4 seccion-datos">
              <!-- Datos del Evento -->
              <div class="bg-blue-50 dark:bg-blue-900/20 print:bg-blue-50 print:dark:bg-blue-50 p-4 sm:p-6 rounded-lg">
                <h4 class="font-bold text-blue-900 dark:text-blue-200 print:text-black mb-3">
                  📊 DATOS DEL EVENTO
                </h4>
          <div class="space-y-1 text-white text-[15px] print:text-[12px] print:space-y-2">
            <p class="print:text-black"><strong>Tipo:</strong> {{ getSecurityAlertTitle() }}</p>
            <p v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="print:text-black">
              <strong>Coordenadas:</strong> {{ manualReport.coordinates.lat.toFixed(6) }}, {{ manualReport.coordinates.lng.toFixed(6) }}
            </p>
          </div>
        </div>

        <!-- Nivel de Alerta -->
        <div class="bg-green-50 dark:bg-green-900/20 print:bg-green-50 print:dark:bg-green-50 p-4 sm:p-6 rounded-lg">
          <h4 class="font-bold text-green-900 dark:text-green-200 print:text-black mb-3">
            🚨 NIVEL DE ALERTA
          </h4>
          <div class="space-y-1 text-white text-[15px] print:text-[12px] print:space-y-2">
            <div class="text-center">
              <div class="text-4xl mb-2">🚨</div>
              <p class="print:text-black font-bold">AVISO DE SEGURIDAD</p>
            </div>
          </div>
        </div>
      </div>
    </div>

     <!-- Ubicación -->
     <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        UBICACIÓN
      </h3>
      <div class="print-map-container h-80 md:h-96">
        <div id="report-map" style="height: 100%; border-radius: 8px;" class="print-visible-map"></div>
      </div>
     </div>

     <!-- Descripción -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        DESCRIPCIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.description }}
        </p>
      </div>
     </div>

     <!-- Recomendaciones -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        RECOMENDACIONES PARA LA POBLACIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.recommendations }}
        </p>
      </div>
     </div>
   </div>

      <!-- Aviso de Tránsito -->
      <div v-if="selectedManualReportType === 'traffic'">
    
    <!-- Información del Evento -->
    <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        INFORMACIÓN DEL EVENTO
      </h3>
      
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 print:grid-cols-2 print:gap-4 seccion-datos">
        <!-- Datos del Evento -->
        <div class="bg-blue-50 dark:bg-blue-900/20 print:bg-blue-50 print:dark:bg-blue-50 p-4 sm:p-6 rounded-lg">
          <h4 class="font-bold text-blue-900 dark:text-blue-200 print:text-black mb-3">
            📊 DATOS DEL EVENTO
          </h4>
          <div class="space-y-1 text-white text-[15px] print:text-[12px] print:space-y-2">
            <p class="print:text-black"><strong>Tipo:</strong> {{ getTrafficAlertTitle() }}</p>
            <p v-if="manualReport.traffic.duration" class="print:text-black">
              <strong>Duración:</strong> {{ manualReport.traffic.duration }}
            </p>
            <p v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="print:text-black">
              <strong>Coordenadas:</strong> {{ manualReport.coordinates.lat.toFixed(6) }}, {{ manualReport.coordinates.lng.toFixed(6) }}
            </p>
          </div>
        </div>

        <!-- Nivel de Alerta -->
        <div class="bg-green-50 dark:bg-green-900/20 print:bg-green-50 print:dark:bg-green-50 p-4 sm:p-6 rounded-lg">
          <h4 class="font-bold text-green-900 dark:text-green-200 print:text-black mb-3">
            🚨 NIVEL DE ALERTA
          </h4>
          <div class="space-y-1 text-white text-[15px] print:text-[12px] print:space-y-2">
            <div class="text-center">
              <div class="text-4xl mb-2">🚦</div>
              <p class="print:text-black font-bold">AVISO DE TRÁNSITO</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Ubicación -->
     <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        UBICACIÓN
      </h3>
      <div class="print-map-container h-80 md:h-96">
        <div id="report-map" style="height: 100%; border-radius: 8px;" class="print-visible-map"></div>
      </div>
     </div>

     <!-- Descripción -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        DESCRIPCIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.description }}
        </p>
      </div>
     </div>

     <!-- Recomendaciones -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        RECOMENDACIONES PARA LA POBLACIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.recommendations }}
        </p>
      </div>
     </div>
   </div>

      <!-- Interrupción de Servicio -->
      <div v-if="selectedManualReportType === 'public_service'">
    
      <!-- Información del Evento -->
      <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2">
        INFORMACIÓN DEL EVENTO
      </h3>
      
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 print:grid-cols-2 print:gap-4 seccion-datos">
        <!-- Datos del Evento -->
        <div class="bg-blue-50 dark:bg-blue-900/20 print:bg-blue-50 print:dark:bg-blue-50 p-4 sm:p-6 rounded-lg">
          <h4 class="font-bold text-blue-900 dark:text-blue-200 print:text-black mb-3">
            📊 DATOS DEL EVENTO
          </h4>
          <div class="space-y-1 text-white text-[15px] print:text-[12px] print:space-y-2">
            <p class="print:text-black"><strong>Tipo:</strong> {{ getPublicServiceAlertTitle() }}</p>
            <p v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="print:text-black">
              <strong>Coordenadas:</strong> {{ manualReport.coordinates.lat.toFixed(6) }}, {{ manualReport.coordinates.lng.toFixed(6) }}
            </p>
          </div>
        </div>

        <!-- Nivel de Alerta -->
        <div class="bg-green-50 dark:bg-green-900/20 print:bg-green-50 print:dark:bg-green-50 p-4 sm:p-6 rounded-lg">
          <h4 class="font-bold text-green-900 dark:text-green-200 print:text-black mb-3">
            🚨 NIVEL DE ALERTA
          </h4>
          <div class="space-y-1 text-white text-[15px] print:text-[12px] print:space-y-2">
            <div class="text-center">
              <div class="text-4xl mb-2">🔧</div>
              <p class="print:text-black font-bold">INTERRUPCIÓN DE SERVICIO</p>
            </div>
          </div>
        </div>
      </div>
    </div>

   <!-- Ubicación -->
     <div v-if="manualReport.coordinates.lat && manualReport.coordinates.lng" class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        UBICACIÓN
      </h3>
      <div class="print-map-container h-80 md:h-96">
        <div id="report-map" style="height: 100%; border-radius: 8px;" class="print-visible-map"></div>
      </div>
     </div>

     <!-- Descripción -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        DESCRIPCIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.description }}
        </p>
      </div>
     </div>

     <!-- Recomendaciones -->
     <div class="mb-8">
      <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-black pb-2">
        RECOMENDACIONES PARA LA POBLACIÓN
      </h3>
      <div class="bg-gray-50 dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg">
        <p class="text-gray-700 dark:text-gray-300 print:text-black text-[15px] print:text-[12px]">
          {{ manualReport.recommendations }}
        </p>
      </div>
     </div>
   </div>
 
  </div>


      <!-- Reportes Automáticos -->
      <div v-if="reportCategory === 'automatic'">

        <!-- Reporte de Cierre de Turno -->
        <div v-if="selectedReportType === 'cierre'">
          
          <!-- Información del Turno -->
          <div class="mt-2 mb-2"> 
            <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2">
              INFORMACIÓN DEL TURNO
            </h3>

<!-- Datos generales y estadísticas -->
<div class="flex justify-center items-stretch gap-2 sm:gap-4 md:gap-6 print:gap-4 seccion-datos h-full">
  
  <!-- 🟦 Datos generales -->
  <div class="flex flex-col 
              bg-blue-50 dark:bg-blue-900/20 print:bg-blue-50 print:dark:bg-blue-50 
              p-4 sm:p-6 rounded-lg
              flex-1 text-left">
    <h4 class="font-bold text-blue-900 dark:text-blue-200 print:text-black mb-2 sm:mb-3 
               text-[10px] sm:text-sm md:text-base">
      DATOS GENERALES
    </h4>
    <div class="flex-1 flex flex-col text-white text-[8px] sm:text-xs md:text-sm print:text-[12px] space-y-1 print:space-y-2">
      <p class="print:text-black"><strong>Supervisor:</strong> {{ activeShift?.supervisor || 'No disponible' }}</p>
      <p class="print:text-black"><strong>Inicio:</strong> {{ activeShift?.startTime || 'No disponible' }}</p>
      <p class="print:text-black"><strong>Cierre:</strong> {{ getCurrentDateTime() }}</p>
      <p class="print:text-black"><strong>Total de Eventos:</strong> {{ events.length }}</p>
    </div>
  </div>

  <!-- 🟩 Estadísticas -->
  <div class="flex flex-col 
              bg-green-50 dark:bg-green-900/20 print:bg-green-50 print:dark:bg-green-50 
              p-4 sm:p-6 rounded-lg
              flex-1 text-left">
    <h4 class="font-bold text-green-900 dark:text-green-200 print:text-black mb-2 sm:mb-3 
               text-[10px] sm:text-sm md:text-base">
      📈 ESTADÍSTICAS
    </h4>
    <div class="flex-1 flex flex-col text-white text-[8px] sm:text-xs md:text-sm print:text-[12px] space-y-1 print:space-y-2">
      <p class="print:text-black"><strong>Tiempo Resp. Promedio:</strong> {{ getAverageResponseTimeFiltered() }} min</p>
      <p class="print:text-black"><strong>Recursos Desplegados:</strong> {{ getTotalResourcesDeployedFiltered() }}</p>
      <p class="print:text-black"><strong>Cámaras Activas:</strong> {{ getTotalOnlineCameras() }}/{{ getTotalCameras() }}</p>
      <p class="print:text-black"><strong>Eventos Críticos:</strong> {{ getEventsByPriorityFiltered('critica') }}</p>
    </div>
  </div>

</div>

          </div>

<!-- Resumen por Prioridad -->
                <div class="mb-8">
            <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2">
              RESUMEN POR PRIORIDAD
            </h3>

           <!-- 🔹 Contenedor flexible con scroll horizontal en móviles -->
<div class="grid grid-cols-4 gap-2 sm:gap-4 pb-2 sm:pb-4 print:gap-2">
    
  <!-- 🔴 Críticos -->
  <div class="w-full p-1 sm:p-4 bg-red-50 dark:bg-red-900/20 rounded-xl border border-red-200 
              dark:border-red-700 flex flex-col items-center justify-center
              print:bg-red-100 print:border-0 print:ring-1 print:ring-gray-100">
    <div class="text-[14px] sm:text-4xl font-bold text-red-600 dark:text-red-200 print:text-black">
      {{ getEventsByPriorityFiltered('critica') }}
    </div>
    <div class="text-[6px] sm:text-sm text-red-600 dark:text-red-200 print:text-black mt-1 sm:mt-2 font-medium">
      🔴 Críticos
    </div>
  </div>

  <!-- 🟠 Altos -->
  <div class="w-full p-1 sm:p-4 bg-orange-50 dark:bg-orange-900/20 rounded-xl border border-orange-200 
              dark:border-orange-700 flex flex-col items-center justify-center
              print:bg-orange-100 print:border-0 print:ring-1 print:ring-gray-100">
    <div class="text-[14px] sm:text-4xl font-bold text-orange-600 dark:text-orange-200 print:text-black">
      {{ getEventsByPriorityFiltered('alta') }}
    </div>
    <div class="text-[6px] sm:text-sm text-orange-600 dark:text-orange-200 print:text-black mt-1 sm:mt-2 font-medium">
      🟠 Altos
    </div>
  </div>

  <!-- 🟡 Medios -->
  <div class="w-full p-1 sm:p-4 bg-yellow-50 dark:bg-yellow-900/20 rounded-xl border border-yellow-200 
              dark:border-yellow-700 flex flex-col items-center justify-center
              print:bg-yellow-100 print:border-0 print:ring-1 print:ring-gray-100">
    <div class="text-[14px] sm:text-4xl font-bold text-yellow-600 dark:text-yellow-200 print:text-black">
      {{ getEventsByPriorityFiltered('media') }}
    </div>
    <div class="text-[6px] sm:text-sm text-yellow-600 dark:text-yellow-200 print:text-black mt-1 sm:mt-2 font-medium">
      🟡 Medios
    </div>
  </div>

  <!-- 🟢 Bajos -->
  <div class="w-full p-1 sm:p-4 bg-green-50 dark:bg-green-900/20 rounded-xl border border-green-200 
              dark:border-green-700 flex flex-col items-center justify-center
              print:bg-green-100 print:border-0 print:ring-1 print:ring-gray-100">
    <div class="text-[14px] sm:text-4xl font-bold text-green-600 dark:text-green-200 print:text-black">
      {{ getEventsByPriorityFiltered('baja') }}
    </div>
    <div class="text-[6px] sm:text-sm text-green-600 dark:text-green-200 print:text-black mt-1 sm:mt-2 font-medium">
      🟢 Bajos
    </div>
  </div>

</div>
          </div>

                <!-- Detalle de Eventos -->
                <div class="mb-8" v-if="events.length > 0">
                  <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2 print:border-b-2 print:border-gray-300 print:ring-1 print:ring-gray-100">DETALLE DE EVENTOS</h3>
                  <div class="overflow-x-auto">
                    <table class="w-full border-collapse border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 text-white print:text-black print:text-[10px]">
                      <thead class="bg-gray-100 dark:bg-gray-700 print:bg-white"> 
                        <tr>
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Código</th>
                         <!-- <th class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Hora</th> -->
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0  print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Tipo</th>
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0  print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Prioridad</th>
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0  print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Ubicación</th>
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0  print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Estado</th>
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">T.Resp</th>
                          <th class="border border-gray-300 dark:border-gray-600 print:border-0  print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-left print:text-black">Agente</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="event in events" :key="event.code" class="hover:bg-gray-50 dark:hover:bg-gray-600 print:hover:bg-white">
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 font-mono text-xs print:text-black">{{ event.code }}</td>
                       <!--  <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 print:text-black">{{ event.time }}</td> -->
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 print:text-black">{{ getEventTypeName(event.type) }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 print:text-black">{{ event.priority.toUpperCase() }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 text-xs print:text-black">{{ getShortLocationText(event.location) }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 print:text-black">{{ getStatusText(event.status) }}</td>
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 print:text-black">{{ event.actualResponseTime || 'N/A' }} min</td>
                          <td class="border border-gray-300 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 px-2 sm:px-4 py-2 print:text-black">{{ event.registeredBy }}</td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>

                <!-- Observaciones y Recomendaciones -->
                <div class="mb-8">
                  <h3 class="text-[9px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2 print:border-b-2 print:border-gray-300 print:ring-1 print:ring-gray-100">OBSERVACIONES</h3>
                  <div class="bg-yellow-50 dark:bg-yellow-900/20 print:bg-white p-4 sm:p-6 rounded-lg border border-yellow-200 dark:border-yellow-800 print:border-0 print:ring-1 print:ring-gray-100">
                    <div class="space-y-2 text-[12px]">
                      <p class="text-yellow-800 dark:text-yellow-200 print:text-black">
                        • Se registraron {{ events.length }} eventos durante el turno
                      </p>
                      <p class="text-yellow-800 dark:text-yellow-200 print:text-black">
                        • Tiempo de respuesta promedio de {{ getAverageResponseTimeFiltered() }} minutos
                      </p>
                      <p class="text-yellow-800 dark:text-yellow-200 print:text-black">
                        • {{ getTotalOnlineCameras() }} de {{ getTotalCameras() }} cámaras operativas
                      </p>
                      <p class="text-yellow-800 dark:text-yellow-200 print:text-black" v-if="getEventsByPriorityFiltered('critica') > 0">
                        • Se atendieron {{ getEventsByPriorityFiltered('critica') }} eventos de prioridad crítica
                      </p>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Reporte Distribucional -->
              <div v-if="selectedReportType === 'distribucional'">
                <div class="mt-2 mb-2"> 
                  <h3 class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2 print:border-b-2 print:border-gray-300 print:ring-1 print:ring-gray-100">DISTRIBUCIÓN TERRITORIAL</h3>
                  
                  <!-- Solo mostrar comunidades con eventos -->
                  <div v-if="getCommunitiesWithEvents().length === 0" class="text-center py-8 text-gray-500 dark:text-gray-400 print:text-black">
                    <p>No hay eventos registrados en ninguna comunidad</p>
                  </div>
                  
                  <div v-else class="space-y-6">
                    <div v-for="community in getCommunitiesWithEvents()" :key="community.id"
                         class="bg-white dark:bg-gray-700 print:bg-white p-4 sm:p-6 rounded-lg border border-gray-200 dark:border-gray-600 print:border-0 print:ring-1 print:ring-gray-100 shadow print:shadow-none">
                      <h4 class="font-bold text-lg sm:text-xl text-gray-900 dark:text-white print:text-black mb-4 flex items-center">
                        <span class="text-xl sm:text-2xl mr-2">🏘️</span>
                        {{ community.name }}
                      </h4>
                      
                      <!-- Información de la comunidad -->
                      <div class="mb-8">
                        <h3
                          class="text-[10px] sm:text-sm font-semibold mb-4 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-2 print:border-b-black"
                        >
                          📍 EVENTOS DE LA COMUNIDAD
                        </h3>

                        <!-- Lista de eventos -->
                        <div class="space-y-4 print:space-y-2">
                          <div
                            v-for="event in getEventsByCommunityDetailed(community.id)"
                            :key="event.code"
                            class="bg-gray-50 dark:bg-gray-600 print:bg-white p-3 rounded-lg border-l-4 border-l-primary print:border-l-black 
                            flex flex-col lg:flex-row justify-between items-stretch gap-3 print:gap-2 print:p-2"
                          >
                          <!-- Información del evento -->
                          <div class="lg:w-[40%] w-full flex flex-col justify-between text-sm print:text-xs print:space-y-0">
                            <div
                              class="flex flex-wrap items-center justify-between mb-2 print:mb-1 border-b border-gray-200 dark:border-gray-500 pb-1 print:pb-0.5"
                            >
                              <div class="flex flex-wrap items-center gap-2 print:gap-1">
                                <span
                                  class="font-mono text-sm bg-blue-100 dark:bg-blue-900 print:bg-gray-100 px-2 py-1 print:px-1 print:py-0 rounded print:text-white"
                                  >{{ event.code }}</span
                                >
                                <span
                                  class="px-2 py-1 print:px-1 print:py-0 text-xs font-medium rounded print:text-black"
                                  >{{ event.priority.toUpperCase() }}</span
                                >
                                <span
                                  class="text-xs text-gray-600 dark:text-gray-400 print:text-black print:text-[0.65rem]"
                                  >{{ event.time }}</span
                                >
                              </div>
                              <span
                                class="px-2 py-1 print:px-1 print:py-0 text-xs font-medium rounded print:text-black"
                                >{{ getStatusText(event.status) }}</span
                              >
                            </div>

                            <p
                              class="font-medium text-gray-900 dark:text-white print:text-black mb-1 print:mb-0.5"
                            >
                              {{ getEventTypeName(event.type) }}
                            </p>
                            <p class="text-gray-600 dark:text-gray-300 print:text-black mb-1 print:mb-0.5">
                              <strong>Ubicación:</strong>
                              {{ event.location.community ? `${event.location.community}` : '' }}{{ event.location.neighborhood ? ` - ${event.location.neighborhood}` : '' }}
                              <span v-if="event.location.reference">
                                - {{ event.location.reference }}
                              </span>
                            </p>

                            <!-- Instituciones y recursos por evento -->
                            <div v-if="getEventInstitutions(event).length > 0" class="mt-2 print:mt-1">
                              <p class="text-xs font-medium text-gray-600 dark:text-gray-400 print:text-black mb-1">
                                <strong>Instituciones y Recursos:</strong>
                              </p>
        
                              <div class="space-y-1 print:space-y-0.5">
                                <div 
                                  v-for="(institution, idx) in getEventInstitutions(event)" 
                                  :key="'inst-'+idx"
                                  class="flex flex-wrap items-center gap-1"
                                >
                                  <!-- Color dinámico según el índice -->
                                  <template v-if="idx % 4 === 0">
                                    <span 
                                      class="text-xs px-2 py-1 print:px-1 print:py-0 rounded-full bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200"
                                    >
                                      {{ institution.name }}
                                    </span>
                                  </template>
                                  <template v-else-if="idx % 4 === 1">
                                    <span 
                                      class="text-xs px-2 py-1 print:px-1 print:py-0 rounded-full bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200"
                                    >
                                      {{ institution.name }}
                                    </span>
                                  </template>
                                  <template v-else-if="idx % 4 === 2">
                                    <span 
                                      class="text-xs px-2 py-1 print:px-1 print:py-0 rounded-full bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200"
                                    >
                                      {{ institution.name }}
                                    </span>
                                  </template>
                                  <template v-else>
                                    <span 
                                      class="text-xs px-2 py-1 print:px-1 print:py-0 rounded-full bg-pink-100 text-pink-800 dark:bg-pink-900 dark:text-pink-200"
                                    >
                                      {{ institution.name }}
                                    </span>
                                  </template>
        
                                  <!-- Recursos asociados a esta institución -->
                                  <template v-for="(resource, rIdx) in getEventResources(event)">
                                    <span 
                                      v-if="resource.fullId && resource.fullId.startsWith(institution.id + '_')"
                                      :key="'res-'+idx+'-'+rIdx"
                                      :class="[
                                        'text-xs px-2 py-1 print:px-1 print:py-0 rounded-full',
                                        idx % 4 === 0 ? 'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200' :
                                        idx % 4 === 1 ? 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200' :
                                        idx % 4 === 2 ? 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200' :
                                        'bg-pink-100 text-pink-800 dark:bg-pink-900 dark:text-pink-200'
                                      ]"
                                    >
                                      {{ resource.name }}
                                      <span v-if="resource.specification" class="ml-1 print:ml-0.5 text-gray-600 dark:text-gray-300 print:text-white">
                                        ({{ resource.specification }})
                                      </span>
                                    </span>
                                  </template>
                                </div>
                              </div>
                           </div>


                        <!-- Afectaciones -->
                        <div v-if="hasImpacts(event)" class="mt-2 print:mt-1">
                          <p class="text-xs font-medium text-gray-600 dark:text-gray-400 print:text-black mb-1">
                            <strong>Afectaciones:</strong>
                          </p>
                          <p class="text-xs text-gray-700 dark:text-gray-300 print:text-black">
                            {{ getImpactsText(event) }}
                          </p>
                        </div>

                        <p
                          v-if="event.coordinates.lat && event.coordinates.lng"
                          class="text-xs text-gray-500 dark:text-gray-400 print:text-black"
                        >
                          <strong>Coordenadas:</strong>
                          {{ event.coordinates.lat.toFixed(6) }},
                          {{ event.coordinates.lng.toFixed(6) }}
                        </p>
                      </div>

                        <!-- Mapa (60%) -->
                        <div
                          v-if="event.coordinates.lat && event.coordinates.lng"
                          class="lg:w-[60%] w-full flex justify-center items-center mini-map-container" >
                          <div
                            :id="`event-mini-map-${event.code}`"
                            class="mini-map w-full h-56 sm:h-64 rounded-md border print:border print:ring-1 print:ring-gray-300">
                          </div>
                        </div> 
                      </div>
                    </div>
                  </div> 
                </div>
              </div>
            </div>
          </div>
          </div>
            
            <!-- Sección de imágenes globales (ANEXOS) -->
            <div v-if="uploadedImages && uploadedImages.length > 0" class="mb-0">
              <h3 class="text-[10px] sm:text-sm font-semibold mb-2 text-gray-900 dark:text-white print:text-black border-b-2 border-primary pb-1 print:border-b-2 print:border-gray-300 print:ring-1 print:ring-gray-100">
                ANEXOS
              </h3>

              <!-- Contenedor general para todas las imágenes -->
              <div class="overflow-y-auto pb-0 mb-12 print:mb-4">
                <div class="flex flex-wrap gap-3 justify-start items-start">
                  <div
                    v-for="(img, index) in uploadedImages"
                    :key="'img-' + index"
                    class="relative group print:break-inside-avoid"
                  >
                    <!-- Imagen cuadrada -->
                    <div class="w-24 h-24 sm:w-32 sm:h-32 lg:w-[14rem] lg:h-[14rem] rounded-md overflow-hidden flex items-center justify-center"> 
                      <img 
                        :src="img.preview" 
                        class="w-full h-full object-cover transition-transform duration-200 group-hover:scale-105 rounded-md"
                      />
                    </div>

                    <!-- Botón eliminar -->
                    <button
                      @click="removeImage(index)"
                      class="absolute top-1 right-1 bg-red-500 hover:bg-red-600 text-white rounded-full w-5 h-5 
                 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity duration-200 text-xs print:hidden"
                      title="Eliminar imagen"
                    >
                      ×
                    </button>
                  </div>
                </div>
              </div>
            </div> 

            <!-- Footer del Reporte -->
            <div 
              class="text-center text-sm text-gray-500 dark:text-gray-400 p-6 border-t border-gray-200 dark:border-gray-600 mt-8 print-footer hidden">
              <div class="max-w-3xl mx-auto">
                <div v-if="sources.length > 0" class="flex flex-col items-center space-y-2 mb-4">
                  <span class="font-medium text-gray-500 dark:text-gray-400">🌐 Fuentes:</span>
                  <div class="flex flex-wrap justify-center items-center gap-2">
                    <template v-for="(source, index) in sources" :key="source.id">
                      <a 
                        :href="source.url" 
                        target="_blank" 
                        rel="noopener noreferrer"
                        class="text-blue-600 dark:text-blue-400 hover:underline whitespace-nowrap"
                      >
                        {{ source.url }}
                      </a>
                      <span 
                        v-if="index < sources.length - 1" 
                        class="text-gray-400 dark:text-gray-500"
                      >|</span>
                    </template>
                  </div>
                </div>

                <p>Municipalidad de Roatán</p>
                <p>📞 Emergencias: *1101</p>
                <p class="mb-1"><strong>Centro de Monitoreo Joseph Solomon</strong></p>
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
      sources: [],
      showSourceInput: false,
      newSource: '',
      currentDateTime: '',
      activeTab: 'dashboard',
      activeShift: null,
      uploadedImages: [],
      closedShifts: [],
      showTurnCodeModal: false,
      showEndShiftModal: false,
      showAgentCodeModal: false,
      showEventStatusModal: false,
      showEditEventModal: false,
      showDeleteEventModal: false,
      showMapModal: false,
      showEventMapModal: false,
      showManualReportMapModal: false,
      selectedEvent: null,
      eventToDelete: null,
      turnCode: '',
      turnCodeError: '',
      agentCode: '',
      agentCodeError: '',
      editingEvent: null,
      editingEventData: null,
      editingEventStatus: '',
      editingEventResponseTime: null,
      editAvailableNeighborhoods: [],
      tempCoordinates: { lat: null, lng: null },
      tempManualReportCoordinates: { lat: null, lng: null },
      modalMap: null,
      eventMap: null,
      manualReportMap: null,
      reportMap: null,
      
      // Reporte manual con formulario expandido
      manualReport: {
        weather: {
          alertType: '',
          eventType: '',
          riskLevel: '',
          probability: '',
          impactTime: '',
          intensity: '',
          duration: '',
          affectedZones: [],
          images: []
        },
        security: {
          alertType: 'preventivo',
          affectedZone: ''
        },
        traffic: {
          alertType: '',
          duration: ''
        },
        publicService: {
          serviceType: '',
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
            'El Swampo', 'Zona Comercial',
            'Barrio El Manantial', 'Los Maestros', 'Coxen Cove',
            'Franco Flat', 'Mant Trap', 'Zompopal', 'Spanish Town',
            'Brass Hill', 'La Punta', 'Willie Warren', 'Loma Linda',
            'Spring Garden 1', 'Spring Garden 2', 'Coconout Garden',
            'New York', 'El Tiquet', 'Nicaragua', 'Palos Altos',
            'Calle 8/ El Mercado', 'La Fe'
          ],
          cameras: { total: 12, online: 11 }
        },
        {
          id: 'frenchharbour',
          name: 'French Harbour',
          population: 14000,
          neighborhoods: [
            'Los Fuertes','La Punta', 'La Loma', 'FirstBight',
            'El Bajo', 'La Rampla', 'La Bahia', 'Monte Placentero'
          ],
          cameras: { total: 10, online: 9 }
        },
        {
          id: 'sandybay',
          name: 'Sandy Bay',
          population: 8000,
          neighborhoods: [
            'Policarpo Galindo', 'Balfate', 'Bellavista', 'Cañaveral',
            'Gibson Bigh', 'Las Gemelas', 'Lawson Rock', "Antony's Key",
            'White Rock Hills', 'La Uva'
          ],
          cameras: { total: 6, online: 6 }
        },
        {
          id: 'westend',
          name: 'West End',
          population: 6000,
          neighborhoods: [
            'Zona Hotelera', 'Playa Pública',
            'Zona de Restaurantes', 'El Berinche', 'West End Village',
            'Half Moon Bay', 'Mangrove Bight'
          ],
          cameras: { total: 5, online: 5 }
        },
        {
          id: 'westbay',
          name: 'West Bay',
          population: 5000,
          neighborhoods: [
            'Resorts', 'Zona de Buceo', 'Playas Privadas', 'Área Residencial',
            'Colonia Trejo y El Barrial', 'Villas Mackay', 'Residencial Merendon Hills',
            'Infinity Bay Spa & Beach Resort', 'West Bay Village', 'Turtle Crossing',
            'The Turrets', 'Lighthouse Estates'
          ],
          cameras: { total: 4, online: 4 }
        },
        {
          id: 'flowersbay',
          name: 'Flowers Bay',
          population: 4000,
          neighborhoods: [
            'Calle Flowers Bay'
          ],
          cameras: { total: 3, online: 3 }
        },
        {
          id: 'frenchkey',
          name: 'French Key',
          population: 4000,
          neighborhoods: [
            'French Key', 'Little French Key'
          ],
          cameras: { total: 3, online: 3 }
        },
        {
          id: 'dixoncove',
          name: 'Dixon Cove',
          population: 4000,
          neighborhoods: [
            'Dixon Cove', 'Isla Bonita', 'El Nance', 'Mahogany', 'Santa Maria', 'Dulce Maria'
          ],
          cameras: { total: 3, online: 3 }
        },
        {
          id: 'brickbay',
          name: 'Brick Bay',
          population: 4000,
          neighborhoods: [
            'Curacion', 'Brick Bay'
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
            { id: 'agentes_tacticos', name: 'Agentes Tácticos', available: 15 }
          ]
        },
        {
          id: 'bomberos',
          name: 'Bomberos',
          icon: '🚒',
          responseTime: 8,
          resources: [
            { id: 'contraincedios', name: 'Contraincendio', available: 3 },
            { id: 'ambulancia', name: 'Ambulancias', available: 2 },
            { id: 'pick_up', name: 'Pick-up', available: 3 },
            { id: 'lanchas', name: 'Lancha', available: 1 },
            { id: 'jetski', name: 'Jetski', available: 1 }
          ]
        },
        {
          id: 'copeco',
          name: 'Copeco',
          icon: '🚑',
          responseTime: 10,
          resources: [
            { id: 'ambulancias_copeco', name: 'Ambulancias', available: 3 },
            { id: 'retro_copeco', name: 'Retroexcavadoras', available: 1 },
            { id: 'volqueta_copeco', name: 'Volqueta', available: 1 },
            { id: 'pick_up_copeco', name: 'Pick-up', available: 3 },
            { id: 'rino_copeco', name: 'Rinos', available: 2 },
            { id: 'cuatri_copeco', name: 'Cuatri-Moto', available: 2 },
            { id: 'lanchas_copeco', name: 'Lanchas', available: 2 }
          ]
        },
        {
          id: 'cruz_roja',
          name: 'Cruz Roja',
          icon: '🏥',
          responseTime: 12,
          resources: [
            { id: 'ambulancias_cruz_roja', name: 'Ambulancias', available: 2 },
            { id: 'pick_up', name: 'Pick-up', available: 1 },
          ]
        }
      ],
      
      eventTypes: [
  { id: 'incendio', name: 'Incendio', code: 'INC', priority: 'alta' },
  { id: 'transito', name: 'Accidente de Tránsito', code: 'TRA', priority: 'alta' },
  { id: 'emergencia_medica', name: 'Emergencia Médica', code: 'MED', priority: 'alta' },
  { id: 'ahogamiento', name: 'Emergencia Marítima', code: 'MAR', priority: 'alta' },
  { id: 'robo', name: 'Robo/Hurto', code: 'ROB', priority: 'alta' },
  { id: 'agresion', name: 'Agresión/Riña', code: 'AGR', priority: 'alta' },
  { id: 'violencia', name: 'Violencia Intrafamiliar', code: 'VIF', priority: 'alta' },
  { id: 'agresion_sexual', name: 'Agresión Sexual', code: 'ASX', priority: 'alta' },
  { id: 'explosion', name: 'Explosión', code: 'EXP', priority: 'alta' },
  { id: 'colapso', name: 'Deslizamiento/Colapso', code: 'DES', priority: 'alta' },
  { id: 'persona_crisis', name: 'Persona en Crisis', code: 'CRI', priority: 'media' },
  { id: 'extraviado', name: 'Persona Extraviada', code: 'EXT', priority: 'media' },
  { id: 'vandalismo', name: 'Vandalismo', code: 'VAN', priority: 'media' },
  { id: 'animal_peligroso', name: 'Animal Peligroso', code: 'ANI', priority: 'media' },
  { id: 'sospechoso', name: 'Actividad Sospechosa', code: 'SOS', priority: 'baja' },
  { id: 'llamada_falsa', name: 'Llamada Falsa/Broma', code: 'FAKE', priority: 'baja' }
],
      
      impactCategories: [
        { id: 'personas', label: 'Personas Afectadas', icon: '👥', bgClass: 'bg-red-50 dark:bg-red-900/20', impacts: [
          { id: 'heridos', label: 'Heridos', placeholder: 'Ej: 3 heridos - 2 graves con fracturas, 1 leve con contusiones' },
          { id: 'fallecidos', label: 'Fallecidos', placeholder: 'Ej: 1 fallecido masculino aprox. 40 años' },
          { id: 'personas_atrapadas', label: 'Personas Atrapadas', placeholder: 'Ej: 2 personas atrapadas en vehículo, conscientes' },
          { id: 'menores_involucrados', label: 'Menores Involucrados', placeholder: 'Ej: 1 menor de 8 años, ileso pero en shock' },
          { id: 'evacuados', label: 'Evacuados', placeholder: 'Ej: 15 personas evacuadas del edificio adyacente' }
        ]},
        { id: 'vehiculos', label: 'Vehículos y Transporte', icon: '🚗', bgClass: 'bg-blue-50 dark:bg-blue-900/20', impacts: [
          { id: 'vehiculos_involucrados', label: 'Vehículos Involucrados', placeholder: 'Ej: 2 autos sedan, 1 pickup - colisión frontal' },
          { id: 'embarcaciones', label: 'Embarcaciones Afectadas', placeholder: 'Ej: Lancha de 20 pies a la deriva, 3 ocupantes' }
        ]},
        { id: 'infraestructura', label: 'Infraestructura', icon: '🏢', bgClass: 'bg-orange-50 dark:bg-orange-900/20', impacts: [
          { id: 'vias_bloqueadas', label: 'Vías Bloqueadas', placeholder: 'Ej: Calle Principal cerrada entre 5ta y 7ma Av - ambos sentidos' },
          { id: 'edificios_danados', label: 'Estructuras Dañadas', placeholder: 'Ej: Fachada de edificio comercial colapsada parcialmente' },
          { id: 'servicios_interrumpidos', label: 'Servicios Interrumpidos', placeholder: 'Ej: Energía eléctrica cortada en 3 cuadras, poste caído' }
        ]},
        { id: 'riesgos', label: 'Riesgos Activos', icon: '⚠️', bgClass: 'bg-red-50 dark:bg-red-900/20', impacts: [
          { id: 'incendio_activo', label: 'Incendio Activo', placeholder: 'Ej: Fuego en 2do piso, propagándose a techos vecinos' },
          { id: 'materiales_peligrosos', label: 'Materiales Peligrosos', placeholder: 'Ej: Derrame de combustible aprox. 50 litros en calzada' },
          { id: 'armas_involucradas', label: 'Armas Involucradas', placeholder: 'Ej: Arma de fuego calibre desconocido asegurada en escena' },
          { id: 'riesgo_inminente', label: 'Riesgos Adicionales', placeholder: 'Ej: Cables eléctricos caídos energizados, riesgo de electrocución' }
        ]},
        { id: 'otros', label: 'Otros Impactos', icon: '🌊', bgClass: 'bg-gray-50 dark:bg-gray-700', impacts: [
          { id: 'area_afectada', label: 'Área/Zona Afectada', placeholder: 'Ej: Área de 200m² aproximadamente, radio de 2 cuadras' },
          { id: 'impacto_ambiental', label: 'Impacto Ambiental', placeholder: 'Ej: Contaminación de playa, mancha de aceite 50m²' },
          { id: 'otros_impactos', label: 'Otros Impactos', placeholder: 'Ej: Información adicional relevante' }
        ]}
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
        impactDetails: {},
        status: 'en_progreso'
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
      },
      
      // Control de categorías expandidas
      expandedCategories: {},
      
      // Control de categorías expandidas en modo edición
      editExpandedCategories: {},
      
      // Variables para edición de eventos
      editingEvent: null,
      editingEventData: null,
      editAvailableNeighborhoods: [],
      showEditEventModal: false,
    }
  },
  
  computed: {
    currentShiftEvents() {
      if (!this.activeShift) return []
      return this.events.filter(event => event.shiftSupervisor === this.activeShift.supervisor)
    },
    
    allImpacts() {
      const impacts = []
      this.impactCategories.forEach(category => {
        category.impacts.forEach(impact => {
          impacts.push(impact)
        })
      })
      return impacts
    }
  },
  
  mounted() {
    this.updateDateTime()
    setInterval(this.updateDateTime, 1000)
    this.resetDashboardDates()
    this.updateDashboard() 
    this.initDarkMode()
    this.loadChartsLibraries()
  },
  
  methods: {
    // Funciones para manejar categorías de impactos
    toggleCategory(categoryId) {
      this.expandedCategories[categoryId] = !this.expandedCategories[categoryId]
    },
    
    getCategoryActiveCount(category) {
      return category.impacts.filter(impact => this.newEvent.impacts[impact.id]).length
    },
    
    isCategorySelected(category) {
      return category.impacts.some(impact => this.newEvent.impacts[impact.id])
    },
    
    toggleCategoryImpacts(category) {
      const isSelected = this.isCategorySelected(category)
      
      if (isSelected) {
        // Deseleccionar todos los impactos de esta categoría
        category.impacts.forEach(impact => {
          this.newEvent.impacts[impact.id] = false
          this.newEvent.impactDetails[impact.id] = ''
        })
        // Colapsar la categoría automáticamente
        this.expandedCategories[category.id] = false
      } else {
        // Expandir la categoría para que el usuario seleccione impactos
        this.expandedCategories[category.id] = true
      }
    },
    
    // Funciones para manejar categorías de impactos en modo edición
    toggleEditCategory(categoryId) {
      this.editExpandedCategories[categoryId] = !this.editExpandedCategories[categoryId]
    },
    
    getEditCategoryActiveCount(category) {
      if (!this.editingEventData || !this.editingEventData.impacts) return 0
      return category.impacts.filter(impact => this.editingEventData.impacts[impact.id]).length
    },
    
    isEditCategorySelected(category) {
      if (!this.editingEventData || !this.editingEventData.impacts) return false
      return category.impacts.some(impact => this.editingEventData.impacts[impact.id])
    },
    
    toggleEditCategoryImpacts(category) {
      const isSelected = this.isEditCategorySelected(category)
      
      if (isSelected) {
        // Deseleccionar todos los impactos de esta categoría
        category.impacts.forEach(impact => {
          this.editingEventData.impacts[impact.id] = false
          this.editingEventData.impactDetails[impact.id] = ''
        })
        // Colapsar la categoría automáticamente
        this.editExpandedCategories[category.id] = false
      } else {
        // Expandir la categoría para que el usuario seleccione impactos
        this.editExpandedCategories[category.id] = true
      }
    },
    
    // Edit and delete event functions
    editEvent(event) {
      this.editingEvent = event
      this.editingEventData = {
        type: event.type,
        code: event.code,
        priority: event.priority,
        location: {
          community: event.location.community,
          neighborhood: event.location.neighborhood,
          reference: event.location.reference
        },
        coordinates: { ...event.coordinates },
        // Inicializar afectaciones
        impacts: { ...event.impacts } || {},
        impactDetails: { ...event.impactDetails } || {},
        // Inicializar instituciones y recursos
        institutionsUsed: { ...event.institutionsUsed } || {},
        resourcesUsed: { ...event.resourcesUsed } || {},
        resourceSpecifications: { ...event.resourceSpecifications } || {},
        responseTimeEstimated: { ...event.responseTimeEstimated } || {}
      }
      
      // Inicializar categorías expandidas para edición
      this.editExpandedCategories = {}
      this.impactCategories.forEach(category => {
        // Expandir categorías que tienen impactos seleccionados
        if (category.impacts.some(impact => this.editingEventData.impacts[impact.id])) {
          this.editExpandedCategories[category.id] = true
        }
      })
      
      this.onEditCommunityChange()
      this.showEditEventModal = true
    },

    onEditCommunityChange() {
      const community = this.communities.find(c => c.id === this.editingEventData.location.community)
      this.editAvailableNeighborhoods = community ? community.neighborhoods : []
      if (!this.editAvailableNeighborhoods.includes(this.editingEventData.location.neighborhood)) {
        this.editingEventData.location.neighborhood = ''
      }
    },

    updateEvent() {
      if (this.editingEvent && this.editingEventData) {
        // Update the event with new data
        this.editingEvent.type = this.editingEventData.type
        this.editingEvent.priority = this.editingEventData.priority
        this.editingEvent.location = { ...this.editingEventData.location }
        this.editingEvent.coordinates = { ...this.editingEventData.coordinates }
        
        // Actualizar afectaciones
        this.editingEvent.impacts = { ...this.editingEventData.impacts }
        this.editingEvent.impactDetails = { ...this.editingEventData.impactDetails }
        
        // Actualizar instituciones y recursos
        this.editingEvent.institutionsUsed = { ...this.editingEventData.institutionsUsed }
        this.editingEvent.resourcesUsed = { ...this.editingEventData.resourcesUsed }
        this.editingEvent.resourceSpecifications = { ...this.editingEventData.resourceSpecifications }
        this.editingEvent.responseTimeEstimated = { ...this.editingEventData.responseTimeEstimated }
        
        this.showEditEventModal = false
        this.editingEvent = null
        this.editingEventData = null
        this.editAvailableNeighborhoods = []
        this.editExpandedCategories = {}
        this.updateDashboard()
      }
    },

    confirmDeleteEvent(event) {
      this.eventToDelete = event
      this.showDeleteEventModal = true
    },

    deleteEvent() {
      if (this.eventToDelete) {
        const index = this.events.findIndex(e => e.code === this.eventToDelete.code)
        if (index !== -1) {
          this.events.splice(index, 1)
        }
        this.showDeleteEventModal = false
        this.eventToDelete = null
        this.updateDashboard()
      }
    },

    // Función de modal personalizado (reemplaza alert, confirm, prompt)
    showCustomModal(message, type = 'alert', onConfirm = null) {
      const modal = document.createElement('div')
      modal.className = 'fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50'
      
      const buttons = type === 'confirm' 
        ? `
          <button class="px-4 py-2 text-gray-600 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-700 rounded mr-2" onclick="this.closest('.fixed').remove()">Cancelar</button>
          <button class="px-4 py-2 bg-red-500 text-white hover:bg-red-600 rounded" onclick="this.closest('.fixed').remove(); if(window.customModalConfirm) window.customModalConfirm()">Confirmar</button>
        `
        : `<button class="px-4 py-2 bg-blue-500 text-white hover:bg-blue-600 rounded" onclick="this.closest('.fixed').remove()">Aceptar</button>`
      
      modal.innerHTML = `
        <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg max-w-sm w-full mx-4">
          <p class="text-gray-700 dark:text-gray-300 mb-4">${message}</p>
          <div class="flex justify-end">
            ${buttons}
          </div>
        </div>
      `
      
      if (onConfirm) {
        window.customModalConfirm = onConfirm
      }
      
      document.body.appendChild(modal)
    },
    
    // Verifica si hay al menos una institución con eventos
    hasInstitutionsWithEvents() {
      return this.institutions.some(institution => this.getEventsByInstitution(institution.id) > 0);
    },
    
    // Verifica si un evento tiene impactos
    hasImpacts(event) {
      return event.affectedAreas && event.affectedAreas.length > 0;
    },
    
    // Obtiene el texto de los impactos formateado
    getImpactsText(event) {
      if (!this.hasImpacts(event)) return '';
      return event.affectedAreas
        .map(area => {
          let text = area.area || '';
          if (area.details) {
            text += ` (${area.details})`;
          }
          return text;
        })
        .join(', ');
    },
    
    // Obtiene las instituciones del evento
    getEventInstitutions(event) {
      if (!event.institutionsUsed) return [];
      return Object.entries(event.institutionsUsed)
        .filter(([id, used]) => used)
        .map(([id]) => this.institutions.find(i => i.id === id))
        .filter(Boolean);
    },
    
    // Obtiene los recursos utilizados en un evento
    getEventResources(event) {
      if (!event.resourcesUsed) return [];
      
      return Object.entries(event.resourcesUsed)
        .filter(([id, used]) => used)
        .map(([id]) => {
          // Buscar el recurso en todas las instituciones
          for (const institution of this.institutions) {
            const resource = institution.resources.find(r => `${institution.id}_${r.id}` === id);
            if (resource) {
              return {
                ...resource,
                fullId: id, // Incluir el ID completo (institución_idrecurso)
                specification: event.resourceSpecifications?.[id] || ''
              };
            }
          }
          return null;
        })
        .filter(Boolean);
    },
    
    // Obtiene los recursos de una institución
    getInstitutionResources(institution) {
      return institution.resources;
    },
    
    handleImageUpload(event) {
  const files = event.target.files;
  if (!files || files.length === 0) return;

  // Si no existe el array global, inicialízalo
  if (!this.uploadedImages) {
    this.uploadedImages = [];
  }

  Array.from(files).forEach(file => {
    // Verificar si el archivo es una imagen
    if (!file.type.match('image.*')) {
      this.showCustomModal('Por favor, seleccione solo archivos de imagen.');
      return;
    }

    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        // Crear un canvas para redimensionar la imagen
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');

        const maxSize = 800;
        let width = img.width;
        let height = img.height;

        // Redimensionar manteniendo proporciones
        if (width > height) {
          if (width > maxSize) {
            height *= maxSize / width;
            width = maxSize;
          }
        } else {
          if (height > maxSize) {
            width *= maxSize / height;
            height = maxSize;
          }
        }

        canvas.width = width;
        canvas.height = height;
        ctx.drawImage(img, 0, 0, width, height);

        // Convertir a base64 (imagen redimensionada)
        const resizedImage = canvas.toDataURL('image/jpeg', 0.8);

        // Agregar imagen al array global
        this.uploadedImages.push({
          file: file,
          preview: resizedImage,
          original: e.target.result
        });
      };
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  });

  // Limpiar input
  event.target.value = '';
},

removeImage(index) {
  if (this.uploadedImages && index >= 0 && index < this.uploadedImages.length) {
    this.uploadedImages.splice(index, 1);
  }
},

removeSource(index) {
  this.sources.splice(index, 1);
},
    
    addSource() {
      this.showSourceInput = true;
      this.$nextTick(() => {
        this.$refs.sourceInput?.focus();
      });
    },
    
    saveSource() {
      if (this.newSource.trim()) {
        // Basic URL validation
        try {
          // Ensure the URL has a protocol
          let url = this.newSource.trim();
          if (!url.match(/^https?:\/\//)) {
            url = 'https://' + url;
          }
          
          // Validate URL
          new URL(url);
          
          this.sources.push({
            id: Date.now(),
            url: url
          });
          this.newSource = '';
          this.showSourceInput = false;
        } catch (e) {
          console.error('URL inválida');
        }
      }
    },
    
    loadChartsLibraries() {
      if (typeof Chart === 'undefined') {
        const script = document.createElement('script')
        script.src = 'https://cdn.jsdelivr.net/npm/chart.js'
        script.onload = () => {
          const plugin = document.createElement('script')
          plugin.src = 'https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2'
          plugin.onload = () => {
            if (typeof ChartDataLabels !== 'undefined') {
              Chart.register(ChartDataLabels)
            }
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
      if (tabId === 'dashboard') {
        this.$nextTick(() => {
          this.updateDashboard()
        })
      }
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
    
    // Shift Management
    startTurn() {
      if (this.turnCode.trim() === '') {
        this.turnCodeError = 'Debe ingresar un código válido'
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
      this.activeTab = 'events'
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
        shiftSupervisor: this.activeShift?.supervisor,
        status: 'en_progreso'
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
    
    editEventStatus(event) {
      this.editingEvent = event
      this.editingEventStatus = event.status || 'en_progreso'
      this.editingEventResponseTime = event.actualResponseTime || 0
      this.showEventStatusModal = true
    },
    
    updateEventStatus() {
      if (this.editingEvent) {
        this.editingEvent.status = this.editingEventStatus
        if (this.editingEventStatus === 'resuelto' && this.editingEventResponseTime) {
          this.editingEvent.actualResponseTime = this.editingEventResponseTime
        }
        this.showEventStatusModal = false
        this.editingEvent = null
        this.editingEventStatus = ''
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
        impactDetails: {},
        status: 'en_progreso'
      }
      this.availableNeighborhoods = []
    },

    getStatusText(status) {
      const statusTexts = {
        'en_progreso': 'En Progreso',
        'resuelto': 'Resuelto',
        'cancelado': 'Cancelado'
      }
      return statusTexts[status] || 'En Progreso'
    },

    getStatusBadgeClass(status) {
      const classes = {
        'en_progreso': 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200',
        'resuelto': 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200',
        'cancelado': 'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200'
      }
      return classes[status] || classes['en_progreso']
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
      
      this.modalMap = L.map('modal-map', {
        zoomControl: false,
        scrollWheelZoom: true,
        doubleClickZoom: false,
        boxZoom: false,
        keyboard: false,
        touchZoom: true
      }).setView([16.3291, -86.5392], 12)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors',
        zoomControl: true
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
      
      this.manualReportMap = L.map('manual-report-map', {
        zoomControl: true,  // Mostrar el control de zoom
        attributionControl: true // Desactivar el control de atribución
      }).setView([16.3291, -86.5392], 12)
      
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
      
      this.$nextTick(() => {
        setTimeout(() => {
          if (this.reportMap) {
            this.reportMap.remove()
          }
          
          const lat = this.manualReport.coordinates.lat
          const lng = this.manualReport.coordinates.lng
          
          this.reportMap = L.map('report-map', {
            zoomControl: false,  // Desactivar controles de zoom
            attributionControl: false // Desactivar control de atribución
          }).setView([lat, lng], 13)
          
          L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
          }).addTo(this.reportMap)
          
          const marker = L.marker([lat, lng]).addTo(this.reportMap)
          marker.bindPopup(`
            <strong>Ubicación del Evento</strong><br>
            Coordenadas: ${lat.toFixed(6)}, ${lng.toFixed(6)}
          `).openPopup()
          
          // Forzar actualización del mapa
          setTimeout(() => {
            this.reportMap.invalidateSize()
          }, 100)
        }, 200)
      })
    },

    initEventMiniMaps() {
      // Cargar Leaflet primero si no está disponible
      if (typeof L === 'undefined') {
        this.loadLeaflet().then(() => {
          this.createEventMiniMaps()
        })
        return
      }
      this.createEventMiniMaps()
    },

    createEventMiniMaps() {
      this.$nextTick(() => {
        this.getCommunitiesWithEvents().forEach(community => {
          const communityEvents = this.getEventsByCommunityDetailed(community.id)
          communityEvents.forEach(event => {
            if (event.coordinates.lat && event.coordinates.lng) {
              setTimeout(() => {
                this.createEventMiniMap(event)
              }, 200)
            }
          })
        })
      })
    },

    createEventMiniMap(event) {
      const mapId = `event-mini-map-${event.code}`
      const mapElement = document.getElementById(mapId)
      if (!mapElement || typeof L === 'undefined') return

      try {
        const miniMap = L.map(mapId).setView([event.coordinates.lat, event.coordinates.lng], 15)
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
          attribution: '© OpenStreetMap contributors'
        }).addTo(miniMap)
        
        const marker = L.marker([event.coordinates.lat, event.coordinates.lng]).addTo(miniMap)
        marker.bindPopup(`<strong>${event.code}</strong><br>${this.getEventTypeName(event.type)}`)
      } catch (error) {
        console.log('Error creating mini map:', error)
      }
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
            const impactType = this.allImpacts.find(i => i.id === key)
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
      this.dashboardStartDate = ''
      this.dashboardEndDate = ''
      this.updateDashboard()
    },
    
    updateDashboard() {
      this.$nextTick(() => {
        this.createCharts()
      })
    },
    
    getFilteredEvents() {
      let allEvents = [...this.allEvents]
      if (this.activeShift) {
        allEvents = [...allEvents, ...this.events]
      }
      
      if (!this.dashboardStartDate || !this.dashboardEndDate) {
        return allEvents
      }
      
      return allEvents.filter(event => {
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
      const eventsWithResponseTime = filteredEvents.filter(event => 
        event.actualResponseTime && 
        event.actualResponseTime > 0 && 
        event.status === 'resuelto'
      )
      
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
        const dateTimeA = new Date(`${a.date} ${a.time}`)
        const dateTimeB = new Date(`${b.date} ${b.time}`)
        return dateTimeB - dateTimeA
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
        if (this.selectedReportType === 'distribucional') {
          setTimeout(() => {
            this.initEventMiniMaps()
          }, 500)
        }
        document.getElementById('report-content').scrollIntoView({ behavior: 'smooth' })
      })
    },

    generateManualReport() {
      // Validar campos requeridos
      if (!this.manualReport.description || !this.manualReport.recommendations) {
        this.showCustomModal('Por favor complete todos los campos obligatorios')
        return
      }
      
      this.showReport = true
      this.reportCategory = 'manual'
      
      // Cerrar el formulario de reporte manual
      this.showManualReportForm = false
      
      // Inicializar el mapa después de que se muestre el reporte
      this.$nextTick(() => {
        // Esperar a que el DOM se actualice completamente
        setTimeout(() => {
          if (this.manualReport.coordinates.lat && this.manualReport.coordinates.lng) {
            this.initReportMap()
          }
          // Desplazarse a la sección del reporte
          document.getElementById('report-content').scrollIntoView({ behavior: 'smooth' })
        }, 300) // Aumentamos el tiempo de espera para asegurar que el DOM esté listo
      })
    },

    getWeatherAlertTitle() {
      const titles = {
        lluvia: 'Alerta por Lluvia',
        tormenta: 'Alerta por Tormenta Eléctrica',
        calor: 'Alerta por Altas Temperaturas',
        frio: 'Alerta por Bajas Temperaturas',
        neblina: 'Alerta por Neblina Densa',
        inundacion: 'Alerta por Inundaciones',
        huracan: 'Alerta por Huracán/Ciclón',
        mar: 'Alerta por Condiciones Marinas',
        sismo: 'Alerta por Sismo/Terremoto',
        tsunami: 'Alerta por Tsunami',
        contaminacion: 'Alerta por Contaminación',
        sargazo: 'Alerta por Afluencia de Sargazo',
        epidemia: 'Alerta por Emergencia Sanitaria'
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

    getTrafficAlertTitle() {
      const titles = {
        cierre_vial: 'Cierre Vial',
        desvio: 'Desvío de Ruta',
        obras: 'Obras en Progreso',
        accidente: 'Accidente de Tránsito'
      }
      return titles[this.manualReport.traffic.alertType] || 'Aviso de Tránsito'
    },

    getPublicServiceAlertTitle() {
      const titles = {
        energia: 'Interrupción del Servicio Eléctrico',
        agua: 'Interrupción del Servicio de Agua',
        internet: 'Interrupción de Internet/Telecomunicaciones',
        gas: 'Interrupción del Servicio de Gas',
        transporte: 'Interrupción del Transporte Público'
      }
      return titles[this.manualReport.publicService.serviceType] || 'Interrupción de Servicio'
    },

    getRiskLevelText() {
      const levels = {
        blanco: 'SIN ALERTA',
        verde: 'ALERTA BAJA',
        amarillo: 'ALERTA MEDIA', 
        rojo: 'ALERTA ALTA'
      }
      return levels[this.manualReport.weather.riskLevel] || 'ALERTA MEDIA'
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
      if (!this.manualReport.weather.affectedZones || this.manualReport.weather.affectedZones.length === 0) {
        return 'Sin zona específica'
      }
      
      if (this.manualReport.weather.affectedZones.includes('municipio')) {
        return 'Todo el municipio'
      }
      
      const affectedCommunities = this.manualReport.weather.affectedZones
        .filter(zoneId => zoneId !== 'municipio')
        .map(zoneId => {
          const community = this.communities.find(c => c.id === zoneId)
          return community ? community.name : zoneId
        })
        .join(', ')
      
      return affectedCommunities || 'Zonas específicas'
    },

    getEventsByInstitution(institutionId) {
      return this.events.filter(event => event.institutionsUsed && event.institutionsUsed[institutionId]).length
    },

    getInstitutionResourceCount(event, institutionId) {
      if (!event || !event.resourcesUsed) return 0;
      
      // Contar recursos que pertenecen a esta institución
      return Object.keys(event.resourcesUsed)
        .filter(resourceId => resourceId.startsWith(institutionId + '_') && event.resourcesUsed[resourceId])
        .length;
    },

    getAvgResponseTimeByInstitution(institutionId) {
      const instEvents = this.events.filter(event => 
        event.institutionsUsed && 
        event.institutionsUsed[institutionId] && 
        event.actualResponseTime &&
        event.status === 'resuelto'
      )
      
      if (instEvents.length === 0) return 'N/A'
      
      const total = instEvents.reduce((sum, event) => sum + event.actualResponseTime, 0)
      return Math.round(total / instEvents.length)
    },

    getEventsByCommunity(communityId) {
      return this.events.filter(event => event.location.community === communityId).length
    },

    getCommunitiesWithEvents() {
      return this.communities.filter(community => 
        this.events.some(event => event.location.community === community.id)
      )
    },

    getEventsByCommunityDetailed(communityId) {
      return this.events.filter(event => event.location.community === communityId)
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
            borderWidth: 1,
            datalabels: {
              color: 'white',
              anchor: 'center',
              align: 'center',
              font: {
                weight: 'bold',
                size: 12
              },
              formatter: (value) => value > 0 ? value : ''
            }
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
    
    // Cambio 4: Mostrar header de dashboard solo al imprimir dashboard
    printDashboard() {
      const dashboardHeader = document.querySelector('.dashboard-print-header')
      if (dashboardHeader) {
        dashboardHeader.style.display = 'block'
      }
      window.print()
      if (dashboardHeader) {
        dashboardHeader.style.display = 'none'
      }
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');

:root {
    --primary: #5D5CDE;
}

@media print {
  @media print {
  /* Evitar que el contenedor se corte o tenga scroll */
  .overflow-y-auto {
    overflow: visible !important;
    max-height: none !important;
  }

  /* Mantener el layout tipo cuadrícula */
  .flex-wrap {
    display: flex !important;
    flex-wrap: wrap !important;
    gap: 0.75rem !important; /* igual a gap-3 */
    justify-content: flex-start !important;
    align-items: flex-start !important;
  }

  /* Evitar que los elementos se corten entre páginas */
  .flex-wrap > div {
    break-inside: avoid !important;
    page-break-inside: avoid !important;
  }

  /* Mantener el mismo tamaño cuadrado */
  .w-24,
  .h-24,
  .sm\:w-32,
  .sm\:h-32,
  .print\:w-24,
  .print\:h-24 {
    width: 14rem !important;   
    height: 14rem !important;  
  }

  /* Forzar proporción cuadrada y buena calidad */
  .flex-wrap img {
    width: 100% !important;
    height: 100% !important;
    object-fit: cover !important;
  }

  /* Ocultar el botón eliminar */
  button[title="Eliminar imagen"] {
    display: none !important;
  }

  /* Ajustar márgenes */
  .overflow-y-auto.pb-0.mb-0 {
    padding-bottom: 0 !important;
    margin-bottom: 0 !important;
  }
}

  .mini-map-container {
    display: block !important;   /* evita el comportamiento del flex */
    height: 12rem !important;    /* igual a h-64 → 256px */
    overflow: hidden;
  }

  .mini-map {
    height: 100% !important;
    aspect-ratio: auto !important;
  }
  .seccion-datos > div {
    padding-left: 3mm !important; 
    padding-top: 4mm !important;
    padding-bottom: 4mm !important;
  }
  
  /* Asegurar que la estructura de eventos mantenga el diseño en modo impresión */
  .space-y-4 > div {
    page-break-inside: avoid;
  }
  
  /* Mantener el diseño de fila para eventos en impresión */
  .flex-col.lg\:flex-row {
    display: flex !important;
    flex-direction: row !important;
  }
  
  /* Mantener los anchos correctos para la información y el mapa */
  .lg\:w-\[40\%\] {
    width: 40% !important;
  }
  
  .lg\:w-\[60\%\] {
    width: 60% !important;
  }
  
  /* Reducir espaciado en la información del evento */
  .print\:space-y-0 > * + * {
    margin-top: 0 !important;
  }
  
  /* Ajustar tamaños de texto para impresión */
  .print\:text-xs {
    font-size: 0.7rem !important;
    line-height: 1.1 !important;
  }
  
  /* Reducir espaciado vertical entre elementos */
  .space-y-4 > div {
    margin-bottom: 0.5rem !important;
  }
  
  /* Reducir espaciado en la vista de impresión */
  @media print {
    .space-y-4 {
      margin-top: 0.25rem !important;
    }
    
    .space-y-4 > div {
      margin-bottom: 0.25rem !important;
    }
    
    .px-2.py-1 {
      padding: 1px 2px !important;
    }
    
    .rounded-full {
      padding: 0px 2px !important;
      font-size: 0.65rem !important;
    }
    
    .text-xs {
      font-size: 0.65rem !important;
      line-height: 1 !important;
    }
    
    .mb-1 {
      margin-bottom: 0.1rem !important;
    }
  }
  
  /* Reducir padding en elementos internos */
  .px-2.py-1.print\:text-black {
    padding: 1px 2px !important;
  }
  
  /* Asegurar que los mapas de eventos sean visibles */
  [id^="event-mini-map-"] {
    display: block !important;
    visibility: visible !important;
    height: 108px !important; /* Reducido en un 40% de 180px */
    width: 100% !important;
    border: 1px solid #000 !important;
  }
  
  
  /* Forzar a los navegadores a imprimir los fondos */
  * {
    -webkit-print-color-adjust: exact !important;   /* Chrome, Safari */
    color-adjust: exact !important;                 /* Firefox */
    print-color-adjust: exact !important;           /* Standard */
  }
  
  /* Eliminar encabezados y pies de página del navegador y evitar contenido en esquinas */
  /* Configuración de página - CRÍTICO PARA FOOTERS */
  @page {
    size: A4;
    margin: 1cm 1cm 1cm 1cm; /* Margen inferior de 4cm para el footer fijo */
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
    marks: none;
  }
  
  /* Eliminar headers y footers automáticos */
  @page :first { 
    margin: 1cm 1cm 1cm 1cm;
  }
  @page :left { 
    margin: 1cm 1cm 1cm 1cm;
  }
  @page :right { 
    margin: 1cm 1cm 1cm 1cm;
  }
  
  /* Estilos para el footer de impresión */
  .dashboard-print-footer {
    display: block !important;
    position: fixed !important;
    bottom: 0 !important;
    left: 0 !important;
    right: 0 !important;
    width: 100% !important;
    padding: 10px 1.5cm !important;
    border-top: 2px solid #3b82f6 !important;
    background-color: #eff6ff !important;
    z-index: 9999 !important;
    page-break-inside: avoid !important;
    text-align: center !important;
    font-size: 0.9em !important;
    color: #1e40af !important;
  }
  
  body, html { 
    -webkit-print-color-adjust: exact; 
    print-color-adjust: exact;
    margin: 0 !important;
    padding: 0 !important;
    background: white !important;
    width: 100% !important;
    height: 100% !important;
    font-size: 12pt;
    line-height: 1.2;
    
    /* Ocultar encabezados y pies de página en Chrome/Edge */
    -webkit-print-header-footer: false;
    print-header-footer: false;
  }
  
  /* Ocultar encabezados y pies de página en Firefox */
  @page :footer { display: none !important; content: none !important; }
  @page :header { display: none !important; content: none !important; }
  
  /* Ocultar cualquier elemento que pueda estar en las esquinas */
  body::before,
  body::after,
  html::before,
  html::after,
  #app::before,
  #app::after,
  .page-marker,
  .corner,
  .watermark,
  .page-break,
  .print-corner {
    display: none !important;
    content: none !important;
    visibility: hidden !important;
  }
  
  /* Forzar colores para impresión */
  * {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
    color-adjust: exact !important;
  }
  
  /* Asegurar fondos blancos y bordes negros cuando sea necesario */
  .bg-white, .print\:bg-white {
    background-color: white !important;
  }
  
  .border, .border-gray-300, .print\:border-gray-300, .print\:border-black {
    border-color: #000 !important;
  }
  
  /* Mapas deben ser visibles en impresión */
  #report-map,
  .leaflet-container,
  .print-map-container,
  .print-visible-map {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
    visibility: visible !important;
    display: block !important;
    width: 100% !important;
    height: 300px !important;
    background: white !important;
    position: relative !important;
  }
  
  /* Asegurar que el contenedor del mapa no tenga desbordamiento */
  .print-map-container {
    overflow: hidden !important;
    position: relative !important;
  }
  
  /* Asegurar que el mapa ocupe todo el espacio disponible */
  .leaflet-container {
    width: 100% !important;
    height: 100% !important;
    min-height: 100% !important;
  }
  
  #app, #report-content, .report-print {
    margin: 0 !important;
    padding: 0 !important;
    width: 100% !important;
    box-shadow: none !important;
    background: white !important;
    border: none !important;
  }
  
  .no-print, .no-print * {
    display: none !important;
  }
  
  .dashboard-print-header {
    display: block !important;
    margin: 0 0 15px 0 !important;
    padding: 10px 0 !important;
    border-bottom: 1px solid #eee;
    page-break-after: avoid;
  }
  
  .dashboard-print-header img {
    margin-right: 10px;
  } 
  
@media print {
  @page {
    size: letter;
    margin: 0;   /* Eliminar márgenes por defecto */
    size: auto;  /* Tamaño de página automático */
  }
  
  .print-footer {
    display: block !important; 
    bottom: 0 !important;
    left: 0 !important;
    right: 0 !important;
    width: 100% !important;
    padding: 10px 1.5cm !important;
    border-top: 2px solid #3b82f6 !important;
    background-color: #eff6ff !important;
    z-index: 9999 !important;
    page-break-inside: avoid !important;
    text-align: center !important;
    font-size: 0.9em !important;
    color: #1e40af !important;
  } 
  
  /* Ocultar controles de zoom de Leaflet en impresión */
  .leaflet-control-zoom, 
  .leaflet-control-zoom-in, 
  .leaflet-control-zoom-out {
    display: none !important;
    visibility: hidden !important;
  }
  
  /* Asegurar que los mapas se muestren correctamente */
  .leaflet-container {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
    transform: none !important; /* Evitar transformaciones que puedan afectar la posición */
  }
  
  /* Asegurar que el mapa ocupe todo el ancho disponible */
  #report-map {
    width: 100% !important;
    height: 300px !important;
    position: relative !important;
    overflow: hidden !important;
    margin: 0 auto !important;
    padding: 0 !important;
    left: 0 !important;
    right: 0 !important;
  }
  
  /* Asegurar que el contenedor de impresión no tenga márgenes adicionales */
  .print-map-container {
    margin: 0 auto !important;
    padding: 0 !important;
    width: 100% !important;
    height: 300px !important;
    position: relative !important;
    overflow: hidden !important;
    display: flex !important;
    justify-content: center !important;
  }
}

  
  /* Asegurar que los contenedores no tengan márgenes ni padding */
  .container, .mx-auto, .p-4, .p-6, .px-4, .py-4, .m-4, .my-4, .mx-4 {
    margin: 0 auto !important;
    padding: 0 !important;
    max-width: 100% !important;
    width: 100% !important;
  }
  
  .report-print * {
    color: black !important;
    background: white !important;
  }
  
  /* Evitar saltos de página dentro de secciones importantes */
  .print-keep-together {
    page-break-inside: avoid;
  }
  
  /* Asegurar que las tablas no se corten */
  table {
    page-break-inside: auto !important;
  }
  
  tr {
    page-break-inside: avoid;
    page-break-after: auto;
  }
  
  /* Mejorar la legibilidad de los textos */
  h1, h2, h3, h4, h5, h6 {
    page-break-after: avoid;
  }
  
  /* Ajustar márgenes para los contenedores */
  .container, .mx-auto {
    width: 100% !important;
    max-width: 100% !important;
    padding: 0 !important;
    margin: 0 !important;
  }
  
  /* Asegurar que las imágenes se ajusten al ancho */
  img, svg {
    max-width: 100% !important;
    height: auto !important;
  }
  
  .print\:hidden {
    display: none !important;
  }
  
  .print\:text-black {
    color: #000 !important;
  }
  
  .print\:bg-white {
    background-color: #fff !important;
  }
  
  .print\:border-gray-300, .print\:border-black {
    border-color: #000 !important;
  }
  
  .print\:shadow-none {
    box-shadow: none !important;
  }

  .print\:grid-cols-2 {
    grid-template-columns: repeat(2, minmax(0, 1fr)) !important;
  }

  .print\:relative {
    position: relative !important;
  }

  .print\:flex-shrink-0 {
    flex-shrink: 0 !important;
  }

  .print\:flex-grow {
    flex-grow: 1 !important;
  }

  .print\:text-center {
    text-align: center !important;
  }

  .print\:mx-4 {
    margin-left: 1rem !important;
    margin-right: 1rem !important;
  }

  .print\:block {
    display: block !important;
  }
}

* { 
  font-family: 'Inter', sans-serif; 
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
  padding: 1rem;
}

.modal-content {
  background: white;
  border-radius: 8px;
  max-width: min(90%, 900px);
  max-height: 90%;
  overflow: auto;
  margin: 0 auto;
}

.modal-map-content {
  max-width: min(95%, 900px);
}

.dark .modal-content {
  background: #1f2937;
}

.chart-container {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 0; /* Permite que el contenedor se encoja */
}

/* Asegurar que los canvas se ajusten correctamente */
canvas {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.form-section {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border: 2px solid #e2e8f0;
  border-radius: 1rem;
  padding: 1rem;
  margin: 1rem 0;
  transition: all 0.3s ease;
}

.dark .form-section {
  background: linear-gradient(135deg, #374151 0%, #1f2937 100%);
  border-color: #4b5563;
}

@media (min-width: 768px) {
  .form-section {
    padding: 1.5rem;
  }
}

.form-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}

.input-group {
  background: white;
  border-radius: 0.75rem;
  padding: 0.75rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  margin: 0.5rem 0;
}

.dark .input-group {
  background: #374151;
}

@media (min-width: 768px) {
  .input-group {
    padding: 1rem;
  }
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
  font-size: 1rem;
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
  font-size: 1rem;
}

.btn-secondary:hover {
  transform: translateY(-1px);
  box-shadow: 0 10px 25px rgba(100, 116, 139, 0.3);
}

.report-type-card {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 1rem;
  padding: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

@media (min-width: 768px) {
  .report-type-card {
    padding: 1.5rem;
  }
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
  padding: 1.5rem;
  margin: 1rem 0;
}

.dark .alert-banner {
  background: linear-gradient(135deg, #451a03 0%, #7c2d12 100%);
  border-color: #ea580c;
}

@media (min-width: 768px) {
  .alert-banner {
    padding: 2rem;
  }
}

/* Mobile responsive adjustments */
@media (max-width: 1279px) {
  .xl\:grid-cols-4 {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  
  .xl\:grid-cols-2 {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
  
  .xl\:grid-cols-3 {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 1023px) {
  .lg\:grid-cols-4 {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  
  .lg\:grid-cols-3 {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  
  .lg\:grid-cols-2 {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
}

@media (max-width: 767px) {
  .md\:grid-cols-4,
  .md\:grid-cols-3,
  .md\:grid-cols-2 {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
  
  .text-lg {
    font-size: 1rem;
  }
  
  .text-xl {
    font-size: 1.125rem;
  }
  
  .text-2xl {
    font-size: 1.25rem;
  }
  
  .px-4 {
    padding-left: 0.75rem;
    padding-right: 0.75rem;
  }
  
  .py-6 {
    padding-top: 1rem;
    padding-bottom: 1rem;
  }
  
  .gap-4 {
    gap: 0.75rem;
  }
}

/* Ensure minimum font size for inputs on mobile */
input[type="text"], 
input[type="number"], 
input[type="date"], 
input[type="email"], 
input[type="password"], 
input[type="tel"], 
input[type="url"], 
select, 
textarea {
  font-size: 16px !important;
}

/* Custom scrollbar for webkit browsers */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}

.dark ::-webkit-scrollbar-track {
  background: #374151;
}

.dark ::-webkit-scrollbar-thumb {
  background: #6b7280;
}

.dark ::-webkit-scrollbar-thumb:hover {
  background: #9ca3af;
}
/* Eliminar encabezados y pies de página del navegador y evitar contenido en esquinas */
  @page {
    size: A4;
    margin: 0.8cm;
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
    marks: none;
    /* Eliminar cualquier contenido automático en esquinas */
    @top-left { content: none !important; }
    @top-center { content: none !important; }
    @top-right { content: none !important; }
    @bottom-left { content: none !important; }
    @bottom-center { content: none !important; }
    @bottom-right { content: none !important; }
    @top-left-corner { content: none !important; }
    @top-right-corner { content: none !important; }
    @bottom-left-corner { content: none !important; }
    @bottom-right-corner { content: none !important; }
  }
  
  /* Eliminar headers y footers automáticos */
  @page :first { 
    margin: 0.8cm; 
    @top-left { content: none !important; }
    @top-right { content: none !important; }
    @bottom-left { content: none !important; }
    @bottom-right { content: none !important; }
  }
  @page :left { 
    margin: 0.8cm; 
    @top-left { content: none !important; }
    @bottom-left { content: none !important; }
  }
  @page :right { 
    margin: 0.8cm; 
    @top-right { content: none !important; }
    @bottom-right { content: none !important; }
  }
  
  body, html { 
    -webkit-print-color-adjust: exact; 
    print-color-adjust: exact;
    margin: 0 !important;
    padding: 0 !important;
    background: white !important;
    width: 100% !important;
    height: 100% !important;
    font-size: 12pt;
    line-height: 1.2;
    
    /* Ocultar encabezados y pies de página en Chrome/Edge */
    -webkit-print-header-footer: false;
    print-header-footer: false;
  }
  
  /* Ocultar encabezados y pies de página en Firefox */
  @page :footer { display: none !important; content: none !important; }
  @page :header { display: none !important; content: none !important; }
  
  /* Ocultar cualquier elemento que pueda estar en las esquinas */
  body::before,
  body::after,
  html::before,
  html::after,
  #app::before,
  #app::after,
  .page-marker,
  .corner,
  .watermark,
  .page-break,
  .print-corner {
    display: none !important;
    content: none !important;
    visibility: hidden !important;
  }
</style>
