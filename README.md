# Bomberman Online — GitHub Pages edition

Real-time multiplayer Bomberman for up to **20 simultaneous players**. The site is
fully static (GitHub Pages); team matches run on a small Node server that one person
hosts, and the page connects to it over a plain WebSocket — ordinary web traffic that
works under corporate proxies like Netskope.

## Two ways to play

1. **Team server:** the host runs the `bomberman-online` Node server behind a public
   tunnel (or on any host) and shares a link like `…?server=<address>`. Everyone opens
   the link, enters a name, hits **Connect**, then **Ready**. Match starts when
   everyone in the lobby (2+, max 20) is ready.
2. **Solo vs bots:** pick a bot count (3/7/11/19) and play instantly. Runs entirely in
   your browser — no network at all.

## Hosting a team match

On the machine that will host, from the `bomberman-online` folder:

```
.\start-team-server.ps1
```

It starts the authoritative game server, opens a Cloudflare quick tunnel, and prints
the full GitHub Pages invite link to share. Notes:

- The tunnel address changes each time the tunnel restarts — share a fresh link per
  session (the lobby's **Copy invite link** button always has the current one).
- The host machine must stay on while people play.
- Anyone with the link can join; the address is random but treat it as public.
- For a permanent address, run the server on an internal VM or Azure App Service
  instead and share `…?server=<that-host>` — nothing else changes.

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
