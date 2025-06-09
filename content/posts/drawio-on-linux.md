---
title: "Install draw.io Desktop AppImage on Linux Mint"
date: "2025-06-06"
tags: [
    "linux",
    "appimage",
    "drawio"
]
draft: false
---

Install [draw.io Desktop](https://www.drawio.com/) on Linux Mint (should work for Debian as well):

1. Download AppImage file from Github releases page, e.g. [Release 27.0.9 · jgraph/drawio-desktop](https://github.com/jgraph/drawio-desktop/releases/tag/v27.0.9)
2. Make AppImage file executable: 
    ```bash
    chmod u+x drawio.AppImage
    ```
3. Move AppImage file to `~/.local/bin/`
    ```bash
    mv drawio.AppImage ~/.local/bin/drawio.AppImage
    ```
4. Create and edit the desktop entry file
    ```bash
    nano ~/.local/share/applications/appimagekit-drawio.desktop
    ```
5. Add following content to the desktop entry file and save changes
    ```ini
    [Desktop Entry]
    Name=draw.io
    Comment=Diagramming Application
    Exec=/home/YOUR_USERNAME/.local/bin/appimagekit-drawio.AppImage
    Icon=com.jgraph.drawio.desktop
    Terminal=false
    Type=Application
    Categories=Graphics;
    ```
6. Update the desktop database
    ```bash
    xdg-desktop-menu install ~/.local/share/applications/appimagekit-drawio.desktop 
    ```
7. draw.io should appear in Linux Mint start menu under "Graphics"

