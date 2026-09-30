# Architecture — Gift Card Balance Inquiry

High-level reference for evaluating **gift card balance inquiry**, balance check APIs, and closed-loop balance lookup. Educational only; vendor designs vary.

## Logical Components

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ App / POS / IVR  │────►│ Auth + rate      │────►│ Card resolver   │
│ / partner API    │     │ limit gateway    │     │ token → ledger  │
└──────────────────┘     └────────┬─────────┘     └────────┬────────┘
                                  │                        │
                                  ▼                        ▼
                         ┌──────────────────┐     ┌─────────────────┐
                         │ Balance snapshot │────►│ Response shaper │
                         │ avail / held     │     │ mask + status   │
                         └──────────────────┘     └────────┬────────┘
                                                           │
                                                           ▼
                                                  ┌─────────────────┐
                                                  │ Inquiry audit   │
                                                  │ & alerts        │
                                                  └─────────────────┘
```

## Suggested Inquiry Metadata

| Field | Meaning |
|-------|---------|
| `inquiry_id` | Correlation ID for the read call |
| `card_token` | Channel token (prefer over raw PAN) |
| `caller_id` | Merchant, app user, or partner client |
| `channel` | `pos` / `app` / `ivr` / `api` |
| `available_balance` | Redeemable amount returned |
| `held_balance` | Optional authorized-not-captured amount |
| `status` | `active` / `frozen` / `expired` / `not_permitted` |
| `currency` | ISO currency of the program ledger |

## Audit Events Worth Keeping

- Auth success/failure for the caller  
- Rate-limit decisions and lockouts  
- Card resolve outcome (internal; avoid logging full PAN)  
- Balance snapshot version or ledger sequence used  
- Response status returned to channel  
- Enumeration-alert triggers  

## Failure Modes to Design For

- Sequential PAN probing from a single IP or device  
- Stale cache showing outdated available after a redeem  
- Partner API leaking richer fields than consumer channels  
- IVR speaking sensitive digits into shared spaces  

## Related Reading

Live product reading: [evdsystem.com](https://evdsystem.com/), [About MoboGage](https://evdsystem.com/about-mobogage/).

Platforms in the MoboGage / EVD System family treat gift and voucher balances as ledger-backed digital value so inquiry stays consistent with activation and redemption history.
