# Bomberman Online — GitHub Pages edition

Real-time multiplayer Bomberman for up to **20 simultaneous players**, deployable as a
fully static site. No game server: one player clicks **Host a new arena** and their
browser runs the authoritative simulation; everyone else joins peer-to-peer over
WebRTC data channels using a 5-character room code (or the invite link).

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
- **Corporate networks can block WebRTC.** Signaling uses WSS (usually fine), but the
  peer-to-peer media path uses UDP/STUN. There's no TURN relay configured, so on
  restrictive networks or between some VPN configurations, peers may fail to connect.
  If that bites your team, the same game with a proper Node server is a more reliable
  option (`bomberman-online`).
- The public PeerJS broker is a free community service — fine for casual team matches,
  but it's a shared dependency with no SLA.
