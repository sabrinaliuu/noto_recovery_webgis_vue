<template>
    <div class="relative w-full h-full bg-slate-100">
        <div v-show="visiblePanels.leftPanel" class="w-96 p-4 overflow-y-auto space-y-4 left-panel" >
            
            <div class="bg-white p-4 rounded-xl shadow-sm border border-slate-200 panel">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-bold mb-4 text-slate-700">Control Panel</h2>
                    <button @click="closeModal('leftPanel')" class="text-2xl hover:text-gray-500">×</button>
                </div>    
                <div class="mb-4">
                    <label class="text-xs font-bold text-slate-500 uppercase">1. Recovery Element</label>
                    <select v-model="currentElem" class="w-full mt-1 p-2 border rounded-md text-slate-700">
                        <option v-for="item in recoveryElements" :key="item" :value="item">{{ item }}</option>
                    </select>
                </div>

                <div>
                    <label class="text-xs font-bold text-slate-500 uppercase">2. Month: {{ monthLabels[currentIdx] }}</label>
                    <input type="range" v-model.number="currentIdx" min="0" :max="monthLabels.length - 1" class="w-full">
                </div>
            </div>

            <div class="bg-white p-4 rounded-xl shadow-sm border border-slate-200 h-90 panel">
                <div class="flex justify-between items-center mb-4">
                    <div>
                        <h2 class="text-lg font-bold text-slate-800">Sentiment Comparison</h2>
                        <p  class="text-xs text-slate-400">Recovery Element：{{ currentElem }}</p>
                    </div>
                    <span  class="bg-gray-100 text-gray-700 px-3 py-1 rounded-full text-xs font-bold">{{ monthLabels[currentIdx] }}</span>
                </div>
                <div class="h-70 pt-4">
                    <canvas id="comparisonChart"></canvas>
                </div>
            </div>
        </div>

        <!-- Map -->
        <div class="flex-grow relative" style="height:100%; width:100%;">
            <l-map ref="map" :zoom="10" :center="[37.3, 137.0]" :use-global-leaflet="false" :options="{ zoomControl: false }" style="height:100%; width:100%;" >
                <l-control-zoom position="topright"></l-control-zoom>
                <l-control position="bottomright" >
                    <h4 class="font-bold mb-2 text-slate-500">Legend</h4>
                    <div class="space-y-1 text-slate-500 pt-2 pb-1">
                        <div><i :style="{ background: sentimentColors['positive'] }"></i>Positive</div>
                        <div><i :style="{ background: sentimentColors['neutral'] }"></i>Neutral</div>
                        <div><i :style="{ background: sentimentColors['negative'] }"></i>Negative</div>
                    </div>
                    <div class="border-t pt-2 mt-2 text-[10px] text-slate-400">
                        <p>© 2026 Sabrina Liu</p>
                    </div>
                </l-control>
                <l-tile-layer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
                <l-geo-json v-if="geojson" :geojson="geojson" :options-style="geojsonStyle" :options="geojsonOptions">
                </l-geo-json>
                <!-- Marker for each region: pie chart -->
                <!-- createPieIcon function will be called when change occurs -->
                <l-marker 
                    v-for="feature in geoData?.features" 
                    :key="feature.properties.SIKUCHOSON"
                    :lat-lng="[feature.geometry.coordinates[1], feature.geometry.coordinates[0]]"
                    :icon="createPieIcon(feature.properties.data[currentElem][currentIdx])"
                    @click="showDetail(feature.properties)"
                >
                    <l-tooltip>{{ feature.properties.SIKUCHOSON }}</l-tooltip>
                </l-marker>
            </l-map>
        </div>

        <!-- Detail Modal -->
        <div v-if="isDetailVisible" class="detail-panel panel" @mousedown="startDrag($event, 'isDetailVisible')">
            <div class="flex justify-between items-center mb-4 border-b pb-2 cursor-move">
                <span class="font-bold text-lg text-gray-700">{{ selectedRegion?.SIKUCHOSON }}</span>
                <button @click="isDetailVisible = false" class="text-2xl hover:text-gray-500">&times;</button>                    
            </div>
            <p  class="text-xs text-slate-400">Recovery Element：{{ currentElem }}</p>
            <div class="h-80 pt-3">
                <canvas id="miniChart"></canvas>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, watch, computed, reactive, nextTick } from 'vue';
import 'leaflet/dist/leaflet.css';
import { LMap, LTileLayer, LMarker, LTooltip, LGeoJson, LControlZoom, LControl } from "@vue-leaflet/vue-leaflet";
import L from 'leaflet';
import Chart from 'chart.js/auto';

// basemap and city polygon
const geojson = ref(null);
const geojsonStyle = (feature) => {
  return {
    fillColor: "#9dc5f3",
    fillOpacity: 0.5,
    color: "#21315f",
    weight: 4
  };
};
const geojsonOptions = {
    onEachFeature: (feature, layer) => {

        // popup
        layer.bindPopup(`<strong>${feature.properties.SIKUCHOSON}</strong>`);

        // hover in
        layer.on('mouseover', (e) => {
        const el = e.target;
        el.setStyle({
            fillColor: "#9dc5f3",
            fillOpacity: 0.8,
            weight: 6,
            color: "#21315f"
        });
        el.bringToFront();
        });

        // hover out
        layer.on('mouseout', (e) => {
        const el = e.target;
        el.setStyle({
            fillColor: "#9dc5f3",
            fillOpacity: 0.5,
            color: "#21315f",
            weight: 4
        });
        });
    }
};

// ref constants
const currentIdx = ref(0);
const currentElem = ref("housing");
const geoData = ref(null);
const isDetailVisible = ref(false);
const selectedRegion = ref(null);

// array/dict constants
const monthLabels = ["202408", "202409", "202410", "202411", "202412", "202501", "202502", "202503", "202504", "202505", "202506", "202507"];
const recoveryElements = ["housing", "social ties", "townscape", "physical and mental health", "preparedness", "relation to government", "economic and financial situation"];
const sentimentColors = {
    "positive": "#10b981", 
    "neutral": "#94a3b8", 
    "negative": "#f43f5e",
    "total": "#6366f1"
};

// define chart instance
let comparisonChart = null;
let miniChart = null;

// -- handle comparison bar chart --
const updateCharts = () => {
    if (!geoData.value || !comparisonChart) return;
    comparisonChart.data.datasets[0].data = geoData.value.features.map(f => f.properties.data[currentElem.value][currentIdx.value]?.pPos || 0);
    comparisonChart.data.datasets[1].data = geoData.value.features.map(f => f.properties.data[currentElem.value][currentIdx.value]?.pNeu || 0);
    comparisonChart.data.datasets[2].data = geoData.value.features.map(f => f.properties.data[currentElem.value][currentIdx.value]?.pNeg || 0);
    comparisonChart.update();
};

// -- handle pie chart --
const createPieIcon = (data) => {
    const size = Math.sqrt(data.count || 1) * 15;
    const html = `
        <div class="pie-icon" style="width: ${size}px; height: ${size}px;">
            <div style="width: 100%; height: 100%; border-radius: 50%; background: conic-gradient(
                ${sentimentColors.positive} 0% ${data.pPos}%,
                ${sentimentColors.neutral} ${data.pPos}% ${data.pPos + data.pNeu}%,
                ${sentimentColors.negative} ${data.pPos + data.pNeu}% 100%
            );"></div>
        </div>
    `;
    return L.divIcon({ html, className: '', iconSize: [size, size] });
};

// -- handle line chart and detail panel --
const showDetail = async (region) => {
    selectedRegion.value = region;
    isDetailVisible.value = true;
    
    await nextTick(); // wait for detail-panel to be rendered
    
    const ctxLine = document.getElementById('miniChart')?.getContext('2d');
    if (!ctxLine) return;
    if (miniChart) {
        miniChart.destroy();
    }

    miniChart = new Chart(ctxLine, {
        type: 'line',
        data: {
            labels: monthLabels,
            datasets: [
                { 
                    label: 'Positive %', 
                    borderColor: sentimentColors.positive, 
                    data: region.data[currentElem.value].map(d => d.pPos || 0) 
                },
                { 
                    label: 'Neutral %', 
                    borderColor: sentimentColors.neutral, 
                    data: region.data[currentElem.value].map(d => d.pNeu || 0) 
                },
                { 
                    label: 'Negative %', 
                    borderColor: sentimentColors.negative, 
                    data: region.data[currentElem.value].map(d => d.pNeg || 0) 
                },
                { 
                    label: 'Total', 
                    borderColor: sentimentColors.total, 
                    borderDash: [5,5], 
                    data: region.data[currentElem.value].map(d => d.count || 0), 
                    yAxisID: 'y1' 
                }
            ]
        },
        options: { 
            responsive: true, 
            maintainAspectRatio: false, 
            scales: { 
                y: {
                    max: 100,
                    position: 'left',
                    title: { display: true, text: 'Percentage (%)' }
                },
                y1: { 
                    position: 'right', 
                    grid: { drawOnChartArea: false },
                    title: { display: true, text: 'Total Count' }
                }
            }
        }
    });
};

const updateMiniCharts = () => {
    if (!selectedRegion.value || !miniChart) return;
    const regionData = selectedRegion.value.data[currentElem.value];
    miniChart.data.datasets[0].data = regionData.map(d => d.pPos || 0);
    miniChart.data.datasets[1].data = regionData.map(d => d.pNeu || 0);
    miniChart.data.datasets[2].data = regionData.map(d => d.pNeg || 0);
    miniChart.data.datasets[3].data = regionData.map(d => d.count || 0);
    miniChart.update();
};

// --- handle control panel ---
const visiblePanels = reactive({
  leftPanel: true
})

const closeModal = (type) => {
  visiblePanels[type] = false
}

const openModal = (type) => {
  visiblePanels[type] = true
}
defineExpose({ openModal }) // for APP.vue to call

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

// --- on mounted ---
onMounted(async () => {

    // fetch geojson data for polygons
    try {
        const response = await fetch(import.meta.env.BASE_URL + '/noto_poly.geojson');
        if (!response.ok) throw new Error('error fetching GeoJSON');
        const data = await response.json();
        geojson.value = data;
    } catch (error) {
        console.error("Error fetching GeoJSON:", error);
    }

    // fetch geojson data for regions
    const res = await fetch(import.meta.env.BASE_URL + "/noto.geojson");
    geoData.value = await res.json();

    // init comparison bar chart
    const ctx = document.getElementById('comparisonChart').getContext('2d');
    comparisonChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: geoData.value.features.map(f => f.properties.SIKUCHOSON),
            datasets: [
                { label: 'Positve%', backgroundColor: sentimentColors.positive, data: [] },
                { label: 'Neutral%', backgroundColor: sentimentColors.neutral, data: [] },
                { label: 'Negative%', backgroundColor: sentimentColors.negative, data: [] }
            ]
        },
        options: { indexAxis: 'x', responsive: true, maintainAspectRatio: false, scales: { x: { stacked: true }, y: { stacked: true, max: 100 } } ,
            plugins: {
                legend: {
                    position: 'bottom',
                    labels: {
                        usePointStyle: true,
                        pointStyle: 'circle'
                    }
                }
            }
        }
    });
    updateCharts(); 
});

// watch for changes in currentIdx and currentElem to update charts
watch([currentIdx, currentElem], updateCharts);
watch([currentElem], updateMiniCharts);
</script>

<style scoped>
.left-panel {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 400px;
    display: flex;
    flex-direction: column;
    gap: 16px;
    z-index: 1000;
}

.detail-panel {
    position: absolute;
    width: 500px;
    max-width: 90vw;
    top: 15px;
    right: 50px;
    z-index: 1000;
}

.leaflet-control {
    background: white;
    padding: 12px;
    line-height: 20px;
    color: #333;
    border-radius: 8px;
    box-shadow: 0 0 15px rgba(0,0,0,0.1);
    font-size: 12px;
    width: 120px;
    height: 140px;
}
.leaflet-control i {
    width: 12px; height: 12px; float: left; margin-right: 8px; opacity: 0.8; border-radius: 50%;
}
</style>

<style>
.leaflet-container {
  width: 100%;
  height: 100%;
}
</style>