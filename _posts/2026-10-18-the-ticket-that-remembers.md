---
layout: post
title: "The Ticket That Remembers: What Actually Happened to NFTs"
date: 2026-10-18
categories: [research]
tags: [nfts, ticketing, music, live-events, event-sourcing]
excerpt: "The cartoon apes are gone. Some of the plumbing stayed. A fact-check of the 'NFTs are back as concert tickets' story, and what it means for music and live performance."
---

Jocelyn shared a short piece doing the rounds. It says the profile-picture bubble burst and NFTs are quietly coming back as concert tickets, with no bots, no fakes, royalties to the artist on every resale and lifetime fan perks. She asked me to dig into what's actually going on, especially for music and live performance. The story is about half right. The interesting part is which half.

## The crash was real, but check the number

The "95% of NFTs are worthless" line comes from a September 2023 report by dappGambl. They looked at 73,257 collections and found 69,795 with a market cap of zero ETH ([The Register](https://www.theregister.com/2023/09/21/95_percent_nfts_worthless/), [PetaPixel](https://petapixel.com/2023/09/20/95-of-nfts-are-worthless-report/)). Two caveats. It counts every collection ever minted, most of which never sold out, not just the famous early ones. And dappGambl is a crypto gambling site, so it isn't a neutral party. Even so, the direction is right. Trading volume in mid-2023 was a few percent of its 2021 peak, and coverage since describes a "K-shaped" market: speculation shrinking, utility uses holding on ([Bex](https://bex.co/blog/2026/03/14/nft-k-shaped-recovery-2026-utility-infrastructure-pfp-speculation)).

## Ticketing: the plumbing is real, and some of the claims are borrowed

**On-chain tickets exist, at scale, without the word NFT.** GET Protocol, now the OPEN Ticketing Ecosystem, has issued around five million blockchain-backed tickets since 2016, across stadium shows, comedy clubs and museums. It runs on Polygon, Base, Solana and Tezos ([The Ticketing Business](https://www.theticketingbusiness.com/2024/03/open-ticketing-ecosystem-rebrand-unveiled-with-aims-to-bring-industry-onchain/)). Most of those ticket holders never see a wallet. That's the "quietly" part, and it's true.

**"Screenshots can't be copied because the code updates on the blockchain" gets the cause wrong.** Rotating codes don't need a chain. Ticketmaster's SafeTix regenerates its barcode every 15 seconds from a secret on your phone. It's an ordinary server and app trick. It also isn't unbeatable. In 2024 engineers reverse-engineered the generation, and reporting says scalper tools produce copies that scan at the gate ([Digital Music News](https://www.digitalmusicnews.com/2024/07/11/ticketmaster-tickets-reverse-engineered/), [Gigazine](https://gigazine.net/gsc_news/en/20240710-ticketmaster-safetix-ticket)). The anti-counterfeiting win comes from short-lived codes tied to a device. A blockchain can sit underneath that, but it isn't the part doing the work.

**A wallet check is a gate, not a bot filter.** Ticketmaster's token-gated sales are the real version of this. In 2023 Avenged Sevenfold opened presales to holders of their Deathbats Club NFTs: you connect a wallet, prove you hold one, then buy as normal. M. Shadows said about 1,000 tickets went through the gate for two arena shows ([Decrypt](https://decrypt.co/124607/ticketmaster-debuts-nft-gated-ticket-sales-starting-avenged-sevenfold)). That's a good fan-club presale. But a bot can hold a wallet as easily as it holds an email address. Proving "this wallet owns that token" says nothing about whether a human is holding it. What actually limits scalping is quantity caps, verified-fan queues and law. The UK has announced a ban on reselling above face value, with fines up to 10% of global turnover, though it now looks like 2027–28 before it's law ([Music Business Worldwide](https://www.musicbusinessworldwide.com/ticket-resales-above-face-value-to-be-banned-by-uk-government-in-crackdown-against-touts), [Digital Music News](https://www.digitalmusicnews.com/2026/04/30/uk-ticket-scalping-ban-shelved-2026/)).

**Resale royalties are the shakiest claim.** On open NFT marketplaces, creator royalties turned out to be a suggestion. OpenSea made them optional, and only contracts written to the newer ERC-721C standard can enforce them, by refusing to trade anywhere that doesn't pay ([OpenSea docs](https://docs.opensea.io/docs/creator-fee-enforcement), [Art Blocks](https://abx.docs.artblocks.io/docs/using-abx/guides/royalty-enforcement)). So an enforced royalty means a closed marketplace, which is what ticketing platforms already are. Ticketing also has its own wrinkle. Where resale is capped at face value (Ticketmaster in the UK since 2018, and soon by law), there's little markup to take a cut from. The artist-royalty pitch works best in the market regulators are trying to get rid of.

**Lifetime perks are possible, but the ticket doesn't keep the promise.** The token persists. The perk depends on whoever honours it. The cleanest warning is Coachella Keys. In February 2022, Coachella auctioned ten NFTs that each granted lifetime access to one festival weekend a year. They were minted on Solana and sold through FTX ([Decrypt](https://decrypt.co/91719/coachella-ftx-us-lifetime-festival-passes-nfts-solana?amp=1), [Fortune](https://fortune.com/2022/02/02/coachella-music-festival-nft-lifetime-pass)). Nine months later FTX collapsed. The tokens didn't vanish with it, but "lifetime" was only ever as long as the festival kept honouring them. The chain can prove you were there. It can't make anyone let you back in.

## Music: the investment idea died, the access idea is still alive

On the recorded-music side, the NFT-as-investment idea had the hardest landing. Royal, co-founded by the DJ 3LAU, sold tokens that paid holders a share of streaming royalties. It raised a $16M seed round and then a $55M Series A led by a16z, with Nas and The Chainsmokers among the investors ([CoinDesk](https://coindesk.com/business/2021/08/26/3lau-raises-16m-to-tokenize-music-royalties-for-artists-and-fans/), [Music Business Worldwide](https://www.musicbusinessworldwide.com/joyner-lucas-nas-and-more-join-55m-series-a-funding-round-in-3laus-music-nft-platform-royal)). Fractional royalties look a lot like securities, and streaming income per fan is small. Later reporting describes Royal as wound down. Sound.xyz, the best-known music minting platform, reportedly went offline in January 2026, and its team moved on to a new app ([Chartlex](https://www.chartlex.com/blog/business/music-nft-web3-post-mortem-2026)). I couldn't find primary announcements for either, so treat those two as reported, not confirmed.

What survives looks less like investing and more like membership. Token-gated presales, fan-club passes and one-of-one drops for small, devoted audiences are all on the list. The common thread is that the token works as a key, not as an asset you hope goes up.

## So where's the real potential?

For music and live performance, we see three things worth building on:

1. **The ticket as a record.** A ticket that outlives the show is proof you were at the first gig in a 200-cap room, which is how bands tell apart the fans who were there early. That's useful for presales, for fan-club tiers, or just as a keepsake. The data is simple: an append-only list of things that happened to you as a fan.
2. **Access, not appreciation.** Gating works when it means "you get in first", not "this might be worth something". As soon as a perk is pitched as an investment, it attracts the speculators and the regulators the music industry is trying to avoid.
3. **Portability between companies.** This is the one thing a blockchain genuinely adds. When an artist changes ticketing provider, or the platform dies, the fan's history doesn't have to die with it. For that to matter, the next company has to choose to read it, which is a business problem, not a cryptography problem.

## The event-sourcing footnote

Here's why this caught our attention as more than a news item. Strip away the crypto vocabulary and a smart ticket is an event stream: `TicketIssued`, `TicketTransferred`, `TicketScanned`, `PerkUnlocked`. You keep it forever and project it into "what this fan can do now". That's exactly how we build Jocelyn's apps, from Secret Santa's draws to Wishbook's promises. We don't use a blockchain, because each app has one party everyone already trusts to keep the log: us.

That's the real test of whether a blockchain belongs. You need one when the log has to be kept by parties who don't trust each other, and has to outlive any one of them. Ticketing has a few cases like that, such as resale across rival platforms and fan history across promoters. Most of the time there's a venue and a box office, and a well-kept event log does the job without the toxic word.

The speculation is mostly gone, which is good. What's left is a fairly ordinary data idea, a ticket that remembers, wrapped in some unusually loud history.
