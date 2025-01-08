---
title: Building a Google Maps Application with React and TypeScript
date: 2025-01-08 10:47:17
categories:
- ReactJS
- TypeScript
- GIS
tags:
- ReactJS
- TypeScript
- GIS
---

## Prerequisites

1. Node.js and npm installed.
2. A Google Cloud account.
3. Basic understanding of React and TypeScript.

---

## Step 1: Set Up Google Developer Console

### 1.1 Enable APIs and Create Credentials
1. Visit the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project or use an existing one.
3. Navigate to `APIs & Services` > `Library` and enable:
   - **Maps JavaScript API**
   - **Places API**
4. Go to `Credentials` and create an **API Key**. Restrict the key to specific APIs and your domain.

### 1.2 Create a Map ID for Advanced Markers
1. Go to `Google Maps Platform` > `Maps` > `Map Management`.
2. Create a new map and generate a **Map ID**.
3. Configure the map settings for **Advanced Markers**.

---

## Step 2: Initialize the React Project

### 2.1 Create a React Application
```bash
npx create-react-app google-map-app --template typescript
cd google-map-app
```

### 2.2 Install Required Libraries
```bash
npm install @react-google-maps/api
npm install --save-dev typescript @types/react
```

---

## Step 3: Project Structure

```
src/
├── components/
│   ├── Map.tsx
│   ├── SearchBox.tsx
│   ├── MarkerManager.tsx
├── styles/
│   └── map.css
├── App.tsx
├── index.tsx
```

---

## Step 4: Build the Application

### 4.1 Configure the Map Component

```tsx
// src/components/Map.tsx
import React, { useState, useCallback } from "react";
import { GoogleMap, useLoadScript, MarkerF } from "@react-google-maps/api";
import "./../styles/map.css";

const mapContainerStyle = {
  width: "100%",
  height: "600px",
};

const defaultCenter = {
  lat: 40.712776,
  lng: -74.005974,
};

interface MapProps {
  onMapClick: (lat: number, lng: number) => void;
  markers: { lat: number; lng: number }[];
}

const Map: React.FC<MapProps> = ({ onMapClick, markers }) => {
  const { isLoaded } = useLoadScript({
    googleMapsApiKey: process.env.REACT_APP_GOOGLE_MAPS_API_KEY!,
    libraries: ["places"],
  });

  if (!isLoaded) return <div>Loading...</div>;

  return (
    <GoogleMap
      mapContainerStyle={mapContainerStyle}
      center={defaultCenter}
      zoom={10}
      onClick={(event) =>
        onMapClick(event.latLng?.lat() || 0, event.latLng?.lng() || 0)
      }
    >
      {markers.map((marker, index) => (
        <MarkerF key={index} position={marker} />
      ))}
    </GoogleMap>
  );
};

export default Map;
```

### 4.2 Create the Search Box Component

```tsx
// src/components/SearchBox.tsx
import React, { useRef } from "react";
import { Autocomplete } from "@react-google-maps/api";

interface SearchBoxProps {
  onPlaceSelect: (lat: number, lng: number) => void;
}

const SearchBox: React.FC<SearchBoxProps> = ({ onPlaceSelect }) => {
  const autocompleteRef = useRef<google.maps.places.Autocomplete | null>(null);

  const handlePlaceChanged = () => {
    const place = autocompleteRef.current?.getPlace();
    if (place && place.geometry) {
      const lat = place.geometry.location?.lat() || 0;
      const lng = place.geometry.location?.lng() || 0;
      onPlaceSelect(lat, lng);
    }
  };

  return (
    <Autocomplete
      onLoad={(autocomplete) => (autocompleteRef.current = autocomplete)}
      onPlaceChanged={handlePlaceChanged}
    >
      <input
        type="text"
        placeholder="Search for places"
        className="search-box"
      />
    </Autocomplete>
  );
};

export default SearchBox;
```

### 4.3 Manage Markers

```tsx
// src/components/MarkerManager.tsx
import React from "react";

interface MarkerManagerProps {
  markers: { lat: number; lng: number }[];
}

const MarkerManager: React.FC<MarkerManagerProps> = ({ markers }) => {
  return (
    <ul className="marker-list">
      {markers.map((marker, index) => (
        <li key={index}>
          {`Lat: ${marker.lat.toFixed(4)}, Lng: ${marker.lng.toFixed(4)}`}
        </li>
      ))}
    </ul>
  );
};

export default MarkerManager;
```

### 4.4 Integrate in `App.tsx`

```tsx
// src/App.tsx
import React, { useState } from "react";
import Map from "./components/Map";
import SearchBox from "./components/SearchBox";
import MarkerManager from "./components/MarkerManager";

const App: React.FC = () => {
  const [markers, setMarkers] = useState<{ lat: number; lng: number }[]>([]);

  const handleMapClick = (lat: number, lng: number) => {
    setMarkers((prev) => [...prev, { lat, lng }]);
  };

  const handlePlaceSelect = (lat: number, lng: number) => {
    setMarkers((prev) => [...prev, { lat, lng }]);
  };

  return (
    <div className="app-container">
      <h1>Google Map Application</h1>
      <SearchBox onPlaceSelect={handlePlaceSelect} />
      <Map onMapClick={handleMapClick} markers={markers} />
      <MarkerManager markers={markers} />
    </div>
  );
};

export default App;
```

---

## Step 5: Add Styles

```css
/* src/styles/map.css */
.app-container {
  font-family: Arial, sans-serif;
  text-align: center;
}

.search-box {
  width: 300px;
  padding: 10px;
  margin: 10px;
}

.marker-list {
  list-style-type: none;
  padding: 0;
}
```

---

## Step 6: Test the Application

1. Run the application:
   ```bash
   npm start
   ```
2. Search for places using the search box.
3. Click on the map to add markers.
4. View the list of markers below the map.

---

## Step 7: Advanced Markers

1. Use the **Map ID** created earlier.
2. Update the `useLoadScript` in `Map.tsx` to include your Map ID:
   ```tsx
   const { isLoaded } = useLoadScript({
     googleMapsApiKey: process.env.REACT_APP_GOOGLE_MAPS_API_KEY!,
     libraries: ["places"],
     mapIds: ["YOUR_MAP_ID"],
   });
   ```
