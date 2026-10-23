# DJ Bust A Nut — Music Website

All website files are in this one folder. There are no subfolders to recreate on GitHub.

## Files
- `index.html` — page structure
- `style.css` — responsive dark-purple design
- `script.js` — player, artist login, uploads, and Firebase integration
- `logo.jpeg` — your artist logo
- `README.md` — these setup instructions

## 1. Create Firebase project
1. Visit https://console.firebase.google.com/ and create a project.
2. In Project settings → General → Your apps, add a Web app (`</>`).
3. Copy its Firebase config into `firebaseConfig` at the top of `script.js`, replacing every `PASTE_YOUR_...` value.
4. In Authentication → Sign-in method, enable **Email/Password**.
5. In Authentication → Users, add your own artist account and set a strong password.
6. Create a Cloud Firestore database and a Cloud Storage bucket. Review current Firebase plan, storage, and billing requirements in the Firebase console.

## 2. Firestore security rules
Copy your artist account's UID from Authentication → Users. In Firestore → Rules, replace `YOUR_AUTH_UID` below with that UID and publish:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isArtist() {
      return request.auth != null && request.auth.uid == "YOUR_AUTH_UID";
    }
    match /songs/{songId} {
      allow read: if true;
      allow create: if isArtist() && request.resource.data.ownerUid == request.auth.uid;
      allow update, delete: if isArtist() && resource.data.ownerUid == request.auth.uid;
    }
  }
}
```

## 3. Storage security rules
In Firebase Storage → Rules, replace `YOUR_AUTH_UID` with the same UID and publish:

```text
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    function isArtist() {
      return request.auth != null && request.auth.uid == "YOUR_AUTH_UID";
    }
    match /songs/{userId}/{fileName} {
      allow read: if true;
      allow write: if isArtist() && userId == request.auth.uid
        && (request.resource == null || (request.resource.size < 100 * 1024 * 1024 && request.resource.contentType.matches('audio/.*')));
    }
    match /covers/{userId}/{fileName} {
      allow read: if true;
      allow write: if isArtist() && userId == request.auth.uid
        && (request.resource == null || (request.resource.size < 10 * 1024 * 1024 && request.resource.contentType.matches('image/.*')));
    }
  }
}
```

## 4. Upload to GitHub
Open your repository, choose **Add file → Upload files**, then drag the five files from this folder into the page and commit. Do not upload the folder as a nested folder if you want `index.html` at the repository root.

## 5. Hosting
Firebase Hosting's usual deployment flow uses the Firebase CLI in a terminal. If you use it, initialize Hosting with this folder as the public directory and deploy. On a Chromebook without terminal access, you can keep the code in GitHub and use a compatible browser-based deployment workflow; follow the deployment provider's current setup instructions.

## Important
- Firebase web config is intended to be present in client-side code. Never put your password, private keys, or service-account JSON in these files.
- Replace the UID placeholder in **both** rules sets before making uploads available.
- The website is a starter until the Firebase setup is completed and tested.
- Large audio files can use significant storage/bandwidth and may incur charges. Check your Firebase plan before uploading many songs.
