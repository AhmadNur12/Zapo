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


Bots

Discover WhatsApp bots, send prompts, and receive streamed responses with zapo. Works with any WhatsApp bot, including Meta AI and third-party agents.

client.bot (WaBotCoordinator) works with WhatsApp bots - any account on the @bot domain. Meta AI is the most common one, but it is not the only bot; listBots() returns every bot available to your account. The coordinator discovers bots, reads their profiles, sends prompts, and decrypts the streamed chunks of a bot’s reply.
​
Discovering bots & getting a bot JID
You don’t hard-code bot JIDs - you discover them with listBots() and pick one:
const bots = await client.bot.listBots()
// WaBotInfo[]: { jid, fbidJid, personaId, isDefault, section?, count? }

// Pick the default bot (typically Meta AI), or choose another by section/name
const bot = bots.find((b) => b.isDefault) ?? bots[0]
const botJid = bot.jid // e.g. '13135550002@bot'

// Inspect a bot's profile (commands, prompts, creator metadata)
const profile = await client.bot.getBotProfile(botJid)
// WaBotProfileResult | null: name, description, category, prompts, commands, creator…
WaBotInfo.jid is the value you pass below as to (direct path) or options.botJid (mention path).
​
Sending a prompt
sendPrompt(to, content, options?) invokes a bot. There are two paths depending on to:
​
Direct path — chat with the bot
When to is a @bot JID, you’re chatting with the bot directly. zapo generates a fresh aiThreadId (a conversation id); reuse it on later prompts to keep context:
// Start a conversation
const first = await client.bot.sendPrompt(botJid, 'Explain WebSockets in one line')

// Continue it — pass the same aiThreadId to keep context
await client.bot.sendPrompt(botJid, 'Now in two lines', {
  aiThreadId: myThreadId
})
​
Mention path — invoke a bot inside a group
When to is a group/chat JID, you must name the bot via options.botJid. The bot is invoked indirectly through a mention:
// botJid comes from listBots() — see "Discovering bots" above
await client.bot.sendPrompt(groupJid, '@MetaAI summarize the last messages', {
  botJid,
  extraMentionedJids: [] // optional extra mentions alongside the bot
})
On the mention path, aiThreadId / aiThreadType are ignored — bots drop the request if persona/thread metadata is attached to a mention.
WaBotPromptOptions extends WaSendMessageOptions and adds botJid, personaId, capabilities, extraMentionedJids, aiThreadId, and aiThreadType.
​
Receiving the streamed reply
A bot’s reply does not arrive as one message. It streams as multiple encrypted chunks, surfaced on the message_bot_chunk event. zapo decrypts them automatically on every incoming message, so you just listen:
const buffers = new Map<string, string>()

client.on('message_bot_chunk', (event) => {
  // WaIncomingBotChunkEvent
  const text = event.message?.conversation ?? ''

  // Concatenate chunks in arrival order using editType
  const prev = buffers.get(event.targetMessageId) ?? ''
  buffers.set(event.targetMessageId, prev + text)

  if (event.editType === 'last' || event.editType === 'full') {
    console.log('full reply:', buffers.get(event.targetMessageId))
    buffers.delete(event.targetMessageId)
  }
})
The chunk event fields:
Field	Meaning
key.participant / key.remoteJid	The bot (sender = key.participant ?? key.remoteJid).
targetMessageId	The id of the prompt this reply answers — your stream key.
editType	Chunk position: first → inner → last, or a single full.
message	The decrypted chunk content (Proto.IMessage).
plaintext	Raw decrypted bytes.
Reconstruct the full answer by concatenating chunks for a given targetMessageId in arrival order until you see last (or a single full).
​
Manual chunk decryption
zapo calls tryDecryptChunk for you on each incoming message, so you rarely need it. If you manage incoming events yourself, you can invoke it explicitly:
client.on('message', async (event) => {
  await client.bot.tryDecryptChunk(event)
})
It silently no-ops when the chunk isn’t addressed to you or the parent prompt secret isn’t available.

Presence & status

Broadcast online presence, send typing and recording indicators, subscribe to contact presence, and post WhatsApp status updates with text and media.

​
Own presence
Broadcast whether the account is online with client.presence.send:
await client.presence.send('available')   // appears online
await client.presence.send('unavailable') // appears offline
The presence announced right after connecting is controlled by the markOnlineOnConnect option.
​
Typing indicators (chat-state)
sendChatstate sends a per-chat hint such as typing or recording:
// "typing…"
await client.presence.sendChatstate(jid, { state: 'composing' })

// "recording audio…"
await client.presence.sendChatstate(jid, { state: 'recording' })

// clear the indicator
await client.presence.sendChatstate(jid, { state: 'paused' })
A common pattern is to show typing briefly before replying:
await client.presence.sendChatstate(jid, { state: 'composing' })
await new Promise((r) => setTimeout(r, 1200))
await client.message.send(jid, 'Done thinking!')
await client.presence.sendChatstate(jid, { state: 'paused' })
​
Subscribing to a contact
To receive a contact’s presence and chat-state, subscribe to them:
await client.presence.subscribe(jid)

client.on('presence', (event) => {
  console.log(event.type, event.lastSeen)
})

client.on('chatstate', (event) => {
  console.log(event.state, 'from', event.participantJid)
})
Subscriptions are per-JID and live only for the current connection. After a reconnect you must re-subscribe to keep receiving updates.
​
Status broadcasts
Post a status (the “stories” feature) with client.status (WaStatusCoordinator). The content is the same content union as a normal message; you provide the recipient list:
const result = await client.status.send({
  content: 'Hello from my status!',
  recipients: ['5511999999999@s.whatsapp.net', '5511888888888@s.whatsapp.net']
})
Media works too:
await client.status.send({
  content: { type: 'image', media: './story.jpg', mimetype: 'image/jpeg' },
  recipients
})
​
Status privacy & mute
// Who can see your status
await client.status.setPrivacy({ /* WaSetStatusPrivacyInput */ })

// Mute a contact's status
await client.status.setUserMuted(jid, true)

// Revoke a status you posted
await client.status.revokeStatus({ messageId, recipients })


Profile, privacy & business


Manage your WhatsApp profile, change privacy settings, edit the blocklist, and read business profiles and hours with the profile coordinator.

​
Profile
client.profile (WaProfileCoordinator) reads and writes profile fields for your account and looks them up for others.
​
Profile picture
import { readFile } from 'node:fs/promises'

// Read someone's picture (or your own) — defaults to the low-res preview
const pic = await client.profile.getProfilePicture(jid)

// High-resolution original (the full-size avatar the user uploaded)
const fullPic = await client.profile.getProfilePicture(jid, 'image')

// Set your own
await client.profile.setProfilePicture(await readFile('./avatar.jpg'))

// Remove it
await client.profile.deleteProfilePicture()
getProfilePicture(jid, type?, existingId?) returns a WaProfilePictureResult — { url?, directPath?, id?, type? }. The second argument picks between the compact 'preview' variant (default) and the high-resolution 'image' original; both flavors are returned as the same envelope, only the bytes behind url / directPath differ. Pass the cached existingId to let the server short-circuit when the picture hasn’t changed (the result then comes back without the new url/directPath).
​
About / status text
const about = await client.profile.getStatus(jid)
await client.profile.setStatus('Available')
​
Push name
pushName is the display name peers see for your account in chats and group participant lists. The change is applied to the local credentials immediately (so client.getCredentials()?.pushName reflects the new value right away) and is routed through an app-state mutation; peers see the new name on your next outgoing message.
await client.profile.setPushName('Alice')
Passing an empty string resets the name to the device fingerprint default.
​
Default disappearing mode
setDisappearingMode sets the account-wide default lifetime applied to new 1:1 chats you start. Existing chats keep their per-chat setting.
// 0 disables, 86400 = 24h, 604800 = 7d, 7776000 = 90d
await client.profile.setDisappearingMode(604_800)

// Disable
await client.profile.setDisappearingMode(0)
For per-group disappearing messages, see Groups → disappearing messages.
​
Batched lookups
const profiles = await client.profile.getProfiles([jidA, jidB])
const usernames = await client.profile.getUsernames([jidA, jidB])
const modes = await client.profile.getDisappearingMode([jidA, jidB])
​
Check if a number is on WhatsApp
Resolve phone numbers to their LID and learn whether each is registered on WhatsApp:
const results = await client.profile.getLidsByPhoneNumbers(['+55 11 99999-9999'])
// → [{ phoneJid: '5511999999999@s.whatsapp.net', lidJid: '…@lid', exists: true }]

for (const r of results) {
  if (r.exists) console.log(r.phoneJid, 'is on WhatsApp →', r.lidJid)
}
​
Usernames
const mine = await client.profile.getOwnUsername()
const available = await client.profile.checkUsernameAvailability('myhandle')

await client.profile.setUsername({ username: 'myhandle' })
await client.profile.deleteUsername()
setUsername, setUsernameKey, and resolveUsername all run the same local rules that WhatsApp applies (length, allowed characters, 4-digit key shape) and throw on failure before hitting the network — a call that returns without throwing has passed local validation, so setUsername returning false narrows the cause to a server-side outcome (taken, rate-limited).
Resolve a handle to its LID with resolveUsername. The result is a discriminated union: 'found' gives you the JID (plus isBusiness / pnJid when applicable), 'key-required' means the server withheld the JID until you supply the 4-digit lookup key, and 'not-found' is terminal.
// '@name' and a 'name:1234' key suffix in the handle are accepted
let result = await client.profile.resolveUsername({ username: '@alice' })

if (result.status === 'key-required') {
  const usernameKey = await askUserForFourDigitKey() // your UI
  result = await client.profile.resolveUsername({ username: '@alice', usernameKey })
}

if (result.status === 'found') {
  console.log(result.jid, result.isBusiness, result.pnJid)
} else if (result.status === 'not-found') {
  console.log('no such handle')
}
​
Privacy
client.privacy (WaPrivacyCoordinator) controls privacy categories and the blocklist.
​
Privacy settings
const settings = await client.privacy.getPrivacySettings()

// Update a single setting
const dhash = await client.privacy.setPrivacySetting('lastSeen', 'contacts')
Setting names live on WA_PRIVACY_SETTING_TO_CATEGORY, allowed values per setting on WA_PRIVACY_SETTING_VALUES — every setting takes only the values WhatsApp Web accepts for it. The full list: lastSeen, online, profilePicture, about, readReceipts, groupAdd, callAdd, messages, defenseMode, linkedProfiles, pix.
linkedProfiles controls who can see the Accounts Center profiles linked to this account. Takes the same visibility values as lastSeen / profilePicture ('all' | 'contacts' | 'contact_blacklist' | 'none'), deny-list included.
pix controls who can see the Pix key on the profile (Brazil-only payments surface). Same visibility values as lastSeen.
setPrivacySetting returns the dhash the server echoes back — the version stamp of that category’s disallowed list, present only while the category sits on 'contact_blacklist' (null otherwise). Setting a category to 'contact_blacklist' here only flips the mode; populate the list with setDisallowedList.
​
Blocklist
const { jids, entries } = await client.privacy.getBlocklist()

await client.privacy.blockUser(jid)
await client.privacy.unblockUser(jid)
WaBlocklistResult now carries entries: readonly WaPrivacyListEntry[] alongside jids — same membership, with a per-entry username?: string handle set only when the server identified the entry that way. block / unblock writes can address a migrated contact by username or display_name (the entry resolves through the same identifier lookup), instead of falling back to unknown_identifier.
​
Disallowed lists
For settings scoped to a specific list of contacts (e.g. “share with everyone except…”):
// Read the current deny-list for a category
const { jids, dhash } = await client.privacy.getDisallowedList('lastSeen')

// Populate or edit it (also switches the category to 'contact_blacklist')
await client.privacy.setDisallowedList('lastSeen', {
  add: ['5511999999999@s.whatsapp.net'],
  remove: ['5511888888888@s.whatsapp.net']
})
setDisallowedList(category, input) carries the mode change and the entries in one stanza (the server has no separate deny-list endpoint). Inputs accept phone jids, LID jids, or bare phone numbers and are resolved to both addressing forms, the same way blockUser does. A migrated contact may be addressed by username or display_name (via the entry’s own identifier resolution) rather than falling back to unknown_identifier. The write is versioned by a dhash read right before sending; if another device mutated the list in between, the server answers 409 and the call refetches the stamp and retries once. The eligible categories are exposed as WA_PRIVACY_DISALLOWED_LIST_CATEGORIES (about, groupAdd, lastSeen, profilePicture, linkedProfiles, pix).
WaPrivacyDisallowedListResult mirrors WaBlocklistResult: entries: readonly WaPrivacyListEntry[] accompanies jids with the same membership, adding a per-entry username?: string when the server identified the entry that way.
​
Reacting to changes on other devices
A privacy change made on the primary (or another companion) surfaces on the privacy event with the full refreshed category set plus any disallowed list the server reported changed. The values come from a fresh read — never from the notification payload — and a burst of changes on the phone collapses into a single event, debounced 1s.
client.on('privacy', (event) => {
  console.log('privacy refreshed:', event.settings)
  for (const list of event.disallowedLists) {
    console.log(list.setting, 'deny-list:', list.jids)
  }
})

// Same story for the blocklist
client.on('blocklist', ({ jids }) => {
  console.log('blocklist now', jids.length, 'entries')
})
Call client.privacy.refreshFromAccountSync() to force a refresh yourself — the result is both returned and re-emitted as the privacy event, so the client’s view stays consistent regardless of who triggered it. Concurrent calls are deduplicated.
​
Business
client.business (WaBusinessCoordinator) reads business profiles and verified names, and manages your own business profile.
Business account — required to edit your own profile or cover photo; the read methods work for any account.
// Read business profiles (batched)
const profiles = await client.business.getBusinessProfile([jidA, jidB])

// Verified-name lookups
const name = await client.business.getVerifiedName(jid)
const names = await client.business.getVerifiedNames([jidA, jidB])

// Edit your own business profile
await client.business.editBusinessProfile({ /* WaEditBusinessProfileInput */ })

// Cover photo
await client.business.updateCoverPhoto(mediaSource)
await client.business.deleteCoverPhoto(coverId)
​
Typed business hours
WaBusinessHoursDay and WaBusinessHoursMode are union aliases ('sun' | 'mon' | …, 'open_24h' | 'specific_hours' | 'appointment_only'). The values are also frozen on WA_BUSINESS_HOURS_DAYS and WA_BUSINESS_HOURS_MODES. Passing an unknown mode to editBusinessProfile now throws a clear local error instead of the server replying with 406 not-acceptable — closed days are still expressed by omitting them from config, not by a dedicated mode.
import { WA_BUSINESS_HOURS_DAYS, WA_BUSINESS_HOURS_MODES } from 'zapo-js'

await client.business.editBusinessProfile({
  businessHours: {
    timezone: 'America/Sao_Paulo',
    config: [
      {
        dayOfWeek: WA_BUSINESS_HOURS_DAYS.MON,
        mode: WA_BUSINESS_HOURS_MODES.SPECIFIC_HOURS,
        openTime: 540,  // minutes from midnight; 540 = 09:00
        closeTime: 1080 // 18:00
      },
      { dayOfWeek: WA_BUSINESS_HOURS_DAYS.SAT, mode: WA_BUSINESS_HOURS_MODES.OPEN_24H }
      // sun is closed — omit it
    ]
  }
})
​
Chat settings
Per-chat settings — mute, pin, archive, read, lock, star, clear, delete — live on client.chat and sync across your devices. They have their own guide:
Managing chats
Mute, pin, archive, mark read, lock, star messages, clear, and delete chats.

Reconnection

zapo does not auto-reconnect by design — follow this pattern to detect dropped sessions, rebuild the client, and resume without duplicate connections.

WaClient does not reconnect automatically. This is a deliberate design choice: reconnection policy (backoff, max retries, alerting) belongs to your application. You listen for connection: close and decide what to do.
​
Internal recovery layers
The “no auto-reconnect” rule applies to the session lifecycle: once a connection: close event fires, the client will not reopen by itself. Two lower-level retries do live inside the stack though – you will see them in logs and you do not need to handle them.
WebSocket transport. If the socket drops before a successful noise handshake, WaComms retries internally every reconnectIntervalMs (default 2000) up to maxReconnectAttempts. Once the handshake completes, the counter resets and any subsequent drop surfaces as a connection event for your app to handle.
Pairing transition. Right after a QR/code pair succeeds, the client restarts the socket as a registered session. No connection: close event fires for this – it is invisible from the outside.
The client_too_old (HTTP 405) recovery covered below is a third, opt-in layer.
​
Connection lifecycle








connect()

opened

failed

connection close

reconnect (isLogout = false)

disconnect()

isLogout = true (re-pair)

Connecting

Open

Closed

​
The connection event
connection is a discriminated union on status:
client.on('connection', (event) => {
  if (event.status === 'open') {
    console.log('connected', { isNewLogin: event.isNewLogin })
    return
  }

  // status === 'close'
  console.log('disconnected', {
    reason: event.reason,
    code: event.code,
    isLogout: event.isLogout
  })
})
On close:
isLogout: true — the device was unlinked (server-side logout). Do not reconnect; the credentials are gone and you must re-pair.
isLogout: false — a transient drop. Safe to reconnect with the stored credentials.
​
A reconnection loop with backoff
const MAX_ATTEMPTS = 10

async function connectWithRetry(client: WaClient) {
  let attempt = 0

  client.on('connection', (event) => {
    if (event.status === 'open') {
      attempt = 0 // reset backoff on a healthy connection
      return
    }
    if (event.isLogout) {
      console.error('logged out — re-pairing required')
      return
    }
    void reconnect()
  })

  async function reconnect() {
    if (attempt >= MAX_ATTEMPTS) {
      console.error('giving up after', attempt, 'attempts')
      return
    }
    const delayMs = Math.min(30_000, 1_000 * 2 ** attempt)
    attempt += 1
    console.log(`reconnecting in ${delayMs}ms (attempt ${attempt})`)
    await new Promise((r) => setTimeout(r, delayMs))
    try {
      await client.connect()
    } catch (err) {
      console.error('reconnect failed', err)
      void reconnect()
    }
  }

  await client.connect()
}
​
Recovering from client_too_old (HTTP 405)
If the server starts rejecting the noise handshake with failure_client_too_old, the bundled WA Web version is out of date. Two options:
Set recoverFromClientTooOld: true on WaClient — on every 405 the client fetches the current version from web.whatsapp.com, swaps it in, and reconnects.
Pass a version resolver that returns a fresh string per connect.
Both are stopgaps — upgrade zapo when a release ships with a refreshed default.
​
After reconnecting
Some state is connection-scoped and must be re-established after a successful reconnect:
Presence subscriptions — re-subscribe() to any contacts you were watching (Presence).
Newsletter live updates — re-subscribeLiveUpdates() if you rely on them.
Persisted state (credentials, Signal sessions, app-state) is restored from the store automatically — you do not re-pair on a normal reconnect.
​
Graceful shutdown
Call disconnect() for a clean shutdown that keeps credentials so you can resume later:
process.on('SIGINT', async () => {
  await client.disconnect()
  process.exit(0)
})
This flushes pending write-behind data and closes the socket without unlinking the device.

Errors & disconnects

Read DisconnectReason codes, handle stream failures and error stanzas from WhatsApp, and decide when to reconnect versus stop the session for good.

zapo surfaces problems through three event channels:
connection with status: 'close' — the socket dropped, carrying a reason and optional code.
stream_failure — a stream-level failure, often just before a close.
stanza_error — a single request/stanza was rejected, without dropping the connection.
For the reconnection loop itself (backoff, isLogout, graceful shutdown), see Reconnection. This page is about understanding why something failed.
​
Disconnect reasons
The close event is { status: 'close', reason, code, isLogout }. reason is a WaDisconnectReason string; code is a numeric WaConnectionCode (or null). Use them to decide whether to reconnect:
client.on('connection', (event) => {
  if (event.status !== 'close') return
  if (event.isLogout || isFatal(event.reason)) {
    console.error('not reconnecting:', event.reason, event.code)
    return
  }
  void reconnect() // see the Reconnection guide
})

const FATAL = new Set([
  'stream_error_replaced',          // same credentials connected elsewhere
  'stream_error_device_removed',    // device unlinked
  'stream_error_force_logout',      // server forced logout (code 516)
  'failure_not_authorized',         // 401
  'failure_banned',                 // 406
  'failure_locked',                 // 403
  'failure_client_too_old',         // 405 — bump the advertised version
  'failure_bad_user_agent',         // 409
  'primary_identity_key_change'     // account identity changed — re-pair
])
const isFatal = (reason: string) => FATAL.has(reason)
stream_error_force_login (code 515) is not fatal — it’s a routine “reconnect now” the server sends right after pairing and occasionally during a session. Just reconnect with the stored credentials.
Transient reasons worth reconnecting on include stream_error_force_login, stream_error_ack, stream_error_xml_not_well_formed, stream_error_other, failure_service_unavailable, and comms_stopped. client_disconnected means you called disconnect() — expected, don’t reconnect.
​
Code reference
When present, code (on the close event and on stream_failure.failureCode) is one of:
Code	Meaning	Action
401	Not authorized	Re-pair
402	Temporarily banned	Back off hard
403	Locked	Stop
405	Client too old	Bump version
406	Banned	Stop
409	Bad user agent	Fix device fingerprint
500	Internal server error	Retry later
503	Service unavailable	Back off and retry
515	Reconnect required	Reconnect now
516	Forced logout	Re-pair (isLogout: true)
The reason strings live in WA_DISCONNECT_REASONS and the 515/516 stream codes in WA_STREAM_SIGNALING, both exported from the package root. The numeric code values are typed as WaConnectionCode (also exported).
​
Stream failures
stream_failure (WaIncomingFailureEvent) carries the raw failure detail and usually precedes a disconnect — log it for context:
client.on('stream_failure', (event) => {
  console.warn('stream failure', {
    reason: event.failureReason,
    code: event.failureCode,
    message: event.failureMessage,
    url: event.failureUrl
  })
})
​
Error stanzas
stanza_error (WaIncomingErrorStanzaEvent) reports that a single stanza was rejected — for example a malformed query or a throttled request. The connection stays up:
client.on('stanza_error', (event) => {
  console.warn('stanza error', event.code, event.text)
})
A rejected client.message.send or client.lowlevel.query typically rejects its own promise too, so wrap individual calls in try/catch for per-operation handling; use stanza_error for visibility into errors that aren’t tied to a call you await.

Multi-session deployments

Run many WhatsApp accounts in one process with a shared store — what’s per-session vs shared, the single-writer rule, memory budget, sharding, and graceful shutdown.

zapo is designed so a single process can drive many accounts off one shared store. Each account lives behind a stable sessionId; everything that’s safe to share (the backend connection pool, the WebSocket factory, the logger) is shared, and everything that’s account-specific (Signal sessions, identities, app-state, mailbox) is partitioned by sessionId.
​
The pattern
import { createStore, WaClient, createPinoLogger } from 'zapo-js'
import { createPostgresStore } from '@zapo-js/store-postgres'

const store = createStore({
  backends: { postgres: createPostgresStore({ pool: { connectionString: process.env.DATABASE_URL } }) },
  providers: {
    auth: 'postgres', signal: 'postgres', preKey: 'postgres',
    session: 'postgres', identity: 'postgres', senderKey: 'postgres',
    appState: 'postgres', privacyToken: 'postgres',
    messages: 'postgres', threads: 'postgres', contacts: 'postgres'
  }
})

const logger = await createPinoLogger({ level: 'info' })

const clients = ['account-a', 'account-b', 'account-c'].map(
  (id) => new WaClient({ store, sessionId: id }, logger)
)

await Promise.all(clients.map((c) => c.connect()))
sessionId is the durable key for an account — same id across restarts resumes the same paired device. Changing it orphans the previous credentials.
​
What’s per-session vs shared
Layer	Scope
Backend connection pool / file handle	Shared across all sessions
Per-domain stores (auth / signal / preKey / session / identity / senderKey / appState / privacyToken / messages / threads / contacts)	Per sessionId
Cache domains (retry / groupMetadata / deviceList / messageSecret)	Per sessionId
L1 cacheLayer (when enabled)	Per sessionId, per process
memory.limits caps	Applied per session (multiply by N for total RAM)
WaClient state (handlers, retry queue, coordinators)	Per WaClient instance
Switching to a multi-tenant setup is a matter of (1) instantiating N WaClients on the same store, and (2) sizing your backend pool + memory budget for N concurrent sessions.
​
Session lifecycle
store.session(sessionId) is memoized. The first call materializes the per-domain bundle (per-session locks, optional cache wrappers, …) and caches it inside the store; later calls with the same id return the same bundle.
const a1 = store.session('account-a')
const a2 = store.session('account-a')
a1 === a2 // true
WaClient calls store.session(sessionId) on demand; you do not usually call it yourself.
​
Adding tenants on the fly
There is no preregistration step — just construct a new WaClient with a new sessionId:
function spawn(sessionId: string): WaClient {
  const client = new WaClient({ store, sessionId }, logger)
  // hook your event listeners, then connect()
  return client
}
​
Removing tenants
For long-running multi-tenant processes, three options — each with a different scope:
await client.logout() — logical removal. Wipes the persistent state for that sessionId (subject to logoutStoreClear) and unlinks the device server-side. The bundle stays in the in-store map until you destroy it or the process ends; a subsequent WaClient({ store, sessionId }) on the same id reuses the same bundle.
await storeSession.destroy() — mid-process reclaim. Tears down the session’s per-domain stores and evicts the bundle from the in-store map (the sessionId is released as destruction starts), so a concurrent or later store.session(id) builds a fresh bundle. The teardown is idempotent — repeat calls await the same in-flight promise — and teardown failures are logged, never thrown.
await store.destroy() — process shutdown. Tears down every live session and the registered backends. store.session() throws afterwards; the store is single-shot.
For a bounded reset that keeps the session alive, await storeSession.destroyCaches() swaps the cache domains (retry / groupMetadata / deviceList / messageSecret) for freshly-built instances instead of closing them — useful when you want to drop stale entries from a persistent cache backend without losing the session. Concurrent resets are serialized and cache references captured before the call reject afterwards, so recreate the client to pick up the fresh cache stores.
session(id) returns the same bundle instance while it’s alive — so a WaClient holding a stale reference before you destroyed the session keeps hitting the closed bundle. In practice: destroy the client’s connection first (client.disconnect()), then session.destroy(), then build a new client if you want the same sessionId back.
​
Process ownership
In multi-process deployments, decide how sessionIds map to processes:
One process per sessionId via consistent hashing / sticky routing on the load balancer or queue (simplest).
Leader election before opening the client (a Postgres advisory lock, Redis SET NX, etcd lease) — useful for HA failover.
The opt-in cacheLayer tightens this: its L1 has no cross-process invalidation channel, so a sessionId’s backend rows should be owned by one process across its lifecycle. A takeover process’s L1 starts cold and may serve stale reads before catching up to writes the previous owner made.
​
Sharing a media processor
WaMediaProcessor is a stateless wrapper around your media binaries (sharp, ffmpeg/ffprobe, file-type). The same instance can serve every WaClient — there is no per-session state inside the processor, so reusing it avoids paying the binary-lookup / lazy-import cost N times.
import { createMediaProcessor } from '@zapo-js/media-utils'

const processor = createMediaProcessor()

const clients = tenants.map((id) => new WaClient(
  { store, sessionId: id, media: { processor } }, // same instance, every session
  logger
))
Each processor method receives an optional ctx: WaMediaProcessorCallContext argument carrying that call’s Logger. The runtime fills it with the calling session’s logger, so warnings (missing binary, failed detectMimetype, …) land with the right per-session bindings automatically — no setup needed. Custom processors should consume ctx.logger per call and not cache it, since the same instance is shared across sessions.
​
Memory budget
WaCreateStoreOptions.memory.limits caps apply per session. With N concurrent sessions, the worst-case in-process RAM scales linearly:
Cap	Per session	With N = 50 sessions
signalSessions: 5_000	up to 5 000 Double-Ratchet entries	up to 250 000
signalRemoteIdentities: 5_000	up to 5 000 identity rows	up to 250 000
groupMetadataGroups: 1_000	up to 1 000 cached groups	up to 50 000
messages: 10_000 (when providers.messages: 'memory')	up to 10 000 messages	up to 500 000
Tune the per-session caps downward as N grows, or move the mailbox/large-cardinality domains to a persistent backend (the in-memory provider exists for tests and small accounts). TTLs in memory.cacheTtlMs are independent of N — they only cap how long an entry survives in each cache.
​
Sharding strategies
Layout	Use when
One process · N sessions · one store	A few tenants, all light traffic. Simplest setup; the process is a shared point of failure for all tenants.
N processes · one session each · shared backend	High per-tenant load, or you want blast-radius isolation per tenant. The most robust at scale. Requires a network backend (@zapo-js/store-postgres / mysql / redis / mongo).
K processes · M/K sessions each · shared backend	The middle ground at scale. Pack tenants per process until CPU saturates, then add a process. Pair with consistent hashing on sessionId so the same account always lands on the same process.
@zapo-js/store-sqlite is single-host only and the SQLite file is held by one process — pick one of the network backends for any layout with more than one process.
​
Graceful shutdown
async function shutdown() {
  await Promise.all(clients.map((c) => c.disconnect()))
  await store.destroy()
}
for (const signal of ['SIGINT', 'SIGTERM'] as const) {
  process.on(signal, shutdown)
}
client.disconnect() flushes the per-session write-behind queue and closes the socket without unlinking the device, so the next boot resumes from the store. store.destroy() then releases the shared backend (pool, file handle, …). Calling disconnect() on every client before store.destroy() ensures each session’s pending writes flush; store.destroy() does not do that for you.
Don’t substitute logout() for disconnect() here — logout() unlinks the device server-side and clears stored state. Use it only when you intentionally want the account removed.

Production & deployment

Run zapo reliably in production: durable persistence, graceful shutdown, scaling multiple sessions, reconnection strategy, and the knobs that matter.

zapo is built for long-lived, multi-session workloads. This page collects the operational decisions that matter once you move past a local prototype.
​
Persist credentials (don’t re-pair)
Production sessions must use a durable store for the auth and Signal domains, plus a stable sessionId across restarts. The in-memory store loses everything on exit, forcing a re-pair on every boot.
const client = new WaClient({ store, sessionId: 'tenant-42' }, logger)
Changing sessionId orphans the previous credentials — treat it as the durable key for a device/account.
​
Choose a store backend
Backend	Best for
@zapo-js/store-sqlite	Single process / single host — the simplest, fastest local option.
@zapo-js/store-postgres · @zapo-js/store-mysql	Multiple hosts, relational ops, managed backups.
@zapo-js/store-redis	Low-latency cache + persistence.
@zapo-js/store-mongo	Document-oriented deployments.
You can mix backends per domain (e.g. auth/signal in Postgres, caches in Redis). See Installation and the stores reference.
​
Graceful shutdown
Call disconnect() (never logout()) on shutdown — it flushes pending write-behind data and closes the socket without unlinking the device, so the next boot resumes from the store.
for (const signal of ['SIGINT', 'SIGTERM'] as const) {
  process.on(signal, async () => {
    await client.disconnect()
    process.exit(0)
  })
}
logout() unlinks the device server-side and clears stored state — it forces a full re-pair. Use it only to permanently disconnect, never as a shutdown hook.
​
Run many sessions
A single store can hold many independent accounts, each keyed by sessionId. Create one WaClient per account:
const clients = tenants.map((id) => new WaClient({ store, sessionId: id }, logger))
await Promise.all(clients.map((c) => c.connect()))
Each client pairs, reconnects, and emits events independently. Budget memory/CPU per session (each holds Signal state and in-memory caches), and shard across processes/hosts as you scale — one process per session is the simplest isolation model. See Multi-session deployments for the full operational guide.
​
Tune for throughput
Write-behind batches incoming message/thread/contact writes off the hot path. Tune writeBehind.maxPendingKeys / maxWriteConcurrency / flushTimeoutMs to your database. (config)
History sync (history.enabled) is on by default and adds a large initial download. Set it to false if you don’t persist mailbox/threads/contacts; set requireFullSync deliberately.
Bots that shouldn’t appear online: markOnlineOnConnect defaults to false, so bots are invisible on connect out of the box. Pass true only when you want a visible “online” presence.
Timeouts (iqTimeoutMs, keepAliveIntervalMs, deadSocketTimeoutMs, …) ship with production defaults — override only with a reason. (config)
​
Reconnection & error policy
zapo does not auto-reconnect — own the policy. Wire a backoff loop (Reconnection) and classify failures (Errors & disconnects) so you stop on fatal reasons (banned, not_authorized, logout) instead of hammering the server.
​
Logging
Use a structured logger in production:
const logger = await createPinoLogger({ level: 'info', pretty: false })
pretty: false emits JSON lines suited to log aggregators. Drop to debug / trace only when investigating.
​
Security & versioning
Credentials are secrets. WaAuthCredentials holds the device keys — if you persist them outside the built-in store, encrypt at rest. (Authentication)
Never enable dangerous.* in production — those flags disable security checks. (config)
Versioning. zapo is 1.0 and follows semantic versioning — breaking changes only land in a new major. Use a version range or lockfile as usual, and review the changelog before major upgrades.

End-to-end testing without WhatsApp

Run your bot / CRM / notification pipeline against @zapo-js/fake-server — a real Noise / Signal / WhatsApp-protocol server in-process, no phone tied up, no risk to production accounts. Works with any conforming client (zapo, Baileys, whatsmeow).

@zapo-js/fake-server is an in-process fake WhatsApp Web server that speaks Noise XX/IK, X3DH + Double Ratchet, group SenderKey, app-state sync, and media upload/download over self-signed HTTPS — everything a real conforming client sees on the wire. It exists so you can test your app end-to-end without touching WhatsApp: no phone tied up, no risk to a production account, no flakiness from a live network, CI-friendly.
This guide targets app authors — bots, CRMs, notification services, integrations — regardless of whether the underlying WhatsApp library is zapo-js, Baileys, whatsmeow, or another conforming client. For the library-internal use (cross-check suite, benchmarks) see Dev tools.
​
Works with any WhatsApp library
The fake server is a wire-level implementation of the WhatsApp protocol. It has no zapo-js dependency at runtime — it speaks the same Noise / Signal / app-state bytes as WhatsApp’s own edges. Strict clients (Baileys forks, whatsmeow, and others) work against it out of the box:
The Noise cert chain sets notBefore / notAfter so validity-checking clients accept it as unexpired.
The passive-set IQ and the encrypt <count> prekey-count query are answered by default (non-zapo clients block on both).
After every login the server sends <ib><offline count="0"/></ib> so buffered-event clients flush.
FakePeer message ids are WA-style hex — no @ characters that would tokenize as a JID in a strict decoder.
If your app runs against a real WA edge, it should run against the fake one with only a URL / cert swap.
​
Install
npm install --save-dev @zapo-js/fake-server zapo-js
zapo-js is a peer dependency of the fake server (it imports the underlying Noise / Signal / crypto / proto primitives at runtime), so it needs to be present in the tree even when your app is built on a different WhatsApp library. Node.js >= 20.9.0.
​
Quick start (programmatic)
The zapo-js wiring below is what a zapo-js app looks like; for Baileys / whatsmeow, keep your existing client setup and only override the socket URL, media proxy, and Noise root CA to point at the fake server.
import { FakeWaServer } from '@zapo-js/fake-server'
import { createStore, WaClient } from 'zapo-js'

const server = await FakeWaServer.start()

const client = new WaClient({
  store: createStore({
    providers: { auth: 'memory', signal: 'memory', senderKey: 'memory', appState: 'memory' }
  }),
  sessionId: 'test',
  chatSocketUrls: [server.url],
  testHooks: { noiseRootCa: server.noiseRootCa },
  proxy: { mediaUpload: server.mediaProxyAgent, mediaDownload: server.mediaProxyAgent }
})

await client.connect()
const pipeline = await server.waitForAuthenticatedPipeline()
// ...drive the flow, assert on both sides...
await server.stop()
testHooks.noiseRootCa trusts the fake server’s certificate without bypassing verification — the full cert-chain check still runs.
​
Pinning across processes
listen() mints a fresh random root CA per start and exposes the public half on server.noiseRootCa — perfect for the in-process test above, where the client is built after the server is listening. A client in a separate process has to pin the trust anchor before it dials, and at that point the server is not yet listening: there’s nothing to read back.
Pass a noiseRootCa derived from a shared seed instead, so both sides can compute the same CA ahead of time:
import { FakeWaServer, type FakeNoiseRootCa } from '@zapo-js/fake-server'
import { deriveRootCaFromSeed } from './my-test-seed' // your own deterministic derivation

const noiseRootCa: FakeNoiseRootCa = await deriveRootCaFromSeed(process.env.TEST_SEED!)

// Server process
const server = await FakeWaServer.start({ noiseRootCa })

// Client process — pins the same anchor before dialling
const client = new WaClient({
  /* ... */,
  testHooks: { noiseRootCa: { publicKey: noiseRootCa.publicKey, serial: noiseRootCa.serial } }
})
FakeNoiseRootCa carries the signing half — the server needs it to sign its cert chain — while the client only ever holds the public counterpart. It signs a chain no real WhatsApp client trusts, so it is test material rather than a real secret, but a seed committed beside the tests is the intended source, not one shared with anything that matters. Omitting the option keeps the previous behaviour: a fresh random CA per listen().
​
Simulating a peer
createFakePeer gives you a simulated contact backed by real Signal crypto. Push messages to your client with sendConversation / sendGroupConversation, and capture what your client sends with expectMessage:
const alice = await server.createFakePeer('5511999999999@s.whatsapp.net')

// Your app receives this as a normal inbound message:
await alice.sendConversation('hello from Alice')

// Your app sends a reply; assert on the decrypted content:
const outbound = await alice.expectMessage({ timeoutMs: 5_000 })
expect(outbound.message?.conversation).toBe('hi Alice')
The peer performs a real X3DH handshake, ratchets each message forward, and validates MACs — so a bug in your app’s Signal handling surfaces the same way it would against WhatsApp itself.
​
Asserting on outbound stanzas
For assertions that don’t need a peer’s viewpoint (a <presence> your app sent, a <receipt> it acked, an IQ it made) subscribe to captured stanzas:
const stanzas: BinaryNode[] = []
server.onCapturedStanza((node) => stanzas.push(node))

await client.presence.send('available')

expect(stanzas.some((n) => n.tag === 'presence' && n.attrs.type === 'available')).toBe(true)
onCapturedStanza returns an unsubscribe function; call it in afterEach to keep tests isolated.
​
Multi-session isolation
Run many app instances against a single server without cross-talk. Provide a sessionKey resolver — the fake server routes each authenticated connection to its own FakeServerSession (isolated peers, groups, prekeys, app-state, captured stanzas):
const server = await FakeWaServer.start({
  sessionKey: ({ clientPayload }) =>
    clientPayload.kind === 'login' ? clientPayload.username : 'pending'
})

// Spawn two apps against the same server (different sessionIds → different sessions):
await appA.connect() // → session '<userA>'
await appB.connect() // → session '<userB>'

// Session-scoped operations:
const alice = await server.sessionFor(pipelineA).createFakePeer(aliceJid)
// or by key:
const bob = await server.session('<userB>').createFakePeer(bobJid)
The resolver runs once per connection, right after authentication, so info.clientPayload is available for keying by login identity. Server-wide handlers (server.registerIqHandler) still apply to every session; session-scoped ones (session.registerIqHandler) stay local.
​
Mobile primary + companion hosting
To test a mobile-primary session — the client.mobile role, where zapo hosts companions — start the server with the raw TCP listener the mobile transport dials, and seed a registered primary into the store. Mobile registration happens out of band against WhatsApp’s HTTP endpoints, so the fake server can’t run it; seedFakeMobilePrimary writes the credentials directly.
import { FakeWaServer, seedFakeMobilePrimary } from '@zapo-js/fake-server'
import { createStore, WaClient } from 'zapo-js'

const server = await FakeWaServer.start({ tcp: true })

const store = createStore({})
const primary = await seedFakeMobilePrimary(store, 'mobile-primary', {
  phoneNumber: '5511999999999'
})

const client = new WaClient({
  store,
  sessionId: 'mobile-primary',
  mobileTransport: {
    deviceInfo: primary.deviceInfo,
    tcpUrl: server.tcpUrl
  },
  testHooks: { noiseRootCa: server.noiseRootCa }
})
await client.connect()
To drive companion linking against this primary, connect a second client as the companion and hand it refs via server.offerCompanionPairing(companionPipeline) — the inverse of runPairing. The primary signs the identity for real; the server just relays:
const companion = new WaClient({ store: companionStore, sessionId: 'companion', /* ... */ })
await companion.connect()
const companionPipeline = await server.waitForAuthenticatedPipeline()

// Push the refs the companion turns into a QR:
const refs = await server.offerCompanionPairing(companionPipeline)

// On the primary side, feed the QR string to client.mobile.linkCompanion(qr).
// The primary signs and uploads pair-device; the server relays pair-success
// back to the companion.
This exercises the full client.mobile.linkCompanion / linkCompanionByCode handshake — signature, ADV epoch, key-index list republish, companion_host_linked event — end to end.
Pairing by code runs between the two clients with the server in the middle; the fake server relays each stage. parseClientPayload on the pipeline classifies the login as web or mobile and surfaces the phone identity (manufacturer, model, os, app version, phone id) for assertions.
​
Programmatic config
FakeWaServer.start(options) accepts:
Option	Purpose
port / path	Bind address override.
tcp	true also starts a raw TCP listener beside the WebSocket one; a mobile-transport client dials it via server.tcpUrl. Default false.
successNodeAttributes	Attributes stamped on the post-handshake <success/> node (lid, display name, props versions, …).
defaultIqHandlers	false starts with an empty router — wire every response yourself via registerIqHandler. Default true.
sessionKey	Multi-session resolver (see above).
onPipeline(listener) fans out to multiple subscribers and returns an unsubscribe function; the pipeline’s parsed clientPayload is exposed on WaFakeConnectionPipeline for identity checks. IQ handlers may return null to fall through to the next matching handler (observe-then-delegate).
​
Standalone CLI
The package ships a fake-wa-server binary. Run it once the dev dep is installed:
npx fake-wa-server --port 5222 --peer 5511888@s.whatsapp.net --log
The --pair <jid> mode drives QR pairing by prompting on stdin for the QR payload the client displays — pair once, then reconnect against the same fake server to iterate.
​
CI recipe
Spin one server per test file (or per suite), tear it down at the end. memory providers on the client keep every test hermetic:
import { FakeWaServer } from '@zapo-js/fake-server'
import { afterAll, beforeAll, test } from 'vitest'

let server: FakeWaServer

beforeAll(async () => {
  server = await FakeWaServer.start()
})

afterAll(async () => {
  await server.stop()
})

test('bot replies to inbound message', async () => {
  const app = await startYourApp({
    socketUrl: server.url,
    noiseRootCa: server.noiseRootCa,
    mediaProxy: server.mediaProxyAgent
  })
  const alice = await server.createFakePeer('5511999999999@s.whatsapp.net')
  await alice.sendConversation('hi')
  const reply = await alice.expectMessage({ timeoutMs: 5_000 })
  expect(reply.message?.conversation).toBe('hello, human')
})
Pair the fake server with in-memory stores on the client. Every test resets on run — no leaked pairing state, no fixture disk juggling.
​
Troubleshooting & FAQ

Answers to the most common questions and pitfalls when running zapo: pairing failures, disconnects, missing events, history sync, and store corruption.

It re-pairs (shows a QR) on every restart

You’re almost certainly running on the in-memory store, which loses credentials when the process exits. Use a durable backend (SQLite, Postgres, …) for the auth domain, and keep a stable sessionId across runs — changing it orphans the previous credentials. See Stores.
The client doesn't reconnect after a drop

By design — zapo does not auto-reconnect. Listen for the connection event with status: 'close' and call connect() again (skip it when isLogout is true). See the reconnection pattern.
My media sends but has no preview / dimensions / waveform

Media still uploads without @zapo-js/media-utils — but without it there’s no processor to generate thumbnails/previews, image-video dimensions, or voice-note waveforms, so it can render as a plain attachment or with no preview. For proper media, install it (npm i @zapo-js/media-utils, plus ffmpeg/ffprobe) and wire a processor through the media option. See Media.
Prefer a stream over a Buffer for media

Pass a file path (string) or a Readable stream to media, not a Buffer — zapo streams the bytes through the pipeline so memory stays flat for large files. On download, prefer downloadToFile/download over downloadBytes.
Proxy isn't being used

The proxy.ws leg needs the ws package (the runtime’s native WebSocket can’t take an HTTP Agent). Media/link-preview legs use an undici dispatcher. See the proxy examples for SOCKS/HTTP/HTTPS and IPv4/IPv6.
Which JID do I reply to in a group?

Always reply to event.key.remoteJid (the group JID), never a participant’s JID. When you have a peer’s LID, prefer the LID — it’s the privacy-preserving, forward-compatible identity. See Identities (PN vs LID).
I receive my own outgoing messages

That’s multi-device sync — your own sends come back on the message event flagged key.fromMe === true. Filter them out if you only want inbound traffic. See Receiving messages.
How do I type the message handler in TypeScript?

Import the event type from the package root — all coordinator and event types are exported:
import type { WaIncomingMessageEvent, WaGroupCoordinator } from 'zapo-js'

client.on('message', (event: WaIncomingMessageEvent) => { /* ... */ })
const groups: WaGroupCoordinator = client.group
Can I register a brand-new number (mobile)?

No. Mobile connections are stable, but zapo intentionally does not provide a registration API — registering a number is complex and requires a physical phone. You connect with already-registered credentials. See Mobile connections.
QR or 8-character code — which should I use?

Both work. QR is the default (auth_qr event). For an 8-character code, call client.auth.requestPairingCode(phone) after the auth_pairing_required event. See Authentication.
logout() vs disconnect()

disconnect() closes the socket but keeps credentials so you can resume later. logout() unlinks the device server-side and clears stored state (per logoutStoreClear). See Authentication.
Handshake fails with HTTP 405 / failure_client_too_old

WhatsApp rejected the bundled WA Web version. Upgrade zapo when possible. As a stopgap, set recoverFromClientTooOld: true to auto-fetch the current version and retry, or pass a version resolver that returns a fresh string per connect. See WhatsApp Web version.
A business/newsletter operation throws

Some operations are gated: editBusinessProfile, cover-photo ops, and broadcast lists are business-only; email binding is mobile-only; several community/newsletter ops require an active MEX transport. The coordinator reference flags each.
​
Still stuck?
Architecture in depth
Understand the layers to debug at the protocol level.
Low-level API
Inspect raw stanzas with the debug events and lowlevel.

Recipes

Copy-paste patterns for the most common things you’ll build with zapo — command bots, media handling, threaded replies, and group moderation tasks.

Short, complete patterns built on the real API. They assume you already have a connected client — see the Quickstart for setup.
​
Extract text from any message
Incoming text can arrive as a plain conversation or an extendedTextMessage (when it has a reply/preview). Normalize both:
function getText(message: { conversation?: string | null; extendedTextMessage?: { text?: string | null } | null } | null | undefined) {
  return message?.conversation ?? message?.extendedTextMessage?.text ?? undefined
}
​
Command router
Parse a leading /command and dispatch. Skip your own outgoing messages with event.key.fromMe:
client.on('message', async (event) => {
  if (event.key.fromMe) return // ignore our own sends (multi-device echo)
  const text = getText(event.message)?.trim()
  const to = event.key.remoteJid
  if (!text || !text.startsWith('/') || !to) return

  const [command, ...args] = text.slice(1).split(/\s+/)
  switch (command) {
    case 'ping':
      await client.message.send(to, 'pong')
      break
    case 'echo':
      await client.message.send(to, args.join(' ') || '(nothing to echo)')
      break
    default:
      await client.message.send(to, `Unknown command: ${command}`)
  }
})
​
Reply with a quote and a mention
client.on('message', async (event) => {
  if (event.key.fromMe) return
  const to = event.key.remoteJid
  const sender = event.key.participant ?? event.key.remoteJid

  await client.message.send(
    to,
    { type: 'text', text: 'got it 👍' },
    { quote: event, mentions: sender ? [sender] : [] }
  )
})
​
Auto-download incoming media
Stream straight to disk — never buffer large files in memory:
client.on('message', async (event) => {
  if (!event.message?.imageMessage) return
  const file = `./media/${Date.now()}.jpg`
  await client.message.downloadToFile(event, file)
  console.log('saved', file)
})
See Media › Downloading incoming media for video/audio/documents and maxBytes.
​
Welcome new group members
The group event fires on membership changes. Greet everyone added (action: 'add') and @-mention them:
client.on('group', async (event) => {
  if (event.action !== 'add' || !event.groupJid || !event.participants?.length) return

  const jids = event.participants.map((p) => p.jid).filter((j): j is string => Boolean(j))
  const mentions = jids.map((j) => `@${j.split('@')[0]}`).join(' ')

  await client.message.send(
    event.groupJid,
    { type: 'text', text: `Welcome ${mentions}! 🎉` },
    { mentions: jids }
  )
})
​
Send a poll
await client.message.send(chatJid, {
  type: 'poll',
  name: 'Lunch?',
  options: ['Pizza', 'Sushi', 'Salad'],
  selectableCount: 1
})
​
React to a message
client.on('message', async (event) => {
  if (event.key.fromMe) return
  // Pass the event verbatim — its key is read for you.
  await client.message.send(event.key.remoteJid, {
    type: 'reaction',
    emoji: '❤️',
    target: event
  })
})
​
Keep the bot alive across drops
zapo doesn’t auto-reconnect — wire the connection event to a backoff loop. The full pattern (including when not to reconnect) is in Reconnection and Errors & disconnects.
Every snippet uses the content union — the same shapes client.message.send accepts everywhere. See Sending messages and the message types reference for the full set.


Voice calls (VoIP)


Place and answer WhatsApp voice calls with @zapo-js/voip — a plugin that exposes a coordinator at client.voip, decodes inbound PCM, and lets you feed live outbound audio with backpressure.

@zapo-js/voip is the official voice-calling plugin for zapo-js. It rides on the plugin system, exposes a WaVoipCoordinator at client.voip, and emits voip_* events on the host client.
Only audio media is implemented. You may flag a call as video in the signaling (see CallOfferOptions.isVideo), but no video media is encoded or transported.
​
Install
npm install @zapo-js/voip @roamhq/wrtc libmlow-wasm
The package has three peer dependencies, all required at runtime:
@roamhq/wrtc (>= 0.10.0) — provides SCTP for the relay transport.
libmlow-wasm (^0.1.1) — WhatsApp’s Opus profile, compiled to WebAssembly. No native build step.
zapo-js (^1.0.0) — the host client.
Some methods also need an ffmpeg binary on PATH:
loadAudio decodes the audio file via ffmpeg.
Streaming audio with feedLiveAudio from a file likewise needs ffmpeg to produce 16 kHz mono Float32Array chunks.
​
Wire the plugin
Add voipPlugin() to WaClientOptions.plugins. The coordinator appears at client.voip and the voip_* events become available on client.on.
import { WaClient } from 'zapo-js'
import { voipPlugin } from '@zapo-js/voip'

const client = new WaClient({
  store,
  sessionId: 'default',
  plugins: [voipPlugin()]
}, logger)

await client.connect()

client.on('voip_call_incoming', (call) => {
  console.log('incoming', call.callId, 'from', call.peerJid)
})
​
Plugin options
voipPlugin({ maxConcurrentCalls: 2, logLevel: 'warn' })
​
maxConcurrentCalls
numberdefault:"1"
Maximum simultaneous non-ended calls (ringing, connecting, or active). Increase to enable parallel multi-call. Outgoing startCall rejects, and incoming calls arrive with acceptBlocked: true, when the limit is hit.
​
logLevel
LogLevel
Minimum log level for the (chatty) VoIP plugin. Defaults to the host client’s level; set it to cap diagnostics independently — for example 'warn' to keep VoIP noise out of a trace host logger.
​
The coordinator surface
​
Placing a call
const callId = await client.voip.startCall({
  peerJid: '5511999999999@s.whatsapp.net'
})
​
peerJid
stringrequired
Bare or device JID to call.
​
isVideo
booleandefault:"false"
Flag the call as video in the signaling stanzas. Video media encode/transport is not implemented; only audio media flows.
​
audioFile
string
Audio file to preload and play once the call connects (needs ffmpeg). Equivalent to calling loadAudio after the call goes active.
​
peerDevices
string[]
Explicit peer device JIDs to ring; omit to resolve them automatically.
startCall resolves with the new call id once the offer is sent. Progress then arrives via voip_call_state. It rejects when you’re at the maxConcurrentCalls limit or if the offer fails to send.
​
Answering and ending
await client.voip.acceptCall(callId)
await client.voip.rejectCall(callId)          // EndCallReason.Declined
await client.voip.endCall(callId)             // EndCallReason.UserEnded
await client.voip.endCall(callId, EndCallReason.Busy)
acceptCall(callId) — accept a ringing incoming call. Throws if callId is unknown or not in an acceptable state.
rejectCall(callId, reason?) — reject a ringing incoming call (default reason Declined). Sends the reject stanza, then tears the call down.
endCall(callId, reason?) — end an active or connecting call (default reason UserEnded). No-op if the call is unknown or already ended.
​
Preloaded outbound audio
await client.voip.loadAudio(callId, './hello.mp3')
Decodes audioPath via ffmpeg and queues it as the outbound audio, played once the call is active. Throws if the file is missing or ffmpeg is unavailable.
For an unbounded or live source (TTS stream, ongoing capture) use external audio mode instead — see below.
​
Mute
client.voip.setMute(callId, true)   // mute local outbound audio
client.voip.setMute(callId, false)  // unmute
​
Live outbound audio (external mode)
For sources you can’t preload — streaming TTS, real-time capture — switch the call into external audio mode and pump samples in as they arrive. The plugin buffers them through a bounded jitter buffer and watermarks the buffer so a respectful producer never loses audio.
client.voip.setExternalAudioMode(callId, true)

const { pauseMs, resumeMs } = client.voip.getFeedWatermarksMs()

// later, repeatedly:
const bufferedMs = client.voip.feedLiveAudio(callId, samples) // Float32Array, 16 kHz mono
if (bufferedMs >= pauseMs) {
  // back off until getLiveBufferMs(callId) <= resumeMs
}
Method	Returns	What it does
setExternalAudioMode(callId, enabled)	void	Switch outbound audio source between preloaded playback and the live feed. Disable to return to preloaded playback.
feedLiveAudio(callId, data)	number	Feed a chunk of mono PCM (Float32Array at the engine sample rate). Returns the audio currently buffered ahead of the sender in milliseconds, so a producer can pace itself. Returns 0 when no session exists. The buffer is bounded and drops the oldest samples on overflow.
getLiveBufferMs(callId)	number	Milliseconds of live audio currently buffered ahead of the sender. 0 when no session exists or external mode is off.
getFeedWatermarksMs()	{ pauseMs, resumeMs }	Backpressure watermarks for the live feed, in milliseconds. Constants of the feed contract, independent of any specific call.
The watermark contract: pause feeding once getLiveBufferMs(callId) >= pauseMs, resume once it drains back to resumeMs. pauseMs stays below the engine’s internal drop threshold, so a producer that respects it never loses audio. If you ignore it, oldest samples are dropped on overflow.
​
Snapshots
const call = client.voip.getCall(callId)   // CallInfo | null
const all  = client.voip.getCalls()        // readonly CallInfo[]
CallInfo captures everything you need to render UI: callId, peerJid, direction, mediaType, createdAt, callerPn, plus a stateData object (state, audioMuted, videoOff, connectedAt, endReason, durationSecs, acceptBlocked, …) and derived getters isInitiator, isActive, isRinging, isEnded, canAccept, canReject, isAcceptBlocked.
​
Events
Add voipPlugin() to plugins and these events become available on client.on. Without the plugin they don’t exist on the type — see Plugins.
Event	Payload	Description
voip_call_state	CallInfo	The call’s state transitioned (ringing, connecting, active, on_hold, ended, …).
voip_call_incoming	CallInfo	A new incoming call.
voip_call_ended	CallInfo	The call ended (any reason).
voip_call_inbound_audio	{ call, pcm }	Decoded peer audio. pcm is 16 kHz mono Float32Array.
voip_call_outbound_audio_finished	CallInfo	The preloaded outbound audio finished sending.
voip_call_error	Error	A non-fatal call error (logging hook).
​
Enums
All enum values are lowercase strings — use the enum constants to avoid typos.
​
CallState
Constant	String value
CallState.Initiating	'initiating'
CallState.Ringing	'ringing'
CallState.IncomingRinging	'incoming_ringing'
CallState.Connecting	'connecting'
CallState.Active	'active'
CallState.OnHold	'on_hold'
CallState.Ended	'ended'
​
CallDirection
Constant	String value
CallDirection.Outgoing	'outgoing'
CallDirection.Incoming	'incoming'
​
CallMediaType
Constant	String value
CallMediaType.Audio	'audio'
CallMediaType.Video	'video'
​
EndCallReason
Constant	String value
EndCallReason.UserEnded	'user_ended'
EndCallReason.Declined	'declined'
EndCallReason.Timeout	'timeout'
EndCallReason.Busy	'busy'
EndCallReason.Cancelled	'cancelled'
EndCallReason.Failed	'failed'
EndCallReason.DoNotDisturb	'do_not_disturb'
EndCallReason.Unknown	'unknown'
​
Audio format
Sample rate: 16 kHz.
Channels: mono.
Sample format: Float32Array in [-1.0, 1.0].
Codec on the wire: WhatsApp’s Opus profile, encoded via libmlow-wasm. The RTP payload type is 120 (PayloadType.WhatsAppOpus).
Both inbound (voip_call_inbound_audio) and outbound (feedLiveAudio) are this same Float32Array shape — keep your buffers in this format and you can route them straight through.
​
Worked example
A minimal incoming auto-accept that records inbound PCM into an in-memory buffer:
import { WaClient, createPinoLogger } from 'zapo-js'
import { CallState, voipPlugin } from '@zapo-js/voip'

const logger = await createPinoLogger({ level: 'info' })

const client = new WaClient({
  store,
  sessionId: 'default',
  plugins: [voipPlugin({ maxConcurrentCalls: 1 })]
}, logger)

const recordings = new Map<string, Float32Array[]>()

client.on('voip_call_incoming', async (call) => {
  if (!call.canAccept) return
  await client.voip.acceptCall(call.callId)
  recordings.set(call.callId, [])
})

client.on('voip_call_inbound_audio', ({ call, pcm }) => {
  // pcm is 16 kHz mono Float32Array
  const buffer = recordings.get(call.callId)
  if (buffer) buffer.push(pcm.slice())
})

client.on('voip_call_state', (call) => {
  if (call.stateData.state === CallState.Active) {
    console.log(`call ${call.callId} active`)
  }
})

client.on('voip_call_ended', (call) => {
  const chunks = recordings.get(call.callId) ?? []
  const total = chunks.reduce((n, c) => n + c.length, 0)
  console.log(`call ${call.callId} ended; recorded ${total} samples`)
  recordings.delete(call.callId)
})

await client.connect()
For a fuller example — placing outgoing calls, preloaded vs streamed playback, hangup-after-audio, WAV recording — see examples/voip-example.ts in the zapo source tree.
​

WAM telemetry (analytics parity)

Emit the client-side w:stats telemetry batches a real WhatsApp Web tab sends, for wire parity and anti-fingerprinting — via the @zapo-js/wam plugin.

@zapo-js/wam is an optional plugin that sends the client-side WAM (w:stats) telemetry batches a real WhatsApp Web client sends after login. This is a parity improvement, not observability for your own code: WAM is WhatsApp’s internal analytics stream, and mirroring its shape reduces the chance a headless session stands out compared to a real Web tab.
You do not need this plugin to send or receive messages, place calls, or use any coordinator. Install it only when you want the session’s wire footprint to include the analytics WA Web tabs emit — for anti-fingerprinting purposes.
​
Install
npm install @zapo-js/wam
The only peer dependency is zapo-js (^1.5.0). The WAM event registry and wire encoder (@vinikjkkj/wa-wam) is pinned as a regular dependencies entry on the plugin, so it comes down transitively — no need to install it explicitly.
​
Wire the plugin
Add wamPlugin() to WaClientOptions.plugins. A WaWamCoordinator appears at client.wam and — by default — a full telemetry pipeline runs in the background.
import { WaClient } from 'zapo-js'
import { wamPlugin } from '@zapo-js/wam'

const client = new WaClient({
  store,
  sessionId: 'default',
  plugins: [wamPlugin()]
}, logger)

await client.connect()
That’s the whole integration in the common case. The plugin observes the host client, commits protocol events as they happen, and fabricates plausible UI telemetry on the side.
​
Plugin options
Every option is optional; each falls back to a value that matches (or closely tracks) what a real WA Web client uses.
​
logLevel
LogLevel
Minimum log level for the WAM plugin. Defaults to the host client’s.
​
flushIntervalMs
numberdefault:"5000"
Coalesce window before a non-empty buffer flushes. Matches WA_WAM_BUFFER_CONSTANTS.inMemoryBufferingDurationSecs when available.
​
maxBufferSize
numberdefault:"50000"
Byte size that forces an immediate flush of a channel’s batch, ahead of the interval. Matches WA_WAM_BUFFER_CONSTANTS.maxBufferSize when available.
​
appVersion
string
Overrides the advertised app version. Defaults to the bundled WA_VERSION.
​
serviceImprovementOptOut
booleandefault:"false"
The service_improvement_opt_out consent bit stamped on outgoing batches. Leave at false for parity with a default WA Web tab.
​
autoEmit
booleandefault:"true"
When true, the plugin observes the host client (frames, connection events, sends) and commits the matching protocol events itself. Set to false to drive client.wam.commit(...) entirely on your own.
​
syntheticUi
boolean | WaWamSyntheticUiOptionsdefault:"true"
Fabricates plausible UI (UiAction) telemetry — chat opens, image opens, audio media loads, info-drawer opens, attachment-tray sends, ambient re-opens, and periodic MemoryStat samples — so the event profile mimics a human WA Web session. Everything is jittered and rate-limited; a badly-timed fabrication is a worse tell than none. Pass false to disable, or an options object (WaWamSyntheticUiOptions) to tune per-event probabilities (chatOpenProbability, imageOpenProbability, audioLoadProbability, infoOpenProbability, attachmentTrayProbability, aboutConsumptionProbability), ambient / memory-sample intervals, an active-hour window, and per-surface capability gates (channels, communities, business — all default false; enable only when the session genuinely has that surface, since firing a channel/community/business event on an account that lacks it is itself a tell).
​
Events emitted
The plugin covers 131 of the registry’s 426 events (~31%). The rest is data a headless client cannot produce or plausibly fake — browser/runtime internals, device and OS state, mobile-app-only flows, crypto internals, ads, server-side aggregates — plus events carried on the private (non-w:stats) channels.
The 131 come from two independently toggled sources:
Source	Flag	Default	Count
Protocol lifecycle	autoEmit	on	19
Integrator actions	autoEmit	on	18
Synthetic UI	syntheticUi	on	94
Protocol lifecycle (19)

Integrator actions (18)

Synthetic UI (94)

Two disciplines keep the emitted set a faithful fingerprint. Auto-emitted protocol and integrator events fire only when every field is honestly derivable from real client activity. Synthetic UI events are fabricated but each replicates WA Web’s actual emit — verified against the deobfuscated web bundle — jittered, rate-limited, and confined to configurable active hours.
​
client.wam
The coordinator exposes three methods.
Method	Signature	Description
commit	<K>(name: K, payload?: WaWamEventArgs<K>) => void	Commit one WAM event into the current channel’s batch. The event is dropped by the same Math.random() * weight > 1 sampling gate WA applies (and dropped silently if the event name is unknown).
flush	() => Promise<void>	Flush every open batch immediately.
dispose	() => Promise<void>	Stop accepting new events, tear down autoEmit and syntheticUi, and flush what’s left. Called automatically when the plugin disposes on disconnect().
// Commit an event manually (autoEmit does most of these for you)
client.wam.commit('UiAction', { uiActionType: 'CHAT_OPEN' })
Batches only upload while the client is connected and the account has finished registering (credentials.meJid populated). A batch produced while offline, or during the pre-login window on a fresh pair, is dropped on flush — the plugin logs a trace and moves on. Once the session is fully logged in, future batches upload normally.
​
What the pipeline does
autoEmit subscribes to the host client’s protocol surface and commits the corresponding WAM events as they occur — this is what makes the “install it and forget” default work.
syntheticUi watches the same surface and fabricates human-scale UI events on the side (chat opens on inbound messages, occasional info-drawer opens, ambient re-opens between activity spikes). Time-of-day and per-event probabilities are tunable; capability gates default to off, so nothing channel/community/business-specific fires unless you turn it on.
Both are on by default. Set either to false to opt out.
​
Worked example
Install the plugin and let the defaults do everything — this is the intended common case:
import { WaClient } from 'zapo-js'
import { wamPlugin } from '@zapo-js/wam'

const client = new WaClient({
  store,
  sessionId: 'main',
  plugins: [wamPlugin()]
}, logger)

await client.connect()
// telemetry batches now flush every ~5s while connected
For a headless bot that never opens chats, disable the synthetic UI so the profile matches the real behavior (a bot that hammers send without ever “opening” a chat is itself a signal):
plugins: [wamPlugin({ syntheticUi: false })]
​

WaClient & coordinators

Complete method reference for WaClient and every coordinator: auth, message, presence, chat, group, newsletter, profile, and more.

This page lists every public method on WaClient and its coordinators, with code-grounded descriptions. Dedicated pages go deeper for message types, chat mutations, and the low-level API.
​
WaClient
new WaClient(options: WaClientOptions, logger?: Logger)
See Configuration for every option.
Member	Signature	Description
connect	() => Promise<void>	Opens the socket and runs the Noise handshake; drives pairing on first run. Resolves once connected.
disconnect	() => Promise<void>	Flushes pending write-behind and closes the socket, keeping credentials.
logout	(reason?: WaLogoutReason) => Promise<void>	Unlinks this companion device server-side; then clears stored state per logoutStoreClear. Throws if not authenticated.
getState	() => WaAuthState	Current auth/connection state.
getCredentials	() => WaAuthCredentials | null	Current credentials, if paired.
getClockSkewMs	() => number | null	Estimated server clock skew (from keep-alive), or null.
ignoreKey	(input: WaIgnoreKey | WaIgnoreKeyPredicate) => () => void	Drop matching inbound stanzas before any handler runs — descriptor or predicate (see Ignoring inbound stanzas). Returns an unregister function.
on / once / off	(event, listener) => this	Typed event emitter over WaClientEventMap.
​
Ignoring inbound stanzas
client.ignoreKey(input) drops matching <message>, <receipt>, <notification>, <presence>, <chatstate>, and <call> stanzas before any handler — including the persistence and decryption paths — sees them. The coordinator still sends the appropriate ack so the server stops re-delivering.
It accepts two shapes:
Declarative descriptor — WaIgnoreKey
import type { WaIgnoreKey } from 'zapo-js'

// Drop everything from one peer
const off = client.ignoreKey({ remoteJid: spammerJid })

// Drop only that peer's messages (keep receipts / presence)
client.ignoreKey({ remoteJid: spammerJid, only: ['message'] })

// Drop your own outbound message echoes (multi-device fan-in)
client.ignoreKey({ fromMe: true, only: ['message'] })

off() // unregister
Field	Type	Notes
remoteJid	string | readonly string[]	The chat JID. Array entries OR. Also matches the alt sender_pn / sender_lid / participant_pn / participant_lid attrs (and sender_lid on <call>), so one JID form catches the other.
fromMe	boolean	Whether the stanza was sent by this account.
id	string	Stanza id.
participant	string	The author in groups / broadcasts. Also matches the alt forms.
only	readonly ('message' | 'receipt' | 'notification' | 'presence' | 'chatstate' | 'call')[]	Restrict to specific tags. Default: all six.
Multiple top-level fields AND together; the remoteJid array ORs. At least one of remoteJid / fromMe / id / participant is required — empty descriptors and empty arrays throw.
​
Predicate — WaIgnoreKeyPredicate
For anything the descriptor can’t express (chat-kind filters, custom rules combining several fields), pass a predicate (ctx: WaIgnoreKeyContext) => boolean. Return true to drop. The lib has already parsed the wire-format node before calling you — no BinaryNode digging required.
import { isGroupJid, isStatusBroadcastJid, isNewsletterJid } from 'zapo-js'

// Drop every group message; keep group receipts/notifications
client.ignoreKey((m) => m.kind === 'message' && isGroupJid(m.remoteJid ?? ''))

// Drop status broadcasts entirely
client.ignoreKey((m) => isStatusBroadcastJid(m.remoteJid ?? ''))

// 1:1 only — drop anything from a group, broadcast, or newsletter
client.ignoreKey((m) => {
  const j = m.remoteJid ?? ''
  return isGroupJid(j) || isStatusBroadcastJid(j) || isNewsletterJid(j)
})
WaIgnoreKeyContext carries the same fields the descriptor matches on, already parsed:
Field	Type	Notes
kind	'message' | 'receipt' | 'notification' | 'presence' | 'chatstate' | 'call'	Stanza tag.
remoteJid	string | null	Deviceless chat JID — group JID for groups, PN or LID user-form JID for 1:1 (the :device segment is stripped to match event.key.remoteJid). Userless from values (e.g. s.whatsapp.net) are kept verbatim.
fromMe	boolean	Already resolved against the account’s meJid.
id	string | undefined	Stanza id.
participant	string | null	The author in groups / broadcasts. Same device-stripping as remoteJid; null for non-group stanzas.
The predicate sees the device-stripped from / participant — but no PN ↔ LID alt-attr resolution. The descriptor form is what handles the PN ↔ LID part automatically; if you need both per-JID matching with alt-form coverage and a custom rule, prefer the descriptor’s remoteJid with only.
Stream-control nodes and the connection-critical success / failure tags bypass filters so the auth flow stays intact.
​
Coordinator getters
Getter	Type	Section
auth	WaAuthClient	auth
message	WaMessageCoordinator	message
presence	WaPresenceCoordinator	presence
chat	WaAppStateMutationCoordinator	chat
group	WaGroupCoordinator	group
status	WaStatusCoordinator	status
broadcastList	WaBroadcastListCoordinator	broadcastList
newsletter	WaNewsletterCoordinator	newsletter
privacy	WaPrivacyCoordinator	privacy
profile	WaProfileCoordinator	profile
business	WaBusinessCoordinator	business
bot	WaBotCoordinator	bot
email	WaEmailCoordinator	email
mobile	WaMobileCoordinator	mobile
lowlevel	WaLowLevelCoordinator	low-level API

auth
client.auth (WaAuthClient). Pairing is mostly event-driven (Authentication); these are the user-facing entry points.
Method	Signature	Description
requestPairingCode	(phoneNumber, shouldShowPushNotification?, customCode?) => Promise<string>	Requests an 8-char pairing code (link-code flow). The client must already be connected — call after the auth_pairing_required event. customCode suggests a code; the server may return a different one.
fetchPairingCountryCodeIso	() => Promise<string>	The ISO country code the server resolved for the account.
getState	(connected?) => { connected, registered, hasQr, hasPairingCode }	Auth readiness flags.
getCurrentCredentials	() => WaAuthCredentials | null	Loaded credentials, or null.

message
WaMessageCoordinator — see Sending & Receiving.
Method	Signature	Description
send	(to, content: WaSendMessageContent, options?: WaSendMessageOptions) => Promise<WaMessagePublishResult>	Sends any content type; handles device fanout and per-send retries. Returns the stanza id + ack metadata.
sendReceipt	(event|events, options?) / (jid, ids, options?) => Promise<void>	Sends a delivery/read/played/inactive receipt. Delivery is auto-acked on decrypt; use this for manual read/played.
requestHistorySync	(input: WaRequestHistorySyncInput) => Promise<{ messageId }>	Asks the server to backfill older messages for a chat. Resolves once dispatched — the backlog arrives later as history_sync_chunk. See Requesting older history.
shareGroupHistory	(groupJid, input: WaShareGroupHistoryInput) => Promise<WaShareGroupHistoryResult>	Shares recent group history with newly added members. Fans the encrypted bundle out only to input.toJids plus this account, then sends a group-wide notice. Gated by the group_history_send AB prop — throws when the account is not allowed to share. See Groups → sharing group history.
upload	(source: WaUploadMediaSource, options: WaUploadMediaOptions) => Promise<WaMediaUploadResult>	Encrypts and uploads standalone media to the WhatsApp CDN and returns the reusable descriptor (url, directPath, mediaKey, hashes, sidecars, mediaKeyTimestamp, mimetype) without sending a message. Spread the result onto the matching proto to send it. See Pre-upload and reuse.
download	(source, options?) => Promise<Readable>	Streams decrypted media (MAC + SHA-256 verified as consumed). Cancel via options.signal.
downloadToFile	(source, filePath, options?) => Promise<void>	Streams decrypted media to a file.
downloadBytes	(source, options?) => Promise<Uint8Array>	Buffers decrypted media into memory — small media only; cap with options.maxBytes.
requestMediaReupload	(event|request, options?) => Promise<WaMediaRetryResult>	Asks the sender’s primary device to re-serve media whose CDN blob returned 404/410 — typically old history-synced media. Encrypts a server-error receipt with the message’s media key and awaits the mediaretry notification the primary answers with; on success, only directPath changes (media key, hashes, and length stay valid). Newsletter messages and messages without downloadable media are rejected. Does not throw on not_found / general_error — check result.result. See Media → requesting a reupload.
tryDecryptAddon	(event) => Promise<void>	Decrypts an addon (poll vote, reaction, …) and emits message_addon. Auto-called by default; opt out with addons: { autoDecrypt: false } to call it yourself.
syncSignalSession	(jid, reasonIdentity?) => Promise<void>	Force-refreshes the Signal session(s) for a JID; reasonIdentity also reissues the trusted-contact token.
getReachoutTimelock	() => Promise<WaReachoutTimelock>	Server-side timelock that throttles cold outreach to non-contacts.
getNewChatMessageCapping	(type?) => Promise<WaMessageCappingInfo>	Per-cycle message quota applied to new-chat threads (quota, used, cycle, status).
source is a WaIncomingMessageEvent or a raw Proto.IMessage.


presence
WaPresenceCoordinator — see Presence & status.
Method	Signature	Description
send	(type?: 'available' | 'unavailable') => Promise<void>	Broadcasts your online/offline presence.
sendChatstate	(jid, options) => Promise<void>	Sends a typing/recording/paused hint into a chat.
subscribe	(jid, options?) => Promise<void>	Subscribes to a contact’s presence/chat-state. Per-jid and per-connection — re-subscribe after reconnect.
​
chat
WaAppStateMutationCoordinator — full reference (incl. the generic set/remove and all schemas) in Chat mutations.
Method	Signature
setChatMute	(chatJid, muted, muteEndTimestampMs?) => Promise<void>
setChatPin / setChatArchive / setChatRead / setChatLock	(chatJid, boolean) => Promise<void>
setMessageStar	(message, starred) => Promise<void>
clearChat / deleteChat	(chatJid, options?) => Promise<void>
deleteMessageForMe	(message, options?) => Promise<void>
setStatusPrivacy / setUserStatusMute	(input) / (jid, muted) => Promise<void>
setBroadcastList / removeBroadcastList	(input) / (id) => Promise<void>
set / remove	(input) => Promise<void>
sync / flushMutations	(options?) / () => Promise<…>
getBlockedCollections / emitEventsFromSyncResult	(syncResult) => …
Pin and archive are mutually exclusive (pinning clears archive and vice-versa); locking clears both. clearChat/deleteChat/deleteMessageForMe are local-only (your devices) — use a revoke to delete for everyone. A mute timer doesn’t auto-unmute client-side.

group
WaGroupCoordinator — see Groups & communities. Participant ops return one WaParticipantActionResult per jid — the IQ succeeds as a whole even when some entries fail, so check each result’s status / code.
Method	Signature	Description
queryGroupMetadata	(groupJid) => Promise<WaGroupMetadata>	Full group metadata.
queryAllGroups	() => Promise<readonly WaGroupMetadata[]>	Every group the account is in.
queryGroupInviteInfo	(code) => Promise<WaGroupInviteInfo>	Preview an invite code (subject, size, ephemeral, description, trimmed participant sample).
createGroup	(subject, participants, options?) => Promise<WaGroupMetadata>	Create a group (you’re auto-added as admin; don’t include your own JID). Returns the new group’s metadata.
setSubject	(groupJid, subject) => Promise<void>	Rename.
setDescription	(groupJid, description|null, prevDescId?) => Promise<void>	Set/clear description.
setSetting	(groupJid, setting, enabled) => Promise<void>	Toggle a boolean group flag (announcement, restrict, ephemeral, group_history, allow_admin_reports, no_frequently_forwarded, community flags).
setMemberAddMode	(groupJid, 'admin_add' | 'all_member_add') => Promise<void>	Restrict member adds to admins (or open to all). Admin op.
setMemberLinkMode	(groupJid, 'admin_link' | 'all_member_link') => Promise<void>	Restrict invite-link sharing to admins (or open to all). Admin op.
setMemberShareGroupHistoryMode	(groupJid, 'admin_share' | 'all_member_share') => Promise<void>	Hide or expose prior chat history to newly added members. Admin op.
setEphemeralDuration	(groupJid, expirationSeconds, trigger?) => Promise<void>	Turn on disappearing messages with a specific lifetime (86400 = 24h, 604800 = 7d, 7776000 = 90d). Use setSetting('ephemeral', false) to disable. Admin op.
addParticipants / removeParticipants	(groupJid, jids) => Promise<readonly WaParticipantActionResult[]>	Add / remove members. One result per jid; inspect status / code for partial failures.
promoteParticipants / demoteParticipants	(groupJid, jids) => Promise<readonly WaParticipantActionResult[]>	Grant / revoke admin. Same per-jid result shape.
leaveGroup	(groupJids) => Promise<void>	Leave one or more groups (batched).
queryInviteCode	(groupJid) => Promise<string>	Fetch the current invite code (the path segment of chat.whatsapp.com/<code>) without rotating it. Admin op — non-admins get 403 not-authorized.
revokeInvite	(groupJid) => Promise<WaRevokeInviteResult>	Rotate the invite code — every old chat.whatsapp.com/<code> link stops working. Returns the new code plus any affectedParticipants.
joinGroupViaInvite	(code) => Promise<WaGroupMetadata>	Join via code. Throws if expired/revoked/full/already a member. Returns the joined group’s metadata.
createCommunity	(subject, options?) => Promise<WaGroupMetadata>	Create a community (request-required unless membershipApprovalMode: 'open').
deactivateCommunity	(communityJid) => Promise<void>	Delete a community.
linkSubGroups / unlinkSubGroups	(communityJid, jids, options?) => Promise<…>	Link / unlink sub-groups (removeOrphanedMembers evicts orphaned members).
queryLinkedGroupsParticipants	(communityJid) => Promise<readonly WaGroupParticipant[]>	Merged participants across a community.
fetchSubGroups	(communityJid) => Promise<WaCommunitySubGroupsResult>	List sub-groups (MEX).
joinLinkedGroup	(communityJid, subGroupJid, options?) => Promise<void>	Join a linked sub-group. Call queryGroupMetadata afterward for the full metadata.
queryMembershipApprovalRequests	(groupJid) => Promise<readonly WaMembershipRequest[]>	Pending join requests.
approveMembershipRequests / rejectMembershipRequests	(groupJid, jids) => Promise<void>	Approve / reject requests.
cancelMembershipRequests	(groupJid, jids) => Promise<void>	Cancel your own pending requests.
isInternalGroup	(groupJid) => Promise<boolean>	true for internal WhatsApp groups (MEX).
transferCommunityOwnership	(communityJid, newOwnerJid) => Promise<void>	Hand off community ownership (MEX).
fetchSubgroupSuggestions	(communityJid, hintSubgroupJid) => Promise<readonly WaCommunitySubGroupSuggestion[]>	Suggested sub-groups (MEX).
submitGroupSuspensionAppeal	(groupJid, options?) => Promise<WaGroupSuspensionAppealResult>	Appeal a suspension (MEX).
Methods marked (MEX) require an active MEX transport and throw when it’s unavailable.
​
newsletter
WaNewsletterCoordinator — see Newsletters. Composed of discovery, admin, and messaging ops.
​
Discovery
Method	Signature	Description
fetch / fetchByInvite	(jid|code, options?) => Promise<WaNewsletterMetadata>	Metadata by JID or invite code.
fetchDehydrated	(keyOrInvite, options?) => Promise<WaNewsletterDehydratedMetadata>	Lightweight metadata (no image/followers).
listSubscribed	(options?) => Promise<readonly WaNewsletterMetadata[]>	Channels you follow.
searchDirectory	(options?) => Promise<WaNewsletterDirectoryResults>	Search the public directory.
fetchRecommended	(options?) => Promise<readonly WaNewsletterMetadata[]>	Recommended channels.
fetchSimilar	(jid, options?) => Promise<readonly WaNewsletterMetadata[]>	Channels similar to one.
fetchDirectoryList	(options) => Promise<WaNewsletterDirectoryResults>	Paged directory by country/category.
fetchDirectoryCategoriesPreview	(options) => Promise<readonly WaNewsletterDirectoryCategoryPreview[]>	Category carousel previews.
fetchIsDomainPreviewable	(domains) => Promise<ReadonlyMap<string, boolean>>	Which domains support link previews.

Admin
Method	Signature	Description
create	(input) => Promise<WaNewsletterMetadata>	Create a channel (auto-accepts creation TOS; picture uploaded inline — keep small).
update	(jid, input) => Promise<WaNewsletterMetadata>	Edit name/description/picture.
delete	(jid) => Promise<void>	Irreversible delete — followers detached, history dropped, JID burned.
fetchAdminInfo	(jid) => Promise<WaNewsletterAdminInfo>	Admin-only metadata view.
fetchAdminCapabilities	(jid) => Promise<ReadonlySet<WaNewsletterCapability>>	Capabilities granted to the account.
fetchFollowers	(jid, options?) => Promise<WaNewsletterFollowersPage>	Paged follower list.
fetchInsights	(jid, metrics) => Promise<… | null>	Admin analytics.
fetchReports	() => Promise<… | null>	Moderation reports against owned channels.
fetchPendingInvites	(jid) => Promise<readonly string[]>	Pending admin invite JIDs.
fetchEnforcements	(jid) => Promise<… | null>	Moderation enforcement state.
fetchPollVoters	(input) => Promise<ReadonlyMap<string, readonly WaNewsletterPollVoter[]>>	Poll voters grouped by option.
fetchMessageReactionSenders	(input) => Promise<readonly WaNewsletterReactionSenders[]>	Reaction senders grouped by emoji.
createAdminInvite	(input) => Promise<WaNewsletterAdminInviteResult>	Invite a user as admin.
acceptAdminInvite	(jid) => Promise<void>	Accept a pending admin invite (auto-accepts TOS).
revokeAdminInvite	(input) => Promise<void>	Revoke a sent admin invite.
changeOwner	(input) => Promise<void>	Transfer ownership to an invited admin.
demoteAdmin	(input) => Promise<void>	Demote an admin to follower.
queryTosState / acceptTos	(noticeIds) => Promise<…>	Query / accept TOS notices.
logExposures	(exposures) => Promise<void>	Report capability exposures (telemetry).

Messaging
Method	Signature	Description
send	(jid, content, options?) => Promise<WaNewsletterSendResult>	Publish a message (any content type).
editMessage	(jid, parentMessageId, content) => Promise<WaNewsletterSendResult>	Edit a published message.
react / revoke / votePoll / sendViewReceipt	(input) => Promise<{ stanzaId }>	React / revoke / vote / view-receipt.
fetchMessages / fetchMessageUpdates	(input) => Promise<BinaryNode>	Page messages / fetch edits-reactions-votes in a range.
subscribeLiveUpdates	(jid) => Promise<{ durationSeconds }>	Subscribe to live updates (re-subscribe after reconnect).
follow / unfollow	(jid) => Promise<void>	Follow / unfollow.
mute	(input) => Promise<void>	Mute / unmute.

privacy
WaPrivacyCoordinator — see Privacy.
Method	Signature	Description
getPrivacySettings	() => Promise<WaPrivacySettings>	Current value of every privacy category. A change made on another device also surfaces on the privacy event, so polling this is only needed for the initial read.
setPrivacySetting	(setting, value) => Promise<string | null>	Update one category. Returns the dhash the server echoes back — the version stamp of the category’s disallowed list, present only while the category sits on 'contact_blacklist' (null otherwise). A 'contact_blacklist' value flips the mode only; populate the list with setDisallowedList.
setDisallowedList	(category: WaPrivacyDisallowedListSettingName, input: { add?: readonly string[], remove?: readonly string[] }) => Promise<string | null>	Adds/removes JIDs on a category’s disallowed list, switching the category to 'contact_blacklist' in the same stanza. Resolves a stale dhash (409) by refetching and retrying once; returns the new dhash.
getDisallowedList	(category) => Promise<WaPrivacyDisallowedListResult>	Per-category excluded JIDs. Returns an empty list when the category is not on 'contact_blacklist'.
refreshFromAccountSync	() => Promise<WaPrivacyAccountSyncResult>	Refetch everything an account-sync update covers — the full category set plus the disallowed lists of WA_PRIVACY_ACCOUNT_SYNC_DISALLOWED_LISTS. The result is emitted as the privacy event too. Concurrent calls are deduplicated.
getBlocklist	() => Promise<WaBlocklistResult>	Account-wide blocklist. Changes made on another device also surface on the blocklist event.
blockUser / unblockUser	(jid) => Promise<void>	Block / unblock. A block stops the peer messaging/calling you and hides your last-seen/online/photo/status from them.

profile
WaProfileCoordinator — see Profile.
Method	Signature	Description
getProfilePicture	(jid, type?: 'preview' | 'image', existingId?) => Promise<WaProfilePictureResult>	Picture envelope ({ url?, directPath?, id?, type? }). type defaults to 'preview' (compact variant); 'image' returns the high-resolution original. Pass the cached existingId to let the server short-circuit when the picture hasn’t changed.
setProfilePicture	(imageBytes, targetJid?) => Promise<string | null>	Set your/a target’s picture. imageBytes is uploaded as-is — pre-encode square JPEG. Returns the picture id.
deleteProfilePicture	(targetJid?) => Promise<void>	Remove the picture (admin op for groups).
getStatus / setStatus	(jid) / (text) => Promise<…>	Get/set the legacy “About”.
setPushName	(name) => Promise<void>	Update the display name broadcast to peers. Applied to the local credentials immediately and routed through an app-state mutation; peers see it on your next outgoing message. Empty string resets to the device default.
getProfiles	(jids) => Promise<readonly WaProfileInfo[]>	Batched picture id + status.
getDisappearingMode	(jids) => Promise<readonly WaDisappearingModeResult[]>	Batched disappearing-mode setting.
setDisappearingMode	(durationSeconds) => Promise<void>	Set the account-wide default disappearing-mode duration for new 1:1 chats (0/86400/604800/7776000). Existing chats keep their setting.
getTextStatuses	(jids) => Promise<readonly WaTextStatusResult[]>	Batched modern text status (emoji + text).
setTextStatus	(input) => Promise<void>	Set your modern text status; text: null/'' clears it.
getUsernames	(jids) => Promise<readonly WaUsernameResult[]>	Batched username lookup.
resolveUsername	(input: WaResolveUsernameInput) => Promise<WaUsernameLookupResult>	Resolve a handle to its LID via usync. A leading @ and a :1234 key suffix in input.username are accepted; input.usernameKey overrides a key parsed out of the handle. Returns a discriminated union: { status: 'found', jid, username, isBusiness, pnJid }, { status: 'key-required', username } (retry with usernameKey), or { status: 'not-found' }. Throws on local-validation failure before any round-trip.
getOwnUsername	() => Promise<WaOwnUsernameResult>	Your username record (value, state, recovery pin).
setUsername	(input) => Promise<boolean>	Reserve a username. Throws on local-validation failure before any round-trip. Returns true only on SUCCESS; otherwise false (taken/rate-limited) without throwing.
deleteUsername	() => Promise<boolean>	Delete your username.
checkUsernameAvailability	(username) => Promise<WaUsernameAvailabilityResult>	Availability + suggestions.
setUsernameKey	(pin) => Promise<boolean>	Set the username recovery PIN. Throws before any round-trip when pin is not exactly 4 digits.
getAboutStatus	(jid) => Promise<string | null>	”About” text via MEX.
getLidsByPhoneNumbers	(phoneNumbers) => Promise<readonly SignalLidSyncResult[]>	Resolve LIDs for phone numbers.

status
WaStatusCoordinator — see Status broadcasts.
Method	Signature	Description
send	(input: WaSendStatusInput) => Promise<WaMessagePublishResult>	Publish a status to recipients.
revokeStatus	(input) => Promise<WaMessagePublishResult>	Revoke a published status.
setPrivacy	(input) => Promise<void>	Account-wide status privacy.
setUserMuted	(jid, muted) => Promise<void>	Mute/unmute a contact’s status.

broadcastList
WaBroadcastListCoordinator.
Method	Signature	Description
setList	(input: WaSetBroadcastListInput) => Promise<void>	Create/update a list (name + recipients).
removeList	(id) => Promise<void>	Delete a list.
send	(input: WaSendBroadcastListMessageInput) => Promise<WaMessagePublishResult>	Send to every member.
Broadcast lists are business-only (backed by the BusinessBroadcastList app-state schema); regular accounts have the mutations rejected.

business
WaBusinessCoordinator — see Business.
Method	Signature	Description
getBusinessProfile	(jids) => Promise<readonly WaBusinessProfileResult[]>	Batched business profiles (about, address, hours). Works from any account.
getVerifiedName / getVerifiedNames	(jid) / (jids) => Promise<…>	Verified-name lookup (single / batched).
editBusinessProfile	(input) => Promise<void>	Edit your business profile. Business-only.
updateCoverPhoto	(media) => Promise<{ id }>	Upload/bind a cover photo. Business-only.
deleteCoverPhoto	(id) => Promise<void>	Delete the cover photo. Business-only.

bot
WaBotCoordinator — see Bots.
Method	Signature	Description
listBots	() => Promise<readonly WaBotInfo[]>	Bots available to the account, grouped by section.
getBotProfile	(jid, options?) => Promise<WaBotProfileResult | null>	A bot’s profile (commands, prompts, creator).
sendPrompt	(to, content, options?) => Promise<WaMessagePublishResult>	Prompt a bot — direct path (to is @bot) or mention path (group + options.botJid).
tryDecryptChunk	(event) => Promise<void>	Decrypt a streamed reply chunk → message_bot_chunk. Auto-called per incoming message.

email
WaEmailCoordinator.
Method	Signature	Description
getStatus	() => Promise<WaEmailStatus>	Current binding (address + verified/confirmed).
setEmail	(email, context?) => Promise<WaEmailStatus>	Bind/rebind an address.
requestVerificationCode	(input) => Promise<void>	Send a verification code to the address.
verifyCode	(code) => Promise<WaEmailVerifyCodeResult>	Submit the emailed code.
confirm	(context?) => Promise<void>	Post-verification ownership confirmation.
Email binding is mobile-only — every method throws on a Web/companion connection. See Mobile connections

mobile
WaMobileCoordinator — see Hosting companion devices. Requires a mobile-primary session; link/revoke/publish methods throw on a Web/companion connection.
Method	Signature	Description
linkCompanion	(qr) => Promise<LinkCompanionResult>	Link a companion by its QR string. Returns { deviceJid, keyIndex }.
linkCompanionByCode	(pairingCode) => Promise<LinkCompanionResult>	Link a companion by its 8-char pairing code (the companion must have requested a code for this account first).
revokeCompanion	(deviceJid, reason?) => Promise<void>	Unlink a hosted companion and republish the key-index list. reason defaults to 'user_initiated'.
revokeAllCompanions	(reason?, { excludeHostedCompanion? }?) => Promise<void>	”Log out all companion devices”; opt-in excludeHostedCompanion spares the companions this account itself hosts.
listCompanions	() => Promise<readonly CompanionRecord[]>	The companions this primary has linked in the current epoch.
reconcileCompanions	() => Promise<readonly string[]>	Reconcile the tracked set against the server (usync); runs automatically on connect and on account_sync. Returns the removed device jids.
publishKeyIndexList	() => Promise<void>	Re-sign and republish the key-index list for the current device set.

lowlevel
WaLowLevelCoordinator — full reference in Low-level API: sendNode, query, registerIncomingHandler, unregisterIncomingHandler, registerIncomingStanzaFilter.
For top-level helpers exported from the package root — message inspection (getContentType), target normalization (resolveMessageTarget), JID predicates and constants — see JIDs, helpers & constants. For typed business hours, see Profile, privacy & business.

Message types

Every send content variant in zapo — the typed builders discriminated by type, and the raw Proto.IMessage fields the library recognizes on receive.

Everything you send goes through client.message.send(to, content, options?). The content argument is a WaSendMessageContent:
type WaSendMessageContent =
  | string                        // shorthand for a text message
  | WaSendTextMessage             // type: 'text'
  | WaSendReactionMessage         // type: 'reaction'
  | WaSendRevokeMessage           // type: 'revoke'
  | WaSendPinMessage              // type: 'pin' | 'unpin'
  | WaSendKeepMessage             // type: 'keep' | 'unkeep'
  | WaSendPollMessage             // type: 'poll'
  | WaSendPollVoteMessage         // type: 'poll-vote'
  | WaSendEventMessage            // type: 'event'
  | WaSendEventResponseMessage    // type: 'event-response'
  | WaSendMediaMessage            // type: 'image' | 'video' | 'ptv' | 'audio' | 'document' | 'sticker' | 'sticker-pack'
  | Proto.IMessage                // raw protobuf — anything not covered above
There are two ways to send: a typed builder (an object with a type discriminator — the library validates and fills protocol fields for you) or a raw Proto.IMessage (you build the protobuf yourself). The same send() accepts both.
​
Shorthand
A plain string is sent as a text message:
await client.message.send(jid, 'Hello!')
​
Typed builders
Each builder is discriminated by its type field. Bold fields are required.
​
Text & media
type	Type	Required fields	Guide
text	WaSendTextMessage	text	Sending messages
image	WaSendMediaMessage	media (mimetype optional with a detectMimetype processor)	Media
video	WaSendMediaMessage	media (mimetype optional with a detectMimetype processor)	Media
ptv	WaSendMediaMessage	media (mimetype optional with a detectMimetype processor)	Media
audio	WaSendMediaMessage	media (mimetype optional with a detectMimetype processor)	Media
document	WaSendMediaMessage	media (mimetype optional with a detectMimetype processor)	Media
sticker	WaSendMediaMessage	media (mimetype defaults to image/webp)	Media
sticker-pack	WaSendStickerPackMessage	stickerPackId, name, publisher, stickers, trayIcon	Media
Media builders also accept any non-managed field of the underlying protobuf message (e.g. caption, gifPlayback, ptt, fileName) via the UserMediaFields mapping. Protocol-managed fields (url, mediaKey, fileSha256, directPath, …) are filled by the builder.
mimetype is optional. The resolution order is: an explicit mimetype you pass wins; otherwise the builder calls media.processor.detectMimetype (provided by @zapo-js/media-utils when file-type is installed); otherwise it throws for image/video/audio/document/ptv. Stickers default to image/webp.
​
Interactive
type	Type	Required fields	Guide
reaction	WaSendReactionMessage	emoji, target	Reactions
poll	WaSendPollMessage	name, options	Polls
poll-vote	WaSendPollVoteMessage	poll, selectedOptionNames	Voting
event	WaSendEventMessage	name, startTime	Events
event-response	WaSendEventResponseMessage	event, response	Event response
pin / unpin	WaSendPinMessage	target (+ optional durationSecs)	Pinning
keep / unkeep	WaSendKeepMessage	target	Keep-in-chat
revoke	WaSendRevokeMessage	target	Revoking
target is a WaMessageTargetInput — a WaMessageKey ({ remoteJid, id, fromMe, participant? }) or a received message event passed verbatim (its key is used). poll/event parents additionally require authorJid and the 32-byte messageSecret.
For revoke, sender-vs-admin is auto-detected from target.fromMe (false triggers an admin revoke). There is no subtype option.
​
Raw Proto.IMessage
For anything without a typed builder, pass a raw protobuf message. The library inspects the populated field and automatically resolves the stanza attributes — message type ([resolveMessageTypeAttr]), media type, polltype, event_type, view_once, and edit — so you only set the content field.
import { proto } from 'zapo-js'

await client.message.send(jid, {
  conversation: 'A raw text message'
})
​
Text
Field	Notes
conversation	Plain text.
extendedTextMessage	Text with context (links, mentions, replies). A non-empty matchedText makes it a link/media message.
​
Media
Field	Resolved media type
imageMessage	image
videoMessage	video (or gif when gifPlayback)
ptvMessage	ptv
audioMessage	audio (or ptt when ptt)
documentMessage / documentWithCaptionMessage	document
stickerMessage	sticker
stickerPackMessage	sticker-pack
Raw media fields require pre-uploaded media (the encryption keys, directPath, and digests must already be set). To upload from bytes/a file, use the typed media builders instead — they perform the upload for you.
​
Location & contacts
// Static location
await client.message.send(jid, {
  locationMessage: { degreesLatitude: -23.55, degreesLongitude: -46.63, name: 'HQ' }
})

// Live location (resolved as `live-location`)
await client.message.send(jid, {
  liveLocationMessage: { degreesLatitude: -23.55, degreesLongitude: -46.63 }
})

// Single contact (vCard)
await client.message.send(jid, {
  contactMessage: { displayName: 'Maria', vcard: 'BEGIN:VCARD\n...\nEND:VCARD' }
})

// Multiple contacts
await client.message.send(jid, {
  contactsArrayMessage: { displayName: 'Team', contacts: [/* IContactMessage[] */] }
})
Field	Resolved media type
locationMessage	location (or live-location when isLive)
liveLocationMessage	live-location
contactMessage	vcard
contactsArrayMessage	contact_array
​
Interactive & business
Field	Resolved type
buttonsMessage	button
buttonsResponseMessage	button_response
listMessage	list
listResponseMessage	list_response
interactiveMessage (native flow)	interactive
interactiveResponseMessage	native_flow_response
templateButtonReplyMessage	text
orderMessage	order
productMessage	product
groupInviteMessage	url
await client.message.send(jid, {
  groupInviteMessage: {
    groupJid, inviteCode, inviteExpiration, groupName
  }
})
​
Polls & events (raw)
Field	Resolved
pollCreationMessage / …V2 / …V3 / …V5	poll (polltype: creation)
pollUpdateMessage	poll (polltype: vote)
pollResultSnapshotMessage	text
eventMessage	event (event_type: creation)
encEventResponseMessage	event (event_type: response)
Poll creation and event messages auto-persist their messageSecret so later votes/responses can be encrypted.
​
Protocol, edits & system
Field	Notes
protocolMessage	Revokes (REVOKE), edits (MESSAGE_EDIT), ephemeral sync, welcome requests.
editedMessage	Edited message wrapper (edit attr).
reactionMessage / encReactionMessage	Reaction (type: reaction); empty text revokes.
pinInChatMessage	Pin/unpin (edit: pin_in_chat).
keepInChatMessage	Keep-in-chat.
encCommentMessage	Comment on a message.
requestPhoneNumberMessage	Request a phone number.
newsletterAdminInviteMessage	Newsletter admin invite.
secretEncryptedMessage	Carries secretEncType: EVENT_EDIT, POLL_EDIT, POLL_ADD_OPTION, MESSAGE_EDIT, MESSAGE_SCHEDULE.
messageHistoryNotice / messageHistoryBundle	Group history sharing.
​
Wrappers
These wrap an inner message; the library unwraps them when resolving attributes:
ephemeralMessage, viewOnceMessage, viewOnceMessageV2, deviceSentMessage, groupMentionedMessage, botInvokeMessage, documentWithCaptionMessage.
For view-once specifically, prefer the viewOnce send option over hand-wrapping.
The full protobuf surface is available under the exported proto namespace — proto.Message, proto.Message.ProtocolMessage.Type, etc. Use it to build any field above and to reference enum values.
​
Raw proto cookbook
Concrete client.message.send(jid, …) payloads for every raw kind live in the Raw proto sends guide. This reference only carries the type-mapping tables above; the guide has the send examples plus payment cards (PIX / review-and-pay).
​
Disappearing wrapper (ephemeralMessage)
Wrap any message so it inherits the chat’s ephemeral timer. This is a wrapper mechanic — the inner message is what actually renders:
await client.message.send(jid, {
  ephemeralMessage: { message: { conversation: 'This disappears' } }
})
For the chat-wide ephemeral toggle (protocolMessage + EPHEMERAL_SETTING), see the guide’s Toggle disappearing messages section.

