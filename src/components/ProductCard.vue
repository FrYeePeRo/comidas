<template>
  <div 
    class="product-card"
    :class="{ 'adding': isAdding }"
    ref="cardRef"
  >
    <!-- Imagen del producto con overlay -->
    <div class="product-image-wrapper">
      <img
        :src="product.image"
        :alt="product.name"
        class="product-image"
        @error="handleImageError"
        @load="onImageLoad"
        loading="lazy"
      />
      
      <!-- Loading skeleton -->
      <div 
        v-if="imageLoading" 
        class="image-skeleton"
      >
        <div class="skeleton-shimmer"></div>
      </div>
      
      <!-- Badges superiores -->
      <div class="product-badges">
        <div class="badge-rating">
          <i class="fas fa-star"></i>
          <span>{{ product.rating }}</span>
        </div>
        <div class="badge-time">
          <i class="fas fa-clock"></i>
          <span>{{ product.time }}</span>
        </div>
      </div>
      
      <!-- Botón de agregar rápido -->
      <button
        @click="handleQuickAdd"
        class="quick-add-btn"
        :disabled="isAdding"
        title="Agregar al carrito"
      >
        <i class="fas fa-plus" v-if="!isAdding"></i>
        <div class="loading-spinner" v-else></div>
      </button>
      
      <!-- Overlay hover -->
      <div class="product-overlay">
        <div class="overlay-content">
          <h5 class="overlay-title">{{ product.name }}</h5>
          <p class="overlay-description">{{ truncatedDescription }}</p>
          <div class="overlay-price">${{ formatPrice(product.price) }}</div>
        </div>
      </div>
    </div>
    
    <!-- Información del producto -->
    <div class="product-info">
      <div class="product-header">
        <h5 class="product-title">{{ product.name }}</h5>
        <div class="product-price">${{ formatPrice(product.price) }}</div>
      </div>
      
      <p class="product-description">{{ product.description }}</p>
      
      <!-- Información adicional -->
      <div class="product-meta">
        <div class="meta-item">
          <i class="fas fa-leaf text-success"></i>
          <span>Ingredientes frescos</span>
        </div>
        <div class="meta-item">
          <i class="fas fa-fire text-warning"></i>
          <span>~450 cal</span>
        </div>
      </div>
      
      <!-- Acciones del producto -->
      <div class="product-actions">
        <button
          @click="handleAddToCart"
          class="btn-add-cart"
          :disabled="isAdding"
        >
          <div class="btn-content">
            <i class="fas fa-shopping-bag btn-icon"></i>
            <span class="btn-text">
              {{ isAdding ? 'Agregando...' : 'Agregar al carrito' }}
            </span>
          </div>
          <div class="btn-loading" v-if="isAdding">
            <div class="loading-dots">
              <span></span>
              <span></span>
              <span></span>
            </div>
          </div>
        </button>
        
        <!-- Botón secundario -->
        <button
          @click="showDetails"
          class="btn-details"
          title="Ver detalles"
        >
          <i class="fas fa-info-circle"></i>
        </button>
      </div>
    </div>
    
    <!-- Indicador de éxito -->
    <div class="success-indicator" :class="{ 'show': showSuccess }">
      <i class="fas fa-check-circle"></i>
      <span>¡Agregado!</span>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

// Props
const props = defineProps({
  product: {
    type: Object,
    required: true,
    validator: (product) => {
      return product.id && product.name && product.price && product.image
    }
  }
})

// Emits
const emit = defineEmits(['add-to-cart'])

// Estado local
const imageLoading = ref(true)
const isAdding = ref(false)
const showSuccess = ref(false)
const cardRef = ref(null)

// Computed properties
const truncatedDescription = computed(() => {
  if (props.product.description.length > 80) {
    return props.product.description.substring(0, 80) + '...'
  }
  return props.product.description
})

// Lifecycle hooks
onMounted(() => {
  console.log(`📦 ProductCard montado: ${props.product.name}`)
  setupIntersectionObserver()
})

onBeforeUnmount(() => {
  if (intersectionObserver) {
    intersectionObserver.disconnect()
  }
})

// Variables para el observer
let intersectionObserver = null

// Métodos
const handleAddToCart = async () => {
  if (isAdding.value) return

  isAdding.value = true
  
  try {
    // Simular delay de red
    await new Promise(resolve => setTimeout(resolve, 600))
    
    emit('add-to-cart', props.product)
    
    // Mostrar feedback de éxito
    showSuccessAnimation()
    
    // Vibración táctil
    if ('vibrate' in navigator) {
      navigator.vibrate([50, 100, 50])
    }
    
  } catch (error) {
    console.error('Error agregando al carrito:', error)
    showErrorAnimation()
  } finally {
    setTimeout(() => {
      isAdding.value = false
    }, 300)
  }
}

const handleQuickAdd = () => {
  handleAddToCart()
}

const onImageLoad = () => {
  imageLoading.value = false
}

const handleImageError = (event) => {
  console.warn(`Error cargando imagen: ${props.product.name}`)
  imageLoading.value = false
  event.target.src = generatePlaceholderImage()
}

const generatePlaceholderImage = () => {
  const colors = ['#E31E24', '#FFD700', '#FF6B35', '#00B894']
  const randomColor = colors[Math.floor(Math.random() * colors.length)]
  
  const svg = `
    <svg width="400" height="300" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:${randomColor};stop-opacity:0.8" />
          <stop offset="100%" style="stop-color:${randomColor};stop-opacity:0.4" />
        </linearGradient>
      </defs>
      <rect width="400" height="300" fill="url(#grad)"/>
      <circle cx="200" cy="120" r="40" fill="rgba(255,255,255,0.3)"/>
      <rect x="150" y="180" width="100" height="8" fill="rgba(255,255,255,0.4)"/>
      <rect x="160" y="200" width="80" height="6" fill="rgba(255,255,255,0.3)"/>
      <text x="200" y="250" text-anchor="middle" fill="rgba(255,255,255,0.9)" font-family="sans-serif" font-size="14" font-weight="600">
        ${props.product.name}
      </text>
    </svg>
  `
  return `data:image/svg+xml;base64,${btoa(svg)}`
}

const formatPrice = (price) => {
  return new Intl.NumberFormat('es-CO').format(price)
}

const showSuccessAnimation = () => {
  showSuccess.value = true
  
  // Ocultar después de 2 segundos
  setTimeout(() => {
    showSuccess.value = false
  }, 2000)
  
  // Animación de la tarjeta
  if (cardRef.value) {
    cardRef.value.classList.add('success-animation')
    setTimeout(() => {
      cardRef.value.classList.remove('success-animation')
    }, 600)
  }
}

const showErrorAnimation = () => {
  if (cardRef.value) {
    cardRef.value.classList.add('error-animation')
    setTimeout(() => {
      cardRef.value.classList.remove('error-animation')
    }, 600)
  }
}

const showDetails = () => {
  // Mostrar modal de detalles o navegar a página de detalles
  console.log(`ℹ️ Mostrar detalles de: ${props.product.name}`)
  
  // Por ahora, simplemente mostrar un toast
  showToast(`Detalles de ${props.product.name}`, 'info')
}

const showToast = (message, type = 'info') => {
  const toast = document.createElement('div')
  toast.className = `product-toast toast-${type}`
  toast.innerHTML = `
    <div class="toast-content">
      <i class="fas fa-info-circle toast-icon"></i>
      <span class="toast-message">${message}</span>
    </div>
  `
  
  document.body.appendChild(toast)
  
  // Mostrar el toast
  setTimeout(() => toast.classList.add('show'), 100)
  
  // Auto-remove después de 3 segundos
  setTimeout(() => {
    toast.classList.remove('show')
    setTimeout(() => document.body.removeChild(toast), 300)
  }, 3000)
}

const setupIntersectionObserver = () => {
  intersectionObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add('animate-in')
          intersectionObserver.unobserve(entry.target)
        }
      })
    },
    { 
      threshold: 0.1,
      rootMargin: '50px'
    }
  )
  
  if (cardRef.value) {
    intersectionObserver.observe(cardRef.value)
  }
}
</script>

<style scoped>
/* Tarjeta principal */
.product-card {
  background: var(--white);
  border-radius: var(--border-radius-lg);
  overflow: hidden;
  transition: all var(--transition-normal);
  position: relative;
  opacity: 0;
  transform: translateY(30px);
  box-shadow: var(--shadow-sm);
  border: 1px solid rgba(227, 30, 36, 0.05);
}

.product-card.animate-in {
  opacity: 1;
  transform: translateY(0);
}

.product-card:hover {
  transform: translateY(-8px);
  box-shadow: var(--shadow-xl);
  border-color: rgba(227, 30, 36, 0.2);
}

.product-card.adding {
  transform: scale(1.02);
}

/* Wrapper de imagen */
.product-image-wrapper {
  position: relative;
  height: 220px;
  overflow: hidden;
  background: var(--light-gray);
}

.product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform var(--transition-slow);
}

.product-card:hover .product-image {
  transform: scale(1.1);
}

/* Skeleton loading */
.image-skeleton {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

/* Badges */
.product-badges {
  position: absolute;
  top: 1rem;
  left: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  z-index: 2;
}

.badge-rating,
.badge-time {
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  color: var(--white);
  padding: 0.4rem 0.8rem;
  border-radius: var(--border-radius-md);
  font-size: 0.8rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.badge-rating {
  background: rgba(255, 215, 0, 0.9);
  color: var(--dark-gray);
}

.badge-time {
  background: rgba(227, 30, 36, 0.9);
}

/* Botón de agregar rápido */
.quick-add-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  width: 45px;
  height: 45px;
  border-radius: 50%;
  border: none;
  background: var(--white);
  color: var(--primary-red);
  box-shadow: var(--shadow-md);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--transition-normal);
  opacity: 0;
  transform: scale(0.8);
  z-index: 2;
}

.product-card:hover .quick-add-btn {
  opacity: 1;
  transform: scale(1);
}

.quick-add-btn:hover {
  background: var(--primary-red);
  color: var(--white);
  transform: scale(1.1);
  box-shadow: var(--shadow-lg);
}

.quick-add-btn:disabled {
  cursor: not-allowed;
  opacity: 0.7;
}

/* Loading spinner */
.loading-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid var(--light-gray);
  border-top: 2px solid var(--primary-red);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Overlay hover */
.product-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, 
    rgba(227, 30, 36, 0.95) 0%, 
    rgba(196, 30, 58, 0.95) 100%);
  color: var(--white);
  opacity: 0;
  transition: all var(--transition-normal);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 1.5rem;
  z-index: 1;
}

.product-card:hover .product-overlay {
  opacity: 1;
}

.overlay-content {
  transform: translateY(20px);
  transition: transform var(--transition-normal);
}

.product-card:hover .overlay-content {
  transform: translateY(0);
}

.overlay-title {
  font-weight: 700;
  margin-bottom: 0.75rem;
  font-size: 1.2rem;
}

.overlay-description {
  margin-bottom: 1rem;
  opacity: 0.9;
  line-height: 1.4;
}

.overlay-price {
  font-size: 1.5rem;
  font-weight: 900;
  color: var(--accent-yellow);
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

/* Información del producto */
.product-info {
  padding: 1.5rem;
}

.product-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.75rem;
  gap: 1rem;
}

.product-title {
  font-weight: 600;
  color: var(--dark-gray);
  margin: 0;
  flex: 1;
  font-size: 1.1rem;
  line-height: 1.3;
}

.product-price {
  font-weight: 700;
  font-size: 1.25rem;
  color: var(--primary-red);
  font-family: var(--font-primary);
  flex-shrink: 0;
}

.product-description {
  color: var(--medium-gray);
  font-size: 0.9rem;
  line-height: 1.5;
  margin-bottom: 1rem;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Meta información */
.product-meta {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.8rem;
  color: var(--medium-gray);
}

.meta-item i {
  font-size: 0.9rem;
}

/* Acciones del producto */
.product-actions {
  display: flex;
  gap: 0.75rem;
  align-items: center;
}

.btn-add-cart {
  flex: 1;
  background: linear-gradient(135deg, var(--primary-red), var(--primary-red-dark));
  border: none;
  color: var(--white);
  padding: 0.875rem 1.5rem;
  border-radius: var(--border-radius-md);
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-normal);
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 48px;
}

.btn-add-cart::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transition: left var(--transition-slow);
}

.btn-add-cart:hover::before {
  left: 100%;
}

.btn-add-cart:hover {
  background: linear-gradient(135deg, var(--primary-red-dark), var(--primary-red));
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
}

.btn-add-cart:disabled {
  cursor: not-allowed;
  opacity: 0.8;
}

.btn-content {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: opacity var(--transition-fast);
}

.btn-add-cart:disabled .btn-content {
  opacity: 0;
}

.btn-icon {
  font-size: 0.9rem;
}

.btn-text {
  font-size: 0.9rem;
}

.btn-loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  opacity: 0;
  transition: opacity var(--transition-fast);
}

.btn-add-cart:disabled .btn-loading {
  opacity: 1;
}

.loading-dots {
  display: flex;
  gap: 0.3rem;
}

.loading-dots span {
  width: 6px;
  height: 6px;
  background: var(--white);
  border-radius: 50%;
  animation: loadingDots 1.4s infinite ease-in-out;
}

.loading-dots span:nth-child(1) { animation-delay: -0.32s; }
.loading-dots span:nth-child(2) { animation-delay: -0.16s; }

@keyframes loadingDots {
  0%, 80%, 100% { transform: scale(0); opacity: 0.5; }
  40% { transform: scale(1); opacity: 1; }
}

.btn-details {
  width: 48px;
  height: 48px;
  border: 2px solid var(--primary-red);
  background: transparent;
  color: var(--primary-red);
  border-radius: var(--border-radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--transition-normal);
  flex-shrink: 0;
}

.btn-details:hover {
  background: var(--primary-red);
  color: var(--white);
  transform: translateY(-2px);
}

/* Indicador de éxito */
.success-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: var(--success);
  color: var(--white);
  padding: 0.75rem 1.5rem;
  border-radius: var(--border-radius-lg);
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: 600;
  box-shadow: var(--shadow-lg);
  z-index: 10;
  opacity: 0;
  transform: translate(-50%, -50%) scale(0.8);
  transition: all var(--transition-normal);
  pointer-events: none;
}

.success-indicator.show {
  opacity: 1;
  transform: translate(-50%, -50%) scale(1);
}

.success-indicator i {
  font-size: 1.1rem;
}

/* Animaciones */
.product-card.success-animation {
  animation: successPulse 0.6s ease-out;
}

.product-card.error-animation {
  animation: errorShake 0.6s ease-out;
}

@keyframes successPulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); box-shadow: 0 0 0 10px rgba(0, 184, 148, 0.3); }
  100% { transform: scale(1); }
}

@keyframes errorShake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

/* Toast notifications */
.product-toast {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: var(--white);
  border-radius: var(--border-radius-md);
  box-shadow: var(--shadow-xl);
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  z-index: 9999;
  transform: translateX(400px);
  transition: transform var(--transition-normal);
  min-width: 250px;
  border-left: 4px solid var(--info);
}

.product-toast.show {
  transform: translateX(0);
}

.toast-icon {
  color: var(--info);
  font-size: 1.1rem;
}

.toast-message {
  color: var(--dark-gray);
  font-weight: 500;
}

/* Responsive */
@media (max-width: 768px) {
  .product-image-wrapper {
    height: 180px;
  }
  
  .product-info {
    padding: 1rem;
  }
  
  .product-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
  
  .product-price {
    font-size: 1.5rem;
  }
  
  .product-actions {
    flex-direction: column;
  }
  
  .btn-add-cart {
    width: 100%;
  }
  
  .btn-details {
    width: 100%;
  }
  
  .product-overlay {
    padding: 1rem;
  }
  
  .overlay-title {
    font-size: 1rem;
  }
  
  .overlay-price {
    font-size: 1.25rem;
  }
  
  .product-toast {
    left: 20px;
    right: 20px;
    min-width: auto;
    transform: translateY(100px);
  }
  
  .product-toast.show {
    transform: translateY(0);
  }
}

@media (max-width: 480px) {
  .product-badges {
    top: 0.5rem;
    left: 0.5rem;
  }
  
  .quick-add-btn {
    top: 0.5rem;
    right: 0.5rem;
    width: 40px;
    height: 40px;
  }
  
  .badge-rating,
  .badge-time {
    padding: 0.3rem 0.6rem;
    font-size: 0.75rem;
  }
  
  .meta-item {
    font-size: 0.75rem;
  }
  
  .btn-text {
    font-size: 0.85rem;
  }
}
</style>
