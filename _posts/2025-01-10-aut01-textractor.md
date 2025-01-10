---
title: "Textractor: Extract Text from Screen Region"
description: "Bash script paired with Kando and Spectacle to extract text from a region of your screen."
date: 2025-01-10
categories: [
    Linux,
    Automation,
]
tags: [
    _linux_automation,
    kando,
    spectacle,
]
# layout: default
---

## Dependencies
This automation was tested on a Arch Linux system with KDE plasma (6.2.4) running Wayland. The following dependencies are required:
- `kando-bin` (AUR)
- `spectacle` (pacman)
- `teseract` (pacman)
- `teseract-data-eng` (pacman)
- `wl-clipboard` (pacman)

Install the dependencies using the following commands:
```bash
yay -S kando-bin
sudo pacman -S spectacle tesseract wl-clipboard
```

### Kando (Aesthetic to invoke automation)
Kando is a pi menu that uses mouse gestures to launch applications, run macros, commands, open URI's, and more. It is a very powerful tool that can be used to automate many tasks. The `kando-bin` package is available in the AUR.

You can check out the Kando's official website [here](https://kando.menu/).

![Kando Demo](https://preview.redd.it/fly-pie-becoming-cross-platform-with-the-name-the-kando-menu-v0-r4komgjdcxsb1.gif?width=828&auto=webp&s=d59f4eeb284c9e11e91f4ccb96ce6b91adaf1166)

### Spectacle (Image acquisition)
Spectacle is a screen capture utility that allows you to capture a region of your screen. It is a power packed tool tool with annotation and recording features.

[https://apps.kde.org/spectacle/](https://apps.kde.org/spectacle/)

### Tessaract (OCR Engine)
Tesseract is an open-source OCR engine that can be used to extract text from images. The `tesseract` and `tesseract-data-eng` packages are available in the official Arch Linux repositories.

[Tesseract OCR git](https://github.com/tesseract-ocr/tesseract)

### wl-clipboard (Clipboard manager)
`wl-clipboard` is a clipboard manager for Wayland. It is used to copy the text extracted by Tesseract to the clipboard.

## Script
You can create a systematic directory structure to store the scripts and the output files. The following is the directory structure I use:

```bash
╰─ pwd && tree .
/home/teleporterx/.dotfiles
.
├── _cleanup
│   ├── clean_screenshots.sh
│   └── clean_tess.sh
├── _dump
│   └── tesseract
│       ├── tess_2025-01-10_12-09-05.txt
│       ├── tess_2025-01-10_12-09-34.txt
│       ├── tess_2025-01-10_12-13-01.txt
│       ├── tess_2025-01-10_12-14-58.txt
│       ├── tess_2025-01-10_12-17-26.txt
│       ├── tess_2025-01-10_12-25-34.txt
│       ├── tess_2025-01-10_12-55-15.txt
│       └── tess_2025-01-10_12-56-16.txt
├── profile
│   └── vscode_settings.json
└── sys_automation
    └── ocr_region.sh

6 directories, 12 files
```

`ocr_region.sh`
```bash
#!/bin/bash

# Generate a timestamp to use as the screenshot filename
timestamp=$(date +"%Y-%m-%d_%H-%M-%S")

# Define the screenshot path using the timestamp
screenshot_path=~/Pictures/Screenshots/screenshot_$timestamp.png
tess_dump=~/.dotfiles/_dump/tesseract/tess_$timestamp
# Capture the region and save the screenshot to the specified path with the timestamp
spectacle -r -b -n -o "$screenshot_path"

# Run Tesseract OCR on the saved screenshot
tesseract "$screenshot_path" $tess_dump

# (DEBUG) Display the OCR result
# cat $tess_dump.txt
# kate $tess_dump.txt
# export WAYLAND_DISPLAY=$WAYLAND_DISPLAY # works without this
cat $tess_dump.txt | wl-copy
```

> make sure the script is executable and the directories & dependencies the script uses exist.
{: .prompt-warning}

```bash
chmod +x ocr_region.sh
mkdir -p ~/.dotfiles/_dump/tesseract
mkdir -p ~/Pictures/Screenshots
```

### Kando Configuration
You can configure Kando to run the script using the run command option. The following is the configuration I use:

![Kando Configuration](assets/linux/automation/textractor/kando_config.png)

### Usage
To use the script, simply invoke Kando and draw a region on the screen. The script will capture the region, extract the text, and copy it to the clipboard.

You can then paste the extracted text wherever you need it.

![Textractor Demo](assets/linux/automation/textractor/textractor.gif)

### Cleanup
You can create a cleanup script to remove the screenshots and OCR output files after a certain period of time. I use the following

```bash
#!/bin/bash
rm /home/teleporterx/Pictures/Screenshots/*
rm /home/teleporterx/.dotfiles/_dump/tesseract/*
```

## Summary
This was a very simple automation that uses multiple tools to extract text from a region of the screen. Originally designed for KDE plasma running Wayland, it can be adapted to other distros (based on availability of dependencies), desktop environments and window managers with minimal changes.