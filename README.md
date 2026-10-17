# JKT48 Sousenkyo General Collection 2026 (fan-made)

React + Vite + Firebase Authentication (anonymous) + Cloud Firestore + callable Cloud Function. This is not affiliated with JKT48 Operation Team.

## Features
- Responsive pink/pastel UI, navigation, search, filter, member cards, ranking, top 3, schedule and WIB countdown.
- Member data lives in `src/members.js`. Names are a starter transcription of the official directory listing; profile nickname/team/photo are deliberately left unfilled rather than guessed. Verify the entire list against https://jkt48.com/member/detail/ before presenting it as a fully verified active roster. The page uses graceful placeholders until `photoUrl` is filled with a properly licensed/authorized photo.
- Votes are stored centrally by a callable Cloud Function using a Firestore transaction. Client rules deny vote writes. A UID can vote once per member; anonymous auth is not a perfect anti-fraud mechanism because sessions can be lost/recreated.
- The backend accepts votes only on 19, 21 and 23 October 2026 in WIB. Each window runs for the whole calendar day in WIB.
- Before Firebase config/deployment, the site explicitly stays in preview mode; no vote is stored and no fake vote numbers are shown.

## Setup & publish (Firebase free tier availability/quotas can change)
1. Install Node.js 20+ and Firebase CLI: `npm install -g firebase-tools`.
2. Create a Firebase project at https://console.firebase.google.com/ . Add a Web app, enable **Authentication > Sign-in method > Anonymous**, and create a **Cloud Firestore** database.
3. Copy `.env.example` to `.env` and fill the Web app config values from Firebase Project Settings. These are client config identifiers; do not place service-account keys in the frontend.
4. Install dependencies: `npm install` and `cd functions && npm install && cd ..`.
5. Sign in and select project: `firebase login` then `firebase use --add`.
6. Deploy backend rules and function: `firebase deploy --only firestore:rules,functions`. Cloud Functions may require a billing-enabled Blaze plan in your project; check current Firebase terms and costs before deploying. Set a budget alert.
7. Build and publish hosting: `npm run build` then `firebase deploy --only hosting`.
8. Test in two browsers/devices. Votes should show up on both only after deployment and when the voting date window is open. Test security rules in Firebase console before public promotion.

## Updating roster
Edit `src/members.js`. If adding/removing/reordering a member, keep the IDs stable and update the exact same allowed IDs in `functions/index.js`. IDs are deliberately stable identifiers, not derived from names. Confirm active status, spelling, team, nickname, and photo rights using the official profile pages. If you add a verified photo, set `photoUrl` to its authorized HTTPS URL; otherwise keep it blank. `membersLastUpdated` should reflect the real review date.

## Security notes
- Never deploy open client write rules for `voteCounts` or `votes`.
- Callable function uses Firebase Auth, member ID allowlist, time-window validation, idempotent vote document IDs, and Firestore transaction. App Check is not enforced in this starter; configure/review App Check before wide public use. No client-side system can promise perfect anti-fraud.
- One anonymous UID gets one vote per member. This is not equivalent to one vote per real person.
- Aggregate totals are public. Individual ballot records are not readable by clients.
- The vote-count listener currently uses `orderBy('memberId')`; every count document written by the function includes that field.
