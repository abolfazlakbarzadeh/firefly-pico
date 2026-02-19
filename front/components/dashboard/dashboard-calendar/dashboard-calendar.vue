<template>
  <van-cell-group inset>
    <dashboard-calendar-switching :start="startDate" :end="endDate" />
  </van-cell-group>
</template>

<script setup>
const dataStore = useDataStore()

const startDate = computed(() => new Date(dataStore.dashboardDateStart))
const endDate = computed(() => new Date(dataStore.dashboardDateEnd))
</script>
