<template>
  <div class="bg-white/90 p-4 rounded-xl shadow-lg border border-slate-200">
    <div class="mt-4 text-xs space-y-2">
      <textarea 
        v-model="inputText"
        class="w-full h-24 p-2 border rounded-lg text-sm text-slate-600"
        placeholder="Please enter a Japanese sentence."
      ></textarea>
      <button 
        @click="handleAnalyze"
        :disabled="loading"
        class="w-full mt-2 bg-slate-600 text-white py-2 rounded-lg hover:bg-slate-700 disabled:bg-slate-300"
      >
      {{ loading ? 'Analyzing...' : 'Submit' }}
      </button>
    </div>
    <div v-if="result" class="mt-4 text-xs space-y-2 pt-4">
        <div class="border-t border-gray-200 pt-3 pb-3">
            <div class="flex flex-wrap gap-1 ">
              <span class="text-sm font-bold text-slate-700">Life Recovery Elements：</span>
              <span v-for="e in result.elements" :key="e" class="bg-gray-100 text-gray-700 px-2 py-1 rounded text-xs font-bold mr-1 mb-1 shadow-sm border border-gray-400">{{ e }}</span>
              <span v-if="!result.elements || result.elements.length === 0" class="text-sm font-bold text-slate-700">Do not exist</span>
            </div>
        </div>
        <div class="text-sm font-bold text-slate-700 border-t border-gray-200 pt-3 pb-3">
          <span>Location：</span>
          <span class="text-gray-600">{{ result.places?.join(', ') || 'Do not eixst Noto Peninsula city name'}}</span>
        </div>
        <div class="text-sm font-bold text-slate-700 border-t border-gray-200 pt-3">
          <span>Sentiment：</span>
          <span class="text-gray-600">{{ result.sentiment }}</span>
        </div>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
const inputText = ref('能登半島地震で被災した輪島塗の工房を支援しようと東日本大震災の被災地で仮設商店として使われていたトレーラーハウス2台が、9日、輪島市の団体「輪島塗若手ネットワーク」に寄贈されました。');
const result = ref(null);
const loading = ref(false);

const handleAnalyze = async () => {
  if (!inputText.value) return;
  loading.value = true;
  try {
    const res = await axios.post('https://noto-recovery-webgis.onrender.com/analyze', {
      text: inputText.value
    });
    result.value = res.data;
  } finally {
    loading.value = false;
  }
};
</script>