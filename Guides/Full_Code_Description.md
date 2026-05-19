# BotChat 🤖💬  
**A TCP/IP Multi-Client Chat with a Launcher Control Center & Popup Chat Windows (NiceGUI + Python Sockets)**

BotChat is a learning-oriented chat system that demonstrates a full client–server workflow using:
- **NiceGUI** for UI (Launcher + Chat windows)
- A custom **TCP server** implemented with Python sockets
- A simple, explicit **line-based protocol** (USERS / MSG / ACK / ERR / RENAME / AVATAR)

The system provides a dedicated **Launcher Control Panel** for operational control:
  - Start/stop the server
  - Spawn multiple chat users (each opens in its own popup window)
  - Display active users
  - Close all chat windows
  - Shutdown the entire system cleanly

In addition to multiple **chat popup windows** representing distinct users:
  - Sending messages (Public & Direct)
  - Changing your avatar image
  - Renaming your UserName
  - Opening a new Launcher window if needed

---

## Table of Contents 📌

- [Features](#features-) 
  - [Launcher (Control Center)](#launcher-control-center-)
  - [Chat Window](#chat-window-)
- [Project Structure](#project-structure-)
- [Architecture Overview](#architecture-overview-) 
  - [UI Layer (NiceGUI)](#-ui-layer-nicegui)
  - [TCP Server Layer (Sockets)](#%EF%B8%8F-tcp-server-layer-sockets)
  - [Shared State (In-Process)](#%EF%B8%8F-shared-state-in-process)
- [Requirements](#requirements-)
- [Configuration (IP / Ports)](#configuration-ip--ports-) 
  - [Deployment Modes (Evaluator Choice)](#deployment-modes-evaluator-choice-)
    - [Mode A — Single Machine (Localhost)](#mode-a--single-machine-localhost-%EF%B8%8F----)
    - [Mode B — Multi-Machine (LAN)](#mode-b--multi-machine-lan-----)
- [Installation](#installation-)
- [How To Run](#how-to-run-%EF%B8%8F) 
  - [Run via Launcher (Recommended)](#1-run-via-launcher-recommended-)
  - [Run Server Manually + UI](#2-run-server-manually--ui-%EF%B8%8F)
  - [Open Pages Manually (No Popup)](#3-open-pages-manually-no-popup-)
- [How to Use the Launcher](#how-to-use-the-launcher-) 
  - [Create a User](#create-a-user-)
  - [Server Toggle (ON/OFF)](#server-toggle-onoff-%EF%B8%8F)
  - [Show Active Users](#show-active-users-%E2%80%8D)
  - [Close All Chats](#close-all-chats-)
  - [Shutdown System](#shutdown-system-)
- [How to Use the Chat Window](#how-to-use-the-chat-window-) 
  - [Send Messages](#send-messages-)
  - [Rename Yourself](#rename-yourself-)
  - [Change Avatar](#change-avatar-)
  - [Scroll-Aware Unread Counter](#scroll-aware-unread-counter-)
  - [Open Launcher from Chat Window](#open-launcher-from-chat-window-)
- [Protocol Reference](#protocol-reference-) 
  - [Server → Clients (Pipe | separated)](#server--clients-pipe--separated-----)
  - [Client → Server](#client--server-----)
- [Important Notes & Warnings](#important-notes--warnings-%EF%B8%8F) 
  - [Shared UI State (In-Process)](#shared-ui-state-in-process)
  - [Single Launcher Instance Policy](#single-launcher-instance-policy)
  - [“SECURITY STATUS: ENCRYPTED”](#security-status-encrypted)
- [Troubleshooting](#troubleshooting-) 
  - [Connection Error in Chat Window](#connection-error-in-chat-window)
  - [Popup Blocked](#popup-blocked)
  - [Ghost Users / Not Disconnecting Cleanly](#ghost-users--not-disconnecting-cleanly)
- [Summary & Conclusions](#summary--conclusions-)

---

## Features ✨
### Launcher (Control Center) 🚀
- ✅ Server ON/OFF toggle with real-time status icon (cloud on/off)
    - Start server as a subprocess (local execution)
    - Stop server gracefully; fallback kill by port (macOS `lsof`)
- ✅ Spawn multiple users (each in a dedicated popup chat window)
- ✅ Display active users (live refreshed dialog)
- ✅ Close all chat windows via BroadcastChannel
- ✅ Full system shutdown: close chats → stop server → close launcher

### Chat Window 💬
- ✅ Global messages (“Everyone”)
- ✅ Direct messages (private to a selected user)
- ✅ Username change with:
  - client-side validation
  - server-side validation
  - server ACK synchronization
  - rollback timer (revert typed name if not confirmed)
- ✅ Avatar selection:
  - DiceBear-based avatars (seed-based)
  - optional background color selection (seed → bg cache)
  - server broadcast so all clients sync the avatar
- ✅ Scroll-aware unread counter:
  - “scroll to bottom” floating button
  - badge indicating unseen messages when user is scrolled up
- ✅ Clean disconnect handshake:
  - sends `CMD:QUIT`
  - waits briefly for server-side close
  - reduces “ghost users”
- ✅ Launcher access from chat window:
  - A dedicated shortcut button allows opening the Launcher directly from any chat window.
  - If the Launcher is already running, the existing window is focused instead of creating a duplicate.

---

## Project Structure 📁
Core files (based on your code):
- [`Run_App.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Run_App.py) – starts NiceGUI server and opens the Launcher in Chrome app-mode
- [`UI_Router.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/UI_Router.py) – routes `/` to Launcher and `mode=chat` to Chat
- [`Launcher_UI.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Launcher_UI.py) – server toggle, create users, active users dialog, shutdown logic
- [`Chat_UI.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Chat_UI.py) – chat messages, sending, DM, rename, avatar sync, scrolling behavior
- [`Main_Server.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Main_Server.py) – TCP server (protocol handling, broadcast, private messages)
- [`Common_Setups.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Common_Setups.py) – configuration (SERVER_IP, ports, Chrome path)
- [`State_Globals.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/State_Globals.py) – UI state (messages, active users, avatars)

---

## Architecture Overview 🧠
BotChat consists of three layers:

### 🌐 UI Layer (NiceGUI)
- **Launcher UI:** [`BotChat/Launcher_UI.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Launcher_UI.py)
- **Chat UI:** [`BotChat/Chat_UI.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Chat_UI.py)
- Routing controlled by:
  - **UI Router:** [`BotChat/UI_Router.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/UI_Router.py)
  - `/` → launcher  
  - `/?mode=chat&nickname=<name>` → chat window

### 🖥️ TCP Server Layer (Sockets)
- **The Server:** [`BotChat/Main_Server.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Main_Server.py)
- Listens on: `HOST=0.0.0.0`, `PORT=<SERVER_PORT>`
- Accepts multiple clients.
- Each client handled in a dedicated thread.
- Maintains:
  - `online_users: Dict[nickname -> socket]`
  - `online_users_lock` for concurrency
  - routing of global/direct messages
  - rename requests (ACK/ERR)
  - avatar broadcasts
 
### 🗂️ Shared State (In-Process)
- Configuration & Global Variables:
  - **Setups:** [`BotChat/Common_Setups.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/Common_Setups.py)
  - Keeps:
    - `SERVER_IP`: depends on the needs
    - `Ports`: server and UI
    - `Chrome path`:  app mode
- The UI (launcher + chat windows in the same NiceGUI process) share Python globals:
  - **Global States for UI:** [`BotChat/State_Globals.py`](https://github.com/Shlomi-Hazan/Chat-Bot-/blob/main/PartTwo/BotChat/State_Globals.py)
  - Stores:
    - `messages`: chat history entries
    - `active_users_list`: server-synced online names
    - `avatar_urls`: synced avatar URL map
    - `avatar_seeds`: stable avatar identity across renames
    - `user_colors_cache`: stable bg color per avatar seed
    
---

## Requirements ✅
- Python 3.10+ recommended
- Packages: `nicegui`

---

## Configuration (IP / Ports) 🌐
All configuration is located in:

`BotChat/Common_Setups.py`

```py
SERVER_IP = '10.0.0.16'
SERVER_PORT = 8081
CHAT_UI_PORT = 8080
```

### Deployment Modes (Evaluator Choice) 🧪
The evaluator may choose one of the following deployment modes by adjusting `SERVER_IP`.

#### Mode A — Single Machine (Localhost) 🖥️ --->
Server and UI run on the same machine.

```py
SERVER_IP = '127.0.0.1'
```
**Recommended for:**
  - Local testing
  - Demonstrations
  - Single-machine evaluation

#### Mode B — Multi-Machine (LAN) 🌐 --->
Server runs on one machine, UI runs on another machine in the same local network.

(If needed-> see the guide for further explanation- [How to Find your IP](/Guides/How_to_find_your_IPv4_Address.md))

```py
SERVER_IP = '<SERVER_MACHINE_LAN_IP>'
# Example:
SERVER_IP = '10.0.0.16'
```
**Notes:**
  - This demonstrates a real client–server architecture.
  - NiceGUI still runs locally on the UI machine at: `http://localhost:<CHAT_UI_PORT>/`

---

## Installation 📦

### 1) Clone the Repository
```py
git clone <repository-url>
cd BotChat
```

### 2) Install Dependencies
```py
pip install nicegui 
```

*(Optional) Use a virtual environment if required by your system.

---

## How To Run ▶️
So where do we start?

### 1) Run via Launcher (Recommended) 🛸
This is the default and recommended execution method.
```py
python BotChat/Run_App.py
```

**What happens:**
  - NiceGUI server starts on: `http://localhost:<CHAT_UI_PORT>/`
  - The Launcher opens automatically in a Chrome “app window” (popup-like UI).
  - You can start/stop the TCP server and spawn chat windows from the Launcher.

### 2) Run Server Manually + UI 🕹️
If you prefer to start the server manually:

**Step A — Start the TCP server**
```py
python BotChat/Main_Server.py
```

**Step B — Start the NiceGUI UI**
```py
python BotChat/Run_App.py
```

*In this mode you can keep the Launcher toggle OFF and just use it to spawn users.

### 3) Open Pages Manually (No Popup) 🌍
If Chrome app-mode popup is blocked, you can open the pages manually.

**Launcher:**
```py
http://localhost:<CHAT_UI_PORT>/
```

**Chat:**
```py
http://localhost:<CHAT_UI_PORT>/?mode=chat&nickname=NAME
```

**For Example:**
```py
http://localhost:8080/?mode=chat&nickname=User1234
```

---

## How to Use the Launcher 🚀

### Create a User 👤

- Enter a nickname (1–9 characters)

- Press Enter or click "LAUNCH CHAT"

- A new chat window opens as a popup
  

### Server Toggle (ON/OFF) ☁️

- The Launcher can start the server as a subprocess

- It also detects real server status by attempting a fast TCP connection to `SERVER_IP:SERVER_PORT`

- “When turning the server OFF, the launcher closes all chat windows and clears local UI state.”
  

### Show Active Users 👨‍💻

- Opens a dialog that refreshes live containing all the connected users 

- Users are synced through a dedicated observer TCP connection (launcher acts as a hidden client)
  

### Close All Chats ❌

- Uses a BroadcastChannel('chat_control_v1')

- Each chat window listens and performs a clean shutdown sequence
  

### Shutdown System 😵

A full shutdown performs:

  - Close all chat windows

  - Stop the server shortly after

  - Close the launcher

  - Terminate the launcher process (optional)

---

## How to Use the Chat Window 💬

### Send Messages ⏩

- Type a message and press Enter (or click send)

- Select target:

  - Everyone → ALL

  - A specific username → direct message

**Validation rules:**

  1) Empty messages are blocked

  2) Sending to yourself is blocked
     

### Rename Yourself 🫆

- Edit from the “My Name” field (1–9 chars) and press Enter

- Client performs local validation (reserved names, duplicates)

- Server validates again and responds with:

  - `ACK|System|<old>|NAME_CHANGED|<new>` (requestor)

  - `RENAME|<old>|<new>` (broadcast)

  - `MSG|System|ALL|...|...` (informative system message)

**Rollback timer:** *If you typed a name but did not confirm (Enter), the input field reverts after ~8 seconds.


### Change Avatar 📸

- Click the avatar (top-left) to open the avatar picker dialog

- Choose an avatar and optionally lock a background color

- Client sends: `CMD:AVATAR:<url>`

- Server broadcasts: `AVATAR|<username>|<url>`

- All clients update the avatar in real-time


### Scroll-Aware Unread Counter 👇

When the user scrolls up:

- New messages increment a counter badge

- A floating “scroll to bottom” button appears
  
- When the user returns to bottom: Counter resets automatically


### Open Launcher from Chat Window 🚀

Each chat window includes a Launcher shortcut button (top-right corner).

- Clicking the button attempts to open the Launcher.
- If the Launcher is already open:
  - The existing Launcher window is brought to focus.
- If the Launcher is closed:
  - A new Launcher window is opened automatically.

This allows users to regain full system control even if the Launcher was closed intentionally or unintentionally.

---

## Protocol Reference 📡
All messages are newline-delimited (\n).


### *Server → Clients (Pipe | separated)* 💾 --->

  **1) USERS — Online Users Update**
  
    Format: `USERS|System|ALL|user1,user2,user3`
  
  **2) MSG — Chat Message**
  
    Format: `MSG|<sender>|<target>|<msg_id>|<content>`
  
  **3) ACK — Confirmation to Requestor**
  
    Format: `ACK|System|<old>|NAME_CHANGED|<new>`
  
  **4) ERR — Error**
  
    Format: `ERR|System|<who>|NAME_TAKEN`
  
  **5) RENAME — Rename Broadcast (for sync)**
  
    Format: `RENAME|<old>|<new>`
  
  **6) AVATAR — Avatar Broadcast**
  
    Format: `AVATAR|<username>|<url>`


### *Client → Server* 🪪 --->

  **A) Normal Messages (Colon : separated)**
  
    Format: `<recipient>:<msg_id>:<text>`
  
  - recipient is ALL or an exact username
  
  - msg_id is generated client-side
  
  **B) Commands**
  
  - Quit / clean disconnect: `CMD:QUIT`
    
  - Rename request: `CMD:NAME_CHANGE:<new_name>`
    
  - Avatar update: `CMD:AVATAR:<avatar_url>`
  
  ---

## Important Notes & Warnings ⚠️

### *Shared UI State (In-Process)*

The NiceGUI app hosts both Launcher and Chat pages in the same Python process.
Therefore, State_Globals.py is used as a shared state container for:
  - messages
  - active_users_list
  - avatar maps and seeds

This is intentional for the scope of the project and simplifies UI synchronization.


### *Single Launcher Instance Policy* 

The system enforces a single Launcher instance by design.

- If a Launcher window is already open, additional launch attempts from chat windows are blocked.
- Instead of opening a duplicate, the existing Launcher window is focused.

This behavior is implemented using browser-side coordination
(`BroadcastChannel` combined with a `localStorage` heartbeat mechanism)
to prevent multiple control panels from running simultaneously.

This design choice avoids race conditions, conflicting server controls,
and inconsistent system state.


### *“SECURITY STATUS: ENCRYPTED”*

This label is UI-only and does not imply actual encryption on the TCP layer.
TCP communication is currently plaintext.

---

## Troubleshooting 🧯

### *Connection Error in Chat Window*

If you see a connection error screen:

  - Make sure Main_Server.py is running
  
  - Check SERVER_IP and SERVER_PORT in Common_Setups.py
  
  - If using LAN mode, verify both machines are on the same network and the port is open

### *Popup Blocked*

If the launcher or chat popups do not open:

  - Allow popups for `localhost`
  
  - Use the “Open Pages Manually” section above

### *Ghost Users / Not Disconnecting Cleanly*

The client performs a clean handshake:

  - sends `CMD:QUIT`

  - waits briefly for server-side close
    
If the browser is force-killed, cleanup may be delayed until socket closes.

---

## Summary & Conclusions 🎓

BotChat demonstrates a complete TCP-based client–server chat system implemented in Python, combining a multi-threaded server with a modern NiceGUI frontend.

The project showcases core networking concepts such as message routing, protocol design, user synchronization, and clean connection handling.

The separation between the Launcher control layer and independent chat windows enables centralized management alongside isolated user sessions.

Overall, the system serves as a clear and practical academic example of client–server architecture and real-time UI coordination.
