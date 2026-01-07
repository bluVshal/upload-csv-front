<template>
  <div class="spinner-overlay">
    <div class="spinner-container">
      <div class="spinner"></div>
      <p class="spinner-text">{{ message }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoadingSpinner',
  props: {
    message: {
      type: String,
      default: 'Processing...'
    }
  }
};
</script>

<style scoped>
.spinner-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.spinner-container {
  text-align: center;
}

.spinner {
  border: 8px solid #f3f3f3;
  border-top: 8px solid #0e4500;
  border-radius: 50%;
  width: 60px;
  height: 60px;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.spinner-text {
  color: white;
  font-size: 18px;
  font-weight: 500;
  margin: 0;
}
</style>
