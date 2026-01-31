---
name: domain-embedded
description: "Use for embedded/no_std Rust. Keywords: embedded, no_std, microcontroller, MCU, bare metal, firmware, HAL, RTIC, embassy, interrupt, GPIO, SPI, I2C"
user-invocable: false
---

# Embedded Domain

> **Layer 3: Domain Constraints**

## Domain Constraints → Rust Implications

| Domain Rule | Why | Rust |
|-------------|-----|------|
| No heap | Deterministic, no OOM | `heapless::Vec<T, N>` |
| No std | Bare metal | `#![no_std]` |
| Real-time | Predictable timing | No dynamic alloc |
| Interrupt safety | ISR can preempt | `Mutex<RefCell<T>>` + critical section |
| Hardware ownership | No conflicts | HAL singletons |

## Critical Rules

- **No Vec, Box, String** → `heapless` collections
- **Shared state must be ISR-safe** → `Mutex<RefCell<Option<T>>>`
- **Peripherals via HAL ownership** → `take().unwrap()` singletons

## Static Peripheral Pattern

```rust
#![no_std]
#![no_main]

use cortex_m::interrupt::{self, Mutex};
use core::cell::RefCell;

static LED: Mutex<RefCell<Option<Led>>> = Mutex::new(RefCell::new(None));

#[entry]
fn main() -> ! {
    let dp = pac::Peripherals::take().unwrap();
    let led = Led::new(dp.GPIOA);

    interrupt::free(|cs| {
        LED.borrow(cs).replace(Some(led));
    });

    loop {
        interrupt::free(|cs| {
            if let Some(led) = LED.borrow(cs).borrow_mut().as_mut() {
                led.toggle();
            }
        });
    }
}
```

## Layer Stack

| Layer | Examples | Purpose |
|-------|----------|---------|
| PAC | stm32f4, esp32c3 | Register access |
| HAL | stm32f4xx-hal | Hardware abstraction |
| Framework | RTIC, Embassy | Concurrency |

## Framework Comparison

| Framework | Style | Best For |
|-----------|-------|----------|
| RTIC | Priority-based | Interrupt-driven |
| Embassy | Async | Complex state machines |
| Bare metal | Manual | Simple apps |

## Key Crates

| Purpose | Crate |
|---------|-------|
| Runtime (ARM) | cortex-m-rt |
| Panic handler | panic-halt, panic-probe |
| Collections | heapless |
| HAL traits | embedded-hal |
| Logging | defmt |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using Vec | heapless::Vec |
| No critical section | Mutex + interrupt::free |
| Blocking in ISR | Defer to main loop |
