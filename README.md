# Zapo

Introduction


zapo is a high-performance TypeScript implementation of the WhatsApp Web protocol, built for high-scale, multi-session bot and automation workloads.

Zapo
zapo (published on npm as zapo-js) is an independent, runtime implementation of the WhatsApp Web protocol written in TypeScript. It is not a wrapper or fork of an existing WhatsApp client library — the protocol source of truth is the deobfuscated WhatsApp Web client, and the goal is behavior parity with WhatsApp Web while improving CPU, memory, and allocation efficiency.
Stability. zapo is 1.0 and stable. It follows semantic versioning — breaking changes only ever land in a new major release. You can upgrade across minors and patches safely; still validate major upgrades against the changelog.
​
Why zapo
Coordinator-first API
Every feature area is a focused coordinator: client.message, client.group, client.newsletter, client.privacy, and more.
Pluggable storage
One createStore factory, per-domain provider selection, and official backends for SQLite, PostgreSQL, MySQL, Redis, and MongoDB.
Multi-session ready
Every query is scoped by sessionId, so a single process can drive many accounts — built for multi-tenant workloads.
Performance-disciplined
Uint8Array everywhere, zero-copy in hot paths, bounded in-memory structures, async I/O, and synchronous crypto (bar elliptic-curve ops) for raw throughput.
​
Design principles
These principles drive every implementation decision in the codebase:
index-first — protocol behavior is validated against WhatsApp Web before anything is implemented.
performance-first — optimize for low CPU, low RAM, low allocations, and zero-copy in hot paths.
async-first I/O — I/O and network operations are asynchronous. Crypto, by contrast, runs synchronously — only elliptic-curve operations are async. Keeping the rest of crypto sync delivered a large, measurable throughput gain.
​
Requirements
Node.js >= 20.9.0
A package manager (npm, pnpm, or similar)
No mandatory runtime dependencies — backends and logging are opt-in peer dependencies.
​
Get started
Installation
Install zapo-js, pick a storage backend, and wire up optional peers.
Quickstart
Connect, scan a QR code, and reply to your first message in minutes.
Architecture
Understand the client, coordinators, stores, and event flow.
Sending messages
Text, replies, mentions, media, polls, reactions, and more.
​
Disclaimer
This project is an independent implementation for engineering and interoperability research. It is not affiliated with or endorsed by WhatsApp.

Installation

Install zapo-js with npm, pnpm, or yarn, choose a storage backend for credentials, and add the optional peer dependencies your features depend on.

​
Requirements
zapo requires Node.js >= 20.9.0. The package ships dual ESM/CJS builds and full TypeScript types.
​
Install the core package

npm
npm install zapo-js
pnpm
pnpm add zapo-js
yarn
yarn add zapo-js
The core package has no mandatory runtime dependencies. Everything else — storage, logging, and the WebSocket transport — is an opt-in peer dependency, so you only install what you use.
​
The package ecosystem
zapo-js is the only required install. Everything else is an optional @zapo-js/* package you add as needed:
zapo-js — core client, coordinators, store contract
@zapo-js/store-sqlite — SQLite backend
@zapo-js/store-postgres — PostgreSQL backend
@zapo-js/store-mysql — MySQL backend
@zapo-js/store-redis — Redis backend
@zapo-js/store-mongo — MongoDB backend
@zapo-js/media-utils — thumbnails, probes, waveforms
@zapo-js/voip — WhatsApp voice calls
@zapo-js/wam — WhatsApp Web telemetry parity (analytics)
@zapo-js/native — optional Rust NAPI + WASM crypto accelerator
@zapo-js/mcp-server — dev tool: drive from an AI agent
@zapo-js/fake-server — dev tool: in-process test server
​
Add a storage backend
zapo persists authentication and Signal state through a pluggable store. Pick the backend that matches your deployment and install its package:
Package	Backend	Best for
@zapo-js/store-sqlite	SQLite (via better-sqlite3)	Local / single-process
@zapo-js/store-postgres	PostgreSQL	Distributed, relational
@zapo-js/store-mysql	MySQL	Distributed, relational
@zapo-js/store-redis	Redis	Cache + persistence
@zapo-js/store-mongo	MongoDB	Document store

SQLite
npm install @zapo-js/store-sqlite better-sqlite3
PostgreSQL
npm install @zapo-js/store-postgres pg
MySQL
npm install @zapo-js/store-mysql mysql2
Redis
npm install @zapo-js/store-redis ioredis
MongoDB
npm install @zapo-js/store-mongo mongodb
You can also run with no backend at all — the built-in memory store works out of the box and is great for tests. It just does not survive a process restart, so you would re-pair on every boot.
​
Optional peer dependencies
Install these only if you use the corresponding feature:

Structured logging
npm install pino pino-pretty
WebSocket proxy
npm install ws
Mobile connections
npm install argo-codec
pino + pino-pretty — required only if you use createPinoLogger. Without them, the built-in ConsoleLogger is used.
ws — only needed to route the WebSocket through a proxy. The runtime’s native WebSocket can’t take an HTTP Agent/dispatcher, so zapo falls back to ws for the proxy.ws leg. Without a proxy, the built-in WebSocket is used and you don’t need this package.
argo-codec — only needed for mobile connections (for now). The standard companion (QR / pairing-code) flow does not use it.
​
Sending media
@zapo-js/media-utils is effectively required to send usable media. Media still uploads without it, but there’s no processor to generate thumbnails/previews, image-video dimensions, or voice-note waveforms — so it can render as a plain attachment or with no preview. Install it whenever your app sends images, video, audio, documents, or stickers.
npm install @zapo-js/media-utils
It shells out to ffmpeg/ffprobe and uses sharp, so make sure those binaries are available. See the media guide for how to wire the processor into the client.
@zapo-js/media-utils also lists file-type (^19) as an optional peer dependency. Install it (npm install file-type) to enable automatic mimetype detection — without it, the media guide’s mimetype resolution falls back to requiring an explicit mimetype on each send.
​
Voice calls
Install @zapo-js/voip to place and receive WhatsApp voice calls. It ships as a WaClient plugin and pulls two peer dependencies of its own:
npm install @zapo-js/voip @roamhq/wrtc libmlow-wasm
@roamhq/wrtc — SCTP for the relay transport.
libmlow-wasm — WhatsApp’s Opus profile, as WebAssembly (no native build).
See the VoIP guide for how to wire voipPlugin() into the client.
​
Telemetry parity
Install @zapo-js/wam to make the session emit the client-side w:stats analytics batches a real WhatsApp Web tab sends — a wire-parity / anti-fingerprinting improvement, not required for messaging.
npm install @zapo-js/wam
The only peer dependency is zapo-js. The WAM event registry (@vinikjkkj/wa-wam) is pinned as a regular dependencies entry on the plugin, so it comes down transitively.
See the WAM guide for how to wire wamPlugin() into the client. It is opt-in: skip it unless you specifically want the parity.
​
Native crypto accelerator
Install @zapo-js/native to move the messaging crypto hot path off pure JavaScript — XEdDSA sign/verify (which dominates SKDM sign in group fanout) and X25519 ECDH scalar mult (avoiding the node:crypto.diffieHellman DER round-trip) — onto a Rust core shipped as a WebAssembly build. No toolchain, no post-install compile.
npm install @zapo-js/native
zapo-js try-requires the binding at module load; when the package (or its native binary) is absent, the pure-JS path takes over and the client works unchanged. Installing it is always safe.
Requires Node.js >= 20.19.0: loading the WASM glue relies on require(esm), unflagged only on 20.19+ / 22.12+. On older runtimes the client silently falls back to the pure-JS path.
Two environment variables force the JS path per primitive, useful for A/B measuring or as a kill switch:
ZAPO_XEDDSA_FORCE_JS=1 — force JS for XEdDSA sign/verify.
ZAPO_X25519_FORCE_JS=1 — force JS for X25519 scalar mult.
To disable the accelerator entirely, set ZAPO_NATIVE_BACKEND=js before the process starts. See the package README for backend selection details.
​
Verify your setup
import { WaClient } from 'zapo-js'

console.log(typeof WaClient) // "function"

Quickstart

Connect a WhatsApp session, scan the QR code or use a pairing code, and reply to your first incoming message in under five minutes with zapo.

This guide builds a minimal “ping → pong” bot. It connects, prints a QR code to pair, and replies pong whenever it receives ping.
1
Install the packages

npm install zapo-js @zapo-js/store-sqlite better-sqlite3 pino pino-pretty
2
Create the store and client

The store persists auth and Signal state. This example uses SQLite, writing to .auth/state.sqlite.
import { createPinoLogger, createStore, WaClient } from 'zapo-js'
import { createSqliteStore } from '@zapo-js/store-sqlite'

const logger = await createPinoLogger({ level: 'info', pretty: true })

const store = createStore({
  backends: {
    sqlite: createSqliteStore({ path: '.auth/state.sqlite', driver: 'auto' })
  },
  providers: {
    auth: 'sqlite',
    signal: 'sqlite',
    preKey: 'sqlite',
    session: 'sqlite',
    identity: 'sqlite',
    senderKey: 'sqlite',
    appState: 'sqlite',
    privacyToken: 'sqlite',
    messages: 'sqlite',
    threads: 'sqlite',
    contacts: 'sqlite'
  }
})

const client = new WaClient(
  {
    store,
    sessionId: 'default',
    connectTimeoutMs: 15_000,
    nodeQueryTimeoutMs: 30_000,
    history: { enabled: true, requireFullSync: true }
  },
  logger
)
3
Handle pairing

On a fresh session the client emits auth_qr. Render the value as a QR code (for example with the qrcode-terminal package) and scan it from WhatsApp → Linked devices.
client.on('auth_qr', ({ qr, ttlMs }) => {
  console.log('Scan this QR within', ttlMs, 'ms:')
  console.log(qr)
})

client.on('auth_paired', ({ credentials }) => {
  console.log('Paired as', credentials.meJid)
})
Prefer an 8-character pairing code instead of a QR? See Authentication.
4
Reply to incoming messages

Listen for the message event and send a reply with client.message.send.
function extractText(message) {
  return (
    message?.conversation ??
    message?.extendedTextMessage?.text ??
    undefined
  )
}

client.on('message', async (event) => {
  const text = extractText(event.message)
  if (text?.trim().toLowerCase() !== 'ping') return

  await client.message.send(event.key.remoteJid, 'pong')
})
5
Connect

await client.connect()
connect() resolves once the socket is open. The first connection drives pairing; subsequent connections reuse the stored credentials.
​
Full example
import { createPinoLogger, createStore, WaClient } from 'zapo-js'
import { createSqliteStore } from '@zapo-js/store-sqlite'

const logger = await createPinoLogger({ level: 'info', pretty: true })

const store = createStore({
  backends: {
    sqlite: createSqliteStore({ path: '.auth/state.sqlite', driver: 'auto' })
  },
  providers: {
    auth: 'sqlite',
    signal: 'sqlite',
    preKey: 'sqlite',
    session: 'sqlite',
    identity: 'sqlite',
    senderKey: 'sqlite',
    appState: 'sqlite',
    privacyToken: 'sqlite',
    messages: 'sqlite',
    threads: 'sqlite',
    contacts: 'sqlite'
  }
})

const client = new WaClient({ store, sessionId: 'default' }, logger)

client.on('auth_qr', ({ qr }) => console.log(qr))
client.on('connection', (event) => console.log('connection:', event.status, event.reason))

client.on('message', async (event) => {
  const text =
    event.message?.conversation ?? event.message?.extendedTextMessage?.text
  if (text?.trim().toLowerCase() !== 'ping') return
  await client.message.send(event.key.remoteJid, 'pong')
})

await client.connect()

Migrating from Baileys

Coming from Baileys? Here’s how the connection lifecycle, store layout, message API, and event names map onto zapo’s coordinator-based client.

zapo is an independent implementation of the WhatsApp Web protocol — not a fork of Baileys. The concepts overlap (companion pairing, Signal sessions, an event stream), but the API is different and it is not a drop-in replacement. This page maps the patterns you already know.
Auth state, message shapes, and method names differ. Plan to rewrite your socket setup and handlers — but the mental model (pair → listen → send) carries over directly.
​
Move existing sessions over (no re-pair)
Switching libraries doesn’t have to mean re-pairing every session. wa-store-migrate converts Baileys auth state directly into zapo’s store layout — same device identity, same Signal sessions with every peer, no QR scan.
npm i wa-store-migrate
The library is pure conversion: it doesn’t read files, open sockets, or talk to any database itself. You hand it a BaileysAuthSnapshot (the same { creds, keys } shape useMultiFileAuthState holds in memory), it returns a ZapoStoreSnapshot, and you write that into a fresh zapo store.
import { migrate, bufferJsonReviver, type BaileysAuthSnapshot } from 'wa-store-migrate'
import { createStore } from 'zapo-js'
import { createSqliteStore } from '@zapo-js/store-sqlite'
import { readFileSync, readdirSync } from 'node:fs'
import { join } from 'node:path'

// 1. Read your Baileys auth folder into a snapshot
function readBaileysMultiFile(dir: string): BaileysAuthSnapshot {
  const creds = JSON.parse(readFileSync(join(dir, 'creds.json'), 'utf-8'), bufferJsonReviver)
  const keys: Record<string, Record<string, unknown>> = {}
  for (const f of readdirSync(dir)) {
    if (f === 'creds.json') continue
    const m = /^([a-z-]+)-(.+)\.json$/i.exec(f)
    if (!m) continue
    const id = m[2]!.replace(/__/g, '/').replace(/-/g, ':')
    ;(keys[m[1]!] ??= {})[id] = JSON.parse(
      readFileSync(join(dir, f), 'utf-8'),
      bufferJsonReviver
    )
  }
  return { creds, keys: keys as never }
}

// 2. Convert to zapo's snapshot shape (pure, no I/O)
const baileys = readBaileysMultiFile('.auth/baileys')
const { data, losses } = migrate({ from: 'baileys', to: 'zapo', data: baileys })
for (const l of losses) {
  console.warn(`[${l.severity}] ${l.domain} x${l.count}: ${l.reason}`)
}

// 3. Write into a brand-new zapo store
const store = createStore({
  backends: { sqlite: createSqliteStore({ path: '.auth/zapo.sqlite', driver: 'auto' }) },
  providers: {
    auth: 'sqlite', signal: 'sqlite', preKey: 'sqlite', session: 'sqlite',
    identity: 'sqlite', senderKey: 'sqlite', appState: 'sqlite',
    privacyToken: 'sqlite',
    messages: 'none', threads: 'none', contacts: 'none'
  }
})
const s = store.session('default')

await s.auth.save(data.credentials)
for (const k of data.preKeys ?? []) await s.preKey.putPreKey(k)
if (data.identities?.length) {
  await s.identity.setRemoteIdentities(
    data.identities.map((i) => ({ address: i.address, identityKey: i.identityKey }))
  )
}
if (data.sessions?.length) {
  await s.session.setSessionsBatch(
    data.sessions.map((x) => ({ address: x.address, session: x.record as never }))
  )
}
for (const sk of data.senderKeys ?? []) await s.senderKey.upsertSenderKey(sk.record as never)
if (data.appState?.keys?.length) await s.appState.upsertSyncKeys(data.appState.keys)
if (data.privacyTokens?.length) await s.privacyToken.upsertBatch(data.privacyTokens)
Point a new WaClient({ store, sessionId: 'default' }) at the resulting store and call connect() — it comes up as the existing device, with every peer Signal session intact.
Auth state in a database, not files? The BaileysAuthSnapshot shape is just { creds, keys: { 'pre-key': {...}, 'session': {...}, 'sender-key': {...}, ... } }. If your Baileys storage is custom (one MySQL table per session, Postgres rows, Redis hashes, …), point your reader at that store and produce the same object — the migrate() call and the zapo write side stay identical. For multiple sessions, run the pipeline once per sessionId.
Loss expectations. Migrating to zapo has no drops — every Signal domain transfers. Expect a few warn-severity entries in losses: skipped HKDF message keys are dropped (sessions self-heal on the next message), only the latest sender-key state is kept (libsignal stores up to 5), and privacy-token timestamps lose sub-second precision. Full table on wa-store-migrate.
​
Creating the socket
Baileys gives you a socket from a factory; zapo gives you a WaClient plus an explicit store.
// Baileys (typical)
const { state, saveCreds } = await useMultiFileAuthState('auth')
const sock = makeWASocket({ auth: state })
sock.ev.on('creds.update', saveCreds)

// zapo
import { createStore, WaClient } from 'zapo-js'
import { createSqliteStore } from '@zapo-js/store-sqlite'

const store = createStore({
  backends: { sqlite: createSqliteStore({ path: '.auth/state.sqlite', driver: 'auto' }) },
  providers: {
    auth: 'sqlite',
    signal: 'sqlite',
    preKey: 'sqlite',
    session: 'sqlite',
    identity: 'sqlite',
    senderKey: 'sqlite',
    appState: 'sqlite',
    privacyToken: 'sqlite',
    messages: 'none',
    threads: 'none',
    contacts: 'none'
  }
})
const client = new WaClient({ store, sessionId: 'default' }, logger)
await client.connect()
There is no creds.update to save by hand — the store persists credentials automatically. Pick any backend (SQLite, Postgres, MySQL, Redis, Mongo) on Installation.
​
Events
Baileys multiplexes everything through sock.ev; zapo exposes a typed event per concern on client.
Baileys	zapo
sock.ev.on('connection.update', ({ connection, qr, lastDisconnect }) => …)	client.on('connection', …) + client.on('auth_qr', …) + client.on('auth_paired', …)
sock.ev.on('creds.update', saveCreds)	(automatic — the store persists creds)
sock.ev.on('messages.upsert', ({ messages }) => …)	client.on('message', (event) => …)
sock.ev.on('messages.update', …) (edits/reactions/polls)	client.on('message_addon', …) / client.on('message_protocol', …)
sock.ev.on('message-receipt.update', …)	client.on('receipt', …)
sock.ev.on('groups.update' / 'group-participants.update', …)	client.on('group', …)
sock.ev.on('presence.update', …)	client.on('presence', …) / client.on('chatstate', …)
The full map is in Events. Each event is strongly typed via WaClientEventMap.
​
Sending messages
// Baileys
await sock.sendMessage(jid, { text: 'hello' })
await sock.sendMessage(jid, { image: { url: './pic.jpg' }, caption: 'hi' })

// zapo
await client.message.send(jid, 'hello') // string shorthand for text
await client.message.send(jid, { type: 'image', media: './pic.jpg', mimetype: 'image/jpeg', caption: 'hi' })
zapo uses a discriminated content union ({ type: 'image' | 'video' | 'audio' | 'document' | 'poll' | 'reaction' | … }) instead of Baileys’ shape-by-key object. Quoting/mentions move from the content object into the options argument ({ quote, mentions }).
​
API mapping
Baileys	zapo
makeWASocket(...)	new WaClient(options, logger)
useMultiFileAuthState(...)	createStore({ backends, providers })
sock.sendMessage(jid, content)	client.message.send(jid, content, options?)
downloadMediaMessage(...)	client.message.download(event) / downloadToFile(event, path)
sock.groupMetadata(jid)	client.group.queryGroupMetadata(jid)
sock.groupCreate(...) / groupParticipantsUpdate(...)	client.group.createGroup(...) / addParticipants / removeParticipants / promoteParticipants / …
sock.updateProfilePicture(...) / updateProfileStatus(...)	client.profile.setProfilePicture(...) / setStatus(...)
sock.updateBlockStatus(...)	client.privacy.blockUser(jid) / unblockUser(jid)
sock.sendPresenceUpdate(...)	client.presence.send(...) / sendChatstate(...)
sock.logout()	client.logout()
jidNormalizedUser(...) / jidDecode(...)	toUserJid(...) / splitJid(...) / parseJidFull(...) (JID helpers)
proto.Message	proto (exported from the package root)
​

Use the docs from your AI assistant


Pull these docs into Claude Code, Cursor, ChatGPT, or any MCP-capable agent — three integration paths, no setup on our side.

These docs are built to be consumed by AI coding agents directly, so you can ask questions like “how do I send a sticker with zapo?” or “what events fire during pairing?” and get answers grounded in the current reference — not stale model memory.
This page is about the documentation MCP — exposing the zapo.to pages to your assistant. It is not the @zapo-js/mcp-server dev tool, which drives a live WaClient from an agent. Different goals, different setup, and you can use them together.
​
Pick a path
One-click contextual
Use the Copy / Open in… menu at the top of any page to send its content to Claude, ChatGPT, or Cursor. No setup.
MCP server
Add https://zapo.to/mcp as an MCP server in your client — the agent then searches and fetches docs pages on demand.
llms.txt
Plain-text bundles at /llms.txt (index) and /llms-full.txt (full corpus) for tools that don’t speak MCP.
​
Contextual buttons
Every page on zapo.to has a menu near the title that lets you:
Copy page as Markdown — paste straight into any chat.
View as Markdown — see the raw source.
Open in ChatGPT / Claude / Cursor — opens the assistant with the current page pre-loaded as context.
Connect MCP — copy the MCP URL for any other client.
Best for a one-off question about a single page.
​
MCP server
For sustained work — building an integration, debugging an event, planning a migration from Baileys — register the docs as a long-lived MCP server. The agent then searches and fetches pages itself as needed, without you copy-pasting.
The endpoint is:
https://zapo.to/mcp
​
Claude Code
claude mcp add zapo-docs --scope user --transport http https://zapo.to/mcp
​
Cursor
Add to ~/.cursor/mcp.json (or the workspace-local .cursor/mcp.json):
{
  "mcpServers": {
    "zapo-docs": {
      "url": "https://zapo.to/mcp"
    }
  }
}
​
Other clients
Any MCP-compatible client (Windsurf, Zed, Continue, custom agents built on the MCP spec) can connect to the same URL — refer to your client’s docs for the exact registration syntax. The server exposes search and page-fetch tools.
Pair the docs MCP with the @zapo-js/mcp-server dev tool to get an agent that can both read the docs and drive a live WaClient — handy for “explain what this event means, then trigger it on a real session” workflows.
​
llms.txt
For tools that don’t support MCP yet, the same corpus is published as static text following the llms.txt convention:
File	Contents
/llms.txt	Index of every page with titles and one-line summaries — small enough to paste into any prompt.
/llms-full.txt	Full page bodies, concatenated. Large; best for one-shot context windows or RAG ingestion.
Both files are regenerated on every deploy, so they always reflect the live docs.

Architecture


How the WaClient, coordinators, stores, transport, and event flow fit together inside zapo, and how data moves from WhatsApp into your code.

zapo is organized around a thin client that delegates every feature to a focused coordinator. The client owns the connection, authentication, and event emitter; coordinators own the domain logic.
​
The client
WaClient is the single entry point. You construct it with options and an optional logger, then call connect():
const client = new WaClient({ store, sessionId: 'default' }, logger)
await client.connect()
The client itself exposes only a small surface: connection lifecycle (connect, disconnect, logout), state queries (getState, getCredentials), and the typed event emitter (on, once, off). Everything else lives behind a coordinator getter.
​
Coordinators
Each coordinator is reached through a getter on the client. They are lazily wired at construction and are safe to hold references to.
Getter	Coordinator	Responsibility
client.auth	WaAuthClient	Pairing, credentials, registration state
client.message	WaMessageCoordinator	Send/receive, receipts, media download, addons
client.presence	WaPresenceCoordinator	Own/peer presence and chat-state
client.chat	WaAppStateMutationCoordinator	Chat settings: mute, pin, archive, read, delete
client.group	WaGroupCoordinator	Groups and communities
client.newsletter	WaNewsletterCoordinator	Channels: create, send, follow, admin
client.status	WaStatusCoordinator	Status broadcast send and reactions
client.broadcastList	WaBroadcastListCoordinator	Broadcast list management and sends
client.privacy	WaPrivacyCoordinator	Privacy categories, blocklist
client.profile	WaProfileCoordinator	Profile picture, status text, username
client.business	WaBusinessCoordinator	Business profile, verified names
client.bot	WaBotCoordinator	Bot profiles and prompts (Meta AI and others)
client.email	WaEmailCoordinator	Bind/verify email on the account
client.lowlevel	WaLowLevelCoordinator	Raw node send/query escape hatch
Because the coordinator types are exported from the package root, you can annotate them in TypeScript:
import type { WaGroupCoordinator } from 'zapo-js'

const groups: WaGroupCoordinator = client.group
​
Data flow


Noise-encrypted frames

WhatsApp Web servers

Transport · binary node decode

Parsers / normalizers · src/client/events

Coordinators · emit typed events

Your listeners

Store · auth, signal, app-state, messages …

Incoming: frames are decoded into binary nodes, parsed and normalized into typed event payloads, then emitted (message, receipt, group, …).
Outgoing: your call to a coordinator (e.g. client.message.send) is built into a protocol node, encrypted, and written to the socket; the coordinator resolves once the server acks.
​
Engineering conventions
If you read the source, these conventions are pervasive and explain a lot of the API shape:
Uint8Array everywhere for binary data (Buffer is avoided), with zero-copy views in hot paths.
Named exports only — there are no default exports.
No enums — constants use Object.freeze({ ... } as const), surfaced as the WA_* objects.
Bounded in-memory structures to prevent unbounded growth in long-lived processes.
​
Authentication

Pair a device with a QR code or 8-character pairing code, persist Noise credentials across restarts, and cleanly log out of a WhatsApp session.

zapo connects as a companion device — exactly like linking WhatsApp Web or Desktop. The first connection pairs the device; after that, credentials stored in your store are reused automatically.
​
The pairing flow
Pairing is driven entirely through events emitted during connect():



WhatsApp
WaClient
Your app
WhatsApp
WaClient
Your app
user scans the QR / enters the code on their phone
connect()
open + Noise handshake
pairing required
auth_qr (or auth_pairing_code)
paired
auth_paired · credentials persisted
Event	Payload	When
auth_qr	{ qr: string, ttlMs: number }	A new QR is available to render. Re-emitted as it rotates.
auth_pairing_code	{ code: string }	An 8-character pairing code was issued (code flow).
auth_pairing_required	{ forceManual: boolean }	The session needs pairing input.
auth_passkey_required	{ hasSigner: boolean }	The server is forcing a passkey to link this device (see Passkey-gated linking below).
auth_paired	{ credentials: WaAuthCredentials }	Pairing succeeded; credentials are now persisted.
Once auth_paired fires, the credentials are written to the store and reused on every subsequent connect() — you will not see auth_qr again unless the session is unlinked or cleared.
Paired. Credentials now live in your store — restart the process and connect() resumes the session with no new QR.
​
Pairing with a QR code
This is the default flow. Render the qr string as a QR image and scan it from WhatsApp → Linked devices → Link a device.
import qrcode from 'qrcode-terminal'

client.on('auth_qr', ({ qr, ttlMs }) => {
  qrcode.generate(qr, { small: true })
  console.log(`QR valid for ${ttlMs}ms`)
})

client.on('auth_paired', ({ credentials }) => {
  console.log('Paired as', credentials.meJid)
})

await client.connect()
The QR rotates automatically; auth_qr fires again with a fresh value each time, so always render the latest one.
​
Pairing with a code
Prefer entering an 8-character code on the phone instead of scanning? Request one through client.auth after the connection is established. Listen for auth_pairing_required, then request the code for the target phone number (digits only, with country code):
client.on('auth_pairing_required', async () => {
  const code = await client.auth.requestPairingCode('5511999999999')
  // Format for display, e.g. "ABCD-1234"
  console.log('Enter on your phone:', code.match(/.{1,4}/g)?.join('-'))
})

client.once('auth_paired', () => console.log('Paired!'))

await client.connect()
requestPairingCode(phoneNumber, shouldShowPushNotification?, customCode?) requires an active connection and returns the code as a string. On the phone, open Linked devices → Link with phone number instead.
​
Passkey-gated linking (Shortcake)
For some accounts, WhatsApp’s server refuses the plain QR / pairing-code flow and demands a WebAuthn passkey assertion from the account owner’s authenticator before a companion can link. The wire protocol behind this is documented in depth in WhatsApp passkey & Shortcake linking; this section covers how to opt in on the client side.
The trigger is server-driven. When it fires, zapo emits auth_passkey_required as a heads-up (hasSigner tells you whether the handshake will actually proceed) and then runs the Shortcake handshake internally — provided you configured a signer.
​
The signer
Set signPasskeyAssertion on WaClientOptions. zapo hands you the server’s raw WebAuthn request options, and you hand back the assertion + credential id:
import { WaClient, type WaShortcakeAssertionSigner } from 'zapo-js'

const signPasskeyAssertion: WaShortcakeAssertionSigner = async (requestOptions) => {
  // requestOptions is a Uint8Array — the raw PublicKeyCredentialRequestOptions
  // JSON the server issued. Route it through your authenticator of choice.
  const { credentialId, webauthnAssertion } = await myAuthenticator.sign(requestOptions)
  return { credentialId, webauthnAssertion }
}

const client = new WaClient({
  store,
  sessionId: 'default',
  signPasskeyAssertion
}, logger)
The credential source stays outside the library — a real hardware/OS authenticator, a virtual authenticator, or a relay to another process — zapo never touches passkey material directly.
There is no headless bypass. Without a signer that produces an assertion the account owner’s own authenticator would sign (see the reverse-engineering page for why the wall holds), the link cannot complete on a passkey-gated account. When hasSigner is false on auth_passkey_required, the lib acks the prologue but the handshake stops — surface a UI prompt telling the user a passkey is required, and either configure a signer for the next attempt or fall back to a device that can complete it.
client.on('auth_passkey_required', ({ hasSigner }) => {
  if (hasSigner) {
    console.log('server is forcing a passkey — running Shortcake handshake…')
  } else {
    console.warn('passkey required but no signPasskeyAssertion is configured; link cannot proceed')
  }
})
auth_paired still fires on success — the Shortcake handshake is just an additional step slotted into the normal pairing flow (usually right after pair-device or a pairing-code companion_finish). No new “paired” event is introduced.
​
Credentials
After pairing, the current credentials are available synchronously:
const credentials = client.getCredentials() // WaAuthCredentials | null
console.log(credentials?.meJid)
WaAuthCredentials contains the device’s secret keys. It is marked @sensitive for a reason: anything that can read these can impersonate the device. If you persist them outside the built-in store, encrypt them at rest.
​
Logging out
logout() unpairs the companion device server-side (it removes this device from the account’s linked devices). It requires an authenticated session:
await client.logout()
By default this also clears stored state. You can control exactly which store domains are wiped on logout via the logoutStoreClear option — see Configuration.
​
Disconnect vs. logout
disconnect()	logout()
Closes the socket	Yes	Yes
Keeps credentials	Yes — reconnect later without re-pairing	No — device is unlinked
Server-side effect	None	Removes the linked device
Use disconnect() for a graceful shutdown you intend to resume; use logout() to permanently unlink.
​
Identities: phone numbers & LID

How WhatsApp’s phone-number JIDs (PN) and privacy LIDs differ, why both exist in multi-device, and how zapo maps and resolves between them.

WhatsApp addresses users two ways, and zapo surfaces both. Understanding the distinction matters as soon as you touch groups, because WhatsApp is migrating group identities to LID for privacy.
​
PN vs LID
Phone-number JID (PN)	LID
Form	5511999999999@s.whatsapp.net	<opaque-id>@lid
Server suffix	@s.whatsapp.net	@lid
Reveals the phone number	Yes	No
Detect with	!isLidJid(jid)	isLidJid(jid)
A LID (“linked identity”) is a stable, opaque identifier that represents a user without exposing their phone number. WhatsApp increasingly uses LIDs in groups and communities so members can interact without sharing their number.
import { isLidJid } from 'zapo-js'

isLidJid('5511999999999@s.whatsapp.net') // false  (PN)
isLidJid('199998888777@lid')             // true   (LID)
​
Addressing mode
When sending into a group, the message is addressed to participants either by PN or by LID — the addressingMode: 'pn' | 'lid'. zapo resolves this automatically: it scans the group participants and uses lid if any participant is a LID, otherwise pn. The server can confirm or override the choice, which is reflected back in the publish result:
const result = await client.message.send(groupJid, 'hi')
console.log(result.ack.addressingMode) // 'pn' | 'lid'
You normally don’t set this yourself — it’s derived from the group’s membership.
​
What you get on an incoming message
WaIncomingMessageEvent.key carries both identifiers when the server provides them. In groups the sender is key.participant; in 1:1 chats it is key.remoteJid. Each gets a parallel *Alt field with the alternate addressing.
Field	Meaning
key.remoteJid	The chat JID (group, 1:1 peer, broadcast, newsletter). In 1:1 chats this is also the sender.
key.remoteJidAlt	The alternate form of remoteJid (PN ↔ LID) in 1:1 chats, when the server shares it.
key.participant	The sender’s JID in groups / broadcasts (the addressing mode matches the chat).
key.participantAlt	The alternate form of participant in groups, when the server shares it.
key.recipientJid	Your receiving JID.
key.recipientAlt	The alternate form of the recipient (when available).
client.on('message', (event) => {
  const sender = event.key.participant ?? event.key.remoteJid
  const senderAlt = event.key.participantAlt ?? event.key.remoteJidAlt
  console.log('primary:', sender)    // e.g. 1999...@lid in a LID group
  console.log('alt:    ', senderAlt) // e.g. 5511...@s.whatsapp.net
})
So in a LID-addressed group you’ll typically see key.participant as a @lid and key.participantAlt as the phone JID (if the server shares it).
​
Replying — which JID to use
client.message.send accepts either a PN or a LID JID and normalizes the target for you, so you rarely have to convert:
// All valid:
await client.message.send('5511999999999', 'hi')                  // digits → PN JID
await client.message.send('5511999999999@s.whatsapp.net', 'hi')   // PN JID
await client.message.send('199998888777@lid', 'hi')               // LID JID
Always prefer sending by LID when you have one. The LID is the privacy-preserving, forward-compatible identity WhatsApp is migrating to — addressing a peer by LID is the future-proof choice and avoids leaking/relying on phone numbers. Fall back to the PN only when no LID is available.
Get the LID from the incoming event’s key.participantAlt / key.remoteJidAlt (when the primary is a PN) or resolve it with getLidsByPhoneNumbers.
In a group, always reply to event.key.remoteJid (the group JID), not to a participant’s JID. For 1:1 chats, prefer the peer’s LID; event.key.remoteJid also works whether it is a PN or a LID.
​
Mapping a phone number to its LID
To resolve LIDs for a set of phone numbers, use the profile coordinator:
const results = await client.profile.getLidsByPhoneNumbers([
  '5511999999999',
  '551188888888'
])

for (const r of results) {
  // SignalLidSyncResult: { queriedJid, phoneJid, lidJid, exists, invalid }
  console.log(r.queriedJid, '→', r.phoneJid, '→', r.lidJid,
    '(exists:', r.exists, 'invalid:', r.invalid, ')')
}
Each SignalLidSyncResult carries five fields:
Field	Type	Notes
queriedJid	string	The normalized JID you asked about — equal to phoneJid unless the server corrected it. Use this to correlate results back to your input by value rather than array position.
phoneJid	string	The server’s canonical form of the number. Differs from queriedJid when the server applied a fix like the Brazilian 9th digit (551188888888 → 5511988888888).
lidJid	string | null	The LID for this account. null when the server has no LID mapping.
exists	boolean	The number is a well-formed WhatsApp account.
invalid	boolean	The server rejected the number as malformed — distinct from a well-formed number that simply has no WhatsApp account (exists: false, invalid: false).
​
LID changes
A user’s LID can change (server-side, for privacy). When it does, you receive a mex_notification of kind lid_change:
client.on('mex_notification', (event) => {
  if (event.kind === 'lid_change') {
    console.log('LID changed:', event.oldLidJid, '→', event.newLidJid)
    // WaMexLidChangeEvent
  }
})
zapo handles the underlying Signal-session bookkeeping; this event is for your own caches/bookkeeping.
​
Where PN ↔ LID linkage is stored
Several app-state schemas track the relationship and sync it across your devices (see chat mutations):
Schema	Role
LidContact	Contact profile (name/username) keyed by a LID.
PnForLidChat	Remembers the phone JID for a chat that is primarily addressed by LID.
ShareOwnPn	Whether your own phone number is shared in a given context.
​
Signal sessions
Signal sessions are keyed by the canonical JID (PN or LID) plus device id. zapo canonicalizes hosted server variants (hosted.lid → lid, hosted → s.whatsapp.net) before lookups, and maintains sessions for both addressing forms. If you ever need to force a fresh session sync for a peer, use:
await client.message.syncSignalSession(jid)

Configuration

Configure WaClient: sessions, timeouts, history sync, presence on connect, addons, proxy, logging, logout cleanup, and production knobs.

WaClient takes a WaClientOptions object and an optional logger:
const client = new WaClient(options, logger)
Only store and sessionId are required; everything else has a sensible default.
​
Required options
​
store
WaStorerequired
The store instance built by createStore. Holds every per-session domain (auth, signal, app-state, …).
​
sessionId
stringrequired
Logical session identifier — it keys every domain inside store. Use a stable string per device/account. Changing it between runs orphans the previous credentials and forces re-pairing.
​
Sessions and multi-tenancy
Every store domain is keyed by sessionId, so a single store can hold many independent accounts. To run several accounts in one process, create one WaClient per sessionId over the same store:
const store = createStore({ /* ... */ })

const accountA = new WaClient({ store, sessionId: 'account-a' }, logger)
const accountB = new WaClient({ store, sessionId: 'account-b' }, logger)

await Promise.all([accountA.connect(), accountB.connect()])
Each client pairs and reconnects independently. For the full picture — what’s per-session vs shared, the single-writer rule across processes, memory budget, sharding, and shutdown — see Multi-session deployments.
​
Device fingerprint
These control how the device appears under Linked devices on the phone:
​
deviceBrowser
stringdefault:"'chrome'"
Browser id advertised during pairing ('chrome', 'firefox', 'safari', …; see WA_BROWSERS). Drives the Linked Devices label.
​
devicePlatform
string
Numeric companion platform id override (WA_COMPANION_PLATFORM_IDS). Inferred from deviceBrowser when omitted; set explicitly for non-browser platforms.
​
deviceOsDisplayName
string
Human-readable OS name shown under Linked devices ('Windows', 'Mac OS', 'Linux', …). Defaults to the current runtime’s OS.
​
deviceOsVersion
string
OS version advertised in DeviceProps.version ('10', '14.6', …). Defaults to the detected runtime OS version. Set this alongside deviceOsDisplayName when advertising an OS the process is not running on, so the name and version stay a matching pair. Values that are not dotted-numeric leave the field unset — matching how WhatsApp Web itself behaves.
new WaClient({
  store,
  sessionId: 'default',
  deviceBrowser: 'Chrome',
  deviceOsDisplayName: 'Mac OS',
  deviceOsVersion: '14.6'
}, logger)
For the MCP server the same overrides are available as MCP_DEVICE_OS_DISPLAY / MCP_DEVICE_OS_VERSION environment variables — when only the version is set the display name is still derived from the host, so pin both to keep the advertised pair consistent.


History sync
​
history
WaHistorySyncOptions
Controls processing of historySyncNotification chunks — both the initial bootstrap WhatsApp pushes after pairing and the on-demand backfill triggered by message.requestHistorySync.
enabled?: boolean — process incoming history chunks. Default true. Set to false to drop them silently (useful when you don’t persist mailbox/threads/contacts and the conversation download would just burn bandwidth). The lib still acks the chunk so the server stops re-sending it, matching wa-web.
requireFullSync?: boolean — request the full archive instead of just recent chats.
groupBundles?: boolean — opt into downloading the group-history bundle a member may share after somebody joins a group; emits group_history_bundle. Off by default — a bundle is media a third party pushes at this account unprompted, so fetching it is opt-in. Bundles addressed to other members are dropped either way. See Groups → sharing group history.
Chunks are decrypted, inflated, and parsed incrementally — one message is alive at a time as the proto stream descends into conversations, so a large chat that would materialize into hundreds of MB of JS objects stays flat. Group history bundles get the same treatment. There is no consumer-facing knob; peak memory during ingest just stopped tracking chunk size.
new WaClient({
  store,
  sessionId: 'default',
  history: { enabled: true, requireFullSync: true }
}, logger)
History arrives as history_sync_chunk events.
​
Timeouts
All in milliseconds; defaults are tuned for production.
Option	Purpose
iqTimeoutMs	Default timeout for IQ queries (default 60s).
nodeQueryTimeoutMs	Default timeout for raw node query() calls.
keepAliveIntervalMs	Interval between keep-alive ping IQs.
deadSocketTimeoutMs	How long without a reply before the socket is considered dead.
mediaTimeoutMs	Media upload/download timeout.
appStateSyncTimeoutMs	App-state sync round timeout.
messageAckTimeoutMs	How long message.send waits for the server <ack> per attempt.
messageMaxAttempts	Max attempts for a single message.send.
messageRetryDelayMs	Delay between message-send retries.
signalFetchKeyBundlesTimeoutMs	Timeout for Signal prekey-bundle fetches.

Search...


Navigation
Core concepts
Configuration
Core concepts
Configuration

Copy page

Configure WaClient: sessions, timeouts, history sync, presence on connect, addons, proxy, logging, logout cleanup, and production knobs.

WaClient takes a WaClientOptions object and an optional logger:
const client = new WaClient(options, logger)
Only store and sessionId are required; everything else has a sensible default.
​
Required options
​
store
WaStorerequired
The store instance built by createStore. Holds every per-session domain (auth, signal, app-state, …).
​
sessionId
stringrequired
Logical session identifier — it keys every domain inside store. Use a stable string per device/account. Changing it between runs orphans the previous credentials and forces re-pairing.
​
Sessions and multi-tenancy
Every store domain is keyed by sessionId, so a single store can hold many independent accounts. To run several accounts in one process, create one WaClient per sessionId over the same store:
const store = createStore({ /* ... */ })

const accountA = new WaClient({ store, sessionId: 'account-a' }, logger)
const accountB = new WaClient({ store, sessionId: 'account-b' }, logger)

await Promise.all([accountA.connect(), accountB.connect()])
Each client pairs and reconnects independently. For the full picture — what’s per-session vs shared, the single-writer rule across processes, memory budget, sharding, and shutdown — see Multi-session deployments.
​
Device fingerprint
These control how the device appears under Linked devices on the phone:
​
deviceBrowser
stringdefault:"'chrome'"
Browser id advertised during pairing ('chrome', 'firefox', 'safari', …; see WA_BROWSERS). Drives the Linked Devices label.
​
devicePlatform
string
Numeric companion platform id override (WA_COMPANION_PLATFORM_IDS). Inferred from deviceBrowser when omitted; set explicitly for non-browser platforms.
​
deviceOsDisplayName
string
Human-readable OS name shown under Linked devices ('Windows', 'Mac OS', 'Linux', …). Defaults to the current runtime’s OS.
​
deviceOsVersion
string
OS version advertised in DeviceProps.version ('10', '14.6', …). Defaults to the detected runtime OS version. Set this alongside deviceOsDisplayName when advertising an OS the process is not running on, so the name and version stay a matching pair. Values that are not dotted-numeric leave the field unset — matching how WhatsApp Web itself behaves.
new WaClient({
  store,
  sessionId: 'default',
  deviceBrowser: 'Chrome',
  deviceOsDisplayName: 'Mac OS',
  deviceOsVersion: '14.6'
}, logger)
For the MCP server the same overrides are available as MCP_DEVICE_OS_DISPLAY / MCP_DEVICE_OS_VERSION environment variables — when only the version is set the display name is still derived from the host, so pin both to keep the advertised pair consistent.
​
History sync
​
history
WaHistorySyncOptions
Controls processing of historySyncNotification chunks — both the initial bootstrap WhatsApp pushes after pairing and the on-demand backfill triggered by message.requestHistorySync.
enabled?: boolean — process incoming history chunks. Default true. Set to false to drop them silently (useful when you don’t persist mailbox/threads/contacts and the conversation download would just burn bandwidth). The lib still acks the chunk so the server stops re-sending it, matching wa-web.
requireFullSync?: boolean — request the full archive instead of just recent chats.
groupBundles?: boolean — opt into downloading the group-history bundle a member may share after somebody joins a group; emits group_history_bundle. Off by default — a bundle is media a third party pushes at this account unprompted, so fetching it is opt-in. Bundles addressed to other members are dropped either way. See Groups → sharing group history.
Chunks are decrypted, inflated, and parsed incrementally — one message is alive at a time as the proto stream descends into conversations, so a large chat that would materialize into hundreds of MB of JS objects stays flat. Group history bundles get the same treatment. There is no consumer-facing knob; peak memory during ingest just stopped tracking chunk size.
new WaClient({
  store,
  sessionId: 'default',
  history: { enabled: true, requireFullSync: true }
}, logger)
History arrives as history_sync_chunk events.
​
Timeouts
All in milliseconds; defaults are tuned for production.
Option	Purpose
iqTimeoutMs	Default timeout for IQ queries (default 60s).
nodeQueryTimeoutMs	Default timeout for raw node query() calls.
keepAliveIntervalMs	Interval between keep-alive ping IQs.
deadSocketTimeoutMs	How long without a reply before the socket is considered dead.
mediaTimeoutMs	Media upload/download timeout.
appStateSyncTimeoutMs	App-state sync round timeout.
messageAckTimeoutMs	How long message.send waits for the server <ack> per attempt.
messageMaxAttempts	Max attempts for a single message.send.
messageRetryDelayMs	Delay between message-send retries.
signalFetchKeyBundlesTimeoutMs	Timeout for Signal prekey-bundle fetches.
​
WhatsApp version
zapo ships with a tested production version baked in per transport. WhatsApp occasionally rejects older clients during the noise handshake with HTTP 405 / failure_client_too_old. You have three options to recover.
​
version
string | () => string | Promise<string>
Override the version string the client advertises. Either a literal or a resolver invoked once per connect() — useful for fetching the current version lazily without rebuilding the client. The accepted shape depends on the transport resolved for the connect:
Web takes a 3- to 5-part version (2.3000.x[.y.z]); the 4th and 5th parts, when supplied, are advertised in the noise payload.
Mobile takes exactly a 4-part Android app version (2.26.x.y); it overrides mobileTransport.deviceInfo.appVersion in the login payload.
An invalid part count for the resolved transport throws on connect().
​
recoverFromClientTooOld
booleandefault:"false"
When true, on failure_client_too_old the client logs a warning, fetches the current version for the active transport (fetchLatestWaWebVersion() for Web, fetchLatestWaMobileVersion() for Mobile), applies it as a one-shot override, and reconnects automatically. On Mobile the override is applied by refreshing deviceInfo.appVersion for the next connect. Treat it as a stopgap until you upgrade zapo — the bundled default is still the recommended path.
import { WaClient, fetchLatestWaWebVersion } from 'zapo-js'

// Pin a specific version
new WaClient({ store, sessionId: 'default', version: '2.3000.1027421623' }, logger)

// Resolve lazily on each connect()
new WaClient({
  store,
  sessionId: 'default',
  version: async () => (await fetchLatestWaWebVersion()).version
}, logger)

// Auto-recover from HTTP 405 once
new WaClient({ store, sessionId: 'default', recoverFromClientTooOld: true }, logger)
​
fetchLatestWaWebVersion()
Scrapes the current client_revision from web.whatsapp.com/sw.js and returns a version string in the 2.3000.x form accepted by version for a Web session.
import { fetchLatestWaWebVersion } from 'zapo-js'

const { version, parts } = await fetchLatestWaWebVersion({
  timeoutMs: 10_000,
  // Route through the same dispatcher you use for media / link-preview
  proxy: dispatcher
})
Options: timeoutMs (default 10s), proxy (undici dispatcher only — http.Agent is not honored by the global fetch), signal, userAgent, headers, and a fetch override for tests. Network and parse errors throw — wrap in try/catch if you want to fall back to the bundled default.

Presence on connect

markOnlineOnConnect
booleandefault:"false"
false (default) — announce as unavailable. Matches WhatsApp Web when the tab is not focused, and keeps headless bots invisible by default. With this off, you keep receiving notifications for messages while “offline”.
true — announce the client as online (matches WhatsApp Web with the tab focused at login time).

Passkey-gated linking

signPasskeyAssertion
WaShortcakeAssertionSigner
External WebAuthn signer for the server-forced Shortcake passkey handshake. Called with the raw PublicKeyCredentialRequestOptions (Uint8Array) the server issued; must return { credentialId, webauthnAssertion }. The credential source (real / virtual authenticator, relay) stays outside the library.
Without this, an account that gets a server-forced passkey prologue emits auth_passkey_required with hasSigner: false and the link stalls — see the reverse-engineering deep dive for the wire-level detail.

Addons (reactions, poll votes)

addons
{ autoDecrypt?: boolean, persistAllSecrets?: boolean }default:"{ autoDecrypt: true, persistAllSecrets: false }"
Encrypted addons (poll votes, reactions, message edits, …) are decrypted automatically and emitted as typed message_addon events. Set autoDecrypt: false to receive them encrypted and decrypt yourself via client.message.tryDecryptAddon(event). The parent message secret is looked up in the messageSecret cache first, then in the messages store.
persistAllSecrets: true persists the 32-byte message secret of every sent and received message, not just the poll / event / bot-prompt ones the library knows will get a follow-up. Encrypted addons whose parent can be any message type — reactions, comments, secretEncryptedMessage edits — need the parent’s secret to decrypt; without this flag, those parents stay decryptable across a restart only when the full messages archive is persistent. Use it to keep them decryptable while storing only the secret (messages: 'none').
Has no effect when the messageSecret cache is 'none' — every secret write lands in the noop store and is silently discarded. With the default 'memory' provider it works for the lifetime of the process but is lost on restart and bounded by the cache’s LRU and messageSecretMs TTL; point messageSecret at a persistent backend to keep secrets across restarts.

Media

media
WaMediaOptions
Media processing. Pass a processor (from @zapo-js/media-utils) to generate thumbnails/previews, probe dimensions and durations, and build voice-note waveforms before upload — then toggle each step. Without a processor media still uploads, just without this processing. See the media guide for the full wiring.
processor?: WaMediaProcessor — the processor instance
generateThumbnail?: boolean — image/video preview thumbnails
generateProbe?: boolean — probe width/height/duration
generateWaveform?: boolean — voice-note (PTT) waveform
generateStickerThumbnail?: boolean
normalizeVoiceNote?: boolean — re-encode PTT audio to the format WhatsApp expects

Link previews

linkPreview
WaLinkPreviewOptions
Global configuration for the built-in link-preview fetcher used when sending text that contains a URL. Override per message with the linkPreview send option.
enabled?: boolean — turn automatic link-preview fetching on or off globally
fetchTimeoutMs?: number — how long to wait for the target page
uploadHqThumbnail?: boolean — upload a high-resolution preview thumbnail
allowPrivateHosts?: boolean — allow fetching private/loopback addresses (off by default, as an SSRF guard)
maxHtmlBytes?: number / maxThumbnailBytes?: number — size caps for the fetched HTML and image
userAgent?: string — User-Agent sent when fetching
proxy?: WaProxyTransport — proxy just this fetcher (same as proxy.linkPreview)
fetcher?: WaLinkPreviewFetcher — replace the default fetcher entirely (e.g. your own scraping pipeline)


Core concepts
Configuration
Core concepts
Configuration

Copy page

Configure WaClient: sessions, timeouts, history sync, presence on connect, addons, proxy, logging, logout cleanup, and production knobs.

WaClient takes a WaClientOptions object and an optional logger:
const client = new WaClient(options, logger)
Only store and sessionId are required; everything else has a sensible default.
​
Required options
​
store
WaStorerequired
The store instance built by createStore. Holds every per-session domain (auth, signal, app-state, …).
​
sessionId
stringrequired
Logical session identifier — it keys every domain inside store. Use a stable string per device/account. Changing it between runs orphans the previous credentials and forces re-pairing.
​
Sessions and multi-tenancy
Every store domain is keyed by sessionId, so a single store can hold many independent accounts. To run several accounts in one process, create one WaClient per sessionId over the same store:
const store = createStore({ /* ... */ })

const accountA = new WaClient({ store, sessionId: 'account-a' }, logger)
const accountB = new WaClient({ store, sessionId: 'account-b' }, logger)

await Promise.all([accountA.connect(), accountB.connect()])
Each client pairs and reconnects independently. For the full picture — what’s per-session vs shared, the single-writer rule across processes, memory budget, sharding, and shutdown — see Multi-session deployments.
​
Device fingerprint
These control how the device appears under Linked devices on the phone:
​
deviceBrowser
stringdefault:"'chrome'"
Browser id advertised during pairing ('chrome', 'firefox', 'safari', …; see WA_BROWSERS). Drives the Linked Devices label.
​
devicePlatform
string
Numeric companion platform id override (WA_COMPANION_PLATFORM_IDS). Inferred from deviceBrowser when omitted; set explicitly for non-browser platforms.
​
deviceOsDisplayName
string
Human-readable OS name shown under Linked devices ('Windows', 'Mac OS', 'Linux', …). Defaults to the current runtime’s OS.
​
deviceOsVersion
string
OS version advertised in DeviceProps.version ('10', '14.6', …). Defaults to the detected runtime OS version. Set this alongside deviceOsDisplayName when advertising an OS the process is not running on, so the name and version stay a matching pair. Values that are not dotted-numeric leave the field unset — matching how WhatsApp Web itself behaves.
new WaClient({
  store,
  sessionId: 'default',
  deviceBrowser: 'Chrome',
  deviceOsDisplayName: 'Mac OS',
  deviceOsVersion: '14.6'
}, logger)
For the MCP server the same overrides are available as MCP_DEVICE_OS_DISPLAY / MCP_DEVICE_OS_VERSION environment variables — when only the version is set the display name is still derived from the host, so pin both to keep the advertised pair consistent.
​
History sync
​
history
WaHistorySyncOptions
Controls processing of historySyncNotification chunks — both the initial bootstrap WhatsApp pushes after pairing and the on-demand backfill triggered by message.requestHistorySync.
enabled?: boolean — process incoming history chunks. Default true. Set to false to drop them silently (useful when you don’t persist mailbox/threads/contacts and the conversation download would just burn bandwidth). The lib still acks the chunk so the server stops re-sending it, matching wa-web.
requireFullSync?: boolean — request the full archive instead of just recent chats.
groupBundles?: boolean — opt into downloading the group-history bundle a member may share after somebody joins a group; emits group_history_bundle. Off by default — a bundle is media a third party pushes at this account unprompted, so fetching it is opt-in. Bundles addressed to other members are dropped either way. See Groups → sharing group history.
Chunks are decrypted, inflated, and parsed incrementally — one message is alive at a time as the proto stream descends into conversations, so a large chat that would materialize into hundreds of MB of JS objects stays flat. Group history bundles get the same treatment. There is no consumer-facing knob; peak memory during ingest just stopped tracking chunk size.
new WaClient({
  store,
  sessionId: 'default',
  history: { enabled: true, requireFullSync: true }
}, logger)
History arrives as history_sync_chunk events.
​
Timeouts
All in milliseconds; defaults are tuned for production.
Option	Purpose
iqTimeoutMs	Default timeout for IQ queries (default 60s).
nodeQueryTimeoutMs	Default timeout for raw node query() calls.
keepAliveIntervalMs	Interval between keep-alive ping IQs.
deadSocketTimeoutMs	How long without a reply before the socket is considered dead.
mediaTimeoutMs	Media upload/download timeout.
appStateSyncTimeoutMs	App-state sync round timeout.
messageAckTimeoutMs	How long message.send waits for the server <ack> per attempt.
messageMaxAttempts	Max attempts for a single message.send.
messageRetryDelayMs	Delay between message-send retries.
signalFetchKeyBundlesTimeoutMs	Timeout for Signal prekey-bundle fetches.
​
WhatsApp version
zapo ships with a tested production version baked in per transport. WhatsApp occasionally rejects older clients during the noise handshake with HTTP 405 / failure_client_too_old. You have three options to recover.
​
version
string | () => string | Promise<string>
Override the version string the client advertises. Either a literal or a resolver invoked once per connect() — useful for fetching the current version lazily without rebuilding the client. The accepted shape depends on the transport resolved for the connect:
Web takes a 3- to 5-part version (2.3000.x[.y.z]); the 4th and 5th parts, when supplied, are advertised in the noise payload.
Mobile takes exactly a 4-part Android app version (2.26.x.y); it overrides mobileTransport.deviceInfo.appVersion in the login payload.
An invalid part count for the resolved transport throws on connect().
​
recoverFromClientTooOld
booleandefault:"false"
When true, on failure_client_too_old the client logs a warning, fetches the current version for the active transport (fetchLatestWaWebVersion() for Web, fetchLatestWaMobileVersion() for Mobile), applies it as a one-shot override, and reconnects automatically. On Mobile the override is applied by refreshing deviceInfo.appVersion for the next connect. Treat it as a stopgap until you upgrade zapo — the bundled default is still the recommended path.
import { WaClient, fetchLatestWaWebVersion } from 'zapo-js'

// Pin a specific version
new WaClient({ store, sessionId: 'default', version: '2.3000.1027421623' }, logger)

// Resolve lazily on each connect()
new WaClient({
  store,
  sessionId: 'default',
  version: async () => (await fetchLatestWaWebVersion()).version
}, logger)

// Auto-recover from HTTP 405 once
new WaClient({ store, sessionId: 'default', recoverFromClientTooOld: true }, logger)
​
fetchLatestWaWebVersion()
Scrapes the current client_revision from web.whatsapp.com/sw.js and returns a version string in the 2.3000.x form accepted by version for a Web session.
import { fetchLatestWaWebVersion } from 'zapo-js'

const { version, parts } = await fetchLatestWaWebVersion({
  timeoutMs: 10_000,
  // Route through the same dispatcher you use for media / link-preview
  proxy: dispatcher
})
Options: timeoutMs (default 10s), proxy (undici dispatcher only — http.Agent is not honored by the global fetch), signal, userAgent, headers, and a fetch override for tests. Network and parse errors throw — wrap in try/catch if you want to fall back to the bundled default.
​
fetchLatestWaMobileVersion()
Scrapes the current WhatsApp for Android version from a public app-listing page and returns a 4-part 2.26.x.y string suitable for version on a Mobile session (or as an override for mobileTransport.deviceInfo.appVersion).
import { fetchLatestWaMobileVersion } from 'zapo-js'

const { version, parts } = await fetchLatestWaMobileVersion({
  timeoutMs: 10_000,
  proxy: dispatcher
})
Options: everything the Web fetcher accepts (timeoutMs, proxy, signal, userAgent, headers, fetch) plus:
url?: string — override the page to scrape. The default source is a public app-listing mirror because WhatsApp’s own whatsapp.com/android page only shows the stale minimum-requirement version; retarget if the layout changes or is unreachable from your network.
versionPattern?: RegExp — override the extraction regex. Must expose the version in capture group 1. The default matches a 4-part 2.x.x.x and returns the first hit on the page.
The parsed string must have exactly four numeric parts; anything else throws (invalid wa-mobile version parsed from page). Network and parse errors throw — wrap in try/catch if you want to fall back to a known-good hardcoded version.
​
Presence on connect
​
markOnlineOnConnect
booleandefault:"false"
false (default) — announce as unavailable. Matches WhatsApp Web when the tab is not focused, and keeps headless bots invisible by default. With this off, you keep receiving notifications for messages while “offline”.
true — announce the client as online (matches WhatsApp Web with the tab focused at login time).
​
Passkey-gated linking
​
signPasskeyAssertion
WaShortcakeAssertionSigner
External WebAuthn signer for the server-forced Shortcake passkey handshake. Called with the raw PublicKeyCredentialRequestOptions (Uint8Array) the server issued; must return { credentialId, webauthnAssertion }. The credential source (real / virtual authenticator, relay) stays outside the library.
Without this, an account that gets a server-forced passkey prologue emits auth_passkey_required with hasSigner: false and the link stalls — see the reverse-engineering deep dive for the wire-level detail.
​
Addons (reactions, poll votes)
​
addons
{ autoDecrypt?: boolean, persistAllSecrets?: boolean }default:"{ autoDecrypt: true, persistAllSecrets: false }"
Encrypted addons (poll votes, reactions, message edits, …) are decrypted automatically and emitted as typed message_addon events. Set autoDecrypt: false to receive them encrypted and decrypt yourself via client.message.tryDecryptAddon(event). The parent message secret is looked up in the messageSecret cache first, then in the messages store.
persistAllSecrets: true persists the 32-byte message secret of every sent and received message, not just the poll / event / bot-prompt ones the library knows will get a follow-up. Encrypted addons whose parent can be any message type — reactions, comments, secretEncryptedMessage edits — need the parent’s secret to decrypt; without this flag, those parents stay decryptable across a restart only when the full messages archive is persistent. Use it to keep them decryptable while storing only the secret (messages: 'none').
Has no effect when the messageSecret cache is 'none' — every secret write lands in the noop store and is silently discarded. With the default 'memory' provider it works for the lifetime of the process but is lost on restart and bounded by the cache’s LRU and messageSecretMs TTL; point messageSecret at a persistent backend to keep secrets across restarts.
​
Media
​
media
WaMediaOptions
Media processing. Pass a processor (from @zapo-js/media-utils) to generate thumbnails/previews, probe dimensions and durations, and build voice-note waveforms before upload — then toggle each step. Without a processor media still uploads, just without this processing. See the media guide for the full wiring.
processor?: WaMediaProcessor — the processor instance
generateThumbnail?: boolean — image/video preview thumbnails
generateProbe?: boolean — probe width/height/duration
generateWaveform?: boolean — voice-note (PTT) waveform
generateStickerThumbnail?: boolean
normalizeVoiceNote?: boolean — re-encode PTT audio to the format WhatsApp expects
​
Link previews
​
linkPreview
WaLinkPreviewOptions
Global configuration for the built-in link-preview fetcher used when sending text that contains a URL. Override per message with the linkPreview send option.
enabled?: boolean — turn automatic link-preview fetching on or off globally
fetchTimeoutMs?: number — how long to wait for the target page
uploadHqThumbnail?: boolean — upload a high-resolution preview thumbnail
allowPrivateHosts?: boolean — allow fetching private/loopback addresses (off by default, as an SSRF guard)
maxHtmlBytes?: number / maxThumbnailBytes?: number — size caps for the fetched HTML and image
userAgent?: string — User-Agent sent when fetching
proxy?: WaProxyTransport — proxy just this fetcher (same as proxy.linkPreview)
fetcher?: WaLinkPreviewFetcher — replace the default fetcher entirely (e.g. your own scraping pipeline)
​
Chat events
​
chatEvents
{ emitSnapshotMutations?: boolean }
Set emitSnapshotMutations: true to re-emit mutation events for every change seen during an app-state snapshot sync. Off by default, since snapshot mutations represent historical state rather than live changes.
​
Write-behind persistence
​
writeBehind
WaWriteBehindOptions
Batches incoming messages before flushing to the messages / threads / contacts stores.
maxPendingKeys?: number
maxWriteConcurrency?: number
flushTimeoutMs?: number
​
Proxy
​
proxy
WaClientProxyOptions
Route each leg through a proxy independently:
ws — the WebSocket connection.
mediaUpload / mediaDownload — media transfers.
linkPreview — the default link-preview fetcher.
Each leg accepts a WaProxyTransport, which is either:
an undici dispatcher (WaProxyDispatcher, e.g. an undici ProxyAgent) — used for the fetch-based legs (media, link preview), or
a Node http/https Agent (WaProxyAgent) — used for the WebSocket (ws) leg.
zapo picks the right form per leg automatically.
The ws leg requires the ws package, because the runtime’s native WebSocket cannot accept an HTTP Agent. Without a proxy, no extra package is needed.
​
HTTP / HTTPS proxy
Use an undici ProxyAgent (a dispatcher) for the media/link-preview legs, and an https-proxy-agent (an http.Agent) for the ws leg:
import { ProxyAgent } from 'undici'
import { HttpsProxyAgent } from 'https-proxy-agent'

const url = 'http://user:pass@proxy.example.com:8080' // or https://…
const dispatcher = new ProxyAgent(url)
const wsAgent = new HttpsProxyAgent(url)

const client = new WaClient({
  store,
  sessionId: 'default',
  proxy: {
    ws: wsAgent,
    mediaUpload: dispatcher,
    mediaDownload: dispatcher,
    linkPreview: dispatcher
  }
}, logger)

SOCKS proxy
Use socks-proxy-agent (works as an http.Agent for every leg, including ws):
import { SocksProxyAgent } from 'socks-proxy-agent'

// socks5 (or socks4) — host can be a domain or an IP
const agent = new SocksProxyAgent('socks5://user:pass@127.0.0.1:1080')

const client = new WaClient({
  store,
  sessionId: 'default',
  proxy: { ws: agent, mediaUpload: agent, mediaDownload: agent, linkPreview: agent }
}, logger)
​
IPv4 and IPv6 hosts
The proxy host can be a domain or an IP literal. IPv6 addresses must be wrapped in brackets:
// IPv4
new ProxyAgent('http://203.0.113.10:8080')
new SocksProxyAgent('socks5://203.0.113.10:1080')

// IPv6 — bracket the address
new ProxyAgent('http://[2001:db8::1]:8080')
new SocksProxyAgent('socks5://[2001:db8::1]:1080')

// With credentials
new ProxyAgent('http://user:pass@[2001:db8::1]:8080')
Point only the legs you need at a proxy — e.g. set just ws to tunnel the connection while letting media transfer directly, or vice-versa.
​
Logout store clearing
​
logoutStoreClear
WaLogoutStoreClearOptions
Per-domain control over what logout() wipes.
By default, the mailbox archive (messages, threads, contacts) is preserved so the user keeps their history when re-pairing. Every other domain (credentials, Signal state, app-state, caches, privacy tokens) is cleared to start the next pair clean. Explicit true / false always wins over the default.
// Preserve everything except auth (re-pair without touching state)
logoutStoreClear: { signal: false, appState: false }

// Wipe the mailbox too (full reset)
logoutStoreClear: { messages: true, threads: true, contacts: true }
​
Logging
WaClient accepts a Logger as the second constructor argument. Omit it and a default ConsoleLogger('info') is used. Levels, lowest to highest: trace, debug, info, warn, error.
Two implementations ship with the package.
​
ConsoleLogger
Zero-dependency. Writes structured records to console.log / console.warn / console.error. Good for development, tests, and serverless functions where you cannot add a logger transport.
import { ConsoleLogger } from 'zapo-js'

const client = new WaClient(options, new ConsoleLogger('info'))

createPinoLogger
Async factory that dynamically loads pino (and pino-pretty when pretty: true), configures it, and wraps it in a PinoLogger adapter. Throws optional dependency "pino" is not installed when pino is missing — install with npm i pino pino-pretty.
import { createPinoLogger } from 'zapo-js'

const logger = await createPinoLogger({ level: 'info', pretty: true })
const client = new WaClient(options, logger)
Field	Type	Description
level	LogLevel	Minimum level to emit. Default 'info'.
name	string	Pino instance name attached to every record.
base	Record<string, unknown> | null	Base bindings merged into every record. Pass null to drop pino’s default pid/hostname.
pinoOptions	Record<string, unknown>	Passthrough into pino() for anything not surfaced above (redaction, custom serializers, …).
pretty	boolean	When true, wires pino-pretty as the transport. Keep at false (default) in production to emit JSON lines.
prettyOptions	PinoPrettyOptions	Forwarded into the pino-pretty transport — see the pino-pretty options.
​
PinoLogger (bring your own Pino)
If you already configure Pino centrally — child loggers, custom transports, file destinations — construct PinoLogger directly to wrap your existing instance. The factory is a convenience; the class is the actual adapter, and using it skips the dynamic pino import.
import pino from 'pino'
import { PinoLogger } from 'zapo-js'

const root = pino({ name: 'my-app', transport: { /* ... */ } })
const child = root.child({ component: 'whatsapp' })

const client = new WaClient(options, new PinoLogger(child, 'info'))
The signature is new PinoLogger(logger, level = 'info'). The level is forwarded to logger.level and used as the adapter’s reported level.


Custom logger
Need a sink the built-in implementations don’t cover — Datadog, OpenTelemetry, syslog, an internal observability pipeline? Implement the Logger interface and pass an instance to WaClient. The interface is small:
import type { Logger, LogLevel } from 'zapo-js'

interface Logger {
  readonly level: LogLevel
  trace(message: string, context?: Readonly<Record<string, unknown>>): void
  debug(message: string, context?: Readonly<Record<string, unknown>>): void
  info(message: string, context?: Readonly<Record<string, unknown>>): void
  warn(message: string, context?: Readonly<Record<string, unknown>>): void
  error(message: string, context?: Readonly<Record<string, unknown>>): void
  /**
   * Returns a derived logger that pre-binds `bindings` into every log call's
   * context. Bindings stack: `parent.child(a).child(b)` merges `{ ...a, ...b }`.
   * Per-call context wins on key conflicts.
   */
  child(bindings: Readonly<Record<string, unknown>>): Logger
}
LogLevel is 'trace' | 'debug' | 'info' | 'warn' | 'error'. The library calls the five level methods directly — there is no level-gating layer in front, so your implementation is responsible for filtering against this.level if you want to skip cheap calls.
A minimal example that forwards to an external sink and tracks bindings through child():
import type { Logger, LogLevel } from 'zapo-js'

const LEVEL_RANK: Record<LogLevel, number> = {
  trace: 10, debug: 20, info: 30, warn: 40, error: 50
}

class MyLogger implements Logger {
  constructor(
    public readonly level: LogLevel = 'info',
    private readonly bindings: Readonly<Record<string, unknown>> = {}
  ) {}

  private write(at: LogLevel, message: string, context?: Readonly<Record<string, unknown>>): void {
    if (LEVEL_RANK[at] < LEVEL_RANK[this.level]) return
    sendToObservability({ level: at, message, ...this.bindings, ...context })
  }

  trace(message: string, context?: Readonly<Record<string, unknown>>) { this.write('trace', message, context) }
  debug(message: string, context?: Readonly<Record<string, unknown>>) { this.write('debug', message, context) }
  info(message: string, context?: Readonly<Record<string, unknown>>)  { this.write('info',  message, context) }
  warn(message: string, context?: Readonly<Record<string, unknown>>)  { this.write('warn',  message, context) }
  error(message: string, context?: Readonly<Record<string, unknown>>) { this.write('error', message, context) }

  child(bindings: Readonly<Record<string, unknown>>): Logger {
    return new MyLogger(this.level, { ...this.bindings, ...bindings })
  }
}

const client = new WaClient(options, new MyLogger('info'))
child() is used internally to attach per-component bindings (e.g. { component: 'noise' }, { component: 'signal', sessionId }). Returning a new instance with merged bindings — instead of mutating — keeps those tags scoped to the producing subsystem.

Plugins
​
plugins
readonly WaClientPluginDefinition[]
Optional WaClient plugins — behavior hooks and/or coordinators exposed at client[exposeAs]. Authored with defineWaClientPlugin. The voice-calling plugin (@zapo-js/voip) is the reference implementation; see the plugin system page for how to wire and author plugins.
import { voipPlugin } from '@zapo-js/voip'

new WaClient({
  store,
  sessionId: 'default',
  plugins: [voipPlugin()]
}, logger)
​
Advanced options
Rarely needed — listed for completeness.
chatSocketUrls?: readonly string[] — override the WhatsApp chat WebSocket endpoint list (e.g. to route through a fake server in tests, or pin a specific edge).
privacyToken?: WaPrivacyTokenOptions — tune trusted-contact-token (TC token) issuance: token durations and bucket counts.
testHooks?: WaClientTestHooks — test-only fixtures (e.g. a custom Noise root CA). These do not bypass any security check; to actually skip a check, use the dangerous options below.
​
Dangerous options
dangerous flags each disable a security check the production path enforces (signature verification, app-state MAC checks, …). They exist for testing against a fake server. Never enable them in production.

Plugins

Extend WaClient with typed plugins: register stanza handlers, expose new coordinators at client[exposeAs], and contribute typed events that only exist when the plugin is installed.

The plugin system lets you extend WaClient without forking it. A plugin can hook into the incoming stanza pipeline, expose its own coordinator at client[exposeAs], contribute typed events to client.on, and clean up on disconnect — all with full TypeScript inference, no global module augmentation.
@zapo-js/voip is the reference implementation; see the VoIP guide for what a finished plugin looks like.
​
What a plugin is
Every plugin is a WaClientPluginDefinition value passed via the plugins option:
new WaClient({ store, sessionId: 'default', plugins: [myPlugin()] }, logger)
There are two flavors:
Behavior plugins run side effects (register stanza handlers, listen to events, schedule work) and don’t expose anything on the client.
Expose plugins also publish a value at client[exposeAs] — typically a coordinator object, like client.voip.
Both flavors can contribute typed events that surface on client.on / client.once / client.off only when the plugin is in the plugins array. The inference is value-derived: if the plugin is not installed, the event name doesn’t exist on the client type.
​
Authoring a plugin
Use defineWaClientPlugin for type-safe authoring. It has two overloads.
​
Behavior plugin
import { defineWaClientPlugin } from 'zapo-js'

export function loggingPlugin() {
  return defineWaClientPlugin({
    id: 'example/logging',
    setup(ctx) {
      const unregister = ctx.registerIncomingHandler({
        tag: 'message',
        handler: async (node) => {
          ctx.logger.info('saw <message>', { from: node.attrs.from })
          // return false so the core handler still runs
          return false
        }
      })
      ctx.registerDispose(() => unregister())
    }
  })
}
​
Expose plugin
The expose overload takes three type parameters: the exposeAs key as a string literal, the type of the value returned by setup, and the event-map type. Inference flows from the value back into client.on and client[exposeAs]:
import { defineWaClientPlugin } from 'zapo-js'

interface MetricsEvents {
  readonly metrics_tick: (payload: { readonly count: number }) => void
}

class Metrics {
  count = 0
  increment(): void {
    this.count += 1
  }
}

export function metricsPlugin() {
  return defineWaClientPlugin<'metrics', Metrics, MetricsEvents>({
    id: 'example/metrics',
    exposeAs: 'metrics',
    setup(ctx) {
      const metrics = new Metrics()
      const interval = setInterval(() => {
        ctx.emit('metrics_tick', { count: metrics.count })
      }, 1000)
      ctx.registerDispose(() => clearInterval(interval))
      return metrics
    },
    dispose(metrics) {
      // metrics is typed as Metrics here
      metrics.count = 0
    }
  })
}
Once you wire metricsPlugin() into plugins, client.metrics is typed as Metrics, and client.on('metrics_tick', ...) is type-checked against MetricsEvents.
The exposeAs parameter is constrained to K extends keyof WaClient ? never : K. Trying to expose a name that already exists on the client — 'message', 'group', 'voip' if voip is also installed — is a TypeScript error at the call site.
​
Plugin context
setup receives a WaClientPluginContext:
Field	Type	What it gives you
client	WaClient	The host client, for late binding (handlers can read client.getCredentials(), etc.).
options	Readonly<WaClientOptions>	The exact options the client was constructed with.
logger	Logger	Child logger already tagged with { plugin: id }.
stores	session stores	Same per-session domains the coordinators use.
deps	WaClientDependencies	Coordinator dependency graph (advanced).
emit	function	Emit a (typed) event on the host client.
on / off / once	functions	Subscribe to events on the host client.
queryWithContext	function	Send an IQ-style query and await the response node.
registerIncomingHandler	function	Hook the incoming stanza pipeline; see below.
registerIncomingStanzaFilter	function	Drop or rewrite stanzas before the core handlers run.
registerDispose	function	Schedule a teardown callback. Runs LIFO on WaClient.disconnect.
ctx.deps is an advanced API for plugin authors — new coordinators may appear in minor releases. Some nested coordinators reach key material, so do not log or persist deps wholesale.
​
Incoming handlers
registerIncomingHandler({ tag, prepend, handler }) lets you intercept a specific stanza tag ('message', 'iq', 'call', 'ack', 'receipt', …). Return true from handler to signal you handled the stanza (the core client won’t ack again); return false to let the rest of the pipeline run.
ctx.registerIncomingHandler({
  tag: 'call',
  prepend: true,
  handler: async (node) => {
    // run before the core handler; returning true claims the stanza
    return true
  }
})
registerIncomingStanzaFilter runs even earlier and can drop stanzas entirely.
​
Typed event extension
Plugins thread their events into the host client through a phantom __pluginEvents marker carried by the return value of defineWaClientPlugin. The third generic parameter is the event map:
defineWaClientPlugin<'voip', WaVoipCoordinator, VoipEvents>({ /* ... */ })
WaClient’s event type is derived from the plugins tuple via WaClientPluginEventsFromPlugins. The practical effect: a voip_* event exists on client.on only when voipPlugin() is in plugins.
const client = new WaClient({
  store,
  sessionId: 'default',
  plugins: [voipPlugin()]
}, logger)

// OK: voipPlugin contributes voip_call_incoming
client.on('voip_call_incoming', (call) => { /* ... */ })

// Without voipPlugin() in plugins, the line above is a type error.
There is no global module augmentation — installing a different WaClient in the same project doesn’t inherit foreign plugin events.
​
Lifecycle
Plugins install during new WaClient(...):
The client is constructed and its coordinators wired.
installWaClientPlugins iterates the plugins array in order. For each plugin:
The id is checked against previously seen ids.
If exposeAs is set, the name is checked against other plugins and against existing members of WaClient.
setup(ctx) runs. The return value (for expose plugins) is published as a non-configurable, non-writable getter on the client.
On client.disconnect(), after incoming handlers drain, the registered dispose callbacks run in reverse (LIFO) order. A throwing dispose is logged and the rest still run.
On the next client.connect(), plugins are reinstalled from scratch — the previous disconnect() disposed them, so connect() runs setup(ctx) again. client[exposeAs] accessors keep working across a reconnect (they resolve against the freshly-installed coordinator instance), and setup sees a clean context every time.
​
Error conditions
installWaClientPlugins throws synchronously during client construction in these cases:
Duplicate id — "duplicate wa client plugin id: <id>". Each plugin must have a unique id across the plugins array.
Duplicate exposeAs — "duplicate wa client plugin exposeAs: <name>". Two expose plugins cannot publish the same name.
Collision with a built-in — "wa client plugin exposeAs \"<name>\" collides with a reserved client member". Trying exposeAs: 'message', 'group', 'on', etc. fails at runtime (and at compile time via the keyof WaClient guard).
// throws: duplicate wa client plugin id
new WaClient({
  store,
  sessionId: 'default',
  plugins: [voipPlugin(), voipPlugin()]
})

// throws: collides with a reserved client member
const badPlugin = defineWaClientPlugin({
  id: 'example/bad',
  exposeAs: 'message' as never,
  setup: () => ({})
})
​
Events

Reference for every WaClient event you can subscribe to: messages, receipts, groups, history sync, app-state mutations, and failures.

WaClient is a strongly-typed event emitter. Every incoming activity — messages, receipts, group changes, presence — is surfaced as an event with a typed payload.
Plugins can contribute their own typed events; they appear on client.on only when the plugin is in the plugins array. See the VoIP guide for an example — its voip_* events are available when voipPlugin() is installed.
​
Listening
import type { WaIncomingMessageEvent } from 'zapo-js'

client.on('message', (event: WaIncomingMessageEvent) => {
  console.log(event.key.remoteJid, event.message)
})

client.once('auth_paired', ({ credentials }) => {
  console.log('paired', credentials.meJid)
})

const handler = (e) => { /* ... */ }
client.on('receipt', handler)
client.off('receipt', handler) // stop listening
on, once, and off are all type-checked against the event map — the payload type is inferred from the event name, so listeners get full autocomplete.
​
Auth & connection
Event	Payload	Description
auth_qr	{ qr, ttlMs }	A QR code to render for pairing.
auth_pairing_code	{ code }	An 8-character pairing code was issued.
auth_pairing_required	{ forceManual }	The session needs pairing input.
auth_passkey_required	{ hasSigner }	The server is forcing a Shortcake passkey to link. hasSigner: false means the handshake stalls until you configure a signer.
auth_paired	{ credentials }	Pairing succeeded.
connection	WaConnectionEvent	Socket opened or closed (see below).
The connection event is a discriminated union on status:
client.on('connection', (event) => {
  if (event.status === 'open') {
    console.log('online; new login?', event.isNewLogin)
  } else {
    console.log('closed:', event.reason, 'logout?', event.isLogout)
  }
})
See Reconnection for the handling pattern.
​
Messages
Event	Payload	Description
message	WaIncomingMessageEvent	An inbound <message> stanza was decrypted.
message_send	WaOutgoingMessageEvent	An outbound message this client is sending, with its decrypted Proto.IMessage — the symmetric counterpart to message. Lets plugins / loggers observe forwards, reactions, polls, media the wire stanza hides.
message_addon	WaIncomingAddonEvent	Reactions, poll votes, comments (decrypted addons).
message_protocol	WaIncomingProtocolMessageEvent	Protocol messages (edits, revokes, …).
message_bot_chunk	WaIncomingBotChunkEvent	Streamed bot response chunks.
message_unavailable	WaIncomingUnavailableMessageEvent	A content-less placeholder arrived (see Unavailable messages).
receipt	WaIncomingReceiptEvent	Delivery / read / played receipts.
See Receiving messages for payload details and text extraction.
​
Unavailable messages
Some incoming <message> stanzas carry an <unavailable/> marker instead of an encrypted body — a view-once whose contents have already been consumed, a hosted/bot message the server could not fan out, or a plain fanout placeholder the primary device can still resend. There is nothing to decrypt on the stanza itself, but the arrival is useful information (audit logs, a “this message is no longer available” UI row, or waiting for the resend to land). The lib acks them and emits a typed message_unavailable event with a kind discriminator:
client.on('message_unavailable', (event) => {
  // event.kind: 'view_once' | 'hosted' | 'bot' | 'other'
  console.log('unavailable', event.kind, 'resendRequested?', event.resendRequested,
    'from', event.key.remoteJid, 'id', event.key.id)
})
Field	Type	Notes
kind	'view_once' | 'hosted' | 'bot' | 'other'	Which flavor the server signalled. 'other' is a plain fanout placeholder — the only recoverable kind.
resendRequested	boolean	true when the lib queued a PLACEHOLDER_MESSAGE_RESEND peer request; the payload then arrives later as a message event with the same key.id. false for the unrecoverable kinds, messages past the AB-props age window, and mobile-primary sessions.
key	WaIncomingMessageKey	Same shape the message event carries — store or correlate it like any other message id.
timestampSeconds	number?	From the stanza’s t attr.
pushName	string?	Sender’s display name from the stanza’s notify attr.
Resend requests are best-effort — like wa-web, they are not persisted, so a failed peer request is not retried. See Receiving messages → Recovering unavailable messages for the details.
​
Presence & chat-state
Event	Payload	Description
presence	WaIncomingPresenceEvent	A contact’s presence changed (available / last-seen).
chatstate	WaIncomingChatstateEvent	Typing / recording / paused.
call	WaIncomingCallEvent	Incoming call signaling.
​
Groups, newsletters & profiles
Event	Payload	Description
group	WaGroupEvent	Group create/subject/participant/setting changes.
newsletter	WaIncomingNewsletterEvent	Newsletter activity.
newsletter_message_update	WaIncomingNewsletterMessageUpdateEvent	Edits/reactions/poll updates on newsletter messages.
business	WaBusinessEvent	Business profile changes.
picture	WaPictureEvent	Profile/group picture changes.
privacy	WaPrivacyAccountSyncResult	The account’s privacy was changed on the primary or another companion — the full category set plus any disallowed list the server reported. Debounced 1s, so a burst of changes on the phone collapses into one refresh. Values always come from a fresh read, never from the notification payload.
blocklist	WaBlocklistResult	The account blocklist after a block/unblock made on another device. Same refetch path as privacy; the payload is the full list, never a delta.
own_username	WaOwnUsernameNotificationEvent	The account’s username was set / deleted / modified on the primary or another companion. kind discriminates the change; username is populated only for kind: 'set'.
​
State, history & MEX
Event	Payload	Description
mutation	WaAppStateMutationEvent	App-state mutation (mute, pin, archive, …) synced from another device. Inbound only; this client’s own outbound actions surface on mutation_send.
mutation_send	WaAppStateMutationEvent	An app-state action this client is sending — the outbound counterpart to mutation, symmetric to how message_send pairs with message. event.source is always 'local'. Optimistic: emitted as the mutation is enqueued, before the server flush confirms it.
history_sync_chunk	WaHistorySyncChunkEvent	A chunk of synced message history (initial bootstrap or message.requestHistorySync backfill). Skipped only when history.enabled is explicitly false. Ingestion is streamed — the proto reader descends into conversations one message at a time, so ingest peak memory stays flat regardless of chunk size. messagesCount counts written records, not read ones; a chunk whose bounded park of messages waiting on Conversation.id overflows fails and stays unacked (the primary resends), rather than completing with silent drops.
group_history_bundle	WaGroupHistoryBundleEvent	A group-history bundle another member shared with this account after it joined a group, already downloaded, filtered and persisted. Requires history.groupBundles: true — the fetch is opt-in because a third party triggers it. See Groups → receiving.
offline_resume	WaOfflineResumeEvent	Progress of the post-connect offline-message drain.
offline_thread_metadata	WaOfflineThreadMetadataEvent	Preview manifest of the offline queue (per-thread latest-stanza timestamps, optional per-thread read watermarks, optional pending status / notification backlog counts). Sent just before the flush; not guaranteed to arrive. Use offline_resume for progress, never this.
mex_notification	WaMexNotificationEvent	MEX (GraphQL) notifications: username, status, LID changes, capping.
​
MEX notification kinds
WaMexNotificationEvent is a discriminated union on kind. Every variant carries operationName (the upstream GraphQL operation) and errors: readonly WaMexNotificationGraphQlError[] (any GraphQL errors the server attached — typically empty).
kind	operationName	Extra fields
username_set	UsernameSetNotification	lidJid, username — a contact set or changed their username.
username_delete	UsernameDeleteNotification	lidJid, displayName: string | null — username cleared (null means the server omitted the fallback name).
username_update_hint	UsernameUpdateNotification	contactHash — side-channel hint that something changed for a contact bucket; refetch through the regular profile path.
own_username_sync	AccountSyncUsernameNotification	ownLidJid, username | null, state | null, pin | null — your own username state synced from another device. null username means it was removed.
text_status_update	TextStatusUpdateNotification	jid, text | null, emoji | null, ephemeralDurationSec | null, lastUpdateTime | null — a contact’s about/status changed. null text/emoji clears that field.
text_status_update_hint	TextStatusUpdateNotificationSideSub	contactHash — side-channel hint; refetch the status.
lid_change	LidChangeNotification	oldLidJid, newLidJid — a user’s LID rotated. See LID changes.
message_capping	MessageCappingInfoNotification	cappingStatus ('NONE' | 'FIRST_WARNING' | 'SECOND_WARNING' | 'CAPPED' | string), plus optional oteStatus, mvStatus, totalQuota, usedQuota, cycleStartTimestamp, cycleEndTimestamp, serverSentTimestamp — your account’s outgoing-message quota state.
unknown	(whatever the server sent)	data: unknown — catch-all for an operationName without a typed variant; the raw GraphQL data payload is passed through verbatim so you can decode it yourself.
client.on('mex_notification', (event) => {
  switch (event.kind) {
    case 'username_set':
      console.log(`${event.lidJid} now goes by @${event.username}`)
      break
    case 'text_status_update':
      console.log(`${event.jid} status:`, event.text, event.emoji)
      break
    case 'message_capping':
      console.log(`capping ${event.cappingStatus}:`, event.usedQuota, '/', event.totalQuota)
      break
    case 'lid_change':
      console.log('LID rotated:', event.oldLidJid, '→', event.newLidJid)
      break
  }
})
*_hint kinds (username_update_hint, text_status_update_hint) carry only a contactHash, not the new value — the server is telling you “something changed for this bucket” and the client is expected to refetch through the regular profile / status fetch path.
​
Companion host (mobile-primary)
Emitted by the client.mobile coordinator when a mobile-primary session links, revokes, or fails to provision a companion device. All three only fire on a mobile-primary session.
Event	Payload	Description
companion_host_linked	{ deviceJid: string, keyIndex: number }	A companion successfully linked via linkCompanion / linkCompanionByCode.
companion_host_revoked	{ deviceJid: string }	A hosted companion was removed — via revokeCompanion / revokeAllCompanions, or dropped during a reconcileCompanions() sweep after the user unlinked it from another surface.
companion_host_error	Error	A link or background provisioning step failed.
​
Failures
Event	Payload	Description
stream_failure	WaIncomingFailureEvent	A stream-level failure (may precede a disconnect).
stanza_error	WaIncomingErrorStanzaEvent	An error stanza from the server.
​
Debug events
A family of debug_* events expose low-level internals — raw frames, decoded nodes, decode errors, unhandled stanzas, and client errors. They are useful for protocol debugging but noisy; subscribe selectively.
client.on('debug_transport_node_in', ({ node }) => console.dir(node, { depth: null }))
client.on('debug_client_error', ({ error }) => console.error(error))
​
debug_decrypted_payload
The plaintext of every decrypted <enc> in the stanza, emitted between the unpad and the proto.Message.decode step. Fires whether or not decoding then succeeds — which is what makes the failing case observable at all. Without this hook, a payload that decrypts but does not decode (a proto field the library does not yet know about, a malformed body) is otherwise lost: decode throws, the stanza is reported through debug_unhandled_stanza, and the bytes go with it — while the decryption has already advanced the ratchet, so the same ciphertext will never decrypt again.
Field	Type	Notes
encIndex	number	Which <enc> of the stanza produced these bytes, counting from zero. A message addressed to several devices carries several <enc> nodes whose ciphertexts are unrelated — attributing a payload to the wrong index attributes it to the wrong sender.
encType	string	The <enc> node’s type attribute: msg, pkmsg, skmsg, …
plaintext	Uint8Array	The unpadded plaintext. A copy, so mutating it cannot alter the message the library then delivers.
client.on('debug_decrypted_payload', ({ encIndex, encType, plaintext }) => {
  fs.appendFileSync('payloads.bin', plaintext)
  console.log('captured', encType, 'idx', encIndex, plaintext.length, 'bytes')
})
Costs nothing when nobody is subscribed — the plaintext copy is only built when at least one listener is attached. Useful for recording traffic for faithful replay (re-encoding a decoded message does not reproduce the original bytes), or decoding with a newer protobuf than the library carries. A listener that throws is swallowed and logged, so a buggy observer cannot poison delivery.
Mobile-registration events (mobile_registration_code, mobile_account_takeover_notice) exist for the mobile-registration path and are not part of the standard companion flow.

Stores

Persist authentication state, Signal sessions, and per-domain protocol data through zapo’s pluggable store interface and bundled backend packages.

A store is where zapo persists everything a session needs to survive a restart: pairing credentials, Signal protocol state, app-state collections, and optionally your message/thread/contact archive. You build one with createStore and pass it to the client.
import { createStore } from 'zapo-js'
import { createSqliteStore } from '@zapo-js/store-sqlite'

const store = createStore({
  backends: {
    sqlite: createSqliteStore({ path: '.auth/state.sqlite', driver: 'auto' })
  },
  providers: {
    auth: 'sqlite',
    signal: 'sqlite',
    preKey: 'sqlite',
    session: 'sqlite',
    identity: 'sqlite',
    senderKey: 'sqlite',
    appState: 'sqlite',
    privacyToken: 'sqlite',
    messages: 'sqlite',
    threads: 'sqlite',
    contacts: 'sqlite'
  }
})
​
The model
createStore separates backends (where data lives) from providers (which backend each domain uses). This lets you mix backends — e.g. keep hot signal state in Redis while archiving messages in Postgres.
createStore({
  backends: {
    redis: createRedisStore({ redis }),
    postgres: createPostgresStore({ pool })
  },
  providers: {
    auth: 'redis',
    signal: 'redis',
    preKey: 'redis',
    session: 'redis',
    identity: 'redis',
    senderKey: 'redis',
    appState: 'redis',
    privacyToken: 'redis',
    messages: 'postgres',
    threads: 'postgres',
    contacts: 'postgres'
  }
})
​
Providers are required when you set backends
As soon as backends contains at least one entry, every persistence domain must be assigned explicitly in providers. The required domains are auth, signal, preKey, session, identity, senderKey, appState, privacyToken, messages, threads, and contacts. Both the TypeScript types and a runtime check enforce this — createStore throws and lists the missing providers.* keys when any are omitted.
Three values are valid for each domain:
A backend name from backends (e.g. 'sqlite') — persist that domain there.
'memory' — keep that domain in the in-tree memory provider for this run.
'none' — only valid for the optional archive domains (messages, threads, contacts); skips the domain entirely.
This guard exists because partial coverage is almost always a bug. If you persist only auth and let Signal state, app-state, or the mailbox fall back to memory, the device pairs once and then loses its protocol state on every restart. Pick 'memory' deliberately when that is what you want.
createStore({
  backends: { sqlite: createSqliteStore({ path: '.auth/state.sqlite' }) },
  providers: {
    auth: 'sqlite',
    signal: 'sqlite',
    preKey: 'sqlite',
    session: 'sqlite',
    identity: 'sqlite',
    senderKey: 'sqlite',
    appState: 'sqlite',
    privacyToken: 'sqlite',
    messages: 'none',  // skip the message archive
    threads: 'none',
    contacts: 'none'
  }
})
When backends is empty or omitted, every domain falls back to memory (mailbox domains to 'none') — useful for tests, but the device re-pairs on every restart.
​
Persisted domains
These hold the state required to keep a session alive. Back them with a durable backend in production.
Domain	Holds
auth	Pairing credentials and device identity. Persist this.
signal	Signal sessions (umbrella over the sub-stores below).
preKey	Signal pre-keys.
session	Signal sessions.
identity	Signal identity keys.
senderKey	Group sender keys.
appState	App-state collections (mute, pin, read, archive, …).
privacyToken	Trusted-contact / privacy tokens.
​
Optional archive domains
These accept 'none' to disable persistence entirely:
Domain	Holds
messages	Message archive (B | 'memory' | 'none').
threads	Thread metadata.
contacts	Contact directory.
​
Cache domains
Configured under cacheProviders and default to bounded memory with TTLs:
Domain	Holds	Default
retry	Outbound message retry queue.	'memory'
groupMetadata	Group metadata cache.	'memory'
chatMetadata	Protocol-derived per-chat state (disappearing-message settings). Read on every 1:1 send to stamp contextInfo; rebuilt from history sync, EPHEMERAL_SETTING messages and incoming ContextInfo, so losing it costs freshness rather than data.	'memory'
deviceList	Device list cache.	'memory'
messageSecret	Message-secret cache for addons.	'memory'
createStore({
  backends: { sqlite },
  providers: { /* ... */ },
  cacheProviders: { groupMetadata: 'sqlite', deviceList: 'sqlite' },
  memory: {
    cacheTtlMs: { groupMetadataMs: 600_000, deviceListMs: 600_000 }
  }
})
Each backend evicts expired entries differently: memory runs an in-process sweep, Redis and MongoDB use native TTL, SQLite filters on read, and PostgreSQL/MySQL require an opt-in poller (result.startCleanup(sessionId)) or cache tables grow forever. See Cache expiry and cleanup for the per-backend matrix.
​
Read-through cache layer
When a hot signal domain points at a persistent backend, every send/recv round-trip pays the backend’s latency to fetch the same peer’s session, identity, or sender key. The cacheLayer option wraps the backend store with a bounded-LRU L1 (the in-tree memory provider) so repeated reads of the same peer skip the backend, while writes stay write-through so the backend remains authoritative.
Four hot domains can be cached:
Domain	Strategy
session	Signal Double-Ratchet sessions. Read-through + write-through.
identity	Remote identity keys. Read-through + write-through.
senderKey	Per-(group, sender) sender keys. Read-through + write-through.
privacyToken	Trusted-contact tokens. Read-through + invalidate-on-write (the backend merges partial fields on upsert).
All flags default to false. A flag is a no-op unless that domain resolves to a real backend in providers — caching 'memory' or 'none' in front of itself buys nothing and is skipped.
createStore({
  backends: { postgres, redis },
  providers: {
    auth: 'postgres',
    signal: 'postgres',
    preKey: 'postgres',
    session: 'postgres',
    identity: 'postgres',
    senderKey: 'postgres',
    appState: 'postgres',
    privacyToken: 'postgres',
    messages: 'postgres',
    threads: 'postgres',
    contacts: 'postgres'
  },
  cacheLayer: {
    session: true,
    identity: true,
    senderKey: true,
    privacyToken: true,
    limits: {
      session: 10_000,
      identity: 10_000,
      senderKey: 5_000,
      privacyToken: 5_000
    }
  }
})
limits caps per-domain entry counts; once exceeded, the L1 evicts LRU. When unset, each domain defaults to the matching memory-provider cap.
​
When to enable it
Turn it on when your backend is a network hop (Redis, Postgres, MySQL, MongoDB) and you send or receive at a rate where the same peers repeat — typical for bots, group fan-out, and multi-tenant gateways. With a local SQLite backend the wins are smaller; measure before flipping it on.
​
Single-writer assumption
The L1 is per-process and has no cross-process invalidation channel. Enable cacheLayer only when a single process owns a given sessionId’s backend rows — the library’s standard connection model. Different sessions sharing one backend are fine; the same session opened from two processes is not.
Do not enable cacheLayer when multiple processes share one backend for the same sessionId. Another process’s writes would leave this cache stale and corrupt the Signal ratchet.
​
Why not every domain?
signal, appState, and preKey are deliberately excluded:
signal — the per-send registration read is already memoized inside the signal lock; a second cache adds nothing.
appState — the sync client already caches collection state for the sync-context lifetime, the only scope where reads both repeat and stay coherent.
preKey — one-time pre-keys are read exactly once then consumed. Serving a consumed key from a stale cache would reuse it and break forward secrecy.
​
Backends
SQLite
@zapo-js/store-sqlite — local, single-process.
PostgreSQL
@zapo-js/store-postgres — distributed, relational.
MySQL
@zapo-js/store-mysql — distributed, relational.
Redis
@zapo-js/store-redis — cache + persistence.
MongoDB
@zapo-js/store-mongo — document store.
Memory
Built in. Great for tests; does not survive a restart.
See the stores reference for each backend’s config options.
​
Memory-only (tests)
For quick experiments or tests, omit backends entirely — every domain falls back to memory:
const store = createStore({})
const client = new WaClient({ store, sessionId: 'test' }, logger)
A memory-only store loses all credentials on restart, so you re-pair every boot. Use a durable backend for anything long-lived.
​
Custom backends
The backend contract is WaStoreBackend<S, C>, parametrized by the persistence domains (S extends WaStoreDomain) and cache domains (C extends WaCacheDomain) the bundle actually implements. Both default to the full matrix, so a bare WaStoreBackend still means every domain — that’s what every in-tree @zapo-js/store-* package ships.
To cover part of the matrix, name the domains you implement:
import type { WaStoreBackend } from 'zapo-js'

const vault = {
  stores: { auth: (sessionId: string) => new MyAuthStore(sessionId) },
  caches: {}
} satisfies WaStoreBackend<'auth', never>

createStore({
  backends: { vault, sqlite },
  providers: {
    auth: 'vault',       // ok — vault declares 'auth'
    signal: 'sqlite',
    // signal: 'vault',  // compile error — vault does not declare 'signal'
    // ...
  }
})
createStore() now infers the backend map (not just the backend names), so routing an undeclared domain — or misspelling a backend name — fails at compile time instead of throwing does not provide <kind>.<domain> on the first session(). The mandatory coverage of every persistence domain once backends is set is unchanged, and 'none' still resolves to the noop store.
A hand-written full backend has to declare the new chatMetadata cache alongside the other cache domains, or narrow itself with WaStoreBackend<S, C>. A bare satisfies WaStoreBackend without the cache factory no longer compiles.
A hand-written WaContactStore needs to read and write the new WaStoredContactRecord.username field — the contact handle without the display-only @, populated when the server surfaces one. The in-tree @zapo-js/store-* packages migrate their schemas automatically on connect (via ensurePgMigrations and its per-backend siblings), so consumers of those packages do not need to do anything.

Sending messages


Send WhatsApp text, threaded replies, mentions, and rich link previews with client.message.send, the typed entry point for all outgoing content.

All outgoing content goes through a single method:
client.message.send(to, content, options?): Promise<WaMessagePublishResult>
to — the recipient JID (5511999999999@s.whatsapp.net, a group ...@g.us, etc.). See JID helpers for building these.
content — a string, a typed content object, or a raw Proto.IMessage.
options — quoting, mentions, forwarding, view-once, edits, and more.
The promise resolves to a WaMessagePublishResult once the server acks:
const result = await client.message.send(jid, 'Hello!')
console.log(result.id) // the message id (stanza id)
​
Plain text
The simplest content is a string:
await client.message.send(jid, 'Hello from zapo!')
For more control, use the text object form — it lets you attach context info and tune link previews:
await client.message.send(jid, {
  type: 'text',
  text: 'Check this out: https://example.com',
  linkPreview: true // auto-fetch a preview
})
​
Replying (quoting)
Pass the original message event (or a reference) as options.quote:
client.on('message', async (event) => {
  await client.message.send(event.key.remoteJid, 'Replying to you', {
    quote: event
  })
})
The quote is rendered as a reply bubble referencing the original message.
​
Mentions
options.mentions is a list of JIDs to tag. Include the matching @number text in the body so WhatsApp renders the mention:
await client.message.send(groupJid, {
  type: 'text',
  text: 'Hey @5511999999999, welcome!'
}, {
  mentions: ['5511999999999@s.whatsapp.net']
})
​
Link previews
Link-preview behavior is controlled per message via the text object’s linkPreview field:
Value	Behavior
undefined	Follow the global linkPreview default.
false	Disable the preview.
true	Force auto-fetch of the preview.
object	Skip the fetch and use the provided preview fields directly.
// Provide your own preview instead of fetching
await client.message.send(jid, {
  type: 'text',
  text: 'https://example.com',
  linkPreview: { title: 'Example', description: 'My custom preview' }
})
Configure the default fetcher globally with the linkPreview client option.
​
Forwarding
Set options.forward to mark a message as forwarded:
await client.message.send(jid, 'Forwarded text', { forward: true })
// or with a frequently-forwarded score
await client.message.send(jid, content, { forward: { score: 4 } })
​
Send options reference
WaSendMessageOptions (third argument) includes:
Option	Type	Purpose
quote	WaIncomingMessageEvent | WaQuoteRef | WaMessageKey	Reply to a message — pass the event verbatim, its key, or a WaQuoteRef.
mentions	string[]	JIDs to mention.
forward	boolean | { score }	Mark as forwarded.
viewOnce	boolean	Wrap image/video/audio as view-once.
editKey	WaMessageKey | WaSendEditKey | WaMessageRef	Edit a previously sent message (see interactive).
expirationSeconds	number	Disappearing-message TTL for this message. Wins over contextInfo.expirationSeconds and over the automatic group-ephemeral inject (the latter is short-circuited as soon as this is defined — even 0).
disableGroupEphemeralAutoInject	boolean	Skip the automatic ephemeral-setting injection on group sends (expiration and disappearingMode from the group metadata cache). No effect on 1:1. Redundant when expirationSeconds is set.
disableDirectEphemeralAutoInject	boolean	Skip the automatic ephemeral-setting injection on 1:1 sends (expiration, ephemeralSettingTimestamp and disappearingMode from the chatMetadata cache) — also skips the cache lookup. No effect on groups. Redundant when expirationSeconds is set.
contextInfo	WaSendContextInfo	Raw context info (advanced).
id	string	Use a specific message id.
ackTimeoutMs / maxAttempts / retryDelayMs	number	Per-send retry tuning.
To send a single message with no expiration into a group with disappearing-mode on, prefer disableGroupEphemeralAutoInject: true over expirationSeconds: 0 — the latter still writes expiration=0 into the outgoing contextInfo. Same story for disableDirectEphemeralAutoInject in 1:1 chats.
​
Sending to a username
A username handle (@alice) is not an addressable JID on its own. Resolve it to the account’s LID with client.profile.resolveUsername first, then send to that JID — the send API takes the JID exactly like any other 1:1 recipient.
resolveUsername returns a discriminated union you have to switch on. Every case has a next step:
const result = await client.profile.resolveUsername({ username: '@alice' })

switch (result.status) {
  case 'found':
    // result.jid is the LID (@lid) to address; result.pnJid is the phone-jid
    // form when the server knows it, and result.isBusiness flags a business
    // account. result.username is the canonical handle the server echoes back.
    await client.message.send(result.jid, 'hi')
    break

  case 'key-required': {
    // The server withheld the JID until the 4-digit recovery key is supplied.
    // Ask the user for it (out of band — WhatsApp UI shows it as "@handle:1234")
    // and retry with the key. A one-off '@user:1234' input works too:
    // resolveUsername parses the ':1234' suffix into usernameKey for you.
    const usernameKey = await askUserForFourDigitKey() // your UI
    const retry = await client.profile.resolveUsername({ username: '@alice', usernameKey })
    if (retry.status === 'found') await client.message.send(retry.jid, 'hi')
    else if (retry.status === 'key-required') console.log('wrong key')
    else console.log('no such handle')
    break
  }

  case 'not-found':
    // Terminal — the handle is not registered on WhatsApp. No retry recovers it.
    console.log('no such handle')
    break
}
resolveUsername throws before any round-trip when the handle or the key fails local validation (bad characters, wrong length, reserved word, key that is not exactly 4 digits) — so a call that returns has passed local rules and the status reflects the server-side outcome.
​
The content union
content accepts any WaSendMessageContent. The typed variants are documented across these guides:
Media
Images, video, audio, documents, stickers.
Polls & reactions
Polls, votes, reactions, pins, edits, revokes, events.
You can always drop down to a raw Proto.IMessage for anything not covered by a typed builder:
import { proto } from 'zapo-js'

await client.message.send(jid, {
  conversation: 'Raw protobuf message'
})
Content types without a typed builder yet — locations, contact vCards, and Business PIX / review-and-pay payment cards — are documented in Raw proto sends.
The full set of recognized Proto.IMessage fields (location, live location, contacts, group invite, product, order, …) is listed in the message types reference.


Raw proto sends

Send content types WhatsApp supports but zapo doesn’t have a typed builder for yet — locations, contact vCards, group invites, buttons, list menus, interactive native flow, products, orders, newsletter admin invites, ephemeral toggles, phone-number requests, and Business PIX / review-and-pay payment cards — as raw Proto.IMessage payloads.

For content types WhatsApp supports but zapo doesn’t wrap in a typed builder yet, client.message.send also accepts a raw Proto.IMessage. Fill the field that names the content type — locationMessage, contactMessage, contactsArrayMessage, interactiveMessage, and so on — and the library encodes it verbatim.
Kinds that already have a typed builder — polls, reactions, edits, revokes, pins, keep-in-chat, view-once wrapping, and quotes / mentions / link previews — belong in Sending messages and Interactive messages. This page is for the raw-only kinds.
The full set of recognized Proto.IMessage fields (location, live location, contacts, group invite, product, order, …) is listed in the message types reference. Some examples below use enum values from the proto namespace:
import { proto } from 'zapo-js'
​
Locations
await client.message.send(jid, {
  locationMessage: {
    degreesLatitude: -23.5613,
    degreesLongitude: -46.6565,
    name: 'Av. Paulista',
    address: 'São Paulo, BR'
  }
})
name and address are optional.
For a live-location message, use the liveLocationMessage field instead — it carries movement metadata (accuracyInMeters, speedInMps, sequenceNumber).
await client.message.send(jid, {
  liveLocationMessage: {
    degreesLatitude: -23.5505,
    degreesLongitude: -46.6333,
    accuracyInMeters: 50,
    speedInMps: 0,
    caption: 'On my way',
    sequenceNumber: 1
  }
})
​
Contacts
A single contact card is a contactMessage with a vCard string:
const vcard = [
  'BEGIN:VCARD',
  'VERSION:3.0',
  'FN:Jeff Singh',
  'TEL;type=CELL;type=VOICE;waid=5511999999999:+55 11 99999-9999',
  'END:VCARD'
].join('\n')

await client.message.send(jid, {
  contactMessage: { displayName: 'Jeff', vcard }
})
The waid=<digits> parameter on the TEL line is what lets the WhatsApp client link the card back to a WhatsApp account — use the recipient’s E.164 phone number without the +.
For multiple cards at once, use contactsArrayMessage:
await client.message.send(jid, {
  contactsArrayMessage: {
    displayName: '2 contacts',
    contacts: [
      { displayName: 'Jeff', vcard },
      { displayName: 'Jane', vcard: janeVcard }
    ]
  }
})
​
Group invite
await client.message.send(jid, {
  groupInviteMessage: {
    groupJid: '123456789-987654@g.us',
    inviteCode: 'AbCdEf123',
    inviteExpiration: Math.floor(Date.now() / 1000) + 86_400,
    groupName: 'My group',
    caption: 'Join us!'
  }
})
​
Buttons
Up to three quick-reply buttons. The header is a oneof — pick text, image, video, location, or document (pre-uploaded for media):
await client.message.send(jid, {
  buttonsMessage: {
    contentText: 'Order placed — what next?',
    footerText: 'Reply within 24h',
    headerType: proto.Message.ButtonsMessage.HeaderType.TEXT,
    text: 'Order #1234',
    buttons: [
      {
        buttonId: 'track',
        buttonText: { displayText: 'Track' },
        type: proto.Message.ButtonsMessage.Button.Type.RESPONSE
      },
      {
        buttonId: 'cancel',
        buttonText: { displayText: 'Cancel' },
        type: proto.Message.ButtonsMessage.Button.Type.RESPONSE
      }
    ]
  }
})
​
List menu
A single-select list of rows grouped into sections:
await client.message.send(jid, {
  listMessage: {
    title: 'Menu',
    description: 'Choose an item',
    buttonText: 'View menu',
    footerText: 'Open 9–18',
    listType: proto.Message.ListMessage.ListType.SINGLE_SELECT,
    sections: [
      {
        title: 'Pizzas',
        rows: [
          { rowId: 'pizza-margherita', title: 'Margherita', description: 'Tomato, mozzarella, basil' },
          { rowId: 'pizza-pepperoni',  title: 'Pepperoni',  description: 'Tomato, cheese, pepperoni' }
        ]
      },
      {
        title: 'Drinks',
        rows: [{ rowId: 'drink-cola', title: 'Cola' }]
      }
    ]
  }
})
​
Interactive native flow (cta_url)
The modern interactive surface — buttons whose params are JSON-encoded ad-hoc payloads:
await client.message.send(jid, {
  interactiveMessage: {
    body: { text: 'Tap below to open the form' },
    footer: { text: 'Powered by your bot' },
    nativeFlowMessage: {
      buttons: [
        {
          name: 'cta_url',
          buttonParamsJson: JSON.stringify({
            display_text: 'Open form',
            url: 'https://example.com/form'
          })
        }
      ],
      messageVersion: 1
    }
  }
})
This is the same wire shape as the PIX / review-and-pay cards in Payments below — only the button name and buttonParamsJson differ.
​
Product
Send a catalog product. The inner productImage must already be uploaded:
await client.message.send(jid, {
  productMessage: {
    businessOwnerJid: '5511999999999@s.whatsapp.net',
    body: 'Take a look at this',
    footer: 'In stock',
    product: {
      productId: '12345',
      title: 'Hat',
      description: 'One size, adjustable',
      currencyCode: 'BRL',
      priceAmount1000: 49_900, // 49.90 BRL — price × 1000
      retailerId: 'sku-001',
      url: 'https://example.com/p/12345',
      productImage: { /* pre-uploaded image fields */ }
    }
  }
})
​
Order
Order confirmation / inquiry:
await client.message.send(jid, {
  orderMessage: {
    orderId: 'ord-abc',
    orderTitle: 'Sample order',
    itemCount: 3,
    status: proto.Message.OrderMessage.OrderStatus.INQUIRY,   // or ACCEPTED / DECLINED
    surface: proto.Message.OrderMessage.OrderSurface.CATALOG,
    sellerJid: '5511888888888@s.whatsapp.net',
    totalAmount1000: 149_700, // 149.70 BRL — total × 1000
    totalCurrencyCode: 'BRL',
    message: 'Order details'
  }
})
​
Newsletter admin invite
Invite a contact to co-admin one of your newsletters:
await client.message.send(contactJid, {
  newsletterAdminInviteMessage: {
    newsletterJid: '120363xxxxxxxxxxxxxx@newsletter',
    newsletterName: 'My Newsletter',
    caption: 'Become a co-admin',
    inviteExpiration: Math.floor(Date.now() / 1000) + 7 * 86_400
  }
})
​
Toggle disappearing messages (ephemeral setting)
Chat-wide toggle for disappearing messages — distinct from the per-message expirationSeconds send option (one message) and the ephemeralMessage wrapper (one message inheriting the chat timer). This one flips the timer for the whole chat.
await client.message.send(jid, {
  protocolMessage: {
    type: proto.Message.ProtocolMessage.Type.EPHEMERAL_SETTING,
    ephemeralExpiration: 7 * 24 * 3600 // seconds; 0 disables
  }
})
​
Request a phone number
await client.message.send(jid, { requestPhoneNumberMessage: {} })
​
Payments (PIX & review-and-pay)
Send WhatsApp Business payment cards — static PIX (payment_info) and order checkout (review_and_pay) — as raw interactiveMessage / nativeFlowMessage payloads through client.message.send. There is no typed builder for them yet, so build the shape yourself the same way as the locations/contacts cards above. The library relays the interactive native-flow buttons; the WhatsApp clients render the card UI.
Payment cards are a Business / native-flow feature. Rendering differs between WhatsApp mobile and WhatsApp Web — prefer the flow that matches the UI you want (payment_info for a PIX-only card, review_and_pay for the “Nº da cobrança” / order card) and verify on both clients.
Amounts are integer minor units plus an offset divisor: { value: 1000, offset: 100 } renders as R$ 10,00. PIX key_type is one of EVP (chave aleatória), EMAIL, PHONE (E.164 preferred), CPF, CNPJ.
​
PIX card (payment_info)
Renders the PIX payment card (key / merchant). Use this for a static PIX key without an order card.
await client.message.send(jid, {
  interactiveMessage: {
    nativeFlowMessage: {
      messageVersion: 1,
      buttons: [
        {
          name: 'payment_info',
          buttonParamsJson: JSON.stringify({
            currency: 'BRL',
            total_amount: { value: 0, offset: 100 },
            reference_id: `PIX${Date.now()}`,
            type: 'physical-goods',
            order: {
              status: 'pending',
              subtotal: { value: 0, offset: 100 },
              order_type: 'ORDER',
              items: [
                { name: '', amount: { value: 0, offset: 100 }, quantity: 0, sale_amount: { value: 0, offset: 100 } }
              ]
            },
            payment_settings: [
              {
                type: 'pix_static_code',
                pix_static_code: {
                  merchant_name: 'Loja Exemplo',
                  key: 'pix@loja.com',
                  key_type: 'EMAIL'
                }
              }
            ],
            share_payment_status: false,
            is_soft_deleted: false,
            referral: 'chat_attachment'
            // display_text: 'Pagar com PIX' // optional
          })
        }
      ]
    }
  }
})
​
Review-and-pay card (review_and_pay)
Renders the order / cobrança card (reference number, items, total). Use this for a checkout-style summary.
await client.message.send(jid, {
  interactiveMessage: {
    body: { text: 'Olá! Sua fatura está disponível.' },
    footer: { text: 'Se já pagou, desconsidere.' },
    nativeFlowMessage: {
      messageVersion: 1,
      buttons: [
        {
          name: 'review_and_pay',
          buttonParamsJson: JSON.stringify({
            currency: 'BRL',
            reference_id: 'PGT-PIX-001',
            type: 'physical-goods',
            total_amount: { value: 10000, offset: 100 }, // R$ 100,00
            payment_settings: [
              {
                type: 'pix_static_code',
                pix_static_code: {
                  merchant_name: 'Loja Exemplo',
                  key: 'pix@loja.com',
                  key_type: 'EMAIL'
                }
              }
            ],
            order: {
              status: 'payment_requested',
              subtotal: { value: 10000, offset: 100 },
              order_type: 'ORDER',
              items: [
                { name: 'Fatura', amount: { value: 10000, offset: 100 }, quantity: 1 }
              ]
              // discount: { value: 500, offset: 100 }
            }
            // additional_note: 'Pagamento até o vencimento'
          })
        }
      ]
    }
  }
})
buttonParamsJson must be a JSON string — build the object in code and stringify it. body / footer on interactiveMessage are optional. Don’t mix payment_info and review_and_pay expecting the same UI — they render different cards, and adding extra cta_copy / CTA buttons in the same nativeFlowMessage can render differently on mobile vs Web.
​
See also
Sending messages — the base client.message.send API, options, and typed content variants.
Interactive messages — typed builders for polls, reactions, edits, revokes, pins, and keep-in-chat.
Message types reference — every recognized Proto.IMessage field and its resolved type.

Media

Send images, video, audio voice notes, documents, and stickers — and stream or download incoming WhatsApp media attachments with zapo’s media helpers.

Media is sent through the same client.message.send method, using a typed media content object. The builder fills in the protocol-managed fields (encryption keys, SHA-256 digests, direct path, upload) for you — you provide the source and, optionally, a mimetype.
For usable media, install @zapo-js/media-utils and wire a processor through the media client option. Media still uploads without it, but without a processor it has no thumbnail/preview, dimensions, or waveform — so it may arrive as a plain attachment.
​
Mimetype resolution
mimetype is optional. The builder resolves it in this order:
The mimetype you pass on the content object wins.
If a WaMediaProcessor with detectMimetype is configured, the builder calls it (sniffing magic bytes). @zapo-js/media-utils implements this on top of file-type ^19 — install file-type to enable detection.
Otherwise the builder throws for image/video/audio/document/ptv messages.
Stickers default to image/webp when no mimetype is set. Readable stream inputs with no mimetype are staged to a temp file before detection runs.
​
Media input
The media field accepts several input types:
type MediaInput = Uint8Array | ArrayBuffer | Readable | string
Prefer a file path (string) or a Readable stream over a Buffer/Uint8Array. zapo streams media through the pipeline without buffering the whole file in memory — passing a path or stream keeps memory flat regardless of file size. Reading a large file into a Buffer first defeats that and is discouraged. (Buffer is also avoided internally in favor of Uint8Array.)
​
Images
// Preferred — pass a file path; zapo streams it
await client.message.send(jid, {
  type: 'image',
  media: './photo.jpg',
  mimetype: 'image/jpeg',
  caption: 'A photo'
})

// Or a Readable stream (e.g. from an HTTP response)
import { createReadStream } from 'node:fs'

await client.message.send(jid, {
  type: 'image',
  media: createReadStream('./photo.jpg'),
  mimetype: 'image/jpeg'
})
​
Video
await client.message.send(jid, {
  type: 'video',
  media: './clip.mp4',
  mimetype: 'video/mp4',
  caption: 'A clip',
  gifPlayback: false
})
For a round push-to-video (PTV) message, use type: 'ptv' with the same shape.
​
Audio & voice notes
// Regular audio
await client.message.send(jid, {
  type: 'audio',
  media: './song.mp3',
  mimetype: 'audio/mpeg'
})

// Voice note (push-to-talk)
await client.message.send(jid, {
  type: 'audio',
  media: './voice.ogg',
  mimetype: 'audio/ogg; codecs=opus',
  ptt: true
})
Voice notes render best as Opus in an OGG container. Enable media processing to auto-generate waveforms and normalize voice notes.
​
Documents
await client.message.send(jid, {
  type: 'document',
  media: './report.pdf',
  mimetype: 'application/pdf',
  fileName: 'Q3 Report.pdf',
  caption: 'The quarterly report'
})
​
Stickers
await client.message.send(jid, {
  type: 'sticker',
  media: await readFile('./sticker.webp'),
  mimetype: 'image/webp'
})
For a full sticker pack, use type: 'sticker-pack' with stickers, a trayIcon, and pack metadata (stickerPackId, name, publisher).
​
View-once
Wrap image/video/audio as view-once with the send option:
await client.message.send(jid, {
  type: 'image',
  media: './secret.jpg',
  mimetype: 'image/jpeg'
}, {
  viewOnce: true
})
​
Pre-upload and reuse
Sometimes you want to encrypt and upload media once and then reference the same descriptor across multiple sends — a broadcast asset, a stock reply, or building a raw proto payload yourself. client.message.upload(source, options) runs the encrypt / media_conn / CDN upload / parse flow the send path uses, but returns the descriptor without sending anything.
import { readFile } from 'node:fs/promises'

const media = await client.message.upload(await readFile('./photo.jpg'), {
  type: 'image',
  mimetype: 'image/jpeg'
})

// Send it as an image message by spreading the descriptor onto the proto:
await client.message.send(jid, {
  imageMessage: {
    url: media.url,
    directPath: media.directPath,
    mediaKey: media.mediaKey,
    fileSha256: media.fileSha256,
    fileEncSha256: media.fileEncSha256,
    fileLength: media.fileLength,
    mediaKeyTimestamp: media.mediaKeyTimestamp,
    mimetype: media.mimetype
  }
})
The upload runs against your connected session (the host token comes from a media_conn IQ). Uint8Array sources take a zero-temp-file fast path; a file path or Readable stream is staged to a temp file so the encrypt pass can hash it deterministically.
​
WaUploadMediaOptions
Field	Type	Notes
type	'image' | 'video' | 'audio' | 'document' | 'sticker' | 'ptv' | 'gif' | 'ptt'	Required. Sets the encryption context and CDN upload path. Unknown types throw before encryption runs.
mimetype	string	Content-Type for the upload, echoed back on the result for the message proto.
mediaKey	Uint8Array	Reuse an existing 32-byte media key instead of generating a fresh one.
sidecar	boolean	Override the streaming sidecar. Default true for video / ptv / audio / gif / ptt, off for the rest.
firstFrameLength	number	Animated-sticker first-frame length; needed to compute the first-frame sidecar.
timeoutMs	number	Per-upload transfer timeout override.
signal	AbortSignal	Cancellation signal forwarded to the CDN request.
​
WaMediaUploadResult
Field	Type	Notes
url	string	Absolute CDN URL of the uploaded ciphertext.
directPath	string	Host-relative path — pairs with a media host or feeds downloadMediaMessage.
mediaKey	Uint8Array	The 32-byte media key used to encrypt. Reuse it in a follow-up call to upload if you want a stable descriptor.
fileSha256	Uint8Array	SHA-256 of the plaintext.
fileEncSha256	Uint8Array	SHA-256 of the encrypted ciphertext‖mac.
fileLength	number	Plaintext byte length.
mediaKeyTimestamp	number	Unix seconds the media key was minted; belongs on the message proto.
mimetype	string?	Echo of the option, when provided.
metadataUrl	string?	Server metadata URL (video).
streamingSidecar	Uint8Array?	Streaming sidecar for seekable playback, when computed.
firstFrameSidecar / firstFrameLength	Uint8Array? / number?	Animated-sticker first-frame sidecar echoes.
mediaKey is sensitive key material — treat it like a password. Don’t log it, ship it to third parties, or persist it in unencrypted stores. Anyone with the key can decrypt the CDN blob.
​
Standalone crypto & transfer
For workflows that need to run encryption/decryption outside a session — a background worker crunching stored ciphertext, a custom relay — WaMediaCrypto and WaMediaTransferClient (plus their result and option types) are re-exported from the package root:
import { WaMediaCrypto, WaMediaTransferClient } from 'zapo-js'
Same primitives the upload / download paths use; the coordinator method just wraps them with the session-bound media_conn handshake.
​
Downloading incoming media
The message coordinator decrypts and downloads media from an incoming event. Three flavors are available — prefer the streaming ones:
client.on('message', async (event) => {
  if (!event.message?.imageMessage) return

  // Preferred — stream to a file (constant memory)
  await client.message.downloadToFile(event, './incoming.jpg')

  // Or consume the Readable stream yourself
  const stream = await client.message.download(event)

  // Avoid for large media — buffers the entire file in memory
  const bytes = await client.message.downloadBytes(event)
})
download() / downloadToFile() stream the media and keep memory flat regardless of size. downloadBytes() materializes the whole file in memory — reach for it only on small media, and cap it with maxBytes.
All three accept either a WaIncomingMessageEvent or a raw Proto.IMessage, plus optional WaDownloadMediaOptions (for example maxBytes to cap downloadBytes).
​
Without a connected client
downloadMediaMessage is a free function that mirrors client.message.download but does not need a paired session. The encrypted-media metadata travels inside the (already decrypted) message itself, so you can re-download media from a persisted event long after the original socket is gone — useful for offline workers, archive replays, or anything that processes stored messages without spinning up a WaClient.
import { downloadMediaMessage } from 'zapo-js'
import { createWriteStream } from 'node:fs'

const stream = await downloadMediaMessage(event)
stream.pipe(createWriteStream('photo.jpg'))
Accepts a WaIncomingMessageEvent or a raw Proto.IMessage, returns a Readable you own (pipe it or .destroy() it — an unconsumed stream leaks the socket). MAC + SHA-256 verification runs as bytes are consumed, same semantics as the coordinator method. Throws when the message has no downloadable media.
Option	Type	Notes
transfer	WaMediaTransferClient	Reuse an existing transfer client — inherits its proxy agents, timeouts, and MAC-verification toggle, and avoids spinning up a new HTTP client per call in a loop. A stateless one is created when omitted (fine for one-offs).
proxy	WaProxyTransport	Per-call proxy for the CDN download leg, mirroring the client’s proxy.mediaDownload. The fetch runs over node:http/node:https, so only the http.Agent form is honored — an undici dispatcher is ignored. Takes precedence over the default agent of a transfer you pass.
signal / timeoutMs / maxBytes	(from WaDownloadMediaOptions)	Same semantics as the coordinator methods.
For lower-level access — when you want to do the CDN fetch yourself, hand the keys to another process, or just inspect what’s downloadable — resolveMediaPayload returns the keys + hashes without doing any I/O:
import { resolveMediaPayload } from 'zapo-js'

const payload = resolveMediaPayload(event.message)
// → WaResolvedMediaPayload | null
// { mediaType, directPath, mediaKey, fileSha256?, fileEncSha256?, mimetype?, fileLength? }
Returns null when the message has no downloadable media, or when the proto carried no directPath / mediaKey. It unwraps ephemeralMessage, viewOnceMessage / viewOnceMessageV2, and documentWithCaptionMessage before resolving. Supported kinds: image, video (gif when gifPlayback), audio (ptt when ptt), document, sticker, ptv.
payload.mediaKey is the AES/MAC seed for the encrypted blob — treat it like a secret. Don’t log it, don’t put it in error messages, and don’t ship it to a third-party service unless that’s the whole point of your pipeline.
​
Requesting a reupload
The CDN drops old media blobs — a download* on a message surfaced by history sync can answer 404/410 well after the original send. client.message.requestMediaReupload() runs the round-trip WhatsApp Web uses to recover: encrypt a server-error receipt with the message’s media key, send it as an ack, and await the mediaretry notification the sender’s primary device answers with. On success only the directPath changes — the media key, hashes, and length of the original message stay valid, so downloading with a spread of the new path works.
client.on('message', async (event) => {
  const image = event.message?.imageMessage
  if (!image) return

  try {
    await client.message.downloadToFile(event, './incoming.jpg')
  } catch {
    // Old blob expired — ask the sender to re-serve it
    const retry = await client.message.requestMediaReupload(event)
    if (retry.result !== 'success') {
      console.warn('reupload', retry.result) // 'not_found' | 'decryption_error' | 'general_error'
      return
    }
    await client.message.downloadToFile(
      { imageMessage: { ...image, directPath: retry.directPath } },
      './incoming.jpg'
    )
  }
})
Pass an explicit WaMediaRetryRequest ({ messageId, chatJid, mediaKey, fromMe, participant? }) when you hold the ids and the media key but not the decoded message — the event form pulls those out for you and rejects newsletter messages plus messages with no downloadable media. options.timeoutMs bounds the wait for the notification; the coordinator does not throw on not_found / general_error (the sender no longer holds the file, or the primary could not re-seal it) — check result.result for one of 'success' | 'not_found' | 'decryption_error' | 'general_error'.
Requests are deduplicated by messageId: concurrent calls for the same message share one round-trip and one server-error receipt.
​
Media processing
For proper media, use a media processor. Install @zapo-js/media-utils and pass one through the media client option — it probes and processes media (dimensions, duration, thumbnails, waveforms, voice-note normalization) before upload. Without it, media still uploads but lacks this processing:
import { createMediaProcessor } from '@zapo-js/media-utils'

const client = new WaClient({
  store,
  sessionId: 'default',
  media: {
    processor: createMediaProcessor(),
    generateThumbnail: true,
    generateWaveform: true,
    normalizeVoiceNote: true
  }
}, logger)
@zapo-js/media-utils shells out to ffmpeg/ffprobe and uses sharp. Make sure those binaries are available in your environment.


Polls, reactions & edits

Send polls and votes, react to messages, pin and edit content, revoke sent messages, and handle the events for each — through the typed content union.

Beyond text and media, client.message.send accepts a family of typed interactive content objects. Each is discriminated by its type field.
​
Targeting a message
Reply / reaction / revoke / pin / keep / event-response all accept a WaMessageTargetInput: either a received message event passed verbatim (its key is used) or an explicit WaMessageKey:
interface WaMessageKey {
  remoteJid: string     // the chat the target lives in
  id: string            // the target's message (stanza) id
  fromMe: boolean       // was the target sent by you?
  participant?: string  // the author — required in groups when targeting someone else's message
}
The easiest path is to pass the event you already have — event.key is already a WaMessageKey:
client.on('message', async (event) => {
  // Use the event itself as the target — its key is read for you.
  await client.message.send(event.key.remoteJid, {
    type: 'reaction',
    emoji: '👍',
    target: event
  })
})
​
Reactions
// Pass the event verbatim, or an explicit WaMessageKey
await client.message.send(jid, {
  type: 'reaction',
  emoji: '👍',
  target: event
})
Pass an empty string as emoji to remove a previous reaction:
await client.message.send(jid, { type: 'reaction', emoji: '', target: event })
​
Polls
const result = await client.message.send(jid, {
  type: 'poll',
  name: 'Lunch?',
  options: ['Pizza', 'Sushi', 'Salad'],
  selectableCount: 1,        // how many options a voter may pick
  allowAddOption: false
})
Options may be plain strings or { name } objects. Order matters — it is used for vote hashing.
​
Voting on a poll
Voting requires the original poll’s identity and its messageSecret (32 bytes from the poll’s messageContextInfo.messageSecret):
await client.message.send(jid, {
  type: 'poll-vote',
  poll: {
    id: pollStanzaId,                  // the poll's stanza id
    fromMe: false,
    authorJid: pollAuthorJid,
    messageSecret: pollMessageSecret,  // Uint8Array, 32 bytes
    participant: pollAuthorJid         // required outside 1:1 chats
  },
  selectedOptionNames: ['Pizza']       // exactly as they appeared in the poll
})
Incoming votes arrive as message_addon events once decrypted.
​
Editing a message
To edit, send the new content and pass editKey in the options. The original must be fromMe. You can pass the received message event verbatim, its key, or an explicit WaSendEditKey ({ id, participant?, timestampMs? }):
// Easiest: forward the original event
await client.message.send(jid, 'Corrected text', { editKey: originalEvent })

// Or build one explicitly
await client.message.send(jid, 'Corrected text', {
  editKey: {
    id: originalStanzaId,
    // participant required in groups for lid/pn-addressed originals
    participant: undefined
  }
})
The new payload is wrapped in a MESSAGE_EDIT protocol message targeting editKey.id.
​
Revoking (delete for everyone)
// Easiest: pass the event you want to revoke
await client.message.send(jid, { type: 'revoke', target: event })

// Or build the target explicitly
await client.message.send(jid, {
  type: 'revoke',
  target: {
    remoteJid: jid,
    id: targetStanzaId,
    fromMe: true,
    // participant required when an admin revokes someone else's message in a group
    participant: undefined
  }
})
Sender-vs-admin revoke is auto-detected from target.fromMe: false triggers an admin revoke. There is no subtype option to pass.
​
Pinning
await client.message.send(jid, { type: 'pin', target: event })   // pin
await client.message.send(jid, { type: 'unpin', target: event }) // unpin
Pins expire — pass durationSecs to override the default. wa-web offers three presets:
await client.message.send(jid, { type: 'pin', target: event, durationSecs: 604_800 })   // 7 days
await client.message.send(jid, { type: 'pin', target: event, durationSecs: 2_592_000 }) // 30 days
Default is 86_400 (24h). The TTL travels with the pin via messageContextInfo.messageAddOnDurationInSecs; without it receiving clients silently drop the pin, so the lib always stamps the default when you omit it. durationSecs is ignored on unpin.
​
Keep-in-chat
For disappearing-message chats, keep (or un-keep) a specific message:
await client.message.send(jid, { type: 'keep', target: event })
await client.message.send(jid, { type: 'unkeep', target: event })
​
Events
Create a calendar-style event message:
await client.message.send(groupJid, {
  type: 'event',
  name: 'Team sync',
  description: 'Weekly catch-up',
  startTime: Math.floor(Date.now() / 1000) + 3600, // unix seconds
  location: { latitude: -23.5, longitude: -46.6, name: 'HQ' },
  joinLink: 'https://meet.example.com/abc',
  hasReminder: true,
  reminderOffsetSec: 600
})
Responding to an event
await client.message.send(jid, {
  type: 'event-response',
  event: {
    id: eventStanzaId,                 // the event's stanza id
    fromMe: false,
    authorJid: eventAuthorJid,
    messageSecret: eventMessageSecret  // 32 bytes
  },
  response: 'going' // 'going' | 'not_going' | 'maybe'
})

Receiving messages

Handle incoming message events: extract text and media, send delivery and read receipts, decrypt addons, and request older history.

Incoming messages arrive on the message event as a WaIncomingMessageEvent.
import type { WaIncomingMessageEvent } from 'zapo-js'

client.on('message', (event: WaIncomingMessageEvent) => {
  // ...
})
​
The event payload
WaIncomingMessageEvent carries a rich key (a superset of Proto.IMessageKey) plus a few top-level fields. Pass the event (or just its key) verbatim to reply / edit / react / revoke / pin / keep.
Field	Type	Description
key.remoteJid	string	Deviceless conversation JID (group or 1:1) — the :device segment is stripped; the device id is exposed via key.senderDevice.
key.id	string	The message (stanza) id.
key.fromMe	boolean	True when the message was sent by this account.
key.participant	string?	The author in groups / broadcasts (omitted in 1:1).
key.isGroup / key.isBroadcast / key.isNewsletter	boolean	Chat-kind flags derived from remoteJid.
key.remoteJidAlt	string?	The remoteJid’s alternate addressing (PN if addressed by LID, or vice-versa) in 1:1 chats.
key.participantAlt	string?	The participant’s alternate addressing in group chats.
key.senderDevice	number	Sender’s device id; 0 when the source JID has no :device segment.
key.senderUsername	string?	Author’s handle, without the @. Absent on self-authored 1:1 stanzas — the peer’s handle surfaces as key.recipientUsername instead.
key.recipientUsername	string?	Peer’s handle, without the @. Populated on self-authored 1:1 stanzas (where senderUsername is absent) and on any message that carries recipient_username / peer_recipient_username.
key.recipientJid / key.recipientAlt	string?	Your receiving JID and its alternate form.
key.serverId	number?	Server-assigned message id for newsletter / channel messages.
message	Proto.IMessage	The decrypted message content.
timestampSeconds	number?	Server timestamp (unix seconds).
expirationSeconds	number?	Disappearing-message TTL the sender attached to this message, when present.
pushName	string?	The sender’s display name.
You also receive your own outgoing messages here (multi-device sync), flagged with key.fromMe === true. Filter them out if you only want inbound traffic.
The whole event (or event.key) is accepted as the target for replies/reactions/revokes/pins/keeps and as editKey for edits — no need to reshape it.
​
Extracting text
A message’s text lives in different fields depending on its type. A small helper covers the common cases:
function extractText(message?: Proto.IMessage | null): string | undefined {
  if (!message) return undefined
  return (
    message.conversation ??
    message.extendedTextMessage?.text ??
    message.imageMessage?.caption ??
    message.videoMessage?.caption ??
    undefined
  )
}

client.on('message', (event) => {
  const text = extractText(event.message)
  if (text) console.log(`${event.pushName}: ${text}`)
})
​
Identifying the message type
message is a protobuf union — inspect which field is set:
client.on('message', (event) => {
  const m = event.message
  if (!m) return

  if (m.conversation || m.extendedTextMessage) console.log('text')
  else if (m.imageMessage) console.log('image')
  else if (m.videoMessage) console.log('video')
  else if (m.audioMessage) console.log('audio')
  else if (m.documentMessage) console.log('document')
  else if (m.stickerMessage) console.log('sticker')
  else if (m.pollCreationMessage) console.log('poll')
  else if (m.locationMessage) console.log('location')
})
To download media from an image/video/audio/document message, see Media → downloading.
​
Sending receipts
client.message.sendReceipt marks messages as received/read/played. The easiest form takes the event(s) directly:
client.on('message', async (event) => {
  // mark as read
  await client.message.sendReceipt(event, { type: 'read' })
})
You can also pass an array of events, or address it manually by JID and ids:
await client.message.sendReceipt(chatJid, [id1, id2], { type: 'read' })
​
Calls
Read-only
Incoming call signaling surfaces as the read-only call event (WaIncomingCallEvent). zapo reports calls — it does not place, accept, or reject them.
client.on('call', (event) => {
  console.log(
    event.type,          // 'offer' | 'accept' | 'terminate' | … | 'unknown'
    event.isVideo ? 'video' : 'voice',
    'from', event.callerPnJid ?? event.callCreatorJid,
    event.groupJid ? `(group ${event.groupJid})` : ''
  )
})
Useful fields: type (the signaling stage), callId, callCreatorJid / callerPnJid (who’s calling), isVideo, groupJid (group calls), and callerPushName. There is no API to answer a call.
​
Addons
Addons are encrypted follow-ups attached to a message: reactions, poll votes, and comments. They surface as the message_addon event.
​
Automatic decryption
Addons are decrypted and emitted for you by default — just subscribe to message_addon:
const client = new WaClient({ store, sessionId: 'default' }, logger)

client.on('message_addon', (event) => {
  console.log('addon:', event)
})
​
Manual decryption
Pass addons: { autoDecrypt: false } to receive the encrypted payload and decrypt on demand from the originating message event:
const client = new WaClient({
  store,
  sessionId: 'default',
  addons: { autoDecrypt: false }
}, logger)

client.on('message', async (event) => {
  await client.message.tryDecryptAddon(event)
})
​
Recovering unavailable messages
Some <message> stanzas arrive as an <unavailable/> placeholder rather than an encrypted body — a consumed view-once, a hosted or bot message the server could not fan out, or a plain fanout placeholder the primary still holds the plaintext for. All four flavors surface as the message_unavailable event with a kind discriminator.
Plain fanout placeholders (kind === 'other') are recoverable: the lib queues a PLACEHOLDER_MESSAGE_RESEND peer request to the primary device, and the recovered payload arrives later as a regular message event with the same key.id. Consumers know to wait for it via resendRequested:
client.on('message_unavailable', (event) => {
  if (event.resendRequested) {
    console.log('waiting for primary to resend', event.key.id)
    // A `message` event with the same key.id will follow (best-effort)
  }
})
The recovery is best-effort — the queued request is not persisted, so a failed peer message is not retried. resendRequested is false for the unrecoverable kinds ('view_once', 'hosted', 'bot'), for stanzas past the server age window (driven by the placeholder_message_resend_maximum_days_limit AB prop, not a hardcoded 30 days), and on mobile-primary sessions (there is no other device to ask). The lib does not time the pending resend out — a real primary can answer past the 30s peer-request default; wa-web behaves the same way.
​
Protocol messages
Edits, revokes, and other protocol-level updates arrive on message_protocol as WaIncomingProtocolMessageEvent (it extends the message event with a protocolMessage field):
client.on('message_protocol', (event) => {
  console.log(event.protocolMessage)
})
​
Requesting older history
The initial pairing flow streams a bounded window of message history. To pull older messages for a specific chat on demand, call client.message.requestHistorySync:
const { messageId } = await client.message.requestHistorySync({
  chatJid,
  oldestMsgId: topMessage.key.id,
  oldestMsgFromMe: topMessage.key.fromMe,
  oldestMsgTimestampMs: topMessage.timestampSeconds * 1000,
  count: 50
})
The method returns once the request is dispatched — not when the chunk arrives. The backfill is delivered later as a history_sync_chunk event, same as the bootstrap history. Subscribe before calling if you need to react to it:
client.on('history_sync_chunk', (event) => {
  // event.conversations, event.pushnamesCount, event.progress, ...
})

await client.message.requestHistorySync({ chatJid })
Pair oldestMsgId, oldestMsgFromMe, and oldestMsgTimestampMs from the topmost message currently visible to page backwards correctly. Omit count to let the server apply its own default (~50).
​
Receipts (inbound)
When others read or play your messages, you receive receipt events:
client.on('receipt', (event) => {
  // event.status: 'delivered' | 'read' | 'played' | 'inactive'
  for (const id of event.messageIds) {
    console.log(event.status, 'for', id)
  }
})
event.messageIds is the full set of stanza ids this receipt acknowledges. WhatsApp batches read/delivery receipts into a single <receipt> carrying a <list><item id=…/> block — messageIds mirrors wa-web’s externalIds: list items first, then event.stanzaId appended last. Single-message receipts contain just [event.stanzaId].
WaIncomingReceiptEvent also carries participantUsername?: string — the participant’s handle without the @, read from the stanza’s participant_username attribute when present.
receipt events still expose stanzaId / chatJid directly (they extend WaIncomingBaseEvent); the rename only applies to message, message_addon, and message_bot_chunk payloads, which now use event.key. event.stanzaId is still the last id in the batch — iterate messageIds to cover the rest.

Managing chats

Mute, pin, archive, mark as read, lock, star, clear, and delete WhatsApp chats with the typed client.chat coordinator backed by app-state mutations.

Per-chat settings live on client.chat (WaAppStateMutationCoordinator). These are app-state mutations — they sync across all your linked devices, and changes made elsewhere arrive back as the mutation event.
These operations affect your account’s view (and your other devices). They do not change anything for the other participants — e.g. deleting a chat doesn’t delete it for them. For delete-for-everyone, use a revoke.
​
Mute
// Mute for 8 hours
await client.chat.setChatMute(chatJid, true, Date.now() + 8 * 3600_000)

// Unmute
await client.chat.setChatMute(chatJid, false)
muteEndTimestampMs is required when muting (epoch ms). For “mute forever”, pass a far-future timestamp. The client does not auto-unmute when the timer expires — that’s when WhatsApp re-enables notifications.
​
Pin & archive
await client.chat.setChatPin(chatJid, true)      // pin
await client.chat.setChatArchive(chatJid, true)  // archive
Pin and archive are mutually exclusive — pinning a chat clears its archive flag and vice-versa. WhatsApp caps the number of pinned chats server-side.
​
Read / unread
await client.chat.setChatRead(chatJid, true)   // mark read
await client.chat.setChatRead(chatJid, false)  // mark unread
​
Lock
await client.chat.setChatLock(chatJid, true)
Locking also clears archive and pin.
​
Star a message
A message is identified by a WaAppStateMessageKey:
interface WaAppStateMessageKey {
  chatJid: string
  id: string            // the message (stanza) id
  fromMe: boolean
  participantJid?: string // group sender
}

await client.chat.setMessageStar(
  { chatJid, id: stanzaId, fromMe: false, participantJid: senderJid },
  true
)
​
Clear & delete
// Clear messages but keep the chat (local-only)
await client.chat.clearChat(chatJid, { deleteStarred: false, deleteMedia: true })

// Delete the chat entirely (removes it from the list + stored messages)
await client.chat.deleteChat(chatJid, { deleteMedia: true })
clearChat keeps starred messages and media by default; set deleteStarred / deleteMedia to wipe those too. Neither leaves a group — use client.group.leaveGroup for that.
​
Delete a message for me
Removes a single message from your own device(s) only — recipients still see it:
await client.chat.deleteMessageForMe(
  { chatJid, id: stanzaId, fromMe: false },
  { deleteMedia: true }
)
To delete for everyone instead, send a revoke.
​
Beyond the helpers
The methods above are typed shortcuts. For anything without a dedicated helper — contacts, labels, quick replies, status privacy, and the full list of app-state schemas — use the generic client.chat.set() / client.chat.remove(). See the chat mutations reference.
​
Reacting to changes
When a chat setting changes on another device, you receive a mutation event:
client.on('mutation', (event) => {
  console.log(event.collection, event.schema, event.operation)
})


Groups & communities

Create groups, manage participants and admins, handle invites, configure community sub-groups, and react to group events with zapo.

Group operations live on client.group (WaGroupCoordinator). Group JIDs end in @g.us.
​
Querying groups
// All groups the account belongs to
const groups = await client.group.queryAllGroups()

// One group's metadata
const meta = await client.group.queryGroupMetadata('123456@g.us')
console.log(meta.subject, meta.participants.length)
WaGroupMetadata includes the subject, owner, participant list (WaGroupParticipant[] with isAdmin / isSuperAdmin), and the full set of group flags (announce, restrict, ephemeral, community flags, …).
​
Creating a group
createGroup returns the full WaGroupMetadata for the new group — no need to call queryGroupMetadata afterward:
const group = await client.group.createGroup('My group', [
  '5511999999999@s.whatsapp.net',
  '5511888888888@s.whatsapp.net'
])

console.log(group.jid, group.participants.length)
​
Managing participants
The four participant methods (addParticipants, removeParticipants, promoteParticipants, demoteParticipants) return a typed WaParticipantActionResult[] — one entry per jid you passed in. The IQ as a whole succeeds even when some participants fail (blocked you, privacy settings disallow add, already a member, …), so inspect the per-jid code to surface partial failures.
const jids = ['5511999999999@s.whatsapp.net']

const results = await client.group.addParticipants(groupJid, jids)

for (const r of results) {
  if (r.status === 'ok') {
    console.log('added', r.jid)
  } else {
    // HTTP-style code: 403 = privacy block, 408 = not allowed,
    // 409 = already in, 404 = not on WhatsApp, ...
    console.warn('failed', r.jid, r.code)
  }
}

await client.group.removeParticipants(groupJid, jids)
await client.group.promoteParticipants(groupJid, jids) // make admin
await client.group.demoteParticipants(groupJid, jids)  // remove admin
Each result also carries phoneNumber and username when the server resolved them, plus the raw BinaryNode under raw for any extra tags the server attached (some 409/408 partial failures hint at how to recover).
​
Group settings
await client.group.setSubject(groupJid, 'New name')
await client.group.setDescription(groupJid, 'A description')   // null to clear
await client.group.setSetting(groupJid, 'announcement', true)  // admins-only messages
await client.group.setSetting(groupJid, 'restrict', true)      // admins-only edit info
await client.group.setSetting(groupJid, 'ephemeral', true)     // disappearing messages on/off
setSetting also covers the boolean toggles ephemeral, group_history, allow_admin_reports, no_frequently_forwarded, and the community flags. Use it to flip a feature on or off; for settings that need a value (mode or duration), use the dedicated setters below.
​
Who can add, link, and share history
// Who can add new members
await client.group.setMemberAddMode(groupJid, 'admin_add')        // admins only
await client.group.setMemberAddMode(groupJid, 'all_member_add')   // anyone

// Who can share the invite link
await client.group.setMemberLinkMode(groupJid, 'admin_link')
await client.group.setMemberLinkMode(groupJid, 'all_member_link')

// Whether new members see prior chat history
await client.group.setMemberShareGroupHistoryMode(groupJid, 'admin_share')      // hide history
await client.group.setMemberShareGroupHistoryMode(groupJid, 'all_member_share') // expose backlog
All three are admin-only — non-admins receive a 403 not-authorized error.
​
Disappearing messages
setSetting(groupJid, 'ephemeral', false) is the explicit disable path. To turn disappearing messages on with a specific lifetime, use setEphemeralDuration:
// 24h / 7d / 90d in seconds
await client.group.setEphemeralDuration(groupJid, 86_400)
await client.group.setEphemeralDuration(groupJid, 604_800)
await client.group.setEphemeralDuration(groupJid, 7_776_000)

// Disable
await client.group.setSetting(groupJid, 'ephemeral', false)
Admin-only. Passing 0 disables disappearing messages — the same as setSetting('ephemeral', false).
​
Invites
// Preview an invite code (the path segment of chat.whatsapp.com/<code>)
const info = await client.group.queryGroupInviteInfo('AbCdEf...')
console.log(info.subject, info.size, info.desc)
// info.participants is a trimmed sample — not the full roster.
// Call queryGroupMetadata after joining for everyone.

// Fetch the current invite code for a group you admin — does NOT rotate it.
const code = await client.group.queryInviteCode(groupJid)
console.log('current invite:', `https://chat.whatsapp.com/${code}`)

// Join via invite code — returns the joined group's metadata
const joined = await client.group.joinGroupViaInvite('AbCdEf...')
console.log(joined.jid, joined.participants.length)

// Rotate the invite — returns the freshly-issued code
const { code: rotated, affectedParticipants } = await client.group.revokeInvite(groupJid)
console.log('new invite:', `https://chat.whatsapp.com/${rotated}`)
// affectedParticipants lists anyone who joined via the now-revoked code
// that the server surfaced in the response (typically with code: 404).
queryInviteCode and revokeInvite are admin-only — non-admins receive a 403 not-authorized.
​
Leaving
await client.group.leaveGroup([groupJid]) // batched — accepts multiple
leaveGroup resolves to void once the server acknowledges the request.
​
Membership approval
For groups that require admin approval to join:
const requests = await client.group.queryMembershipApprovalRequests(groupJid)

await client.group.approveMembershipRequests(groupJid, [requesterJid])
await client.group.rejectMembershipRequests(groupJid, [requesterJid])

// Cancel your own pending request
await client.group.cancelMembershipRequests(groupJid, [myJid])
​
Communities
Communities are parent groups that link sub-groups:
// Create a community
const community = await client.group.createCommunity('My community')

// Link / unlink existing groups as sub-groups
await client.group.linkSubGroups(community.jid, [subGroupJidA, subGroupJidB])
await client.group.unlinkSubGroups(community.jid, [subGroupJidA], {
  removeOrphanedMembers: true
})

// List sub-groups (and the announcement group)
const subs = await client.group.fetchSubGroups(community.jid)

// Join a linked sub-group you don't yet belong to.
// The IQ result carries no group payload — call queryGroupMetadata
// on the sub-group after this resolves to get the full metadata.
await client.group.joinLinkedGroup(community.jid, subGroupJid)
const subMeta = await client.group.queryGroupMetadata(subGroupJid)

// Merged participants across the whole community
const everyone = await client.group.queryLinkedGroupsParticipants(community.jid)
Other community operations include deactivateCommunity, transferCommunityOwnership, and fetchSubgroupSuggestions.
​
Sharing group history
When a new member joins a group whose memberShareGroupHistoryMode exposes the backlog, any existing member can push the recent messages to them directly. Both sides live on client.message.
​
Sending
const result = await client.message.shareGroupHistory('12036@g.us', {
  toJids: ['5511999999999@s.whatsapp.net'],
  count: 50 // default: group_history_message_count_limit (100)
})

console.log(result.bundleMessageId, result.messagesCount)
shareGroupHistory(groupJid, input) resolves toJids against the live participant list, uploads the zlib-compressed GroupHistory payload, and fans the bundle out only to those receivers plus this account — members who are not receiving it never see the stanza. A group-wide notice message follows so other clients can render the “history was shared” marker.
Input fields:
toJids — required. Written in the group’s own addressing mode (a LID-addressed group only matches @lid entries; a PN one only matches @s.whatsapp.net). Read the mode off client.group.queryGroupMetadata(), whose participants carry both forms. Anything else throws.
count?: number — how many of the most recent messages to read from the messages mailbox store. Ignored when messages is supplied.
sinceMs?: number — only read messages at or after this ms timestamp from the store. Ignored when messages is supplied.
messages?: readonly Proto.IWebMessageInfo[] — supply the messages directly, bypassing the mailbox store. Required when the messages store domain is 'none' (the default) — there is nothing to read back.
outOfWindowPinnedMessages?: readonly Proto.IWebMessageInfo[] — pinned messages older than the shared window; the receiver injects these regardless of the age cutoff.
WaShareGroupHistoryResult returns bundleMessageId, noticeMessageId? (absent when the notice failed to send after the bundle was already delivered — do not retry the share), messagesCount, historyReceivers, and nonHistoryReceivers.
The sender side is gated per account by the group_history_send AB prop. When it is off, the server rejects the stanza with SMAX_INVALID after the upload is spent, so the client checks the prop up front and throws before uploading. Admin-only groups (memberShareGroupHistoryMode: 'admin_share') reject a share from a regular member server-side — check the mode with queryGroupMetadata first.
​
Receiving
Downloading a bundle is opt-in — a bundle is media a third party pushes at this account unprompted. Enable it in the client config:
new WaClient({
  store,
  sessionId: 'default',
  history: { groupBundles: true }
}, logger)

client.on('group_history_bundle', (event) => {
  console.log('history bundle for', event.groupJid,
    '+', event.messagesCount, 'msgs',
    '(dropped', event.droppedCount, ')')
})
The receiver verifies this account is in historyReceivers before spending a CDN fetch, drops stubs / foreign-chat / age-expired / ephemeral-expired entries, persists the rest, and emits group_history_bundle. Out-of-window pins ride along exempt from the age cutoff. Window limits come from the server-synced AB props, not hardcoded defaults.
Bundles derive their media keys from a Group History HKDF context — distinct from history sync’s WhatsApp History Keys. Bundles addressed to other members are dropped either way.
​
Group events
Changes made by others (subject, participants, settings) arrive on the group event:
client.on('group', (event) => {
  console.log(event.action, 'in', event.groupJid)
})
See Events for the full payload.

Newsletters (channels)

Create, discover, follow, post to, react on, and administer WhatsApp channels (newsletters) using the client.newsletter coordinator in zapo.

Newsletters — WhatsApp channels — live on client.newsletter (WaNewsletterCoordinator). The coordinator combines three operation sets: discovery, admin, and messaging. Newsletter JIDs end in @newsletter.
​
Discovery
// Fetch metadata by JID or invite code
const meta = await client.newsletter.fetch('1234567890@newsletter')
const byInvite = await client.newsletter.fetchByInvite('AbCdEf')

// Channels the account follows
const subscribed = await client.newsletter.listSubscribed()

// Search the public directory
const results = await client.newsletter.searchDirectory({ /* text, categories */ })

// Recommendations & similar channels
const recommended = await client.newsletter.fetchRecommended()
const similar = await client.newsletter.fetchSimilar(newsletterJid)
​
Following
await client.newsletter.follow(newsletterJid)
await client.newsletter.unfollow(newsletterJid)
await client.newsletter.mute({ newsletterJid, mute: true })
​
Posting
send takes the same content union as a normal message — text, media, polls, and so on:
const result = await client.newsletter.send(newsletterJid, 'Hello, subscribers!')

// Edit a published message
await client.newsletter.editMessage(newsletterJid, result.id, 'Edited')

// Revoke it
await client.newsletter.revoke({ newsletterJid, originalMessageId: result.id })
​
Reactions & poll votes
await client.newsletter.react({ newsletterJid, parentMessageServerId, reactionCode: '🔥' })
await client.newsletter.votePoll({ /* WaNewsletterVotePollInput */ })
await client.newsletter.sendViewReceipt({ /* WaNewsletterViewReceiptInput */ })
​
Reading messages
const page = await client.newsletter.fetchMessages({
  newsletterJid,
  count: 50
})

// Edits / reactions / poll updates since a point
const updates = await client.newsletter.fetchMessageUpdates({
  newsletterJid,
  count: 50,
  since: someTimestamp
})

// Live updates subscription
const { durationSeconds } = await client.newsletter.subscribeLiveUpdates(newsletterJid)
​
Administration
For channels the account owns:
// Create a channel
const created = await client.newsletter.create({ name: 'My channel', description: '...' })

// Update editable fields (name / description / picture)
await client.newsletter.update(newsletterJid, { name: 'Renamed' })

// Delete
await client.newsletter.delete(newsletterJid)

// Admin views
const adminInfo = await client.newsletter.fetchAdminInfo(newsletterJid)
const followers = await client.newsletter.fetchFollowers(newsletterJid)
const insights = await client.newsletter.fetchInsights(newsletterJid, metrics)
​
Admins & ownership
await client.newsletter.createAdminInvite({ /* WaNewsletterAdminInviteInput */ })
await client.newsletter.changeOwner({ /* ... */ })
await client.newsletter.demoteAdmin({ /* ... */ })
​
Poll voters & reaction senders
const voters = await client.newsletter.fetchPollVoters({
  newsletterJid,
  messageServerId,
  voteHash
})

const reactors = await client.newsletter.fetchMessageReactionSenders({
  newsletterJid,
  messageServerId
})
​
Events
Newsletter activity arrives on newsletter, and edits/reactions/poll updates on newsletter_message_update:
client.on('newsletter', (event) => console.log(event))
client.on('newsletter_message_update', (event) => console.log(event))

Broadcast lists

Define a WhatsApp broadcast list with zapo and send a single message to many recipients at once without creating a group or revealing the list.

A broadcast list sends a single message to many contacts at once — each recipient receives it as a normal 1:1 chat and can’t see who else is on the list. Broadcast lists live on client.broadcastList (WaBroadcastListCoordinator).
Business-only
Business-only. Broadcast lists are backed by the BusinessBroadcastList app-state schema and only work on WhatsApp Business accounts. On a regular account the server rejects the underlying mutations.
​
Defining a list
setList creates or updates a list definition — it then appears under Broadcast lists on the phone, synced through app-state. Participants are identified by their LID (lidJid), optionally paired with the phone-number JID (pnJid):
await client.broadcastList.setList({
  id: 'list-1',
  listName: 'Friends',
  participants: [
    { lidJid: 'a@lid', pnJid: 'a@s.whatsapp.net' },
    { lidJid: 'b@lid' }
  ],
  labelIds: ['L1'] // optional — attach business labels
})
Remove a list by its id:
await client.broadcastList.removeList('list-1')
​
Sending to a list
send takes the same content union as client.message.send — text, media, polls, and so on — plus the broadcast listJid (the list id with an @broadcast suffix) and the explicit recipients to fan out to:
const result = await client.broadcastList.send({
  listJid: 'list-1@broadcast',
  content: 'Weekend sale starts now! 🎉',
  recipients: ['a@lid', 'b@lid']
  // options: { ... }  // same shape as client.message.send options
})

console.log(result.id) // the published message id
Each recipient is encrypted for individually (a fanout), so a single send call is effectively N direct sends behind one request. Pass the usual send options through options.
Broadcast lists are not newsletters/channels: a broadcast reaches your existing contacts as private 1:1 messages, while a channel is a public, follower-based feed.
