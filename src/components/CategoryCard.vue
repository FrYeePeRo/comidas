<template>
  <div 
    class="card border-0 h-100 antaurus-hover-lift cursor-pointer"
    @click="handleSelect"
    ref="cardRef"
  >
    <div class="card-body text-center p-4">
      <div 
        class="rounded-circle d-inline-flex align-items-center justify-center mb-3 antaurus-hover-scale"
        :class="colorClass"
        style="width: 70px; height: 70px; font-size: 2rem;"
      >
        {{ category.icon }}
      </div>
      
      <h5 class="card-title fw-bold mb-2">{{ category.name }}</h5>
      
      <p class="card-text small mb-2 text-muted">
        {{ category.count }} productos
      </p>
      
      <div class="small text-muted">
        {{ category.description }}
      </div>
      
      <!-- Progress indicator -->
      <div class="mt-3">
        <div class="progress" style="height: 3px;">
          <div 
            class="progress-bar bg-primary"
            :style="{ width: progressWidth + '%' }"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// Props
const props = defineProps({
  category: {
    type: Object,
    required: true
  }
})

// Emits
const emit = defineEmits(['select'])

// Refs
const cardRef = ref(null)

// Computed properties
const colorClass = computed(() => {
  const colorMap = {
    1: 'bg-danger',     // Hamburguesas
    2: 'bg-warning',    // Acompañamientos
    3: 'bg-info',       // Bebidas
    4: 'bg-secondary',  // Postres
    5: 'bg-success',    // Desayunos
    6: 'bg-primary'     // Combos
  }
  return colorMap[props.category.id] || 'bg-primary'
})

const progressWidth = computed(() => {
  const maxProducts = 4 // Máximo de productos por categoría para el cálculo
  return Math.min((props.category.count / maxProducts) * 100, 100)
})

// Lifecycle
onMounted(() => {
  console.log(`🏷️ CategoryCard montado: ${props.category.name}`)
  setupAnimation()
})

// Métodos
const handleSelect = () => {
  emit('select', props.category)
  
  // Feedback háptico
  if ('vibrate' in navigator) {
    navigator.vibrate(30)
  }
  
  console.log(`📊 Categoría seleccionada: ${props.category.name}`)
}

const setupAnimation = () => {
  if (cardRef.value) {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            entry.target.style.animation = 'fadeInUp 0.6s ease-out'
            observer.unobserve(entry.target)
          }
        })
      },
      { threshold: 0.1 }
    )
    
    observer.observe(cardRef.value)
  }
}
</script>

<style scoped>
.cursor-pointer {
  cursor: pointer;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
