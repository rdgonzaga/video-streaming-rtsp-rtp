# Video Streaming over RTSP/RTP (Python)

This project is a simple client-server video streaming app using:

- RTSP over TCP for control messages (SETUP, PLAY, PAUSE, TEARDOWN)
- RTP over UDP for video frame delivery
- Tkinter + PIL for the client GUI

## Project Files

- `Server.py` - Starts the RTSP server socket.
- `ServerWorker.py` - Handles RTSP requests and sends RTP packets.
- `ClientLauncher.py` - Launches the client app with CLI args.
- `Client.py` - Client GUI, RTSP request logic, RTP receive/playback logic.
- `RtpPacket.py` - RTP packet encode/decode utilities.
- `VideoStream.py` - Reads MJPEG frames from the input file.

## Requirements

- Python 3.x
- Pillow (for `PIL`)

Install dependency:

```bash
pip install pillow
```

## How to Run

### 1. Start the server

```bash
python Server.py <server_port>
```

Example:

```bash
python Server.py 5544
```

Notes:

- Use a port greater than 1024.
- The server listens for incoming RTSP (TCP) connections on this port.

### 2. Start the client

```bash
python ClientLauncher.py <server_host> <server_port> <rtp_port> <video_file>
```

Example:

```bash
python ClientLauncher.py 127.0.0.1 5544 25000 chaeyoung.mjpeg
```

Argument meanings:

- `server_host`: Machine/IP where the server is running.
- `server_port`: Same RTSP port used by the server.
- `rtp_port`: Local UDP port for receiving RTP packets.
- `video_file`: MJPEG file requested from the server (for example, `movie.Mjpeg`).

## Typical RTSP Flow

A normal interaction sequence is:

1. `SETUP` - Create session and negotiate transport.
2. `PLAY` - Start streaming.
3. `PAUSE` - Pause streaming (optional).
4. `PLAY` - Resume streaming (optional).
5. `TEARDOWN` - End session and close sockets.

The server replies to each request. In this project, success is `200 OK`.

## RTSP Message Format Used

Requests are expected in this order:

1. Request line
2. `CSeq`
3. `Transport` for SETUP, or `Session` for PLAY/PAUSE/TEARDOWN

Example:

```text
SETUP movie.Mjpeg RTSP/1.0
CSeq: 1
Transport: RTP/UDP; client_port= 25000
```

```text
PLAY movie.Mjpeg RTSP/1.0
CSeq: 2
Session: 123456
```

## RTP Packet Basics (This Lab)

RTP header is 12 bytes:

- Version (`V`) = 2
- Padding (`P`) = 0
- Extension (`X`) = 0
- CSRC count (`CC`) = 0
- Marker (`M`) = 0
- Payload type (`PT`) = 26 (MJPEG)
- Sequence number = frame number
- Timestamp = current time value
- SSRC = any server identifier integer

Frames are sent every 50 ms from server to client over UDP.

## Client State Model

Client state transitions follow:

- `INIT` -> `READY` on successful `SETUP`
- `READY` -> `PLAYING` on successful `PLAY`
- `PLAYING` -> `READY` on successful `PAUSE`
- Session ends after `TEARDOWN`

## Troubleshooting

- If client cannot connect, verify `server_host` and `server_port`.
- If no video appears, verify `rtp_port` is open and not used by another process.
- Ensure the video file path/name passed to client exists on the server side.
- If `PIL` import fails, run `pip install pillow`.

## References

- RTSP: RFC 2326
- RTP: RFC 1889 / RFC 3550
