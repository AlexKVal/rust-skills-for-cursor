---
name: domain-fintech
description: "Use when building fintech apps. Keywords: fintech, trading, decimal, currency, financial, money, transaction, ledger, payment"
user-invocable: false
---

# FinTech Domain

> **Layer 3: Domain Constraints**

## Domain Constraints → Rust Implications

| Domain Rule | Why | Rust |
|-------------|-----|------|
| Audit trail | Regulatory compliance | Arc<T>, immutable, event sourcing |
| Precision | No floating point errors | `rust_decimal::Decimal` |
| Consistency | Money can't disappear | Transaction types, validated totals |
| Reproducibility | Deterministic execution | No race conditions |

## Critical Rules

- **NEVER use f64 for money** → `rust_decimal::Decimal`
- **Transactions must be immutable** → `Arc<T>`, event sourcing
- **Shared data must be thread-safe** → `Arc` not `Rc`

## Currency Pattern

```rust
use rust_decimal::Decimal;

#[derive(Clone, Debug, PartialEq)]
pub struct Amount {
    value: Decimal,
    currency: Currency,
}

impl Amount {
    pub fn add(&self, other: &Amount) -> Result<Amount, CurrencyMismatch> {
        if self.currency != other.currency {
            return Err(CurrencyMismatch);
        }
        Ok(Amount::new(self.value + other.value, self.currency))
    }
}
```

## Key Crates

| Purpose | Crate |
|---------|-------|
| Decimal math | rust_decimal |
| Date/time | chrono, time |
| Validation | validator |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using f64 | rust_decimal |
| Mutable transaction | Immutable + events |
| String for amount | Validated newtype |
| Silent overflow | Checked arithmetic |
