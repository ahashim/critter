# 🦔 Critter

### A peer-to-peer microblogging NFT platform for EVM-like blockchains

Given the advertising and data-mining middlemen of traditional digital content,
the emerging NFT market for said content, and the crypto startups merely
mimicking the same old dynamics, Critter was born.

Utilizing ownership fundamentals and financial incentives, Critter consolidates
the full content pipeline – creation, virality, and reward. Through that
process, it aims to empower makers to leverage the possibilities of blockchain
to forge a new, more transparent system that can serve them directly.

- [Core Concepts](#core-concepts)
- [Project Roadmap](#project-roadmap)
- [Architecture](#architecture)

## Core concepts

- Every address is a unique username.
- Every post (called a squeak) is an NFT.
- Squeaks can be bought & sold via a bidding price discovery mechanism.
- Once the author sells their squeak, the ownership is transferred to a new
  user.
- Interactions (like, dislike, & resqueaks) cost a fee.
- Fees for positive interactions (like & resqueak) are paid out to the **owner**
  of the squeak (not necessarily the original author).
- Negative interactions (dislike) or undoing positive interactions will be paid
  to Critter.
- Only owners can delete squeaks.
- Deleting a squeak costs a fee of `blocks elapsed x deletion fee`.
- Every squeak has a "virality" coefficient that is tracked on-chain.
- When a squeak goes "viral", future profits from that point on are split
  among the owner and everbody who has positively interacted up until that
  point.

## Project Roadmap

- [x] ERC721 interface compatability.
- [x] Create account.
- [x] Update username.
- [x] Post squeak.
- [x] Delete squeak.
- [x] Upgradeable contracts via [UUPS proxy pattern](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable).
- [x] `CRTTR` console for local contract interaction.
- [x] Platform Fees + P2P user payments.
- [x] Liking a squeak & undoing a like.
- [x] Disliking a squeak & undoing a dislike.
- [x] Resqueaking & undoing a resqueak.
- [x] Adding a `withdraw` function to transfer funds out from the treasury.
- [ ] Adding functions to update platform fees.
- [ ] Develop & implement "virality" algorithm.
- [ ] Account funding via ERC-20 compatible tokens.
- [ ] Auction mechanism to bid on posted squeaks
  - Ideally [Vickrey auctions](https://github.com/JoWxW/Vickrey-Auction/blob/master/contracts/VickreyAuction.sol).
- [ ] Moderation
  - Implement basic moderation status on account structs:
    `{Active, Suspended, Banned}`
  - Off-chain content reporting.
- [ ] Media support for squeaks via IPFS (images, video, documents,
      etc&hellip;).
  - Can use a pinning service such as [Pinata](https://www.pinata.cloud/) for
    writes, and [Cloudflare IPFS](https://cloudflare-ipfs.com/ipns/ipfs.io/)
    for reads.
- [ ] Support for transaction-hash based generative art 🎨.
- [ ] Harden contract with [security best practices](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/).
- [ ] Fuzz testing with [Echidna](https://github.com/crytic/echidna).
- [ ] Deploy to an EVM compatible layer 2 solution such as [zkSync](https://portal.zksync.io/).

## Architecture

#### Algorithms

- Design & test algorithms with the help of matplotlib + numpy (and some "good
  old-fashioned" calculus).

#### Smart Contracts

- All smart contracts are written in Solidity & deployed to an EVM compatible
  Layer 2 solution.
- Contracts are pausable & upgradeable in order to safely add features & fix
  bugs in an iterative manner.
  - Existing storage variables will always remain, and only be appended to.
  - New functionality will be added via additional contracts and/or deploying
    new versions of existing contract methods.
- Deploying to [zkSync](https://portal.zksync.io/) is the currently the cheapest
  option in terms of transaction fees.

#### Indexer

- Indexer written in Golang to watch on-chain event logs & create entries from
  them directly to a database.
  - Will send to the `fanout-service` on the server for further data
    validation/sanitization.
  - Upgrade this to use a message queue after indexing (w/ built-in
    backpressure), which then sends data out to the `fanout-service` service
    when server/database load kicks up.
- Does not need to be highly available (one instance will suffice, and can run
  locally).
- Deploy to a [Digital Ocean](https://www.digitalocean.com/) droplet.

#### Database

- All user/squeak data not-stored on chain will be in the database.
- Database cluster will likely be MySql/MariaDB to optimize for reads.
- `tokenID`'s can serve as initial primary keys.
  - Will have to update this to `sha4(chainId + tokenId)` when database requires
    partitioning.
- Deploy across regions with [Fly.io](https://fly.io/).

#### Client

- Initial desktop user interface written in HTML/CSS/Typescript using:
  - [htmx](https://htmx.org/)
  - [tailwind](https://tailwindcss.com/)
  - [alpinejs](https://alpinejs.dev/)
- Will use SSR from Golang server + websockets to avoid marshalling/routing
  layers.

#### Server

- Server written in Golang.
  - Using [Gin web framework](https://gin-gonic.com/)
  - Handles client/service API key generation via an `auth-service`.
    - Can likely use `redis` or another in-memory k/v store to handle session
      keys.
  - Handles data routing from the indexer/message-queue via a `fanout-service`.
  - Local LRU cache layer for reading high volume squeaks.
- Deploy with [Fly.io](https://fly.io/).

#### Search

- Search cluster will be powered by [Typesense](https://typesense.org)
  - Uses [Raft algorithm](https://raft.github.io/) to maintain durability, so
    3 nodes might be enough.
- API keys generated on a per client basis via `auth-service` to directly have
  clients hit search servers (with limited permissions).
- Deploy across regions with [Fly.io](https://fly.io/).
