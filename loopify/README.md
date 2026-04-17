# Loopify — Production System Design Specification (v4.0)

## STATUS

This document defines the complete production architecture of Loopify, including all runtime systems, interaction layers, synchronization models, persistence mechanisms, and observability infrastructure. The system is designed to support real-time collaborative loop creation, transformation, arrangement, and playback with deterministic timing guarantees.

---

# 0. SYSTEM OVERVIEW

Loopify is a distributed, cycle-synchronized audio composition platform built on deterministic loop scheduling, peer-to-peer state propagation, and reusable compositional units.

Core primitives:

- Loop (atomic audio unit)
- LoopSet (grouped loops)
- Arrangement (time-ordered LoopSets)
- Session (shared state across peers)

All playback is derived from a shared temporal model:

cycleTime = fixedDuration (e.g., 2s, 4s, etc.)

---

# 1. CORE AUDIO ENGINE

## 1.1 Architecture

- Web Audio API (AudioContext)
- AudioWorklet for real-time processing
- Dedicated scheduling thread

## 1.2 Scheduler

class LoopScheduler {
  constructor(ctx) {
    this.ctx = ctx
    this.sources = new Map()
  }

  schedule(loop, startTime, offset = 0) {
    const source = this.ctx.createBufferSource()
    source.buffer = loop.buffer
    source.loop = true

    source.connect(loop.outputNode)
    source.start(startTime, offset)

    this.sources.set(loop.id, source)
  }

  stop(loopId) {
    const src = this.sources.get(loopId)
    if (src) src.stop()
  }
}

## 1.3 Timing Model

globalTime = audioCtx.currentTime + offset

phase = ((globalTime - epoch) % cycleDuration) / cycleDuration

## 1.4 Guarantees

- Sample-accurate scheduling
- Continuous playback without drift
- Deterministic phase alignment

---

# 2. RECORDING ENGINE

## 2.1 Input Pipeline

getUserMedia → MediaStream → AudioWorklet → RingBuffer → Float32 capture

## 2.2 Recording Control

States:

- idle
- armed
- recording
- committing

## 2.3 Quantized Recording

startTime = nextCycleBoundary()

record(startTime, startTime + cycleDuration)

## 2.4 Loop Finalization

- Normalize buffer
- Trim silence (optional)
- Align to exact cycle length
- Assign loopId
- Insert into scheduler

## 2.5 Streaming Capture

- Audio chunks emitted during recording
- Chunks transmitted to peers immediately
- Final buffer assembled post-cycle

---

# 3. LOOP MODEL

Loop {
  id
  buffer
  gainNode
  metadata {
    bpm
    length
    source (mic|synth|drum)
    hash
  }
}

## 3.1 Hashing

hash = SHA-256(Float32Array)

Used for:

- Deduplication
- Cache lookup
- Integrity validation

---

# 4. LOOPSET SYSTEM

## 4.1 Structure

LoopSet {
  id
  name
  loopIds[]
  gain
  muteState
}

## 4.2 Behavior

- LoopSets act as atomic playback units
- All loops inside a set share activation state
- LoopSets can be scheduled, duplicated, and reused

## 4.3 Nested Control

- Per-loop overrides allowed
- Set-level gain applies multiplicatively

---

# 5. ARRANGEMENT ENGINE

## 5.1 Timeline Model

timeline = [
  {
    id
    setId
    startCycle
    duration
    transition
  }
]

## 5.2 Playback Resolver

onCycleStart(cycle):

  entry = timeline.find(t => cycle in range)

  if entry.setId !== activeSet:
    transitionTo(entry.setId, entry.transition)

## 5.3 Transitions

- hard
- fade (configurable duration)
- crossfade

## 5.4 LoopSet Switching

- Preload next set
- Align start at cycle boundary
- Apply transition envelope

---

# 6. SYNTH ENGINE

## 6.1 Signal Chain

oscillator → filter → gain → master

## 6.2 Components

- Oscillators: sine, square, saw, triangle
- Filter: lowpass, highpass, bandpass
- Envelope: ADSR
- LFO: pitch, filter, amplitude

## 6.3 Voice Management

- Polyphonic allocation
- Voice stealing strategy

## 6.4 Rendering

- OfflineAudioContext used for loop rendering
- Output converted to AudioBuffer
- Inserted as Loop

---

# 7. DRUM MACHINE

## 7.1 Grid

- 16-step sequencer
- Configurable step resolution

## 7.2 Playback

onStep(stepIndex):
  if pattern[stepIndex]:
    trigger(sample)

## 7.3 Swing

if (stepIndex % 2 === 1):
  time += swingOffset

## 7.4 Sample Engine

- Preloaded buffers
- Velocity scaling
- Per-step gain

---

# 8. NETWORK SYNCHRONIZATION

## 8.1 Protocol

Loop Exchange Protocol (LEP)

## 8.2 Transport

- WebRTC DataChannels (primary)
- WebSocket signaling
- STUN/TURN fallback

## 8.3 State Model

Shared:

- epoch
- cycleDuration
- loop metadata
- loop chunks
- arrangement state

## 8.4 Phase Sync

delta = remotePhase - localPhase

offset += delta * smoothingFactor

## 8.5 Chunk Transfer

- Binary ArrayBuffer
- Sequenced packets
- Retransmission on loss

---

# 9. CACHING + STORAGE

## 9.1 IndexedDB Schema

db.loops
db.loopSets
db.arrangements
db.audioChunks

## 9.2 Storage Formats

- Float32 (raw)
- Opus (compressed)

## 9.3 Cache Strategy

- Hash-based deduplication
- LRU eviction under pressure
- Lazy decoding

---

# 10. INTERACTION SYSTEM

## 10.1 Gesture Mapping

Tap → Select  
Double Tap → Mute  
Drag → Adjust Gain  
Pinch → Time Stretch  
Long Press → Delete  

## 10.2 Recording Interaction

tap → arm  
next cycle → record  
auto finalize → broadcast  

## 10.3 Loop Editing

- Gain control
- Mute/solo
- Speed adjustment
- Replace buffer

---

# 11. VISUALIZATION SYSTEM

## 11.1 Loop View

- Radial waveform display
- Phase indicator
- Gain ring

## 11.2 Grid View

- Loop slots
- Activation states
- Color-coded sources

## 11.3 Timeline View

- Horizontal arrangement
- LoopSet blocks
- Transition markers

## 11.4 Real-Time Indicators

- Phase alignment
- Active loops
- Recording state

---

# 12. LOGGING + OBSERVABILITY

## 12.1 Metrics

- phase drift (ms)
- RTT latency
- buffer decode time
- packet loss rate
- CPU usage
- memory usage

## 12.2 Event Logging

log(eventType, payload, timestamp)

## 12.3 Debug Panel

{
  peers,
  syncHealth,
  drift,
  transportState,
  audioLoad
}

## 12.4 Visualization

- Time-series graphs
- Peer sync indicators
- Network health status

---

# 13. PERFORMANCE MANAGEMENT

## 13.1 Limits

- Max peers: ~8
- Active loops: ~16–24
- Concurrent buffers: bounded

## 13.2 Optimization

- AudioWorklet processing
- Buffer pooling
- Lazy instantiation
- Chunk streaming

## 13.3 Degradation Strategy

- Disable visualization effects
- Reduce loop count
- Lower sample rate (fallback)

---

# 14. FAILURE HANDLING

Case                     | Handling
-------------------------|------------------------------
Late join                | Full state sync + cache fill
Missing chunks           | Retransmit request
Peer disconnect          | Preserve local state
Drift spike              | Gradual correction
Buffer decode failure    | Retry + fallback silence
CPU overload             | Loop throttling

---

# 15. SESSION MODEL

Session {
  id
  peers[]
  loops[]
  loopSets[]
  arrangement
  epoch
}

## 15.1 Join Flow

1. Connect signaling server
2. Establish WebRTC
3. Receive session snapshot
4. Sync epoch + loops
5. Enter playback

---

# 16. SHARING + COLLABORATION

## 16.1 Session Sharing

- Invite via link/token
- Join as active peer

## 16.2 State Propagation

- Loop creation broadcast
- Arrangement updates broadcast
- LoopSet changes broadcast

## 16.3 Consistency Model

- Eventual consistency
- Deterministic playback via shared timebase

---

# 17. SECURITY

## 17.1 Transport

- DTLS (WebRTC)
- WSS for signaling

## 17.2 Validation

- Hash verification for audio
- Schema validation for messages

---

# 18. SYSTEM GUARANTEES

- Deterministic loop synchronization
- Phase-aligned playback across peers
- Reusable compositional units (LoopSets)
- Real-time loop creation and distribution
- Persistent session recovery
- Interactive manipulation without timing disruption

---

# 19. SYSTEM CHARACTERISTICS

- No dependency on continuous audio streaming
- State-based synchronization model
- Cycle-locked execution
- Modular audio generation pipeline

---

# FINAL SUMMARY

Loopify provides a complete loop-based composition and collaboration system with:

- Mic recording, synthesis, and drum generation
- Reusable LoopSets and structured arrangements
- Deterministic synchronization across peers
- Fully interactive control surface
- Persistent caching and session recovery
- Real-time observability and diagnostics

All subsystems operate on a shared temporal model, ensuring consistent musical timing under real-world constraints.