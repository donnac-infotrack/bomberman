# Bomberman Online — GitHub Pages edition

Real-time multiplayer Bomberman for up to **20 simultaneous players**. The site is
fully static (GitHub Pages); team matches run on a small Node server that one person
hosts, and the page connects to it over a plain WebSocket — ordinary web traffic that
works under corporate proxies like Netskope.

## Two ways to play

1. **Play online:** open the site, enter a name, hit **Play online**. It connects to the
   always-on team server (hosted on Render) — no setup, no host, nothing local. Match
   starts when everyone in the room (2+, max 20) is ready.
   - **Rooms:** leave the room field blank to join the shared **MAIN** room, or type a
     code (e.g. `ALPHA`) so your team plays on its own arena — same code = same game,
     different codes run at the same time (up to 200 rooms). The lobby's **Copy invite
     link** shares a `…?room=CODE` link that drops teammates straight into your room.
   - The first player in a room and anyone named **Dunix** can kick players.
2. **Solo vs bots:** pick a bot count (3/7/11/19) and play instantly. Runs entirely in
   your browser — no network at all.

## The always-on server

The team server lives in the `bomberman-server` repo, deployed to Render as a free web
service. The Pages client points at it by default (`DEFAULT_SERVER` in `index.html`),
so players never type an address. To move the server, redeploy and update that one
constant (or override per session with `…?server=<host>`).

Free-tier note: the Render instance sleeps after ~15 min idle and takes ~40s to wake,
so the first join after a quiet spell is slow, then instant for everyone. The
`bomberman-online` folder still has the local/tunnel host scripts if ever needed
offline.

## Gameplay

WASD/arrows to move, Space to bomb. Blast crates for power-ups (extra bombs, longer
blasts, faster boots). Bombs chain-react; you can walk off a bomb you just placed.
Last bomber standing wins. After 90 seconds the arena collapses inward — sudden death.
The dead spectate until the next lobby; mid-match joiners spectate too.

## Architecture

- `index.html` — static client: canvas renderer with position smoothing, lobby flow,
  sounds, touch controls. In team mode it only sends inputs over WSS; in solo mode it
  runs the same authoritative simulation locally (on a Web Worker tick) with AI bots
  driven by BFS pathfinding over a live danger map.
- The team server lives in the sibling `bomberman-online` project — a 30 Hz
  authoritative Node simulation broadcasting ~2 KB snapshots at 15 Hz. Client and
  server speak the same JSON protocol in both modes.

Earlier revisions included a WebRTC peer-to-peer mode. It was removed: corporate
endpoint agents (Netskope/Zscaler) block the peer data path regardless of network, so
it failed exactly where the team wanted to play. The git history has it if ever needed.

## Deploying the site

Push `index.html` + `README.md` to a public repo, enable Pages (deploy from branch,
root). Done — the game is `https://<owner>.github.io/<repo>/`.
