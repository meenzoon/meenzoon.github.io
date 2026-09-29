---
layout: default
---

> ✨ **Crafted with Claude Opus 5.5**
>
> *Born from a real-world frustration, shaped together with AI.*
> This post was written with **Claude Opus 5.5** by Anthropic.

---

# Working with Multiple Obsidian Vaults in a Single Window — Setting Up BRAT + Folder Bridge

> **TL;DR**
> Obsidian opens a new window every time you open a vault, and it only recognizes files inside a single root folder.
> With the **Folder Bridge** plugin, you can **"mount" vaults and folders scattered across your drives into one hub vault** and work with all of them from a single window.
> Folder Bridge isn't in the official community plugin directory yet, so this guide starts by installing **BRAT** and then uses it to install Folder Bridge straight from GitHub.

---

## 1. The Problem

The longer you use Obsidian, the more vaults you collect: one for work, one for personal study, one for blog drafts… And with every new vault, the friction adds up.

- **Every vault opens in a new window.** Switching to another vault leaves the current window open and spawns a new one. With three or four Obsidian windows in the taskbar, it's easy to lose track of which vault you're actually in.
- **I wanted everything in one place.** I wanted search, links, and the graph to work across vault boundaries.
- **My files live in different places.** Obsidian treats one folder as the root and only recognizes files underneath it. To use several vaults as one, you'd have to move them all under a single parent folder. My files are spread across `C:`, `D:`, an external SSD, and a NAS, so that simply wasn't practical.

Symbolic links are one workaround, but they're created differently on each OS and aren't something Obsidian recommends, so I was hesitant to go that route. That's when I found **Folder Bridge**.

---

## 2. What Is Folder Bridge?

Folder Bridge is a plugin that **"mounts" folders outside your vault so they appear as regular folders inside it.** The key point is that nothing gets copied or moved. Your files stay exactly where they are, and all reads and writes happen directly at their original location.

Key features:

- **Mount multiple folders at once** — attach as many folders as you like and choose exactly where each one appears inside the vault.
- **Works with core features** — the file explorer, Quick Switcher, search, and embeds all work on mounted files.
- **Multiple mount types** — local folders (including external drives, NAS, and UNC paths), **other Obsidian vaults**, WebDAV, S3-compatible storage, and SFTP.
- **Safety features** — per-mount read-only mode, ignore lists, and background health checks every 30 seconds.

Here's what my setup looks like:

```text
📁 Hub  ← the only vault I open in Obsidian
├── 📁 .obsidian            (plugins, themes, and settings live here)
├── 📁 Work     ◀── D:\Company\WorkVault        (mounted vault)
├── 📁 Study    ◀── E:\Study\StudyVault         (mounted vault)
├── 📁 Blog     ◀── C:\Users\me\Documents\Blog  (mounted folder)
└── 📁 Archive  ◀── \\NAS\share\archive         (read-only mount)
```

> ⚠️ At the time of writing, Folder Bridge is **not listed in Obsidian's official community plugin directory.** Searching under Settings → Community plugins → Browse won't find it, so you'll need to install it through **BRAT**.

---

## 3. Step 1 — Create a Hub Vault

Obsidian plugins are **installed per vault**. So before installing anything, decide which vault will be your "hub," the one you'll keep open from now on.

1. Create an empty folder for the hub (e.g. `D:\ObsidianHub`).
2. Click the vault name in Obsidian's bottom-left corner and open **Manage vaults**.
3. Use **Create new vault** or **Open folder as vault** to open the folder you just created.

Everything from here on is done inside this hub vault.

<!-- 📷 Screenshot: Manage vaults screen -->

---

## 4. Step 2 — Install BRAT

**BRAT (Beta Reviewers Auto-update Tool)** is a plugin that takes a GitHub repository address, downloads that plugin's release files (`main.js`, `manifest.json`, `styles.css`), installs them, and keeps them updated. It replaces the manual routine of creating folders and copying files. BRAT itself is an official community plugin, so you install it like any other.

1. Open **Settings** (the ⚙️ icon at the bottom left, or `Ctrl + ,` / `Cmd + ,` on macOS).
2. Go to **Community plugins** in the left sidebar.
3. If this is your first time, click **Turn on community plugins** to turn off Restricted mode.
4. Click **Browse** and search for `BRAT`.
5. Select **Obsidian42 - BRAT**, then click **Install** → **Enable**.

Once **BRAT** appears under the community plugins section of the Settings sidebar, you're done.

<!-- 📷 Screenshot: BRAT in the community plugin browser -->

---

## 5. Step 3 — Install Folder Bridge with BRAT

Now register Folder Bridge's GitHub repository with BRAT. Use whichever method you prefer:

- **Option A (Command palette):** Press `Ctrl + P` (`Cmd + P` on macOS) → run `BRAT: Add a beta plugin for testing`
- **Option B (Settings):** Settings → **BRAT** → click **Add beta plugin**

Fill in the dialog as follows:

| Field | Value |
| --- | --- |
| Repository | `https://github.com/tescolopio/Obsidian_FolderBridge` |
| Version | Latest version |
| Enable after installing | Checked (enables the plugin right away) |

Click **Add Plugin** and BRAT will download and install the latest release. After the confirmation notice, check Settings → Community plugins → Installed plugins to make sure **Folder Bridge** is turned on. If a folder-plus icon (📁➕) shows up in the left ribbon, you're all set.

<!-- 📷 Screenshot: BRAT "Add beta plugin" dialog -->

> 💡 **You can install other unofficial plugins the same way.**
> Just enter the plugin's GitHub URL (or `username/repository`) in the Repository field. The only requirement is that the repository's **Releases** include `main.js` and `manifest.json`.

**Updating:** From the command palette, run the command starting with `BRAT: Check for updates…` to check and update every plugin installed through BRAT in one go. You can also enable automatic updates on startup in BRAT's settings.

---

## 6. Step 4 — Mount Your Scattered Vaults and Folders

Now for the main event: attaching your existing vaults and folders to the hub, one by one.

1. Click the **folder-plus icon** in the ribbon, or go to Settings → **Folder Bridge** → **Add Mount Point**.
2. Choose a **Mount Type**:
   - To attach an existing Obsidian vault → **Another Obsidian vault**
   - To attach a regular folder → **Local folder**
3. **Real path** — click **Browse…** and select the actual folder. For a vault, pick the **vault's root folder** (the one containing the `.obsidian` folder). You can also narrow it down to a specific subfolder of that vault if needed.
4. **Virtual path** — where the folder will appear inside the hub vault, and under what name (e.g. `Work`, `Study`).
5. Optionally set a **Label** (the name shown in the settings list) and **Read-only** (blocks all writes).
6. Click **Validate & Add**.

The folder shows up in the file explorer immediately, with no restart required. Repeat for every vault or folder you want to attach.

When you use the "Another Obsidian vault" type, Folder Bridge **automatically adds** the other vault's `.obsidian`, `.trash`, and `.smart-connections` folders to the ignore list, which keeps the two vaults' configuration files from interfering with each other.

Here's how I set mine up (paths are examples):

| Virtual path | Real location | Mount Type | Options |
| --- | --- | --- | --- |
| `Work` | `D:\Company\WorkVault` | Another Obsidian vault | - |
| `Study` | `E:\Study\StudyVault` | Another Obsidian vault | - |
| `Blog` | `C:\Users\me\Documents\Blog` | Local folder | - |
| `Archive` | `\\NAS\share\archive` | Local folder | Read-only, Use polling |

### Handy Features

- **Edit a mount:** In Settings → Folder Bridge, click the pencil icon on any mount to change its path, name, or options without removing it.
- **Move a mount:** Drag a mounted folder in the file explorer, or right-click it → **Move mount to…** to change its virtual location. The actual files on disk are never touched.
- **Hide specific folders:** Right-click any file or folder inside a mount → **Ignore in Folder Bridge** to hide it. Excluding large folders like `node_modules` speeds up indexing.
- **NAS / network drives:** If changes aren't being picked up, the README recommends editing the mount, turning on **Use polling**, and setting the interval to around 3000–5000 ms.

---

## 7. What I Liked, and What to Watch Out For

**What I liked**

- I keep **exactly one** Obsidian window open and handle all my notes from it.
- One search covers every vault, and notes from different vaults can link to each other with `[[links]]`.
- Since files stay where they are, I didn't have to change any of my existing backup or sync setups.

**What to watch out for**

1. **Remember that this is an unofficial plugin.** It hasn't gone through the official review process, and it accesses your file system directly. Before installing, double-check that you're using the original repository (`tescolopio/Obsidian_FolderBridge`); there are several forks on GitHub.
2. **Back up first.** Back up any important vaults beforehand, and consider testing with **Dry-run mode** in the settings, which logs write operations instead of actually performing them.
3. **Plugins and themes follow the hub vault.** The mounted vaults' `.obsidian` settings are ignored, so any plugins you used in the original vaults need to be reinstalled in the hub.
4. **Links may break if you open the original vault on its own.** Links you create from the hub to notes in another vault only resolve inside the hub. If you open the original vault by itself, those links won't find their targets. It's also best not to edit the same file in the hub and the original vault at the same time.
5. **Check platform support.** Windows and Linux are tested, macOS hasn't been officially tested yet, Android supports only WebDAV and S3 mounts, and iOS isn't supported.

---

## Wrapping Up

A new window every time you switch vaults sounds like a small thing, but day after day it became a real source of stress. After installing Folder Bridge through BRAT and mounting every vault into a single hub, I now handle all my notes from one window while the files stay right where they've always been. If you've run into the same frustration, back up your vaults and give it a try.

### References

- Folder Bridge on GitHub: <https://github.com/tescolopio/Obsidian_FolderBridge>
- BRAT on GitHub: <https://github.com/TfTHacker/obsidian42-brat>
- BRAT documentation: <https://tfthacker.com/BRAT>
- Folder Bridge announcement on the Obsidian Forum: <https://forum.obsidian.md/t/new-plugin-folder-bridge-mount-your-folders-into-obsidian/111496>
