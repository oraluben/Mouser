# Windows remote desktop sessions

By default, Mouser pauses all input handling when its own Windows session is
connected through RDP. The connecting computer handles mouse mappings; the
remote copy does not reverse scrolling, consume mouse buttons, execute volume
actions, or inject shortcuts. Windows still receives input the client forwards.

The **Point & Scroll → Remote Desktop → Automatically pause Mouser in remote
desktop sessions** switch disables this policy when the old behavior is wanted.
Mappings, scroll preferences, and the manual remapping toggle are preserved.
Settings can be edited while paused and take effect on the next local resume.

| Scenario | Expected behavior with auto-pause enabled |
| --- | --- |
| Start on the local console | Run with saved mappings and scroll direction |
| Start inside RDP | Do not start input hooks, HID interception, or foreground-app detection |
| Local session transfers to RDP | Stop input processing and HID polling; release simulated held mouse buttons and discard pending actions |
| Disconnect and reconnect RDP | Stay paused; do not repeat the warning |
| Return to the local console | Restore saved behavior automatically |
| Change mappings or scroll direction while remote | Save preferences without activating input processing |
| Disable auto-pause while remote | Resume the old behavior; preserve the manual enable/disable choice |
| Multiple users/sessions | Inspect the current process's session only |
| Local Windows instance connects to another PC | The local instance remains active; the RDP-hosted instance pauses |
| Nested RDP | Each RDP-hosted instance pauses |
| Session detection is temporarily unavailable | Pause until a console session can be confirmed; retry automatically |
| macOS/Linux or non-RDP console sharing tools | Keep existing behavior |

The UI shows a persistent paused banner and tray tooltip, plus one non-blocking
warning per remote period. The warning latch resets only after a confirmed local
session. The process stays alive so it can resume without restarting the app.

Detection uses `WTSQuerySessionInformationW(WTSClientProtocolType)` for the current
session (console `0`, RDP `2`). A positive `SM_REMOTESESSION` is a fallback; a
negative metric alone is insufficient to identify a local RemoteFX session.
WTS session notifications trigger immediate checks, with a one-second timer as
a fallback. No per-mouse hook callback makes blocking WTS calls.

Automated coverage includes both axes' four inversion combinations, remote
startup, local/remote transitions, opt-out, unknown session detection, stale
callbacks, held-button cleanup, and queued scroll cleanup. Manual verification
should connect/disconnect RDP and return to console using the same running app,
checking that only the client maps input and the warning does not repeat.
