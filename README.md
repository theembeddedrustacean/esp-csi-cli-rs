# esp-csi-cli-rs

`esp-csi-cli-rs` is a command-line interface (CLI) application that runs on top of the `esp-csi-rs` crate.  `esp-csi-cli-rs` provides a user friendly interface for configuring and collecting Wi-Fi Channel State Information (CSI) on ESP devices. It allows users to configure various parameters related to CSI data collection.

In order to use this crate, you would need to build and flash the source code for your target device. Currently supported devices include:

- ESP32
- ESP32-C2
- ESP32-C3
- ESP32-C6
- ESP32-S3
- ESP32-H2

## Features

* **Multiple Wi-Fi Modes:** Configure the ESP device as an Access Point (AP), Station (STA), AP+STA, or Sniffer.
* **Configurable Network Architecture:** Set up the network topology for different scenarios.
* **Traffic Generation:** Optionally generate ICMP or UDP traffic at configurable intervals.
* **Fine-grained CSI Control:** Enable or disable specific CSI features like LLTF, HTLTF, STBC HTLTF, and LTF Merge.
* **CLI Control:** Interact with the device using simple commands over a serial connection.
* **Configuration Management:** Show the current configuration or reset to defaults.
* **Timed Collection:** Start CSI collection for a specific duration or run indefinitely.
* **Flexible Logging:** Supports standard `println!` or the more efficient `defmt` logging.


## Requirements

* **Hardware:** An ESP development board (e.g., ESP32-C3, ESP32-S3, ESP32...etc.).
* **Software:** `espflash` to flash the binary and monitor the output. Installation instructions are available [here](https://docs.esp-rs.org/book/tooling/espflash.html). 

> ‼️ Installing `espflash` requires a Rust installation. If you don't have Rust installed, follow instruction on the [rustup](https://rustup.rs/) website.

## Usage

1.  **Download Binary:** Navigate to the /binaries folder in the repository and identify the correct .elf binary for your ESP chip. Here are the file names for the different devices:
    * ESP32-C3: NAME?? `esp-csi-cli-rs-esp32c3.elf`
    * ESP32-C3 with `defmt`: NAME?? `esp-csi-cli-rs-esp32c3-defmt.elf`
    * ESP32-S3: `xtensa-esp32s3-espidf`
    * ESP32: `xtensa-esp32-espidf`

2.  **Flash:** Connect to your ESP device over USB and use `espflash` to flash the downloaded binary to your ESP by running the following command:
    ```bash
    espflash flash [path to downloaded .elf binary]
    ```
3.  **Monitor & Interact with CLI:** Use `espflash` to connect to and interact with your device by running either of the commands below. If you are using a file with a `defmt` extension note that you'll need to pass the same .elf file from step 2 to the monitor.
    ```bash
    # Example w/o defmt
    espflash --monitor [path to .elf binary]

    # Example w/ defmt
    espflash monitor —elf [path to attached file] —log-format defmt 
    ```
> 📝 When logging over `defmt` the monitor requires the original binary to be able to decode the incoming characters.

> 🛑 Step 2 needs only to be performed once. If you disconnect the ESP device all you need to do is run step 3 when reconnecting it followed by ctrl+R to reset the device. 

> 🛑 If you encounter strange behaviour with the CLI, it often helps to press ctrl+R to reset the device. Alternatively, you can terminate the whole session by pressing ctrl+C. Session termination requires that you run step 3 again to activate the monitor.

## Commands

* **`help [command]`**
    * Description: Display the main help menu or details for a specific command.
    * Example: `help set-wifi`

* **`set-traffic [OPTIONS]`**
    * Description: Configure traffic-related parameters.
    * Options:
        * `--enable`: Enable traffic generation (default: disabled).
        * `--type=<icmp|udp>`: Set the type of traffic (default: icmp).
        * `--interval-ms=<NUMBER>`: Specify the traffic interval in milliseconds (default: 1000).
    * Examples:
        * `set-traffic --enable --type=udp --interval-ms=50`
        * `set-traffic --enable`

* **`set-network [OPTIONS]`**
    * Description: Configure network architecture settings (primarily for NTP sync).
    * Options:
        * `--arch=<rsta|rapsta|apsta|sniff>`: Define the network architecture (default: sniff).
    * Examples:
        * `set-network --arch=rsta`
        * `set-network --arch=apsta`

* **`set-csi [OPTIONS]`**
    * Description: Configure CSI feature flags.
    * Options:
        * `--disable-lltf`: Disable LLTF CSI (default: enabled).
        * `--disable-htltf`: Disable HTLTF CSI (default: enabled).
        * `--disable-stbc-htltf`: Disable STBC HTLTF CSI (default: enabled).
        * `--disable-ltf-merge`: Disable LTF Merge CSI (default: enabled).
    * Examples:
        * `set-csi --disable-lltf --disable-ltf-merge`
        * `set-csi --disable-htltf`

* **`set-wifi [OPTIONS]`**
    * Description: Configure WiFi settings. **Note:** Replace spaces in SSIDs or Passwords with underscores (`_`).
    * Options:
        * `--mode=<ap|station|sniffer|ap-station>`: Specify WiFi operation mode (default: sniffer).
        * `--max-connections=<NUMBER>`: Set the maximum number of AP connections (default: 1).
        * `--hide-ssid`: Hide the SSID for the AP (default: visible).
        * `--ap-ssid=<SSID>`: Set the SSID for the AP.
        * `--ap-password=<PASSWORD>`: Set the password for the AP.
        * `--sta-ssid=<SSID>`: Set the SSID for the station.
        * `--sta-password=<PASSWORD>`: Set the password for the station.
    * Examples:
        * `set-wifi --mode ap --max-connections 5 --hide-ssid --ap-ssid=My_ESP_AP --ap-password=secret_pass`
        * `set-wifi --mode station --sta-ssid=My_Home_Network --sta-password=my_wifi_key`

* **`start [OPTIONS]`**
    * Description: Start the CSI collection process. Ensure the device is configured first.
    * Options:
        * `--duration=<SECONDS>`: Specify the duration (in seconds) for CSI collection. If omitted, collection runs indefinitely (or technically, for about a week).
    * Examples:
        * `start`
        * `start --duration=120`

* **`show-config`**
    * Description: Display the current configuration settings for all parameters.
    * Example: `show-config`

* **`reset-config`**
    * Description: Reset all configurations to their default values.
    * Example: `reset-config`

## Configuration Examples

1.  **Configure as an AP and start collecting for 5 minutes (ESP32-C3):**
    ```
    # Commands in serial monitor:
    reset-config
    set-wifi --mode ap --ap-ssid=ESP_CSI_AP 
    set-wifi --ap-password=testing123
    set-traffic --enable --type=icmp --interval-ms=500
    show-config
    start --duration=300
    ```

2.  **Configure as a Station connected to an existing network, disable some CSI features, and start collecting indefinitely (ESP32-S3):**
    ```
    # Commands in serial monitor:
    reset-config
    set-wifi --mode station 
    set-wifi --sta-ssid=My_Router
    set-wifi --sta-password=router_password
    set-csi --disable-htltf --disable-stbc-htltf
    show-config
    start
    ```

## Important Notes

* SSIDs and passwords containing spaces must have the spaces replaced with underscores (`_`) when using the `set-wifi` command. The application will convert them back internally.
* Ensure the target AP is running before starting collection in Station mode.


## Building From Source
Rather than flashing the precompiled binaries, it's possible to clone the repository and build the source. This would require some additional dependencies and modifications depending on the device you are using.

### Dependencies
At a minimum, you would need the following:
* Rust toolchain with ESP target support installed. Full instructions for setting up a development environment are available [here](https://docs.esp-rs.org/book/installation/index.html). 
* Tool for flashing the firmware. It is recommended to use `esp-flash`. Installation instructions are available [here](https://docs.esp-rs.org/book/tooling/espflash.html).
* A terminal program to view the output. It is also recommended to use  `esp-flash` which was installed in the previous step.

### Procedure
1. Clone this repository & identify the device you are using (ESP32-C3, ESP32-S3, ESP32-C6...etc.). The project repository code configuration defaults to the ESP32-C3 device. If you are using a ESP32-C3 you can skip to step 4. Also if you wish to enable `defmt` logging, follow the steps in the following section.
2. Modify .cargo/config.toml
3. Modify Cargo.toml dependencies
4. Build
5. Flash
6. Monitor

3.  **Build:** Navigate to the project directory (`csi-cli-esp32c3`) and build the project using Cargo, specifying your target:
    ```bash
    cargo build --release --target [specify target]

    # Example for ESP32-C3
    cargo build --release --target riscv32imc-esp-espidf

    # Example for ESP32-S3
    cargo build --release --target xtensa-esp32s3-espidf
    ```
    *(Note: The `build.rs` file includes specific linker arguments (`-Tlinkall.x`) needed for the build.)*


## Enabling Logging w/ `defmt`
This application can use either standard `println!` macros or the `defmt` framework for logging.

* **How it works:** A highly efficient logging framework for embedded devices. Requires specific setup.
* **Viewing Output:** Requires `probe-rs` or `espflash monitor --log-format defmt`.
* **Pros:** Very fast, low overhead, structured logging, log levels (info, warn, error, etc.).
* **Cons:** Requires configuration changes and specific tools to view output.

* **To enable `defmt`:**
    1.  **Modify `Cargo.toml`:**
        * Add `defmt` and `defmt-rtt` as dependencies.
        * Enable the `defmt` feature for `esp-hal` (or `esp-idf-hal` if used).
        ```toml
        # Example additions/modifications in Cargo.toml
        [dependencies]
        # ... other dependencies
        defmt = "0.3"
        defmt-rtt = "0.4"
        esp-backtrace = { version = "0.11.0", features = ["esp32c3", "panic-handler", "exception-handler", "println", "defmt"] } # Ensure defmt feature is added if using esp-backtrace

        # Example for esp-hal (adjust version and features as needed)
        esp-hal = { version = "0.18.0", features = ["esp32c3", "async", "defmt"] } # Add 'defmt' feature

        # ... other dependencies

        [features]
        default = []
        # Add other features if needed
        ```
    2.  **Modify `.cargo/config.toml`:**
        * Configure the runner to use `probe-rs` or `espflash` with `defmt` enabled.
        ```toml
        # Example .cargo/config.toml for probe-rs
        [target.'cfg(target_arch = "riscv32")']
        runner = "probe-rs run --chip esp32c3" # Adjust chip as needed

        [target.'cfg(target_arch = "xtensa")']
        runner = "probe-rs run --chip esp32s3" # Adjust chip as needed

        # --- OR ---

        # Example .cargo/config.toml for espflash (simpler)
        [target.'cfg(any(target_arch = "riscv32", target_arch = "xtensa"))']
        runner = "espflash flash --monitor --log-format defmt" # Monitor will show defmt logs
        ```
    3.  **Modify Code:**
        * Replace `use esp_println::println;` with `use defmt::{info, warn, error};` (and other levels as needed).
        * Replace `println!(...)` calls with `info!(...)`, `warn!(...)`, etc.

* **Building/Running with `defmt`:**
    * If using `probe-rs` runner: `cargo run --release --target <your-target>`
    * If using `espflash` runner: `cargo run --release --target <your-target>` (will flash and open monitor)
    * Alternatively, build (`cargo build ...`) then view logs: `espflash monitor --log-format defmt target/.../async_main`
