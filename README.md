# Glass Pumpkin

[![Build Status](https://travis-ci.org/mikelodder7/glass_pumpkin.svg?branch=master)](https://travis-ci.org/mikelodder7/glass_pumpkin)
[![Build status](https://ci.appveyor.com/api/projects/status/1htmp82mdvmfjjap?svg=true)](https://ci.appveyor.com/project/mikelodder7/glass-pumpkin)
[![build status](https://gitlab.com/mikelodder7/glass_pumpkin/badges/master/pipeline.svg)]
[![Crate][crate-image]][crate-link]
[![Docs][docs-image]][docs-link]
![Apache 2.0/MIT Licensed][license-image]

A random number generator for generating large prime numbers, suitable for cryptography.

# Purpose
`glass_pumpkin` is a cryptographically-secure, random number generator, useful for generating large prime numbers.
This library is inspired by [pumpkin](https://github.com/zcdziura/pumpkin) except its meant to be used with rust stable.
It also lowers the 512-bit restriction to 128-bits so these can be generated and used for elliptic curve prime fields.
It exposes the prime testing functions as well.
This crate uses [num-bigint](https://crates.io/crates/num-bigint) instead of `ramp`. I have found
`num-bigint` to be just as fast as `ramp` for generating primes. On average, generating primes takes less
than 200ms and safe primes about 10 seconds on modern hardware.

# Installation
Add the following to your `Cargo.toml` file:
```toml
glass_pumpkin = "0.4"
```

# Example
```rust
extern crate glass_pumpkin;

use glass_pumpkin::prime;

fn main() {
    let p = prime::new(1024);
    let q = prime::new(1024);

    let n = p * q;

    println!("{}", n);
}
```

You can also supply `OsRng` and generate primes from that.
```rust
extern crate glass_pumpkin;
extern crate rand;

use glass_pumpkin::prime;

use rand::prelude::*;

fn main() {
    let mut rng = OsRng::new().unwrap();
    let p = prime::from_rng(1024, &mut rng);
    let q = prime::from_rng(1024, &mut rng);

    let n = p * q;
    println!("{}", n);
}
```

# Prime Generation

`Primes` are generated similarly to OpenSSL except it applies some recommendations from the [Prime and Prejudice](https://eprint.iacr.org/2018/749.pdf) paper and uses
the Baillie-PSW method:

1. Generate a random odd number of a given bit-length.
1. Divide the candidate by the first 2048 prime numbers. This helps to
    eliminate certain cases that pass Miller-Rabin but are not prime.
1. Test the candidate with Fermat's Theorem.
1. Runs log2(bitlength) + 5 Miller-Rabin tests with one of them using generator `2`.
1. Run lucas test.

Safe primes require (n-1)/2 also be prime.

# Prime Checking

You can use this crate to check numbers for primality.
```rust
extern crate glass_pumpkin;
extern crate num_bigint;

use glass_pumpkin::prime;
use glass_pumpkin::safe_prime;
use num_bigint::BigUint;

fn main() {

    if prime::check(&BigUint::from(5)) {
        println!("is prime");
    }

    if safe_prime::check(&BigUint::from(7)) {
        println!("is safe prime");
    }
}
```

Stronger prime checking that uses the Baillie-PSW method is an option
by using the `strong_check` methods available in the `prime` and `safe_prime`
modules. Primes generated by this crate will pass the Baillie-PSW
test when using cryptographically secure random number generators. For now,
`prime::new()` and `safe_prime::new()` will continue to use generation
method as describe earlier.

[//]: # (badges)

[crate-image]: https://img.shields.io/crates/v/glass_pumpkin.svg
[crate-link]: https://crates.io/crates/glass_pumpkin
[docs-image]: https://docs.rs/glass_pumpkin/badge.svg
[docs-link]: https://docs.rs/glass_pumpkin/
[license-image]: https://img.shields.io/badge/license-Apache2.0/MIT-blue.svg
