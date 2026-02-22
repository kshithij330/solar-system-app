# AR Solar System Explorer — Complete Build Specification

## Project Overview

Build a **single-file web application** (`index.html`) that uses the device camera as a background, detects hand gestures via **MediaPipe Hands**, and renders an interactive 3D solar system using **Three.js**. The app runs entirely in the browser — no backend, no build step, no npm. All libraries are loaded via CDN.

The experience is a **void space** — no camera feed visible to the user, just pure black/deep space with stars. The hand is tracked invisibly in the background. Users navigate through the solar system using hand gestures.

---

## Tech Stack

| Library | Version | CDN URL |
|---|---|---|
| Three.js | r128 | `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js` |
| MediaPipe Hands | latest | `https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js` |
| MediaPipe Camera Utils | latest | `https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js` |
| MediaPipe Drawing Utils | latest | `https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js` |
| GSAP (animations) | 3.x | `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js` |

> **Important:** Do NOT use THREE.OrbitControls, THREE.CapsuleGeometry, or any Three.js add-ons not available on the r128 CDN. Use only the core Three.js library.

---

## File Structure

The entire app is **one file**: `index.html`

```
index.html
  ├── <head> — CDN scripts, CSS styles
  ├── <body>
  │     ├── #canvas-container   — Three.js renderer fills this
  │     ├── #video-element      — Hidden camera feed for MediaPipe
  │     ├── #ui-overlay         — HTML UI layer (info panels, HUD)
  │     └── <script>            — All JS in one block
  └── </html>
```

---

## Visual Design & Aesthetic

### Color Palette
```
Background:       #000005  (near-black deep space)
Star field:       White dots, varying opacity 0.3–1.0
Accent/glow:      #4fc3f7  (ice blue)
UI panels:        rgba(5, 10, 30, 0.85) with border #4fc3f7
Text primary:     #e0f7fa
Text secondary:   #80deea
Highlight:        #ffd54f  (gold, for selected planet name)
Danger/close:     #ef5350
```

### Font
Use Google Fonts: `Exo 2` (sci-fi feel, clean readability).
```html
<link href="https://fonts.googleapis.com/css2?family=Exo+2:wght@300;400;600;700&display=swap" rel="stylesheet">
```

### Star Field
Generate 3000 stars as `THREE.Points` with a `THREE.BufferGeometry`. Positions are random within a sphere of radius 800. Use `THREE.PointsMaterial` with `size: 0.8`, `sizeAttenuation: true`, `color: 0xffffff`, `transparent: true`, `opacity: 0.8`. Stars should slowly rotate on the Y axis at 0.00005 rad/frame.

---

## Solar System Data

Hardcode the following planet data. All sizes are **relative/artistic** — not to scale — chosen to be visually appealing at the orbit distances specified.

```javascript
const SOLAR_DATA = {
  sun: {
    name: "The Sun",
    radius: 6,
    color: 0xFDB813,
    emissive: 0xFF6600,
    emissiveIntensity: 1.2,
    rotationSpeed: 0.002,
    info: {
      type: "G-type Main-Sequence Star",
      age: "4.6 Billion Years",
      diameter: "1,392,700 km",
      mass: "1.989 × 10³⁰ kg",
      surfaceTemp: "5,778 K",
      coreTemp: "15,000,000 K",
      distanceFromEarth: "149.6 million km (1 AU)",
      funFact: "The Sun contains 99.86% of the total mass of the Solar System. It would take 1.3 million Earths to fill the Sun."
    }
  },
  planets: [
    {
      name: "Mercury",
      radius: 0.8,
      color: 0xB5B5B5,
      emissive: 0x222222,
      orbitRadius: 14,
      orbitSpeed: 0.008,
      rotationSpeed: 0.003,
      tilt: 0.034,
      info: {
        type: "Terrestrial Planet",
        diameter: "4,879 km",
        mass: "3.3 × 10²³ kg",
        distanceFromSun: "57.9 million km",
        orbitalPeriod: "88 Earth days",
        surfaceTemp: "-180°C to 430°C",
        moons: "0",
        funFact: "Despite being closest to the Sun, Mercury is not the hottest planet. It has no atmosphere to trap heat, causing extreme temperature swings."
      }
    },
    {
      name: "Venus",
      radius: 1.4,
      color: 0xE8C47A,
      emissive: 0x8B6914,
      orbitRadius: 20,
      orbitSpeed: 0.006,
      rotationSpeed: -0.001,
      tilt: 3.096,
      info: {
        type: "Terrestrial Planet",
        diameter: "12,104 km",
        mass: "4.87 × 10²⁴ kg",
        distanceFromSun: "108.2 million km",
        orbitalPeriod: "225 Earth days",
        surfaceTemp: "465°C (average)",
        moons: "0",
        funFact: "Venus rotates backwards compared to most planets, and a day on Venus is longer than its year. It is the hottest planet in our solar system."
      }
    },
    {
      name: "Earth",
      radius: 1.5,
      color: 0x2E86AB,
      emissive: 0x1a4a5e,
      orbitRadius: 28,
      orbitSpeed: 0.005,
      rotationSpeed: 0.01,
      tilt: 0.4101,
      info: {
        type: "Terrestrial Planet",
        diameter: "12,742 km",
        mass: "5.97 × 10²⁴ kg",
        distanceFromSun: "149.6 million km (1 AU)",
        orbitalPeriod: "365.25 days",
        surfaceTemp: "-88°C to 58°C",
        moons: "1 (The Moon)",
        funFact: "Earth is the only known planet to harbor life. Its magnetic field protects life from harmful solar radiation. Over 70% of its surface is covered in water."
      }
    },
    {
      name: "Mars",
      radius: 1.1,
      color: 0xC1440E,
      emissive: 0x6b1a04,
      orbitRadius: 37,
      orbitSpeed: 0.004,
      rotationSpeed: 0.009,
      tilt: 0.4396,
      info: {
        type: "Terrestrial Planet",
        diameter: "6,779 km",
        mass: "6.39 × 10²³ kg",
        distanceFromSun: "227.9 million km",
        orbitalPeriod: "687 Earth days",
        surfaceTemp: "-153°C to 20°C",
        moons: "2 (Phobos, Deimos)",
        funFact: "Mars hosts Olympus Mons, the largest volcano in the solar system at 21 km high — nearly three times the height of Mount Everest."
      }
    },
    {
      name: "Jupiter",
      radius: 3.5,
      color: 0xC88B3A,
      emissive: 0x5a3a0a,
      orbitRadius: 52,
      orbitSpeed: 0.002,
      rotationSpeed: 0.02,
      tilt: 0.0546,
      info: {
        type: "Gas Giant",
        diameter: "139,820 km",
        mass: "1.898 × 10²⁷ kg",
        distanceFromSun: "778.5 million km",
        orbitalPeriod: "11.86 Earth years",
        surfaceTemp: "-108°C (cloud tops)",
        moons: "95 confirmed",
        funFact: "Jupiter's Great Red Spot is a storm that has been raging for at least 350 years. It is so large that three Earths could fit inside it."
      }
    },
    {
      name: "Saturn",
      radius: 3.0,
      color: 0xE4D191,
      emissive: 0x7a6530,
      orbitRadius: 68,
      orbitSpeed: 0.0015,
      rotationSpeed: 0.018,
      tilt: 0.4665,
      hasRings: true,
      info: {
        type: "Gas Giant",
        diameter: "116,460 km",
        mass: "5.68 × 10²⁶ kg",
        distanceFromSun: "1.43 billion km",
        orbitalPeriod: "29.46 Earth years",
        surfaceTemp: "-139°C (cloud tops)",
        moons: "146 confirmed",
        funFact: "Saturn is the least dense planet in the Solar System — it would float on water. Its iconic rings are made of ice and rock, spanning up to 282,000 km wide."
      }
    },
    {
      name: "Uranus",
      radius: 2.2,
      color: 0x7DE8E8,
      emissive: 0x2a7a7a,
      orbitRadius: 82,
      orbitSpeed: 0.001,
      rotationSpeed: -0.012,
      tilt: 1.7064,
      info: {
        type: "Ice Giant",
        diameter: "50,724 km",
        mass: "8.68 × 10²⁵ kg",
        distanceFromSun: "2.87 billion km",
        orbitalPeriod: "84 Earth years",
        surfaceTemp: "-197°C (average)",
        moons: "28 confirmed",
        funFact: "Uranus rotates on its side with an axial tilt of 98°, likely caused by a massive collision long ago. Its poles receive more sunlight than its equator."
      }
    },
    {
      name: "Neptune",
      radius: 2.0,
      color: 0x3F54BA,
      emissive: 0x1a2060,
      orbitRadius: 96,
      orbitSpeed: 0.0008,
      rotationSpeed: 0.014,
      tilt: 0.4944,
      info: {
        type: "Ice Giant",
        diameter: "49,244 km",
        mass: "1.02 × 10²⁶ kg",
        distanceFromSun: "4.5 billion km",
        orbitalPeriod: "165 Earth years",
        surfaceTemp: "-201°C (average)",
        moons: "16 confirmed",
        funFact: "Neptune has the strongest winds in the Solar System, reaching speeds of 2,100 km/h. Its largest moon Triton orbits backwards and is slowly spiraling inward."
      }
    }
  ]
};
```

---

## Application State Machine

The app has a clear state machine. All transitions must use smooth animations.

```
STATE = {
  OVERVIEW,       // Full solar system visible, gently rotating
  FOCUSED_SUN,    // Camera zoomed to Sun, hand can rotate it
  FOCUSED_PLANET, // Camera zoomed to a specific planet
  INFO_OPEN       // Info panel visible for current focus
}

currentFocusIndex = -1  // -1 = Sun, 0 = Mercury, 1 = Venus, ... 7 = Neptune
```

### State Transitions
```
OVERVIEW       → FOCUSED_SUN      : User performs PINCH gesture
FOCUSED_SUN    → INFO_OPEN        : User performs CLICK gesture (info panel opens)
INFO_OPEN      → FOCUSED_SUN      : User performs CLICK gesture again (info panel closes)
FOCUSED_SUN    → FOCUSED_PLANET   : User performs PINCH gesture (advances to Mercury)
FOCUSED_PLANET → FOCUSED_PLANET   : User performs PINCH gesture (advances to next planet)
FOCUSED_PLANET → INFO_OPEN        : User performs CLICK gesture
INFO_OPEN      → FOCUSED_PLANET   : User performs CLICK gesture (closes info)
```

After Neptune, the next PINCH returns to OVERVIEW with a full zoom-out animation.

---

## Gesture Detection

### MediaPipe Setup

```javascript
const hands = new Hands({
  locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
});

hands.setOptions({
  maxNumHands: 1,
  modelComplexity: 1,
  minDetectionConfidence: 0.75,
  minTrackingConfidence: 0.75
});

hands.onResults(onHandResults);

const camera = new Camera(videoElement, {
  onFrame: async () => { await hands.send({ image: videoElement }); },
  width: 640,
  height: 480
});
camera.start();
```

### Landmark Reference
MediaPipe returns 21 hand landmarks (0–20).

Key landmarks:
- `4` = Thumb tip
- `8` = Index finger tip
- `5` = Index finger MCP (knuckle)
- `6` = Index finger PIP
- `12` = Middle finger tip
- `0` = Wrist

### Gesture: PINCH

Pinch is detected when the distance between landmark 4 (thumb tip) and landmark 8 (index tip) falls below a threshold.

```javascript
function getPinchDistance(landmarks) {
  const thumb = landmarks[4];
  const index = landmarks[8];
  const dx = thumb.x - index.x;
  const dy = thumb.y - index.y;
  return Math.sqrt(dx * dx + dy * dy);
}

// Normalized distance (landmarks are in 0-1 range)
// Threshold: 0.06 = pinched, > 0.10 = released
```

Use a **debounce/state machine** for pinch detection to avoid accidental triggers:
```javascript
let pinchState = 'open';          // 'open' | 'closing' | 'pinched'
let pinchCooldown = false;        // 800ms cooldown after trigger

function detectPinch(landmarks) {
  const dist = getPinchDistance(landmarks);
  if (dist < 0.06 && pinchState === 'open' && !pinchCooldown) {
    pinchState = 'pinched';
    pinchCooldown = true;
    onPinch(); // trigger
    setTimeout(() => {
      pinchCooldown = false;
      pinchState = 'open';
    }, 800);
  }
  if (dist > 0.10) pinchState = 'open';
}
```

### Gesture: CLICK (Air Tap)

Detect a "click" as a quick downward then upward motion of the index finger tip (landmark 8).

```javascript
// Track last N positions of index tip Y
const indexYHistory = [];
const HISTORY_SIZE = 10;

function detectClick(landmarks) {
  const indexTip = landmarks[8];
  indexYHistory.push(indexTip.y);
  if (indexYHistory.length > HISTORY_SIZE) indexYHistory.shift();
  
  if (indexYHistory.length < HISTORY_SIZE) return;
  
  // Look for dip: goes down (y increases) then back up
  const mid = Math.floor(HISTORY_SIZE / 2);
  const firstHalf = indexYHistory.slice(0, mid);
  const secondHalf = indexYHistory.slice(mid);
  
  const maxFirst = Math.max(...firstHalf);
  const minSecond = Math.min(...secondHalf);
  const firstMean = firstHalf.reduce((a,b)=>a+b,0)/firstHalf.length;
  const lastMean = secondHalf.reduce((a,b)=>a+b,0)/secondHalf.length;
  
  const dipAmount = maxFirst - minSecond;
  
  if (dipAmount > 0.04 && firstMean < lastMean && !clickCooldown) {
    clickCooldown = true;
    onClickGesture();
    setTimeout(() => { clickCooldown = false; }, 1000);
  }
}
```

### Gesture: ROTATE (Hand Pan)

When in FOCUSED_SUN or FOCUSED_PLANET state, track the palm center (average of landmarks 0, 5, 9, 13, 17) and map movement delta to rotation of the focused object.

```javascript
let lastPalmX = null;
let lastPalmY = null;

function getPalmCenter(landmarks) {
  const pts = [0, 5, 9, 13, 17].map(i => landmarks[i]);
  return {
    x: pts.reduce((s,p)=>s+p.x,0)/5,
    y: pts.reduce((s,p)=>s+p.y,0)/5
  };
}

function detectRotation(landmarks) {
  if (appState !== 'FOCUSED_SUN' && appState !== 'FOCUSED_PLANET') return;
  
  const palm = getPalmCenter(landmarks);
  if (lastPalmX !== null) {
    const dx = palm.x - lastPalmX;
    const dy = palm.y - lastPalmY;
    
    // Apply rotation to focused object
    focusedMesh.rotation.y += dx * 4.0;
    focusedMesh.rotation.x += dy * 2.0;
  }
  lastPalmX = palm.x;
  lastPalmY = palm.y;
}
```

Reset `lastPalmX/Y` to null when no hand is detected.

---

## Three.js Scene Setup

### Renderer
```javascript
const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
renderer.shadowMap.enabled = false;
renderer.setClearColor(0x000005, 1);
document.getElementById('canvas-container').appendChild(renderer.domElement);
```

### Camera
```javascript
const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 2000);
camera.position.set(0, 80, 160);
camera.lookAt(0, 0, 0);
```

### Lighting
```javascript
// Ambient — dim space ambience
const ambientLight = new THREE.AmbientLight(0x111133, 0.5);
scene.add(ambientLight);

// Point light from Sun position — illuminates planets
const sunLight = new THREE.PointLight(0xFDB813, 3.0, 500);
sunLight.position.set(0, 0, 0);
scene.add(sunLight);

// Subtle fill from behind camera
const fillLight = new THREE.DirectionalLight(0x334466, 0.3);
fillLight.position.set(0, 100, 200);
scene.add(fillLight);
```

---

## Object Construction

### Sun

```javascript
function createSun() {
  const geo = new THREE.SphereGeometry(SOLAR_DATA.sun.radius, 64, 64);
  const mat = new THREE.MeshStandardMaterial({
    color: SOLAR_DATA.sun.color,
    emissive: SOLAR_DATA.sun.emissive,
    emissiveIntensity: SOLAR_DATA.sun.emissiveIntensity,
    roughness: 0.8,
    metalness: 0.0
  });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.name = 'sun';
  scene.add(mesh);
  
  // Glow effect: large transparent sphere
  const glowGeo = new THREE.SphereGeometry(SOLAR_DATA.sun.radius * 1.4, 32, 32);
  const glowMat = new THREE.MeshBasicMaterial({
    color: 0xFF8800,
    transparent: true,
    opacity: 0.08,
    side: THREE.BackSide
  });
  const glow = new THREE.Mesh(glowGeo, glowMat);
  mesh.add(glow);
  
  // Second larger glow
  const glow2Geo = new THREE.SphereGeometry(SOLAR_DATA.sun.radius * 2.0, 32, 32);
  const glow2Mat = new THREE.MeshBasicMaterial({
    color: 0xFF4400,
    transparent: true,
    opacity: 0.03,
    side: THREE.BackSide
  });
  mesh.add(new THREE.Mesh(glow2Geo, glow2Mat));
  
  return mesh;
}
```

### Orbit Rings

For each planet, create a visible orbit ring (torus geometry):

```javascript
function createOrbitRing(radius) {
  const geo = new THREE.TorusGeometry(radius, 0.05, 8, 180);
  const mat = new THREE.MeshBasicMaterial({
    color: 0x334466,
    transparent: true,
    opacity: 0.35
  });
  const ring = new THREE.Mesh(geo, mat);
  ring.rotation.x = Math.PI / 2;
  scene.add(ring);
  return ring;
}
```

### Planets

```javascript
function createPlanet(data, index) {
  // Pivot object for orbit
  const pivot = new THREE.Object3D();
  pivot.name = `pivot_${data.name}`;
  scene.add(pivot);
  
  // Planet sphere
  const geo = new THREE.SphereGeometry(data.radius, 48, 48);
  const mat = new THREE.MeshStandardMaterial({
    color: data.color,
    emissive: data.emissive,
    emissiveIntensity: 0.2,
    roughness: 0.85,
    metalness: 0.1
  });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.position.x = data.orbitRadius;
  mesh.rotation.z = data.tilt;
  mesh.name = data.name;
  pivot.add(mesh);
  
  // Spread starting angles
  pivot.rotation.y = (index / SOLAR_DATA.planets.length) * Math.PI * 2;
  
  // Saturn's rings
  if (data.hasRings) {
    const ringGeo = new THREE.RingGeometry(data.radius * 1.5, data.radius * 2.6, 80);
    const ringMat = new THREE.MeshBasicMaterial({
      color: 0xC2A55A,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.75
    });
    const ringMesh = new THREE.Mesh(ringGeo, ringMat);
    ringMesh.rotation.x = Math.PI / 2.5;
    mesh.add(ringMesh);
  }
  
  return { pivot, mesh };
}
```

### Asteroid Belt (decorative)

Add a subtle asteroid belt between Mars and Jupiter:

```javascript
function createAsteroidBelt() {
  const count = 800;
  const positions = new Float32Array(count * 3);
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const radius = 43 + Math.random() * 5; // between Mars (37) and Jupiter (52)
    const y = (Math.random() - 0.5) * 1.5;
    positions[i * 3] = Math.cos(angle) * radius;
    positions[i * 3 + 1] = y;
    positions[i * 3 + 2] = Math.sin(angle) * radius;
  }
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  const mat = new THREE.PointsMaterial({
    color: 0x886644,
    size: 0.25,
    transparent: true,
    opacity: 0.6
  });
  scene.add(new THREE.Points(geo, mat));
}
```

---

## Camera Animation System

All camera movements use GSAP tweens for smooth cinematic transitions.

### Camera Target Tracking

Maintain camera state:
```javascript
const cameraState = {
  targetPosition: new THREE.Vector3(0, 80, 160),
  targetLookAt: new THREE.Vector3(0, 0, 0),
  currentLookAt: new THREE.Vector3(0, 0, 0),
  lerpSpeed: 0.05
};
```

In `animate()` loop, lerp toward targets:
```javascript
camera.position.lerp(cameraState.targetPosition, cameraState.lerpSpeed);
cameraState.currentLookAt.lerp(cameraState.targetLookAt, cameraState.lerpSpeed);
camera.lookAt(cameraState.currentLookAt);
```

### Transition to OVERVIEW
```javascript
function transitionToOverview() {
  appState = 'OVERVIEW';
  currentFocusIndex = -1;
  
  gsap.to(cameraState.targetPosition, {
    x: 0, y: 80, z: 160,
    duration: 2.5,
    ease: "power3.inOut"
  });
  gsap.to(cameraState.targetLookAt, {
    x: 0, y: 0, z: 0,
    duration: 2.5,
    ease: "power3.inOut"
  });
  
  // Re-enable all planet orbits
  planetsAnimating = true;
  hideInfoPanel();
  showHUD('overview');
}
```

### Transition to Focus (Sun or Planet)
```javascript
function transitionToFocus(targetMesh) {
  const worldPos = new THREE.Vector3();
  targetMesh.getWorldPosition(worldPos);
  
  const radius = targetMesh.geometry.parameters.radius || 2;
  const zoomDist = radius * 7 + 8;
  
  gsap.to(cameraState.targetPosition, {
    x: worldPos.x,
    y: worldPos.y + radius * 1.5,
    z: worldPos.z + zoomDist,
    duration: 1.8,
    ease: "power3.inOut"
  });
  gsap.to(cameraState.targetLookAt, {
    x: worldPos.x,
    y: worldPos.y,
    z: worldPos.z,
    duration: 1.8,
    ease: "power3.inOut"
  });
}
```

---

## Animation Loop

```javascript
let lastTime = 0;

function animate(time) {
  requestAnimationFrame(animate);
  
  const delta = (time - lastTime) / 1000;
  lastTime = time;
  
  // Star field slow rotation
  starField.rotation.y += 0.00005;
  
  if (planetsAnimating) {
    // Rotate planets on pivot (orbit)
    SOLAR_DATA.planets.forEach((data, i) => {
      planets[i].pivot.rotation.y += data.orbitSpeed;
      // Self-rotation
      planets[i].mesh.rotation.y += data.rotationSpeed;
    });
    // Sun self-rotation
    sunMesh.rotation.y += SOLAR_DATA.sun.rotationSpeed;
  }
  
  // If focused, keep camera following the focused object if it's a planet
  if (appState === 'FOCUSED_PLANET' || appState === 'INFO_OPEN') {
    if (currentFocusIndex >= 0) {
      const planetMesh = planets[currentFocusIndex].mesh;
      const worldPos = new THREE.Vector3();
      planetMesh.getWorldPosition(worldPos);
      const radius = SOLAR_DATA.planets[currentFocusIndex].radius;
      const zoomDist = radius * 7 + 8;
      
      cameraState.targetPosition.set(worldPos.x, worldPos.y + radius * 1.5, worldPos.z + zoomDist);
      cameraState.targetLookAt.copy(worldPos);
    }
  }
  
  // Lerp camera
  camera.position.lerp(cameraState.targetPosition, 0.06);
  cameraState.currentLookAt.lerp(cameraState.targetLookAt, 0.06);
  camera.lookAt(cameraState.currentLookAt);
  
  renderer.render(scene, camera);
}

requestAnimationFrame(animate);
```

---

## Hand Results Handler

```javascript
function onHandResults(results) {
  if (!results.multiHandLandmarks || results.multiHandLandmarks.length === 0) {
    lastPalmX = null;
    lastPalmY = null;
    gestureIndicator.style.opacity = '0';
    return;
  }
  
  const landmarks = results.multiHandLandmarks[0];
  
  // Show gesture indicator
  gestureIndicator.style.opacity = '1';
  updateGestureIndicatorPosition(landmarks);
  
  // Run gesture detectors
  detectPinch(landmarks);
  detectClick(landmarks);
  detectRotation(landmarks);
}
```

---

## Pinch Handler (State Machine)

```javascript
function onPinch() {
  // Visual feedback
  flashGestureRipple();
  
  if (appState === 'OVERVIEW') {
    // Zoom to Sun
    appState = 'FOCUSED_SUN';
    currentFocusIndex = -1;
    transitionToFocus(sunMesh);
    showHUD('sun');
    
  } else if (appState === 'FOCUSED_SUN' || appState === 'FOCUSED_PLANET') {
    // Advance to next planet
    currentFocusIndex++;
    
    if (currentFocusIndex >= SOLAR_DATA.planets.length) {
      // Gone past Neptune, return to overview
      transitionToOverview();
      return;
    }
    
    appState = 'FOCUSED_PLANET';
    const targetMesh = planets[currentFocusIndex].mesh;
    transitionToFocus(targetMesh);
    showHUD('planet', currentFocusIndex);
    
  } else if (appState === 'INFO_OPEN') {
    // While info is open, pinch closes info and moves to next
    hideInfoPanel();
    currentFocusIndex++;
    
    if (currentFocusIndex >= SOLAR_DATA.planets.length) {
      transitionToOverview();
      return;
    }
    
    appState = 'FOCUSED_PLANET';
    transitionToFocus(planets[currentFocusIndex].mesh);
    showHUD('planet', currentFocusIndex);
  }
}
```

---

## Click Handler

```javascript
function onClickGesture() {
  if (appState === 'FOCUSED_SUN') {
    appState = 'INFO_OPEN';
    showInfoPanel('sun');
    
  } else if (appState === 'FOCUSED_PLANET') {
    appState = 'INFO_OPEN';
    showInfoPanel(currentFocusIndex);
    
  } else if (appState === 'INFO_OPEN') {
    hideInfoPanel();
    appState = currentFocusIndex === -1 ? 'FOCUSED_SUN' : 'FOCUSED_PLANET';
  }
}
```

---

## UI: HTML / CSS Layer

All UI is a `position: fixed` overlay on top of the Three.js canvas.

### Full CSS

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #000005;
  overflow: hidden;
  font-family: 'Exo 2', sans-serif;
  color: #e0f7fa;
}

#canvas-container {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
}

#video-element {
  position: fixed;
  top: 0; left: 0;
  width: 1px; height: 1px;
  opacity: 0;
  pointer-events: none;
}

#ui-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
  pointer-events: none;
  z-index: 10;
}

/* ─── HUD: Top Center Title ─────────────────────────────── */
#hud-title {
  position: absolute;
  top: 32px;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  transition: opacity 0.5s;
}

#hud-title .body-label {
  font-size: 13px;
  font-weight: 300;
  letter-spacing: 4px;
  text-transform: uppercase;
  color: #80deea;
  margin-bottom: 6px;
}

#hud-title .body-name {
  font-size: 36px;
  font-weight: 700;
  letter-spacing: 2px;
  color: #ffd54f;
  text-shadow: 0 0 30px rgba(255, 213, 79, 0.6);
}

/* ─── HUD: Bottom Gesture Hints ─────────────────────────── */
#hud-hints {
  position: absolute;
  bottom: 36px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 32px;
  align-items: center;
}

.hint-item {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  color: #80deea;
  opacity: 0.8;
}

.hint-icon {
  width: 32px;
  height: 32px;
  border: 1.5px solid #4fc3f7;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
}

/* ─── HUD: Navigation Progress ─────────────────────────── */
#nav-progress {
  position: absolute;
  bottom: 90px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
  align-items: center;
}

.nav-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #334466;
  border: 1px solid #4fc3f7;
  transition: all 0.4s ease;
}

.nav-dot.active {
  background: #ffd54f;
  box-shadow: 0 0 10px rgba(255, 213, 79, 0.8);
  transform: scale(1.4);
}

.nav-dot.sun-dot {
  width: 10px;
  height: 10px;
  background: #FF8800;
  border-color: #FDB813;
}

/* ─── Gesture Indicator ─────────────────────────────────── */
#gesture-indicator {
  position: absolute;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: rgba(79, 195, 247, 0.4);
  border: 2px solid #4fc3f7;
  transform: translate(-50%, -50%);
  pointer-events: none;
  transition: opacity 0.3s;
  opacity: 0;
  box-shadow: 0 0 15px rgba(79, 195, 247, 0.6);
}

/* ─── Pinch Ripple ──────────────────────────────────────── */
.ripple {
  position: absolute;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 2px solid #4fc3f7;
  transform: translate(-50%, -50%) scale(0);
  pointer-events: none;
  animation: rippleAnim 0.6s ease-out forwards;
}

@keyframes rippleAnim {
  0%   { transform: translate(-50%, -50%) scale(0.3); opacity: 1; }
  100% { transform: translate(-50%, -50%) scale(2.5); opacity: 0; }
}

/* ─── Info Panel ─────────────────────────────────────────── */
#info-panel {
  position: absolute;
  left: 32px;
  top: 50%;
  transform: translateY(-50%);
  width: 320px;
  max-height: 75vh;
  overflow-y: auto;
  background: rgba(5, 10, 30, 0.88);
  border: 1px solid #4fc3f7;
  border-radius: 12px;
  padding: 28px 24px;
  box-shadow:
    0 0 0 1px rgba(79, 195, 247, 0.1),
    0 8px 40px rgba(0, 0, 0, 0.8),
    inset 0 0 60px rgba(79, 195, 247, 0.03);
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.4s ease, transform 0.4s ease;
  transform: translateY(-50%) translateX(-30px);
  scrollbar-width: thin;
  scrollbar-color: #4fc3f7 transparent;
}

#info-panel.visible {
  opacity: 1;
  pointer-events: auto;
  transform: translateY(-50%) translateX(0);
}

#info-panel .panel-type {
  font-size: 10px;
  letter-spacing: 4px;
  text-transform: uppercase;
  color: #4fc3f7;
  margin-bottom: 6px;
}

#info-panel .panel-name {
  font-size: 28px;
  font-weight: 700;
  color: #ffd54f;
  margin-bottom: 20px;
  text-shadow: 0 0 20px rgba(255, 213, 79, 0.4);
  line-height: 1.1;
}

#info-panel .divider {
  width: 100%;
  height: 1px;
  background: linear-gradient(to right, #4fc3f7, transparent);
  margin: 16px 0;
}

#info-panel .stat-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 12px;
  gap: 12px;
}

#info-panel .stat-label {
  font-size: 10px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: #4fc3f7;
  min-width: 90px;
  line-height: 1.5;
}

#info-panel .stat-value {
  font-size: 13px;
  color: #e0f7fa;
  text-align: right;
  line-height: 1.5;
  font-weight: 600;
}

#info-panel .fun-fact {
  margin-top: 16px;
  padding: 14px;
  background: rgba(79, 195, 247, 0.05);
  border-left: 2px solid #4fc3f7;
  border-radius: 0 6px 6px 0;
  font-size: 12px;
  color: #b2ebf2;
  line-height: 1.7;
  font-style: italic;
}

#info-panel .close-hint {
  margin-top: 20px;
  text-align: center;
  font-size: 10px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: #4fc3f7;
  opacity: 0.6;
}

/* ─── Loading Screen ─────────────────────────────────────── */
#loading-screen {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
  background: #000005;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 100;
  transition: opacity 0.8s ease;
}

#loading-screen .load-title {
  font-size: 42px;
  font-weight: 700;
  letter-spacing: 6px;
  color: #ffd54f;
  text-shadow: 0 0 40px rgba(255, 213, 79, 0.5);
  margin-bottom: 12px;
}

#loading-screen .load-subtitle {
  font-size: 13px;
  letter-spacing: 6px;
  text-transform: uppercase;
  color: #4fc3f7;
  margin-bottom: 48px;
}

.load-bar-container {
  width: 280px;
  height: 2px;
  background: rgba(79, 195, 247, 0.2);
  border-radius: 1px;
  overflow: hidden;
}

.load-bar {
  height: 100%;
  background: linear-gradient(to right, #4fc3f7, #ffd54f);
  width: 0%;
  transition: width 0.3s ease;
}

#loading-screen .load-status {
  margin-top: 20px;
  font-size: 11px;
  letter-spacing: 3px;
  text-transform: uppercase;
  color: #80deea;
  opacity: 0.7;
}

/* ─── Permission Modal ───────────────────────────────────── */
#permission-modal {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
  background: rgba(0, 0, 5, 0.92);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
}

.permission-box {
  background: rgba(5, 10, 30, 0.95);
  border: 1px solid #4fc3f7;
  border-radius: 16px;
  padding: 48px;
  max-width: 400px;
  text-align: center;
}

.permission-box h2 {
  font-size: 24px;
  color: #ffd54f;
  margin-bottom: 16px;
}

.permission-box p {
  font-size: 14px;
  color: #80deea;
  line-height: 1.7;
  margin-bottom: 32px;
}

.permission-box button {
  background: transparent;
  border: 1.5px solid #4fc3f7;
  color: #4fc3f7;
  font-family: 'Exo 2', sans-serif;
  font-size: 13px;
  letter-spacing: 3px;
  text-transform: uppercase;
  padding: 14px 36px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s;
}

.permission-box button:hover {
  background: rgba(79, 195, 247, 0.12);
  box-shadow: 0 0 20px rgba(79, 195, 247, 0.3);
}

/* Scrollbar styles */
#info-panel::-webkit-scrollbar { width: 4px; }
#info-panel::-webkit-scrollbar-track { background: transparent; }
#info-panel::-webkit-scrollbar-thumb { background: #4fc3f7; border-radius: 2px; }

/* Responsive */
@media (max-width: 600px) {
  #info-panel { width: calc(100vw - 48px); left: 24px; }
}
```

---

## UI: JavaScript Functions

### Show Info Panel

```javascript
function showInfoPanel(target) {
  const panel = document.getElementById('info-panel');
  let data, name, type;
  
  if (target === 'sun') {
    data = SOLAR_DATA.sun.info;
    name = SOLAR_DATA.sun.name;
    type = data.type;
  } else {
    const planet = SOLAR_DATA.planets[target];
    data = planet.info;
    name = planet.name;
    type = data.type;
  }
  
  panel.innerHTML = `
    <div class="panel-type">${type}</div>
    <div class="panel-name">${name}</div>
    <div class="divider"></div>
    ${Object.entries(data).filter(([k]) => k !== 'type' && k !== 'funFact').map(([key, val]) => `
      <div class="stat-row">
        <div class="stat-label">${formatKey(key)}</div>
        <div class="stat-value">${val}</div>
      </div>
    `).join('')}
    <div class="divider"></div>
    <div class="fun-fact">${data.funFact}</div>
    <div class="close-hint">✦ Click gesture to close ✦</div>
  `;
  
  panel.classList.add('visible');
}

function hideInfoPanel() {
  document.getElementById('info-panel').classList.remove('visible');
}

function formatKey(key) {
  return key.replace(/([A-Z])/g, ' $1').replace(/^./, s => s.toUpperCase());
}
```

### Show HUD

```javascript
function showHUD(mode, planetIndex) {
  const titleEl = document.getElementById('hud-title');
  const hintsEl = document.getElementById('hud-hints');
  const dotsEl = document.getElementById('nav-progress');
  
  if (mode === 'overview') {
    titleEl.innerHTML = `
      <div class="body-label">Solar System</div>
      <div class="body-name">Overview</div>
    `;
    hintsEl.innerHTML = `
      <div class="hint-item"><div class="hint-icon">✌</div>Pinch to enter</div>
    `;
    // Update dots - none active
    updateNavDots(-1);
    
  } else if (mode === 'sun') {
    titleEl.innerHTML = `
      <div class="body-label">G-Type Star</div>
      <div class="body-name">The Sun</div>
    `;
    hintsEl.innerHTML = `
      <div class="hint-item"><div class="hint-icon">☝</div>Click for info</div>
      <div class="hint-item"><div class="hint-icon">✌</div>Pinch to advance</div>
      <div class="hint-item"><div class="hint-icon">✋</div>Move to rotate</div>
    `;
    updateNavDots(-1);
    
  } else if (mode === 'planet') {
    const planet = SOLAR_DATA.planets[planetIndex];
    titleEl.innerHTML = `
      <div class="body-label">${planet.info.type}</div>
      <div class="body-name">${planet.name}</div>
    `;
    hintsEl.innerHTML = `
      <div class="hint-item"><div class="hint-icon">☝</div>Click for info</div>
      <div class="hint-item"><div class="hint-icon">✌</div>Pinch to next</div>
      <div class="hint-item"><div class="hint-icon">✋</div>Move to rotate</div>
    `;
    updateNavDots(planetIndex);
  }
}

function updateNavDots(activeIndex) {
  const container = document.getElementById('nav-progress');
  container.innerHTML = `<div class="nav-dot sun-dot ${activeIndex === -1 ? 'active' : ''}"></div>`;
  SOLAR_DATA.planets.forEach((p, i) => {
    container.innerHTML += `<div class="nav-dot ${i === activeIndex ? 'active' : ''}"></div>`;
  });
}
```

### Gesture Indicator Update

```javascript
function updateGestureIndicatorPosition(landmarks) {
  const indicator = document.getElementById('gesture-indicator');
  const palm = getPalmCenter(landmarks);
  // Mirror X because camera feed is mirrored
  const x = (1 - palm.x) * window.innerWidth;
  const y = palm.y * window.innerHeight;
  indicator.style.left = x + 'px';
  indicator.style.top = y + 'px';
}
```

### Ripple Flash

```javascript
function flashGestureRipple() {
  const indicator = document.getElementById('gesture-indicator');
  const rect = indicator.getBoundingClientRect();
  
  const ripple = document.createElement('div');
  ripple.classList.add('ripple');
  ripple.style.left = (rect.left + rect.width / 2) + 'px';
  ripple.style.top = (rect.top + rect.height / 2) + 'px';
  document.getElementById('ui-overlay').appendChild(ripple);
  
  ripple.addEventListener('animationend', () => ripple.remove());
}
```

---

## Loading Screen Logic

```javascript
function startLoadingSequence() {
  const bar = document.querySelector('.load-bar');
  const status = document.querySelector('.load-status');
  
  const steps = [
    { msg: 'Initializing renderer...', pct: 20 },
    { msg: 'Building solar system...', pct: 50 },
    { msg: 'Loading hand tracking model...', pct: 75 },
    { msg: 'Awaiting camera access...', pct: 90 }
  ];
  
  let i = 0;
  const tick = () => {
    if (i >= steps.length) return;
    bar.style.width = steps[i].pct + '%';
    status.textContent = steps[i].msg;
    i++;
    setTimeout(tick, 400);
  };
  tick();
}

function hideLoadingScreen() {
  const screen = document.getElementById('loading-screen');
  screen.style.opacity = '0';
  setTimeout(() => screen.style.display = 'none', 800);
}
```

---

## Complete HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Solar System AR Explorer</title>
  <link href="https://fonts.googleapis.com/css2?family=Exo+2:wght@300;400;600;700&display=swap" rel="stylesheet">
  <style>
    /* [ALL CSS FROM ABOVE] */
  </style>
</head>
<body>

<!-- Hidden camera input for MediaPipe -->
<video id="video-element" autoplay muted playsinline></video>

<!-- Three.js canvas -->
<div id="canvas-container"></div>

<!-- UI Overlay -->
<div id="ui-overlay">
  <div id="hud-title"></div>
  <div id="nav-progress"></div>
  <div id="hud-hints"></div>
  <div id="gesture-indicator"></div>
  <div id="info-panel"></div>
</div>

<!-- Loading Screen -->
<div id="loading-screen">
  <div class="load-title">SOLAR</div>
  <div class="load-subtitle">System Explorer</div>
  <div class="load-bar-container">
    <div class="load-bar"></div>
  </div>
  <div class="load-status">Initializing...</div>
</div>

<!-- Permission Modal (shown before camera access) -->
<div id="permission-modal">
  <div class="permission-box">
    <h2>Camera Access Required</h2>
    <p>This app uses your camera to detect hand gestures. Your camera feed is never recorded or transmitted — it stays local.</p>
    <button onclick="startApp()">Enable Camera</button>
  </div>
</div>

<!-- Scripts -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>

<script>
  // [ALL JAVASCRIPT FROM ABOVE, IN ORDER:]
  // 1. SOLAR_DATA constant
  // 2. App state variables
  // 3. Three.js scene, renderer, camera setup
  // 4. Star field creation
  // 5. Sun creation (createSun)
  // 6. Orbit rings creation (createOrbitRing for each planet)
  // 7. Planet creation loop (createPlanet for each in SOLAR_DATA.planets)
  // 8. Asteroid belt (createAsteroidBelt)
  // 9. Lighting setup
  // 10. Camera state object
  // 11. Gesture state variables (pinchState, cooldowns, indexYHistory, etc.)
  // 12. Gesture functions (getPinchDistance, getPalmCenter, detectPinch, detectClick, detectRotation)
  // 13. onPinch handler
  // 14. onClickGesture handler
  // 15. Camera transition functions (transitionToOverview, transitionToFocus)
  // 16. UI functions (showInfoPanel, hideInfoPanel, showHUD, updateNavDots, updateGestureIndicatorPosition, flashGestureRipple, formatKey)
  // 17. onHandResults function
  // 18. animate() loop + requestAnimationFrame
  // 19. Loading screen logic
  // 20. startApp() function (hides permission modal, starts camera, initializes MediaPipe, kicks off loading)
  // 21. Window resize handler
</script>

</body>
</html>
```

---

## startApp() Function — Full Bootstrap

```javascript
async function startApp() {
  // Hide permission modal
  document.getElementById('permission-modal').style.display = 'none';
  
  // Show loading screen
  startLoadingSequence();
  
  // Init Three.js scene (if not already done)
  initScene();
  
  // Start animation loop
  requestAnimationFrame(animate);
  
  // Init MediaPipe
  const handsModel = new Hands({
    locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
  });
  
  handsModel.setOptions({
    maxNumHands: 1,
    modelComplexity: 1,
    minDetectionConfidence: 0.75,
    minTrackingConfidence: 0.75
  });
  
  handsModel.onResults(onHandResults);
  
  const videoEl = document.getElementById('video-element');
  
  const cam = new Camera(videoEl, {
    onFrame: async () => {
      await handsModel.send({ image: videoEl });
    },
    width: 640,
    height: 480
  });
  
  // Update loading bar
  document.querySelector('.load-bar').style.width = '90%';
  document.querySelector('.load-status').textContent = 'Starting camera...';
  
  await cam.start();
  
  // Done
  document.querySelector('.load-bar').style.width = '100%';
  document.querySelector('.load-status').textContent = 'Ready';
  
  setTimeout(() => {
    hideLoadingScreen();
    showHUD('overview');
    updateNavDots(-1);
  }, 600);
}
```

---

## initScene() — Full Scene Initialization

```javascript
function initScene() {
  // Scene
  const scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2(0x000005, 0.0008);
  
  // Renderer
  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.setClearColor(0x000005, 1);
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  // Camera
  const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 2000);
  camera.position.set(0, 80, 160);
  camera.lookAt(0, 0, 0);
  
  // Expose to outer scope
  window._scene = scene;
  window._renderer = renderer;
  window._camera = camera;
  
  // Lights
  scene.add(new THREE.AmbientLight(0x111133, 0.5));
  const sunLight = new THREE.PointLight(0xFDB813, 3.0, 500);
  sunLight.position.set(0, 0, 0);
  scene.add(sunLight);
  const fillLight = new THREE.DirectionalLight(0x334466, 0.3);
  fillLight.position.set(0, 100, 200);
  scene.add(fillLight);
  
  // Stars
  createStarField(scene);
  
  // Sun
  window._sunMesh = createSun(scene);
  
  // Orbit rings + planets
  window._planets = [];
  SOLAR_DATA.planets.forEach((data, i) => {
    createOrbitRing(scene, data.orbitRadius);
    const p = createPlanet(scene, data, i);
    window._planets.push(p);
  });
  
  // Asteroid belt
  createAsteroidBelt(scene);
  
  // Resize handler
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
}
```

---

## Important Implementation Notes

### 1. Globals vs Closures
Since this is a single-file app with no module system, use `window._varName` for shared state between functions, or declare all core variables at the top of the script tag before any function definitions.

### 2. Gesture Sensitivity Tuning
The pinch threshold of `0.06` and click dip of `0.04` are starting points. Add a small settings overlay (hidden by default, toggled with a keyboard shortcut like `Shift+S`) that lets the developer adjust these values via range sliders, to make calibration easy without redeploying.

### 3. Performance Considerations
- Use `Math.min(window.devicePixelRatio, 2)` on the renderer to prevent excessive GPU load on Retina screens.
- Limit the MediaPipe model complexity to `1` (not 2) for real-time performance on mid-range devices.
- The orbit rings use `THREE.TorusGeometry` with only 8 tube segments for performance.
- Star field uses `THREE.Points`, not individual Mesh objects — never create 3000 individual meshes.

### 4. No OrbitControls
Do NOT use `THREE.OrbitControls`. Camera movement is handled entirely through the custom gesture system and GSAP tweens.

### 5. RingGeometry UV Fix (Saturn)
`THREE.RingGeometry` in Three.js r128 has known UV issues. If the ring renders with incorrect shading, rotate the ring geometry vertices manually or use a custom shader. The simplest fix: create the ring with `THREE.DoubleSide` material and `depthWrite: false`.

### 6. Safari / iOS Compatibility
- Add `playsinline` attribute to the video element (already included above).
- Camera access on Safari requires HTTPS. For local testing, use `localhost` which is exempt.
- MediaPipe Camera Utils handles `getUserMedia` internally; do not call it separately.

### 7. CORS / CDN Notes
All CDNs used support CORS. If you self-host, ensure proper `Access-Control-Allow-Origin` headers on the MediaPipe model files.

### 8. Gesture Conflict Prevention
Pinch and click gesture detectors must not fire simultaneously. Add a global `gestureOnCooldown` flag that blocks all gesture processing for 800ms after any gesture fires:
```javascript
let gestureOnCooldown = false;
function triggerGestureOnce(fn) {
  if (gestureOnCooldown) return;
  gestureOnCooldown = true;
  fn();
  setTimeout(() => { gestureOnCooldown = false; }, 800);
}
```

---

## Milestone Checklist for the Agent

When building this app, complete these milestones in order:

- [ ] **M1:** Bare HTML file with Three.js scene renders (black background, camera, lighting)
- [ ] **M2:** Star field renders and slowly rotates
- [ ] **M3:** Sun with glow renders at origin
- [ ] **M4:** All 8 planets with orbit rings render and animate
- [ ] **M5:** Asteroid belt renders
- [ ] **M6:** Camera permission modal and loading screen UI implemented
- [ ] **M7:** MediaPipe hands initialized, camera feed running, gesture indicator dot moves with hand
- [ ] **M8:** Pinch gesture fires reliably, triggers console.log
- [ ] **M9:** Click gesture fires reliably, triggers console.log
- [ ] **M10:** State machine implemented — pinch transitions through Sun → Mercury → ... → Neptune → Overview
- [ ] **M11:** Camera zoom-in transitions work (GSAP tweens)
- [ ] **M12:** Hand-rotate on focused object works
- [ ] **M13:** Info panel opens/closes with click gesture, shows correct data
- [ ] **M14:** HUD title and hint text updates correctly per state
- [ ] **M15:** Navigation dots update correctly
- [ ] **M16:** Final polish — ripple effects, smooth transitions, performance check on mobile

---

## Deliverable

A single `index.html` file. No other files. No build step. Open in any modern browser (Chrome/Edge recommended for best MediaPipe support), allow camera access, and the app runs.

The file should be approximately **600–900 lines** of clean, commented code.

---

*End of specification. All code patterns, data, CSS, and architecture decisions above are final and should be implemented exactly as described.*