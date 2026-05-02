# Life Recovery after Noto Peninsula Earthquake
<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/ChatGPT-74aa9c?style=for-the-badge&logo=openai&logoColor=white" />
  <img src="https://img.shields.io/badge/QGIS-93b023?style=for-the-badge&logo=qgis&logoColor=white" />
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black" />
  <img src="https://img.shields.io/badge/Vue%20js-35495E?style=for-the-badge&logo=vuedotjs&logoColor=4FC08D" />
  <img src="https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/Leaflet-199900?style=for-the-badge&logo=Leaflet&logoColor=white" />
  <img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white" /></br>
  <img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/GIT-E44C30?style=for-the-badge&logo=git&logoColor=white" />
  <img src="https://img.shields.io/badge/R-276DC3?style=for-the-badge&logo=r&logoColor=white" />
  <img src="https://img.shields.io/badge/Visual_Studio-5C2D91?style=for-the-badge&logo=visual%20studio&logoColor=white" />
</p>

:round_pushpin:This is the migration to the new <b>Vue</b> version as `Vue + Leaflet` demo. 
If you want to see the other versions, please find the link below: 

<p>
  :arrow_forward:<b>For JS + Flask version: </b></br>
  ⚫<a href="https://github.com/sabrinaliuu/noto_recovery_webgis" target="_blank">Github</a>
  🔵<a href="https://sabrinaliuu.github.io/noto_recovery_webgis/" target="_blank">Website</a>
</p>
<p>
   :arrow_forward:<b>For JS + ASP.NET version: </b></br>
  ⚫<a href="https://github.com/sabrinaliuu/noto_recovery_webgis_" target="_blank">Github</a>
  🔵<a href="https://notorecovery-fmdvf9cucdbpgacw.canadacentral-01.azurewebsites.net/" target="_blank">Website</a>
</p>

## About the website
https://sabrinaliuu.github.io/noto_recovery_webgis_vue/

<img width="960" height="473" alt="image" src="https://github.com/user-attachments/assets/43e3ac22-6477-4c01-b226-d4f2b7c8cfad" />

### Introduction
This is a website that visualizes the study <i>Analyzing the Seven Critical Elements of Life Recovery Using News: A Case Study of the 2024 Noto Peninsula Earthquake</i> by Yen-Ching Liu* and Shosuke Sato.

### Interface
#### - Map and Charts
You can select <b>Recovery Element and Month</b>.
The sentiment composition for each city will show on the map in a pie chart and under the map in a bar chart.<br>
As you select the pie chart, the details for the change in sentiment will display on the left.

#### - Recognition: NLP Model Application
You can enter a recovery-related sentence in <b>Japanese</b> here.<br>
The <b>Life Recovery Elements, Cities in Noto Peninsula, and Sentiment </b> will be recognized by our NLP models and displayed.

`Note`: If you want to know more about the Recognition model and the backend by applying Flask, please visit <a href="https://github.com/sabrinaliuu/noto_recovery_webgis/tree/master/backend">this link</a>.

### Web API
If you want to use the NLP model, you can retrieve it by
```
## Python
# - Example -
import requests
r = requests.post("https://noto-recovery-webgis.onrender.com/analyze", json={"text":"能登半島地震で被災した輪島塗の工房を支援しようと東日本大震災の被災地で仮設商店として使われていたトレーラーハウス2台が、9日、輪島市の団体「輪島塗若手ネットワーク」に寄贈されました"})
print(r.json())

# Return json
{"elements":["relation to government","economic and financial situation"],"places":["輪島市"],"sentiment":"Neutral"}
```

### Skills
Use `JS/CSS/Vue/Leaflet (frontend) + Flask (backend)`

### For using Vue
Please install
```
npm install tailwindcss @tailwindcss/vite
npm install tailwindcss @tailwindcss/vite
npm install chart.js
```
And import
```
// leaflet // import to *.vue 
<script setup>
import 'leaflet/dist/leaflet.css';
import { LMap, LTileLayer, LMarker, LTooltip, LGeoJson, LControlZoom, LControl } from "@vue-leaflet/vue-leaflet";
import L from 'leaflet';
</script>

// tailwindcss // import to main.css
@import "tailwindcss";

// chart.js // import to *.vue 
<script setup>
import Chart from "chart.js/auto";
</script>
```

## About the study
In this study, NLP methods were applied to recognize the <b>recovery elements (keyword-based approach), sentiments (GPT-based approach), and locations (keyword-based approach)</b> in each paragraph. In other words, the textual data was converted into numerical data for further quantitative analysis.

Details are provided below: 
* Material: News on the Yahoo! Japan News website
* Time: August 2024 to July 2025
* Seven Critical Elements for Life Recovery: housing, social ties, townscape, physical and mental health, preparedness, relation to government, and economic and financial situation (Tatsuki and Hayashi, 2002)
* Sentiments: positive, negative, and neutral
* 12 cities in Ishikawa Prefecture on the Noto Peninsula: Suzu-shi (珠洲市), Wajima-shi (輪島市), Noto-cho (能登町), Anamizu-machi (穴水町), Shika-machi (志賀町), Nanao-shi (七尾市), Nakanoto-machi (中能登町), Hakui-shi (羽咋市), Hodatsushimizu-cho (宝達志水町), Kahoku-shi (かほく市), Uchinada-machi (内灘町), Tsubata-machi (津幡町)

This demonstrates the feasibility of using NLP methods on news data to gain insights into news media perceptions and to assess news coverage of disaster recovery.

<mark>The study will be published in the <a href="https://www.fujipress.jp/jdr/"  target="_blank">Journal of Disaster Research</a>, Vol. 21, No. 3, in June 2026, 
as <i>Yen-Ching Liu*, Shosuke Sato: Analyzing the Seven Critical Elements of Life Recovery Using News: A Case Study of the 2024 Noto Peninsula Earthquake</i>.</mark>
