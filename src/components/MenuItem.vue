<template>
  <div class="col">
    <div class="card h-100 border-0 shadow-sm">
      <img :src="item.image" class="card-img-top" :alt="item.name">
      <div class="card-body">
        <h6 class="card-title">{{ item.name }}</h6>
        <p class="card-text text-muted small">{{ item.description }}</p>
        <div class="d-flex justify-content-between align-items-center">
          <span class="price">{{ item.price }}</span>
          <button class="btn btn-sm btn-outline-danger">+</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    item: Object
  }
}
</script>

<style scoped>
.card {
  border-radius: 12px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.card-img-top {
  object-fit: cover;
  height: 200px;
  border-radius: 12px;
}

.card-body {
  background-color: #f9f9f9;
}

.card-body .price {
  font-size: 1.25rem;
  color: #e74c3c;
}

.card-body button {
  background-color: #e74c3c;
  border: none;
  color: white;
  padding: 8px 15px;
  border-radius: 5px;
  transition: background-color 0.3s;
}

.card-body button:hover {
  background-color: #c0392b;
}
</style>
