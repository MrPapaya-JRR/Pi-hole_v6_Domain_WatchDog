# Pi-hole v6 Domain WatchDog v1.09

Pi-hole v6 Domain WatchDog is a Windows helper app for Pi-hole v6 that watches your clipboard and dragged links for URLs, extracts the domains, and lets you send them to your Pi-hole block list in batches with simple confirmations.  
It reduces the manual copy/paste work of adding domains in the Pi-hole web interface.

---

## Features

- Automatic clipboard monitoring on a configurable timer  
- Drag-and-drop support from your web browser  
- Smart domain extraction from copied URLs  
- Batch blocking with per-domain confirmation dialogs  
- Regex-based blocking for domain families and subdomains  
- Fallback to exact-domain blocking if regex fails  
- Exceptions list so trusted domains are never collected  
- Built-in configuration editor with automatic backup  
- Help and About tabs with usage info and credits  
- Window size and column widths remembered between runs  
- Basic Pi-hole connection testing and error logging  

---

## Requirements

- Pi-hole v6 installation (Raspberry Pi, Linux, Windows, etc.)  
- Network access from this Windows machine to your Pi-hole web/API port  
- Valid Pi-hole admin password  
- Windows 11 (may work on older versions but is not guaranteed)  

---

## Installation and first run

1. Copy `Pi-hole_v6_Domain_WatchDog.exe` and `Pi-hole_v6_Domain_WatchDog.ini` into the same folder.  
2. Open `Pi-hole_v6_Domain_WatchDog.ini` in a text editor.  
3. Enter your Pi-hole IP/hostname, port, and admin password.  
4. Optionally adjust the clipboard check interval and any initial exceptions.  
5. Save the INI file.  
6. Run the program; it will load your configuration, test the Pi-hole connection, and open the main window.  

> Note: Only one instance of the program can run at a time. If you try to start it again, the existing window is brought to the front.

---

## Configuring `Pi-hole_v6_Domain_WatchDog.ini`

The INI file controls how the app connects to Pi-hole and how it behaves.

The URL that you use to access your Pi-Hole Dashboard gives you most of the values to
enter in the [MAIN] section.
 
For example: https://192.168.1.2:8489/admin/
https:// means YES we are using HTTPS   http:// means NO we are not using HTTPS   
The Pi-Hole IP in this case would be 192.168.1.2
The Port would be 8489
Only you know what your password is

### Core settings

- **PIHOLEIP**  
  IP address or hostname of your Pi-hole.  
  Example: `192.168.1.2` or `pi.hole`

- **PIHOLEPORT**  
  Port used by the Pi-hole web interface.  
  Example: `80`, `8080`, or another custom port.

- **PIHOLEPASSWORD**  
  The same admin password used for the Pi-hole web interface.

- **CHECKINTERVAL**  
  Seconds between clipboard checks (for example, `2`).

- **USEHTTPS**  
  - `0` – use HTTP  
  - `1` – use HTTPS  
  If your Pi-hole web interface uses `https://`, set this to `1`.

- **EXCEPTIONS**  
  Space-separated list of domains that should be ignored and never collected.  
  Example:  
  `EXCEPTIONS google.com reddit.com example.com`

### Layout settings

The program automatically stores your window size and Monitor tab column widths in the INI file.  
When you resize the window or adjust column widths and then exit, those values are saved and used at next startup.  
You normally do not need to edit these entries manually.

---

## Main window and tabs

The main window contains a tab control, a status line, and four primary buttons along the bottom.

### Monitor tab

Shows the domains collected from your clipboard and dragged links.

Columns:

- Number  
- Status (`PENDING`, `BLOCKED`, `FAILED`)  
- Domain  
- Original URL  
- Regex pattern  

Behavior:

- The app checks the clipboard every `CHECKINTERVAL` seconds.  
- If the text looks like a URL or path with a domain:
  - It extracts the domain name.
  - If the domain matches any exception, it is ignored.
  - If the domain is already in the list, it is skipped.
  - Otherwise, a new entry with status **PENDING** is added.

Drag-and-drop:

- Drag a link from your browser into the **Monitor** tab to add it as if it had been copied.

Buttons:

- **List URLs (L)** – refreshes the displayed list of URLs and statuses.  
- **Process URLs (P)** – starts the process of sending domains to Pi-hole.  
- **Clear List (C)** – removes all collected URLs from the list (with confirmation).  
- **Exit (ESC)** – closes the program and saves window/column settings.

### Exceptions tab

Manages domains that should never be collected or blocked.

You can:

- **Add** – enter a domain such as `example.com`. The program cleans off protocols and common prefixes.  
- **Edit** – modify an existing exception; duplicates are prevented.  
- **Delete** – remove a domain after confirmation.

Drag-and-drop:

- Drag a URL into the **Exceptions** tab to add its “apex” domain (e.g., `example.com` from `sub.example.com`) as a new exception.

All changes are written back into the INI file immediately.

### Configuration tab

Shows the full INI file in an editor.

Buttons:

- **Edit**  
  - First click: switches the editor into editable mode (background changes).  
  - Second click: prompts to save changes, creates a `.backup` copy, and writes the new INI file.  
  - After saving, you can choose to reload the configuration; if you do, the clipboard timer is restarted with the new `CHECKINTERVAL`.

- **Reload**  
  Discards unsaved edits and reloads the INI file from disk.

### Help and About tabs

- **Help** – embedded usage guide explaining:
  - How monitoring and drag-and-drop work
  - What each tab and button does
  - Meaning of configuration values
  - General tips and notes

- **About** – shows:
  - Program name and version  
  - Purpose and feature summary  
  - Author and link information  
  - License and requirements  
  - Credits and acknowledgements  

---

## Processing and blocking domains

When you click **Process URLs**:

1. The app authenticates with the Pi-hole v6 API using your configured IP, port, and password.  
2. A session ID is obtained and stored for later requests.  
3. Each **PENDING** domain is shown in a confirmation dialog with:
   - Item number
   - Domain
   - Shortened original URL
   - Regex pattern that will be submitted

For each dialog you can choose:

- **Yes** – attempt to block the domain.  
- **No** – skip this domain for now.  
- **Cancel** – stop processing further entries.

Blocking steps:

- First, a regex-based block is sent to the Pi-hole “deny regex” endpoint.  
- If that fails with a pattern-related error, the app retries with a plain “deny exact” domain entry.  
- If authentication fails mid-run, the app attempts to re-authenticate once and retries the same domain.

When finished, a summary dialog shows:

- Number of domains successfully blocked  
- Number that failed  
- Number skipped  
- Number remaining in the list  

---

## Tips and troubleshooting

- **No URLs appear**  
  - Check that `CHECKINTERVAL` is a reasonable positive number.  
  - Verify you are copying full URLs or dragging actual links.  
  - Confirm the domain is not in your `EXCEPTIONS` list.

- **Cannot connect to Pi-hole**  
  - Verify `PIHOLEIP` and `PIHOLEPORT` match your Pi-hole installation.  
  - Ensure your firewall or router is not blocking access from this PC.  
  - Set `USEHTTPS` according to how you normally access the Pi-hole dashboard.

- **Authentication errors**  
  - Confirm `PIHOLEPASSWORD` by logging into the Pi-hole web interface.  
  - Update the INI file if you recently changed the password and reload the configuration.

- **Layout issues**  
  - Drag the window edges to resize; new size is saved on exit.  
  - Adjust Monitor column widths by dragging their separators; they will be remembered on next start.

If this tool saves you time, consider supporting the author using the link provided on the About tab.
ko-fi.com/mrpapaya


;; v1.00 - Initial test version
;; v1.01 - Added minimum window size, improved the resizing behavior, and added About and Help tabs
;; v1.02 - Added an Exceptions tab with a list and buttons to Add, Edit, and Delete exceptions, with all changes saved to the INI file.
;; v1.03 - Added a configuration tab that displays the INI file content with Edit and Reload buttons.
;; v1.04 - Added debug logging
;; v1.05 - Added TestPiHoleConnection() upon startup
;; v1.06 - Trim #DQUOTE$ off of .ini entries in LoadConfiguration()
;; v1.07 - Changed flags parameter from 0 to #PB_HTTP_NoSSLCheck in httpRequest = statements so that the program will
;;         accept the self-signed certificate provided by Pi-hole. Also updated TestPiholeConnection() to treat Status
;;         401 As a success rather than an error. The test is only checking For reachability, Not authentication.
;;         Added Mutex routines.
;; v1.08 - Add drag & drop functionality to both the Monitor & Exceptions tab.
;; v1.09 - Window size & Monitor tab column widths are saved to the .ini file

