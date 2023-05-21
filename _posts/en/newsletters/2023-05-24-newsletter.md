---
title: 'Bitcoin Optech Newsletter #252'
permalink: /en/newsletters/2023/05/24/
name: 2023-05-24-newsletter
slug: 2023-05-24-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter describes research into zero-knowledge validity
proofs for Bitcoin and related protocols.  Also included is another entry
in our limited weekly series about mempool policy, plus our regular
sections describing updates to clients and services, new releases and
release candidates, and changes to popular Bitcoin infrastructure
projects.

## News

- **State compression with zero-knowledge validity proofs:** Robin Linus
  [posted][linus post] to the Bitcoin-Dev mailing list about a
  [paper][lg paper] he co-authored with Lukas George about using
  validity proofs to reduce the amount of state that a client needs to
  download in order to trustlessly verify future operations in a system.
  They first apply their system to Bitcoin.  They report having a
  prototype that proves the cumulative amount of proof of work in a
  chain of block headers and allows a client to verify that a particular
  block header is part of that chain.  This allows a client that
  receives multiple proofs to determine which demonstrates the most
  proof of work.

    They also have a suboptimal prototype which proves that all
    transaction state changes for the block chain respected the currency
    rules (e.g., how many bitcoins can be created by a new block, that
    each non-coinbase transaction must not create UTXOs with more
    bitcoin value than those it destroys (spends), and that a miner may
    claim any difference between the UTXOs destroyed in a block and
    those created).  A client receiving this proof and a copy of the
    current UTXO set would be able to verify that the set was accurate
    and complete.  They call this an _assumevalid_ proof after the
    [feature in Bitcoin Core][assumevalid] that optionally skips
    verification of scripts for older blocks when many contributors
    agree that their nodes all validated those blocks successfully.

    To minimize the complexity of their proof, they use a version of
    [utreexo][topic utreexo] with a hash function optimized for their
    system.  They separately suggest that combining their proof with a
    utreexo client will allow that client to begin operating as a full
    node almost immediately after downloading a very small amount of
    data.

    Regarding the usability of their prototypes, they write that "we
    have implemented the header chain proof and the assumevalid state
    proof as prototypes.  The former is feasible to prove, while the
    latter still requires performance improvements to prove
    reasonable-sized blocks".  They are also working on validating
    complete blocks, including scripts, but say that they need at least
    a 40x speed up for that to be viable.

    In addition to state compression of the Bitcoin block chain, they
    also describe a protocol that can be used for a
    client-side-validation token protocol similar to that used for Lightning Labs'
    Taproot Assets and some uses of RGB (see Newsletters [#195][news195
    taro] and [#247][news247 rgb]).  When Alice transfers to Bob some
    amount of a token, Bob needs to verify the history of every previous
    transfer of those specific tokens back to when they were created.
    In an ideal scenario, that history grows linearly with the number of
    transfers.  But if Bob wants to pay Carol an amount higher than he
    received from Alice, he will need to combine some of the tokens he
    received from Alice with some tokens he received in a
    different transaction.  Carol will then need to verify both the
    history going back through Alice and also the history of Bob's other
    tokens.  This is called a merge.  If merges happen often, the size
    of the history that needs to be verified approaches the size of the
    history of every transfer of that token between any users.  In
    comparative terms, in Bitcoin every full node verifies every
    transaction made by every user; in token protocols using client-side
    validation, that's not strictly required but eventually becomes
    effectively the case if merges are common.

    That means a protocol that can compress the state of Bitcoin can
    also be adapted to compress the state of a token's history, even one
    where merges are common.  The authors describe how they would
    accomplish that.  Their goal is to produce a proof that each
    previous transfer of the token followed the token's rules, including
    using their proofs for Bitcoin to prove that each previous transfer
    was anchored in the block chain.  Alice could then transfer the
    tokens to Bob and give him a short constant-sized proof of validity;
    Bob could verify the proof to know that the transfer happened at a
    certain block height and paid his token wallet, giving him exclusive
    control over the tokens.

    Although the paper frequently mentions additional research and
    development that can be performed,
    we find this to be encouraging progress towards a
    [feature][coinwitness] that Bitcoin developers have desired for over
    a decade.

## Waiting for confirmation #2: FIXME:glozow title

_A limited weekly series about transaction relay, mempool inclusion, and
mining transaction selection---including why Bitcoin Core has a more
restrictive policy than allowed by consensus and how wallets can use
that policy most effectively._

{% include specials/policy/en/02-cache-utility.md %}

## Changes to services and client software

*In this monthly feature, we highlight interesting updates to Bitcoin
wallets and services.*

FIXME:bitschmidty

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

- [Core Lightning 23.05][] is a release of the newest version of this LN
  implementation.  It includes support for [blinded payments][topic rv
  routing], version 2 [PSBTs][topic psbt], and more flexible feerate
  management among many other improvements.

- [Bitcoin Core 23.2][] is a maintenance release for the previous
  major version of Bitcoin Core.

- [Bitcoin Core 24.1][] is a maintenance release for the current
  version of Bitcoin Core.

- [Bitcoin Core 25.0rc2][] is a release candidate for the next major
  version of Bitcoin Core.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo], [Core
Lightning][core lightning repo], [Eclair][eclair repo], [LDK][ldk repo],
[LND][lnd repo], [libsecp256k1][libsecp256k1 repo], [Hardware Wallet
Interface (HWI)][hwi repo], [Rust Bitcoin][rust bitcoin repo], [BTCPay
Server][btcpay server repo], [BDK][bdk repo], [Bitcoin Improvement
Proposals (BIPs)][bips repo], [Lightning BOLTs][bolts repo], and
[Bitcoin Inquisition][bitcoin inquisition repo].*

- [Bitcoin Core #27021][] Implement Mini version of BlockAssembler to calculate mining scores FIXME:harding (hoping glozow or Xekyo will volunteer, though)

- [LND #7668][] adds the ability to associate up to 500 characters of
  private text with a channel when opening it and allows the operator to
  retrieve that information later, which may help them recall why they
  opened that particular channel.

- [LDK #2204][] adds the ability to set custom feature bits for
  announcing to peers, or for use when attempting to parse a peer's
  announcement.

- [LDK #1841][] implements a version of a security recommendation
  previously added to the LN specification (see [Newsletter
  #128][news128 bolts803]) where a node using [anchor outputs][topic
  anchor outputs] should not attempt to batch together inputs controlled
  by multiple parties when the transaction needs to be confirmed
  promptly.  This prevents other parties from being able to delay
  confirmation.

- [BIPs #1412][] updates [BIP329][] for [wallet label export][topic
  wallet labels] with a field to store key origin information.
  Additionally, the specification now suggests a label length limit of
  255 characters.

{% include references.md %}
{% include linkers/issues.md v=2 issues="27021,7668,2204,1841,1412" %}
[Core Lightning 23.05]: https://github.com/ElementsProject/lightning/releases/tag/v23.05
[bitcoin core 23.2]: https://bitcoincore.org/bin/bitcoin-core-23.2/
[bitcoin core 24.1]: https://bitcoincore.org/bin/bitcoin-core-24.1/
[bitcoin core 25.0rc2]: https://bitcoincore.org/bin/bitcoin-core-25.0/
[linus post]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2023-May/021679.html
[lg paper]: https://zerosync.org/zerosync.pdf
[news128 bolts803]: /en/newsletters/2020/12/16/#bolts-803
[news247 rgb]: /en/newsletters/2023/04/19/#rgb-update
[news195 taro]: /en/newsletters/2022/04/13/#transferable-token-scheme
[coinwitness]: https://bitcointalk.org/index.php?topic=277389.0
[assumevalid]: https://bitcoincore.org/en/2017/03/08/release-0.14.0/#assumed-valid-blocks