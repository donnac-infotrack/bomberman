# Bomberman Online — GitHub Pages edition

Real-time multiplayer Bomberman for up to **20 simultaneous players**, deployable as a
fully static site. No game server: one player clicks **Host a new arena** and their
browser runs the authoritative simulation; everyone else joins peer-to-peer over
WebRTC data channels using a 5-character room code (or the invite link).

## Three ways to play

1. **Team server (works everywhere, incl. corporate networks):** someone runs the
   `bomberman-online` Node server behind a public tunnel or host and shares a link like
   `…github.io/bomberman/?server=<host>`. The client connects over plain WSS — to
   proxies like Netskope it's ordinary web traffic. Same wire protocol, same game.
2. **Peer-to-peer rooms:** host a room, share the 5-character code. Works on unmanaged
   home networks; endpoint proxies (Netskope/Zscaler) usually break it — see below.
3. **Solo vs bots:** fully local, works everywhere, no network at all.

## How to play

1. Open the site. Enter a name.
2. One person clicks **Host a new arena** and shares the room code or **Copy invite link**.
3. Everyone else enters the code (or opens the link) and clicks **Join**, then **Ready**.
4. The match starts when everyone in the lobby (2+, max 20) is ready.
5. WASD/arrows to move, Space to bomb. Blast crates for power-ups (extra bombs, longer
   blasts, faster boots). Bombs chain-react. Last bomber standing wins; after 90 seconds
   the arena collapses inward — sudden death. The dead spectate until the next lobby.

## Deploying to GitHub Pages

The site is these three files: `index.html`, `peerjs.min.js`, `README.md`.

1. Push this folder to a GitHub repository (public, on the default branch).
2. Repo **Settings → Pages → Build and deployment**: Source = *Deploy from a branch*,
   Branch = `main`, folder `/ (root)`.
3. Open `https://<owner>.github.io/<repo>/` — that's the game.

## How it works

- **No backend.** The "server" from the multiplayer design lives inside the host's
  browser: a 30 Hz authoritative simulation (movement, bombs, chain blasts, power-ups,
  sudden death, win detection) broadcasting ~2 KB JSON snapshots at 15 Hz to every peer.
- **WebRTC + PeerJS.** Peers find each other through PeerJS's free public signaling
  broker (`0.peerjs.com`) using the room code as the host's peer ID, then all game
  traffic flows directly between browsers. `peerjs.min.js` is vendored so the page has
  no CDN dependency.
- **Background-tab safe.** The host's game loop is driven by a Web Worker timer with an
  accumulator, so browsers' background-tab throttling can't freeze the match if the
  host switches windows.
- Guests only send inputs; all rules run on the host — everyone sees the same match.

## Limitations to know about

- **The host's tab is the server.** If the host closes it, the arena dies (the page
  warns them). Refresh and host a new room.
- **Corporate networks can block WebRTC.** The peer path tries UDP first, then falls
  back to free TURN relays over TCP/443 (Open Relay by Metered), which traverses many
  corporate firewalls. But SSL-inspection proxies such as **Netskope or Zscaler** can
  still block the PeerJS signaling socket (`wss://0.peerjs.com`) or the relay itself —
  if joins consistently time out at the office, that's why. The reliable option on such
  networks is the Node server version (`bomberman-online`) run on a machine inside the
  office LAN: that traffic never leaves the building, so the proxy never sees it.
  Alternatively, ask IT to allow `0.peerjs.com` and `*.relay.metered.ca`.
- The public PeerJS broker is a free community service — fine for casual team matches,
  but it's a shared dependency with no SLA.
