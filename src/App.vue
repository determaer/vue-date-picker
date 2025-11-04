<script setup lang="ts">
import DatePicker from "./components/DatePicker.vue";
</script>

<template>
  <DatePicker
    date="2025-11-25"
    @selected-date="
      (date) => {
        console.log(date);
      }
    "
  />
</template>

<style scoped></style>

