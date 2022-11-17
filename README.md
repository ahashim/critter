# 🦔 Critter

### A peer-to-peer microblogging NFT platform for EVM-like blockchains

- [Core Concepts](#core-concepts)
- [Installation](#installation)
- [Architecture](#architecture)
- [Project Roadmap](#project-roadmap)

## Core concepts

Given the advertising and data-mining middlemen of traditional digital content,
and the emerging NFT market for said content, Critter was born.

Utilizing ownership fundamentals and financial incentives, Critter consolidates
the full content pipeline – creation, virality, and reward. Through that
process, it aims to empower makers to leverage the possibilities of blockchain
to forge a new, more transparent system that can serve them directly.

- Every address is a unique user.
- Users can follow one another.
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
- Every squeak has a "virality" score that is tracked on-chain.
- When a squeak goes "viral", future profits from that point on are split
  among the owner and those who helped it go viral (likers & resqueakers).

## Installation

The Critter monorepo and all of its submodules can be installed by cloning the
project recursively:

```zsh
git clone https://github.com/ahashim/critter.git --recursive
```

## Architecture

Critter is built using the
[CQRS pattern](https://martinfowler.com/bliki/CQRS.html):

- Webserver server-side renders the frontend.
- Frontend posts directly to the smart-contracts via
  [ethers.js](https://docs.ethers.io/v5/)
- Indexer ingests & writes events monotonically from the contract into the
  database.
- Webserver will serve read requests from the frontend via the database.

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
- Deploying to an L2 such as [zkSync](https://portal.zksync.io/) is the
  currently the cheapest option in regards to fees & marketplace-liquidity.

#### Server

- Server written in Golang.
  - Using [Ent ORM](https://entgo.io/) to make graph traversal across followers
    easier to manage.
  - Server side rendering via go templates.
- Indexer will be a separate process spawned from the server repo (to share
  config & ORM objects).
- Local LRU cache layer for reading high volume squeak content via redis.
- Session store via redis.

#### Client

- Initial desktop & mobile user interface written in HTML/CSS/Typescript using:
  - [htmx](https://htmx.org/)
  - [alpinejs](https://alpinejs.dev/)
  - [tailwind](https://tailwindcss.com/)

#### Cache

- Redis for in-memory cache layer.
  - Can be used to store sessions.

#### Database

- All contract events emitted on-chain will be replicated the database to
  read from via the ORM.
- A single postgres instance with a replica should suffice.

#### Search

- Search cluster will be powered by [Typesense](https://typesense.org)
  - Uses [Raft algorithm](https://raft.github.io/) to maintain durability, so
    3 nodes are sufficient.
- API keys generated on a per client basis to directly have clients hit search
  servers (with limited permissions).

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
- [x] Develop & implement "virality" algorithm.
- [x] Scout & scout pool payments.
- [x] Treasurer ability to manage platform fees for different interactions.
- [x] Account moderation via account statuses.
- [x] Relationships: following & followers users.
- [x] Relationships: blocking & unblocking users.
- [x] Harden contract with [security best practices](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/).
- [ ] Fuzz testing with [Echidna](https://github.com/crytic/echidna).
- [ ] Deploy to an EVM compatible layer 2 solution [zkSync](https://portal.zksync.io/).
  - Ensure its deployed via UUPS upgradeable proxy.
- [ ] Server controller logic + frontend design.
- [ ] Public API.
- [ ] Auction mechanism to bid on posted squeaks.
  - Ideally [Vickrey auctions](https://github.com/JoWxW/Vickrey-Auction/blob/master/contracts/VickreyAuction.sol).
- [ ] Media support for squeaks via IPFS (images, video, documents,
      etc&hellip;).
  - Can use a pinning service such as [Pinata](https://www.pinata.cloud/) for
    writes, and [Cloudflare IPFS](https://cloudflare-ipfs.com/ipns/ipfs.io/)
    for reads.
- [ ] Support for transaction-hash based generative art 🎨.
