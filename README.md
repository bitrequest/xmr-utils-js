# XMR Utils JS

Pure JavaScript Monero (XMR) cryptographic utilities library. Extracted from [bitrequest](https://github.com/bitrequest/bitrequest.github.io) for reuse in other projects.

*No Node.js, no WebAssembly, just vanilla JavaScript.*

## Features

- **Key Generation** - Generate Monero secret spend keys from BIP39 seeds
- **Address Derivation** - Create main addresses and subaddresses
- **Mnemonic Support** - Convert between secret keys and 25-word Monero mnemonics
- **View Key Extraction** - Parse view keys from addresses
- **Transaction Parsing** - Decode raw transaction hex, extract payment IDs
- **RingCT Decryption** - Decode encrypted output amounts
- **Ed25519 Curve** - Pure JS implementation of Curve25519 operations

## Live Demo

**[🔐 XMR Utils Test Suite](https://bitrequest.github.io/unit_tests_xmr_utils.html)** - Interactive tests and tools

---

## Download

### Quick Download (Terminal)

Run this command to download all required files:

```bash
mkdir xmr_utils && cd xmr_utils && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_xmr_utils.html && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_xmr_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js
```

Then open `unit_tests_xmr_utils.html` in your browser.

### Manual Download

Right-click and "Save As" for each file:

| File | Description |
|------|-------------|
| [assets_js_lib_sjcl.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js) | Stanford JavaScript Crypto Library |
| [assets_js_lib_crypto_utils.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js) | Crypto utilities (hashing, encoding) |
| [assets_js_lib_xmr_utils.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_xmr_utils.js) | Monero cryptographic functions |
| [unit_tests_xmr_utils.html](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_xmr_utils.html) | Interactive test suite |

---

## Usage

### HTML Setup

```html
<script src="assets_js_lib_sjcl.js"></script>
<script src="assets_js_lib_crypto_utils.js"></script>
<script src="assets_js_lib_xmr_utils.js"></script>
```

### Generate Keys from BIP39 Seed

```javascript
// Get secret spend key from BIP39 seed (128-char hex)
const bip39_seed = "5b56c417303faa3fcba7e57400e120a0...";
const ssk = XmrUtils.get_ssk(bip39_seed, true);

// Generate main address (index 0)
const keys = XmrUtils.xmr_getpubs(ssk, 0);
// {
//   address: "4...",
//   psk: "...",      // public spend key
//   pvk: "...",      // public view key
//   svk: "..."       // secret view key
// }

// Generate subaddress (index 1, 2, ...)
const subaddress = XmrUtils.xmr_getpubs(ssk, 1);
```

### Generate Random Monero Wallet

```javascript
// Generate random 256-bit secret spend key
const entropy = XmrUtils.mn_random(256);

// Convert to 25-word Monero mnemonic
const mnemonic = XmrUtils.secret_spend_key_to_words(entropy);
// "asylum bikini liar fazed hamburger physics opus ..."
```

### Create Address from Public Keys

```javascript
const address = XmrUtils.pub_keys_to_address(publicSpendKey, publicViewKey, 0);
// Main address (index 0) or subaddress (index > 0)
```

### Extract View Key from Address

```javascript
const vk = XmrUtils.get_vk(address);
// Returns public view key hex
```

### Parse Transaction

```javascript
const tx = XmrUtils.parse_xmr_tx_hex(rawTxHex);
// {
//   version, unlock_time, vin, vout,
//   extra, rct_signatures, ...
// }

// Extract payment ID (if present)
const paymentId = XmrUtils.extract_xmr_payment_id(extra, txPubKey, viewKey);
```

### Decode Encrypted Amount (RingCT)

```javascript
const amount = XmrUtils.decode_rct_amount(rctData, outputIndex, sharedSecret);
// Returns decrypted amount in atomic units
```

---

## API Reference

### Key Generation

| Function | Description |
|----------|-------------|
| `get_ssk(bip39_seed, is_seed)` | Derive secret spend key from BIP39 seed |
| `mn_random(bits)` | Generate random entropy (128/256 bits) |
| `sc_reduce32(hex)` | Reduce 32-byte value mod curve order |
| `xmr_get_publickey(privateKey)` | Derive public key from private key |
| `xmr_getpubs(ssk, index)` | Generate full key set and address |

### Mnemonic Functions

| Function | Description |
|----------|-------------|
| `secret_spend_key_to_words(ssk)` | Convert secret key to 25-word mnemonic |

### Address Operations

| Function | Description |
|----------|-------------|
| `pub_keys_to_address(psk, pvk, index)` | Create address from public keys |
| `get_vk(address)` | Extract public view key from address |
| `vk_obj(vk)` | Create view key object for sharing |
| `get_spend_pubkey_from_address(address)` | Extract public spend key |
| `base58_encode(data)` | Monero Base58 encoding |
| `base58_decode(data)` | Monero Base58 decoding |

### Hashing

| Function | Description |
|----------|-------------|
| `fasthash(hex)` | Keccak-256 hash (Monero variant) |
| `crc_32(str)` | CRC32 checksum for mnemonics |

### Elliptic Curve (Ed25519)

| Function | Description |
|----------|-------------|
| `xmr_getpoint(hex)` | Decode point from hex |
| `point_multiply(hex)` | Scalar multiplication |
| `point_to_monero_hex(point)` | Encode point to Monero format |
| `xmr_invert(number, modulo)` | Modular inverse |

### Transaction Parsing

| Function | Description |
|----------|-------------|
| `parse_xmr_tx_hex(tx_hex)` | Parse raw transaction |
| `extract_xmr_payment_id(extra, tx_pub, vk)` | Extract payment ID |
| `decode_rct_amount(rct, idx, secret)` | Decrypt RingCT amount |

### Payment ID

| Function | Description |
|----------|-------------|
| `xmr_pid()` | Generate random 16-char payment ID |
| `check_pid(payment_id)` | Validate payment ID format |

---

## Test Vectors

The library includes test vectors for verification:

```javascript
// From BIP39 test phrase
const bip39_seed = "5b56c417303faa3fcba7e57400e120a0ca83ec5a4fc9ffba757fbe63fbd77a89...";
const ssk = XmrUtils.get_ssk(bip39_seed, true);
const keys = XmrUtils.xmr_getpubs(ssk, 0);
// Generates valid Monero address starting with "4..."
```

---

## Technical Notes

- Uses **Ed25519** curve (Curve25519) - different from Bitcoin's secp256k1
- Implements Monero's specific **Keccak-256** variant for hashing
- Supports **RingCT** (Ring Confidential Transactions) amount decryption
- View tags optimization for faster transaction scanning
- Compatible with standard Monero 25-word mnemonics

---

## Security Warning

⚠️ **Never enter real secret keys or mnemonics on websites or share them with anyone.**

---

## Related Projects

- [bitrequest](https://github.com/bitrequest/bitrequest.github.io) - Cryptocurrency point-of-sale PWA
- [bip39-utils-js](https://github.com/bitrequest/bip39-utils-js) - BIP39/BIP32 HD wallet utilities
- [crypto_utils.js](https://github.com/bitrequest/bitrequest.github.io/blob/master/assets_js_lib_crypto_utils.js) - Low-level crypto utilities

---

## License

MIT License

## Credits

Developed for [bitrequest.io](https://bitrequest.io) - Lightweight cryptocurrency point-of-sale.