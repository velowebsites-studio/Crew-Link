# Crew-Link

CrewLink is a real-time car meet & cruise map, inspired by the free-roam maps
in NFS/Forza — see where other racers are live, chat with the crew, drop
meetup pins, and draw race routes that everyone on the map can see and
follow.

It's a single-file app (`index.html`) built with Tailwind CSS, Leaflet, and
Firebase (Auth + Firestore) for realtime sync.

## Features

- **Live driver locations** — everyone's position updates on the map in
  real time via Firestore.
- **Crew chat** — a shared chat room for the whole map.
- **Meetup pins** — drop gas station / meetup / checkpoint / hazard pins
  that everyone can see.
- **Race routes** — click "Draw Route" (or the route icon on the map),
  tap the map to lay down waypoints, then save it. The route is drawn as a
  polyline with start/finish flags and is visible to everyone on the map so
  a crew can follow the same course.

## Setup

CrewLink needs a Firebase project to sync data between racers. Without one,
the app will still load and show the map, but nothing will sync between
browsers.

1. Create a project at [firebase.google.com](https://firebase.google.com).
2. In **Authentication**, enable the **Anonymous** sign-in provider.
3. In **Firestore Database**, create a database (production mode is fine).
4. In **Project Settings -> General -> Your apps**, add a Web app and copy
   its config object.
5. Open `index.html`, find the `firebaseConfig` object near the bottom of
   the file (in the `<script type="module">` block), and paste your values
   in place of the `YOUR_...` placeholders.
6. Deploy the security rules in [`firestore.rules`](./firestore.rules) to
   your project (Firebase Console -> Firestore Database -> Rules, or via
   the Firebase CLI: `firebase deploy --only firestore:rules`).
7. Open `index.html` in a browser (or serve it with any static file host).

If the config is left as placeholders, the app shows a banner explaining
what's missing instead of crashing.

## Data model

All app data lives under `artifacts/{appId}/public/data/...` in Firestore:

- `locations/{uid}` — each driver's latest lat/lng, name, car, color.
- `chat/{msgId}` — crew chat messages.
- `pins/{pinId}` — meetup/checkpoint/hazard pins.
- `routes/{routeId}` — race routes: title, notes, color, ordered list of
  `{lat, lng}` waypoints, and distance in miles.
