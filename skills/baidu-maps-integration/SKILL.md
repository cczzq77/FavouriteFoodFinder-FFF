---
name: baidu-maps-integration
description: Integration with Baidu Maps API for geolocation, geocoding, POI search, and interactive map generation. Used by the Favourite Food Finder expert.
skill_type: integration
version: 1.0.0
---

# Baidu Maps Integration Skill

Integrates Baidu Maps services for location-based food discovery. Provides geolocation, geocoding, POI search, distance calculation, and interactive map generation.

## Supported APIs

### 1. IP 定位 API
- URL: `https://api.map.baidu.com/location/ip?ak={AK}&coor=bd09ll`
- Returns approximate location based on IP address.
- Response: `{"address": "CN|北京|北京市|None|CHINANET|1|None", "content": {"address": "北京市", "address_detail": {"city": "北京市", "city_code": 131, "province": "北京市", "street": "", "district": "", "street_number": ""}, "point": {"x": "116.40387397", "y": "39.91488908"}}}`

### 2. 地理编码 API
- URL: `https://api.map.baidu.com/geocoding/v3/?address={地址}&output=json&ak={AK}`
- Converts address to coordinates (bd09ll).
- Response: `{"status": 0, "result": {"location": {"lng": 116.404, "lat": 39.915}, "precise": 1, "confidence": 80, "level": "门址"}}`

### 3. 逆地理编码 API
- URL: `https://api.map.baidu.com/reverse_geocoding/v3/?location={lat},{lng}&output=json&ak={AK}`
- Converts coordinates to address.

### 4. 地点搜索 API
- URL: `https://api.map.baidu.com/place/v2/search?query={关键词}&location={lat},{lng}&radius={半径}&output=json&ak={AK}`
- Search for POIs within a radius.

## Distance Calculation

Use the **Haversine formula** for accurate Earth-surface distance calculation:

```python
import math

def haversine_distance(lat1, lng1, lat2, lng2):
    R = 6371000  # Earth radius in meters
    phi1 = math.radians(lat1)
    phi2 = math.radians(lat2)
    dphi = math.radians(lat2 - lat1)
    dlambda = math.radians(lng2 - lng1)
    a = math.sin(dphi/2)**2 + math.cos(phi1) * math.cos(phi2) * math.sin(dlambda/2)**2
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1-a))
    return R * c  # distance in meters
```

## Map HTML Template

The skill includes an interactive map template using Baidu Maps JavaScript API GL.

Template location: `templates/map.html`

Key features of the map template:
- Auto-center on user location
- Draw 1km radius circle
- Mark favorited restaurants within/outside radius with different icons
- Info windows on click showing restaurant details
- Sidebar listing matched restaurants sorted by distance

## Baidu Maps Coordinate System

Baidu Maps uses **BD-09** coordinate system (baidu encrypted coordinates).
- Do NOT mix WGS-84 (GPS) or GCJ-02 (AMap/Tencent) coordinates directly.
- Use Baidu's coordinate conversion API if needed: `https://api.map.baidu.com/geoconv/v1/?coords={lng},{lat}&from=1&to=5&ak={AK}`

## AK Configuration

**IMPORTANT — Two AKs are required**, because Baidu separates JS API (browser) from Web Service APIs (server):

| Application | Type | Purpose | Services Needed |
|-------------|------|---------|-----------------|
| App A | **Browser** | Map display (JS API GL) | JavaScript API, JS Geocoding, etc. |
| App B | **Server** | Backend API calls (Geocoding, Place Search) | Geocoding, Place Search, IP Positioning |

Users must obtain two AKs from https://lbsyun.baidu.com/:
1. Register and login at Baidu Maps Open Platform
2. **Create App A (Browser)** — select "Browser" type
   - Enable: JavaScript API GL and any JS-prefixed services
   - Referer whitelist: `*` (for local development)
3. **Create App B (Server)** — select "Server" type
   - Enable: Geocoding API, Place Search API, IP Positioning (if available)
   - IP whitelist: `0.0.0.0/0`
4. Store both AKs — browser AK for HTML map pages, server AK for backend API calls.
