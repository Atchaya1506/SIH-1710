# Smart India Hackathon Workshop
# Date: 16/03/2026
## Register Number: 212224220015
## Name: ATCHAYA B
## Problem Title
SIH 1710: Enhancing Navigation for Railway Station Facilities and Locations
## Problem Description
Background: Railway stations are complex environments with numerous facilities and locations such as ticket counters, platforms, restrooms, food courts, and waiting areas. Passengers often face difficulties in navigating these spaces, especially in large or unfamiliar stations. Efficient and user-friendly navigation systems are crucial for improving passenger experience, reducing congestion, and ensuring timely travel connections. Description: The problem involves developing a comprehensive navigation solution for railway stations that assists passengers in locating various facilities and destinations within the station premises. This includes creating detailed maps, providing real-time directions, and integrating features such as accessibility options for individuals with disabilities. The solution should be intuitive, easy to use, and accessible via multiple platforms, including mobile devices and digital kiosks. Key challenges include updating navigation information in real-time, ensuring accuracy, and accommodating the diverse needs of all passengers. Expected Solution: The expected solution is a multi-platform navigation system that provides detailed, real-time directions to all facilities and locations within a railway station. This system should include: A mobile application with 3D interactive maps and step-by-step navigation. Digital kiosks located throughout the station with touch-screen interfaces. Voice-guided navigation for visually impaired passengers. Regular updates to reflect changes in station layout and facility locations. Integration with existing railway apps and services for seamless user experience. The solution should enhance the overall passenger experience by reducing confusion, saving time, and improving accessibility within the station.

## Problem Creater's Organization
Ministry of Railway

## Idea

Solution: StationAR — a phone-based Augmented Reality navigation system that overlays live directional arrows, distance counters, and facility labels directly onto the passenger's camera view. No map-reading required. Just point your phone and follow the arrows.

Core Innovation: 

Unlike outdoor AR navigation (Google Maps Live View), StationAR works entirely indoors using BLE beacon trilateration instead of GPS. It is the first AR navigation system designed specifically for Indian railway stations, with PNR-aware coach positioning and 12-language voice guidance built in.

## Proposed Solution / Architecture Diagram

StationAR is a lightweight mobile module within the RailNav 360° app. It activates automatically when the passenger enters the station geofence and consists of five core components:

Component 1 — Indoor Positioning Engine:
BLE beacons installed every 10–15 metres across the station broadcast unique signals. The app receives signal strength (RSSI) from at least 3 beacons simultaneously and uses trilateration to calculate the passenger's exact X,Y position on the station floor plan with 1–2 metre accuracy. When beacon signals are weak (crowded areas), the IMU Dead Reckoning fallback activates — using the phone's accelerometer and gyroscope to track movement and maintain position accuracy.

Component 2 — AR Overlay Engine:
Built on ARCore (Android) and ARKit (iOS), the engine anchors 3D virtual arrows, distance labels, and facility icons to real-world coordinates. As the passenger walks, the overlay updates in real-time using the phone's camera, gyroscope, and beacon position data combined. Arrows smoothly animate to indicate direction. At each turn point, a large animated turn indicator appears with a voice prompt.

Component 3 — PNR-Aware Route Planner:
When the passenger scans their PNR or enters their train number, the system fetches live data from the NTES API to determine the current platform assignment and coach stop position. The AR route is automatically set to guide the passenger to their exact coach door — not just the platform. If the platform changes, a push notification fires instantly and the AR route recalculates in under 3 seconds.

Component 4 — AI Crowd-Aware Pathfinding:
The route engine uses Dijkstra + A* algorithm on the station's IndoorGML graph. Before finalising the route, it queries the live crowd heatmap (generated from CCTV + YOLOv8 model) and selects the least-congested valid path. If a passage becomes congested mid-journey, the app re-routes the passenger automatically.

Component 5 — Offline-First Architecture:
On first launch, the station's IndoorGML floor map, beacon map, and facility data are downloaded and cached locally. Core AR navigation works with zero internet connection. Only real-time crowd data and NTES platform updates require connectivity — and even these degrade gracefully to last-known data when offline.

## Architecture Diagram
```
[ Passenger Phone ]
        |
        | Camera + IMU + BLE scan
        |
[ Indoor Positioning Engine ]
   BLE Trilateration + IMU Dead Reckoning
   Output: X, Y coordinates (1–2m accuracy)
        |
        | Position data
        |
[ Route Engine (Cloud + On-device) ]
   Dijkstra + A* on IndoorGML graph
   + AI Crowd Heatmap query (YOLOv8)
   + NTES API (live platform data)
   Output: Optimised node path
        |
        | Path nodes + turn instructions
        |
[ AR Overlay Engine ]
   ARCore (Android) / ARKit (iOS)
   Anchors 3D arrows to real-world coords
   Real-time update as passenger moves
        |
        | Camera view + AR objects
        |
[ Passenger Screen ]
   Live camera + glowing arrows
   Distance labels + facility icons
   Voice guidance (Bhashini TTS)
   Haptic turn alerts
        |
        | Background sync
        |
[ Cloud Backend ]
   NTES API ←→ Platform change alerts
   Admin Portal ←→ Map updates (60s sync)
   Crowd Model ←→ Heatmap feed
```
## Use Cases

Use Case 1 — First-time passenger at an unfamiliar station:
A passenger arriving at Chennai Central for the first time opens the app, scans their PNR, and activates AR mode. The camera view immediately shows a glowing arrow pointing toward Platform 9 with "320 metres" displayed. Voice guidance in Tamil says "Walk straight for 300 metres, then turn left at the footbridge." The passenger reaches their coach position without stopping once to read a signboard.

Use Case 2 — Last-minute platform change:
A passenger is already walking toward Platform 4 when NTES pushes a platform change to Platform 11. The app vibrates, displays a red alert overlay on the camera view, and instantly recalculates the route. The new AR arrows appear within 3 seconds showing the revised path. The passenger avoids the wrong platform entirely.

Use Case 3 — Elderly passenger who cannot read English signboards:
A 68-year-old passenger from a rural district sets the app language to Telugu. All AR labels, voice prompts, and distance indicators appear in Telugu. The route automatically avoids stairs and selects elevator + ramp paths. Font size is increased to accessibility mode automatically based on phone accessibility settings.

Use Case 4 — Passenger needing accessible route:
A wheelchair user opens the app and the system detects accessibility preference. The route engine filters out all stair segments and selects only elevator and ramp-accessible corridors. The AR overlay highlights accessible paths in a distinct colour and alerts the passenger if an elevator is out of service (fed from admin portal).

Use Case 5 — Finding a facility quickly between trains:
A passenger with a 20-minute layover needs the nearest food court and restroom. They tap "Quick Find" in the app. AR overlays appear simultaneously showing the nearest restroom (40m, left) and food court (90m, straight). The passenger makes both stops and returns to the platform with time to spare.

Use Case 6 — No internet connectivity zone:
A passenger enters a basement passage where mobile data drops to zero. The app switches automatically to offline mode — beacon positioning continues, IMU dead reckoning fills gaps, and the locally cached floor map keeps AR arrows accurate. The passenger experiences no interruption in navigation.

## Technology Stack

Mobile App — Flutter (Android + iOS)
AR Engine — ARCore (Android) / ARKit (iOS)
Indoor Positioning — BLE Beacon trilateration + Kalman Filter IMU fallback
Map Format — IndoorGML (ISO standard)
Routing — Dijkstra + A* algorithm
AI Crowd Detection — YOLOv8 on CCTV feed
Voice Guidance — Bhashini API (12 Indian languages)
Backend — Node.js + FastAPI (Python)
Database — PostgreSQL + PostGIS, MongoDB, InfluxDB
Real-time — Redis pub/sub + Firebase FCM
Offline — SQLite on-device cache
Cloud — AWS / NIC Cloud, Docker + Kubernetes


## Dependencies

Hardware Dependencies:

BLE beacons (Estimote or Kontakt brand) installed by Indian Railways at 10–15 metre intervals across all station floors, platforms, concourses, and underground passages. Estimated 200–400 beacons per large station
Station Wi-Fi network (RAILWIRE already deployed at 6000+ stations) for real-time sync
Existing CCTV network access (via COIS) for crowd heatmap model input
No special hardware required on passenger side — any Android 7+ or iOS 12+ smartphone works

Data and API Dependencies:

NTES real-time train and platform data API from Indian Railways
Station CAD floor plan files (AutoCAD format) from respective Railway zones for conversion to IndoorGML
Bhashini API access from Government of India for multilingual TTS
IRCTC PNR API for coach and train data

Permission and Compliance Dependencies:

Railway Board MoU for beacon installation and station deployment
CCTV data access agreement with RPF / Railway Security
Digital Personal Data Protection (DPDP) Act 2023 compliance — no passenger location data stored on server, all positioning processed on-device
Android and iOS app store approval

Government Infrastructure Dependencies:

RAILWIRE Wi-Fi for connectivity
NIC Cloud for data residency compliance (sensitive railway data must stay on Indian servers)
Bhashini (MeitY) for language support — already a government initiative, free API tier available
