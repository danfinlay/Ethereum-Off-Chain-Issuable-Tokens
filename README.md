# Off-chain Issuable Tokens

Some tokens have a central issuing authority, like for coins in an online game, which may only gain interesting value meriting an on-chain transaction once they accrue in a significant quantity.

In cases like this, issuance can be performed in an offchain manner that maintains on-chain trustlessness.

State channels require an initial deposit because it is assumed that the sender has a limited quantity of tokens. In the case of a centralized issuer, you have an implicit infinite deposit of tokens, and this allows tokens to be counterfactually issued with a simple signed message.

An example issuance message, signed by the issuer might look like this:

```
struct IssuanceMessage {
  address recipient;
  uint amount;
}
```

A sample extension to the token interface would be a method such as:

```solidity
function redeemPromisedTokens(bytes payload, uint8 v, bytes32 r, bytes32 s) public returns (bool);
```

## Pseudolidity Implementation

```
contract OffchainIssuableToken is MintableToken {

  mapping (address => uint) amountMintedSoFar;

  struct IssuanceMessage {
    address recipient;
    uint amount;
  }

  function redeemPromisedTokens(bytes payload, uint8 v, bytes32 r, bytes32 s) public returns (bool) {
    assert(verify(payload, v, r, s));

    IssuanceMessage memory msg = parse(payload, v, r, s);

    // Ensure user submits larger balanced messages
    // to prevent replay attacks:
    assert(msg.amount > amountMintedSoFar[msg.sender]);

    // Use safe math in practice:
    uint amountToMint = msg.amount - amountMintedSoFar[msg.sender]

    mint(msg.recipient, amountToMint);

    // Keep track of amount issued for that user.
    amountMintedSoFar[msg.sender] = msg.amount;
  }
}
```

## Wallet Implementation

Wallets could store these messages and display them like tokens, and they would not need to be privately held unless privacy were a goal, because the only thing the message could be used for would be issuing these tokens.

## Use Cases

An AI company wants to reward online participants for training its web interface, and wants to issue tokens for each attempt without an on-chain transaction.

Personal currencies that are backed primarily by an individual's trust, so the threat of inflation is already accounted for in the risk of holding that coin.

An organization wants to crowdfund in person, in cash, and give out certificates for token redemption without bothering with on-chain transactions, especially when (like with some crowd-funding ventures) the project may not complete, and the tokens may not be worth redeeming anyway.

Casual or localized token experiments: Selling a gift card

This pattern also works when offline, so even in a low-internet environment, if a person or entity has the ability to sign minting messages for a token that has a known value, they could sign these messages as a way to pay someone "once they're online".


## Possible other features

The signed message could include a reference to the issuing contract, to allow the issuing key to issue multiple different tokens.

There could be a `redeemAndSend` method so that a user can both redeem their tokens and send some of them in a single transaction.

The token contract is given the power to verify signatures and enforce issuance however it wants. There could be a list of approved issuers, and individual issuers could even have issuance limits to tranche liability of compromised issuing keys.

If a withdraw period + unainamous signatures were added (like in a traditional state channel), then the tokens issued could also be bi-directionally exchanged, without ever making an on-chain transaction.

If these messages can themselves be signed to delegate permission to withdraw the coin to another account (recursive), this system has evolved into [social collateral cash](https://gist.github.com/danfinlay/73b6ffd11aea5a85767fe20c6ad868c5), wherein recipients _are_ in a game of chicken to withdraw, and so a run on the bank only does not happen if the token is issued to parties that are trusted to only use the power if needed. (I enjoy exploring those mechanics on the blog [Capabul](https://medium.com/capabul)).

