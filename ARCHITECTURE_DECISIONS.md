# Architecture Decisions

## Summary of Key Decisions

| Aspect | Decision | Rationale |
|--------|----------|-----------|
| **Blob Ownership** | Developer's wallet owns Sui objects | Simpler UX, end-users don't need wallets |
| **Blob Type** | Deletable with auto-renewal | Users can remove content + GDPR compliance |
| **Payment Method** | SUI deposits only | Fully on-chain, no fiat complexity |
| **Bandwidth Overage** | Stop serving (video unviewable) | No credit risk, clear incentive to top up |
| **Renewal Failure** | Don't renew, let blob expire | Developer responsibility, no debt |
| **Playback Format** | HLS streaming | Industry standard, adaptive bitrate |
| **Self-hostable** | Yes | Platform is infrastructure, not gatekeeper |
| **Transcoding** | Original only (no multi-resolution) | Simpler, cheaper, focused scope |
| **Bandwidth Quota** | Cumulative (never resets) | Simple accounting, deposit adds quota |
| **Deletion Refunds** | Storage resource reusable + SUI rebate | Walrus reclaims storage, Sui refunds object storage |
| **Private Videos** | Future improvement (public only for now) | Focus on core functionality first |

---

## Flow Diagrams

See `diagrams/` folder for detailed PlantUML diagrams:

1. **Developer Onboarding** - Registration and SUI deposit
2. **Video Upload** - Chunked upload, transcoding, Walrus storage
3. **Video Playback** - HLS streaming with caching and bandwidth tracking
4. **Auto-Renewal** - Scheduled job to extend blob lifetimes
5. **Video Deletion** - Developer-initiated removal with refund
6. **Bandwidth Enforcement** - Quota checking and blocking
7. **Embed Player** - iframe and direct HLS URL embedding
8. **Component Overview** - High-level system architecture

---

## On-Chain Data Model

### DeveloperAccount Object (Sui)
```move
struct DeveloperAccount has key, store {
    id: UID,
    owner: address,              // Developer's wallet
    balance: u64,                // SUI balance (in MIST)
    bandwidth_quota: u64,        // Total bandwidth allowed (bytes)
    bandwidth_used: u64,         // Bandwidth consumed this period
    created_at: u64,             // Timestamp
}
```

### Video Object (Sui)
```move
struct Video has key, store {
    id: UID,
    owner: address,              // Developer's wallet
    title: String,
    blob_ids: vector<BlobId>,    // Walrus blob references
    sui_object_ids: vector<ID>,  // Walrus Sui object IDs (for extend/delete)
    manifest_blob_id: BlobId,    // HLS manifest location
    expiry_epoch: u64,           // When blobs expire
    bandwidth_consumed: u64,     // Bytes served for this video
    created_at: u64,
    metadata: VideoMetadata,     // Duration, resolution, etc.
}
```

---

## Pricing Model

### Cost Components

1. **Storage Cost** (paid at upload)
   - Based on: total blob size × epochs
   - Matches Walrus pricing with margin

2. **Bandwidth Cost** (deducted from quota)
   - Quota calculated from SUI deposit
   - Example: 1 SUI = 10 GB bandwidth

3. **Renewal Cost** (paid from balance)
   - Same as storage cost calculation
   - Deducted automatically if balance sufficient

### Example Pricing

```
Deposit: 100 SUI
├── Bandwidth quota: 1000 GB
└── Available for storage/renewal

Upload 1GB video for 53 epochs (2 years):
├── Storage cost: ~5 SUI (example)
└── Remaining balance: 95 SUI

Video watched 500 GB worth:
├── Bandwidth used: 500 GB
└── Bandwidth remaining: 500 GB
```

---

## API Endpoints (Draft)

### Developer API

```
POST   /v1/auth/register         # Create account
GET    /v1/account               # Get balance & usage
POST   /v1/deposit               # Get deposit address/instructions

POST   /v1/videos/upload         # Initiate upload
PUT    /v1/upload/{session}/chunk/{n}  # Upload chunk
POST   /v1/upload/{session}/complete   # Finalize upload

GET    /v1/videos                # List videos
GET    /v1/videos/{id}           # Get video details
DELETE /v1/videos/{id}           # Delete video
```

### Playback API

```
GET    /v/{video_id}/stream.m3u8     # HLS manifest
GET    /v/{video_id}/seg_{n}.ts      # HLS segment
GET    /embed/{video_id}             # Embed player page
```

---

## Resolved Decisions

| Question | Decision |
|----------|----------|
| Transcoding | Original only - no multi-resolution |
| Bandwidth reset | Cumulative (never resets) |
| Deletion refunds | Storage resource reusable + SUI rebate |
| Private videos | Future improvement |

## Open Questions (for later)

1. **Analytics**: What metrics to expose to developers?
   - Views, bandwidth, geographic distribution, device types?

2. **Rate limiting**: Should there be API rate limits beyond bandwidth?

3. **Webhook events**: What events should trigger webhooks?
   - Upload complete, video expired, bandwidth warning, etc.

---

## Scope

### Core Features
- Developer registration & SUI deposits
- Video upload (original resolution only)
- HLS playback with caching
- Bandwidth quota enforcement
- Auto-renewal of blobs
- Video deletion
- Basic embed player
- Public videos only

### Future Improvements
- Multi-resolution transcoding
- Private/encrypted videos (Seal integration)
- Advanced analytics dashboard
- Webhook integrations
- Custom player branding
- Geographic distribution / edge caching
