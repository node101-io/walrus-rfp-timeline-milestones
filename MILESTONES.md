# Milestones

## 1. Research & Competitive Analysis (Weeks 1-2)

**Deliverables:** Competitive analysis of existing video platforms (Mux, Cloudflare Stream, Livepeer), market positioning document, finalized product requirements.

**Success:** Published research document articulating the decentralized video infrastructure gap and informing all downstream design decisions.

---

## 2. Technical Architecture & Sui Smart Contracts (Weeks 3-4)

**Deliverables:** Finalized architecture spec (all flows, API contracts, data models). Sui smart contracts deployed to testnet: `DeveloperAccount`, `Video` objects, core functions (`deposit`, `register_video`, `delete_video`, `get_bandwidth_status`). Unit tests with full coverage.

**Success:** Contracts on Sui testnet passing tests for the full on-chain lifecycle. Architecture spec complete enough for any engineer to implement any flow independently.

---

## 3. Walrus Integration & Video Upload Pipeline (Weeks 5-6)

**Deliverables:** Walrus blob lifecycle (store/read/delete/extend). Backend API foundation with auth. Complete upload pipeline: chunked uploads, HLS transcoding, Walrus storage, Sui registration.

**Success:** Developer uploads a video via API, it gets transcoded to HLS, stored on Walrus, and registered on Sui with storage cost deducted.

---

## 4. Video Playback & Cache Layer (Week 7)

**Deliverables:** HLS playback endpoints, cache layer with TTL logic, async bandwidth logging to Sui, video status validation with proper HTTP error codes.

**Success:** End user opens a playback URL and watches HLS-streamed video. Repeat requests served from cache. Expired videos return appropriate errors.

---

## 5. Bandwidth Enforcement, Auto-Renewal, Deletion & Embed (Week 8)

**Deliverables:** Bandwidth quota enforcement with 429 blocking. Auto-renewal scheduler for expiring blobs. Video deletion with Walrus storage reclaim and SUI rebate. Embed player (iframe + direct HLS).

**Success:** Full video lifecycle management works end-to-end. Bandwidth-exceeded developers are blocked until top-up. Deletion returns storage and SUI. Videos embeddable on third-party sites.

---

## 6. Seal Integration & Private Video Pipeline (Weeks 9-10)

**Deliverables:** Seal SDK integration (envelope encryption). Private video upload with AES-encrypted segments and Seal-encrypted keys. Private playback with wallet auth and client-side decryption. Whitelist management on-chain.

**Success:** Whitelisted users connect wallet, sign once, and stream decrypted video client-side. Non-whitelisted users get access denied. Backend never sees decrypted content.

---

## 7. Developer SDK, Reference App & Open-Source Release (Weeks 11-12)

**Deliverables:** TypeScript SDK (npm-publishable) wrapping all API endpoints. Reference app demoing public + private video flows. Full documentation (API ref, self-hosting guide, contract deployment). Open-source repository.

**Success:** SDK enables any platform operation in <20 lines of code. A new developer can clone the repo and self-host their own instance following the guide.
