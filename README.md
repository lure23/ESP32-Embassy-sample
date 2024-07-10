# ESP32 Embassy Sample

You have [`esp-hal/examples`](https://github.com/esp-rs/esp-hal/tree/main/examples) running and would like to proceed to your own code.

There are two ways ahead.

![](.ai-images/paths-separating-on-grassy-meadows.png)


## The narrow path (`esp-template`)

>Note: `esp-template` does not support Embassy, yet (Jul'24). See ["Add support for generating embassy/async projects"](https://github.com/esp-rs/esp-template/issues/110).

The ESP32 Rust [docs](https://docs.esp-rs.org/esp-hal/esp-hal/0.18.0/esp32c3/esp_hal/index.html#creating-a-project) suggest using:

```
$ cargo generate -a esp-rs/esp-template
```

There is a "but" with this.

It asks you for the target device, and customizes the generated repo accordingly. You are locked in in whichever ESP32 variant you choose.

I would like to keep some wiggle room - like the `esp-hal/examples` supports all variants - but not use `cargo xtask` like it does.

This can be done!


## The wide path

1. Pick and copy `esp-hal/examples` folder to a suitable place.
2. Edit the `Cargo.toml`, replacing directory path references with versions (i.e. pull the libraries from `crates.io`).

	>Note! In Jul'24, the published versions are lagging behind the examples code!  You will need to keep the `path` references, for now.

	**For now**
	
	- Clone the whole `esp-hal` folder somewhere that you have access to it. 
	- Change the `path = "../{...}"` entries to point to the right path

	This is kinda annoying for CI/CD and what not, but hopefully won't last many months.

	**Eventually - once published versions are fine:**

	```diff
	esp-alloc           = { version = "0.4.0" }
	esp-backtrace       = { version = "0.12.1", features = ["exception-handler", "panic-handler", "println"] }
	esp-hal             = { version = "0.18.0", features = ["log"] }
	esp-hal-embassy     = { version = "0.1.0", optional = true }
	esp-hal-smartled    = { version = "0.11.0", optional = true }
	#esp-ieee802154      = { version = "0", optional = true }
	esp-println         = { version = "0.9.1", features = ["log"] }
	#esp-println         = { path = "../esp-println", default-features=false, features = ["log", "jtag-serial", "defmt-espflash"] }
	#esp-println         = { path = "../esp-println", default-features=false, features = ["log", "auto", "defmt-espflash"] }
	esp-storage         = { version = "0.3.0", optional = true }
	esp-wifi            = { version = "0.6.0", optional = true }
	```

3. Build

	>From the output of `cargo xtask`, we can pick up the `features` it eventually used (and add `esp32c3` to the mix).
	
	Then:
	
	```
	$ cd examples
	```

	```
	$ cargo build --bin embassy_hello_world --features esp32c3,embassy,esp-hal-embassy/integrated-timers --target riscv32imc-unknown-none-elf
	[...]
	```

4. Run

   ```
   $ espflash flash --monitor target/riscv32imc-unknown-none-elf/release/embassy_hello_world
   ```

	>Note: You can also directly do `cargo run`, with the same parameters as for the build.
	

## Optimizations

### Disable unneeded crates

We can likely disable these crates:

|||
|---|---|
|`embedded-hal-02`|It's the v.0.2 of HAL, just for backwards compatibility.|


### Add a `Makefile`?

It would be nice if the `target` could be automatically deduced from the features, but the author doesn't know how this can be done in Cargo alone. 

One could:

- make shell scripts that map to the right commands
- have a `Makefile` do that

This really is a matter of Your taste, and the author *omits* a Makefile sample, to remain objective. Hard. Did it! :)


## Debug vs. Release story

This is unknown to the author.

The `esp-hal/examples` defaults to compiling in debug mode, but the build gives this warning:

>**WARNING: use `--release`**
>
>  We *strongly* recommend using release profile when building `esp-hal`.
>  The dev profile can potentially be one or more orders of magnitude
>  slower than release, and may cause issues with timing-senstive
>  peripherals and/or devices.

Adding `--release` in the command obviously removes the warning, but the author wants to understand the pros/cons of debug/release setup a bit better.


