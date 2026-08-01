---
name: Quote and execute a String transaction
description: Authenticate a player, request a quote, and execute a String fiat-for-Web3 transaction (fiat on/off ramp, NFT purchase, cross-chain) as merchant of record.
api: https://string-api.xyz
docs: https://docs.string.xyz/docs/transact
operations:
  - get_login          # GET /login  (RequestToSign — issue wallet nonce)
  - post_login-sign    # POST /login/sign (verify signed nonce → session)
  - post_quote         # POST /quote (estimate transaction cost)
  - post_transaction   # POST /transaction (execute)
generated: '2026-07-21'
method: generated
---

# Quote and execute a String transaction

Use this skill to take a player from login through an executed String transaction. String is the merchant of record for the fiat leg and delivers the digital asset on-chain.

## Prerequisites
- A **public API key** for the game, sent as the `X-Api-Key` header on every request (see `authentication/string-technology-inc-authentication.yml`).
- For NFT-purchase transaction types, the game's **smart contract and functions must be registered** beforehand (via the developer dashboard) — the registered contract determines the transaction type.
- Test against the **sandbox** first (`sandbox.string.xyz`); production access requires an upgrade.

## Steps

1. **Authenticate the player (wallet signature).**
   - Call `GET /login` (RequestToSign) with the player's wallet address to receive a nonce payload to sign.
   - Have the wallet sign the nonce, then call `POST /login/sign` with the signed `noncePayload`. String verifies the signature and establishes an authenticated session (auth cookies/token). Refresh with `POST /login/refresh`.

2. **Request a quote.**
   - Call `POST /quote` with the transaction details to get the estimated cost. Do not proceed to execution on a stale quote.

3. **Collect and submit payment information.**
   - Gather the player's card/payment info via the String Checkout/Direct/Unity surface so raw card data is handled by String, not your server.

4. **Execute the transaction.**
   - Call `POST /transaction` with the `quote` and `paymentInfo`. Set the `saveCard` query flag if the player opted to save the card.
   - String moves the fiat leg and delivers the digital asset on-chain, then emails the player a receipt.

## Error handling
Handle the documented statuses (see `errors/string-technology-inc-problem-types.yml`): `400` (bad quote/payment payload), `401` (bad/absent `X-Api-Key` or unauthenticated player), `403` (wrong key class / org scope), `500` (server error). Idempotency is not documented — do not blindly retry `POST /transaction` on a timeout; re-check status before re-submitting.
