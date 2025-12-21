# FT8 CHAT PROTOCOL — SUMMARY (v1.0)

> **Status:** Design notes / working spec  
> Everything is still evolving, but internally consistent.

---

## 1. Purpose

FT8 CHAT is a **half-duplex, token-based text chat protocol** built on FT8 free-text messages.

It is designed for:

- short, human-readable conversations  
- minimal overhead  
- no per-message or per-chunk ACKs  

---

## 2. Session Start

### Caller initiates

```
CQ CHAT CALLER_CALL GRID
```

### DX replies

```
CALLER_CALL DX_CALL SNR
```

After this exchange:

- both stations enter **CHAT MODE**
- both stations use the **caller’s frequency**
- FT8 slot parity is fixed for the session

---

## 3. Slot Rules

- Each station has an **allowed initial FT8 slot** (parity-based)
- A station may **start transmitting only in its allowed slot**
- During its turn, a station may **transmit sequential allowed slots**
- **Only one station owns the channel at a time**

---

## 4. Message Groups

- One logical message = one **MESSAGE GROUP**
- A group contains **1 to 6 chunks**
- Chunks are numbered `0 .. k`
- The group always ends with **`Zk`**
- `Zk` means the group contains **exactly `k + 1` chunks**

---

## 5. Chunk Format

Each FT8 free-text transmission is **13 characters total**:

```
SEQ + PAYLOAD
```

Where:

- `SEQ` — one character  
  - `0–5` : data chunk  
  - `Z`   : end-of-group  
- `PAYLOAD` — up to 12 characters

### Maximum message length

- 6 chunks × 12 chars − 1 char = **71 characters**

---

## 6. Examples

### Single-chunk message

```
Z0HELLO
```

### Multi-chunk message

```
0HELLO WHATS
1UP NICE 2 CU
Z2AGN
```

---

## 7. Turn Handoff

- Sending `Zk` **ends the sender’s turn**
- Channel ownership is **offered to the other station**
- The receiver (DX) **must transmit a decodable message in its first allowed slot**
- Valid first responses include:
  - `Z0OK` (automatic keepalive)
  - real prepared message content
- The first DX transmission **implicitly acknowledges the handoff**

---

## 8. Zk Retransmission Rule

- `Zk` is normally sent **once**
- If another sender-allowed slot exists **before DX’s first allowed slot**, `Zk` may be repeated
- After DX’s first allowed slot:
  - if DX energy or decode is detected → handoff succeeded  
  - if **no DX energy** → sender may resend `Zk` (sender-allowed slots only)
- Recommended retry limit: **3 attempts**, then drop chat

---

## 9. Missed Chunks

- Receiver determines missing chunks using `Zk`
- **No protocol-level retransmission**
- Operator may request repeats **inside chat text**, for example:

```
MISSED 1 PSE
```

---

## 10. Heartbeat / Liveness

- Channel owner must send **`Z0OK`** when idle
- Heartbeat interval: **~1 minute**
- Any normal message resets the heartbeat timer
- If **two consecutive heartbeats are missed**, the connection is considered lost

`Z0OK`:

- asserts liveness
- **does NOT transfer ownership**

---

## 11. FCC Identification

FCC ID requirement is satisfied by transmitting:

```
DE CALLSIGN
```

Rules:

- ID interval ≈ **10 minutes**
- ID must be sent **only in the station’s allowable slot**
- ID frames are **ignored by protocol logic**

---

## 12. Transparent ID Injection

- UI / firmware may **automatically inject** `DE CALLSIGN` into outgoing messages
- Injection occurs only if the ID timer requires it
- User text always has priority
- If ID does not fit within 71 characters:
  - skip injection
  - send standalone ID later
- Operator remains unaware of injection

---

## 13. Sequence Rules

- Sequence numbers **reset to 0** at the start of every message group
- No sequence continuity across turns
- Duplicate chunks may be ignored

---

## 14. Design Philosophy

- No per-chunk ACKs
- No protocol-level retransmission
- FT8-native timing and energy awareness
- Human-focused interaction
- Simple, bounded state machines suitable for firmware
