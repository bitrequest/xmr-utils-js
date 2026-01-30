# XMR Utils JS

Pure JavaScript Monero (XMR) cryptographic utilities library. Extracted from [bitrequest](https://github.com/bitrequest/bitrequest.github.io) for reuse in other projects.

*No Node.js, no WebAssembly, just vanilla JavaScript.*

## Features

- **Key Generation** - Generate Monero secret spend keys from BIP39 seeds or random entropy
- **Address Derivation** - Create main addresses and subaddresses
- **Mnemonic Support** - Convert between secret keys and 25-word Monero mnemonics
- **View Key Extraction** - Parse view keys from addresses for watch-only wallets
- **Base58 Encoding** - Monero-specific Base58 encoding/decoding
- **Transaction Parsing** - Decode raw transaction hex
- **Ed25519 Curve** - Pure JS implementation of Curve25519 operations
- **Cryptographic Hashing** - Keccak-256 (Monero variant), CRC32
- **Built-in Compatibility Testing** - Verify browser/environment support

## Live Demo

**[🔐 XMR Utils Test Suite](https://bitrequest.github.io/unit_tests_xmr_utils.html)** - Interactive tests and tools

---

## Download

### Quick Download (Terminal)

```bash
mkdir xmr_utils && cd xmr_utils && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_xmr_utils.html && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_xmr_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js
```

Then open `unit_tests_xmr_utils.html` in your browser.

### Manual Download

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

---

## Quick Start Examples

### Generate Monero Wallet from BIP39 Seed

```javascript
// From BIP39 seed (128-char hex from mnemonic_to_seed)
const bip39Seed = "5b56c417303faa3fcba7e57400e120a0ca83ec5a4fc9ffba757fbe63fbd77a89a1a3be4c67196f57c39a88b76373733891bfaba16ed27a813ceed498804c0570";

// Derive secret spend key
const ssk = XmrUtils.get_ssk(bip39Seed, true);

// Generate main address (index 0)
const keys = XmrUtils.xmr_getpubs(ssk, 0);
// {
//   address: "4...",   // Main Monero address
//   psk: "...",        // Public spend key (32 bytes hex)
//   pvk: "...",        // Public view key (32 bytes hex)
//   svk: "..."         // Secret view key (32 bytes hex)
// }

console.log(keys.address);
// "4A2F9pQB9sJzGcWo8cRMQHHJhVf7A8E5Ld9..."
```

### Generate Subaddresses

```javascript
const ssk = XmrUtils.get_ssk(bip39Seed, true);

// Main address (index 0)
const main = XmrUtils.xmr_getpubs(ssk, 0);
console.log("Main:", main.address);  // Starts with "4"

// Subaddress 1 (index 1)
const sub1 = XmrUtils.xmr_getpubs(ssk, 1);
console.log("Sub 1:", sub1.address);  // Starts with "8"

// Subaddress 2 (index 2)
const sub2 = XmrUtils.xmr_getpubs(ssk, 2);
console.log("Sub 2:", sub2.address);  // Starts with "8"
```

### Generate Random Monero Wallet

```javascript
// Generate random 256-bit entropy
const entropy = XmrUtils.mn_random(256);

// Reduce to valid scalar
const sskBytes = XmrUtils.sc_reduce32(entropy);

// Generate keys
const keys = XmrUtils.xmr_getpubs(sskBytes, 0);

// Convert to 25-word Monero mnemonic
const mnemonic = XmrUtils.secret_spend_key_to_words(sskBytes);
// "asylum bikini liar fazed hamburger physics opus ..." (25 words)
```

### Recover Wallet from Monero Mnemonic

```javascript
// 25-word Monero mnemonic (native format, not BIP39)
const mnemonic = "western adventure fungal unbending onward odometer husband cobra hotel likewise scrub idled omnibus teeming lettuce rejoices zippers alley firm tadpoles hope tasked obedient oust oust";

// Convert mnemonic back to secret spend key
const sskBytes = XmrUtils.words_to_secret_spend_key(mnemonic);
// Validates checksum, throws on invalid input

// Derive full wallet
const keys = XmrUtils.xmr_getpubs(sskBytes, 0);

console.log(keys.ssk);     // Secret spend key (hex)
console.log(keys.svk);     // Secret view key (hex)
console.log(keys.address); // "477h3C6E6C4VLMR36bQL3yLcA8Aq3jts..."
```

### Create Address from Public Keys

```javascript
// If you have public spend key and public view key
const address = XmrUtils.pub_keys_to_address(publicSpendKey, publicViewKey, 0);
// Main address (index 0) starts with "4"

// Subaddress format
const subaddress = XmrUtils.pub_keys_to_address(publicSpendKey, publicViewKey, 1);
// Subaddress (index > 0) starts with "8"
```

### Extract View Key from Address

```javascript
const address = "4A2F9pQB9sJzGcWo8cRMQHHJhVf7A8E5Ld9...";

// Get view key object for watch-only wallet
const vkObj = XmrUtils.vk_obj(address);
// {
//   address: "4A2F9pQB...",
//   vk: "..." // Public view key hex
// }
```

### Decode Monero Address

```javascript
const address = "4A2F9pQB9sJzGcWo8cRMQHHJhVf7A8E5Ld9...";

// Decode Base58 address
const decoded = XmrUtils.base58_decode(address);
// Returns hex string with network byte + public keys + checksum

// Extract public spend key
const psk = XmrUtils.get_spend_pubkey_from_address(address);
// 32-byte public spend key hex
```

---

## Payment IDs

```javascript
// Generate random 16-character payment ID
const pid = XmrUtils.xmr_pid();
// "a1b2c3d4e5f67890"

// Validate payment ID format
const isValid = XmrUtils.check_pid(pid);
// true for valid 16 or 64 character hex

// Reject invalid
XmrUtils.check_pid("invalid");  // false
XmrUtils.check_pid("");         // true (empty is allowed)
```

---

## Cryptographic Functions

### Keccak-256 (Monero Variant)

```javascript
// Monero uses Keccak-256 (not standard SHA3-256)
const hash = XmrUtils.fasthash(hexData);
// Returns 64-char hex (32 bytes)
```

### Scalar Reduction

```javascript
// Reduce 32-byte value mod curve order (Ed25519)
const reduced = XmrUtils.sc_reduce32(hexOrBytes);
// Returns reduced scalar as Uint8Array
```

### Point Multiplication

```javascript
// Ed25519 scalar multiplication
const scalar = CryptoUtils.hex_to_bytes("...");
const point = XmrUtils.point_multiply(scalar);
// Returns point object

// Convert point to Monero hex format
const hex = XmrUtils.point_to_monero_hex(point);
// 32-byte compressed point
```

### CRC32

```javascript
// Used in mnemonic checksums
const checksum = XmrUtils.crc_32("test string");
// Returns integer checksum
```

---

## Base58 Encoding

Monero uses a modified Base58 encoding (different from Bitcoin's Base58Check).

```javascript
// Encode
const bytes = CryptoUtils.hex_to_bytes(hexString);
const encoded = XmrUtils.base58_encode(bytes);

// Decode
const decoded = XmrUtils.base58_decode(encoded);
// Returns hex string
```

---

## Compatibility Testing

The library includes built-in test functions to verify browser/environment compatibility before use.

### Quick Compatibility Check

```javascript
// Check if XMR derivation works
const results = XmrUtils.test_xmr_compatibility();
if (results.compatible) {
	console.log("Environment compatible!");
}
```

### Individual Test Functions

```javascript
// Test spend key → address derivation (standalone)
XmrUtils.test_xmr_derivation();  // returns true/false

// Test spend key → view key derivation
XmrUtils.test_xmr_keys();  // returns true/false

// Test address generation
XmrUtils.test_xmr_address();  // returns true/false

// Full compatibility check (includes CryptoUtils tests)
XmrUtils.test_xmr_compatibility();
// Returns: {
//   compatible: true/false,
//   crypto_api: true/false,
//   bigint: true/false,
//   keys: true/false,
//   address: true/false,
//   errors: [],
//   timing_ms: 12.5
// }
```

### Test Constants

The library exposes test vectors derived from the standard [BIP39 test phrase](https://github.com/bitcoinbook/bitcoinbook/blob/f8b883dcd4e3d1b9adf40fed59b7e898fbd9241f/ch05.asciidoc): `army van defense carry jealous true garbage claim echo media make crunch`

```javascript
const TC = XmrUtils.xmr_utils_const;

TC.version          // "1.1.0"

// Derived via BIP44 path m/44'/128'/0'/0/0 + sc_reduce32(fasthash(privkey))
TC.test_spend_key   // "007d984c3df532fdd86cd83bf42482a5c2e180a51ae1d0096e13048fba1fa108"
TC.test_view_key    // "e4d63789cdfa2ec48571e93e47520690b2c6e11386c90448e8b357d1cd917c00"
TC.test_address     // "477h3C6E6C4VLMR36bQL3yLcA8Aq3jts1AHLzm5QXipDdXVCYPnKEvUKykh2GTYqkkeQoTEhWpzvVQ4rMgLM1YpeD6qdHbS"
```

---

## API Reference

### Key Generation

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `get_ssk` | `bip39_seed, is_seed` | `Uint8Array` | Derive secret spend key from BIP39 |
| `mn_random` | `bits` (128/256) | `string` (hex) | Generate random entropy |
| `sc_reduce32` | `hex\|bytes` | `Uint8Array` | Reduce to valid Ed25519 scalar |
| `xmr_get_publickey` | `privateKey` | `string` (hex) | Derive public key from private |
| `xmr_getpubs` | `ssk, index` | `object` | Generate full key set and address |

### Mnemonic Functions

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `secret_spend_key_to_words` | `sskBytes` | `string` | Convert to 25-word Monero mnemonic |
| `words_to_secret_spend_key` | `mnemonic` | `Uint8Array` | Convert 25-word mnemonic to secret spend key |

### Address Operations

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `pub_keys_to_address` | `psk, pvk, index` | `string` | Create address from public keys |
| `get_spend_pubkey_from_address` | `address` | `string` | Extract public spend key |
| `vk_obj` | `address` | `object` | Get view key object |
| `base58_encode` | `bytes` | `string` | Monero Base58 encode |
| `base58_decode` | `string` | `string` (hex) | Monero Base58 decode |

### Payment ID

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `xmr_pid` | - | `string` | Generate random 16-char payment ID |
| `check_pid` | `payment_id` | `boolean` | Validate payment ID format |

### Hashing

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `fasthash` | `hex` | `string` (hex) | Keccak-256 hash |
| `crc_32` | `string` | `number` | CRC32 checksum |
| `make_crc_table` | - | `array` | Generate CRC32 lookup table |

### Elliptic Curve (Ed25519)

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `xmr_getpoint` | `hex` | `object` | Decode point from hex |
| `point_multiply` | `scalar` | `object` | Scalar * base point |
| `point_to_monero_hex` | `point` | `string` | Encode point to hex |
| `xmr_invert` | `number, modulo` | `BigInt` | Modular inverse |
| `xmod` | `a, b` | `BigInt` | Modulo operation |
| `xpow_mod` | `base, exp, mod` | `BigInt` | Modular exponentiation |

### Byte Conversion

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `str_to_bin` | `string` | `Uint8Array` | String to bytes |
| `bytes_to_number_le` | `bytes` | `BigInt` | Little-endian bytes to number |
| `uint64_to_8be` | `num, size` | `Uint8Array` | Number to big-endian bytes |
| `uint32_hex` | `num` | `string` | Number to little-endian hex |
| `xmr_number_to_hex` | `bigint` | `string` | BigInt to padded hex |

### Transaction Parsing

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `parse_xmr_tx_hex` | `tx_hex` | `object` | Parse raw transaction |

### Constants

```javascript
XmrUtils.xmr_CURVE.P   // Field prime
XmrUtils.xmr_CURVE.Gx  // Base point X
XmrUtils.xmr_CURVE.Gy  // Base point Y
XmrUtils.xmr_CURVE.L   // Curve order
```

### Testing Functions

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `test_xmr_derivation` | `[spend_key, expected_address]` | `boolean` | Test spend key → address derivation |
| `test_xmr_keys` | - | `boolean` | Test spend key → view key derivation |
| `test_xmr_address` | - | `boolean` | Test address generation |
| `test_xmr_compatibility` | - | `object` | Full compatibility check with timing |

### Exported Constants

| Property | Type | Description |
|----------|------|-------------|
| `xmr_utils_const` | `object` | Test vectors and version info |
| `xmr_CURVE` | `object` | Ed25519 curve parameters (P, Gx, Gy, L) |

---

## Monero Address Formats

| Prefix | Network | Type |
|--------|---------|------|
| `4` | Mainnet | Standard address |
| `8` | Mainnet | Subaddress |
| `9` | Testnet | Standard address |
| `A` | Testnet | Subaddress |

---

## Interactive Test Suite

The test suite (`unit_tests_xmr_utils.html`) includes:

### Automated Tests (63+ tests)

**Built-in Library Tests**
- `XmrUtils.test_xmr_derivation` - Spend key → address derivation
- `XmrUtils.test_xmr_keys` - Spend key → view key derivation
- `XmrUtils.test_xmr_address` - Address generation
- `XmrUtils.test_xmr_compatibility` - Full compatibility check

**Test Constants Validation**
- Verify `test_spend_key` produces `test_address`
- Verify `test_spend_key` produces `test_view_key`

**Unit Tests**
- Key derivation from BIP39 seeds
- Address generation and validation
- Subaddress derivation
- Mnemonic conversion (roundtrip, checksum validation)
- Base58 encode/decode
- Scalar reduction
- Elliptic curve operations
- Hashing functions

### Interactive Tools
1. **Derive XMR Keys** - Generate keys from secret spend key
2. **Generate Subaddresses** - Create multiple subaddresses
3. **SSK to Mnemonic** - Convert secret key to 25 words
4. **Mnemonic to SSK** - Recover secret key from 25-word mnemonic
5. **Fasthash (Keccak)** - Test Keccak-256 hashing
6. **Decode Address** - Parse Monero address components
7. **Scalar Reduce** - Test sc_reduce32
8. **Random Wallet** - Generate new random wallet
9. **Payment ID** - Generate and validate payment IDs
10. **Base58 Encode** - Test Monero Base58 encoding
11. **Point Multiply** - Test Ed25519 operations
12. **CRC32** - Calculate checksums
13. **Byte Conversion** - Test number/byte utilities

---

## Technical Notes

### Ed25519 vs secp256k1

Monero uses the **Ed25519** curve (also called Curve25519), which is different from Bitcoin's secp256k1:

| Property | Ed25519 (Monero) | secp256k1 (Bitcoin) |
|----------|------------------|---------------------|
| Field size | 255 bits | 256 bits |
| Curve type | Edwards | Weierstrass |
| Key size | 32 bytes | 32 bytes |
| Signature | EdDSA | ECDSA |

### Keccak-256 vs SHA3-256

Monero uses the original **Keccak-256** algorithm, not the standardized SHA3-256:
- Keccak-256 was the SHA3 competition winner
- NIST modified padding for final SHA3 standard
- Monero kept original Keccak padding

### View-Only Wallets

With the secret view key (svk) and public spend key (psk), you can:
- Monitor incoming transactions
- View transaction history
- **Cannot** spend funds

```javascript
const keys = XmrUtils.xmr_getpubs(ssk, 0);
// Share these for watch-only:
// - keys.pvk (public view key)
// - keys.psk (public spend key)
// - keys.address

// Keep secret:
// - keys.svk (secret view key) - needed for viewing
// - ssk (secret spend key) - needed for spending
```

---

## Security Warning

⚠️ **Never enter real secret keys or mnemonics on websites or share them with anyone.**

This library is intended for:
- Educational purposes
- Development and testing
- Generating view-only wallet credentials

For production wallets, use the official Monero wallet software.

---

## Dependencies

- **sjcl.js** - Stanford JavaScript Crypto Library
- **crypto_utils.js** - Low-level utilities (keccak256, hex conversion)

---

## Related Projects

- [crypto-utils-js](https://github.com/bitrequest/crypto-utils-js) - Low-level cryptocurrency utilities
- [bip39-utils-js](https://github.com/bitrequest/bip39-utils-js) - BIP39/BIP32 HD wallet utilities
- [bitrequest](https://github.com/bitrequest/bitrequest.github.io) - Cryptocurrency point-of-sale PWA

---

## License

AGPL-3.0 License

## Credits

Developed for [bitrequest.io](https://bitrequest.io) - Lightweight cryptocurrency point-of-sale.
