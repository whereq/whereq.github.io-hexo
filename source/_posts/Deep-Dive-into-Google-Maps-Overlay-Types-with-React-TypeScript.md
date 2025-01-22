---
title: Deep Dive into Google Maps Overlay Types with React + TypeScript
date: 2025-01-22 12:05:47
categories:
- ReactJS
- TypeScript
- GIS
tags:
- ReactJS
- TypeScript
- GIS
---

1. [Introduction to Google Maps Overlays](#introduction-to-google-maps-overlays)
2. [Markers](#markers)
   - [Basic Marker](#basic-marker)
   - [Custom Marker with `AdvancedMarkerElement`](#custom-marker-with-advancedmarkerelement)
3. [Polygons](#polygons)
   - [Drawing a Polygon](#drawing-a-polygon)
   - [Handling Polygon Events](#handling-polygon-events)
4. [Polylines](#polylines)
   - [Drawing a Polyline](#drawing-a-polyline)
   - [Handling Polyline Events](#handling-polyline-events)
5. [Circles](#circles)
   - [Drawing a Circle](#drawing-a-circle)
   - [Handling Circle Events](#handling-circle-events)
6. [Rectangles](#rectangles)
   - [Drawing a Rectangle](#drawing-a-rectangle)
   - [Handling Rectangle Events](#handling-rectangle-events)
7. [InfoWindows](#infowindows)
   - [Adding an InfoWindow](#adding-an-infowindow)
   - [Handling InfoWindow Events](#handling-infowindow-events)
8. [GroundOverlay](#groundoverlay)
   - [Adding a GroundOverlay](#adding-a-groundoverlay)
9. [Custom Overlays](#custom-overlays)
   - [Creating a Custom Overlay](#creating-a-custom-overlay)
10. [Conclusion](#conclusion)

---

## Introduction to Google Maps Overlays

Overlays are graphical elements that you can add to a Google Map to represent data or highlight specific areas. Google Maps supports the following overlay types:

- **Markers**: Represent points of interest.
- **Polygons**: Represent closed shapes with multiple vertices.
- **Polylines**: Represent lines with multiple vertices.
- **Circles**: Represent circular areas.
- **Rectangles**: Represent rectangular areas.
- **InfoWindows:** Display information about a specific location when a marker is clicked.
- **GroundOverlay:** Displays an image or a video over a defined geographical area.
- **Custom Overlays:** Allows developers to create their own custom overlays using HTML elements and JavaScript.

---

## Markers

Markers are used to represent points of interest on the map. Google Maps provides two types of markers:
- **Basic Marker**: The traditional marker with a default icon.
- **AdvancedMarkerElement**: A customizable marker with support for custom icons and content.

### Basic Marker

Here’s how to create a basic marker:

```typescript
import React, { useEffect, useRef } from "react";

const BasicMarker: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Add a basic marker
      new google.maps.Marker({
        position: { lat: 37.7749, lng: -122.4194 },
        map: mapRef.current,
        title: "San Francisco",
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default BasicMarker;
```

---

### Custom Marker with `AdvancedMarkerElement`

The `AdvancedMarkerElement` allows you to create highly customizable markers. Here’s an example:

```typescript
import React, { useEffect, useRef } from "react";

const CustomMarker: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Create a custom marker
      const pinElement = document.createElement("div");
      pinElement.innerHTML = "📍"; // Custom icon
      pinElement.style.fontSize = "24px";

      new google.maps.marker.AdvancedMarkerElement({
        position: { lat: 37.7749, lng: -122.4194 },
        map: mapRef.current,
        content: pinElement,
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default CustomMarker;
```

---

## Polygons

Polygons are used to represent closed shapes with multiple vertices. You can customize their appearance and handle user interactions.

### Drawing a Polygon

Here’s how to draw a polygon:

```typescript
import React, { useEffect, useRef } from "react";

const PolygonOverlay: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Define the polygon's path
      const polygonCoords = [
        { lat: 37.7749, lng: -122.4194 },
        { lat: 37.7849, lng: -122.4294 },
        { lat: 37.7949, lng: -122.4094 },
      ];

      // Draw the polygon
      new google.maps.Polygon({
        paths: polygonCoords,
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.35,
        map: mapRef.current,
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default PolygonOverlay;
```

---

### Handling Polygon Events

You can handle events like `click`, `mouseover`, and `mouseout` on polygons:

```typescript
const polygon = new google.maps.Polygon({
  paths: polygonCoords,
  strokeColor: "#FF0000",
  strokeOpacity: 0.8,
  strokeWeight: 2,
  fillColor: "#FF0000",
  fillOpacity: 0.35,
  map: mapRef.current,
});

polygon.addListener("click", (event: google.maps.PolyMouseEvent) => {
  console.log("Polygon clicked at:", event.latLng);
});
```

---

## Polylines

Polylines are used to represent lines with multiple vertices. They are useful for drawing routes or paths.

### Drawing a Polyline

Here’s how to draw a polyline:

```typescript
import React, { useEffect, useRef } from "react";

const PolylineOverlay: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Define the polyline's path
      const polylineCoords = [
        { lat: 37.7749, lng: -122.4194 },
        { lat: 37.7849, lng: -122.4294 },
        { lat: 37.7949, lng: -122.4094 },
      ];

      // Draw the polyline
      new google.maps.Polyline({
        path: polylineCoords,
        strokeColor: "#0000FF",
        strokeOpacity: 1.0,
        strokeWeight: 4,
        map: mapRef.current,
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default PolylineOverlay;
```

---

### Handling Polyline Events

You can handle events like `click` on polylines:

```typescript
const polyline = new google.maps.Polyline({
  path: polylineCoords,
  strokeColor: "#0000FF",
  strokeOpacity: 1.0,
  strokeWeight: 4,
  map: mapRef.current,
});

polyline.addListener("click", (event: google.maps.PolyMouseEvent) => {
  console.log("Polyline clicked at:", event.latLng);
});
```

---

## Circles

Circles are used to represent circular areas on the map. You can customize their radius, color, and opacity.

### Drawing a Circle

Here’s how to draw a circle:

```typescript
import React, { useEffect, useRef } from "react";

const CircleOverlay: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Draw the circle
      new google.maps.Circle({
        center: { lat: 37.7749, lng: -122.4194 },
        radius: 1000, // Radius in meters
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.35,
        map: mapRef.current,
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default CircleOverlay;
```

---

### Handling Circle Events

You can handle events like `click` on circles:

```typescript
const circle = new google.maps.Circle({
  center: { lat: 37.7749, lng: -122.4194 },
  radius: 1000,
  strokeColor: "#FF0000",
  strokeOpacity: 0.8,
  strokeWeight: 2,
  fillColor: "#FF0000",
  fillOpacity: 0.35,
  map: mapRef.current,
});

circle.addListener("click", (event: google.maps.MapMouseEvent) => {
  console.log("Circle clicked at:", event.latLng);
});
```

---

## Rectangles

Rectangles are used to represent rectangular areas on the map. You can customize their bounds, color, and opacity.

### Drawing a Rectangle

Here’s how to draw a rectangle:

```typescript
import React, { useEffect, useRef } from "react";

const RectangleOverlay: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Define the rectangle's bounds
      const rectangleBounds = {
        north: 37.7849,
        south: 37.7649,
        east: -122.4094,
        west: -122.4294,
      };

      // Draw the rectangle
      new google.maps.Rectangle({
        bounds: rectangleBounds,
        strokeColor: "#0000FF",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#0000FF",
        fillOpacity: 0.35,
        map: mapRef.current,
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default RectangleOverlay;
```

---

### Handling Rectangle Events

You can handle events like `click` on rectangles:

```typescript
const rectangle = new google.maps.Rectangle({
  bounds: rectangleBounds,
  strokeColor: "#0000FF",
  strokeOpacity: 0.8,
  strokeWeight: 2,
  fillColor: "#0000FF",
  fillOpacity: 0.35,
  map: mapRef.current,
});

rectangle.addListener("click", (event: google.maps.MapMouseEvent) => {
  console.log("Rectangle clicked at:", event.latLng);
});
```

---


## InfoWindows

InfoWindows are used to display additional information about a location on the map. They are typically attached to markers but can also be used independently.

### Adding an InfoWindow

Here’s how to add an InfoWindow to a marker:

```typescript
import React, { useEffect, useRef } from "react";

const InfoWindowExample: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Create a marker
      const marker = new google.maps.Marker({
        position: { lat: 37.7749, lng: -122.4194 },
        map: mapRef.current,
        title: "San Francisco",
      });

      // Create an InfoWindow
      const infoWindow = new google.maps.InfoWindow({
        content: "<h3>San Francisco</h3><p>Welcome to the Golden City!</p>",
      });

      // Attach the InfoWindow to the marker
      marker.addListener("click", () => {
        infoWindow.open(mapRef.current, marker);
      });
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default InfoWindowExample;
```

---

### Handling InfoWindow Events

You can handle events like `closeclick` on InfoWindows:

```typescript
infoWindow.addListener("closeclick", () => {
  console.log("InfoWindow closed");
});
```

---

## GroundOverlay

GroundOverlays are used to overlay images (e.g., maps, satellite imagery) on specific areas of the map.

### Adding a GroundOverlay

Here’s how to add a GroundOverlay:

```typescript
import React, { useEffect, useRef } from "react";

const GroundOverlayExample: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Define the bounds for the GroundOverlay
      const bounds = {
        north: 37.7849,
        south: 37.7649,
        east: -122.4094,
        west: -122.4294,
      };

      // Add a GroundOverlay
      new google.maps.GroundOverlay(
        "https://www.example.com/image.png", // URL of the image
        bounds,
        {
          map: mapRef.current,
          opacity: 0.5, // Set the opacity of the overlay
        }
      );
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default GroundOverlayExample;
```

---

## Custom Overlays

Custom Overlays allow you to create fully custom overlays using HTML, CSS, and JavaScript. This is useful for adding complex UI elements to the map.

### Creating a Custom Overlay

Here’s how to create a custom overlay:

```typescript
import React, { useEffect, useRef } from "react";

class CustomOverlay extends google.maps.OverlayView {
  private div: HTMLElement | null = null;

  constructor(
    private position: google.maps.LatLng,
    private content: string
  ) {
    super();
  }

  onAdd(): void {
    this.div = document.createElement("div");
    this.div.style.position = "absolute";
    this.div.style.backgroundColor = "white";
    this.div.style.border = "1px solid black";
    this.div.style.padding = "10px";
    this.div.innerHTML = this.content;

    const panes = this.getPanes();
    if (panes) {
      panes.overlayLayer.appendChild(this.div);
    }
  }

  draw(): void {
    if (this.div) {
      const projection = this.getProjection();
      const point = projection.fromLatLngToDivPixel(this.position);

      if (point) {
        this.div.style.left = `${point.x}px`;
        this.div.style.top = `${point.y}px`;
      }
    }
  }

  onRemove(): void {
    if (this.div) {
      this.div.parentNode?.removeChild(this.div);
      this.div = null;
    }
  }
}

const CustomOverlayExample: React.FC = () => {
  const mapRef = useRef<google.maps.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current) {
      mapRef.current = new google.maps.Map(document.getElementById("map") as HTMLElement, {
        center: { lat: 37.7749, lng: -122.4194 },
        zoom: 12,
      });

      // Create a custom overlay
      const overlay = new CustomOverlay(
        new google.maps.LatLng(37.7749, -122.4194),
        "<h3>Custom Overlay</h3><p>This is a custom overlay!</p>"
      );

      // Add the overlay to the map
      overlay.setMap(mapRef.current);
    }
  }, []);

  return <div id="map" style={{ height: "100vh", width: "100%" }} />;
};

export default CustomOverlayExample;
```

## References:

### Setting Up Google Maps in React
Before diving into overlays, ensure you have set up the Google Maps JavaScript API in your React application. Follow the steps in the [Google Maps Documentation](https://developers.google.com/maps/documentation/javascript/tutorial) to obtain your API key and include it in your project.