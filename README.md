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
