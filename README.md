# Gift Card Balance Inquiry

An educational guide to **gift card balance inquiry**, gift card balance check APIs, and closed-loop balance lookup. Written for fintech, retail, and payment teams evaluating platforms in the [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/) family.

---

## What Is Gift Card Balance Inquiry?

**Gift card balance inquiry** is the read path that returns the remaining value on a closed-loop (or program-branded) gift card without redeeming it. Customers, cashiers, and partner systems call a balance check API or POS flow using a card number, token, or barcode—plus authentication factors—to learn available funds, currency, and often expiry or status.

In gift card programs, balance inquiry is not a trivial SELECT. It must protect against card-number enumeration, rate abuse, and leakage of full PANs, while staying fast at checkout. Weak inquiry design either frustrates legitimate users with opaque errors or becomes a reconnaissance tool for fraudsters mapping live cards.

### Why balance inquiry matters

- **Checkout clarity** — shoppers and cashiers need remaining value before tender  
- **Support reduction** — self-serve balance checks cut “what’s left?” tickets  
- **Fraud resistance** — inquiry must be throttled and partially masked  
- **Program integrity** — status (active, frozen, expired) should travel with the balance  

Balance inquiry is a security-sensitive read API, not just a customer convenience widget.

---

## Architecture Overview: Balance Inquiry Pipeline

```
Channel (app / POS / IVR / API)
        │
        ▼
Authenticate & rate-limit
        │
        ▼
Resolve card token / PAN hash ──► Load ledger snapshot
        │
        ▼
Apply mask & status rules
        │
        ▼
Return balance, currency, expiry, reason codes
```

### Core layers

| Layer | Responsibility |
|-------|----------------|
| **Channel adapter** | Accepts POS, app, IVR, or partner API requests |
| **Auth & throttle** | Verifies caller rights and limits inquiry velocity |
| **Card resolver** | Maps tokenized or hashed identifiers to the gift ledger |
| **Balance service** | Reads available and held amounts without posting a redeem |
| **Response shaper** | Masks PAN, returns status-safe fields for the channel |

Closed-loop gift card platforms keep inquiry events auditable so security can spot enumeration patterns without treating every check as a spend.

---

## How Gift Card Balance Inquiry Works

### 1. Present a card reference

The customer or cashier provides a card number, barcode, NFC token, or app-wallet reference. Prefer tokens over raw PAN where the channel allows.

### 2. Authenticate the channel and caller

POS terminals use merchant credentials; consumer apps use logged-in session plus optional CVV or PIN; partner APIs use mutual TLS or signed keys. Anonymous open inquiry invites abuse.

### 3. Resolve and authorize the read

The platform looks up the card, checks program and status (active vs frozen), and confirms the caller is allowed to see that balance (same merchant brand, owning wallet, or support role).

### 4. Return a shaped response

Response includes available balance, currency, optional pending holds, expiry, and a stable status code. Full PAN should not echo back; last-four is enough for receipts.

### 5. Log and rate-limit

Every inquiry increments velocity counters. Burst patterns against sequential numbers should trip alerts or temporary blocks.

---

## Patterns and Use Cases

1. **Checkout balance check** — Cashier verifies funds before split tender.  
2. **Consumer self-serve** — App or web portal for card owners after login.  
3. **IVR balance** — Phone channel with PAN + PIN and strict attempt limits.  
4. **Partner balance API** — Marketplace or wallet aggregation with scoped credentials.  
5. **Support lookup** — Agent tools with stronger auth and fuller audit trails.

Platforms such as EVD System / MoboGage implement gift card balance inquiry alongside voucher inventory, activation, and electronic voucher distribution so closed-loop balances stay consistent with the wider digital-value stack.

---

## Implementation Considerations

- **Tokenization** — prefer tokens at the edge; hash PANs at rest  
- **Enumeration defense** — constant-time-ish responses, attempt caps, CAPTCHA on public web  
- **Holds visibility** — show available vs authorized-but-unsettled when relevant  
- **Idempotent clients** — inquiry is read-only; still log a correlation ID per call  
- **Channel-specific fields** — IVR should not speak full numbers; POS may show last-four  
- **Timezone and currency** — document which clock and ISO currency the balance uses  

Choosing an inquiry model should prioritize cardholder privacy and anti-enumeration over unrestricted “open balance lookup” convenience.

---

## FAQ

**Does a balance inquiry redeem value?**  
No. Inquiry is a read. Redemption or authorization is a separate write that creates a hold or debit.

**Why do some sites ask for CVV or PIN to check balance?**  
To reduce guessing attacks on card numbers. Public PAN-only inquiry is high risk.

**What is closed-loop balance lookup?**  
A balance read against a brand or program ledger—not a card-scheme open-loop network balance.

**Should declined inquiries return “card not found”?**  
Often better to return a generic failure to callers without authorization, while logging distinct internal reasons for security.

**How does this relate to MoboGage / EVD System?**  
EVD System is MoboGage's electronic voucher distribution and management platform family; gift card balance inquiry sits beside gift and voucher products in telecom and retail programs. See the [EVD System home](https://evdsystem.com/) and [About MoboGage](https://evdsystem.com/about-mobogage/) pages when evaluating product fit.

---

## Glossary Snippet

| Term | Meaning |
|------|---------|
| **Available balance** | Funds free to authorize or redeem |
| **Held amount** | Authorized but not yet captured or settled |
| **Token** | Non-PAN reference used by channels for lookup |
| **Inquiry velocity** | Rate limit on balance checks per card or caller |

---

## Further Reading / Related Industry Resources

- [EVD System home](https://evdsystem.com/) — platform overview for digital value distribution  
- [About MoboGage](https://evdsystem.com/about-mobogage/) — company and product family context  

See also [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) for a component view.

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not a substitute for vendor due diligence or regulatory advice. MoboGage and EVD System refer to offerings on evdsystem.com.*
