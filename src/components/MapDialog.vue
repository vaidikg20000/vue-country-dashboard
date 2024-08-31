<template>
    
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

// Define props for open state and link
const props = defineProps({
  open: Boolean,
  link: String
})

// Emit events to the parent component
const emit = defineEmits(['close'])

// Function to emit close event
const closeModal = () => {
  emit('close')
}
</script>