# Crew-Link

CrewLink is a real-time car meet & cruise map, inspired by the free-roam maps
in NFS/Forza — see where other racers are live, chat with the crew, drop
meetup pins, and draw race routes that everyone on the map can see and
follow.

It's a single-file app (`index.html`) built with Tailwind CSS, Leaflet, and
Firebase (Auth + Firestore) for realtime sync.

## Features

- **Email/password login** — racers create an account or sign in before
  they can see anything. Live locations, chat, pins, and routes are only
  visible to signed-in users, both in the UI and enforced by the Firestore
  security rules.
- **Live driver locations** — everyone's position updates on the map in
  real time via Firestore.
- **Crew chat** — a shared chat room for the whole map.
- **Meetup pins** — drop gas station / meetup / checkpoint / police-hazard
  pins that everyone can see.
- **Real gas stations** — the gas pump button on the map queries
  [OpenStreetMap's Overpass API](https://overpass-api.de/) for actual fuel
  stations within 5 miles of you and drops them on the map live (not
  stored in Firestore, refetched on demand). Coverage/accuracy depends on
  OpenStreetMap's community data for your area.
- **Race routes** — click "Draw Route" (or the route icon on the map),
  tap the map to lay down waypoints, then save it. Waypoints are snapped to
  actual roads via the [OSRM](https://project-osrm.org/) routing API (falls
  back to a straight line between points if that service is unreachable),
  drawn as a polyline with start/finish flags, and visible to everyone on
  the map so a crew can follow the same course.
- **Delete your own pins & routes** — a "Delete" button shows up on any
  pin or route you created (in the sidebar list and in its map popup) so
  you can clean up meetups/routes that are no longer relevant. Only the
  original author can delete their own pin or route — the Firestore
  security rules enforce this server-side, not just in the UI.

## Setup

CrewLink needs a Firebase project to sync data between racers. Without one,
the app shows a "Firebase Not Configured" banner instead of the login
screen.

1. Create a project at [firebase.google.com](https://firebase.google.com).
2. In **Authentication**, enable the **Email/Password** sign-in provider
   (Build -> Authentication -> Sign-in method -> Email/Password -> Enable).
3. In **Firestore Database**, create a database (production mode is fine).
4. In **Project Settings -> General -> Your apps**, add a Web app and copy
   its config object.
5. Open `index.html`, find the `firebaseConfig` object near the bottom of
   the file (in the `<script type="module">` block), and paste your values
   in place of the `YOUR_...` placeholders.
6. Deploy the security rules in [`firestore.rules`](./firestore.rules) to
   your project (Firebase Console -> Firestore Database -> Rules -> paste
   -> Publish, or via the Firebase CLI: `firebase deploy --only firestore:rules`).
7. Open `index.html` in a browser (or serve it with any static file host).
   You'll land on a Sign In / Sign Up screen — create an account to get in.

If the config is left as placeholders, the app shows a banner explaining
what's missing instead of crashing.

## Data model

All app data lives under `artifacts/{appId}/public/data/...` in Firestore:

- `locations/{uid}` — each driver's latest lat/lng, name, car, color.
- `chat/{msgId}` — crew chat messages.
- `pins/{pinId}` — meetup/gas-station/checkpoint/police-hazard pins.
- `routes/{routeId}` — race routes: title, notes, color, the road-snapped
  `points` polyline, the original clicked `waypoints`, `roadSnapped` flag,
  and distance in miles.
