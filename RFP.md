# {Project Name} - node101

# 1. Project Name

**{Project Name}**

# 2. Purpose

{Project Name} is a developer-first video infrastructure platform built on Walrus and Sui. The goal is to give developers the tools and APIs to upload, manage, serve, and protect video without depending on a centralized video platform.

The project is intentionally open-source and self-hostable. node101 will operate the primary hosted backend experience, which is the default path for developers using the product. Teams can still run their own backend because video assets remain on Walrus, and ownership, balances, metadata, and access rules remain on Sui.

* **Current Challenge**: Existing video platforms already solve upload, transcoding, and playback well, but they do so inside a vendor-controlled backend model. That model is convenient, but it creates lock-in around storage, delivery, metadata, access control, and application logic. Developers can build on top of the provider's APIs, but the underlying system remains tightly coupled to one vendor, which makes portability and deeper control difficult.
* **Proposed Solution**: node101 provides a hosted video infrastructure platform as the default developer experience, while keeping the system open underneath. Developers onboard through node101's hosted API, receive API keys, deposit SUI, upload videos, transcode them into HLS, store them on Walrus, and register their lifecycle on Sui. Public videos are streamed through a cache-backed aggregator, and private videos use Seal so playback keys are released only to approved wallets and decryption happens client-side. The important point is not that teams are expected to self-host or replace the backend, but that they retain that option because the underlying data and state are not trapped inside node101's infrastructure.
* **Key Features**:
  * Developer onboarding with API keys and SUI-funded balances
  * Chunked upload flow with HLS transcoding and Walrus blob storage
  * Public playback through node101's hosted aggregator, with a self-hosted path available for teams that want to run their own backend
  * Bandwidth quota enforcement backed by on-chain balances
  * Scheduled blob auto-renewal for retained content
  * Video deletion with Walrus storage reclaim and correct handling of any SUI returned during deletion
  * Embedded playback via iframe page or direct HLS URL
  * Private video encryption with Seal and wallet whitelist access control
  * TypeScript SDK and reference application for faster adoption

# 3. Target Audience

* **Primary User Personas**:
  * Developers building applications that need reliable video upload, playback, and management APIs
  * Teams that want wallet-aware access control and on-chain video ownership, and access rules
  * Startups, creator tools, and infrastructure teams that want a hosted default but value the ability to self-host instead of being locked into a centralized platform
  * End users who consume public video through standard web players and private video through a wallet-aware browser flow
* **Accessibility Goals**:
  * Reduce onboarding friction by making node101's hosted backend the default path for using the product, while preserving a self-hosting option for teams that want their own deployment
  * Ensure public video playback works through standard HLS players with no extra user steps beyond normal video playback
  * Limit wallet-specific UX to private-video access flows, where the wallet is only required to prove access rights and unlock client-side decryption
  * Provide documentation, SDKs, and a reference application so teams can move from evaluation to production with minimal custom infrastructure work

# 4. Project Versions (Iterative Approach)

node101's commercial path is the hosted backend and managed support. Hosted pilot usage begins with the MVP, with pricing and packaging refined after the product has been validated with real users.

## Version 1: Minimum Viable Product (MVP)

Version 1 focuses on the complete public-video lifecycle and the first hosted experience for developers.

* **Developer** shall be able to register a project, receive an API key, deposit SUI, and view available balance and bandwidth quota.
  * Testing plan: Move unit tests for account creation and deposits, API integration tests for registration and authentication, manual wallet deposit verification on Sui testnet.
* **Developer** shall be able to upload a video, which is then transcoded into HLS and stored on Walrus, and receive playback and embed URLs.
  * Testing plan: Automated integration tests for upload sessions and Walrus registration, manual end-to-end upload checks, media validation on generated manifests and segments.
* **Developer** shall be able to distribute a public video either through a direct playback URL or through an iframe-based embedded player.
  * Testing plan: API and playback integration tests, cache hit/miss validation, manual browser playback tests across desktop and mobile.
* **Developer** shall be able to rely on automatic blob renewal when balance is sufficient, and shall be able to delete a video and receive storage reclaim plus any SUI returned when the related on-chain objects are deleted.
  * Testing plan: Automated tests for renewal eligibility and deletion flows, cache invalidation tests, manual verification of expired and deleted asset behavior.

## Version 2: Enhanced Features

Version 2 expands the MVP with private video workflows, stronger developer tooling, and broader pilot readiness.

* **Developer** shall be able to upload a private video, define an on-chain whitelist, and have HLS assets encrypted with an AES key that is itself encrypted through Seal.
  * Testing plan: Integration tests for private upload registration and whitelist persistence, encryption/decryption validation for manifests and segments, manual secure playback checks.
* **Whitelisted end user** shall be able to connect a wallet once, prove access through Seal, obtain the decryption key in-browser, and stream decrypted video client-side without the backend ever seeing the plaintext key.
  * Testing plan: Manual wallet-auth playback tests, negative tests for non-whitelisted users, automated checks for private metadata responses and access denial paths.
* **Developer** shall be able to manage platform actions through a TypeScript SDK and validate implementation patterns through a reference application.
  * Testing plan: SDK unit and integration tests, reference app end-to-end smoke tests, onboarding walkthroughs with pilot developers.
* **Pilot customer or design partner** shall be able to choose between node101's hosted backend and the self-hosted deployment path while using the same protocol and application flows.
  * Testing plan: Deployment validation in hosted and self-hosted environments, documentation walkthrough tests, feedback sessions with pilot teams.

## Future Versions: Long-Term Vision

If adoption and resources justify continued expansion, future versions will extend delivery quality, monetization options, and ecosystem integrations.

* **Developer** shall be able to publish adaptive bitrate streams, custom branded playback experiences, and domain-specific embed surfaces.
  * Testing plan: Cross-device playback tests, load and performance testing, manual QA on branded embed configurations.
* **Developer** shall be able to create richer access policies such as token-gated playback, analytics events, and webhook-driven workflow integrations.
  * Testing plan: Contract and API integration tests, event delivery tests, staged rollout validation with pilot projects.
* **node101 hosted customers** shall be able to use a more fully managed offering with support, operational tooling, and potential usage-based pricing while the open-source core remains available for self-hosters.
  * Testing plan: Operational readiness reviews, billing and metering tests if introduced, pilot feedback analysis before wider rollout.

# 5. Technology and Tools

The stack below reflects the current design direction from the architecture and sequence diagrams. These are intended implementation choices, not irreversible commitments, but they are specific enough to guide execution.

* **Frontend**:
  * *React / Next.js for the reference application, hosted onboarding surface, and embed/player pages*
    * Decision rationale: Strong fit for building developer dashboards, documentation-friendly app surfaces, hosted embed endpoints, and wallet-aware browser flows with a mature ecosystem.
  * *HLS.js and/or Video.js for browser playback*
    * Decision rationale: Standard tooling for HLS playback in modern browsers and a practical fit for the iframe embed flow documented in the diagrams.
* **Backend**:
  * *TypeScript / Node.js services for API endpoints, upload orchestration, playback aggregation, renewal scheduling, and bandwidth accounting*
    * Decision rationale: A strong fit for integration-heavy services, shared language with the SDK, and practical handling of streaming APIs and background jobs.
  * *FFmpeg for transcoding and HLS packaging*
    * Decision rationale: Widely used, reliable tooling for converting uploads into HLS manifests and segments suitable for Walrus storage and standard player playback.
  * *Docker for deployment packaging*
    * Decision rationale: Keeps the stack portable across self-hosted environments and node101's hosted backend infrastructure.
* **Blockchain Tools**:
  * *Move smart contracts on Sui for developer accounts, balances, video metadata, bandwidth state, and whitelists*
    * Decision rationale: Keeps ownership, billing-related state, and access rules on-chain where they are transparent and programmable.
  * *Sui TypeScript SDK and wallet integration*
    * Decision rationale: Supports developer deposits, contract interactions, and wallet-based authorization for private playback flows.
  * *Walrus storage tooling for HLS manifests, segments, and deletable blobs*
    * Decision rationale: Walrus is the core storage layer for public and private video assets, including deletable storage and renewal flows.
  * *Seal SDK for private key access and client-side decryption flows*
    * Decision rationale: Enables wallet-authorized access to encrypted content keys without exposing private video keys to the backend.
* **Database**:
  * *PostgreSQL for off-chain metadata, indexing, upload-session state, and operational records*
    * Decision rationale: Useful for the developer experience and hosted operations without replacing on-chain ownership or Walrus storage.
  * *Redis for manifest/segment caching and short-lived coordination state*
    * Decision rationale: Matches the playback design where the backend aggregates Walrus reads, caches frequently requested assets, and reduces repeated fetch cost and latency.
* **Other Integrations**:
  * *GitHub for source control and CI/CD workflows*
    * Decision rationale: Supports open-source development, review workflows, and automated validation.
  * *Self-hosted deployment path plus node101-operated hosted backend*
    * Decision rationale: Preserves decentralization and portability while still offering a lower-friction default path for teams adopting the product.

# 6. Timeline

The timeline below groups the current engineering milestones into funding-oriented phases that leave room for Walrus feedback after each major release. Phase 1 establishes the public MVP foundation, Phase 2 completes the full public lifecycle and introduces the private-video beta, and Phase 3 packages the system for pilots and external adoption. Budget values are intentionally left blank for later discussion, but the milestone conditions are written to support future disbursement conversations.

## Phase 1: Research, Architecture, and Public MVP - {Budget}

* **Duration**: 4-6 weeks
* **Team**: To be finalized
* **Key Deliverables**:
  * Competitive analysis of current video infrastructure options and positioning for Walrus-native video tooling
  * Finalized architecture covering API-key onboarding, upload flow, contract design, Walrus blob lifecycle, playback aggregation, and hosted versus self-hosted deployment
  * Sui smart contracts for developer accounts, balances, video metadata, and whitelist-ready structures
  * Public-video MVP covering registration, SUI deposit flow, chunked upload, HLS transcoding, Walrus storage, playback URLs, and basic cache-backed playback
* **Review Gate**:
  * Architecture review and MVP demo with Walrus feedback before Phase 2 work is locked
* **Draft Milestone Conditions**:
  * A developer can register, receive an API key, deposit SUI, upload a video, and receive a valid playback URL
  * A public video can be streamed end-to-end from Walrus through the backend playback layer

## Phase 2: Playback Hardening and Private Video Beta - {Budget}

* **Duration**: 4-6 weeks
* **Team**: To be finalized
* **Deliverables**:
  * Bandwidth quota enforcement with developer top-up recovery
  * Batched bandwidth accounting from backend playback activity back to Sui
  * Auto-renewal scheduler for expiring blobs
  * Developer-initiated deletion with cache invalidation, Walrus storage reclaim, and correct handling of any SUI returned during deletion
  * Embed delivery via iframe page and direct HLS option
  * Private video pipeline with AES-encrypted HLS assets, Seal-encrypted content keys, and wallet whitelist access
* **Review Gate**:
  * Private-video beta review with Walrus, including access-control behavior and playback UX feedback
* **Draft Milestone Conditions**:
  * A whitelisted wallet can play a private video successfully through client-side decryption
  * A non-whitelisted wallet is denied access cleanly
  * Deletion and renewal behaviors match the documented Walrus and Sui lifecycle

## Phase 3: SDK, Pilot Rollout, and Feedback Iteration - {Budget}

* **Duration**: 2-4 weeks
* **Team**: To be finalized
* **Deliverables**:
  * TypeScript SDK covering core developer flows
  * Reference application demonstrating public and private video workflows
  * Self-hosting documentation and deployment guide
  * Hosted backend pilot onboarding for early adopters
  * Product and documentation iteration based on pilot feedback
* **Review Gate**:
  * Pilot feedback review with Walrus and adjustment of milestone/disbursement conditions based on real usage
* **Early Traction Targets**:
  * 2-3 pilot developer teams onboarded
  * 1 hosted pilot integration serving real video traffic
  * 1 external self-hosted deployment completed outside the core team
  * At least 20 videos uploaded across public and private flows
  * At least 500 successful playback sessions across pilot environments

## Phase 4: Future Iterations - {Budget}

* **Duration**: Ongoing
* **Team**: To be finalized
* **Focus Areas**:
  * Adaptive bitrate and delivery-quality improvements
  * Token-gated access policies and richer on-chain permission models
  * Analytics, webhooks, and operational tooling
  * Hosted product refinement, support workflows, and packaging based on pilot demand

## Contingency & Feedback Loops

* **Buffer Time**: Reserve 1-2 additional weeks between phases for integration issues, contract feedback, and deployment hardening where needed.
* **Feedback**: Treat each phase review gate as a formal checkpoint with Walrus so scope, milestone conditions, and success metrics can be refined before the next phase begins.
* **Iteration Principle**: Prefer shipping an MVP and beta to real users early, then use traction and feedback to shape later milestones rather than locking the entire roadmap upfront.

# 7. Budget Request Overview

**Total Requested Budget:**

**$[To be finalized]**

*Briefly describe the budget intent:*
This section is intentionally left open while scope and milestone conditions are being discussed with Walrus. The eventual budget is expected to cover core engineering time, design and documentation work, infrastructure and storage costs, testing environments, pilot support, and the tools required to deliver the phases above. The milestone structure in this document is intended to support a later conversation about concrete budget disbursement conditions.

# 8. Past Demonstration of Technical Expertise

This section is intentionally left as a placeholder until the team inserts prior projects, repositories, products, hackathon work, and individual bios that best represent node101's technical background.

* [Insert prior open-source repository or product link] - summarize the relevant backend, blockchain, or developer-platform work completed by the team
* [Insert prior smart contract, SDK, or infrastructure project] - describe which team members owned architecture, implementation, and maintenance
* [Insert hackathon, grant, or production deployment reference] - note outcomes, users, or recognition where applicable
* [Insert team bios or credentials] - summarize the experience most relevant to shipping this project
