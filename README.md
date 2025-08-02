# 🎵 Set FLAC File Icons with Embedded Cover Art on macOS

Tired of seeing all your `.flac` files in Finder with a boring, generic music icon because macOS doesn’t support FLAC? This solution automatically extracts embedded album art from FLAC metadata and sets it as a Finder thumbnail icon.

> ✅ Works with individual files, folders, or even recursively through subfolders
> 🧠 No need to convert to ALAC or use a separate music library manager
> 💡 Custom Quick Action for Finder via Automator

---

## ✨ Preview

**Before:**

Generic music icons for all `.flac` files.

**After:**

Each `.flac` file has its own album art thumbnail in Finder 🎉

---

## 🛠️ Requirements

Make sure these tools are installed using [Homebrew](https://brew.sh):

```bash
brew install ffmpeg flac fileicon
```

⚠️ `fileicon` must be installed via:

```bash
brew install fileicon
```

---

## 💡 Note

* This only works on **HFS+** or **APFS** formatted drives.
* Icons will **not show** on **exFAT** or **FAT32** volumes.
* Extra metadata adds a few KB to each file, but does **not** affect audio or tag integrity.
* Works across macOS versions and syncs fine with OneDrive, Dropbox, etc.

---

## 🧪 How It Works

1. `metaflac` extracts the embedded album art to a temporary image.
2. `fileicon` assigns the extracted image as the custom icon for the file.
3. The script cleans up after itself and reports success.

---

## ⚙️ Installation

### Step 1: Create Automator Quick Action

1. Open `Automator`
2. Create a new **Quick Action**
3. Set the following at the top:

   * **Workflow receives:** `files or folders`
   * **In:** `Finder`
4. Add action: **Run Shell Script**

   * **Shell:** `/bin/bash`
   * **Pass input:** `as arguments`

### Step 2: Paste This Script

<details>
<summary>Flat Folder Version (no recursion)</summary>

```bash
#!/bin/bash
set -euo pipefail

METAFLAC="/opt/homebrew/bin/metaflac"
FILEICON="/opt/homebrew/bin/fileicon"

set_icon_for_flac() {
    flac="$1"
    tmp_cover="cover.jpg"

    [ -f "$tmp_cover" ] && rm "$tmp_cover"
    "$METAFLAC" --export-picture-to="$tmp_cover" "$flac" 2>/dev/null || true

    if [ -f "$tmp_cover" ]; then
        "$FILEICON" set "$flac" "$tmp_cover" || echo "Could not set icon for $flac"
        rm "$tmp_cover"
    fi
}

for item in "$@"; do
    if [ -d "$item" ]; then
        shopt -s nullglob
        for flac in "$item"/*.flac "$item"/*.FLAC; do
            set_icon_for_flac "$flac"
        done
        shopt -u nullglob
    else
        ext="${item##*.}"
        ext_lower=$(printf '%s' "$ext" | tr '[:upper:]' '[:lower:]')
        if [ "$ext_lower" = "flac" ]; then
            set_icon_for_flac "$item"
        fi
    fi
done

echo "Done"
```

</details>

<details>
<summary>Recursive Version (includes subfolders)</summary>

```bash
#!/bin/bash
set -euo pipefail

METAFLAC="/opt/homebrew/bin/metaflac"
FILEICON="/opt/homebrew/bin/fileicon"

set_icon_for_flac() {
    flac="$1"
    tmp_cover="cover.jpg"

    [ -f "$tmp_cover" ] && rm "$tmp_cover"
    "$METAFLAC" --export-picture-to="$tmp_cover" "$flac" 2>/dev/null || true

    if [ -f "$tmp_cover" ]; then
        "$FILEICON" set "$flac" "$tmp_cover" || echo "Could not set icon for $flac"
        rm "$tmp_cover"
    fi
}

for item in "$@"; do
    if [ -d "$item" ]; then
        find "$item" -type f \( -iname "*.flac" \) | while IFS= read -r flac; do
            set_icon_for_flac "$flac"
        done
    else
        ext="${item##*.}"
        ext_lower=$(printf '%s' "$ext" | tr '[:upper:]' '[:lower:]')
        if [ "$ext_lower" = "flac" ]; then
            set_icon_for_flac "$item"
        fi
    fi
done

echo "Done (Recursive)"
```

</details>

### Step 3: Save the Workflow

Name it something like `Set FLAC Icons` and save.

---

## 🧩 Usage

1. Navigate to your folder or select `.flac` files in Finder.
2. Right-click > **Quick Actions** > `Set FLAC Icons`
3. Done! Finder will update thumbnails based on embedded cover art.

---

## 🤖 Credits

This project was put together after extensive experimentation and need — for all FLAC collectors who manage their music with care and want better Finder integration on macOS. 🖖

---

## 🧼 Uninstall

If you ever want to remove the custom icons:

```bash
find /path/to/music -name '*.flac' -exec xattr -d com.apple.FinderInfo {} \;
```
