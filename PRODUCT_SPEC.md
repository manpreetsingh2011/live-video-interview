# Product Spec: Live Interview Platform

> **Status**: Draft v2
> **Last Updated**: 2026-06-18

---

## 1. Executive Summary

- **What:** A free, ad-supported, web-based live interviewing platform with pluggable question types.
- **No accounts required:** Both participants join via a link — zero sign-up friction.
- **Core experience:** Real-time collaborative workspace (whiteboard, coding, etc.) + video/audio side by side.
- **MVP question types:** Whiteboard (Basic), Whiteboard (Flowchart), Whiteboard (Sketch) — each a separate, independent type.
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

---

## 5. Functional Requirements

### 6.1 Room Creation & Joining

**FR-01 [P0]: Landing Page**
- Minimal landing page: "Create Interview Room" CTA + "Join Room" input (room code/link)
- No marketing fluff — get to the room fast

**FR-02 [P0]: Room Creation**
- Interviewer clicks "Create Interview Room"
- Question type picker appears with three MVP options:
  1. **Whiteboard (Basic)** — boxes, circles, arrows, text (Fabric.js)
  2. **Whiteboard (Flowchart)** — smart connectors, shape libraries, layers (MxGraph)
  3. **Whiteboard (Sketch)** — hand-drawn feel, freehand strokes (Rough.js/Excalidraw)
- Each is a separate, independent question type with its own rendering engine, event model, and data format
- Room is created with a unique 8-character room code
- Interviewer enters the room immediately

**FR-03 [P0]: Share Link**
- Room URL: `https://domain/room/{roomCode}`
- Copy link to clipboard button
- Room code displayed for verbal sharing (e.g., "join at domain.com, code: abc-defg")

**FR-04 [P0]: Joining**
- Candidate opens link → sees room info + "Join Interview" button
- No account, no sign-up
- Browser prompts for camera/mic permissions
- Enters room — sees whiteboard + video tiles

**FR-05 [P1]: Room State & Presence**
- Both participants enter independently
- Each controls their own camera/mic activation
- UI shows: participant joined | participant left | participant camera/mic status
- Room is active until both participants leave or interviewer ends the session

### 6.2 Question Type System

**FR-06 [P0]: Pluggable Question Type Architecture**
- The workspace area renders a question-type-specific component
- Each question type defines:
  - A workspace UI component
  - A data model for its state (serializable)
  - A recording format (event log schema)
  - A replay component for playback
- Question types are selected at room creation (by the interviewer)
- Future question types can be added without modifying core platform code

**FR-07 [P0]: Current & Planned Question Types**

| Type | MVP? | Workspace | Recording Format |
|------|------|-----------|-----------------|
| **Whiteboard (Basic)** — simple diagramming | ✅ MVP | Fabric.js canvas (shapes, arrows, text) | Shape events + snapshots |
| **Whiteboard (Flowchart)** — smart connector diagrams | ✅ MVP | MxGraph canvas (shape libs, layers, routing) | Shape+connector events + snapshots |
| **Whiteboard (Sketch)** — hand-drawn / freeform | ✅ MVP | Rough.js / Excalidraw canvas (hand-drawn strokes, freehand) | Stroke events + snapshots |
| **Coding** (code editor) | Future | In-browser code editor with syntax highlighting | Code edit events + snapshots |
| **MCQ / Quiz** | Future | Question + multiple choice options | Answer selections + timer |
| **Document / Text** | Future | Rich text / markdown editor | Text edit events + snapshots |

### 6.3 Collaborative Whiteboard

**FR-08 [P0]: Whiteboard Engine**
- Full-screen canvas filling most of the viewport
- Real-time sync via CRDT (Conflict-free Replicated Data Type)
- Every action (add shape, move, delete, edit text) broadcasts to the other participant instantly
- Late-joining participant receives full whiteboard state on entry

**FR-09 [P0]: Whiteboard Modes**

| Feature | Basic | Flowchart | Hand-drawn |
|---------|-------|-----------|------------|
| Rectangles, circles, diamonds | ✅ | ✅ | ✅ (hand-drawn) |
| Arrows & connectors | Simple | Smart connectors with routing | Hand-drawn arrows |
| Text labels | ✅ | ✅ | ✅ |
| Freehand drawing | ❌ | ❌ | ✅ |
| Shape libraries (cloud, DB, etc.) | ❌ | ✅ | ❌ |
| Layers & grouping | ❌ | ✅ | ❌ |
| Grid snapping | ❌ | ✅ | ❌ |
| Color picker | ✅ | ✅ | ✅ |
| Undo/redo | ✅ | ✅ | ✅ |
| Zoom & pan | ✅ | ✅ | ✅ |
| Export as PNG | ✅ | ✅ | ✅ |

**FR-10 [P0]: Whiteboard Toolbar**
- Shape selection (rectangle, circle, diamond, arrow, text)
- Pointer / select tool
- Delete tool
- Color picker (fill + stroke)
- Undo / Redo buttons
- Clear canvas
- Export as image
- Zoom controls (zoom in, zoom out, fit to screen)

**FR-11 [P0]: Real-Time Sync**
- Uses WebSocket-based CRDT (e.g., Y.js with y-websocket provider)
- Changes propagate within <100ms
- Each participant sees the other's cursor position (optional)
- Conflict resolution: CRDT ensures both reach the same state regardless of operation order

### 6.4 Live Video/Audio

**FR-12 [P0]: Peer-to-Peer WebRTC**
- Direct P2P connection using standard WebRTC
- STUN servers for NAT traversal
- TURN server support configurable for future addition

**FR-13 [P0]: Video Layout**
- Two video tiles overlaid on the whiteboard or positioned in a corner
- Self-view: small tile (bottom-right)
- Remote participant: medium tile (top-right or bottom-left)
- Tiles are draggable (optional)
- When camera is off: show name/initials placeholder

**FR-14 [P0]: Media Controls**
- Camera toggle
- Microphone toggle
- Mute indicator
- End call button (interviewer: ends for both; candidate: leaves only themselves)

### 6.5 Text Chat

**FR-15 [P1]: In-Call Chat**
- Toggleable chat sidebar
- Real-time messaging via WebSocket
- Ephemeral (not recorded in the official recording)
- Notification badge when minimized and new message arrives

### 6.6 Recording (Dual Recording)

The recording system captures two synchronized data streams:

**FR-16 [P0]: Video Recording**
- Uses browser `MediaRecorder` API
- Captures: remote participant video track + all audio tracks (both participants)
- Does NOT capture the whiteboard canvas in the video (whiteboard is recorded separately)
- Format: `.webm` (VP8/Opus)
- Triggered by interviewer's "Record" button
- Recording indicator shown to both participants
- Interviewer can stop recording at any time

**FR-17 [P0]: Workspace Recording (JSON)**
- The active question type's workspace actions logged as JSON events with timestamps.
- Each question type defines its own event schema. Example (whiteboard):

```json
{
  "sessionStartTime": "2026-06-18T10:00:00Z",
  "events": [
    {
      "timestamp": 1234.567,
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
      "type": "move_shape",
      "data": {
        "id": "shape-1",
        "x": 150,
        "y": 250
      }
    },
    {
      "timestamp": 9123.456,
      "type": "add_arrow",
      "data": { ... }
    }
  ],
  "snapshots": [
    {
      "timestamp": 5000,
      "state": { ... }  // full serialized whiteboard state
    },
    {
      "timestamp": 10000,
      "state": { ... }
    }
  ]
}
```

- Snapshots: Full workspace state serialized every 5 seconds
- Timestamps are relative to session start (millisecond precision)
- Events logged only during recording (start/stop)
- The JSON includes a `questionType` field so the player knows which replay component to use

**FR-18 [P0]: Sync Mechanism**
- Both recordings share the same time origin (`performance.now()` at session start)
- Frame-level sync achieved by matching event timestamps to video timecodes
- When the video shows time `T`, the player applies all whiteboard events up to `T`

### 6.7 Playback

**FR-19 [P0]: In-Browser Player**
- Custom player component for synchronized replay
- Layout: Workspace replay on the left/center, video player on the right/bottom
- The workspace side renders the replay component corresponding to the recorded `questionType`
- Whiteboard replay renders the canvas with event replay; future types will render their own replay views

```
┌─────────────────────┬──────────────┐
│                     │              │
│   Workspace         │   Video      │
│   Replay            │   Player     │
│   (question-type    │   (WebM)     │
│    specific)        │              │
│                     │              │
├─────────────────────┴──────────────┤
│  Play/Pause  │  Scrub Bar  │ Time  │
└────────────────────────────────────┘
```

**FR-20 [P0]: Playback Controls**
- Play / Pause (controls both video + diagram replay simultaneously)
- Scrub bar for seeking
- Time display (current / total)
- Speed control (1x, 1.5x, 2x)
- Skip forward/backward by 10 seconds

**FR-21 [P1]: Seeking Behavior**
- On seek, snapshots restore the workspace state to the nearest snapshot before the target time
- Then events are fast-forwarded from the snapshot to the exact target time
- This avoids replaying every event from the beginning

**FR-22 [P0]: Playback Entry Points**
- The playback page is a URL: `https://domain/playback?video={driveFileId}&workspace={driveFileId}`
- This URL can be shared with the hiring team
- The page loads the video from a proxy or direct Drive URL and the workspace JSON from Drive
- **Alternative**: A self-contained zip with player.html + video.webm + workspace.json

### 6.8 Google Drive Integration

**FR-23 [P0]: Upload Flow**
- After session ends, prompt interviewer with "Upload to Google Drive"
- Google OAuth consent screen (scopes: `https://www.googleapis.com/auth/drive.file`)
- Creates or finds a folder named "Live Interview Recordings"
- Uploads two files:
  - `interview-{roomCode}-{date}.webm`
  - `interview-{roomCode}-{date}.json`
- Shows upload progress bar
- On completion: shows "Open in Drive" link + playback URL

**FR-24 [P0]: File Structure in Drive**
```
Live Interview Recordings/
├── interview-abc-defg-2026-06-18.webm
├── interview-abc-defg-2026-06-18.json
├── interview-hij-klmn-2026-06-19.webm
└── interview-hij-klmn-2026-06-19.json
```

**FR-25 [P0]: OAuth Implementation**
- One-time OAuth flow (no session persistence since no accounts)
- Token stored in memory only — user must re-auth if they refresh
- Token used only for the upload operation
- `drive.file` scope ensures only files created by the app are accessible

### 6.9 Ad Integration

**FR-26 [P1]: Banner Ads**
- Non-intrusive banner during the call
- Placement: narrow bottom bar (below controls, above canvas edge)
- Ads rotate every 60 seconds
- Ads NOT included in recording (not part of canvas or video capture)
- Ad provider: TBD

### 6.10 Error & Edge Cases

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

---

## 6. Technical Architecture

### 7.1 High-Level Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    Browser A (Interviewer)                    │
│  ┌───────────────────────────────────────────────────────┐   │
│  │  Next.js App                                         │   │
│  │  ┌──────────────────┐  ┌──────────────────────┐      │   │
│  │  │ Workspace Area   │  │ WebRTC (P2P)         │      │   │
│  │  │ (question-type   │  │ RTCPeerConnection    │      │   │
│  │  │  plugin)         │  │ MediaStream          │      │   │
│  │  │ ┌─────────────┐  │  └──────────┬───────────┘      │   │
│  │  │ │ Whiteboard  │  │             │                   │   │
│  │  │ │ (Fabric.js /│  │             │                   │   │
│  │  │ │  Rough.js / │  │             │                   │   │
│  │  │ │  MxGraph)   │  │             │                   │   │
│  │  │ └─────────────┘  │             │                   │   │
│  │  │ │ Whiteboard     │             │                   │   │
│  │ │ (Basic)        │             │                   │   │
│  │ │ Whiteboard     │             │                   │   │
│  │ │ (Flowchart)    │             │                   │   │
│  │ │ Whiteboard     │             │                   │   │
│  │ │ (Sketch)       │             │                   │   │
│  │ │ (future:       │             │                   │   │
│  │ │  coding, MCQ)  │             │                   │   │
│  │  │  MCQ, etc.)      │             │                   │   │
│  │  └────────┬─────────┘             │                   │   │
│  │           │                       │                   │   │
│  │  ┌────────▼───────────────────────▼───────────┐       │   │
│  │  │  Y.js CRDT (workspace sync)                │       │   │
│  │  │  + WebSocket Client                        │       │   │
│  │  └────────┬───────────────────────────────────┘       │   │
│  │           │                                           │   │
│  │  ┌────────▼──────────────────────┐                    │   │
│  │  │  Recording Manager           │                    │   │
│  │  │  - MediaRecorder (AV)        │                    │   │
│  │  │  - Event Logger (JSON)       │                    │   │
│  │  │  - Snapshot Taker            │                    │   │
│  │  │  (delegates to question type │                    │   │
│  │  │   for workspace state)       │                    │   │
│  │  └────────┬─────────────────────┘                    │   │
│  │           │                                          │   │
│  │  ┌────────▼──────────────────────┐                   │   │
│  │  │  Google Drive Uploader       │                   │   │
│  │  │  (OAuth + Drive API)         │                   │   │
│  │  └───────────────────────────────┘                   │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────┬──────────────────────────────┘
                               │
                  ┌────────────▼────────────┐
                  │   Signaling Server      │
                  │   (WebSocket)           │
                  │   - Room management     │
                  │   - WebRTC signaling    │
                  │   - CRDT relay (yjs)    │
                  │   - Chat relay          │
                  └────────────┬────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                    Browser B (Candidate)                     │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Same app, no recording/upload controls               │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Frontend (Next.js)

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router), TypeScript |
| Styling | Tailwind CSS |
| State | Zustand or React context for call/room state |
| Question Type Plugin System | Plugin registry + dynamic component loading |
| Whiteboard Basic | Fabric.js |
| Whiteboard Flowchart | MxGraph / Joint.js |
| Whiteboard Sketch | Rough.js + Excalidraw |
| Future: Coding | Monaco Editor (CodeMirror) |
| Future: MCQ | Custom React components |
| CRDT Sync | Y.js + y-websocket provider |
| WebRTC | Native `RTCPeerConnection` |
| Recording | `MediaRecorder` API + question-type-specific event logger |
| Google Drive | `gapi` client library / REST calls |

### 7.3 Backend (Signaling Server)

| Component | Technology |
|-----------|-----------|
| Runtime | Node.js |
| Server | WebSocket (ws library or Socket.IO) |
| CRDT relay | y-websocket |
| Room state | In-memory Map |

**Server Responsibilities:**
- Room creation and lifecycle
- WebRTC signaling relay (offer/answer/ICE candidates)
- Y.js CRDT document synchronization
- Chat message relay
- No persistent storage — everything ephemeral

### 7.4 Question Type Plugin Architecture

Each question type is a self-contained plugin:

```typescript
interface QuestionTypePlugin {
  id: string                    // 'whiteboard' | 'coding' | 'mcq' | ...
  label: string                 // Display name
  icon: ReactNode               // Icon for picker
  
  // Workspace component (rendered in the main area)
  Workspace: React.ComponentType<{
    roomId: string
    role: 'interviewer' | 'candidate'
    yDoc: Y.Doc
    recording?: RecordingController
  }>
  
  // Replay component (rendered in playback)
  Replay: React.ComponentType<{
    events: Event[]
    snapshots: Snapshot[]
    currentTime: number
    isPlaying: boolean
  }>
  
  // Serialization
  serializeState(yDoc: Y.Doc): object
  deserializeState(data: object): void
  
  // Recording
  getEventSchema(): EventSchema
  captureSnapshot(yDoc: Y.Doc): object
}
```

The core platform manages room lifecycle, WebRTC, recording coordination, Drive upload, and ads. Each question type provides its own workspace UI, sync model (via shared Y.Doc), recording event format, and replay component.

### 7.5 MVP Question Types: Whiteboard (Basic / Flowchart / Sketch)

Each is a separate question type plugin sharing a canvas metaphor but with different engines:

| Type | Library | Shape Model | Key Events | Unique To |
|------|---------|-------------|------------|-----------|
| **Basic** | Fabric.js | Simple rect, circle, arrow, text | addShape, move, delete, editText | Minimal, clean |
| **Flowchart** | MxGraph | Smart shapes, routing, layers, groups | addShape, connect, route, group, layer | Connector routing, shape libraries |
| **Sketch** | Rough.js + Excalidraw | Hand-drawn strokes, freehand paths | strokeStart, strokeMove, strokeEnd, addText | Freehand drawing, sketch feel |

**Separate CRDT Data Models (Y.js docs are namespaced by question type):**

```typescript
// Whiteboard Basic
Y.Doc {
  questionType: 'whiteboard-basic',
  shapes: Y.Map<BasicShape>
  // { id, type: 'rect'|'circle'|'arrow'|'text', x, y, w, h, fill, stroke, text }
}

// Whiteboard Flowchart
Y.Doc {
  questionType: 'whiteboard-flowchart',
  cells: Y.Map<Cell>     // mxGraph cells
  // cells with connection IDs, routing points, layers
}

// Whiteboard Sketch
Y.Doc {
  questionType: 'whiteboard-sketch',
  elements: Y.Map<ExcalidrawElement>
  // Excalidraw's native element format (strokes, text, images)
}
```

- Each is fully independent — different event schemas, rendering engines, serialization
- Core platform only knows about the plugin interface, not the internals
- New question types (coding, MCQ) follow the same pattern

### 7.6 Recording Architecture

The Recording Manager is question-type-agnostic. It delegates workspace-related recording to the active question type plugin.

**During session:**

```
RecordingManager {
  activeQuestionType: QuestionTypePlugin

  start() {
    this.startTime = performance.now()
    
    // Start video recording (always the same regardless of question type)
    this.videoRecorder = new MediaRecorder(composedStream)
    this.videoChunks = []
    this.videoRecorder.ondataavailable = e => this.videoChunks.push(e.data)
    this.videoRecorder.start()
    
    // Start workspace event capture (delegates to question type)
    this.events = []
    this.workspace.onAction(action => {
      this.events.push({
        timestamp: performance.now() - this.startTime,
        type: action.type,
        data: action.data
      })
    })
    
    // Start snapshot timer (delegates serialization to question type)
    this.snapshotInterval = setInterval(() => {
      this.snapshots.push({
        timestamp: performance.now() - this.startTime,
        state: this.activeQuestionType.serializeState(this.yDoc)
      })
    }, 5000)
  }
  
  stop() {
    this.videoRecorder.stop()
    clearInterval(this.snapshotInterval)
    
    // Build recording package
    return {
      video: new Blob(this.videoChunks, { type: 'video/webm' }),
      questionType: this.activeQuestionType.id,
      workspace: {
        events: this.events,
        snapshots: this.snapshots,
        sessionDuration: performance.now() - this.startTime
      }
    }
  }
}
```

**After session:**

```
1. RecordingManager returns { videoBlob, questionType, workspace }
2. Prompt user: Download or Upload to Google Drive
3. If Download: 
   - Save video as .webm
   - Save workspace JSON (includes questionType field) with download attribute
4. If Upload to Drive:
   - Google OAuth flow (if not already authed)
   - Upload video file to Drive
   - Upload workspace JSON to Drive
   - Show playback URL: /playback?video={id}&workspace={id}
```

### 7.7 Playback Architecture

The Playback Player is also question-type-agnostic. It renders the appropriate replay component based on the recorded `questionType`.

**Player Component:**

```
PlaybackPlayer {
  videoFile: Blob or URL,
  workspaceData: { questionType, events, snapshots }
  replayComponent: ReplayPlugin
  
  // Resolve replay component from questionType
  replayComponent = questionTypeRegistry.get(workspaceData.questionType).Replay
  
  // State
  currentTime: number
  isPlaying: boolean
  
  play() {
    video.play()
    animationFrame = requestAnimationFrame(updateWorkspace)
  }
  
  pause() {
    video.pause()
    cancelAnimationFrame(animationFrame)
  }
  
  seek(time) {
    video.currentTime = time / 1000
    // Delegate to replay component
    replayComponent.seek(time, workspaceData)
    currentTime = time
  }
  
  updateWorkspace() {
    if (isPlaying) {
      now = video.currentTime * 1000
      replayComponent.applyEventsBetween(currentTime, now, workspaceData)
      currentTime = now
      animationFrame = requestAnimationFrame(updateWorkspace)
    }
  }
}
```

### 7.7 Deployment

| Component | Hosting |
|-----------|---------|
| Next.js frontend | Vercel (free tier) |
| WebSocket signaling server | Railway / Fly.io / Render |
| File storage (during upload) | None (direct browser-to-Drive) |
| TURN server (future) | Twilio / self-hosted coturn |
| Analytics | Plausible or PostHog |

---

## 7. Data Model (In-Memory)

```typescript
interface Room {
  id: string                    // 8-char room code
  questionType: string          // 'whiteboard-basic' | 'whiteboard-flowchart' | 'whiteboard-sketch' | 'coding' | ...
  createdAt: Date
  participants: Participant[]
  chatMessages: ChatMessage[]
  createdBy: string             // socket ID of creator
  
  // Y.js document for CRDT sync
  yDoc: Y.Doc
}

interface Participant {
  socketId: string
  role: 'interviewer' | 'candidate'
  joinedAt: Date
  videoEnabled: boolean
  audioEnabled: boolean
  name?: string                 // optional display name
}

interface ChatMessage {
  senderId: string
  text: string
  timestamp: Date
}
```

No persistent database — everything in server memory. Room auto-destructs when all participants leave or after inactivity timeout.

---

## 8. Google Drive API Integration

**Auth Flow:**
1. User clicks "Upload to Google Drive"
2. Google OAuth 2.0 implicit flow (client-side)
3. Scope: `https://www.googleapis.com/auth/drive.file`
4. Token stored in JavaScript memory only

**Upload Flow:**
```
1. GET /drive/v3/files?q=name='Live Interview Recordings' and mimeType='application/vnd.google-apps.folder'
2. If not found: POST /drive/v3/files (create folder)
3. POST /drive/v3/files (upload .webm, parent=recordingFolderId)
4. POST /drive/v3/files (upload .json, parent=recordingFolderId)
5. Return file IDs and webViewLinks
```

**File Metadata:**
```json
{
  "name": "interview-abc-defg-2026-06-18.webm",
  "mimeType": "video/webm",
  "parents": ["recordingFolderId"],
  "description": "Recorded on LiveInterviewPlatform. 
                  Diagram playback: interview-abc-defg-2026-06-18.json"
}
```

---

## 9. Playback URL & Sharing

**Format:**
```
https://domain/playback?id={sessionId}
```

**How it works:**
- `sessionId` is a random token generated at session end
- The token maps to the Google Drive file IDs (stored in URL params or a short-lived mapping)
- **Alternative approach**: The playback URL contains Drive file IDs directly:
  ```
  https://domain/playback?v={driveVideoId}&d={driveDiagramId}
  ```
- The page loads the video via Drive's export URL and diagram JSON via Drive's export
- This URL can be shared with anyone (they'll need Google Drive access to the files)

**Access Control:**
- Files are private to the interviewer's Google Drive by default
- Reviewer needs access to view — interviewer must share Drive files manually
- (Future: Add auto-sharing to specific email)

---

## 10. Monetization

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

## 11. Roadmap

| Phase | Features | Status |
|-------|----------|--------|
| **MVP** | Room creation, question type picker (whiteboard only), P2P video/audio, collaborative whiteboard (all 3 modes), dual recording (A/V + workspace JSON), download recording, Google Drive upload, text chat, synchronized playback player, banner ads | Current focus |
| **Phase 2** | Coding question type (in-browser code editor), playback seeking optimization, TURN server support, mobile browser polish | Next |
| **Phase 3** | MCQ / quiz question type, scheduling / calendar, undo/redo improvements, cursor presence, whiteboard export | Future |
| **Phase 4** | More question types, panel interviews, ATS integrations, question templates, premium tier (no ads) | Future |

---

## 12. Success Metrics

| Metric | Target |
|--------|--------|
| Room creation → interview success rate | >90% |
| P2P connection success rate | >85% (no TURN) |
| Whiteboard sync latency (p95) | <200ms |
| Recording start success rate | >95% |
| Recording → Drive upload completion | >90% |
| Playback load success rate | >95% |
| Average session duration | >25 min (system design interviews) |
| Candidate join rate | >85% |
| Ad impressions per session | 3+ (sessions >5 min) |

---

## 13. Open Questions

- [ ] Ad provider / network selection
- [ ] Whiteboard library decision: build custom vs integrate existing (Excalidraw, draw.io, etc.)
- [ ] CRDT library: Y.js vs Liveblocks vs custom
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

## 14. Competitor Landscape

| Tool | Free? | Question Types? | Workspace + Video Sync? | Drive Upload? |
|------|-------|-----------------|--------------------------|---------------|
| Google Meet + Jamboard | ✅ | Whiteboard only | ❌ No sync | ✅ |
| Zoom + Miro | ⚠️ Limited | Whiteboard only | ❌ No sync | ❌ |
| CoderPad | ❌ Paid | Coding, whiteboard | ❌ No sync | ❌ |
| CodeInterview | ✅ Limited | Coding, whiteboard | ❌ No sync | ❌ |
| **Ours** | **✅ Full** | **Pluggable (whiteboard MVP, more later)** | **✅ Synchronized dual recording** | **✅** |

---

*This is a living document. Open a PR or issue for changes.*
