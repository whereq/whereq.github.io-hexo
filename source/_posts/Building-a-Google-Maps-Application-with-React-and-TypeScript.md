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

- **Node.js** installed (version 14 or higher).
- **npm** or **yarn** for package management.
- A Google Cloud Platform (GCP) account to obtain the API Key and set up the Map ID.

## Project Setup

1. **Create a React Application**:
   ```bash
   npx create-react-app google-maps-app --template typescript
   cd google-maps-app
   ```

2. **Install Required Dependencies**:
   ```bash
   npm install @react-google-maps/api
   npm install react-hook-form
   ```

3. **Setup Google Cloud Platform**:
   - **Enable the Maps JavaScript API**:
     1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
     2. Create a new project or select an existing one.
     3. Navigate to `APIs & Services` > `Library` and enable the `Maps JavaScript API`.
   - **Generate an API Key**:
     1. Go to `APIs & Services` > `Credentials` and create an API Key.
     2. Restrict the API Key to ensure security by specifying HTTP referrers (your domain or `localhost`).
   - **Create a Map ID**:
     1. Go to `Google Maps Platform` > `Maps` > `Map Management`.
     2. Create a new Map ID and configure it with desired styles and settings.

## Project Structure

```
google-maps-app/
├── public/
├── src/
│   ├── components/
│   │   ├── MapContainer.tsx
│   │   ├── SearchBox.tsx
│   ├── App.tsx
│   ├── index.tsx
│   ├── types/
│   │   ├── index.d.ts
│   ├── utils/
│       ├── mapUtils.ts
```

## Coding the Application

### Step 1: Setup TypeScript Types

Create `src/types/index.d.ts` to define types for the Google Maps objects:

```typescript
declare global {
  interface Window {
    google: any;
  }
}

export interface MarkerData {
  position: google.maps.LatLngLiteral;
  title: string;
}
```

### Step 2: Utility Functions

Create `src/utils/mapUtils.ts` to define helper functions:

```typescript
export const loadGoogleMapsScript = (apiKey: string): Promise<void> => {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = `https://maps.googleapis.com/maps/api/js?key=${apiKey}&libraries=places`;
    script.async = true;
    script.defer = true;
    script.onload = () => resolve();
    script.onerror = (e) => reject(e);
    document.head.appendChild(script);
  });
};
```

### Step 3: Map Container

Create `src/components/MapContainer.tsx`:

```tsx
import React, { useEffect, useRef, useState } from 'react';
import { MarkerData } from '../types';

type MapContainerProps = {
  apiKey: string;
  mapId: string;
};

const MapContainer: React.FC<MapContainerProps> = ({ apiKey, mapId }) => {
  const mapRef = useRef<HTMLDivElement>(null);
  const [map, setMap] = useState<google.maps.Map | null>(null);
  const [markers, setMarkers] = useState<google.maps.Marker[]>([]);

  useEffect(() => {
    const initMap = async () => {
      try {
        await loadGoogleMapsScript(apiKey);
        if (mapRef.current) {
          const mapInstance = new google.maps.Map(mapRef.current, {
            center: { lat: 37.7749, lng: -122.4194 },
            zoom: 12,
            mapId: mapId,
          });
          setMap(mapInstance);
        }
      } catch (error) {
        console.error('Failed to load Google Maps:', error);
      }
    };

    initMap();
  }, [apiKey, mapId]);

  const addMarker = (markerData: MarkerData) => {
    if (map) {
      const marker = new google.maps.marker.AdvancedMarkerElement({
        map,
        position: markerData.position,
        title: markerData.title,
      });
      setMarkers((prev) => [...prev, marker]);
    }
  };

  return <div ref={mapRef} style={{ width: '100%', height: '500px' }} />;
};

export default MapContainer;
```

### Step 4: Search Box

Create `src/components/SearchBox.tsx`:

```tsx
import React, { useEffect, useRef } from 'react';

type SearchBoxProps = {
  onPlaceSelected: (place: google.maps.places.PlaceResult) => void;
};

const SearchBox: React.FC<SearchBoxProps> = ({ onPlaceSelected }) => {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (inputRef.current) {
      const autocomplete = new google.maps.places.Autocomplete(inputRef.current);
      autocomplete.addListener('place_changed', () => {
        const place = autocomplete.getPlace();
        if (place) {
          onPlaceSelected(place);
        }
      });
    }
  }, []);

  return <input ref={inputRef} type="text" placeholder="Search places..." />;
};

export default SearchBox;
```

### Step 5: Combine Components

Update `src/App.tsx`:

```tsx
import React, { useState } from 'react';
import MapContainer from './components/MapContainer';
import SearchBox from './components/SearchBox';
import { MarkerData } from './types';

const App: React.FC = () => {
  const [markers, setMarkers] = useState<MarkerData[]>([]);
  const apiKey = 'YOUR_GOOGLE_MAPS_API_KEY';
  const mapId = 'YOUR_MAP_ID';

  const handlePlaceSelected = (place: google.maps.places.PlaceResult) => {
    if (place.geometry?.location) {
      const markerData = {
        position: {
          lat: place.geometry.location.lat(),
          lng: place.geometry.location.lng(),
        },
        title: place.name || '',
      };
      setMarkers((prev) => [...prev, markerData]);
    }
  };

  return (
    <div>
      <SearchBox onPlaceSelected={handlePlaceSelected} />
      <MapContainer apiKey={apiKey} mapId={mapId} />
    </div>
  );
};

export default App;
```

## Running the Application

1. Replace `YOUR_GOOGLE_MAPS_API_KEY` and `YOUR_MAP_ID` in `App.tsx` with your actual values.
2. Start the application:
   ```bash
   npm start
   ```
3. Open `http://localhost:3000` in your browser.


