---
description: Block SDK Integration
title: ⚡️ Integrate the Block SDK
sidebar_position: 2
---

:::info Block SDK
**`The Block SDK`** will work out-of-the-box for chains that support `ABCI++` (Cosmos-SDK version of `0.47` or higher). There are no other dependencies required.

<!-- TODO: David update link once repo is renamed -->

The Block SDK is **open-source software** licensed under MIT. It is free to use, takes less than [20 mins to set up](https://github.com/skip-mev/pob#protocol-owned-builder), and has existing plug-and-play Lanes that work immediately!

:::

### 1. 💅 Determine the `lanes` desired

Visit our [Lane App Store](lanes/existing-lanes/default) and select the `lanes` you want, or [Build Your Own](lanes/build-your-own-lane). Skip currently supports the following `lanes`:

1. [MEV Lane](lanes/existing-lanes/mev) that allows for MEV recapture by auctioning off the top of block space.
2. [Free Lane](lanes/existing-lanes/free) that allows for transactions with certain properties (e.g. from trusted accounts or performing encouraged actions) to be included in the next block for free.
3. [Default Lane](lanes/existing-lanes/default) that accepts all other transactions.

### 2. ⚡️ Instantiate the Block SDK in base app

The `lanes` are ordered by priority and are configurable by developers. The first `lane` is the highest priority `lane` and the last `lane` is the lowest priority `lane`.

```go
// Set the block SDK mempool into the app.
// Create the lanes.
//
// NOTE: The lanes are ordered by priority. The first lane is the highest priority
// lane and the last lane is the lowest priority lane.
// Top of block lane allows transactions to bid for inclusion at the top of the next block.
//
// block.BaseLaneConfig is utilized for basic encoding/decoding of transactions, configuring how much
// block space the lane can consume, and how many transactions the lane can retain.
tobConfig := block.BaseLaneConfig{
    Logger:        app.Logger(),
    TxEncoder:     app.txConfig.TxEncoder(),
    TxDecoder:     app.txConfig.TxDecoder(),
    // the desired portion of total block space to be reserved for the lane. a value of 0
    // indicates that the lane can use all available block space.
    MaxBlockSpace: sdk.ZeroDec(),
}
// Top of block lane allows transactions to bid for inclusion at the top of the next block.
tobLane := auction.NewTOBLane(
    tobConfig,
    // the maximum number of transactions that the mempool can store. a value of 0 indicates
    // that the mempool can store an unlimited number of transactions.
    0,
    // AuctionFactory is responsible for determining what is an auction bid transaction and
    // how to extract the bid information from the transaction. There is a default implementation
    // that can be used or application developers can implement their own.
    auction.NewDefaultAuctionFactory(app.txConfig.TxDecoder()),
)

// Free lane allows transactions to be included in the next block for free.
freeConfig := block.BaseLaneConfig{
    Logger:        app.Logger(),
    TxEncoder:     app.txConfig.TxEncoder(),
    TxDecoder:     app.txConfig.TxDecoder(),
    MaxBlockSpace: sdk.ZeroDec(),
}
freeLane := free.NewFreeLane(
    freeConfig,
    // The FreeFactory is responsible for determining what is a free transaction. There is a
    // default implementation that can be used or application developers can implement
    // their own.
    free.NewDefaultFreeFactory(app.txConfig.TxDecoder()),
)

// Default lane accepts all other transactions.
defaultConfig := block.BaseLaneConfig{
    Logger:        app.Logger(),
    TxEncoder:     app.txConfig.TxEncoder(),
    TxDecoder:     app.txConfig.TxDecoder(),
    MaxBlockSpace: sdk.ZeroDec(),
}
defaultLane := base.NewDefaultLane(defaultConfig)

// Set the lanes into the mempool.
lanes := []block.Lane{
    tobLane,
    freeLane,
    defaultLane,
}
mempool := block.NewMempool(lanes...)
app.App.SetMempool(mempool)

// Set the lane antehandlers on the lanes.
for _, lane := range lanes {
    lane.SetAnteHandler(anteHandler)
}
app.App.SetAnteHandler(anteHandler)
```

### 3. 🚀 Instantiate the Block SDK Proposal Handlers

```go
proposalHandlers := abci.NewProposalHandler(
    app.Logger(),
    app.txConfig.TxDecoder(),
    mempool, // Block SDK mempool
)
app.App.SetPrepareProposal(proposalHandlers.PrepareProposalHandler())
app.App.SetProcessProposal(proposalHandlers.ProcessProposalHandler())
```

Every application utilizing the Block SDK should configure a `DefaultLane` that
accepts transactions that do not belong to any other `lane`. This is analogous
to how transactions are handled today in the standard `CometBFT` mempool.

:::info Transaction to `LANE` Matching

Transactions are inserted into all lanes the transaction matches to. However,
if developers want transactions to match to a single lane, they should utilize
the `IgnoreList` field on the lane’s `Config` to ignore lanes that the transaction should not be included in.

:::

#### ™️ Coming Soon

The Block SDK will have its own dedicated gRPC service for searchers, wallets, and users that allows them to query what `lane` their transaction might belong in, what fees they might have to pay for a given transaction, and the general state of the Block SDK mempool.
