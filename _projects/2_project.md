---
layout: page
title: ClipboardHistory on Linux
description:
img: 
importance: 2
category: work
related_publications: false
---

## Motivation and Overview
I built this as a side project for a utility I use often on Windows but couldn't find one for Linux so I decided to write one. I thought it would serve as a good practice. 

This project implements a Clipboard History utility for Linux, allowing users to track and manage their clipboard contents efficiently. The utility provides a simple way to maintain a history of copied items and retrieve them as needed.

## Features
This application provides the following featues:
- Track clipboard contents across multiple copy operations
- Easy retrieval of previously copied items
- Lightweight and minimal resource usage
- Simple command-line interface

**⚠️ IMPORTANT:** This project **only supports text content**. It does not store or process images, rich text, or files.

The project is PEP8 and Shellcheck compliant as much as possible. For exaple, the only PEP8 errors that might arise are E501 errors.

## Technical Implementation
The UI in Python3 was handled by the `gi` module, `evdev` was used for Keyboard Events.

The PEP8 Compliance was checked using [AutoPEP8](https://packagecontrol.io/packages/AutoPEP8)  and POSIX Compliance for the shell scripts by using [Shellcheck](https://github.com/koalaman/shellcheck).

### File hierarchy:

    ├── HotkeyHandler
    │   ├── HotkeyHandler_Wayland.py
    │   └── keyconfig.json
    ├── LICENSE
    ├── README.md
    ├── WaylandCheck.sh
    ├── WaylandClipboard.py
    ├── WaylandStartup.sh
    ├── clipboardStorage.py
    └── removeService.sh

The Clipboard History persists between boots, and can be found at `/home/$USER/.config/clipboard_history.json`. You can change this in the `clipboardStorage.py` and `WaylandClipboard`file. 

If you wish to read the JSON directly, I would recommend using the `json.tool` tool in Python, you can do so as `python3 -m json.tool file.json`

## Installation
### Prerequisites
The application requires a **Wayland** display., and having `systemd`
as your **init** system.
Other pre-requisites are:
* `wl-clipboard`
* Python3
	* `venv`
	* `evdev`

The `WaylandCheck.sh` script checks for these pre-requisites and if not found, installs them.
### How to Install
1. Clone the repository using 
	`git clone https://github.com/I-9028/ClipboardHistoryonLinux.git`
	
	Enter into the repo folder by `cd ClipboardHistoryonLinux/`
	
3. Assign executable permissions to the "WaylandCheck.sh" script using

    `chmod +x WaylandCheck.sh`

4. It checks if you have all the required dependencies, and creates a Python Virtual Environment. Run it as follows:

    `sudo ./WaylandCheck.sh`
    
5. Then its the startup script for the application itself,

    ``./WaylandStartup.sh``
    
	It starts the service and lets you know if you need to reboot. If it asks you to reboot, please **do so**. For some reason, logging out and re-logging in does not solve the issue.
For this script to work,  the user must be in the *input* group, wich is not very secure. I intend to look for a workaround.
 
4. Monitor the status of the service using

    `systemctl --user status ClipboardHistory.service`

### Uninstallation
If you simply wish to stop the service and remove the Configuration FIles generated, run the `removeService.sh` script as follows:

    ``./removeService.sh``
    
This **does not** remove the Virtual Environment. To remove the Virtual Environment, run `sudo rm -rf ClipboardVenv/`

## Usage
By default, the key-bindings are set to `LeftAlt + V`, these can be changed by modifying the `HotkeyHandler/config.json` file. 

- Copy items as usual
- Use Hotkey COmbination t pull the window up.
    - View clipboard history
    - Select and paste previous items
    - Clear clipboard history

        {% include figure.liquid loading="eager" path="assets/img/ClipboardHistory/AppDemo.png" title="App Demo" class="img-fluid rounded z-depth-1" %}

## Limitations and Potential Improvements
- Current implementation may have platform-specific constraints
- Future enhancements could include:
  - GUI interface
  - Advanced filtering
  - Persistent storage across sessions

## License

This project is licensed under the MIT License.

## Contact

Prathamesh D. (Me)
- GitHub: [@I-9028](https://github.com/I-9028)
- Project Link: [https://github.com/I-9028/ClipboardHistoryonLinux](https://github.com/I-9028/ClipboardHistoryonLinux)