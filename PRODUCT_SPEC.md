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

## 5. Functional Requirements

### 5.1 Room Creation & Joining

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

### 5.2 Question Type System

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

| Type | MVP? | Workspace | Recording Format |
|------|------|-----------|-----------------|
| **Whiteboard (Basic)** — simple diagramming | ✅ MVP | Rectangles, circles, diamonds, arrows, and text labels on a fabric.js canvas. Color picker (fill + stroke), undo/redo, zoom/pan, export as PNG. Minimal and clean. | All shape add/move/delete/edit events + full state snapshot every 60s |
| | | ⚠️ *Future: Consider replacing with tldraw for richer shape system and built-in CRDT sync* | |
| **Whiteboard (Diagrams)** — smart connector diagrams | ✅ MVP | Shape libraries (cloud, DB, server, etc.), smart connector routing with edge routing, layers, grouping, and grid snapping on a maxGraph (mxGraph) canvas — the engine behind draw.io. Same drawing tools as Basic plus professional diagramming features. | All shape/connector add/move/delete/route events + layer/group changes + full state snapshot every 60s |
| | | ⚠️ *Future: Consider replacing with tldraw for unified whiteboard engine, or React Flow for structured node-graph diagrams* | |
| **Whiteboard (Excalidraw)** — hand-drawn / freeform | ✅ MVP | Hand-drawn freeform strokes, freehand arrows, and text on an Excalidraw canvas. Sketch-like aesthetic. Color picker, undo/redo, zoom/pan, export as PNG. | All stroke start/move/end events + text add/edit + full state snapshot every 60s |
| **Coding** (code editor) | Future | In-browser code editor with syntax highlighting | Code change events (insert/delete/replace by position) + full source text snapshot every 60s |
| **MCQ / Quiz** | Future | Question + multiple choice options | Answer selection events + timer events |
| **Document / Text** | Future | Rich text / markdown editor | Text insert/delete events + full document snapshot every 60s |

### 5.3 Collaborative Whiteboard

**FR-07 [P0]: Whiteboard Engine**
- Full-screen canvas filling most of the viewport
- Real-time sync via CRDT (Conflict-free Replicated Data Type)
- Every action (add shape, move, delete, edit text) broadcasts to the other participant instantly
- Late-joining participant receives full whiteboard state on entry

**FR-08 [P0]: Whiteboard Modes**

| Feature | Basic | Diagrams | Excalidraw |
|---------|-------|-----------|------------|
| Rectangles, circles, diamonds | ✅ | ✅ | ✅ (freehand) |
| Arrows & connectors | Simple | Smart connectors with routing (draw.io style) | Freehand arrows |
| Text labels | ✅ | ✅ | ✅ |
| Freehand drawing | ❌ | ❌ | ✅ |
| Shape libraries (cloud, DB, etc.) | ❌ | ✅ | ❌ |
| Layers & grouping | ❌ | ✅ | ❌ |
| Grid snapping | ❌ | ✅ | ❌ |
| Color picker | ✅ | ✅ | ✅ |
| Undo/redo | ✅ | ✅ | ✅ |
| Zoom & pan | ✅ | ✅ | ✅ |
| Export as PNG | ✅ | ✅ | ✅ |

**FR-09 [P0]: Whiteboard Toolbar**
- Shape selection (rectangle, circle, diamond, arrow, text)
- Pointer / select tool
- Delete tool
- Color picker (fill + stroke)
- Undo / Redo buttons
- Clear canvas
- Export as image
- Zoom controls (zoom in, zoom out, fit to screen)

**FR-10 [P0]: Real-Time Sync**
- Uses WebSocket-based CRDT (e.g., Y.js with y-websocket provider)
- Changes propagate within <100ms
- Each participant sees the other's cursor position (optional)
- Conflict resolution: CRDT ensures both reach the same state regardless of operation order

### 5.4 Live Video/Audio

**FR-11 [P0]: Peer-to-Peer WebRTC**
- Direct P2P connection using standard WebRTC
- STUN servers for NAT traversal
- TURN server support configurable for future addition

**FR-12 [P0]: Video Layout**
- Two video tiles overlaid on the whiteboard or positioned in a corner
- Self-view: small tile (bottom-right)
- Remote participant: medium tile (top-right or bottom-left)
- Tiles are draggable (optional)
- When camera is off: show name/initials placeholder

**FR-13 [P0]: Media Controls**
- Camera toggle
- Microphone toggle
- Mute indicator
- End call button (interviewer: ends for both; candidate: leaves only themselves)

### 5.5 Text Chat

**FR-14 [P1]: In-Call Chat**
- Toggleable chat sidebar
- Real-time messaging via WebSocket
- Ephemeral (not recorded in the official recording)
- Notification badge when minimized and new message arrives

### 5.6 Recording (Dual Recording)

The recording system captures two synchronized data streams:

**FR-15 [P0]: Video Recording**
- Uses browser `MediaRecorder` API
- Captures: composed video stream (both participants' video tracks) + all audio tracks (both participants)
- Does NOT capture the whiteboard canvas in the video (whiteboard is recorded separately)
- Format: `.webm` (VP8/Opus)
- Triggered by interviewer's "Record" button
- Recording indicator shown to both participants
- Interviewer can stop recording at any time

**FR-16 [P0]: Workspace Recording (JSON)**
- All question workspaces' actions logged as JSON events with timestamps in a single JSON file
- Each event is tagged with its `questionId` so the player can route it to the correct replay component
- Each question type defines its own event schema. Example:

```json
{
  "sessionStartTime": "2026-06-18T10:00:00Z",
  "sessionDuration": 3600000,
  "questions": [
    {
      "id": "q1",
      "type": "whiteboard-basic",
      "label": "System Design: API Gateway"
    },
    {
      "id": "q2",
      "type": "whiteboard-diagrams",
      "label": "Database Schema"
    }
  ],
  "events": [
    {
      "timestamp": 1234.567,
      "questionId": "q1",
      "type": "add_shape",
      "data": {
        "shapeType": "rectangle",
        "id": "shape-1",
        "x": 100,
        "y": 200,
        "width": 150,
        "height": 80,
        "fill": "#fff",
        "stroke": "#000",
        "text": "API Gateway"
      }
    },
    {
      "timestamp": 5678.901,
      "questionId": "q1",
      "type": "move_shape",
      "data": {
        "id": "shape-1",
        "x": 150,
        "y": 250
      }
    },
    {
      "timestamp": 9123.456,
      "questionId": "q2",
      "type": "add_cell",
      "data": { ... }
    }
  ],
  "snapshots": [
    {
      "timestamp": 5000,
      "questionId": "q1",
      "state": { ... }  // full serialized whiteboard-basic state
    },
    {
      "timestamp": 10000,
      "questionId": "q2",
      "state": { ... }  // full serialized whiteboard-diagrams state
    }
  ]
}
```

- Each question's snapshots captured every 60 seconds (independently per question)
- Timestamps are relative to session start (millisecond precision)
- Events logged only during recording (start/stop)
- The `questions` array maps question IDs to their types, so the player knows which replay component to use for each

**FR-17 [P0]: Sync Mechanism**
- Both recordings share the same time origin (`performance.now()` at session start)
- Frame-level sync achieved by matching event timestamps to video timecodes
- When the video shows time `T`, the player applies all whiteboard events up to `T`

### 5.7 Playback

**FR-18 [P0]: In-Browser Player**
- Custom player component for synchronized replay
- Layout: Workspace replay on the left/center, video player on the right/bottom
- Question tabs are shown above the workspace replay area, matching the interview's questions
- Switching tabs during playback rebuilds the workspace state at the current video time for that question
- Each question renders the replay component corresponding to its recorded type

```
┌──────────────────────┬──────────────┐
│  [Q1] [Q2] [Q3]     │              │
├──────────────────────┤   Video      │
│                      │   Player     │
│   Workspace          │   (WebM)     │
│   Replay             │              │
│   (per question      │              │
│    type)             │              │
│                      │              │
├──────────────────────┴──────────────┤
│  Play/Pause  │  Scrub Bar  │ Time  │
└────────────────────────────────────┘
```

**FR-19 [P0]: Playback Controls**
- Play / Pause (controls both video + diagram replay simultaneously)
- Scrub bar for seeking
- Time display (current / total)
- Speed control (1x, 1.5x, 2x)
- Skip forward/backward by 10 seconds

**FR-20 [P1]: Seeking Behavior**
- On seek, snapshots restore the workspace state to the nearest snapshot before the target time
- Then events are fast-forwarded from the snapshot to the exact target time
- This avoids replaying every event from the beginning

**FR-21 [P0]: Playback Entry Points**
- The playback page is a URL: `https://domain/playback?video={driveVideoId}&workspace={driveWorkspaceId}`
- This URL can be shared with the hiring team
- The page loads the video from a proxy or direct Drive URL and the workspace JSON from Drive
- **Alternative**: A self-contained zip with player.html + video.webm + workspace.json

### 5.8 Google Drive Integration

**FR-22 [P0]: Upload Flow**
- After session ends, prompt interviewer with "Upload to Google Drive"
- Google OAuth consent screen (scopes: `https://www.googleapis.com/auth/drive.file`)
- Creates or finds a folder named "Live Interview Recordings"
- Uploads two files:
  - `interview-{roomId}-{date}.webm`
  - `interview-{roomId}-{date}.json`
- Shows upload progress bar
- On completion: shows "Open in Drive" link + playback URL

**FR-23: File Structure in Drive**
Files are organized in a single folder with the following naming convention:
```
Live Interview Recordings/
├── interview-a1b2c3d4-2026-06-18.webm
├── interview-a1b2c3d4-2026-06-18.json
├── interview-e5f6g7h8-2026-06-19.webm
└── interview-e5f6g7h8-2026-06-19.json
```

**FR-24 [P0]: OAuth Implementation**
- One-time OAuth flow (no session persistence since no accounts)
- Token stored in memory only — user must re-auth if they refresh
- Token used only for the upload operation
- `drive.file` scope ensures only files created by the app are accessible

### 5.9 Ad Integration

**FR-25 [P1]: Banner Ads**
- Non-intrusive banner during the call
- Placement: narrow bottom bar (below controls, above canvas edge)
- Ads rotate every 60 seconds
- Ads NOT included in recording (not part of canvas or video capture)
- Ad provider: TBD

### 5.10 Error & Edge Cases

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

## 6. Technical Considerations

### Frontend

- **Framework**: Next.js (App Router), TypeScript, Tailwind CSS
- **Whiteboard engines**: Fabric.js (Basic), maxGraph/mxGraph (Diagrams), Excalidraw (freehand) — each a separate question type plugin
- **Real-time sync**: CRDT via Y.js with Firestore provider (y-firestore); each question gets its own Y.Doc
- **Video/audio**: Native RTCPeerConnection (P2P WebRTC) with STUN; TURN support configurable for future
- **Recording**: MediaRecorder API for video (.webm); question-type-specific event logger for workspace (.json)
- **State management**: Zustand or React context for call/room state

### Backend / Real-Time Infrastructure

- **Platform**: Firebase (Firestore) for room management, WebRTC signaling relay, Y.js CRDT relay, and chat relay
- **No dedicated signaling server** — Firebase handles real-time sync and presence natively
- **No persistent storage** beyond Firestore; rooms auto-destruct on inactivity

### Storage & Auth

- **No server-side file storage** — recordings upload directly from browser to Google Drive
- **Auth**: Google OAuth 2.0 PKCE (drive.file scope); token stored in memory only
- **Playback files** hosted on Google Drive; accessed via proxy URL

### Plugin Architecture

- Each question type is a self-contained plugin defining: workspace UI, CRDT data model, recording event schema, and replay component
- Core platform handles WebRTC, recording coordination, Drive upload, and ads (room lifecycle via Firebase)

### Deployment

- **Frontend**: Vercel (or Firebase Hosting)
- **Backend**: Firebase (Firestore, no dedicated server)

## 7. Playback URL & Sharing

**Format:**
```
https://domain/playback?video={driveVideoId}&workspace={driveWorkspaceId}
```

**How it works:**
- The playback URL contains Google Drive file IDs directly (no session token needed)
- The page fetches video via a server-side proxy (to attach auth headers and bypass CORS) and workspace JSON via Drive API export
- This URL can be shared with anyone (they'll need Google Drive access to the files)

**Access Control:**
- Files are private to the interviewer's Google Drive by default
- Reviewer needs access to view — interviewer must share Drive files manually
- (Future: Add auto-sharing to specific email)

---

## 8. Monetization

| Feature | Free (ad-supported) | Premium (future) |
|---------|-------------------|-------------------|
| Video/audio | ✅ | ✅ |
| Collaborative whiteboard | ✅ | ✅ |
| Recording | ✅ (dual) | ✅ |
| Google Drive upload | ✅ | ✅ |
| Ads | ✅ In-session banners | ❌ No ads |
| Recording length | Unlimited | Unlimited |
| Whiteboard style | All three | All three |
| TURN relay | ❌ | ✅ |
| Custom branding | ❌ | ✅ |

**Ad Placement:** Bottom banner, 60s rotation, non-intrusive
**Ad Network:** TBD (AdSense, Carbon, or direct)

---

## 9. Roadmap

| Phase | Features | Status |
|-------|----------|--------|
| **MVP** | Room creation, question type picker (whiteboard only), P2P video/audio, collaborative whiteboard (all 3 modes), dual recording (A/V + workspace JSON), download recording, Google Drive upload, text chat, synchronized playback player, banner ads | Current focus |
| **Phase 2** | Coding question type (in-browser code editor), playback seeking optimization, TURN server support, mobile browser polish | Next |
| **Phase 3** | MCQ / quiz question type, scheduling / calendar, multi-level undo/redo, cursor presence, whiteboard export | Future |
| **Phase 4** | More question types, panel interviews, ATS integrations, question templates, premium tier (no ads) | Future |

---

## 10. Success Metrics

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

## 11. Open Questions

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

## 12. Competitor Landscape

| Tool | Free? | Question Types? | Workspace + Video Sync? | Drive Upload? |
|------|-------|-----------------|--------------------------|---------------|
| Google Meet + Jamboard | ✅ | Whiteboard only | ❌ No sync | ✅ |
| Zoom + Miro | ⚠️ Limited | Whiteboard only | ❌ No sync | ❌ |
| CoderPad | ❌ Paid | Coding, whiteboard | ❌ No sync | ❌ |
| CodeInterview | ✅ Limited | Coding, whiteboard | ❌ No sync | ❌ |
| **Ours** | **✅ Full** | **Pluggable (whiteboard MVP, more later)** | **✅ Synchronized dual recording** | **✅** |

---

*This is a living document. Open a PR or issue for changes.*
