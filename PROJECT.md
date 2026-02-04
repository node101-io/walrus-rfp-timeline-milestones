# Walrus Video Infrastructure Platform

A developer-first video infrastructure built on Walrus and Sui, offering cloud-grade video hosting without centralized dependencies.

## Problem

Building video infrastructure is expensive and complex. Teams must integrate object storage, upload pipelines, CDN delivery, access control, and billing across multiple vendors. This creates lock-in, high costs, and operational burden.

Existing solutions optimize for delivery, not ownership. Videos are tied to platforms with no guarantees around portability, integrity, or long-term availability.

## Solution

An open-source, **self-hostable** video backend that uses:
- **Walrus** for decentralized blob storage
- **Sui** for on-chain state (ownership, balances, metadata)
- **HLS streaming** for standard video playback

Developers get a simple API to upload, manage, and serve videos. End users watch videos through standard players.

## Why Self-Hostable Matters

This platform is **not a centralized service**. It's infrastructure that anyone can run.

| Concern | How We Address It |
|---------|-------------------|
| "What if you go down?" | Self-host your own instance |
| "What if you censor content?" | Run your own backend, same protocol |
| "Is this really decentralized?" | Storage (Walrus) and state (Sui) are decentralized. Backend is just an API layer anyone can deploy |

The backend is a **convenience layer**, not a gatekeeper. All video data lives on Walrus. All ownership records live on Sui. The backend simply provides developer-friendly APIs and caching.

```
┌─────────────────────────────────────────────────────────┐
│  Your App                                               │
├─────────────────────────────────────────────────────────┤
│  Platform Backend (self-hostable, replaceable)         │
├─────────────────────────────────────────────────────────┤
│  Sui (ownership, state)  +  Walrus (video storage)     │
│  ─────────────── decentralized layer ───────────────   │
└─────────────────────────────────────────────────────────┘
```

## Architecture Overview

![High-Level Architecture](diagrams/08-component-overview.puml)

### Key Components

| Component | Role |
|-----------|------|
| **Platform Backend** | API gateway, upload handling, playback serving, caching |
| **Sui Smart Contract** | Developer accounts, balances, video metadata, bandwidth tracking |
| **Walrus Storage** | HLS segments and manifests as deletable blobs |

### Data Flow

1. **Developer** deposits SUI, gets bandwidth quota
2. **Upload**: Video → Backend → Transcode to HLS → Store on Walrus → Register on Sui
3. **Playback**: End user → Backend (cache) → Walrus → HLS stream
4. **Renewal**: Scheduled job auto-extends blobs if balance sufficient

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Blob ownership | Developer's wallet | Simpler UX, end-users don't need wallets |
| Blob type | Deletable + auto-renewal | Content removal possible, GDPR compliant |
| Payment | SUI deposits only | Fully on-chain, no fiat complexity |
| Playback | HLS streaming | Industry standard, works everywhere |
| Bandwidth exceeded | Videos become unviewable | No credit risk, clear incentive to top up |
| Deletion | Storage resource reusable + SUI rebate | Walrus reclaims storage, Sui refunds object fees |

See [ARCHITECTURE_DECISIONS.md](ARCHITECTURE_DECISIONS.md) for full details.

## Sequence Diagrams

Detailed flows are documented in PlantUML format:

| Flow | File | Description |
|------|------|-------------|
| Developer Onboarding | [01-developer-onboarding.puml](diagrams/01-developer-onboarding.puml) | Registration and SUI deposit |
| Video Upload | [02-video-upload.puml](diagrams/02-video-upload.puml) | Chunked upload, HLS transcoding, Walrus storage |
| Video Playback | [03-video-playback.puml](diagrams/03-video-playback.puml) | HLS streaming with caching |
| Auto-Renewal | [04-auto-renewal.puml](diagrams/04-auto-renewal.puml) | Scheduled blob lifetime extension |
| Video Deletion | [05-video-deletion.puml](diagrams/05-video-deletion.puml) | Content removal with storage reclaim |
| Bandwidth Enforcement | [06-bandwidth-enforcement.puml](diagrams/06-bandwidth-enforcement.puml) | Quota checking and blocking |
| Embed Player | [07-embed-player.puml](diagrams/07-embed-player.puml) | iframe and direct HLS embedding |
| Component Overview | [08-component-overview.puml](diagrams/08-component-overview.puml) | High-level architecture |

## Pricing Model

Developers pay in SUI. Costs map directly to underlying resources:

| Resource | What Developer Pays |
|----------|---------------------|
| **Storage** | Blob size × epochs (passed through from Walrus) |
| **Bandwidth** | Per GB served (from deposited balance) |
| **Renewal** | Same as storage cost (auto-deducted if balance available) |

When a video is deleted:
- Walrus storage resource is **reclaimed** (reusable for new uploads)
- Sui object storage fee is **refunded** as rebate

## Scope

### Core Features
- Developer registration & SUI deposits
- Video upload (original resolution)
- HLS playback with caching
- Bandwidth quota enforcement
- Auto-renewal of blobs
- Video deletion with storage reclaim
- Basic embed player
- Public videos only

### Future Improvements
- Multi-resolution transcoding (adaptive bitrate)
- Private/encrypted videos (Seal integration)
- Analytics dashboard
- Webhook integrations
- Custom player branding

## Deliverables

- [ ] Competitive analysis of existing video platforms (Mux, Cloudflare Stream, etc.)
- [ ] Technical architecture documentation
- [ ] Sui smart contracts (Move)
- [ ] Walrus storage integration
- [ ] Upload & playback pipeline
- [ ] Developer SDK (TypeScript)
- [ ] Reference implementation
- [ ] Documentation & open-source repository

## Why Walrus + Sui?

| Traditional Video Hosting | This Platform |
|---------------------------|---------------|
| Vendor lock-in | Self-hostable, open protocol |
| Opaque storage | Walrus: verifiable, decentralized |
| Platform owns content | Developer wallet owns Sui objects |
| Monthly bills, credit cards | SUI deposits, on-chain accounting |
| Single point of failure | Walrus redundancy across storage nodes |

Walrus provides the **storage layer** with built-in redundancy and verifiability. Sui provides the **ownership layer** with programmable logic. Together, they enable video infrastructure that's both developer-friendly and genuinely decentralized.

---

## Links

- [Architecture Decisions](ARCHITECTURE_DECISIONS.md)
- [Sequence Diagrams](diagrams/)
