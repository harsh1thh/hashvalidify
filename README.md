# Hashvalidify

A tiny browser-first experiment that keeps crypto funds and shared files from vanishing when someone mistypes an address.

## Why

Mistyped wallet addresses or file handles mean funds gone and data gone. This package pairs addresses with SHA-256 sums on both the sender and receiver so nothing moves unless the identities match.

## Browser-only plan

- Use `crypto.subtle.digest` for hashing, no external dependencies.
- Embed a component library block that renders the receiver address input and checksum display.
- The sender holds their wallet address and checksum, compares against the receiver block before sending.

## Transaction guard flow

1. Receiver publishes canonical wallet/file address + SHA-256 checksum.
2. The sender pastes the receiver payload and verifies both values.
3. The sender app cross-checks its own wallet address and checksum for extra safety.
4. Only after a dual match does the UI unlock the “send” action.

## Minimal usage sketch

```html
<!-- tough love boilerplate -->
<div id="receiver-block">
  <!-- ...existing markup... -->
</div>
<script type="module">
  // ...existing code...
  async function digestString(input) {
    const enc = new TextEncoder().encode(input);
    const hash = await crypto.subtle.digest('SHA-256', enc);
    return [...new Uint8Array(hash)].map(b => b.toString(16).padStart(2, '0')).join('');
  }
  // ...existing code...
  async function verifyPayload(receiverAddr, receiverHash, senderAddr, senderHash) {
    const [calcReceiver, calcSender] = await Promise.all([
      digestString(receiverAddr),
      digestString(senderAddr),
    ]);
    return calcReceiver === receiverHash && calcSender === senderHash;
  }
  // ...existing code...
</script>
```

## Hash algorithm choices

- Keep SHA-256 as the default guard.
- Consider SHA-384 or SHA-512 when you want a stronger checksum without new tooling.
- For lighter and faster flows, try shortened hashes (`SHA-1` is still fast but weaker) or experiment with in-browser wasm versions of BLAKE2s/BLAKE3 if you can tolerate custom builds.

## Roadmap

- [x] Outline intent + workflow
- [ ] Scaffold wallet guard component
- [ ] Add file-share checksum module
- [ ] Wire up verification demo page
- [ ] Publish npm alpha + docs