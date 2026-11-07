# Pin Türkiye: A GeoGame

**Course:** GMT 458 – Web GIS
**Assignment 2:** GeoGame (Design Phase)
**Design deadline:** 9 November 2026

---

## 1. Concept

**Pin Türkiye** is a time-limited geography game. In each round the game shows the **name of a place in Turkey** (a province, a district, or a famous landmark). The player must **click on the map where that place is**. The closer the click is to the real location, the more points the player earns.

The player has **60 seconds** to place as many pins as possible and has **3 lives**. Difficulty changes how much help the map gives: on harder levels, labels and borders disappear from the base map, so the player has to rely on real geographic knowledge.


---

## 2. Game Progression

| Item | Decision |
|---|---|
| **Game type** | Time-based: place as many pins as possible within **60 seconds** |
| **Number of questions** | Unlimited within the time limit (typically 10–20 for a good player). Each place name is one question. |
| **Lives** | **3 lives (♥♥♥)**. A guess further than the level's "fail distance" costs 1 life. The game ends when time runs out or lives reach 0. |
| **Time bonus** | A very accurate guess (within the level's "perfect distance") gives **+3 seconds**. |
| **Difficulty levels** | Easy / Medium / Hard (see below) |

### Difficulty levels

| Level | Places asked | Base map | Perfect distance | Fail distance |
|---|---|---|---|---|
| **Easy** | 20 biggest cities (Istanbul, Ankara, Izmir, ...) | Labels + province borders | < 50 km | > 300 km |
| **Medium** | All 81 provinces | Province borders only, no labels | < 30 km | > 200 km |
| **Hard** | Districts, small towns and landmarks (Anıtkabir, Nemrut, Ani, Efes, ...) | Plain map, no labels, no borders | < 15 km | > 100 km |

The game also gets harder **during** a round: after every 5 correct answers the game gradually removes map help (labels first, then borders).

### Scoring

```
distance_km = haversine(user_click, real_location)

score  = max(0, 1000 - distance_km * k)     // up to 1000 points per place
         (k depends on the difficulty level)
streak = +10% per consecutive good guess (max +50%)
time   = +3 seconds for a "perfect" guess
```

At the end, the final score and statistics are shown. The best scores are stored in `localStorage` as a top-10 table.

---

## 3. Requirements

### 3.1 Functional requirements

1. FR1 – Display an interactive base map of Turkey.
2. FR2 – Load the place data (provinces, districts, landmarks) from local JSON/GeoJSON files.
3. FR3 – Let the user select a difficulty level on the start screen.
4. FR4 – In each round, show a place name; the user clicks the map to place a guess pin.
5. FR5 – After a guess, reveal the real location, draw a line between guess and truth, and show the distance and points earned.
6. FR6 – Show the **60-second countdown**, the **lives** and the **score** at all times.
7. FR7 – End the game when time is up or lives are 0, then show the result screen.
8. FR8 – Store and display the top-10 high scores (localStorage).
9. FR9 – "Play again" restarts the game without reloading the page.
10. FR10 – On the result screen, show a Chart.js chart of distance error per round and a Deck.gl arc layer connecting each guess to the real location.

### 3.2 Non-functional requirements

- NFR1 – Runs fully on the client side (static files) so it can be hosted on **GitHub Pages**.
- NFR2 – Responsive layout (desktop first, usable on mobile).
- NFR3 – Data files are small (< 2 MB total) so the game loads quickly.
- NFR4 – Plain HTML, CSS and JavaScript (no backend).
- NFR5 – Clear, high-contrast UI with large buttons.

---

## 4. Technologies / JS Libraries

| Library | Purpose |
|---|---|
| **Leaflet** | Main interactive map, click events, markers, lines, province polygons |
| **Deck.gl** *(bonus)* | Arc layer of guess → real location on the result screen |
| **Chart.js** *(bonus)* | Charts of distance error and score per round |
| **Turf.js** | Distance calculation and geometry helpers |

Also used: HTML5, CSS3 (Flexbox/Grid), vanilla JavaScript (ES6), GitHub Pages for hosting.
Base map tiles: CARTO "Positron" (with labels / no labels variants) based on OpenStreetMap data.

---

## 5. Data

| Data set | Source | Content |
|---|---|---|
| Turkey provinces (81) | Natural Earth / GADM / geoBoundaries | Polygons + names (GeoJSON) |
| Place list | Compiled by hand from OpenStreetMap / Wikipedia | Name, type, coordinates, difficulty level |

**Place list fields:**

| Field | Description |
|---|---|
| `name` | Name shown to the player |
| `type` | `province`, `district` or `landmark` |
| `lat`, `lon` | Real location (WGS84) |
| `level` | `easy`, `medium` or `hard` |
| `region` | One of the 7 geographic regions (used in the result summary) |

**Preparation steps:**

1. Download the province boundaries and simplify them (mapshaper) to keep the file small.
2. Compute province centroids (or use the capital-city coordinates).
3. Add ~60–100 districts and landmarks with coordinates from OpenStreetMap.
4. Export to `data/provinces.geojson` and `data/places.json`.

The data layer is separate from the game logic, so the same game can later be switched to another country or theme by replacing the data files.

---

## 6. Frontend Layout (Sketches)

### 6.1 Start screen

```
+--------------------------------------------------------------+
|                        PIN TÜRKİYE                           |
|                                                              |
|          Find the place on the map before time runs out!     |
|                                                              |
|              Select difficulty:                              |
|          [  EASY  ]   [ MEDIUM ]   [  HARD  ]               |
|                                                              |
|                   [   ▶  START GAME   ]                      |
|                                                              |
|              [ How to play ]     [ High scores ]             |
+--------------------------------------------------------------+
```

### 6.2 Game screen

```
+--------------------------------------------------------------+
|  ⏱ 00:42        SCORE: 3450         LIVES: ♥ ♥ ♡    Round 5   |
+----------------------------------------------+---------------+
|                                              |  FIND:        |
|                                              |               |
|                                              |   ERZURUM     |
|              MAP (Leaflet)                   |               |
|                                              |  Type:        |
|                                              |  Province     |
|                    ✖ your guess              |               |
|                                              |  Streak: 🔥 x3|
|        (click the map to place your pin)     |               |
|                                              |  [ SUBMIT ]   |
+----------------------------------------------+---------------+
|  Last result: Sivas — 42 km away → +790 points               |
+--------------------------------------------------------------+
```

### 6.3 Feedback after each guess

```
 Map shows:  ✖ your guess ─────── ● real location
             (line + distance label)
 Popup: "Erzurum — 42 km away  (+790 points)"
```

### 6.4 Game-over screen

```
+--------------------------------------------------------------+
|                        GAME OVER                             |
|              Final score: 7,820    Rank: #3                  |
+----------------------------+---------------------------------+
|  Chart.js bar chart:       |  Deck.gl map:                   |
|  distance error per round  |  arcs from each guess to        |
|  ▂ ▅ ▃ ▇ ▁ ▄               |  the real location              |
+----------------------------+---------------------------------+
|        [ ↻ PLAY AGAIN ]          [ 🏆 HIGH SCORES ]          |
+--------------------------------------------------------------+
```

### 6.5 Game flow

```
Start → choose difficulty → timer starts (60 s)
   → show place name → user clicks map → submit
   → reveal real location → score / lives / time bonus updated
   → next place ──┐
        ▲         │ (repeat until time = 0 or lives = 0)
        └─────────┘
   → Game over → charts + high scores → play again
```

---

## 7. Project Structure (planned)

```
/
├── index.html
├── css/
│   └── style.css
├── js/
│   ├── main.js        # game loop, timer, lives
│   ├── map.js         # Leaflet map and layers
│   ├── data.js        # data loading and place selection
│   ├── score.js       # scoring and high scores
│   └── results.js     # Chart.js + Deck.gl result screen
├── data/
│   ├── provinces.geojson
│   └── places.json
└── README.md
```

---

## 8. Implementation Plan

| Step | Task |
|---|---|
| 1 | Prepare data (provinces GeoJSON, place list) |
| 2 | Base map, click handling, markers and lines |
| 3 | Game loop: timer, lives, scoring |
| 4 | Difficulty levels, high scores, UI polish |
| 5 | Result screen with Chart.js and Deck.gl (bonus) |
| 6 | Testing and deployment to GitHub Pages |

**Git usage:** at least 1 design commit (this README) before the design deadline and 5+ implementation commits before the final deadline.

---

## 9. Author

- Name: _your name_
- Student ID: _your number_
