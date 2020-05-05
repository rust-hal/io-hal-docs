# GPIO

## `split`

* Pass `&mut rcc.bus_name` as an argument (F1, F3, L4)
* Pass `&mut rcc`  as an argument (F0, L0, G0, G4)
  * Preferred, as different MCUs have different buses for GPIO
* No arguments (F4)
  * Even better, but requires critical sections or `unsafe`

## Pin setup

* Pass GPIO registers explicitly (F1, F3, L4)
  * [-] Too verbose
  * [-] Different MCU families have different registers
  * [-] Sometimes one have to guess which register to pass depending on a pin number (xxxh/xxxl)
* Pass a critical section (F0)
  * [+] No overhead
  * [-] Quite inconvenient: one have to pass all the initialized pins out of the `interrupt::free` closure
* No arguments, use `interrupt::free` inside the pin setup function
  * [-] Additional overhead for `interrupt::free`, could be a problem for flash-limited applications
* No arguments, no `interrupt::free` inside (F4, L0, G0, G4)
  * [-] Unsound, can lead to race conditions
* No arguments, no `interrupt::free` inside, mark function unsafe - **recommended**
  * [+] No additional overhead, user can avoid `interrupt::free`.
  * [-] unsafety — it would be better if hal provided safe abstractions
  * motivation: a critical section is not required in general,
  but can be added around the function call if necessary.

In some rare cases you can also use binband region or atomics to access shared GPIO registers
in a safe way without critical sections.

## Alternate function setup

* Pass GPIO registers explicitly (F1, F3, L4)
  * [-] Too verbose
  * [-] Different MCU families have different registers
  * [-] Sometimes one have to guess which register to pass depending on a pin number (xxxh/xxxl)
* Pass a critical section (F0)
  * [+] No overhead
  * [-] Quite inconvenient: one have to pass all the initialized pins out of the `interrupt::free` closure
* No arguments, use `interrupt::free` inside the pin setup function
  * [-] Additional overhead for `interrupt::free`
* No arguments, no `interrupt::free` inside (F4)
  * [-] Unsound, can lead to race conditions
* `pub(crate) fn set_alt_mode` (L0, G0, G4)
  * [+] eliminates the need for users to set AF explicitly
  * [-] Unsound, can lead to race conditions
* `pub(crate) fn unsafe set_alt_mode`
  * [+] eliminates the need for users to set AF explicitly
  * [-] requires `interrupt::free` block around each call
* `pub(crate) fn set_alt_mode` + `interrupt::free` inside the function - **recommended**
  * [+] eliminates the need for users to set AF explicitly
  * [+] `interrupt::free` overhead is less significant than is the previous case

## Erased types

## Method naming

## Additional features

* `with_*` methods — temporary usage of a pin in different pin mode (L0)
