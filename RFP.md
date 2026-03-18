# {RFP name} \- {Team name}

# 1\. Project Name

{Project Name}

# 2\. Purpose

* **Current Challenge**: Existing video platforms already solve upload, transcoding, and playback well, but they do so inside a vendor-controlled backend model. That model is convenient, but it creates lock-in around storage, delivery, metadata, access control, and application logic. Developers can build on top of the provider's APIs, but the underlying system remains tightly coupled to one vendor, which makes portability and deeper control difficult.
* **Proposed Solution**: node101 provides a hosted video infrastructure platform as the default developer experience, while keeping the system open underneath. Developers onboard through node101's hosted API, receive API keys, deposit SUI, upload videos, transcode them into HLS, store them on Walrus, and register their lifecycle on Sui. Public videos are streamed through a cache-backed aggregator, and private videos use Seal so playback keys are released only to approved wallets and decryption happens client-side. The important point is not that teams are expected to self-host or replace the backend, but that they retain that option because the underlying data and state are not trapped inside node101's infrastructure.
* **Key Features**
  * Developer onboarding with API keys and SUI-funded balances
  * Chunked upload flow with HLS transcoding and Walrus blob storage
  * Public playback through node101's hosted aggregator, with a self-hosted path available for teams that want to run their own backend
  * Bandwidth quota enforcement backed by on-chain balances
  * Scheduled blob auto-renewal for retained content
  * Video deletion with Walrus storage reclaim and correct handling of any SUI returned during deletion
  * Embedded playback via iframe page or direct HLS URL
  * Private video encryption with Seal and wallet whitelist access control
  * TypeScript SDK and reference application for faster adoption

# 3\. Target Audience

* **Primary User Personas**
  * Developers building applications that need reliable video upload, playback, and management APIs
  * Teams that want wallet-aware access control and on-chain video ownership, and access rules
  * Startups, creator tools, and infrastructure teams that want a hosted default but value the ability to self-host instead of being locked into a centralized platform
  * End users who consume public video through standard web players and private video through a wallet-aware browser flow
* **Accessibility Goals**
  * Reduce onboarding friction by making node101's hosted backend the default path for using the product, while preserving a self-hosting option for teams that want their own deployment
  * Ensure public video playback works through standard HLS players with no extra user steps beyond normal video playback
  * Limit wallet-specific UX to private-video access flows, where the wallet is only required to prove access rights and unlock client-side decryption
  * Provide documentation, SDKs, and a reference application so teams can move from evaluation to production with minimal custom infrastructure work

# 4\. Project Versions (Iterative Approach)

node101's commercial path is the hosted backend and managed support. The hosted product becomes available with the MVP, with broader pilot onboarding and pricing refinement happening after initial user validation.

## **Version 1: Minimum Viable Product (MVP)**

Version 1 focuses on the complete public-video lifecycle and the first hosted experience for developers.

* Developer shall be able to register a project, receive an API key, deposit SUI, and view available balance and bandwidth quota.
  * Testing plan: Move unit tests for account creation and deposits, API integration tests for registration and authentication, manual wallet deposit verification on Sui testnet.
* Developer shall be able to upload a video, which is then transcoded into HLS and stored on Walrus, and receive playback and embed URLs.
  * Testing plan: Automated integration tests for upload sessions and Walrus registration, manual end-to-end upload checks, media validation on generated manifests and segments.
* Developer shall be able to distribute a public video either through a direct playback URL or through an iframe-based embedded player.
  * Testing plan: API and playback integration tests, cache hit/miss validation, manual browser playback tests across desktop and mobile.
* Developer shall be able to rely on automatic blob renewal when balance is sufficient, and shall be able to delete a video and receive storage reclaim plus any SUI returned when the related on-chain objects are deleted.
  * Testing plan: Automated tests for renewal eligibility and deletion flows, cache invalidation tests, manual verification of expired and deleted asset behavior.

##

## **Version 2: Enhanced Features**

## Version 2 expands the MVP with private video workflows, stronger developer tooling, and broader pilot readiness.

* Developer shall be able to upload a private video, define an on-chain whitelist, and have all HLS segments and the manifest encrypted with a single per-video AES key. That AES key is encrypted through Seal using an identity tied to the video's access policy, and the encrypted key is stored on-chain with the video metadata.
  * Testing plan: Integration tests for private upload registration and whitelist persistence, encryption/decryption validation for manifests and segments, verification that the backend does not retain or expose the plaintext AES key after the Seal encryption step, manual secure playback checks.
* Whitelisted end user shall be able to open the hosted player page, sign once per session with their wallet to prove access, and stream the video with all decryption happening client-side. The backend never sees the plaintext key.
  * Testing plan: Manual playback tests for whitelisted and non-whitelisted wallets, session key expiry tests, end-to-end decryption validation through the custom hls.js loader.
* Developer shall be able to manage platform actions through a TypeScript SDK and validate implementation patterns through a reference application.
  * Testing plan: SDK unit and integration tests, reference app end-to-end smoke tests, onboarding walkthroughs with pilot developers.

## **Future Versions: Long-Term Vision**

* Developer shall be able to upload a video and have it automatically transcoded into multiple quality renditions, with the player switching bitrate based on the viewer's connection speed.
  * Testing plan: Cross-device playback tests, bitrate switching validation under simulated network conditions, segment continuity checks.
* Developer shall be able to gate a video behind NFT ownership or token balance instead of a manual wallet whitelist, using the existing Seal and wallet integration.
  * Testing plan: Contract and API integration tests for token-gated access policies, positive and negative access tests with qualifying and non-qualifying wallets.
* Developer shall be able to view per-video analytics including view counts, bandwidth consumed, and playback activity to understand usage and SUI spend.
  * Testing plan: Analytics data accuracy tests, aggregation validation, API integration tests.
* Developer shall be able to receive webhook notifications for platform events such as upload completion, blob renewal, and bandwidth threshold alerts.
  * Testing plan: Event delivery tests, retry and failure handling validation, end-to-end integration tests with external endpoints.

# 5\. Technology and Tools

* # Frontend:

  * React / Next.js for the reference application, hosted onboarding surface, and embed/player pages
    * Decision rationale: Strong fit for building developer-facing product surfaces, hosted player pages, and wallet-aware browser flows with a mature ecosystem.
  * HLS.js for private playback and in-browser decryption
    * Decision rationale: Gives the frontend a JavaScript-controlled HLS pipeline so encrypted manifests and segments can be decrypted client-side before playback.
  * Video.js for hosted player UI and iframe embeds
    * Decision rationale: Provides a familiar open-source player layer for public playback and embedded player experiences.
* Backend:
  * HonoJS on Cloudflare Workers for API endpoints, authentication, playback routing, and lightweight orchestration
    * Decision rationale: Strong fit for a TypeScript-first hosted control plane, with fast global request handling and a runtime model that maps well to API and edge delivery workloads.
  * Dockerized Node.js worker with FFmpeg for transcoding, HLS packaging, and scheduled background jobs
    * Decision rationale: Better fit for CPU-intensive and longer-running media processing tasks, while keeping the media pipeline portable across node101-hosted and self-hosted deployments.
  * Docker for packaging self-hosted deployments and background processing services
    * Decision rationale: Keeps the backend portable and reproducible across node101's hosted environment and self-hosted installations.
* Blockchain Tools:
  * Sui smart contracts and the Sui TypeScript SDK for developer balances, video state, and wallet-based interactions
    * Decision rationale: Keeps the core application state on-chain while letting the hosted product and SDK interact with it through a standard TypeScript stack.
  * Walrus for HLS manifests, segments, and deletable video blobs
    * Decision rationale: Walrus is the storage layer for public and private video assets, including deletion and renewal flows.
  * Seal SDK for private-video key protection and client-side decryption
    * Decision rationale: Allows authorized users to unlock the per-video key in the browser without exposing plaintext keys to the backend.
* Database:
  * PostgreSQL for off-chain metadata, upload-session state, and operational indexing
    * Decision rationale: Keeps the off-chain data model portable across node101-hosted and self-hosted deployments, while leaving core ownership and access state on Sui and Walrus.
  * Cloudflare Hyperdrive for low-latency database access from Workers in the hosted deployment
    * Decision rationale: Improves connectivity between Cloudflare Workers and the primary SQL database without forcing the product into a Cloudflare-only database model.
* Other Integrations:
  * Cloudflare Cache for public playback response caching
    * Decision rationale: Fits the hosted playback architecture by caching frequently requested manifests and segments close to end users.
  * Cloudflare Queues for background job dispatch and buffering
    * Decision rationale: Useful for decoupling upload completion, media processing, renewal checks, and other asynchronous workflows from request handling.

6\. Timeline

**Phase 1: Research, Architecture, and Public MVP \- {Budget}**

* Duration: 4-6 weeks
* Team: To be finalized
* Key Deliverables:
  * Competitive analysis of current video infrastructure options and positioning for Walrus-native video tooling
  * Finalized architecture covering API-key onboarding, upload flow, contract design, Walrus blob lifecycle, playback aggregation, and hosted versus self-hosted deployment
  * Sui smart contracts for developer accounts, balances, video metadata, and whitelist-ready structures
  * Public-video MVP covering registration, SUI deposit flow, chunked upload, HLS transcoding, Walrus storage, playback URLs, and basic cache-backed playback
* Review Gate:
  * Architecture review and MVP demo with Walrus feedback before Phase 2 work is locked
* Draft Milestone Conditions:
  * A developer can register, receive an API key, deposit SUI, upload a video, and receive a valid playback URL
  * A public video can be streamed end-to-end from Walrus through the backend playback layer

**Phase 2: Playback Hardening and Private Video Beta \- {Budget}**

* Duration: 4-6 weeks
* Team: To be finalized
* Deliverables:
  * Bandwidth quota enforcement with developer top-up recovery
  * Batched bandwidth accounting from backend playback activity back to Sui
  * Auto-renewal scheduler for expiring blobs
  * Developer-initiated deletion with cache invalidation, Walrus storage reclaim, and correct handling of any SUI returned during deletion
  * Public video embed delivery via iframe page and direct HLS option
  * Private video pipeline with AES-encrypted HLS assets, Seal-encrypted content keys, wallet whitelist access, and a hosted private player flow
* Review Gate:
  * Private-video beta review with Walrus, including access-control behavior and playback UX feedback
* Draft Milestone Conditions:
  * A whitelisted wallet can play a private video successfully through client-side decryption
  * A non-whitelisted wallet is denied access cleanly
  * Deletion and renewal behaviors match the documented Walrus and Sui lifecycle

**Phase 3: SDK, Pilot Rollout, and Feedback Iteration \- {Budget}**

* Duration: 2-4 weeks
* Team: To be finalized
* Deliverables:
  * TypeScript SDK covering core developer flows
  * Reference application demonstrating public and private video workflows
  * Self-hosting documentation and deployment guide
  * Hosted backend pilot onboarding for early adopters
  * Product and documentation iteration based on pilot feedback
* Review Gate:
  * Pilot feedback review with Walrus and adjustment of milestone/disbursement conditions based on real usage
  * Validation that hosted and self-hosted deployments support the same core workflows against the same Walrus and Sui state
* Early Traction Targets:
  * 1 hosted pilot integration serving real video traffic
  * At least 5 videos uploaded across public and private flows
  * At least 100 successful playback sessions across pilot environments

**Phase 4: Future Iterations \- {Budget}**

* Duration: Ongoing
* Team: To be finalized
* Focus Areas:
  * Adaptive bitrate and delivery-quality improvements
  * Token-gated access policies and richer on-chain permission models
  * Analytics, webhooks, and operational tooling
  * Hosted product refinement, support workflows, and packaging based on pilot demand

**Contingency & Feedback Loops**

* Buffer Time: Reserve 1-2 additional weeks between phases for integration issues, contract feedback, and deployment hardening where needed.
* Feedback: Treat each phase review gate as a formal checkpoint with Walrus so scope, milestone conditions, and success metrics can be refined before the next phase begins.
* Iteration Principle: Prefer shipping an MVP and beta to real users early, then use traction and feedback to shape later milestones rather than locking the entire roadmap upfront.

# 7\. Budget Request Overview

**Total Requested Budget:**

**$\[Insert Total Amount\]**

*Briefly describe the budget intent:*
“This budget covers core team development time, design resources, infrastructure costs (e.g., hosting, storage, APIs), and any tools or licenses needed for the proposed timeline.”

# 8\. Past Demonstration of Technical Expertise

*Use the space below to provide any past work that demonstrates the team’s or team members’ technical experience and skills.*

* (e.g. link to Hackathon GitHub repo \- Team members A and B built this project at Hackathon X where they won prize W)
* (e.g. link to product app \- Team members C designed, built, and maintained the smart contracts)
* (e.g. Team members D and E graduated with their B.S in CS from University Z)