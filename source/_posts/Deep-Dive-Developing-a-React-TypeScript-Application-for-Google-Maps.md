---
title: Deep Dive Developing a React TypeScript Application for Google Maps
date: 2025-01-14 17:42:46
categories:
- ReactJS
- TypeScript
- GIS
tags:
- ReactJS
- TypeScript
- GIS
---


## Choosing the Right React Library for Google Maps

There are two popular libraries for integrating Google Maps into React applications:

### 1. [react-google-maps-api](https://github.com/JustFly1984/react-google-maps-api)
#### Features:
- Built with TypeScript support out of the box.
- Highly optimized for performance.
- Provides detailed type definitions for Google Maps API objects and events.
- Supports SSR (Server-Side Rendering) and lazy loading of the Google Maps API.

#### Pros:
- Excellent TypeScript support.
- Well-maintained and frequently updated.
- Extensive documentation and examples.

#### Cons:
- Requires a steeper learning curve due to its focus on optimization and customization.

### 2. [@react-google-maps/api](https://visgl.github.io/react-google-maps/)
#### Features:
- A modern, lightweight alternative to `react-google-maps-api`.
- Provides a simple and intuitive API for Google Maps integration.
- Supports TypeScript.
- Optimized for performance with lazy loading and minimal re-renders.

#### Pros:
- Lightweight and fast.
- Easy to use with minimal boilerplate.
- Excellent TypeScript support.

#### Cons:
- Less mature compared to `react-google-maps-api`.
- Fewer community resources and examples.

### Recommendation:
For most applications, `react-google-maps-api` is the preferred choice due to its robust TypeScript support and active development. However, for simpler projects, `@react-google-maps/api` may be sufficient.

---

## Key Considerations for Google Maps Event Handling

### 1. Event Compatibility
Google Maps provides different event types for various objects. Ensure the event you are attaching is supported by the target object. For example:
- **Polygon** and **Polyline** support the `rightclick` event.
- **MVCArray** does **not** support `rightclick`, making it challenging to attach such an event directly to a vertex.

#### Tip:
Always consult the [Google Maps JavaScript API documentation](https://developers.google.com/maps/documentation/javascript/reference) to verify event compatibility.

### 2. Modern GUI Concepts
When building interactions, remember:
- Many devices, such as iPads and smartphones, lack traditional mouse functionality.
- Avoid relying on events like `rightclick` for core features. Instead, design user interfaces with touch and gesture interactions in mind.

### 3. Event Attributes
Google Maps events like `MapMouseEvent` or `PolyMouseEvent` lack a `button` attribute to identify the mouse button used. Instead, use the `domEvent` property for this information.

#### Example:
```typescript
polygon.addListener("mousedown", (event: google.maps.PolyMouseEvent) => {
  const mouseEvent = event.domEvent as MouseEvent;
  if (mouseEvent.button === 0) {
    console.log("Left mouse button clicked");
  } else if (mouseEvent.button === 2) {
    console.log("Right mouse button clicked");
  }
});
```

---

## Example: Attaching Event Listeners to Map Objects
Below is a React component for managing drawing tools and attaching event listeners to polygons and polylines.

```typescript
import React, { useEffect, useRef, useCallback, useState } from "react";

const addPathListeners = (path: google.maps.MVCArray<google.maps.LatLng>) => {
  path.forEach((_, index) => {
    google.maps.event.addListener(path, "mouseover", () => {
      console.log(`Mouse over vertex ${index}`);
      document.body.style.cursor = "pointer";
    });

    google.maps.event.addListener(path, "mouseout", () => {
      document.body.style.cursor = "default";
    });
  });
};

const attachListenersToEditablePolygon = (polygon: google.maps.Polygon) => {
  const path = polygon.getPath();

  addPathListeners(path);

  google.maps.event.addListener(path, "insert_at", () => addPathListeners(path));
  google.maps.event.addListener(path, "remove_at", () => addPathListeners(path));
  google.maps.event.addListener(path, "set_at", () => addPathListeners(path));
};

const MapDrawingManager: React.FC<{ mapRef: React.RefObject<google.maps.Map | null> }> = ({ mapRef }) => {
  const drawingManagerRef = useRef<google.maps.drawing.DrawingManager | null>(null);

  const handlePolygonComplete = useCallback((polygon: google.maps.Polygon) => {
    polygon.setOptions({
      fillColor: "#0000FF80",
      strokeColor: "#0000FF",
      strokeWeight: 4,
      editable: true,
    });

    attachListenersToEditablePolygon(polygon);
  }, []);

  useEffect(() => {
    if (mapRef.current && !drawingManagerRef.current) {
      const manager = new google.maps.drawing.DrawingManager({
        drawingMode: google.maps.drawing.OverlayType.POLYGON,
        drawingControl: true,
      });

      google.maps.event.addListener(manager, "polygoncomplete", handlePolygonComplete);

      manager.setMap(mapRef.current);
      drawingManagerRef.current = manager;

      return () => {
        manager.setMap(null);
        drawingManagerRef.current = null;
      };
    }
  }, [mapRef, handlePolygonComplete]);

  return null;
};

export default MapDrawingManager;
```

---

## Migrating from `google.maps.marker` to `advancedmarker`

Google has deprecated the `google.maps.marker` class in favor of the new `advancedmarker` class. The `advancedmarker` class offers improved performance and additional features, making it the recommended choice for new projects.

### Key Differences:
- **Performance**: `advancedmarker` is optimized for better performance, especially in applications with a large number of markers.
- **Features**: `advancedmarker` supports advanced features like custom HTML content and improved event handling.

### Migration Steps:
1. **Update Imports**: Replace `google.maps.marker` with `google.maps.advancedmarker`.
2. **Refactor Code**: Update your code to use the new `advancedmarker` API. This may involve changes to how you create, configure, and interact with markers.
3. **Test Thoroughly**: Ensure that your application works as expected after the migration.

#### Example:
```typescript
// Old way using google.maps.marker
const marker = new google.maps.Marker({
  position: { lat: -34.397, lng: 150.644 },
  map: mapRef.current,
});

// New way using google.maps.advancedmarker
const advancedMarker = new google.maps.advancedmarker.AdvancedMarker({
  position: { lat: -34.397, lng: 150.644 },
  map: mapRef.current,
});
```

---

## Best Practices
1. **Design for Touch-First:** Avoid interactions relying on mouse-specific events like `rightclick`.
2. **Event Verification:** Confirm the compatibility of events with the target object.
3. **TypeScript Usage:** Leverage TypeScript for better type safety and developer experience.
4. **Code Modularity:** Organize logic into reusable functions, such as `addPathListeners`.
5. **Migrate to `advancedmarker`:** Update your code to use the new `advancedmarker` class for better performance and features.

