# networking-101 — 7-Unit Sprint Plan

Build 10 networking projects in TypeScript/Node. Learn networking fundamentals, industry-level TS/Node, and Low-Level Design at the same time. Target: interview-ready for backend roles from startup to FAANG.

---

## How to read this plan

**Assumed starting point:**
- You are comfortable with JavaScript: closures, `this`, prototypes, promises, `async/await`, array methods, destructuring, modules.
- You are **new to TypeScript**. Every TS feature is introduced deliberately, one project at a time, easiest first. Nothing later depends on a TS feature you haven't met yet.
- You have not done formal LLD before. Patterns are introduced only when a project makes you feel the pain the pattern solves.

**This is a 7-unit sprint, not 7 consecutive days.**
- Each unit is roughly 6–8 hours of focused work. Do a unit in one sitting, split across two evenings, or spread over a weekend. The unit is the atom, not the calendar day.
- Do **not** split a single project across a gap. Projects `03`, `05`, and `08` lose their thread if you stop mid-parser.
- Units are strictly ordered. Unit 4 assumes Unit 2's framing knowledge; Unit 6 assumes Unit 5's proxy exists.

**Resuming after a gap — 15-minute ritual before every unit:**
1. Re-read `docs/NOTES.md` for the previous unit only.
2. Run the previous unit's tests. Green means your environment is intact.
3. Re-draw the previous project's class diagram from memory. If you can't, re-read its README before continuing.

**The core loop for every project:**
`problem → networking mechanism → LLD design (interface first) → TS/Node API → code → packet capture → notes`

If you learn the Node API before the mechanism, you get working code and zero understanding. That is exactly what fails interviews.

---

## Repo structure

```
networking-101/
├─ package.json              # pnpm workspaces
├─ tsconfig.base.json        # strict, NodeNext
├─ packages/
│  └─ core/                  # Result, assertNever, logger, branded types, resilience kit
├─ projects/
│  ├─ 01-tcp-echo/           ├─ 06-http-client-pool/
│  ├─ 02-tcp-chat/           ├─ 07-reverse-proxy-lb/
│  ├─ 03-binary-protocol/    ├─ 08-websocket/
│  ├─ 04-dns-client/         ├─ 09-tls-mtls/
│  ├─ 05-http-server/        └─ 10-resilience-kit/
└─ docs/
   ├─ NOTES.md               # one page per project: concept → answer
   ├─ LLD.md                 # class diagram + pattern + rejected alternative
   └─ INTERVIEW.md           # 40 Q&As from your own notes
```

Every project gets a `README.md` with a hand-drawn byte/packet diagram, a class diagram drawn *before* coding, and 5 interview Q&As. **The notes are the deliverable. The code is the excuse.**

---

# UNIT 0 — Setup & Foundations

*Duration: 4 hours. Do this before touching any project.*

## 0.1 — Environment

- [ ] `pnpm init`, workspaces covering `packages/*` and `projects/*`
- [ ] Install: `typescript`, `tsx`, `vitest`, `@types/node`
- [ ] Install CLI tooling: `wireshark` / `tcpdump`, `curl`, `dig`, `nc`, `ss`, `openssl`, `autocannon`
- [ ] `git init`, first commit

## 0.2 — TypeScript onboarding *(you are new to TS — budget 2 hours)*

Learn only this much now. Everything else arrives project by project.

- [ ] **Why TS exists** — it is a *compile-time* layer. Types are erased at runtime. `tsx` runs TS directly; `tsc --noEmit` is your type checker.
- [ ] **Basic annotations** — variables, parameters, return types. Let inference do most of the work; annotate boundaries, not locals.
- [ ] **`interface` vs `type`** — `interface` for object shapes and contracts, `type` for unions and aliases. Don't overthink it.
- [ ] **Union types and narrowing** — `string | null`, then `if (x === null) return`. TS follows your control flow.
- [ ] **`unknown` vs `any`** — `any` disables the compiler and is banned in this repo. `unknown` forces you to check first.
- [ ] **Optional and readonly** — `foo?: string`, `readonly bar: number[]`.
- [ ] **`tsconfig.base.json`** with `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`, `module: "NodeNext"`, `target: "ES2023"`. Turn strict on **from day one** — loosening later is easy, tightening after 3000 lines is misery.
- [ ] **`module: NodeNext` implications** — ESM vs CJS, `.js` extensions in import paths even for `.ts` files, `"type": "module"`. This trips up every TS beginner and is a real interview topic.

**Write in `packages/core` now:**
- [ ] `type Result<T, E> = { ok: true; value: T } | { ok: false; error: E }` — your first discriminated union
- [ ] `function assertNever(x: never): never` — the exhaustiveness helper you'll use in every project

**What NOT to do:**
- Do not watch a 12-hour TypeScript course. You will forget all of it. The 2 hours above plus per-project features is enough.
- Do not learn decorators, namespaces, `enum`, or abstract classes. You will not use them. `enum` in particular is legacy TS — use `as const` objects and union types.
- Do not install `ts-node`, ESLint configs, Prettier plugins, or a build pipeline. `tsx` + `vitest` + `tsc --noEmit` is the whole toolchain.
- Do not disable `strict` "just to get started."

## 0.3 — LLD foundation *(45 min)*

- [ ] **Coupling vs cohesion** — the only two metrics that matter. High cohesion inside a module, low coupling between modules.
- [ ] **Composition over inheritance** — you will use inheritance exactly twice in this repo (`Transform`, `EventEmitter`). Everything else composes.
- [ ] **Program to an interface, not an implementation** — declare the `interface` in its own file, implement it in another.
- [ ] **The SOLID names only** — do not study them yet. Each letter is attached to a project below, which is the only way they stick.
- [ ] **Class diagram habit** — boxes and arrows, five minutes, before code. Every single project.

**What NOT to do:**
- Do not read a design patterns book cover to cover. Pattern catalogs learned out of context produce over-engineering, which is a visible, scored failure in LLD interviews.

## 0.4 — Networking foundation *(60 min)*

- [ ] **Layering** — link → network (IP) → transport (TCP/UDP) → application. Use the 4-layer TCP/IP model. OSI's 7 layers are interview trivia only.
- [ ] **Encapsulation** — draw one nested box: `[Ethernet [IP [TCP [HTTP]]]]`. Every future debugging session is "which box is broken?"
- [ ] **Addressing identity** — MAC (link) vs IP (host) vs port (process).
- [ ] **Memorize this sentence:** "A socket is a 5-tuple: protocol + src IP + src port + dst IP + dst port."
- [ ] **Listening socket vs connection socket** — one listener, many connections. This is why `net.createServer` hands you a *new* socket per client.
- [ ] Run and actually read the output of `ip a`, `ss -tuln`, `ping -c3 google.com`.

**What NOT to do:**
- Do not memorize the OSI 7 layers as your foundation. It is the most common way beginners burn a day and retain nothing usable.
- Do not study subnetting math, routing protocols (BGP/OSPF), or VLANs. Zero relevance to backend interviews.

---

# UNIT 1 — Transport Layer

*~6 hours. Projects `01-tcp-echo`, `02-tcp-chat`, plus the event loop block.*

---

## Project: `01-tcp-echo`

**Description:**
A TCP server that echoes back whatever a client sends, plus a matching client. Deliberately trivial in behavior so all your attention goes to socket lifecycle and layer separation.

**Networking concept to learn:**
Learn strictly in this order, because each item explains the next. Ports (well-known / registered / ephemeral ranges) → the TCP three-way handshake, SYN / SYN-ACK / ACK, and what each side knows after each step → connection teardown, FIN/ACK in both directions, half-open connections → `TIME_WAIT` and why it exists (stray duplicate packets from the old connection) → `SO_REUSEADDR`, which is meaningless magic unless you understand `TIME_WAIT` first → sequence numbers, ACKs, and retransmission, conceptually only, since you never touch these directly in Node.

**LLD concept to learn:**
**Single Responsibility Principle and layer separation.** Split the system into three layers with three different reasons to change: transport (socket I/O), protocol (bytes to messages and back), application (what to do with a message). `TcpTransport` must know nothing about echoing. `EchoHandler` must know nothing about sockets — its signature takes a string and returns a string, with no `socket` anywhere in it. Test of success: you can unit-test `EchoHandler` with zero networking.

**Duration to build it:**
1 hour.

**TypeScript concept to learn (new):**
Interfaces as contracts. Declare `interface MessageHandler { handle(input: string): string }` in its own file before writing any implementation. Also typed function signatures, and letting `strict` force you to handle `socket.remoteAddress` possibly being `undefined`.

**Node/JS APIs to use:**
`net.createServer`, `net.connect`, and the `data` / `end` / `error` / `close` handlers. `Buffer` basics and `buffer.toString('utf8')`. Verify externally with `nc localhost 8080`, `telnet`, and `ss -tnp`.

**What NOT to do:**
- Do not skip the client and just use `nc` for everything. Writing the client teaches you the connect side of the handshake.
- Do not put `console.log("echo: " + data)` inside the socket handler. That is exactly the layer violation this project exists to prevent.
- Do not add a message protocol, delimiters, or JSON yet. Project `03` is where framing lives; adding it here means you never feel *why* you need it.
- Do not ignore the `error` event. An unhandled socket `error` crashes the process, and discovering that now is cheaper than in Unit 5.
- Do not use `any` to silence a strict-mode complaint. Every complaint in this project is TS telling you something true.

---

## Project: `02-tcp-chat`

**Description:**
A multi-client chat server over raw TCP. Clients join rooms with a nickname, messages broadcast to everyone in the room, and the server shuts down gracefully on SIGINT.

**Networking concept to learn:**
Flow control → the sliding window → **backpressure**. The causal chain: the receiver is slow, so the kernel send buffer fills, so `socket.write()` returns `false`, so you must stop writing and wait for the `drain` event. This is the slow-consumer problem and the single most-tested "do you actually know Node" question. Also: what graceful shutdown means at the TCP level — stop accepting new connections, let in-flight work finish, then close.

**LLD concept to learn:**
**Observer pattern and event-driven design.** Publishers do not know their subscribers. Node's `EventEmitter` *is* the Observer pattern, so you are learning the formal name for something you already use. Build a `Room` that holds subscribers and exposes `broadcast()`, where `Room` never references a socket type — define `interface Subscriber { send(msg: Message): void }` and have the socket-backed class implement it. Payoff: backpressure becomes a *design* decision owned by the subscriber, not an `if` buried in your broadcast loop.

**Duration to build it:**
2 hours.

**TypeScript concept to learn (new):**
**Discriminated unions** — `type Command = { type: 'join'; room: string } | { type: 'msg'; text: string } | { type: 'leave' }` — with an exhaustive `switch` ending in `default: assertNever(cmd)`. Add a fourth variant and watch the compiler point at the exact switch you forgot. This is the moment TypeScript stops feeling like paperwork.

**Node/JS APIs to use:**
`Map<string, Subscriber>` for the client registry. `socket.write()`'s boolean return value and the `drain` event. `server.close()` and why it does not kill live sockets. `unref()`. A `process.on('SIGINT')` shutdown handler.

**What NOT to do:**
- Do not ignore `write()`'s return value. Broadcasting to a slow client without backpressure grows an unbounded in-memory buffer until the process dies. Reproduce it deliberately, then fix it.
- Do not store clients in an array and remove with `indexOf`. Use a `Map` keyed by connection ID; you need O(1) removal in Unit 5.
- Do not use bare `string` for room names and nicknames everywhere. Note the discomfort — Project `04` names the smell (primitive obsession) and fixes it.
- Do not put `JSON.parse` in your message handler and call it a protocol. It will break the moment a message splits across two `data` events. **Let it break.** That break is the entire motivation for Unit 2.
- Do not add authentication, persistence, or a web UI. Scope creep here costs you Unit 2.

---

## Block: Event loop deep dive

**Description:**
Not a project — a runtime study block with tiny experiments. Do it at the end of Unit 1 while the chat server is fresh, because you will break it on purpose.

**Networking concept to learn:**
None. This is runtime, not wire.

**LLD concept to learn:**
None.

**Duration:**
2 hours.

**TypeScript/Node concepts to learn:**
The event loop phases in order: timers → pending callbacks → poll → check → close. Then write one script queuing `process.nextTick`, `queueMicrotask`, `setImmediate`, and `setTimeout(0)`, and be able to explain **every line** of the output order. Then the libuv threadpool (default size 4) and what actually uses it — `fs`, `dns.lookup`, `crypto` — versus what does not — net sockets, which use epoll/kqueue directly. Finally, block the loop with a 5-second `while` loop inside your chat server, watch every client freeze simultaneously, then fix it with `worker_threads`.

**What NOT to do:**
- Do not read about the event loop without running the ordering script. The output is counterintuitive, and reading alone produces false confidence.
- Do not conclude "Node is single-threaded therefore slow." Understand *why* it handles thousands of concurrent connections fine, and precisely when it does not (CPU-bound work).
- Do not treat `worker_threads` as a general performance tool. It is for CPU-bound work only; using it for I/O makes things worse.

---

# UNIT 2 — Framing

*~4 hours. Project `03-binary-protocol` plus a refactor pass. The highest-leverage unit in the entire repo.*

---

## Project: `03-binary-protocol`

**Description:**
A binary message codec with a length-prefixed wire format — `[4-byte big-endian length][1-byte type][payload]` — implemented as a `Transform` stream that correctly reassembles messages regardless of how TCP chops up the byte stream.

**Networking concept to learn:**
Start here and internalize it before anything else: **TCP is a byte stream, not a message stream.** There are no message boundaries on the wire. Then the mechanics that cause this: MTU (~1500 bytes) → MSS → segmentation, so your 10 KB write becomes several packets. Then Nagle's algorithm and delayed ACK — why they coalesce small writes, why they interact badly with each other, and what `setNoDelay(true)` actually disables. Then the consequence, which is the whole point: one `write()` may arrive as three `data` events, and three `write()`s may arrive as one. Now framing is obviously necessary rather than arbitrary. Finish with framing strategies — length-prefix vs delimiter vs fixed-size, and the tradeoffs of each — and endianness, where big-endian is network byte order, which is why you use `readUInt32BE` and not `LE`.

**LLD concept to learn:**
**Dependency Inversion Principle.** High-level modules depend on abstractions, not concretions. Define `interface Encoder<T> { encode(msg: T): Buffer }` and `interface Decoder<T> { push(chunk: Buffer): T[] }`, then write two implementations: length-prefixed and newline-delimited. Your chat server must depend on `Codec`, never on `LengthPrefixedCodec`. This is the cleanest possible demonstration of DIP, sitting on top of the highest-value networking concept in the repo.

**Duration to build it:**
3 hours.

**TypeScript concept to learn (new):**
**Generics.** `Encoder<T>` and `Decoder<T>` are your first real generic interfaces. Learn type parameters, generic constraints (`<T extends Message>`), and why `Decoder<ChatMessage>` catches at compile time what `Decoder<any>` would wave through.

**Node/JS APIs to use:**
Subclass `Transform` from `node:stream` — one of only two legitimate inheritance uses in this repo. `Buffer.readUInt32BE` / `writeUInt32BE`, `Buffer.concat`, and `Buffer.subarray` versus `slice` (know why `slice` is a footgun on Buffers). Stream `objectMode`. A `vitest` test feeding the parser one byte at a time, and another concatenating three frames into a single chunk.

**What NOT to do:**
- Do not assume one `data` event equals one message, even though it will appear to work locally. Localhost has a huge MTU and no congestion, so the bug hides perfectly and then shows up in production.
- Do not use `\n` as your delimiter and stop there. Length-prefixed is what teaches you binary protocols; the delimiter version exists only as the second implementation that proves your abstraction works.
- Do not `Buffer.concat` the entire accumulated buffer on every chunk in the hot path. Notice the O(n²) behavior and be ready to discuss it — this exact question gets asked.
- Do not put message *interpretation* inside the codec. The codec produces frames; something else decides what a frame means. Merging them is what makes protocols untestable.
- Do not skip the byte-at-a-time fuzz test. It is the only test that actually proves the parser is correct, and it takes six lines.

---

## Refactor pass: retrofit the codec into `02-tcp-chat`

**Duration:** 45 minutes.

**The exercise:** swap your chat server from newline-delimited to length-prefixed by changing **one constructor argument**. If it requires any other edit, your abstraction leaked — find where and fix it. That single-argument swap is the proof DIP landed, and it is a story worth telling in an interview.

**TypeScript concept:** constructor injection and interface-typed fields.

**What NOT to do:**
- Do not skip this because the chat server "already works." The refactor *is* the lesson; the working code is not.
- Do not fix a leak by widening the interface until both codecs fit. That inverts the dependency back the wrong way.

---

# UNIT 3 — UDP & DNS

*~5 hours. Project `04-dns-client`, plus an optional stretch project.*

---

## Project: `04-dns-client`

**Description:**
Build `dig` from scratch. Hand-encode a DNS query packet, send it over UDP to `8.8.8.8:53`, and parse the response — including compression pointers. Support A, AAAA, CNAME, MX, and TXT records.

**Networking concept to learn:**
Start with UDP: connectionless, unordered, unreliable, no handshake — but **datagram boundaries are preserved**, the exact opposite of everything in Unit 2. That contrast is the lesson; do not skim it. Then when UDP wins: small request/response, latency-sensitive, cheap app-level retry. Then DNS as a system: resolver → root → TLD → authoritative, recursive versus iterative resolution. Then record types (A, AAAA, CNAME, MX, NS, TXT, SOA) and what question each answers. Then TTL and the layers of caching — OS, resolver, application — and why DNS-based failover is slow. Then the wire format: header flags, QNAME label encoding, and finally **message compression pointers** (the `0xC0` prefix), which must come last because they make no sense until you have parsed a flat response. Finish with the causal chain: 512-byte UDP limit → TC flag set → client retries over TCP → EDNS0 raises the limit.

**LLD concept to learn:**
**Builder pattern plus immutable value objects.** A DNS query has nine-plus fields, so a positional constructor would be unreadable. Build `new DnsQueryBuilder().id(x).recursionDesired().question('a.com', 'A').build()` returning a frozen `DnsQuery`. The critical discipline: **keep construction separate from serialization.** The builder makes an object; a separate codec turns that object into a `Buffer`. Merging them is the most common way people get this wrong. Alongside it, learn **value objects** — `DomainName`, `Ttl`, `RecordType` as distinct types rather than raw `string` and `number`. Passing raw primitives everywhere is a named code smell: primitive obsession.

**Duration to build it:**
4 hours.

**TypeScript concept to learn (new):**
**Branded types** — `type DomainName = string & { readonly __brand: 'DomainName' }` — so a function taking a `DomainName` rejects an arbitrary string. Plus `readonly`, `as const`, and `Object.freeze` for immutability. Plus your first real use of `Result<T, E>`: the parser returns `Result<DnsResponse, ParseError>` instead of throwing.

**Node/JS APIs to use:**
`dgram` sockets. Bit manipulation with `&`, `|`, `>>`, and masks for the header flags. `DataView` for multi-byte reads. Method chaining with correct `this` typing in the builder.

**What NOT to do:**
- Do not use the `dns` module. It does the entire job and teaches you nothing. The point is the bytes.
- Do not attempt compression pointers before you can parse a flat, uncompressed response. Doing both at once is how people abandon this project.
- Do not follow a compression pointer without a loop guard. A malicious or corrupt response can point at itself and hang your process — note this as a real parser security concern, because "how do you handle malformed input" is a standard follow-up.
- Do not build the query bytes inside the builder. That merges construction with serialization and destroys the point of the pattern.
- Do not throw exceptions for parse failures. Use `Result`. Parsing untrusted network input is exactly where typed errors earn their keep.
- Do not skip TXT and MX because A records already work. Each has a distinct RDATA shape, and handling that variety is what makes the parser design real.

---

## Project: `[stretch] concurrent port scanner`

**Description:**
Scan a host's ports with bounded concurrency. Optional — skip if you are behind.

**Networking concept to learn:**
ICMP basics, TCP connect scan, `traceroute`. Plus 30 minutes on CIDR and subnets: low weight for backend interviews, but it appears in cloud and VPC questions.

**LLD concept to learn:**
A reusable semaphore / concurrency limiter class.

**Duration to build it:**
1.5 hours.

**TypeScript/Node concepts to use:**
`Promise.allSettled`, an async semaphore, `AbortController`, async iterators.

**What NOT to do:**
- Do not fire 65,535 connections with `Promise.all`. That is precisely why the semaphore exists — feel the file-descriptor exhaustion first.
- Do not scan hosts you do not own. Scan `localhost` or a machine you control.
- Do not spend more than 90 minutes here. It is a stretch item, not a checkpoint.

---

# UNIT 4 — HTTP

*~7 hours. Projects `05-http-server` and `06-http-client-pool`.*

---

## Project: `05-http-server`

**Description:**
An HTTP/1.1 server built on a raw TCP socket, with the `http` module banned. Parses request lines and headers by hand, handles both `Content-Length` and chunked bodies, supports keep-alive, and serves static files with caching headers.

**Networking concept to learn:**
The request/response model and statelessness, and therefore why cookies and tokens exist. URL anatomy — scheme, host, port, path, query, fragment — including that the fragment never leaves the browser. Message structure: start line, headers, blank line, body, where finding `\r\n\r\n` is essentially the parser's entire job. Method semantics: safe versus idempotent versus cacheable, since "is PUT idempotent? is POST?" is asked constantly. Status code families, then the ones carrying real logic: 301 vs 302 vs 307/308, 401 vs 403, 429, 502 vs 503 vs 504. Then **body length determination** in order: `Content-Length`, then `Transfer-Encoding: chunked`, then connection-close as HTTP/1.0 legacy — and only *after* those, **request smuggling**, which is what happens when two servers disagree about which of CL and TE wins. Then persistent connections: keep-alive, pipelining, application-layer head-of-line blocking. Then caching: `Cache-Control`, `ETag` with `If-None-Match`, `Last-Modified` with `If-Modified-Since`, and the 304 flow. Then content negotiation: `Accept`, `Content-Type`, charset, `Content-Encoding`. Finish with CORS — origin definition, simple versus preflighted requests, `OPTIONS`, credentials — learned here at the protocol level, not as browser folklore.

**LLD concept to learn:**
**Chain of Responsibility, as a middleware pipeline.** `type Middleware = (ctx: Ctx, next: () => Promise<void>) => Promise<void>`. Build logging, auth, static-file serving, and 404 as independent middleware. This is literally how Express and Koa work, so you understand both for free afterward. Two things interviewers probe: how errors propagate through the chain, and the "`next()` called twice" bug.

**Duration to build it:**
4 hours, plus 30 minutes for the comparison pass.

**TypeScript concept to learn (new):**
**Template literal types** — `` type Route = `${Method} ${string}` `` — plus `satisfies` for a route table that keeps its literal types while still being checked, and mapped/utility types like `Record<string, string>` for headers. This is where TS starts doing things no other mainstream language does.

**Node/JS APIs to use:**
Raw `net` sockets, `Buffer.indexOf` to find the header terminator, `createReadStream` with `pipeline` for file serving — never `readFile` a whole file into memory. Async middleware composition with `async/await`.

**Comparison pass (30 min):**
Rewrite the whole thing in about 30 lines using `node:http`, then diff the two mental models. You now know exactly what that module was doing for you.

**What NOT to do:**
- Do not use the `http` module for the main build. Parsing it by hand is the entire value.
- Do not assume the full request headers arrive in one `data` event. Same lesson as Unit 2, and it applies just as hard here.
- Do not implement chunked encoding with a regex. Parse the chunk-size lines properly.
- Do not accept both `Content-Length` and `Transfer-Encoding` on one request. Reject it, and write down why — that is the request smuggling attack class and it makes a great interview answer.
- Do not `readFile` a file into memory to serve it. Stream it. A 2 GB download must not become 2 GB of RSS.
- Do not trust the path from the request line. Handle `../` traversal, and note the vulnerability in your README.
- Do not build routing with a giant `if/else` on `req.url`. Middleware plus a route table is the point.
- Do not add templating, sessions, or a database. Not this repo.

---

## Project: `06-http-client-pool`

**Description:**
Your own HTTP client with a connection pool: keep-alive reuse, three separate timeouts, redirect following, and cancellation.

**Networking concept to learn:**
The economics of connection reuse — a TCP handshake plus a TLS handshake per request is why pooling exists. HTTP/1.1 head-of-line blocking from the client's side. DNS caching and why the resolver sits in your latency budget. The three distinct timeouts beginners collapse into one: connect timeout, headers timeout, and body/total timeout. Redirect semantics, including which methods change on a 301 versus a 307. And why `undici` exists at all.

**LLD concept to learn:**
**Object Pool plus Factory, and resource lifecycle discipline.** `acquire()` and `release()` must be symmetric, and `release()` must live in a `finally`. `ConnectionFactory` creates sockets; `ConnectionPool` owns min/max size, idle eviction, and a wait queue for exhaustion. Learn the three failure modes as *design* concerns rather than bugs: connection leak from a missing release, pool exhaustion deadlock, and stale connection reuse. This is also the honest place to discuss **Singleton** and why a single injected instance beats a global.

**Duration to build it:**
2.5 hours.

**TypeScript concept to learn (new):**
Async iteration with types — `Symbol.asyncIterator`, `AsyncIterableIterator<Buffer>`, and `for await...of` over a response body. Plus generic constraints on the pool: `Pool<T extends Disposable>`.

**Node/JS APIs to use:**
`AbortController`, `AbortSignal.timeout`, and `AbortSignal.any` for composing cancellation. A promise-based wait queue. `try/finally` everywhere a resource is held. Optionally `using` and `Symbol.dispose` as a stretch.

**What NOT to do:**
- Do not call `release()` outside a `finally`. Any thrown error leaks a connection, and pool exhaustion under load is the result.
- Do not use one timeout value for everything. Three timeouts, three reasons, three knobs.
- Do not reuse a socket that errored or that the server may have closed. Stale connection reuse produces the classic intermittent `ECONNRESET` that is miserable to debug.
- Do not follow redirects without a hop limit. Infinite redirect loops are real.
- Do not make the pool a module-level global for convenience. Inject it — you need multiple pools with different configs in Unit 5.
- Do not buffer the entire response body into memory by default. Expose a stream and let the caller decide.

---

# UNIT 5 — Proxying & Resilience

*~6 hours. Projects `07-reverse-proxy-lb` and `10-resilience-kit`.*

---

## Project: `07-reverse-proxy-lb`

**Description:**
A reverse proxy and load balancer. Accepts a client connection, picks a backend by a pluggable strategy, and pipes traffic both ways — with health checking, correct header handling, and `CONNECT` tunneling.

**Networking concept to learn:**
Forward proxy versus reverse proxy first, framed by the direction of trust — get this straight before anything else. Then L4 versus L7 balancing: what each layer can see, and therefore what each can route on. Then the algorithms in order: round-robin → weighted → least-connections → **consistent hashing**, and learn the hash ring only *after* seeing plain modulo hashing and its resharding problem, or the ring is unmotivated. Then health checking: active probes versus passive ejection and outlier detection. Then **hop-by-hop versus end-to-end headers** — `Connection`, `Keep-Alive`, and `Transfer-Encoding` must not be forwarded, an RFC rule proxies get wrong constantly. Then `X-Forwarded-For` and `Forwarded`, and why trusting them blindly is a spoofing vulnerability. Then the `CONNECT` method and tunneling, the mechanism that lets a proxy carry TLS it cannot read. Finish with NAT — private versus public addressing, port translation, and why inbound connections to a home machine fail.

**LLD concept to learn:**
**Strategy pattern and the Open/Closed Principle.** Software should be open for extension and closed for modification: adding a new balancing algorithm must not touch the proxy. Define `interface LoadBalancingStrategy { pick(backends: Backend[], req: RequestMeta): Backend }` and implement `RoundRobin`, `LeastConnections`, and `ConsistentHash`, selected by config. The smell you are eliminating is a `switch (algorithm)` sitting in the hot path. Replacing conditionals with polymorphism is *the* core LLD move, and Strategy is the single most-asked pattern in LLD rounds.

**Duration to build it:**
3.5 hours.

**TypeScript concept to learn (new):**
Config-driven dependency injection with a typed registry: `Record<StrategyName, () => LoadBalancingStrategy>`, where `StrategyName` is a union derived from the registry via `keyof typeof`. Plus `readonly` arrays so backend lists cannot be mutated by accident.

**Node/JS APIs to use:**
`pipeline()` from `node:stream/promises` — and understand why raw `.pipe()` leaks sockets when one side errors. `Duplex` as your parameter type, **not** `net.Socket` (this matters enormously in Unit 6). `crypto.createHash` for the ring, plus a sorted array with binary search for lookups.

**What NOT to do:**
- Do not use raw `.pipe()` for proxying. It does not propagate errors or clean up the other half of the connection, and you will leak sockets under any real failure.
- Do not forward hop-by-hop headers to the backend. This is the correctness bug separating a toy proxy from a real one.
- Do not blindly append to `X-Forwarded-For` from an untrusted source. Note the spoofing risk explicitly.
- Do not put `if (strategy === 'roundRobin')` anywhere. That is the exact anti-pattern this project exists to eliminate.
- Do not implement `hash % n` and call it consistent hashing. Build the ring with virtual nodes, and be able to explain what happens to key distribution when a backend is removed.
- Do not type any function parameter as `net.Socket`. Type it `Duplex`. Unit 6 depends on this and you will thank yourself.
- Do not add TLS yet. That is Project `09`, deliberately, so the substitution lesson lands.

---

## Project: `10-resilience-kit`

**Description:**
A composable reliability layer living in `packages/core`: timeout, retry, circuit breaker, rate limiter, and metrics — each a wrapper with the same interface. Built now, wired into everything afterward.

**Networking concept to learn:**
This block is distributed systems rather than wire protocol. The failure taxonomy: connect timeout versus read timeout versus total deadline — three distinct clocks. Retries and the idempotency requirement: never retry a non-idempotent request without an idempotency key. Then the causal chain thundering herd → exponential backoff → **jitter**, and specifically why full jitter beats naive backoff. Circuit breakers and why they beat pure retry under sustained failure. Rate limiting: token bucket versus leaky bucket versus fixed window versus sliding window, and where to enforce — edge versus service. Load shedding, bulkheads, and deadline propagation. Finally tail latency: p50/p95/p99, why averages lie, and how retries make p99 *worse* under load.

**LLD concept to learn:**
**Decorator pattern and composition chains.** Define `interface Caller<Req, Res> { call(req: Req): Promise<Res> }`, then stack wrappers that each implement the same interface: `withTimeout(withRetry(withCircuitBreaker(withMetrics(httpCaller))))`. Reason explicitly about **order** — why timeout goes outside retry, and what changes if you flip them. The circuit breaker itself is a state machine (closed → open → half-open), previewing Unit 6's State pattern. This stack is essentially what a service mesh sidecar does, and being able to whiteboard it reads as genuinely senior.

**Duration to build it:**
2.5 hours.

**TypeScript concept to learn (new):**
Higher-order functions that **preserve generics**: `function withRetry<Req, Res>(inner: Caller<Req, Res>, opts: RetryOpts): Caller<Req, Res>`. Getting type parameters to flow through a four-deep wrapper chain without widening to `any` is the exercise.

**Node/JS APIs to use:**
`AbortSignal` propagation through every layer. A discriminated union for breaker state. `perf_hooks` for latency histograms. Injected `clock` and `random` — **never** call `Date.now()` or `Math.random()` inside the logic.

**What NOT to do:**
- Do not call `Date.now()` or `Math.random()` directly in retry or breaker logic. Inject both. Otherwise your backoff tests are non-deterministic and slow, and "how would you test this?" has no good answer.
- Do not retry every failure. Retrying a 400 or 404 is pointless load; retrying a non-idempotent POST is a correctness bug.
- Do not implement backoff without jitter. Synchronized retries from many clients are precisely the herd that takes down a recovering service.
- Do not merge timeout, retry, and breaker into one `resilientCall()`. Separate decorators are the entire point, and merging them makes the ordering question unanswerable.
- Do not let the circuit breaker be a pile of booleans. It is a three-state machine — model it as one.
- Do not skip wiring this into Projects `06` and `07`. An unused library teaches nothing.

---

# UNIT 6 — Real-time & Security

*~7 hours. Projects `08-websocket` and `09-tls-mtls`.*

---

## Project: `08-websocket`

**Description:**
RFC 6455 implemented from scratch, with the `ws` library banned. Handshake, frame parsing and masking, ping/pong, close codes — tested against a real browser via a small static HTML page.

**Networking concept to learn:**
Start with the polling ladder: short polling → long polling → Server-Sent Events → WebSocket, understanding the specific pain each step solved, because "why not just use SSE?" is a standard follow-up. Then the HTTP `Upgrade` mechanism and `101 Switching Protocols`: the handshake *is* HTTP, and nothing after it is. Then `Sec-WebSocket-Key` plus the magic GUID, SHA-1, and base64 — not security, but proof of protocol awareness, defending against caching proxies. Then frame anatomy: FIN, RSV, opcode, the MASK bit, and the 7 / 7+16 / 7+64 payload length encoding. Then the client-masking rationale — cache poisoning of intermediaries — which makes an excellent interview answer. Then control frames: ping/pong keepalive, close codes, and idle timeouts at load balancers. Finish with scaling: sticky sessions, why a stateless load balancer breaks you, and pub/sub fan-out across nodes.

**LLD concept to learn:**
**State pattern, as an explicit state machine.** Behavior changes with state, so model states explicitly instead of scattering booleans like `isConnected`, `isClosing`, and `hasHandshaked`. Build two machines: the connection lifecycle `CONNECTING → OPEN → CLOSING → CLOSED`, where each state defines what `send()` and `close()` do; and the frame parser `READ_HEADER → READ_LENGTH → READ_MASK → READ_PAYLOAD`. State machines are the backbone of classic LLD problems — elevator, vending machine, order lifecycle, parking lot — so this transfers directly.

**Duration to build it:**
4 hours, plus 30 minutes for the SSE comparison.

**TypeScript concept to learn (new):**
Using discriminated unions and `assertNever` as a **design tool** rather than a type-safety nicety: make illegal states unrepresentable, so `ClosedConnection` simply has no `send` method and the compiler enforces the protocol. This is the most senior-feeling thing TypeScript can do for you.

**Node/JS APIs to use:**
`node:crypto` with `createHash('sha1')`. XOR unmasking loops over `Buffer`. `server.on('upgrade')` on top of your Project `05` server. A static HTML page using the browser's native `WebSocket` to prove interoperability.

**SSE comparison (30 min):**
Add an SSE endpoint reusing the Observer design from Project `02`. Know when SSE is the better answer: server-to-client only, auto-reconnect built in, plain HTTP end to end.

**What NOT to do:**
- Do not use the `ws` library. The frame parser is the entire learning.
- Do not test only against your own client. Test against a real browser — the browser masks, and if your unmasking is wrong you will never discover it client-to-client.
- Do not skip masking because "it's not really security." A spec-compliant server must reject an unmasked client frame, and knowing why is a strong answer.
- Do not ignore continuation frames and assume every message is a single frame. Send something over 64 KB and watch it break.
- Do not use booleans for connection state. That is the exact anti-pattern this project targets.
- Do not skip ping/pong. Idle WebSocket connections get silently killed by intermediaries; keepalive is why production apps work.
- Do not build a full chat product on top of it. You already built chat in Project `02`; this is about the protocol.

---

## Project: `09-tls-mtls`

**Description:**
Generate your own CA, server cert, and client cert; run a TLS server; then enable mutual TLS. Then TLS-terminate in your Unit 5 proxy. This is as much a **refactoring exercise** as a build.

**Networking concept to learn:**
Threat model first: confidentiality, integrity, authenticity — and which mechanism provides which. Then symmetric versus asymmetric crypto and hashing, twenty minutes, conceptual only. Then certificates and the chain of trust: CA, intermediate, leaf, root store, and **CN versus SAN** (SAN is what actually gets validated now). Then the TLS 1.2 handshake, then 1.3 and what it cut — round trips, RSA key exchange, bad ciphers. Then **SNI**, why virtual hosting needs it, and that it travels in plaintext, which is why ECH exists. Then **ALPN**, which is how HTTP/2 gets negotiated during the handshake. Then mTLS and where it is used: service mesh, zero trust. Then session resumption and the 0-RTT replay risk. Finish with TLS termination versus passthrough, which connects straight back to Unit 5.

**LLD concept to learn:**
**Liskov Substitution Principle.** A subtype must be usable anywhere the supertype is, without the caller knowing. `tls.TLSSocket` is substitutable for `net.Socket` — and *that is precisely why* you can bolt TLS onto your Unit 5 proxy with almost no code change. So this is a test of your earlier work: if any function you wrote takes `net.Socket` instead of `Duplex`, that is an LSP violation **you** introduced. Find them and fix them. Note the **Decorator** link too: TLS wraps a stream, exposes the same interface, adds behavior — the same shape as Project `10`. LSP is the SOLID letter people fumble, and a concrete story makes you sound like you have shipped things.

**Duration to build it:**
2.5 hours.

**TypeScript concept to learn (new):**
Structural typing and interface widening: why `Duplex` as a parameter type is strictly better than `net.Socket`, and how TS's structural type system makes substitution work without explicit inheritance declarations.

**Node/JS APIs to use:**
`tls.createServer` with `requestCert: true` and `rejectUnauthorized: true`. The `openssl` CLI to build a CA and issue server and client certs. `openssl s_client -connect example.com:443` to read a real handshake byte by byte.

**What NOT to do:**
- Do not use a self-signed cert with `rejectUnauthorized: false` and move on. Build a real CA and validate the chain — the trust model is the point.
- Do not set `NODE_TLS_REJECT_UNAUTHORIZED=0`. Ever. If it is tempting, your cert chain is wrong, and that is the thing to fix.
- Do not implement any cryptography yourself. Use `node:tls`. Understanding the protocol is the goal; hand-rolled crypto is a bug factory.
- Do not skip refactoring Units 4 and 5 to accept `Duplex`. Skipping it means skipping the LSP lesson, which is the entire LLD content of this project.
- Do not treat CN as the hostname field. Modern validation uses SAN, and getting this wrong in an interview is a tell.
- Do not go deep on cipher suite internals. Know that negotiation happens and what 1.3 removed; the mathematics is not backend interview material.

---

# UNIT 7 — Scale, Verify, Consolidate

*~8 hours. No new protocols — this unit turns the repo into an interview artifact.*

---

## Block: Multi-core & HTTP/2

**Description:**
Scale your Unit 4 server across cores, benchmark it, then study the protocol generations that came after HTTP/1.1.

**Networking concept to learn:**
State HTTP/1.1's problems *first*: head-of-line blocking, connection-per-origin limits, header bloat. Only then **HTTP/2**: binary framing, streams, multiplexing, HPACK header compression, per-stream flow control, and why server push failed and was deprecated. Then the problem H2 *cannot* fix: head-of-line blocking at the **TCP** layer, since one lost segment stalls every multiplexed stream. Then **QUIC and HTTP/3**: UDP-based, genuinely independent streams, 0-RTT, and connection migration by connection ID.

**LLD concept to learn:**
None new — this is composition of everything so far.

**Duration:**
2.5 hours.

**TypeScript/Node concepts to use:**
`cluster` with shared listening sockets. `worker_threads` for CPU-bound work versus `cluster` for I/O concurrency — know which problem each solves. `node:http2` with stream multiplexing. `autocannon` to benchmark single-process versus clustered and see real numbers.

**What NOT to do:**
- Do not present HTTP/2 as "HTTP/1.1 but faster." State the problem, then the solution, every time.
- Do not claim H2 solved head-of-line blocking. It solved it at the application layer only; that nuance is exactly what senior interviews probe.
- Do not use `cluster` to fix a CPU-bound handler. That is `worker_threads`.
- Do not implement HTTP/2 from scratch. HPACK alone would eat the whole unit for low marginal learning.

---

## Block: Observability & testing

**Description:**
Structured logging with request-ID propagation across proxy hops, latency histograms, and a real test suite across all projects.

**Networking concept to learn:**
Request-ID propagation across hops, and measuring p50/p95/p99 on your own stack rather than reading about them.

**LLD concept to learn:**
**Dependency Injection and designing for testability.** Inject collaborators — clock, logger, random source, socket factory — rather than importing them. Every hidden dependency is an untestable seam. Learn the vocabulary properly: fakes versus mocks versus stubs, and why a real server on port `0` beats a mocked socket. "How would you test this?" follows almost every LLD design question, and "I injected the clock, so —" is the answer that lands.

**Duration:**
2.5 hours.

**TypeScript/Node concepts to use:**
`vitest` integration tests that listen on port `0` and read `server.address().port` — never hardcode ports in tests. Fuzz the framing parser with randomized chunk splits. `perf_hooks` histograms. Structured JSON logging. `AsyncLocalStorage` for request context propagation.

**What NOT to do:**
- Do not hardcode port numbers in tests. Port `0` gives you an OS-assigned free port and lets tests run in parallel.
- Do not mock `net.Socket`. Start a real server; it is faster to write and tests something true.
- Do not chase 100% coverage. Cover the parsers, the state machines, and the retry logic — that is where the bugs live.
- Do not report average latency anywhere. Percentiles only, and be ready to explain why.

---

## Block: Refactor & document

**Description:**
Re-read your Unit 1 code with Unit 7 eyes, then produce the three documents that are the actual output of this sprint.

**LLD concept to learn:**
**Refactoring and code smells.** Name what you find: god class, primitive obsession, feature envy, shotgun surgery, long parameter list. Do one real refactor per smell, each as its own commit with the smell name in the message.

**Duration:**
3 hours.

**Deliverables:**
- `docs/LLD.md` — one class diagram per project, the pattern used, and the alternative you rejected with the reason. "Why *not* inheritance here" is a stronger answer than naming the pattern.
- `docs/INTERVIEW.md` — 40 Q&As drawn from your own project notes.
- Rehearse **out loud**: "walk me through what happens when I type a URL and press Enter," touching every project you built.

**What NOT to do:**
- Do not rewrite Unit 1 from scratch. Refactor it in small commits, so the git history shows the reasoning.
- Do not add patterns retroactively to look sophisticated. If a project did not need one, that restraint is itself the correct answer.
- Do not write `INTERVIEW.md` by pasting questions from the internet. Derive them from your own notes, or you will not survive follow-ups.
- Do not rehearse silently. Speaking exposes the gaps that reading hides.

---

# Definition of Done

A project is **not** done until every box is ticked. No exceptions, no "I'll come back to it."

**Code**
- [ ] `tsc --noEmit` passes under the full strict config
- [ ] Zero `any`, zero `@ts-ignore`, zero non-null assertions (`!`)
- [ ] Every public boundary is an `interface`, declared before its implementation
- [ ] The designated LLD pattern is present — and **only** that pattern
- [ ] All I/O errors handled: `error`, `close`, `timeout`, unexpected EOF
- [ ] Graceful shutdown wired in: stop accepting → drain in-flight → force-kill after N seconds
- [ ] No `Date.now()` or `Math.random()` inside logic; both injected

**Tests**
- [ ] At least one unit test of core logic with **zero networking involved** — if that is impossible, your layer separation failed
- [ ] At least one integration test binding to port `0`
- [ ] For any parser: a fuzz test feeding input one byte at a time and in randomly-sized chunks

**Verification — the tool triangle, all three, every time**
- [ ] Ran it and it works
- [ ] Poked it with a real CLI tool (`curl -v` / `dig` / `nc` / `openssl s_client`)
- [ ] Watched your own bytes on the wire in Wireshark or `tcpdump`

**Documentation**
- [ ] `README.md` has a hand-drawn byte/packet diagram
- [ ] `README.md` has a class diagram, drawn *before* the code
- [ ] Two sentences justifying the pattern: what breaks without it, and what it costs
- [ ] One sentence on the alternative design you rejected, and why
- [ ] 5 interview Q&As in your own words
- [ ] `docs/NOTES.md` updated with the networking concepts in the causal order you learned them
- [ ] The project's **What NOT to do** list reviewed — confirm you did none of them

**Retention**
- [ ] Every line typed by hand — nothing copy-pasted
- [ ] Committed per milestone, with messages explaining the **concept**, not the code
- [ ] Can whiteboard the core mechanism from memory without looking

---

# End Goal

By the end of Unit 7 you can:

**Networking**
- [ ] Whiteboard "what happens when I type a URL and press Enter" end to end — DNS, TCP, TLS, HTTP, proxying, caching — citing code you personally wrote at each step
- [ ] Explain why TCP needs framing, and implement a length-prefixed parser from memory
- [ ] Compare TCP vs UDP, HTTP/1.1 vs 2 vs 3, and polling vs SSE vs WebSocket — always stating the problem before the solution
- [ ] Read a packet capture and identify handshakes, retransmissions, and resets

**TypeScript / Node**
- [ ] Write strict TS where the type system enforces the design: discriminated unions, branded types, generics, template literal types, `satisfies`, exhaustiveness checking
- [ ] Explain the event loop phases, the libuv threadpool, and when to reach for `worker_threads` versus `cluster`
- [ ] Handle streams correctly: backpressure, `pipeline` versus `pipe`, async iteration, cancellation via `AbortSignal`
- [ ] Go from "new to TS" to writing TS that passes a senior code review — which is what a strict config plus ten real projects actually buys you

**LLD**
- [ ] Name and apply SRP, OCP, LSP, and DIP, plus Observer, Builder, Chain of Responsibility, Object Pool, Strategy, State, and Decorator — each anchored to a system you shipped
- [ ] Start any design question with the interface, not the class
- [ ] Answer "how would you test this?" with "I injected the clock, so —" instead of hesitating
- [ ] Identify code smells by name and justify a refactor

**Artifacts**
- [ ] A public repo with 10 working projects, tests, and diagrams
- [ ] `docs/LLD.md` — class diagram, pattern, rejected alternative, per project
- [ ] `docs/INTERVIEW.md` — 40 Q&As from your own notes, rehearsed out loud

---

# Sprint-wide rules

1. **Interface first, always.** Write the interface and its method signatures before one line of implementation. An awkward interface means the design is wrong, and now is the cheap time to find out.
2. **One pattern per project, deliberately.** Do not sprinkle five patterns into the TCP echo server. Restraint is scored in LLD interviews.
3. **Justify, don't decorate.** Every pattern gets its two-sentence cost/benefit note. That is the answer format interviewers want.
4. **Never copy-paste.** Type every line. Re-derive the framing parser from memory as a Unit 7 warm-up.
5. **Timebox hard.** If a project runs 50% over: commit what works, write the notes, move on. Coverage beats polish here.
6. **One unit per sitting, in order.** Never split a project across a gap, especially `03`, `05`, and `08`.
7. **Skip entirely:** ORMs, Express/Nest, Docker, deployment, auth libraries, CI pipelines. All noise for this goal.
