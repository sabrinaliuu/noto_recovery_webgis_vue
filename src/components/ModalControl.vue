<template>

  <div v-if="visiblePanels.overview" class="modal-panel shadow-2xl" @mousedown="startDrag($event, 'overview')">
    <div class="flex justify-between items-center mb-4 cursor-move">
      <h2 class="text-xl font-bold text-gray-700">Overview</h2>
      <button @click="closeModal('overview')" class="text-2xl hover:text-gray-500">×</button>
    </div>
    <div class="p-5 text-sm text-slate-600 leading-relaxed space-y-3 select-text">
        <p>This is the visualization of study <i>Analyzing the Seven Critical Elements of Life Recovery Using News: A Case Study of the 2024 Noto Peninsula Earthquake</i>by Yen-Ching Liu* and Shosuke Sato.</p>
        <p>In this study, <b>NLP methods</b> were applied to recognized the <b>recovery elements, sentiments, and locations</b> in each paragraph. In other words, the textual data was converted into numerical data for further quantitative analysis.</p>
        <p>This demonstrates the feasibility of using NLP methods on news data to gain insights into news media perceptions and to assess news coverage of disaster recovery. </p>
        <br/><hr><br/>
        <p>● Material: News on Yahoo! Japan News website</p>
        <p>● Time: August 2024 to July 2025</p>
        <p>● Seven Critical Elements for Life Recovery: housing, social ties, townscape, physical and mental health, preparedness, relation to government, and economic and financial situation (Tatsuki and Hayashi, 2002)</p>
        <p>● Sentiments: positive, negative, and neutral</p>
        <p>● 12 cities in Ishikawa Prefecture on the Noto Peninsula: Suzu-shi (珠洲市), Wajima-shi (輪島市), Noto-cho (能登町), Anamizu-machi (穴水町), Shika-machi (志賀町), Nanao-shi (七尾市), Nakanoto-machi (中能登町), Hakui-shi (羽咋市), Hodatsushimizu-cho (宝達志水町), Kahoku-shi (かほく市), Uchinada-machi (内灘町), Tsubata-machi (津幡町)</p>
        <br/><hr><br/>
        <p>This website is produced by Yen-Ching Liu with Gemini</p>
    </div>
  </div>

  <div v-if="visiblePanels.about" class="modal-panel shadow-2xl" @mousedown="startDrag($event, 'about')" >
    <div class="flex justify-between items-center mb-4 cursor-move">
      <h2 class="text-xl font-bold text-gray-700">How to Use</h2>
      <button @click="closeModal('about')" class="text-2xl hover:text-gray-500">×</button>
    </div>
    <div class="p-5 text-sm text-slate-600 leading-relaxed space-y-4">
      <h4 class="font-bold text-slate-800">Pie Chart</h4>
      <p>● <b>Color</b>：Proportion of <span class="text-emerald-600 font-bold">positive</span>, <span class="text-slate-400 font-bold">Neutral</span>, <span class="text-rose-500 font-bold">negative</span> sentiment<br>
         ● <b>Size</b>：More paragraphs → Bigger</p>
      <br/><hr><br/>
      <h4 class="font-bold text-slate-800">Control Panel</h4>
      <p>If you click on the pie chart, the line chart for that city will appear：<br>
         ●  <b>Solid Line (for left y-axis)</b>：change of the proportion of sentiments<br>
         ●  <b>Dot Line (for right y-axis)</b>：number of paragraphs</p>
      <br/><hr><br/>
      <h4 class="font-bold text-slate-800">Recognition: NLP Model Application</h4>
      <p>You can enter recovery-related sentence in Japanese here. The <b>Life Recovery Elements, Cities in Noto Peninsula, and Sentiment</b> will be recognized and displayed.</p>   
    </div>
  </div>

  <div  v-if="visiblePanels.recognition" class="modal-panel shadow-2xl" @mousedown="startDrag($event, 'recognition')">
    <div class="flex justify-between items-center mb-4 cursor-move">
      <h2 class="text-xl font-bold text-gray-700">Recognition</h2>
      <button @click="closeModal('recognition')" class="text-2xl hover:text-gray-500">×</button>
    </div>
    <div class="px-5 pb-2 text-sm text-slate-500">
      <p>Please enter recovery-related sentence in Japanese here. The <b>Life Recovery Elements, Cities in Noto Peninsula, and Sentiment</b> will be recognized and displayed.</p>
    </div>
    <Nlp />
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import Nlp from './Nlp.vue'

// panel visibility state
const visiblePanels = reactive({
  overview: false,
  about: false,
  recognition: false
})

const closeModal = (type) => {
  visiblePanels[type] = false
}

const openModal = (type) => {
  visiblePanels[type] = true
}
defineExpose({ openModal }) // for App.vue to use

const startDrag = (e, type) => {
  if (e.target.tagName === 'BUTTON' || e.target.tagName === 'TEXTAREA' || e.target.tagName === 'INPUT') return

  const el = e.currentTarget
  let startX = e.clientX - el.offsetLeft
  let startY = e.clientY - el.offsetTop

  const onMouseMove = (e) => {
    el.style.left = e.clientX - startX + 'px'
    el.style.top = e.clientY - startY + 'px'
    el.style.transform = 'none' 
  }

  const onMouseUp = () => {
    document.removeEventListener('mousemove', onMouseMove)
    document.removeEventListener('mouseup', onMouseUp)
  }

  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('mouseup', onMouseUp)
}
</script>

<style scoped>
.modal-panel {
  position: fixed; 
  top: 80px;
  right: 40px;
  width: 600px;
  height: 500px;
  max-width: 90vw;
  max-height: 85vh;
  overflow-y: auto;
  z-index: 2500;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 24px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.modal-panel::-webkit-scrollbar {
  width: 6px;
}
.modal-panel::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 10px;
}
</style>