

# Omer 'N Chat

A private real-time chat app with voice/video calls, direct messages, and emoji reactions. Built for a small group of people who want their own space to hang out.

---

## How It Works

### Tech Stack

| Layer | Technology |
|---|---|
| Chat & presence | Firebase Firestore (real-time) |
| Auth | Firebase Authentication |
| Calls | PeerJS / WebRTC (peer-to-peer) |
| Hosting | Any static host (just one HTML file) |

---

### Authentication

The app uses **Firebase Auth with email/password**. Here's the full flow:

1. **Sign Up** — Enter a display name, email, and password. A verification email is sent automatically via Firebase.
2. **Verify** — Click the link in the email. Come back to the app and press "I confirmed it — let me in!". The app calls `user.reload()` then `getIdToken(true)` to force a fresh token, then loads your profile and launches.
3. **Sign In** — Email + password. If your email isn't verified yet, you're sent back to the verify screen.
4. **Forgot Password** — Click "Forgot your password?" on the sign in screen. Enter your email and Firebase sends a reset link.

> **Display names are permanent.** When you first sign up, your name is written to Firestore and locked. There's no way to change it — this is intentional to prevent impersonation.

---

### Real-Time Chat

- Messages are stored in Firestore at `nexus/room1/messages`
- The app listens with `onSnapshot` — new messages appear instantly for everyone
- Messages support **emoji reactions** (stored as arrays on each message document)
- URLs in messages auto-link
- **Typing indicators** are powered by a separate `nexus/room1/typing` collection — your entry appears when you type and is deleted after 3 seconds of inactivity, or when you send/sign out

---

### Direct Messages (DMs)

- DMs are stored at `nexus/room1/dms/{key}/messages` where `key` is the two usernames sorted alphabetically and joined with `__` (e.g. `alice__bob`)
- Click anyone's name in the Online list or in chat to open a DM panel
- Fully private — only the two participants can read it (enforced by Firestore security rules)

---

### Voice & Video Calls

Calls are **fully peer-to-peer** using PeerJS (WebRTC under the hood). No call audio or video ever touches a server.

**How a call works:**

1. When you load the app, a PeerJS connection opens and gets assigned a random **Peer ID**
2. That Peer ID is automatically posted as a card in `#general`
3. Anyone can click 📹 Video or 🎙 Voice on your card to call you
4. You get an incoming call popup — accept and you're connected directly
5. Supports up to **4 people** at once (1 local + 3 remote slots)

**Call features:**
- Mute/unmute mic
- Toggle camera on/off
- Screen sharing via `getDisplayMedia()`
- Live call timer
- Automatic cleanup when someone hangs up

> **Note:** Peer IDs reset every time the page loads. Always grab a fresh ID from `#general` if a call isn't connecting.

---

### Online Presence

- When you sign in, your name and Peer ID are written to `nexus/room1/presence`
- A heartbeat runs every 20 seconds to keep your entry fresh
- On sign-out or page close (`beforeunload`), your presence entry is deleted
- Everyone's online list updates in real time via `onSnapshot`

---

### File Structure

This is a **single HTML file** — no build step, no npm, no framework.

```
index.html   ← the entire app (HTML + CSS + JS)
```

Just open it in a browser or drop it on any static host (Netlify, GitHub Pages, Vercel, etc.).

---

### Firebase Setup

The app connects to a Firebase project with the following services enabled:

- **Authentication** — Email/Password provider
- **Firestore** — Database for messages, presence, DMs, typing, and user profiles

The config is hardcoded in the script block at the bottom of `index.html`. To use your own Firebase project, replace the `FBCFG` object with your own credentials from the Firebase Console.

---

### Known Limitations

- Peer IDs are ephemeral — they reset on every page reload
- No message editing or deletion
- No push notifications
- Firestore security rules should be tightened before sharing with untrusted users
- WebRTC calls may not work on some school or corporate networks that block UDP traffic
