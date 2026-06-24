<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>主題図：横浜市 朝の人口移動</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet-polylinedecorator/1.6.0/leaflet.polylineDecorator.min.js"></script>
<style>
* { margin:0; padding:0; box-sizing:border-box; }
body { font-family:"Hiragino Mincho ProN","Yu Mincho",serif; background:#1a1a2e; }

.page-wrap {
  width:210mm; min-height:297mm; margin:20px auto;
  background:#faf8f2;
  box-shadow:0 4px 32px rgba(0,0,0,0.35);
  display:flex; flex-direction:column;
  padding:10mm 10mm 10mm 10mm;
}

.header {
  text-align:center;
  border-bottom:2.5px solid #1a1a2e;
  padding-bottom:6px; margin-bottom:8px;
}
.header h1 { font-size:18px; letter-spacing:5px; color:#1a1a2e; }
.header p  { font-size:10px; color:#555; margin-top:3px; letter-spacing:1.5px; }

#map {
  width:100%; height:185mm;
  border:1.5px solid #666;
  z-index:1;
}

/* 矢印SVGオーバーレイ */
#arrow-svg {
  position:absolute;
  top:0; left:0;
  width:100%; height:100%;
  pointer-events:none;
  z-index:500;
}

.map-container { position:relative; width:100%; height:185mm; }

/* 凡例 */
.legend-box {
  margin-top:8px;
  border:1.2px solid #aaa;
  padding:7px 12px;
  background:#fdfcf7;
  display:flex; gap:18px; flex-wrap:wrap;
}
.leg-title { font-size:10px; font-weight:bold; margin-bottom:4px; border-bottom:1px solid #ddd; padding-bottom:2px; color:#1a1a2e; }
.leg-row { display:flex; align-items:center; gap:7px; font-size:9px; margin-bottom:3px; }

.note {
  margin-top:7px; font-size:8.5px; color:#555;
  line-height:1.7; border-left:3px solid #999; padding-left:8px;
}
.footer-line {
  margin-top:8px; font-size:9px; color:#888;
  border-top:1px solid #ddd; padding-top:5px;
  display:flex; justify-content:space-between;
}

/* Leafletのポップアップ */
.leaflet-popup-content { font-family:"Hiragino Mincho ProN","Yu Mincho",serif; font-size:12px; }

/* 矢印ラベル */
.flow-label {
  background:rgba(255,255,255,0.88);
  border:1px solid #999;
  border-radius:3px;
  padding:1px 5px;
  font-size:11px;
  font-weight:bold;
  white-space:nowrap;
  font-family:"Hiragino Mincho ProN","Yu Mincho",serif;
}

@media print {
  body { background:white; }
  .page-wrap { margin:0; box-shadow:none; }
}
</style>
</head>
<body>
<div class="page-wrap">
  <div class="header">
    <h1>主　題　図</h1>
    <p>横浜市中心部　朝の人口移動（通勤・通学）　7:00〜9:00</p>
  </div>

  <div class="map-container">
    <div id="map"></div>
  </div>

  <!-- 凡例 -->
  <div class="legend-box">
    <div style="flex:1.5">
      <div class="leg-title">矢印の太さ ＝ 移動人口規模（推定）</div>
      <div class="leg-row">
        <svg width="70" height="20"><path d="M5 10 L65 10" fill="none" stroke="#1a5276" stroke-width="18" stroke-linecap="round" opacity="0.75"/><polygon points="58,4 70,10 58,16" fill="#1a5276"/></svg>
        <span>約２万人以上（最大）</span>
      </div>
      <div class="leg-row">
        <svg width="70" height="16"><path d="M5 8 L60 8" fill="none" stroke="#c0392b" stroke-width="13" stroke-linecap="round" opacity="0.72"/><polygon points="54,3 68,8 54,13" fill="#c0392b"/></svg>
        <span>約１〜１．５万人</span>
      </div>
      <div class="leg-row">
        <svg width="70" height="13"><path d="M5 6 L58 6" fill="none" stroke="#1e8449" stroke-width="9" stroke-linecap="round" opacity="0.70"/><polygon points="52,2 66,6 52,10" fill="#1e8449"/></svg>
        <span>約７千〜１万人</span>
      </div>
      <div class="leg-row">
        <svg width="70" height="11"><path d="M5 5 L57 5" fill="none" stroke="#7d3c98" stroke-width="6" stroke-linecap="round" opacity="0.68"/><polygon points="51,1 65,5 51,9" fill="#7d3c98"/></svg>
        <span>約５千〜７千人</span>
      </div>
      <div class="leg-row">
        <svg width="70" height="9"><path d="M5 4 L56 4" fill="none" stroke="#d35400" stroke-width="4" stroke-linecap="round" opacity="0.65"/><polygon points="50,1 64,4 50,7" fill="#d35400"/></svg>
        <span>約３千〜５千人</span>
      </div>
    </div>
    <div style="flex:1">
      <div class="leg-title">矢印の色 ＝ 目的地方向</div>
      <div class="leg-row"><svg width="14" height="14"><rect width="14" height="14" rx="2" fill="#1a5276"/></svg><span>→ 横浜駅・みなとみらい</span></div>
      <div class="leg-row"><svg width="14" height="14"><rect width="14" height="14" rx="2" fill="#c0392b"/></svg><span>→ 桜木町・関内</span></div>
      <div class="leg-row"><svg width="14" height="14"><rect width="14" height="14" rx="2" fill="#1e8449"/></svg><span>→ 中華街・元町</span></div>
      <div class="leg-row"><svg width="14" height="14"><rect width="14" height="14" rx="2" fill="#7d3c98"/></svg><span>→ 港北NT・センター北</span></div>
      <div class="leg-row"><svg width="14" height="14"><rect width="14" height="14" rx="2" fill="#d35400"/></svg><span>→ 戸塚方面</span></div>
    </div>
    <div style="flex:1">
      <div class="leg-title">出発地・目的地</div>
      <div class="leg-row"><svg width="14" height="14"><circle cx="7" cy="7" r="6" fill="#c0392b" stroke="white" stroke-width="1.5"/></svg><span>主要出発地</span></div>
      <div class="leg-row"><svg width="14" height="14"><circle cx="7" cy="7" r="6" fill="#1a5276" stroke="white" stroke-width="1.5"/></svg><span>主要目的地（駅・エリア）</span></div>
      <div style="font-size:9px; margin-top:4px; color:#555;">
        対象：通勤・通学者<br>
        時間帯：平日 7:00〜9:00<br>
        出典：国勢調査・流動人口統計（推定）
      </div>
    </div>
  </div>

  <div class="note">
    ※ 矢印の太さと数値は移動人口の相対的な規模を示す推定値です。実測データではなく統計資料をもとにした模式的な表現です。<br>
    ※ 地図タイル：国土地理院（地理院タイル）を使用
  </div>

  <div class="footer-line">
    <span>主題図　横浜市中心部　朝の人口移動　　氏名：＿＿＿＿＿＿＿＿＿＿</span>
    <span>作成日：＿＿年＿＿月＿＿日</span>
  </div>
</div>

<script>
// ===== 地図初期化（国土地理院タイル） =====
const map = L.map('map', {
  center: [35.435, 139.635],
  zoom: 13,
  zoomControl: true,
  attributionControl: true
});

// 国土地理院タイル（標準地図）
L.tileLayer('https://cyberjapandata.gsi.go.jp/xyz/std/{z}/{x}/{y}.png', {
  attribution: '<a href="https://maps.gsi.go.jp/development/ichiran.html">国土地理院</a>',
  maxZoom: 18
}).addTo(map);

// ===== 場所の定義 =====
const places = {
  // 目的地
  yokohama:   [35.4658, 139.6225],  // 横浜駅
  sakuragicho:[35.4502, 139.6317],  // 桜木町
  kannai:     [35.4436, 139.6384],  // 関内
  chinatown:  [35.4436, 139.6490],  // 中華街
  minatomirai:[35.4574, 139.6367],  // みなとみらい
  // 出発地
  minamiota:  [35.4162, 139.6178],  // 南太田
  makita:     [35.4078, 139.6180],  // 蒔田
  idogaya:    [35.4039, 139.6147],  // 井土ヶ谷
  gumyoji:    [35.3968, 139.6085],  // 弘明寺
  kouhoku:    [35.5430, 139.5980],  // 港北NT（センター北）
  totsuka:    [35.3980, 139.5905],  // 戸塚
  isezaki:    [35.4453, 139.6422],  // 伊勢佐木長者町
  koganecho:  [35.4299, 139.6274],  // 黄金町
};

// ===== マーカースタイル =====
function makeIcon(color, size=12) {
  return L.divIcon({
    className: '',
    html: `<div style="
      width:${size}px; height:${size}px;
      background:${color};
      border:2px solid white;
      border-radius:50%;
      box-shadow:0 1px 4px rgba(0,0,0,0.4);
    "></div>`,
    iconSize: [size, size],
    iconAnchor: [size/2, size/2]
  });
}

// 目的地マーカー（青）
[
  [places.yokohama,    '横浜駅'],
  [places.sakuragicho, '桜木町'],
  [places.kannai,      '関内'],
  [places.chinatown,   '中華街・元町'],
  [places.minatomirai, 'みなとみらい'],
].forEach(([latlng, name]) => {
  L.marker(latlng, {icon: makeIcon('#1a5276', 14)})
    .bindTooltip(`<b style="color:#1a5276">${name}</b>（目的地）`, {permanent:true, direction:'top', className:'flow-label'})
    .addTo(map);
});

// 出発地マーカー（赤）
[
  [places.minamiota, '南太田'],
  [places.makita,    '蒔田'],
  [places.idogaya,   '井土ヶ谷'],
  [places.gumyoji,   '弘明寺'],
  [places.koganecho, '黄金町'],
].forEach(([latlng, name]) => {
  L.marker(latlng, {icon: makeIcon('#c0392b', 12)})
    .bindTooltip(`<b style="color:#c0392b">${name}</b>`, {permanent:true, direction:'bottom', className:'flow-label'})
    .addTo(map);
});

// 遠距離出発地（緑・紫）
L.marker(places.kouhoku, {icon: makeIcon('#1e8449', 14)})
  .bindTooltip('<b style="color:#1e8449">港北NT・センター北</b>（出発地）', {permanent:true, direction:'bottom', className:'flow-label'})
  .addTo(map);
L.marker(places.totsuka, {icon: makeIcon('#7d3c98', 12)})
  .bindTooltip('<b style="color:#7d3c98">戸塚</b>（出発地）', {permanent:true, direction:'right', className:'flow-label'})
  .addTo(map);

// ===== 流動量ラベル付き矢印を描く関数 =====
function drawFlow(from, to, color, weight, label, curvature=0.3) {
  // 曲線ポリライン（中間点をずらして曲げる）
  const lat1=from[0], lng1=from[1];
  const lat2=to[0],   lng2=to[1];
  const midLat = (lat1+lat2)/2 + (lng2-lng1)*curvature;
  const midLng = (lng1+lng2)/2 - (lat2-lat1)*curvature;

  // 矢印ポリライン
  const pl = L.polyline([[lat1,lng1],[midLat,midLng],[lat2,lng2]], {
    color: color,
    weight: weight,
    opacity: 0.72,
    smoothFactor: 1,
    lineCap: 'round',
    lineJoin: 'round'
  }).addTo(map);

  // 矢頭（終点付近に小さい三角形）
  const angle = Math.atan2(lat2-midLat, lng2-midLng) * 180 / Math.PI;
  const arrowHead = L.polylineDecorator(pl, {
    patterns: [{
      offset: '95%',
      repeat: 0,
      symbol: L.Symbol.arrowHead({
        pixelSize: weight * 2.2,
        polygon: true,
        pathOptions: { color: color, fillColor: color, fillOpacity: 0.9, weight: 1 }
      })
    }]
  });

  // ラベル（中間点付近）
  if (label) {
    L.marker([midLat, midLng], {
      icon: L.divIcon({
        className: '',
        html: `<div style="
          background:rgba(255,255,255,0.90);
          border:1.5px solid ${color};
          border-radius:4px;
          padding:1px 6px;
          font-size:12px;
          font-weight:bold;
          color:${color};
          white-space:nowrap;
          font-family:'Hiragino Mincho ProN','Yu Mincho',serif;
          box-shadow:0 1px 3px rgba(0,0,0,0.2);
        ">${label}</div>`,
        iconAnchor: [30, 10]
      })
    }).addTo(map);
  }

  return pl;
}

// ===== Leaflet.PolylineDecorator（矢頭用）=====
// CDNから読み込み

function drawAllFlows() {
  // ===== 主要流動を描画 =====

  // 南太田 → 横浜駅（最大 約2万人）
  drawFlow(places.minamiota, places.yokohama, '#1a5276', 20, '約２万人', 0.25);

  // 南太田 → 桜木町（大 約1.5万人）
  drawFlow(places.minamiota, places.sakuragicho, '#c0392b', 15, '約1.5万人', -0.2);

  // 南太田 → 中華街（大 約1.2万人）
  drawFlow(places.minamiota, places.chinatown, '#1e8449', 12, '約1.2万人', -0.15);

  // 黄金町 → 横浜駅（中 約8千人）
  drawFlow(places.koganecho, places.yokohama, '#1a5276', 9, '約8千人', 0.3);

  // 黄金町 → 桜木町（中 約6千人）
  drawFlow(places.koganecho, places.sakuragicho, '#c0392b', 7, '約6千人', -0.15);

  // 蒔田・井土ヶ谷 → 桜木町（中 約8千人）
  drawFlow(places.idogaya, places.sakuragicho, '#c0392b', 9, '約8千人', -0.2);

  // 蒔田 → 関内（中 約6千人）
  drawFlow(places.makita, places.kannai, '#1e8449', 7, '約6千人', -0.1);

  // 弘明寺 → 横浜駅（小中 約5千人）
  drawFlow(places.gumyoji, places.yokohama, '#1a5276', 6, '約5千人', 0.25);

  // 港北NT → 横浜駅（大 約1万人）
  drawFlow(places.kouhoku, places.yokohama, '#7d3c98', 11, '約1万人', 0.2);

  // 港北NT → みなとみらい（中 約7千人）
  drawFlow(places.kouhoku, places.minatomirai, '#7d3c98', 8, '約7千人', -0.15);

  // 戸塚 → 横浜駅（中 約7千人）
  drawFlow(places.totsuka, places.yokohama, '#d35400', 8, '約7千人', 0.2);

  // 戸塚 → 桜木町（小中 約4千人）
  drawFlow(places.totsuka, places.sakuragicho, '#d35400', 5, '約4千人', -0.15);
}


drawAllFlows();

</script>
</body>
</html>