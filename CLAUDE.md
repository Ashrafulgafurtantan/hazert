# CLAUDE.md

## 📋 Project Overview

**HazERT** is a TanStack Start-based flood alert and water level monitoring web application focused on the Norfolk, Virginia area. The application provides real-time and historical tide/water level data visualization through interactive maps, charts, and WMS (Web Map Service) layers from GeoServer.

### Key Capabilities
- **Interactive Maps**: Multi-layer mapping with Google Maps base layers and WMS overlay data
- **Side-by-Side Comparison**: Compare different DEM (Digital Elevation Model) datasets
- **Tide Monitoring**: Real-time station data with interactive charts and historical trends
- **Water Level Visualization**: ApexCharts-powered tide data with historical vs predicted data
- **Station Management**: Filterable monitoring sites with categorized station types
- **Interactive Station Points**: Click-to-query GeoServer point layers with modal data display

## 🏗️ Architecture

### Technology Stack

- **Framework**: TanStack Start (full-stack React with file-based routing)
- **Routing**: TanStack Router with file-based routes in `src/routes/`
- **Data Fetching**: TanStack Query for server state management
- **Maps**: React Leaflet with Leaflet.js for interactive mapping
- **Charts**: ApexCharts/React-ApexCharts for data visualization
- **WMS Integration**: Leaflet WMS layers connecting to Norfolk GeoServer
- **Mock API**: MSW (Mock Service Worker) for development data
- **Styling**: Tailwind CSS for component styling
- **Icons**: Lucide React for consistent iconography
- **TypeScript**: Full type safety with custom interfaces

### Core Data Flow

1. **Map Layer Management**: WMS layers from Norfolk GeoServer (`http://202.4.127.189:5459/geoserver/wms`)
2. **Station Data**: Mock tide stations with real-time values via MSW handlers
3. **Tide Data**: Time-series water level data with historical/predicted classifications
4. **Layer Comparison**: Side-by-side visualization using leaflet-side-by-side plugin
5. **Interactive Points**: GeoServer GetFeatureInfo requests for real-time station data on map clicks

## 🗂️ File Structure

```
src/
├── components/
│   ├── ui/
│   │   ├── CompareMap.tsx              # Side-by-side map comparison
│   │   ├── BrushChart.tsx              # ApexCharts brush/zoom component
│   │   └── TideMonitoringSiteCategories.tsx  # Station filtering sidebar
│   ├── TideChart.jsx                   # Main tide data chart component
│   ├── DefaultCatchBoundary.tsx        # Error boundary component
│   ├── NotFound.tsx                    # 404 component
│   └── DebugMSW.jsx                    # MSW debugging component
├── hooks/
│   ├── useMapLayers.ts                 # Map data fetching hooks
│   ├── useStations.js                  # Station data management
│   └── useTideData.js                  # Tide data fetching
├── routes/
│   ├── __root.tsx                      # Root layout with QueryClient
│   ├── index.tsx                       # Main map with WMS layers
│   ├── compare-map.tsx                 # Side-by-side comparison route
│   ├── water-level.tsx                 # Tide monitoring interface
│   └── temporary.tsx                   # Temporary/test route
├── types/
│   └── map.ts                          # TypeScript interfaces for map data
├── mocks/
│   ├── handlers.js                     # MSW API handlers
│   └── browser.js                      # MSW browser setup
├── integrations/
│   └── tanstack-query/
│       └── root-provider.tsx           # QueryClient configuration
├── utils/
│   └── seo.ts                          # SEO meta tag utilities
└── styles/
    └── app.css                         # Global styles
```

## 🔑 Key Files & Components

### Map Components

#### `src/routes/index.tsx` - Main Map Interface
- **Purpose**: Primary flood alert map with WMS layers and base layer switching
- **Features**:
  - Norfolk DEM raster data overlay (`NorflokDEM10m_Prj1`, `NorflokDEM10m_Prj2`)
  - Interactive point layer (`noaa_predictions`) with click-to-query functionality
  - Layer visibility controls with toggleable point layer
  - Base layer switcher (Default/Satellite/Terrain)
  - Google Maps base layers
  - Real-time station data modal on point clicks
- **Location**: Norfolk/Moyock area (36.8443205, -76.2820786)

#### `src/components/ui/CompareMap.tsx` - Side-by-Side Comparison
- **Purpose**: Advanced map comparison using leaflet-side-by-side plugin
- **Features**:
  - Drag slider for layer comparison
  - Loading states and error handling
  - Legend component for layer identification
  - Automatic layer switching

#### `src/routes/water-level.tsx` - Tide Monitoring
- **Purpose**: Station-based water level monitoring with charts
- **Features**:
  - Interactive station markers with water level values
  - Station selection and popup details
  - Integrated tide chart display
  - Two-panel layout (map + charts)

### WMS Integration

#### WMS Layers Configuration
```javascript
const WMS_LAYERS = [
  {
    id: 'raster_data_1',
    url: 'http://202.4.127.189:5459/geoserver/wms',
    layers: 'flood-app:NorflokDEM10m_Prj1',
    format: 'image/png',
    version: '1.3.0'
  },
  {
    id: 'raster_data_2',
    url: 'http://202.4.127.189:5459/geoserver/wms',
    layers: 'flood-app:NorflokDEM10m_Prj2',
    format: 'image/png',
    version: '1.3.0'
  },
  {
    id: 'raster_geo_point',
    name: 'Raster Geo Point',
    url: 'http://202.4.127.189:5459/geoserver/wms',
    layers: 'flood-app:noaa_predictions',
    format: 'image/png',
    version: '1.3.0'
  }
]
```

### Data Management

#### `src/hooks/useMapLayers.ts` - Map Data Hooks
- **`useMapLayers()`**: Fetches available map layers
- **`useCompareMapData()`**: Loads side-by-side comparison configuration
- **`useMapLayer(layerId)`**: Individual layer data fetching
- **`useStationClick(clickParams)`**: GeoServer GetFeatureInfo requests for interactive point data

#### `src/hooks/useStations.js` - Station Management
- **Purpose**: Loads GeoJSON FeatureCollection of monitoring stations
- **Data Structure**: Norfolk area stations with coordinates and current values

#### `src/hooks/useTideData.js` - Tide Data
- **Purpose**: Time-series tide data with configurable date ranges
- **Features**: Historical vs predicted data classification, retry logic

### Charts & Visualization

#### `src/components/TideChart.jsx` - Tide Data Visualization
- **Technology**: ApexCharts with time-series configuration
- **Features**:
  - Historical vs predicted data series (blue/purple)
  - Smooth monotonic cubic curves
  - Interactive tooltips and zoom
  - Loading states

#### `src/components/ui/BrushChart.tsx` - Advanced Charting
- **Purpose**: Brush/zoom chart component with main + navigator charts
- **Features**: Connected brush selection, gradient fills, synchronized zoom

### API & Mock Data

#### `src/mocks/handlers.js` - MSW API Handlers
- **Endpoints**:
  - `GET /api/stations` - Station FeatureCollection
  - `GET /api/stations/:id/tides` - Time-series tide data
  - `GET /api/map-layers` - Available map layers
  - `GET /api/map-layers/compare` - Comparison configuration
- **Data Generation**: Realistic tide patterns with tidal high/low cycles

## ⚙️ Setup & Configuration

### Development Commands
```bash
# Start development server (port 3000)
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start
```

### Environment Setup
- **Node.js**: Modern ES modules with TypeScript
- **Vite**: Development server with HMR
- **MSW**: Mock service worker for API simulation
- **TanStack Router**: File-based routing with type safety

### Key Configuration Files
- **`vite.config.ts`**: Vite + TanStack Start + React configuration
- **`tsconfig.json`**: TypeScript with path aliases (`~/*`, `@/*`)
- **`package.json`**: Dependencies including Leaflet, ApexCharts, MSW

## 🎯 Key Features

### Map Functionality
1. **Multi-Layer Mapping**: Base layers + WMS overlays
2. **Layer Controls**: Toggle visibility, opacity, z-index management
3. **Interactive Markers**: Station data with custom circular icons
4. **Side-by-Side Comparison**: Leaflet plugin for DEM dataset comparison
5. **Point Layer Interaction**: Click-to-query station points with real-time data modal display

### Alert Systems
1. **Station Monitoring**: 15 Norfolk area stations with real-time values
2. **Visual Indicators**: Color-coded markers based on water levels
3. **Interactive Popups**: Station details and data access
4. **Status Tracking**: Active/inactive station management

### Data Visualization
1. **Time-Series Charts**: Historical and predicted tide data
2. **Brush Navigation**: Zoom and pan through time ranges
3. **Multi-Series Data**: Historical (blue) vs Predicted (purple)
4. **Export Capabilities**: Chart download and selection tools

### Responsive Design
1. **Mobile-Friendly**: Responsive map and chart layouts
2. **Touch Support**: Mobile map interaction
3. **Adaptive UI**: Collapsible sidebars and panels

## 🚀 Development Notes

### Adding New WMS Layers
1. Update `WMS_LAYERS` configuration in route components
2. Add layer metadata to `src/types/map.ts`
3. Update MSW handlers for new layer APIs

### Interactive Point Layers
1. **Click Handler**: Map click events capture coordinates and build GetFeatureInfo requests
2. **GeoServer Integration**: Direct JSON API calls to Norfolk GeoServer WMS endpoints
3. **Modal Display**: Station data displayed in responsive modal with data tables
4. **TypeScript Support**: Full type safety for GeoServer response structures

### Extending Station Data
1. Modify station data in `src/mocks/handlers.js`
2. Update TypeScript interfaces in components
3. Add new chart series or map markers as needed

### Chart Customization
- ApexCharts options in `TideChart.jsx`
- Custom color schemes and themes
- Interactive features and tooltips

This TanStack Start application provides a solid foundation for flood monitoring and water level visualization, with extensible architecture for additional GIS features and real-time data integration. The interactive point layer system enables direct querying of GeoServer data sources with responsive modal interfaces for detailed station information display.