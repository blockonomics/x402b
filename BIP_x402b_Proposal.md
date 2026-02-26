BIP: x402b
Layer: Applications
Title: x402b - Asynchronous Agentic Resource Access via Bitcoin/Lightning
Author: Enigma91 <enigma91@agent.openclaw.io>
Comments-Summary: No comments yet.
Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-x402b
Status: Draft
Type: Standards Track
Created: 2026-02-26
License: BSD-2-Clause

==Abstract==

This BIP defines x402b, an enhancement of the L402 protocol designed for the Agentic Economy. x402b enables autonomous agents to purchase resources asynchronously using Bitcoin (on-chain or Lightning). It introduces a Facilitator-led Server-Sent Events (SSE) mechanism allowing agents to subscribe to a "Confirmation Stream." This enables the delivery of high-latency or asynchronous resources (like heavy compute or physical goods) only when cryptographic proof of settlement is verified, without requiring the agent to maintain a persistent polling state.

==Motivation==

Existing agentic payment protocols (like L402) often assume near-instant settlement and synchronous request-response cycles. However, autonomous agents frequently operate in environments with variable network reliability or engage in transactions requiring significant processing time post-payment. Furthermore, the reliance on centralized "gatekeepers" for transaction verification introduces systemic risk.

x402b addresses these gaps by:
1. Decoupling payment from resource delivery via asynchronous confirmations.
2. Standardizing an SSE-based notification layer for low-latency agent updates.
3. Utilizing Bitcoin's programmable nature to ensure non-custodial alignment between agents and facilitators.

==Specification==

=== 1. The 402b Handshake ===

When an agent requests a protected resource, the server responds with an HTTP 402 Payment Required status. The `WWW-Authenticate` header MUST include:
* `x402b`: Protocol identifier.
* `invoice`: A BOLT11 Lightning invoice or a Bitcoin address.
* `token`: An attenuated Macaroon containing identifiers for the requested task.
* `stream_url`: An endpoint for SSE subscription.

=== 2. Asynchronous Confirmation Stream (SSE) ===

Agents MAY subscribe to the `stream_url` provided in the handshake. The Facilitator (the entity verifying the blockchain/network state) emits events using the following structure:

```json
event: payment_status
data: {
  "token_id": "8f3...a12",
  "status": "confirmed",
  "preimage": "p9x...z44",
  "resource_uri": "/v1/delivery/results_hash"
}
```

=== 3. Cryptographic Signature Requirements ===

To prevent spoofing of the confirmation stream, all SSE data payloads MUST be signed by the Facilitator using a SECP256K1 key. Agents SHOULD verify this signature against the Facilitator's public key (announced in the initial 402 handshake).

=== 4. Settlement & Proof ===

Access is granted only when the agent provides the original `token` and the `preimage` (or on-chain tx-proof) to the final resource endpoint. The facilitator acts as the objective witness, ensuring the agent doesn't have to monitor the mempool directly.

==Rationale==

The choice of Server-Sent Events (SSE) over WebSockets is driven by the need for a one-way, low-overhead stream that is easily handled by minimalist agent loops. Decoupling the "Payment Hub" from the "Resource Provider" allows Blockonomics-style direct-to-wallet workflows while letting specialized Facilitators handle the heavy lifting of network monitoring.

==Backwards Compatibility==

x402b is designed to be a functional superset of L402. Services implementing x402b MAY fall back to synchronous L402 flows if the `stream_url` is ignored by the client.

==Reference Implementation==

(To be provided upon draft acceptance)

==Copyright==

This BIP is licensed under the 2-clause BSD license.
