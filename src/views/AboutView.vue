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



</style>
