# Keyboard Shortcuts

Every keyboard shortcut in LogLens — Editor window and runtime overlay.

---

## Editor Window

| Shortcut | Action |
|---|---|
| **Ctrl+F** | Focus the search field |
| **Escape** | Close Options panel; or clear search if the search field is focused |
| **Ctrl+C** | Copy the selected entry's message |
| **Ctrl+Shift+C** | Copy selected entry with full details: `HH:mm:ss.fff  LEVEL  [TAG] message` + stack trace |
| **F4** | Jump to source — opens your IDE at the file and line for the selected entry |
| **F5** / **Ctrl+R** | Force refresh — rebuild the tree view |
| **Enter** | Toggle the stack trace panel for the selected entry |
| **Home** | Jump to the first entry and select it |
| **End** | Jump to the last entry and select it |
| **Up / Down** | Navigate the log list |

---

## Runtime Overlay

| Shortcut | Action |
|---|---|
| **F2** (configurable) | Toggle overlay visibility. Change the key in Project Settings > LogLens > Overlay > Toggle Key. |
| **Ctrl+=** / **Ctrl+NumPad+** | Zoom in |
| **Ctrl+-** / **Ctrl+NumPad-** | Zoom out |
| **Ctrl+0** | Reset zoom to default |

---

## Command Bar (Overlay)

| Key | Action |
|---|---|
| **Tab** / **Down** | Move to the next autocomplete suggestion |
| **Up** | Move to the previous suggestion |
| **Enter** | Execute the selected or typed command |
| **Escape** | Clear the field and dismiss autocomplete |

---

## Notes

- On macOS, **Ctrl** maps to **Cmd** for standard shortcuts (Cmd+C, Cmd+F, etc.)
- The overlay toggle key (default F2) is configurable in **Project Settings > LogLens > Overlay > Toggle Key**
- Keyboard shortcuts in the overlay work in Editor Play mode and standalone builds. On mobile and console, use the [public API](Overlay.md#public-api) instead.
