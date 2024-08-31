<template>
  <div class="about">
    <h1>This is an about page</h1>
    <q-btn
      label="Map"
      color="primary"
      @click="icon=true"
      />
      <div v-if="icon" class="modal-overlay">
      <div class="modal-content">
        <h3>My Modal</h3>
        <p>This is a modal example.</p>
        <button @click="closeModal">Close Modal</button>
      </div>
    </div>
    
  </div>
</template>

<script setup>
import {ref} from "vue";

const icon = ref(false);

const closeModal = () =>{
  icon.value = false;
}
</script>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}


.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
</style>
