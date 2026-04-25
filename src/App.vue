<template>
  <header>
    <div id="topBar">
        <div class="flex items-center gap-3">
            <button @click="handleLeftPanel('leftPanel')">☰</button>
            <h1>Life Recovery after Noto Peninsula Earthquake</h1>
        </div>
        <div class="flex gap-4">
            <button @click="handleOpenModal('overview')">Overview</button>
            <button @click="handleOpenModal('about')">Instruction</button>
            <button @click="handleOpenModal('recognition')">Recognition</button>
        </div>
    </div>
  </header>

  <main>
    <Map ref="chartRef" />
    <ModalControl ref="modalRef" />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import Nlp from './components/Nlp.vue'
import Map from './components/Map.vue'
import ModalControl from './components/ModalControl.vue'

// references to child components
const modalRef = ref<InstanceType<typeof ModalControl> | null>(null)
const chartRef = ref<InstanceType<typeof Map> | null>(null)

// call openModal from ModalControl when top bar buttons are clicked
const handleOpenModal = (type: string) => {
  if (modalRef.value) {
    modalRef.value.openModal(type)
  }
}

// call openModal from ModalControl when left panel button is clicked
const handleLeftPanel = (type: string) => {
  if (chartRef.value) {
    chartRef.value.openModal(type)
  }
}
</script>

<style>
#app {
  display: block; 
  max-width: none ;
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
}

html, body, #app {
  height: 100%;
  margin: 0;
  overflow: hidden; 
}

main {
  position: fixed;
  top: 60px;   
  left: 0;
  right: 0;
  bottom: 0; 
  overflow: hidden;
}
main > * {
  height: 100%;
}

/* for all panel */
.panel {
    background: rgba(255,255,255,0.95);
    border-radius: 12px;
    padding: 20px;
    box-shadow: 0 6px 20px rgba(0,0,0,0.15);
}  
</style>
<style scoped>
#topBar {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 60px;
    background: #6a7282;
    color: white;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    z-index: 2000;
    font-weight: bold;
    font-size: 18px;
    box-shadow: 0 2px 6px rgba(0,0,0,0.15);
}
</style>
