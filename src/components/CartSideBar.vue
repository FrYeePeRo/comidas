<template>
  <!-- Backdrop -->
  <div 
    class="cart-backdrop"
    :class="{ 'show': isVisible }"
    @click="handleClose"
  ></div>
  
  <!-- Sidebar del carrito -->
  <div 
    class="cart-sidebar"
    :class="{ 'open': isVisible }"
    ref="sidebarRef"
  >
    <div class="cart-container">
      <!-- Header del carrito -->
      <div class="cart-header">
        <div class="cart-title-section">
          <div class="cart-icon-wrapper">
            <i class="fas fa-shopping-bag"></i>
            <span v-if="cart.length > 0" class="cart-count-badge">{{ cartItemsCount }}</span>
          </div>
          <div class="cart-title-content">
            <h3 class="cart-title">Tu Pedido</h3>
            <p class="cart-subtitle" v-if="cart.length > 0">
              {{ cart.length }} {{ cart.length === 1 ? 'producto' : 'productos' }}
            </p>
          </div>
        </div>
        
        <button
          @click="handleClose"
          class="close-button"
          title="Cerrar carrito"
        >
          <i class="fas fa-times"></i>
        </button>
      </div>
      
      <!-- Contenido del carrito -->
      <div class="cart-content">
        <!-- Estado vacío -->
        <div v-if="cart.length === 0" class="empty-cart">
          <div class="empty-illustration">
            <div class="empty-cart-icon">
              <i class="fas fa-shopping-bag"></i>
            </div>
            <div class="empty-circles">
              <div class="circle circle-1"></div>
              <div class="circle circle-2"></div>
              <div class="circle circle-3"></div>
            </div>
          </div>
          
          <div class="empty-content">
            <h4 class="empty-title">Tu carrito está vacío</h4>
            <p class="empty-description">
              Agrega productos deliciosos del menú para empezar tu pedido
            </p>
            <router-link to="/menu" class="btn-browse-menu" @click="handleClose">
              <i class="fas fa-utensils me-2"></i>
              Explorar Menú
            </router-link>
          </div>
        </div>
        
        <!-- Lista de productos -->
        <div v-else class="cart-items">
          <div
            v-for="(item, index) in cart"
            :key="item.id"
            class="cart-item"
            :style="{ animationDelay: `${index * 0.1}s` }"
          >
            <div class="item-image-wrapper">
              <img
                :src="item.image"
                :alt="item.name"
                class="item-image"
                @error="handleImageError"
              />
            </div>
            
            <div class="item-details">
              <h5 class="item-name">{{ item.name }}</h5>
              <p class="item-description">{{ truncateText(item.description, 40) }}</p>
              
              <div class="item-footer">
                <div class="item-price">
                  <span class="unit-price">${{ formatPrice(item.price) }}</span>
                  <span class="total-price" v-if="item.quantity > 1">
                    Total: ${{ formatPrice(item.price * item.quantity) }}
                  </span>
                </div>
                
                <div class="quantity-controls">
                  <button
                    @click="updateQuantity(item.id, item.quantity - 1)"
                    class="quantity-btn decrease"
                    :disabled="item.quantity <= 1"
                  >
                    <i class="fas fa-minus"></i>
                  </button>
                  
                  <span class="quantity-display">{{ item.quantity }}</span>
                  
                  <button
                    @click="updateQuantity(item.id, item.quantity + 1)"
                    class="quantity-btn increase"
                  >
                    <i class="fas fa-plus"></i>
                  </button>
                </div>
              </div>
            </div>
            
            <button
              @click="removeItem(item.id)"
              class="remove-item-btn"
              title="Eliminar producto"
            >
              <i class="fas fa-trash-alt"></i>
            </button>
          </div>
        </div>
      </div>
      
      <!-- Footer del carrito -->
      <div v-if="cart.length > 0" class="cart-footer">
        <!-- Resumen de costos -->
        <div class="cost-summary">
          <div class="cost-row">
            <span class="cost-label">Subtotal</span>
            <span class="cost-value">${{ formatPrice(total) }}</span>
          </div>
          
          <div class="cost-row">
            <span class="cost-label">
              Domicilio
              <i 
                class="fas fa-info-circle info-icon" 
                title="Gratis en pedidos superiores a $20.000"
              ></i>
            </span>
            <span class="cost-value" :class="{ 'free': deliveryFee === 0 }">
              {{ deliveryFee === 0 ? 'GRATIS' : `$${formatPrice(deliveryFee)}` }}
            </span>
          </div>
          
          <div class="cost-row total-row">
            <span class="cost-label">Total</span>
            <span class="cost-value total-amount">${{ formatPrice(finalTotal) }}</span>
          </div>
        </div>
        
        <!-- Promoción -->
        <div class="promo-banner" v-if="total < 20000">
          <i class="fas fa-gift"></i>
          <span>
            Agrega ${{ formatPrice(20000 - total) }} más para 
            <strong>envío gratis</strong>
          </span>
        </div>
        
        <!-- Botones de acción -->
        <div class="cart-actions">
          <button 
            @click="proceedToCheckout"
            class="checkout-btn"
            :disabled="isProcessing"
          >
            <div class="btn-content">
              <i class="fas fa-credit-card btn-icon"></i>
              <span class="btn-text">
                {{ isProcessing ? 'Procesando...' : 'Proceder al Pago' }}
              </span>
            </div>
            <div class="btn-loading" v-if="isProcessing">
              <div class="loading-spinner"></div>
            </div>
          </button>
          
          <div class="alternative-actions">
            <a 
              :href="whatsappUrl"
              target="_blank"
              class="action-btn whatsapp-btn"
              @click="trackEvent('whatsapp_order')"
            >
              <i class="fab fa-whatsapp"></i>
              <span>Pedir por WhatsApp</span>
            </a>
            
            <a 
              href="tel:+573232494876"
              class="action-btn phone-btn"
              @click="trackEvent('phone_order')"
            >
              <i class="fas fa-phone"></i>
              <span>Llamar para ordenar</span>
            </a>
          </div>
        </div>
        
        
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch, nextTick } from 'vue'

// Props
const props = defineProps({
  cart: {
    type: Array,
    required: true
  },
  total: {
    type: Number,
    required: true
  }
})

// Emits
const emit = defineEmits(['close', 'update-quantity', 'proceed-checkout'])

// Estado local
const isVisible = ref(false)
const isProcessing = ref(false)
const sidebarRef = ref(null)

// Computed properties
const deliveryFee = computed(() => {
  return props.total < 20000 ? 3000 : 0
})

const finalTotal = computed(() => {
  return props.total + deliveryFee.value
})

const cartItemsCount = computed(() => {
  return props.cart.reduce((total, item) => total + item.quantity, 0)
})

const whatsappUrl = computed(() => {
  let message = "🍔 ¡Hola! Me gustaría hacer el siguiente pedido:\n\n"
  
  props.cart.forEach((item, index) => {
    message += `${index + 1}. ${item.name} x${item.quantity} - $${formatPrice(item.price * item.quantity)}\n`
  })
  
  message += `\n💰 Subtotal: $${formatPrice(props.total)}`
  message += `\n🚗 Domicilio: ${deliveryFee.value === 0 ? 'GRATIS' : '$' + formatPrice(deliveryFee.value)}`
  message += `\n🎯 Total: $${formatPrice(finalTotal.value)}`
  message += `\n\n¡Muchas gracias! 🙏`
  
  return `https://wa.me/573232494876?text=${encodeURIComponent(message)}`
})

// Lifecycle hooks
onMounted(() => {
  console.log('🛒 CartSidebar moderno montado')
  setupEventListeners()
  showSidebar()
})

onBeforeUnmount(() => {
  cleanup()
})

// Watchers
watch(() => props.cart.length, (newLength, oldLength) => {
  if (newLength > oldLength) {
    playAddSound()
  }
})

// Métodos
const showSidebar = () => {
  nextTick(() => {
    isVisible.value = true
    lockBodyScroll()
  })
}

const handleClose = () => {
  isVisible.value = false
  
  setTimeout(() => {
    emit('close')
    unlockBodyScroll()
  }, 300)
}

const updateQuantity = (productId, newQuantity) => {
  emit('update-quantity', productId, newQuantity)
  
  // Vibración táctil
  if ('vibrate' in navigator) {
    navigator.vibrate(30)
  }
  
  // Sonido de feedback
  playClickSound()
}

const removeItem = (productId) => {
  // Animación de salida
  const itemElement = document.querySelector(`[data-item-id="${productId}"]`)
  if (itemElement) {
    itemElement.classList.add('removing')
  }
  
  setTimeout(() => {
    updateQuantity(productId, 0)
  }, 300)
}

const proceedToCheckout = async () => {
  isProcessing.value = true
  
  try {
    // Simular proceso de checkout
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    emit('proceed-checkout')
    showSuccessMessage()
    
  } catch (error) {
    console.error('Error en checkout:', error)
    showErrorMessage()
  } finally {
    isProcessing.value = false
  }
}

const handleImageError = (event) => {
  event.target.src = generatePlaceholderImage()
}

const generatePlaceholderImage = () => {
  const svg = `
    <svg width="80" height="80" xmlns="http://www.w3.org/2000/svg">
      <rect width="80" height="80" fill="#f8f9fa"/>
      <circle cx="40" cy="30" r="12" fill="#dee2e6"/>
      <rect x="28" y="50" width="24" height="4" fill="#dee2e6"/>
      <rect x="30" y="58" width="20" height="3" fill="#ced4da"/>
    </svg>
  `
  return `data:image/svg+xml;base64,${btoa(svg)}`
}

const formatPrice = (price) => {
  return new Intl.NumberFormat('es-CO').format(price)
}

const truncateText = (text, maxLength) => {
  if (text.length <= maxLength) return text
  return text.substring(0, maxLength) + '...'
}

const setupEventListeners = () => {
  document.addEventListener('keydown', handleKeyPress)
}

const handleKeyPress = (event) => {
  if (event.key === 'Escape') {
    handleClose()
  }
}

const lockBodyScroll = () => {
  document.body.style.overflow = 'hidden'
  document.body.style.paddingRight = getScrollbarWidth() + 'px'
}

const unlockBodyScroll = () => {
  document.body.style.overflow = ''
  document.body.style.paddingRight = ''
}

const getScrollbarWidth = () => {
  const scrollDiv = document.createElement('div')
  scrollDiv.style.cssText = 'width: 100px; height: 100px; overflow: scroll; position: absolute; top: -9999px;'
  document.body.appendChild(scrollDiv)
  const scrollbarWidth = scrollDiv.offsetWidth - scrollDiv.clientWidth
  document.body.removeChild(scrollDiv)
  return scrollbarWidth
}

const cleanup = () => {
  document.removeEventListener('keydown', handleKeyPress)
  unlockBodyScroll()
}

const playAddSound = () => {
  // Crear contexto de audio para feedback sonoro
  try {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()
    const oscillator = audioContext.createOscillator()
    const gainNode = audioContext.createGain()
    
    oscillator.connect(gainNode)
    gainNode.connect(audioContext.destination)
    
    oscillator.frequency.setValueAtTime(800, audioContext.currentTime)
    gainNode.gain.setValueAtTime(0.1, audioContext.currentTime)
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1)
    
    oscillator.start()
    oscillator.stop(audioContext.currentTime + 0.1)
  } catch (error) {
    // Fallar silenciosamente si el audio no está disponible
  }
}

const playClickSound = () => {
  // Sonido más suave para clicks
  try {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()
    const oscillator = audioContext.createOscillator()
    const gainNode = audioContext.createGain()
    
    oscillator.connect(gainNode)
    gainNode.connect(audioContext.destination)
    
    oscillator.frequency.setValueAtTime(600, audioContext.currentTime)
    gainNode.gain.setValueAtTime(0.05, audioContext.currentTime)
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.05)
    
    oscillator.start()
    oscillator.stop(audioContext.currentTime + 0.05)
  } catch (error) {
    // Fallar silenciosamente
  }
}

const showSuccessMessage = () => {
  showToast('¡Pedido enviado exitosamente!', 'success')
}

const showErrorMessage = () => {
  showToast('Error al procesar el pedido. Intenta de nuevo.', 'error')
}

const showToast = (message, type) => {
  const toast = document.createElement('div')
  toast.className = `cart-toast toast-${type}`
  toast.innerHTML = `
    <div class="toast-content">
      <i class="fas ${type === 'success' ? 'fa-check-circle' : 'fa-exclamation-circle'} toast-icon"></i>
      <span class="toast-message">${message}</span>
    </div>
  `
  
  document.body.appendChild(toast)
  
  setTimeout(() => toast.classList.add('show'), 100)
  
  setTimeout(() => {
    toast.classList.remove('show')
    setTimeout(() => document.body.removeChild(toast), 300)
  }, 4000)
}

const trackEvent = (eventName) => {
  console.log(`📊 Event tracked: ${eventName}`)
  
  // Aquí se puede integrar con analytics
  if (typeof gtag !== 'undefined') {
    gtag('event', eventName, {
      event_category: 'cart',
      event_label: 'order_action'
    })
  }
}
</script>

<style scoped>
/* Backdrop */
.cart-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(4px);
  z-index: 1040;
  opacity: 0;
  visibility: hidden;
  transition: all var(--transition-normal);
}

.cart-backdrop.show {
  opacity: 1;
  visibility: visible;
}

/* Sidebar principal */
.cart-sidebar {
  position: fixed;
  top: 0;
  right: 0;
  width: 420px;
  max-width: 100vw;
  height: 100vh;
  background: var(--white);
  z-index: 1050;
  transform: translateX(100%);
  transition: transform var(--transition-normal);
  box-shadow: -10px 0 30px rgba(0, 0, 0, 0.15);
}

.cart-sidebar.open {
  transform: translateX(0);
}

.cart-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

/* Header del carrito */
.cart-header {
  padding: 2rem 1.5rem 1rem;
  border-bottom: 1px solid rgba(227, 30, 36, 0.1);
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
}

.cart-title-section {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.cart-icon-wrapper {
  position: relative;
  width: 50px;
  height: 50px;
  background: linear-gradient(135deg, var(--primary-red), var(--primary-red-dark));
  border-radius: var(--border-radius-lg);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--white);
  font-size: 1.25rem;
}

.cart-count-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background: var(--accent-yellow);
  color: var(--dark-gray);
  font-size: 0.7rem;
  font-weight: 700;
  padding: 0.2rem 0.5rem;
  border-radius: 50px;
  min-width: 20px;
  text-align: center;
  line-height: 1;
}

.cart-title-content {
  flex: 1;
}

.cart-title {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--dark-gray);
  margin: 0;
  font-family: var(--font-primary);
}

.cart-subtitle {
  color: var(--medium-gray);
  margin: 0;
  font-size: 0.9rem;
}

.close-button {
  width: 40px;
  height: 40px;
  border: none;
  background: var(--light-gray);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--transition-normal);
  color: var(--medium-gray);
}

.close-button:hover {
  background: var(--primary-red);
  color: var(--white);
  transform: rotate(90deg);
}

/* Contenido del carrito */
.cart-content {
  flex: 1;
  overflow-y: auto;
  padding: 0 1.5rem;
}

/* Estado vacío */
.empty-cart {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  text-align: center;
  padding: 2rem;
}

.empty-illustration {
  position: relative;
  margin-bottom: 2rem;
}

.empty-cart-icon {
  width: 120px;
  height: 120px;
  background: linear-gradient(135deg, var(--light-gray), #e9ecef);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3rem;
  color: var(--medium-gray);
  margin: 0 auto;
  position: relative;
  z-index: 2;
}

.empty-circles {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
}

.circle {
  position: absolute;
  border-radius: 50%;
  background: rgba(227, 30, 36, 0.1);
  animation: float 3s ease-in-out infinite;
}

.circle-1 {
  width: 20px;
  height: 20px;
  top: 20px;
  right: 10px;
  animation-delay: 0s;
}

.circle-2 {
  width: 15px;
  height: 15px;
  bottom: 30px;
  left: 15px;
  animation-delay: 1s;
}

.circle-3 {
  width: 12px;
  height: 12px;
  top: 50px;
  left: 20px;
  animation-delay: 2s;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

.empty-content {
  max-width: 280px;
}

.empty-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--dark-gray);
  margin-bottom: 0.75rem;
}

.empty-description {
  color: var(--medium-gray);
  margin-bottom: 2rem;
  line-height: 1.5;
}

.btn-browse-menu {
  background: linear-gradient(135deg, var(--primary-red), var(--primary-red-dark));
  color: var(--white);
  text-decoration: none;
  padding: 0.875rem 2rem;
  border-radius: var(--border-radius-md);
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  transition: all var(--transition-normal);
}

.btn-browse-menu:hover {
  color: var(--white);
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
}

/* Lista de productos */
.cart-items {
  padding: 1rem 0;
}

.cart-item {
  display: flex;
  gap: 1rem;
  padding: 1.5rem 0;
  border-bottom: 1px solid rgba(227, 30, 36, 0.1);
  position: relative;
  opacity: 0;
  transform: translateX(20px);
  animation: slideInFade 0.5s ease-out forwards;
}

@keyframes slideInFade {
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.cart-item.removing {
  animation: slideOutFade 0.3s ease-out forwards;
}

@keyframes slideOutFade {
  to {
    opacity: 0;
    transform: translateX(-20px);
    height: 0;
    padding: 0;
    margin: 0;
  }
}

.item-image-wrapper {
  flex-shrink: 0;
  width: 80px;
  height: 80px;
  border-radius: var(--border-radius-md);
  overflow: hidden;
  background: var(--light-gray);
}

.item-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.item-details {
  flex: 1;
  min-width: 0;
}

.item-name {
  font-size: 1rem;
  font-weight: 600;
  color: var(--dark-gray);
  margin-bottom: 0.25rem;
  line-height: 1.3;
}

.item-description {
  font-size: 0.85rem;
  color: var(--medium-gray);
  margin-bottom: 1rem;
  line-height: 1.4;
}

.item-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.item-price {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.unit-price {
  font-weight: 600;
  color: var(--primary-red);
  font-size: 0.95rem;
}

.total-price {
  font-size: 0.8rem;
  color: var(--medium-gray);
}

.quantity-controls {
  display: flex;
  align-items: center;
  background: var(--light-gray);
  border-radius: var(--border-radius-md);
  padding: 0.25rem;
  gap: 0.5rem;
}

.quantity-btn {
  width: 30px;
  height: 30px;
  border: none;
  background: var(--white);
  border-radius: var(--border-radius-sm);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--transition-fast);
  color: var(--primary-red);
  font-size: 0.8rem;
}

.quantity-btn:hover:not(:disabled) {
  background: var(--primary-red);
  color: var(--white);
  transform: scale(1.1);
}

.quantity-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.quantity-display {
  font-weight: 600;
  color: var(--dark-gray);
  min-width: 20px;
  text-align: center;
  font-size: 0.9rem;
}

.remove-item-btn {
  position: absolute;
  top: 0.5rem;
  right: 0;
  width: 30px;
  height: 30px;
  border: none;
  background: transparent;
  color: var(--medium-gray);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--transition-normal);
  opacity: 0;
}

.cart-item:hover .remove-item-btn {
  opacity: 1;
}

.remove-item-btn:hover {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
  transform: scale(1.1);
}

/* Footer del carrito */
.cart-footer {
  padding: 1.5rem;
  border-top: 1px solid rgba(227, 30, 36, 0.1);
  background: var(--light-gray);
}

.cost-summary {
  background: var(--white);
  border-radius: var(--border-radius-md);
  padding: 1.5rem;
  margin-bottom: 1rem;
}

.cost-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.cost-row:last-child {
  margin-bottom: 0;
}

.cost-label {
  color: var(--medium-gray);
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.info-icon {
  font-size: 0.8rem;
  opacity: 0.7;
  cursor: help;
}

.cost-value {
  font-weight: 600;
  color: var(--dark-gray);
}

.cost-value.free {
  color: var(--success);
  font-weight: 700;
}

.total-row {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(227, 30, 36, 0.1);
  margin-top: 0.75rem;
}

.total-row .cost-label,
.total-row .cost-value {
  font-size: 1.1rem;
  font-weight: 700;
  color: var(--dark-gray);
}

.total-amount {
  color: var(--primary-red) !important;
  font-size: 1.25rem !important;
}

/* Banner promocional */
.promo-banner {
  background: linear-gradient(135deg, rgba(255, 215, 0, 0.1), rgba(255, 107, 53, 0.1));
  border: 1px solid rgba(255, 215, 0, 0.3);
  border-radius: var(--border-radius-md);
  padding: 1rem;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: 0.9rem;
  color: var(--dark-gray);
}

.promo-banner i {
  color: var(--accent-orange);
  font-size: 1.1rem;
}

/* Acciones del carrito */
.cart-actions {
  margin-bottom: 1rem;
}

.checkout-btn {
  width: 100%;
  background: linear-gradient(135deg, var(--primary-red), var(--primary-red-dark));
  border: none;
  color: var(--white);
  padding: 1rem 1.5rem;
  border-radius: var(--border-radius-md);
  font-weight: 700;
  font-size: 1rem;
  cursor: pointer;
  transition: all var(--transition-normal);
  position: relative;
  overflow: hidden;
  margin-bottom: 1rem;
  min-height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.checkout-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transition: left var(--transition-slow);
}

.checkout-btn:hover::before {
  left: 100%;
}

.checkout-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, var(--primary-red-dark), var(--primary-red));
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
}

.checkout-btn:disabled {
  opacity: 0.8;
  cursor: not-allowed;
}

.btn-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  transition: opacity var(--transition-fast);
}

.checkout-btn:disabled .btn-content {
  opacity: 0;
}

.btn-loading {
  position: absolute;
  opacity: 0;
  transition: opacity var(--transition-fast);
}

.checkout-btn:disabled .btn-loading {
  opacity: 1;
}

.loading-spinner {
  width: 24px;
  height: 24px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top: 2px solid var(--white);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.alternative-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.75rem;
}

.action-btn {
  padding: 0.75rem 1rem;
  border-radius: var(--border-radius-md);
  text-decoration: none;
  font-weight: 600;
  font-size: 0.85rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: all var(--transition-normal);
  text-align: center;
}

.whatsapp-btn {
  background: rgba(37, 211, 102, 0.1);
  color: #25d366;
  border: 1px solid rgba(37, 211, 102, 0.3);
}

.whatsapp-btn:hover {
  background: #25d366;
  color: var(--white);
  transform: translateY(-2px);
}

.phone-btn {
  background: rgba(74, 144, 226, 0.1);
  color: #4a90e2;
  border: 1px solid rgba(74, 144, 226, 0.3);
}

.phone-btn:hover {
  background: #4a90e2;
  color: var(--white);
  transform: translateY(-2px);
}

/* Información del carrito */
.cart-info {
  display: flex;
  justify-content: space-between;
  gap: 0.5rem;
  font-size: 0.75rem;
  color: var(--medium-gray);
}

.info-item {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  flex: 1;
  text-align: center;
  justify-content: center;
}

.info-item i {
  color: var(--success);
  font-size: 0.8rem;
}

/* Toast notifications */
.cart-toast {
  position: fixed;
  bottom: 20px;
  left: 20px;
  background: var(--white);
  border-radius: var(--border-radius-md);
  box-shadow: var(--shadow-xl);
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  z-index: 9999;
  transform: translateY(100px);
  transition: transform var(--transition-normal);
  min-width: 300px;
}

.cart-toast.show {
  transform: translateY(0);
}

.cart-toast.toast-success {
  border-left: 4px solid var(--success);
}

.cart-toast.toast-error {
  border-left: 4px solid #ef4444;
}

.toast-icon {
  font-size: 1.2rem;
}

.toast-success .toast-icon {
  color: var(--success);
}

.toast-error .toast-icon {
  color: #ef4444;
}

.toast-message {
  color: var(--dark-gray);
  font-weight: 500;
}

/* Responsive */
@media (max-width: 768px) {
  .cart-sidebar {
    width: 100%;
  }
  
  .cart-header {
    padding: 1.5rem 1rem 1rem;
  }
  
  .cart-content {
    padding: 0 1rem;
  }
  
  .cart-footer {
    padding: 1rem;
  }
  
  .alternative-actions {
    grid-template-columns: 1fr;
  }
  
  .cart-info {
    flex-direction: column;
    gap: 0.75rem;
  }
  
  .info-item {
    justify-content: flex-start;
  }
}

@media (max-width: 480px) {
  .cart-title {
    font-size: 1.25rem;
  }
  
  .cart-icon-wrapper {
    width: 45px;
    height: 45px;
    font-size: 1.1rem;
  }
  
  .item-footer {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.75rem;
  }
  
  .quantity-controls {
    align-self: flex-end;
  }
}
</style>
