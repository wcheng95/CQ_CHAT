Everything is still only in my mind



FT8 CHAT PROTOCOL — SUMMARY (v1.0)



PURPOSE

FT8 CHAT is a half-duplex, token-based text chat protocol built on FT8 free-text messages.

It is designed for short human-readable conversations with minimal overhead and no per-message ACKs.



SESSION START

Caller sends:

CQ CHAT <CALLER> <GRID>



DX replies:

<CALLER> <DXCALL> <SNR>



Both stations then enter CHAT MODE, using the caller’s frequency and fixed FT8 slot parity.



SLOT RULES

• Each station has an allowed initial FT8 slot (parity-based).

• A station may start TX only in its allowed slot.

• During its turn, a station may transmit sequential allowed slots.

• Only one station owns the channel at a time.



MESSAGE GROUPS

• One logical message = one MESSAGE GROUP.

• A group contains 1–6 chunks.

• Chunks are numbered 0..k and end with Zk.

• Zk indicates the group has exactly k+1 chunks.



Chunk format (13 chars total):

<SEQ><PAYLOAD>



SEQ:

0–5 = data chunk

Z = end-of-group



Maximum message length = 71 characters.



Examples:

Z0HELLO

0HELLO WHATS

1UP NICE 2 CU

Z2AGN



TURN HANDOFF

• Sending Zk ends the sender’s turn and offers channel ownership.

• Receiver (DX) must transmit a decodable message in its first allowed slot.

• Any valid message is acceptable (Z0OK or real content).

• The first DX transmission implicitly acknowledges the handoff.



Zk RETRANSMISSION RULE

• Zk is normally sent once.

• If another sender-allowed slot exists before DX’s first allowed slot, Zk may be repeated.

• After DX’s first allowed slot:

– If DX energy or decode is detected → handoff succeeded.

– If no DX energy → sender may resend Zk (caller-allowed slots only).

• Retry limit recommended: 3 attempts, then drop chat.



MISSED CHUNKS

• Receiver determines missing chunks using Zk.

• No protocol-level retransmission.

• Operator may request repeats inside chat text (e.g. “MISSED 1 PSE”).



HEARTBEAT / LIVENESS

• Channel owner must send Z0OK as a heartbeat when idle.

• Heartbeat interval ≈ 1 minute.

• Any normal message resets the heartbeat timer.

• If two consecutive heartbeats are missed, the connection is considered lost.



Z0OK:

• Asserts liveness

• Does NOT transfer ownership



FCC IDENTIFICATION

• FCC ID is satisfied by transmitting:

DE <CALLSIGN>



• ID interval ≈ 10 minutes.

• ID must be sent in the station’s allowable slot only.

• ID frames are ignored by protocol logic.



TRANSPARENT ID INJECTION

• UI / firmware may transparently inject “DE <CALLSIGN>” into message text.

• Injection occurs only if ID timer requires it.

• User text has priority.

• If ID does not fit within 71 characters, send standalone ID later.

• Operator is unaware of injection.



SEQUENCE RULES

• Sequence numbers reset to 0 at the start of every message group.

• No sequence continuity across turns.

• Duplicate chunks may be ignored.



DESIGN PHILOSOPHY

• No per-chunk ACKs

• No protocol-level retransmission

• FT8-native timing and energy detection

• Human-focused interaction

• Bounded buffers and simple state machines



