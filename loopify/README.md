# Loopify — Full System Design (v2)

## 1. Vision

Loopify is a zero-install, browser-based, mobile-first collaborative music environment that enables real-time loop creation, synchronization, and arrangement over peer-to-peer networks.

This version expands the system into a **fully-featured distributed music studio**, combining:

- Latency-resilient loop synchronization (LEP v2)
- WebRTC mesh networking with signaling
- Modular audio engine (loops, synth, sampler, effects)
- Advanced UI (clip launcher + radial visualization hybrid)
- Gesture-driven interaction
- AI-assisted music generation
- Persistent and shareable sessions

---

## 2. System Architecture

### 2.1 Layers

Client (Browser)
├── UI Layer (Canvas + DOM)
├── Interaction Layer (Gestures, Touch, MIDI)
├── Audio Engine (Web Audio API)
├── Sync Engine (Transport + LEP)
├── P2P Layer (WebRTC Mesh)
├── Persistence (IndexedDB)
└── AI Engine (Local + Remote inference optional)

Optional Backend
├── Signaling Server (WebSocket)
└── TURN/STUN (ICE fallback)

---

## 3. Session Model

### 3.1 Session Identity

sessionId = base58(randomBytes(16))
peerId = base58(randomBytes(12))

### 3.2 URL Format

https://loopify.app/?session=<SESSION_ID>

### 3.3 Join Flow

1. Parse session ID  
2. Connect to signaling server  
3. Discover peers  
4. Establish WebRTC mesh  
5. Sync SESSION_STATE  
6. Align transport cycle  

---

## 4. Loop Exchange Protocol (LEP v2)

### 4.1 Design Goals

- No real-time streaming dependency  
- Deterministic loop alignment  
- Bandwidth-efficient  
- Fault-tolerant  

---

### 4.2 Time Model

cycleDuration = (60 / BPM) * beatsPerBar * bars

cycleEpoch = shared timestamp  
localOffset = drift correction  

phase = ((now + offset - epoch) % cycleDuration) / cycleDuration  

---

### 4.3 Scheduling Strategy

- All loops scheduled one cycle ahead  
- Incoming loops queued into next boundary  
- Drift corrected gradually (no jumps)  

---

### 4.4 Message Types

LOOP_ADD

{
  type: "LOOP_ADD",
  loopId,
  encodedAudio,
  format: "f32|opus",
  bpm,
  bars,
  author,
  createdAt
}

LOOP_UPDATE

{
  type: "LOOP_UPDATE",
  loopId,
  mute,
  gain,
  speed
}

LOOP_REMOVE

{ type: "LOOP_REMOVE", loopId }

SESSION_STATE

{
  type: "SESSION_STATE",
  bpm,
  loops: [...],
  epoch
}

PEER_PING

{
  type: "PING",
  t0
}

---

### 4.5 Audio Encoding

Float32 (fast, large)  
Opus (compressed, slower)  

Pipeline:

AudioBuffer → Float32 → (optional Opus) → Base64  

---

## 5. P2P Networking

### 5.1 Topology

- Mesh (≤8 peers recommended)  
- Adaptive pruning if overloaded  

---

### 5.2 Connection Flow

1. WebSocket signaling  
2. SDP exchange  
3. ICE candidates  
4. DataChannel open  
5. State sync  

---

### 5.3 Channels

- control (reliable)  
- audio (chunked binary)  
- ping (latency measurement)  

---

### 5.4 Reliability

- Chunked buffer transfer  
- Retransmission for missing chunks  
- Hash verification  

---

### 5.5 NAT Handling

- STUN default  
- TURN fallback (optional relay)  

---

## 6. Audio Engine

### 6.1 Graph

[Sources] → [Per-loop Gain] → [FX Bus] → [Master] → Output

---

### 6.2 Components

- AudioContext  
- LoopScheduler  
- Recorder  
- SynthEngine  
- DrumMachine  
- Effects Rack  

---

### 6.3 Loop Playback

source.start(startTime, offset)  
source.loop = true  

---

### 6.4 Effects

- Reverb (Convolver)  
- Delay  
- Filter  
- Compressor  

---

## 7. Synth Engine

### 7.1 Features

- Oscillators: sine, square, saw, triangle  
- ADSR envelope  
- Filter  
- LFO  

---

### 7.2 Sequencer

- 8–32 steps  
- Quantized to cycle  
- Velocity + pitch  

---

## 8. Drum Machine

- Grid pads (4x4 / 4x8)  
- Kick, Snare, Hi-hat, Percussion  
- Swing, velocity, chaining  

---

## 9. AI Engine

### Capabilities

- Beat generation  
- Bassline suggestions  
- Chord progression loops  

### Modes

- Local  
- Remote  

### Inputs

- BPM  
- Active loops  
- Genre  

### Outputs

- MIDI  
- Audio loops  

---

## 10. Transport Controller

- Cycle timing  
- onCycleStart  
- onBeat  

Drift correction:

offset += (remotePhase - localPhase) * smoothingFactor  

---

## 11. UI System

Top Bar: Session / Share / BPM  
Main: Radial + Clip grid  
Bottom: Loop inspector  

---

## 12. Visualization

- Waveform rings  
- Radius = loop length  
- Color = peer  
- Arranged in grid or lanes

Playhead:

angle = phase * 2π  

States:

- Recording (pulse)  
- Muted (dim)  
- Active (bright)  

---

## 13. Clip Launcher

- Grid layout  
- Quantized triggering  

---

## 14. Interaction Model

Tap → select  
Double tap → mute  
Drag → gain  
Pinch → speed  
Long press → delete  

Recording:

1. Tap  
2. Pre-roll  
3. Record  
4. Stop  
5. Broadcast  

---

## 15. Loop Inspector

- Mute  
- Gain  
- Speed  
- FX  

---

## 16. Network Visualization

- Pulses on updates  
- Peer indicators  

---

## 17. Persistence

IndexedDB:

- Loops  
- Metadata  

Restore:

- Decode  
- Reschedule  

Export:

session.json  

---

## 18. Performance

- Max ~16 loops  
- Max 8 bars  
- <10s buffers  

Optimizations:

- OffscreenCanvas  
- AudioWorklets  
- Lazy decode  

---

## 19. Security

- No raw mic streaming  
- Isolation  
- Optional encryption  

---

## 20. Failure Handling

- Peer drop handling  
- Retransmission  
- Drift smoothing  

---

## 21. Future Extensions

- MIDI  
- Cloud sync  
- Timeline  
- Plugins  
- Spatial audio  

---

## 22. Implementation Phases

1. Audio engine  
2. Recording + viz  
3. WebRTC  
4. LEP  
5. UI gestures  
6. Synth/drums  
7. AI  

---

## 23. Summary

Loopify v2 is a distributed music system using loop-based synchronization instead of fragile real-time streaming.

It delivers:

- P2P collaboration  
- Deterministic timing  
- Modular audio tools  
- Gesture-first UX  
- AI-assisted creativity  

Result: fast, resilient, musical collaboration in the browser.