# Product Spec: Live Interview Platform

> **Status**: Draft v2
> **Last Updated**: 2026-06-18

---

## 1. Executive Summary

- **What:** A free, ad-supported, web-based live interviewing platform with pluggable question types.
- **No accounts required:** Both participants join via a link — zero sign-up friction.
- **Core experience:** Real-time collaborative workspace (whiteboard, coding, etc.) + video/audio side by side.
- **MVP question types:** Whiteboard (Basic), Whiteboard (Diagrams), Whiteboard (Excalidraw) — each a separate, independent type.
- **Dual recording:** Video (participants + audio) + JSON (workspace interaction log with periodic snapshots), recorded in sync.
- **Google Drive:** Both recordings upload to the interviewer's Drive — no proprietary storage.
- **Synchronized playback:** Custom in-browser player replays video and workspace replay together, letting reviewers see *how* a candidate reasoned step by step.
- **Revenue:** Non-intrusive in-session banner ads.
- **Future:** Coding, MCQ, and more question types as plugins.

---

## 2. Problem Statement

Hiring teams cannot review *how* a candidate reasoned through a technical problem because existing tools (Zoom + Miro + separate editors) record only video or final results without synchronized workspace state, resulting in shallow evaluations and poor hiring decisions.

Additional constraints:
- **Zero friction required**: Candidates refuse to create accounts or install software for a single interview — teams lose candidates before the interview starts when tools require sign-up.
- **Static review is insufficient**: Screen recordings capture video only. The workspace evolution (whiteboard strokes, code edits, diagram construction) is lost. Reviewers see a static final answer, not the candidate's step-by-step reasoning process.
- **Tool fragmentation**: Interviewers juggle 3–4 separate tools (video call, virtual whiteboard, code editor, screen recorder) with no integration between them.
- **No free alternative exists**: Existing platforms are either paid (CoderPad), limited to one question type, or lack synchronized workspace-and-video recording.

---

## 3. Target Audience

| Segment | Description |
|---------|-------------|
| **Primary** | Growth-stage tech companies (50–500 employees) running structured technical interviews — especially system design and architecture rounds where reasoning process matters most |
| **Secondary** | Any company doing structured interviews (whiteboard, coding, quiz) |
| **Tertiary** | Tutoring, pair programming, or design collaboration sessions |

---

## 4. User Personas

### Persona A: Alex, Senior Engineer / Interviewer

- **Role**: Conducts technical interviews as part of a structured hiring loop
- **Needs**: 
  - Create a room instantly, send link to candidate
  - Choose question types per interview — especially system design whiteboarding for senior IC candidates
  - Collaborative workspace (whiteboard) with candidate
  - Video/audio to communicate
  - Record the entire session (video + workspace interaction log)
  - Upload recordings to Google Drive so the hiring team can review how the candidate reasoned
- **Pain points**: Juggles Zoom + whiteboard tool + screen recording; reviewers miss the reasoning process
- **Device**: Laptop, Chrome

### Persona B: Casey, Candidate

- **Role**: Interview participant
- **Needs**: Click link, join, interact with whatever question type is presented — no account
- **Pain points**: Tired of creating accounts and context-switching between tools
- **Device**: Laptop, any modern browser

### Persona C: Dana, Hiring Manager / Reviewer

- **Role**: Watches recordings later
- **Needs**: See the full interview replay — video of both participants + the workspace (whiteboard, code, etc.) being built step by step — in sync
- **Pain points**: Current recordings miss workspace evolution; sees final state only
- **Device**: Any browser (playback is web-based)

---

## 5. Initial Setup (Bring Your Own Accounts)

The platform does not manage its own infrastructure. Every customer brings their own accounts via OAuth:

**Firebase (mandatory):**
- Customer clicks "Connect Google Account" → Google OAuth consent screen
- Scopes: Firebase Management API, Cloud Storage, Cloud Firestore
- Platform lists the customer's Firebase projects and lets them select one
- Once selected, the platform auto-configures Firestore, Firebase Realtime Database (for CRDT relay), and Firebase Storage, and applies security rules via API
- The connected Firebase project handles: room state, CRDT relay, chat, recording metadata, and workspace JSON storage (paid tier)

**Video provider (optional, mandatory for recording):**
- Customer connects their preferred video provider via OAuth (e.g., Zoom, Google Meet SDK, Microsoft Teams)
- Each provider is a self-contained plugin (same pattern as question types)
- Recording is only available when a video provider is connected

**Setup flow:**
1. Customer signs up on the platform
2. Platform presents a guided setup wizard: Firebase OAuth first, video provider OAuth optional
3. On completion, platform validates the Firebase connection and lists available projects
4. Customer selects which Firebase project to use
5. Platform auto-configures the project (enables Firestore/Storage, applies rules)
6. Customer selects and connects their video provider (e.g., Zoom); validates recording permissions
7. Customer is ready to create interview rooms

**Free vs Paid:**
- Free tier: Firebase required, no video provider needed (P2P WebRTC, no recording)
- Paid tier: Both Firebase and a video provider required (e.g., Zoom)

---

## 6. Functional Requirements

### 6.1 Room Creation & Joining

**FR-01 [P0]: Landing Page**
- Two actions: "Create Interview Room" button and "Join Room" input (paste full link)
- No marketing fluff — get to the room fast

**FR-02 [P0]: Room Creation**
- Click "Create Interview Room"
- Room is created with a unique ID
- Shareable link is generated: `https://domain/room/{roomId}`
- Creator sees the link on screen to copy and share with participants
- Creator can also join via the link like anyone else

**FR-03 [P0]: Joining**
- Open link → "Join Interview" button
- Enter display name (optional, max 128 characters)
- Pick a role: **Interviewer** or **Candidate**
- No account, no sign-up
- No validation for interviewer role — anyone can pick it
- Multiple interviewers and multiple candidates can coexist in the same room (recommended use is 1:1)
- Browser prompts for camera/mic permissions
- Enter room — see workspace area + video tiles
- Name and role shown in participant list

**FR-04 [P1]: Room State & Presence**
- Interviewer capabilities: start/stop recording, end session, upload to Drive
- Candidate capabilities: draw on whiteboard, control own camera/mic, use chat, leave room (self only)
- Each controls their own camera/mic activation
- UI shows: participant joined | participant left | participant role and camera/mic status
- Room is active until both participants leave or interviewer ends the session
- If no questions have been added, the workspace area is empty with a prompt to "Add question"; video tiles remain active so participants can discuss

### 6.2 Question Type System

**FR-05 [P0]: Pluggable Question Type Architecture**
- The workspace area supports multiple questions, each in its own tab
- Each question has its own question type and independent workspace state
- Only interviewers can add questions
- A persistent "+" button at the end of the tab bar opens a picker to select a question type
- Multiple questions of the same type can be added (e.g., two Whiteboard (Basic) tabs)
- When a new question is added, the workspace auto-switches to the new tab
- Each question type defines:
  - A workspace UI component
  - A data model for its state (serializable)
  - A recording format (event log schema)
  - A replay component for playback
- Questions are add-only (no reorder or delete)
- When switching between tabs, the workspace state for that question is preserved
- Future question types can be added without modifying core platform code

**FR-06 [P0]: Current & Planned Question Types**

| Type | MVP? | Workspace | Key Features | Recording Format |
|------|------|-----------|--------------|-----------------|
| **Whiteboard (Basic)** — simple diagramming | ✅ MVP | Fabric.js canvas (shapes, arrows, text) | Rectangles, circles, diamonds, simple arrows, text labels. Color picker (fill + stroke), undo/redo, zoom/pan, export as PNG. Minimal and clean. | All shape add/move/delete/edit events + full state snapshot every 60s |
| | | ⚠️ *Future: Consider replacing with tldraw* | | |
| **Whiteboard (Diagrams)** — smart connector diagrams | ✅ MVP | maxGraph (mxGraph) canvas — the engine behind draw.io | Shape libraries (cloud, DB, server, etc.), smart connector routing, layers & grouping, grid snapping. Same drawing tools as Basic plus professional diagramming features. | All shape/connector add/move/delete/route events + layer/group changes + full state snapshot every 60s |
| | | ⚠️ *Future: Consider replacing with tldraw or React Flow* | | |
| **Whiteboard (Excalidraw)** — hand-drawn / freeform | ✅ MVP | Excalidraw canvas (hand-drawn strokes, freehand) | Hand-drawn freeform strokes, freehand arrows, text labels. Freehand drawing only. Color picker, undo/redo, zoom/pan, export as PNG. Sketch-like aesthetic. | All stroke start/move/end events + text add/edit + full state snapshot every 60s |
| **Coding** (code editor) | Future | In-browser code editor (Monaco / CodeMirror) | Syntax highlighting, language support TBD | Code change events (insert/delete/replace by position) + full source text snapshot every 60s |
| **MCQ / Quiz** | Future | Custom React components | Multiple choice, timer | Answer selection events + timer events |
| **Document / Text** | Future | Rich text / markdown editor | Text formatting | Text insert/delete events + full document snapshot every 60s | |

### 6.3 Live Video/Audio

**FR-07 [P0]: Video/Audio**
- Free tier: P2P WebRTC (direct browser-to-browser) with STUN servers
- Paid tier: Pluggable video provider (e.g., Zoom SDK) via the user's own account — handles video/audio streaming, recording, and NAT traversal
- Provider architecture follows the same plugin pattern as question types: each provider defines its own SDK integration, OAuth flow, and recording interface
- TURN server support configurable for future addition

**FR-08 [P0]: Video Layout**
- Two video tiles positioned on the right side of the workspace by default
- Self-view: small tile
- Remote participant: larger tile above self-view
- Tiles are draggable — users can reposition them anywhere on screen
- When camera is off: show name/initials placeholder

**FR-09 [P0]: Media Controls**
- Camera toggle
- Microphone toggle
- Mute indicator
- End call button (interviewer: ends for both and auto-stops recording if paid tier; candidate: leaves only themselves)

### 6.4 Text Chat

**FR-10 [P1]: In-Call Chat**
- Toggleable chat sidebar
- Any participant can send messages (interviewer or candidate)
- Real-time messaging via Firebase
- Chat history is preserved — late joiners see all previous messages
- Messages are recorded in the workspace JSON with timestamps alongside whiteboard events
- Chat replay is shown in the playback player, synchronized with video and workspace replay
- Blinking animation on chat icon when new message arrives (sidebar minimized or closed)

### 6.5 Recording (Dual Recording)

The recording system is only available on the paid tier. It captures two synchronized data streams plus chat messages interleaved in the workspace JSON:

**FR-11 [P0]: Video Recording** (paid tier only)
- Video/audio recording is handled server-side by the connected video provider (e.g., Zoom)
- Triggered by interviewer's "Record" button (via the provider's recording API)
- Recording indicator shown to both participants
- Interviewer can stop recording at any time

**FR-12 [P0]: Workspace Recording (JSON)** (paid tier only)
- All question workspace actions logged as JSON events with timestamps in a single file
- Chat messages are interleaved in the same file by timestamp
- Each event and snapshot is tagged with its `questionId` to map it to the correct question
- Timestamps are relative to session start (millisecond precision)
- Each question's full state snapshotted every 60 seconds for efficient seeking during playback
- Automatically uploaded to the user's Firebase Storage after session ends

**FR-13 [P0]: Sync Mechanism**
- Both recordings share the same time origin
- Frame-level sync achieved by matching workspace event timestamps to video timecodes from the provider's recording
- When the video shows time `T`, the player applies all workspace events up to `T`

### 6.6 Playback

**FR-14 [P0]: In-Browser Player**
- Custom player component for synchronized replay
- Layout: Workspace replay on the left/center, video player on the right/bottom
- Question tabs are shown above the workspace replay area, matching the interview's questions
- During recording, tab switches are tracked as events so the playback knows which question was active at any point
- By default, playback auto-switches to the question the candidate was viewing at that moment
- Reviewers can override — manually switching tabs pauses auto-follow for that question
- Switching tabs during playback rebuilds the workspace state at the current video time for that question
- Each question renders the replay component corresponding to its recorded type

```
┌──────────────────────┬──────────────┐
│  [Q1] [Q2] [Q3]      │              │
├──────────────────────┤   Video      │
│                      │   Player     │
│   Workspace          │   (provider  │
│   Replay             │   recording) │
│   (per question      │              │
│    type)             │              │
│                      │              │
├──────────────────────┴──────────────┤
│  Play/Pause  │  Scrub Bar   │ Time  │
└─────────────────────────────────────┘
```

**FR-15 [P0]: Playback Controls**
- Play / Pause (controls both video + diagram replay simultaneously)
- Scrub bar for seeking
- Time display (current / total)
- Speed control (1x, 1.5x, 2x)
- Skip forward/backward by 10 seconds

**FR-16 [P1]: Seeking Behavior**
- On seek, snapshots restore the workspace state to the nearest snapshot before the target time
- Then events are fast-forwarded from the snapshot to the exact target time
- This avoids replaying every event from the beginning

**FR-17 [P0]: Playback Entry Points** (paid tier only)
- **Direct playback URL**: `https://domain/playback?provider={providerRecordingId}&workspace={firebaseStoragePath}` — can be shared with the hiring team
- **Recordings list page**: `https://domain/recordings` — queries Firestore for all recording metadata for this Firebase instance and displays a list (date, question types, participants, provider)
- Each entry links to the playback URL
- The page loads the video via the provider's embed player and the workspace JSON from Firebase Storage

### 6.7 Storage (paid tier only)

On the paid tier, the workspace JSON recording is stored in the user's own Firebase Storage. Video/audio is handled by Zoom server-side recording (not stored by us).

**FR-18 [P0]: Upload Flow (Workspace JSON)**
- After session ends, the workspace JSON is automatically uploaded to the user's Firebase Storage bucket
- Path: `recordings/{roomId}/{date}/workspace.json`
- Shows upload progress bar
- Saves recording metadata to Firestore (roomId, date, question types, participants, Firebase Storage path, provider recording ID)
- Shows playback URL

**FR-19: File Structure in Firebase Storage**
```
recordings/
├── a1b2c3d4/
│   └── 2026-06-18/
│       └── workspace.json
└── e5f6g7h8/
    └── 2026-06-19/
        └── workspace.json
```

**FR-20 [P0]: Firebase Auth**
- Firebase Auth is configured as part of the user's Firebase project setup
- The workspace JSON upload uses Firebase SDK authentication (no separate OAuth flow)
- Security Rules restrict access to the bucket owner and authorized viewers

### 6.8 Ad Integration

**FR-21 [P1]: Banner Ads**
- Non-intrusive banner during the call
- Placement: narrow bottom bar (below controls, above canvas edge)
- Ads rotate every 60 seconds
- Ads NOT included in recording (not part of canvas or video capture)
- Ad provider: TBD

### 6.9 Error & Edge Cases

| Scenario | Expected Behavior |
|----------|------------------|
| Camera/mic permission denied | Proceed without, show placeholder, allow enable later |
| Unsupported browser | Show list of supported browsers (Chrome, Firefox, Safari, Edge) |
| Network disconnection | Auto-reconnect WebSocket; video freezes then resumes |
| WebSocket disconnects mid-drawing | Buffered changes are synced on reconnect |
| Both draw conflicting changes | CRDT resolves automatically |
| Recording fails (codec) | Show error toast, suggest retry |
| Large diagram with many shapes | Virtual rendering / LOD to keep performance smooth |
| Candidate joins while interviewer already drawing | Send full state on join |
| Google Drive upload fails | Show error toast + retry button + download fallback |
| JSON recording file is corrupt | Use snapshots to reconstruct nearest valid state |
| Room link visited after session ended | Show "Session has ended" message with CTA to create new room |
| Interviewer closes browser mid-recording | Recording lost (in-memory). Warn on tab close. |
| OAuth popup blocked by browser | Show instructions to allow pop-ups, offer manual fallback |
| Room ID collision | Extremely unlikely with UUID. Collision check on creation; retry if collision |
| Interviewer tab crashes (not close) mid-recording | Recording lost. Warn via `beforeunload` event |
| Candidate's browser doesn't support VP8/Opus | Detect codec support via `MediaRecorder.isTypeSupported()`; fall back to default codec |
| Auto-destruct timeout for empty rooms | Destroy room after 30 min of inactivity; warn participants before destruction |
| Both participants click record simultaneously | Only the interviewer has the record button (candidate cannot trigger recording) |

---

## 7. Technical Considerations

### Frontend

- **Framework**: Next.js (App Router), TypeScript, Tailwind CSS
- **Whiteboard engines**: Fabric.js (Basic), maxGraph/mxGraph (Diagrams), Excalidraw (freehand) — each a separate question type plugin
- **Real-time sync**: CRDT via Y.js with Firestore provider (y-firestore); each question gets its own Y.Doc
- **Video/audio**: Free tier — P2P WebRTC. Paid tier — pluggable video provider via the user's own account (e.g., Zoom SDK)
- **Recording**: Paid tier only — provider server-side recording for video/audio; workspace JSON logged in browser and stored in Firebase Storage
- **State management**: Zustand or React context for call/room state

### Backend / Real-Time Infrastructure

- **Platform**: Firebase (Firestore) for room management, WebRTC signaling relay, Y.js CRDT relay, and chat relay
- **No dedicated signaling server** — Firebase handles real-time sync and presence natively
- **No persistent storage** beyond Firestore; rooms auto-destruct on inactivity

### Storage & Auth

- **Paid tier**: Workspace JSON stored in the user's Firebase Storage bucket. Video/audio recording handled by the connected provider (server-side).
- **Auth**: Firebase OAuth for Firebase project setup. Provider OAuth (e.g., Zoom) for video account integration.
- **Playback**: Workspace JSON fetched from Firebase Storage. Video embedded via the provider's player.

### Plugin Architecture

- Video providers follow the same plugin pattern as question types: each provider defines its own SDK integration, OAuth flow, recording interface, and embed player
- Each question type is a self-contained plugin defining: workspace UI, CRDT data model, recording event schema, and replay component
- Core platform handles video (P2P or provider), recording coordination, storage, and ads (room lifecycle via Firebase)

### Deployment

- **Frontend**: Vercel (or Firebase Hosting)
- **Backend**: Firebase (Firestore, no dedicated server)

## 8. Playback URL & Sharing

**Format:**
```
https://domain/playback?provider={providerRecordingId}&workspace={firebaseStoragePath}
```

**How it works:**
- The playback URL contains the video provider's recording ID and the Firebase Storage path to the workspace JSON
- The page embeds the video via the provider's player (e.g., Zoom's embed player) and fetches the workspace JSON from Firebase Storage
- This URL can be shared with the hiring team (they'll need access to the Firebase Storage file and the provider's recording)
- **Recordings list page** at `https://domain/recordings` queries Firestore to display all uploaded recordings for this Firebase instance

**Access Control:**
- Firestore rules and the video provider's own access controls handle permissions
- (Future: Add auto-sharing to specific email)

---

## 9. Monetization

| Feature | Free | Premium (paid) |
|---------|------|----------------|
| Video/audio | P2P WebRTC (2-3 participants) | Pluggable video provider via user's own account (e.g., Zoom SDK; up to 10 participants) |
| Collaborative whiteboard | ✅ | ✅ |
| Recording | ❌ Not available | ✅ Provider server-side recording + workspace JSON in Firebase Storage |
| Ads | ✅ In-session banners | ❌ No ads |
| Session length | Unlimited | Unlimited |
| Whiteboard style | All three | All three |
| Custom branding | ❌ | ✅ |

**Ad Placement:** Bottom banner, 60s rotation, non-intrusive
**Ad Network:** TBD (AdSense, Carbon, or direct)

---

## 10. Roadmap

| Phase | Features | Status |
|-------|----------|--------|
| **MVP** | Room creation, question type picker (whiteboard only), free tier (P2P video/audio, no recording), collaborative whiteboard (all 3 modes), text chat, banner ads | Current focus |
| **Phase 2** | Paid tier: Zoom SDK integration, server-side recording, workspace JSON storage in Firebase Storage, synchronized playback player, recordings list page | Next |
| **Phase 3** | Coding question type (in-browser code editor), mobile browser polish | Future |
| **Phase 4** | More question types, panel interviews, ATS integrations, question templates, premium tier (no ads) | Future |

---

## 11. Success Metrics

| Metric | Target |
|--------|--------|
| Room creation → interview success rate | >90% |
| P2P connection success rate | >85% (no TURN) |
| Whiteboard sync latency (p95) | <200ms |
| Recording start success rate | >95% |
| Recording → Drive upload completion | >90% |
| Playback load success rate | >95% |
| Completed interview rate (session >20 min) | >70% |
| Candidate join rate | >85% |
| Ad impressions per session | 3+ (sessions >5 min) |

---

## 12. Open Questions

- [ ] Ad provider / network selection
- [ ] Google Drive API quota handling
- [ ] Browser support for `MediaRecorder` with specific codecs
- [ ] Playback: Drive file accessibility (CORS / proxy needed?)
- [ ] Privacy policy & terms of service
- [ ] Rate limiting for room creation
- [ ] Analytics/tracking (privacy-first)
- [ ] Platform name & branding
- [ ] Custom domain support
- [ ] Should playback page also be ad-supported?

---

## 13. Competitor Landscape

| Tool | Free? | Question Types? | Workspace + Video Sync? | Drive Upload? |
|------|-------|-----------------|--------------------------|---------------|
| Google Meet + Jamboard | ✅ | Whiteboard only | ❌ No sync | ✅ |
| Zoom + Miro | ⚠️ Limited | Whiteboard only | ❌ No sync | ❌ |
| CoderPad | ❌ Paid | Coding, whiteboard | ❌ No sync | ❌ |
| CodeInterview | ✅ Limited | Coding, whiteboard | ❌ No sync | ❌ |
| **Ours** | **✅ Full** | **Pluggable (whiteboard MVP, more later)** | **✅ Synchronized dual recording** | **✅** |

---

*This is a living document. Open a PR or issue for changes.*

---

## 14. General Notes / Points to Think About Later

*(Points 1-3 on Zoom SDK, BYO accounts, and tiered pricing have been adopted into the main spec.)*

- Video providers follow the same pluggable pattern as question types. Adding a new provider (e.g., Google Meet SDK, Microsoft Teams) means implementing a new provider plugin with its own OAuth, streaming, recording, and embed player — no core changes.
