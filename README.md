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
