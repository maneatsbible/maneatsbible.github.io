# Loopify — Design Specification (Vanilla JS P2P Audio Loop Collaboration)

## 1. Overview

**Loopify** is a mobile-first, browser-based peer-to-peer (P2P) application for recording, exchanging, and collaboratively arranging looping audio in real time. It is designed to be resilient to latency through a **Loop Exchange Protocol (LEP)**, allowing participants to stay musically synchronized without requiring tight real-time streaming.

Core principles:
- Latency-tolerant collaboration (loop-based sync, not stream-based)
- Zero-install, URL-based session sharing
- Creative immediacy (tap → record → loop → share)
- Visual clarity (waveform rings + rotating playheads)
- Robust offline-first P2P architecture

---

## 2. Architecture

### 2.1 Stack

- Frontend: Vanilla JavaScript (ES Modules)
- Audio Engine: Web Audio API
- P2P Layer: WebRTC (DataChannels + optional MediaStreams)
- Signaling: Lightweight WebSocket relay (stateless)
- Storage: IndexedDB (local loop persistence)
- Rendering: Canvas 2D (waveform rings) + minimal DOM

---

## 3. Session Model

### 3.1 Session Initialization

A session is created locally and assigned a unique ID:

sessionId = base58(randomBytes(16))

URL format:

https://loopify.app/?session=<SESSION_ID>

On load:
- If `session` param exists → join session
- Else → create new session

### 3.2 Sharing

- Copy/share button:
  - Copies full URL
  - Native mobile share API (if available)

---

## 4. Loop Exchange Protocol (LEP)

### 4.1 Core Idea

Instead of syncing real-time audio streams, peers exchange loop buffers + timing metadata, allowing playback to align on loop boundaries.

### 4.2 Time Model

Each session has a shared tempo (BPM) and loop length (bars)

cycleDuration = (60 / BPM) * beatsPerBar * bars

Each peer computes:

localCycleStart = performance.now() aligned to nearest cycle

### 4.3 Latency Handling

- Loops are scheduled one cycle ahead
- Incoming loops are:
  - Buffered
  - Phase-aligned
  - Activated at next cycle boundary

### 4.4 Data Messages

LOOP_ADD

{
  "type": "LOOP_ADD",
  "loopId": "...",
  "audioBuffer": "...encoded...",
  "length": 1,
  "bpm": 120,
  "createdAt": 123456789,
  "author": "peerId"
}

LOOP_UPDATE

{
  "type": "LOOP_UPDATE",
  "loopId": "...",
  "mute": false,
  "speed": 1.0
}

SESSION_STATE

{
  "type": "SESSION_STATE",
  "bpm": 120,
  "loops": [...]
}

---

## 5. P2P Layer

### 5.1 Topology

- Mesh network (small groups, <8 peers recommended)
- Graceful degradation:
  - Drop inactive peers
  - Limit loop buffer size

### 5.2 Connection Flow

1. Peer joins via session ID
2. Signaling server exchanges SDP offers
3. WebRTC DataChannels established
4. Full session state sync

### 5.3 Limitations

- NAT/firewall issues may block connections
- Mesh scaling constraints
- Large audio buffers increase bandwidth

Mitigations:
- Compress buffers (PCM → Float32 → optional Opus encoding)
- Limit loop duration (1–8 bars)
- Cap concurrent loops per peer

---

## 6. Audio Engine

### 6.1 Core Components

- AudioContext
- Master Gain Node
- Loop Scheduler
- Recorder Node
- Synth Engine

### 6.2 Loop Playback

Each loop:
- Stored as AudioBuffer
- Played via:

source.start(startTime, offset)
source.loop = true

### 6.3 Recording

- Uses getUserMedia({ audio: true })
- Records into buffer via ScriptProcessor / AudioWorklet
- Quantized start:
  - Recording begins at next cycle boundary
- Auto-trim to loop length

---

## 7. Loop Model

class Loop {
  constructor({
    id,
    buffer,
    bpm,
    length,
    author
  }) {
    this.id = id;
    this.buffer = buffer;
    this.bpm = bpm;
    this.length = length;

    this.muted = false;
    this.speed = 1.0;
    this.gain = 1.0;

    this.createdAt = performance.now();
    this.author = author;
  }
}

---

## 8. Controllers

### 8.1 SessionController

- Manages peers
- Syncs session state
- Handles join/leave events

### 8.2 LoopController

- Add/remove/update loops
- Schedule playback
- Apply quantization

### 8.3 TransportController

- Maintains global cycle timing
- Emits:
  - onCycleStart
  - onBeat

---

## 9. UI Design (Mobile First)

[ Top Bar ]
- Session ID
- Share Button
- BPM Control

[ Loop Canvas ]
- Radial waveform rings

[ Controls ]
- Record
- Synth
- Add Beat

[ Bottom Panel ]
- Loop controls (selected loop)

---

## 10. Loop Visualization

### 10.1 Waveform Rings

Each loop is rendered as:
- Circular waveform
- Radius based on length
- Arranged in grid
- Color per peer

### 10.2 Playhead

- Rotating dial (like a clock hand)

angle = (currentTime % cycleDuration) / cycleDuration * 2π

### 10.3 States

- Muted → dimmed
- Active → bright
- Recording → pulsing glow

---

## 11. UI Components

### 11.1 LoopWidget

Responsibilities:
- Render waveform ring
- Handle touch interactions

Interactions:
- Tap → select loop
- Double tap → mute/unmute
- Drag → adjust gain
- Pinch → speed control

---

### 11.2 RecorderWidget

- Big circular record button

States:
- Idle
- Counting (pre-roll)
- Recording
- Finalizing

---

### 11.3 SynthWidget

Simple synth:
- Oscillator types: sine, square, saw
- Step sequencer (8–16 steps)
- Quantized to loop cycle

---

### 11.4 Beat Maker

- Grid-based drum sequencer
- Preloaded samples (kick, snare, hat)
- Syncs to BPM

---

## 12. Interaction Model

### 12.1 Recording Flow

1. Tap record
2. Pre-roll countdown (1 bar)
3. Record starts at cycle boundary
4. Recording auto-stops at loop end
5. Loop is created + broadcast

### 12.2 Loop Editing

- Mute toggle
- Speed:
  - 0.5x
  - 1x
  - 2x
- Gain slider

---

## 13. Network Activity Visualization

- Subtle pulses on loop rings when:
  - Data received
  - Loop updated

- Peer indicators:
  - Colored dots
  - Connection strength

---

## 14. Studio Concerns

### 14.1 Latency

- Hidden via cycle scheduling
- No attempt at real-time streaming sync

### 14.2 Drift

- Periodic re-alignment:

adjust localCycleStart slightly

### 14.3 Audio Quality

Tradeoff:
- Smaller buffers → faster sync
- Larger buffers → better quality

---

## 15. Persistence

- Save loops in IndexedDB
- Restore session on reload
- Option:
  - Export session JSON

---

## 16. Performance Considerations

- Limit:
  - Max loops: ~16
  - Buffer length: <10s

- Use:
  - OffscreenCanvas (if available)
  - AudioWorklets instead of ScriptProcessor

---

## 17. Future Enhancements

- AI-assisted beat generation
- MIDI device support
- Loop effects (reverb, delay)
- Cloud relay fallback

---

## 18. Summary

Loopify combines:
- A latency-resilient loop protocol
- A robust P2P mesh network
- A playful, tactile UI

The result is a system where:
- Recording is immediate
- Sharing is effortless (URL-based)
- Collaboration feels musical, not technical

---

## 19. Implementation Priorities

1. Core audio loop engine
2. Cycle synchronization
3. WebRTC P2P layer
4. Loop visualization
5. Recording pipeline
6. Sharing via URL
7. UI polish and gestures

---

End of specification.