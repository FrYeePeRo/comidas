<template>
  <!-- Header de Categorías para Menú -->
  <section class="bg-white shadow-sm sticky-top border-bottom" style="top: 76px; z-index: 1020;">
    <!-- Métodos de entrega -->
    <div class="border-bottom">
      <div class="container-xl">
        <div class="row g-0">
          <div class="col-6">
            <button
              @click="$emit('update-delivery', 'pickup')"
              class="btn w-100 py-3 border-0 rounded-0"
              :class="deliveryMethod === 'pickup' ? 'btn-primary' : 'btn-light'"
            >
              <i class="fas fa-store me-2"></i>Recoger en tienda
            </button>
          </div>
          <div class="col-6">
            <button
              @click="$emit('update-delivery', 'delivery')"
              class="btn w-100 py-3 border-0 rounded-0"
              :class="deliveryMethod === 'delivery' ? 'btn-primary' : 'btn-light'"
            >
              <i class="fas fa-motorcycle me-2"></i>Domicilio
            </button>
          </div>
        </div>
      </div>
    </div>

    

    <!-- Categorías Móvil - Scroll horizontal -->
    <div class="d-lg-none">
      <div class="container-fluid">
        <div class="py-3">
          <h6 class="text-muted mb-3 fw-bold px-3">
            <i class="fas fa-utensils me-2"></i>
            Categorías:
          </h6>
          <div class="d-flex gap-2 overflow-auto pb-2 px-3" style="scrollbar-width: none; -ms-overflow-style: none;">
            <button
              v-for="category in categories"
              :key="category.id"
              @click="handleCategorySelect(category)"
              class="btn text-nowrap flex-shrink-0"
              :class="selectedCategory?.id === category.id ? 'btn-primary' : 'btn-outline-primary'"
              style="min-width: 120px;"
            >
              <div class="d-flex flex-column align-items-center">
                <span class="fs-5 mb-1">{{ category.icon }}</span>
                <span class="small fw-bold">{{ category.name }}</span>
                <span class="text-muted" style="font-size: 0.7rem;">{{ category.count }}</span>
              </div>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Botón para limpiar selección -->
    <div v-if="selectedCategory" class="container-xl">
      <div class="py-2 border-top">
        <button
          @click="handleClearSelection"
          class="btn btn-outline-secondary btn-sm"
        >
          <i class="fas fa-times me-1"></i>
          Limpiar selección
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
// Props
const props = defineProps({
  categories: {
    type: Array,
    required: true
  },
  selectedCategory: {
    type: Object,
    default: null
  },
  deliveryMethod: {
    type: String,
    default: 'pickup'
  }
})

// Emits
const emit = defineEmits(['select-category', 'clear-selection', 'update-delivery'])

// Métodos
const handleCategorySelect = (category) => {
  emit('select-category', category)
  
  // Feedback háptico
  if ('vibrate' in navigator) {
    navigator.vibrate(30)
  }
  
  // Scroll suave al contenido
  setTimeout(() => {
    const content = document.querySelector('#products-content')
    if (content) {
      content.scrollIntoView({ behavior: 'smooth', block: 'start' })
    }
  }, 100)
  
  console.log(`📂 Categoría seleccionada: ${category.name}`)
}

const handleClearSelection = () => {
  emit('clear-selection')
  console.log('🧹 Selección de categoría limpiada')
}
</script>

<style scoped>
/* Estilo hover para categorías */
.category-btn:hover:not(.btn-primary) {
  background-color: #f8f9fa !important;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

/* Transiciones suaves */
.category-btn {
  transition: all 0.2s ease;
}

/* Ocultar scrollbar en móvil */
.overflow-auto::-webkit-scrollbar {
  display: none;
}

/* Asegurar posicionamiento sticky */
.sticky-top {
  position: sticky !important;
  z-index: 1020;
}
</style>
