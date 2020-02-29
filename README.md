# bourneagn_rs
Inline Bash scripting within Rust: because you want it. Right?

## Desired features
Drawing inspiration from `inline::python` macros, there's a chance that we can use the dark magic of Web Assembly + Rust to control low-level system targets from our Javascript API.
Specifically, the proof-of-concept test case this package hopes to achieve is creating a `bash! { ... }` macro that can run any BASH commands with the granted authority. In other words, if the user runs a particular browser sandbox as root, then it might be possible to echo new values into the `/sys/class/backlight/@insert_device_symlink/brightness` and control their system level screen brightness from a button click in the browser!

At the time of creating this repository, the projected server API to be compiled into wasm32_unknown_unknown might resemble:

```rust
#![feature(proc_macro_hygiene)]
use bourneagn_rs::bash;

// Abstracted behaviors:
fn get_min_backlight(){}
fn get_max_backlight(){}
fn get_step_backlight(){}
fn get_curr_backlight(){}
fn get_blight_file() -> Result<String> {
    // ( ... snip ... )
    String::from("/sys/class/backlight/intel-backlight/backlight")
}
fn main() {

    // Backlight variables
    let step: u32 = get_step_backlight();
    let cw_brightness: u32 = get_curr_backlight();
    let min_brightness: u32 = get_min_backlight();
    let mut max_brightness: u32 = get_max_backlight();
    let step: u32 = get_step_backlight();

    // Adjust the max to accommodate upper limit
    max_brightness = *(&max_brightness - &step);

    // Lookup the correct system backlight file
    let backlight_file: Result<String> = get_blight_file()?;
    // Case-check the backlight-file just to be sure
    bash! {
        for brightness in ${'min_brightness..'max_brightness}
        do
            echo $brightness > 'backlight_file && sleep 0.25
        done
        echo 'cw_brightness > 'backlight_file
    } 
}
```
