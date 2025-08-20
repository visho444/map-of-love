<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1"/>
<title>Map of Love</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<style>
  html, body {
    height: 100%;
    margin: 0;
  }
  #overlay {
    position: fixed; inset: 0;
    background: rgba(255,255,255,0.8);
    display: flex; flex-direction: column;
    justify-content: center; align-items: center;
    z-index: 999;
  }
  #overlay h1 {
    font-family: Courier, monospace;
    font-size: clamp(28px, 8vw, 48px);
    margin-bottom: 20px;
  }
  #start-btn {
    background: #ff4d6d;
    color: white;
    border: none;
    padding: 14px 26px;
    border-radius: 10px;
    font-size: 18px;
    font-family: Courier, monospace;
    cursor: pointer;
    touch-action: manipulation; /* iPhone tap fix */
  }
  #map {
    height: 100vh; /* important fix for iPhone */
    width: 100%;
  }
  .heart-pin {
    font-size: 24px;
    line-height: 24px;
    text-align: center;
  }
  #info {
    position: fixed;
    bottom: -100%;
    left: 50%;
    transform: translateX(-50%);
    background: white;
    padding: 14px;
    padding-bottom: calc(env(safe-area-inset-bottom) + 14px); /* iPhone safe area */
    border-radius: 14px 14px 0 0;
    box-shadow: 0 -4px 20px rgba(0,0,0,0.2);
    max-width: 360px;
    width: 100%;
    transition: bottom 0.5s ease;
    z-index: 1000;
  }
  #info.active { bottom: 0; }
  .loc-name {
    font-weight: bold;
    margin-bottom: 8px;
    text-align: center;
    font-family: Courier, monospace;
  }
  .polaroid {
    background: white;
    padding: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.3);
    border-radius: 6px;
    text-align: center;
  }
  .polaroid img {
    width: 100%;
    border-radius: 6px;
  }
  .note {
    margin-top: 8px;
    font-size: 14px;
    color: #444;
  }
  .next-btn {
    background: #ff4d6d;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 8px;
    margin-top: 10px;
    cursor: pointer;
    touch-action: manipulation; /* iPhone tap fix */
  }
</style>
</head>
<body>

<div id="overlay">
  <h1>Map of Love</h1>
  <button id="start-btn">Start Journey</button>
</div>

<div id="map"></div>

<div id="info">
  <div class="loc-name" id="location-name"></div>
  <div class="polaroid" id="polaroid-container"></div>
  <button class="next-btn" id="next-btn">Next</button>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
const locations = [
  {
    lat: 12.0335,
    lng: 79.8522,
    name: "THE STUDY LÉCOLE INTERNATIONALE",
    photo: "https://i.ibb.co/zVjjs0wZ/IMG-5143.jpg",
    note: "WHERE OUR LIFE STARTED"
  },
  { lat: 11.9130, 
    lng: 79.8294, name:"PONDY MARINA", 
    photo:"https://i.ibb.co/xNzGydt/IMG-5255.jpg", 
    note:"PONDY BEACH DATES" },
  { lat: 12.9629,
    lng: 77.5775, name:"BANGALORE", 
    photo:"https://i.ibb.co/r2KnvbVf/IMG-5256.jpg", 
    note:"YOU AND ME FIRST TIME IN ANOTHER CITY" },
  { lat: 11.9292, lng: 79.8200, name:"PROVIDENCE MALL", 
    photo:"https://i.ibb.co/TDWBFJc2/IMG-5257.jpg", 
    note:"OUR FIRST ANNIVERSARY TOGETHER" },
  { lat: 11.992449, lng: 79.850507, name:"SUNCITY", 
    photo:"https://i.ibb.co/tTDRQJK5/IMG-5252.jpg", 
    note:"OUR SECRET SPOT" },
  { lat: 12.9929, lng: 80.2179, name:"PHOENIX MALL", 
    photo:"https://i.ibb.co/dsdFz6Zx/f1afec7a-6256-4ca5-8756-6a8a23e2f479.jpg", 
    note:"SHOPPING DATE WITH YOU" },
  { lat: 12.8980, lng: 80.2541, name:"AKKARAI BEACH", 
    photo:"https://i.ibb.co/bMvYbk9F/IMG-5117.jpg", 
    note:"EVENINGS LIKE THIS>>" },
  { lat: 13.1516667, lng: 80.24, name:"CAR DATES", 
    photo:"https://i.ibb.co/zTg2YRGr/IMG-5251.jpg", 
    note:"GUJJAAALSSS" }
];

let revealedPins = 0;

const overlay = document.getElementById('overlay');
const infoBox = document.getElementById('info');
const locationName = document.getElementById('location-name');
const polaroid = document.getElementById('polaroid-container');
const nextBtn = document.getElementById('next-btn');

const map = L.map('map').setView([20, 0], 2); // Initial world view

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '&copy; OpenStreetMap contributors'
}).addTo(map);

const heartIcon = L.divIcon({
  className: 'heart-pin',
  html: '❤️',
  iconSize: [24, 24],
  iconAnchor: [12, 12]
});

function showPin(index) {
  const loc = locations[index];
  const marker = L.marker([loc.lat, loc.lng], { icon: heartIcon }).addTo(map);
  marker.on('click', () => {
    map.setView([loc.lat, loc.lng], 16, { animate: true, duration: 0.8 });
    setTimeout(() => { showInfo(index); }, 650);
  });
  revealedPins++;
}

function showInfo(index) {
  const loc = locations[index];
  locationName.textContent = loc.name;

  polaroid.innerHTML = '';
  const img = document.createElement('img');
  img.src = loc.photo || "https://via.placeholder.com/300x200.png?text=Your+Photo+Here";
  polaroid.appendChild(img);

  const noteDiv = document.createElement('div');
  noteDiv.className = 'note';
  noteDiv.textContent = loc.note;
  polaroid.appendChild(noteDiv);

  infoBox.classList.add('active');

  nextBtn.textContent = index === locations.length - 1 ?
    "Let's continue this journey for a lifetime ❤️" : "Next";

  nextBtn.onclick = () => {
    infoBox.classList.remove('active');
    if (index === locations.length - 1) {
      window.location.href = "https://444you.my.canva.site/";
    } else {
      map.setView([20.5937, 78.9629], 4, { animate: true, duration: 0.8 });
      setTimeout(() => {
        showPin(revealedPins);
      }, 1500);
    }
  };
}

document.getElementById('start-btn').addEventListener('click', () => {
  overlay.style.display = 'none';
  showPin(revealedPins);
});
</script>
</body>
</html>
