# Payday Planner

A personal pay-day money planner: split each pay between the house transfer, the bills account, savings and car mileage money, tick bills off as they go, and track what's owed.

- Runs as an installable web app (Android: open in Chrome → ⋮ → **Install app**).
- Sign-in and syncing between devices use Firebase (Email/Password auth + Cloud Firestore).
- Works offline; changes sync when you're back online.

## Files
- `index.html` – the whole app
- `manifest.webmanifest`, `icons/` – install details and icons
- `sw.js` – offline support (bump `VERSION` after changing any file)
- `vendor/` – Firebase JS SDK 10.14.1 (compat build)

## Firestore rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /users/{uid}/{doc=**} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```
