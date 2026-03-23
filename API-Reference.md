# API Reference

All public types are in the `MysticCode.LogLens` namespace.

```csharp
using MysticCode.LogLens;
```

**See also:** [Getting Started](Getting-Started.md) | [Tag System](Tag-System.md) | [Runtime Overlay](Overlay.md)

---

## Contents

| Class / Type | Description |
|---|---|
| [LogLens](#loglens-static-class) | Primary logging API — `Log`, `Warning`, `Error`, `Exception`, type-based variants |
| [LogLensOverlay](#loglensoverlay) | Runtime overlay — visibility, zoom, filters, position, export, toasts |
| [LogLensCommands](#loglenscommands-static-class) | Overlay command bar — register/execute custom commands |
| [LensLogTagAttribute](#lenslogtagattribute) | Attribute for type-based tag resolution |
| [LogCollector](#logcollector-static-class) | Read-only access to captured entries and tag counts |
| [LogEntry](#logentry-struct) | Immutable struct for a single log entry |
| [Build Stripping](#build-stripping) | `LOGLENS_ENABLED` and `LOGLENS_DISABLE` compile-time defines |

---

## LogLens (static class)

The primary API for logging with tags and guaranteed jump-to-source in the Editor.

**Methods:** [Log](#log) | [Warning](#warning) | [Error](#error) | [Exception](#exception) | [Log\<T\>, Warning\<T\>, Error\<T\>](#logt-warningt-errort) | [SetMaxEntryCap](#setmaxentrycap) | [GetMaxEntryCap](#getmaxentrycap)

### Conditional Compilation

All `Log`, `Warning`, `Error`, and `Exception` methods carry three `[Conditional]` attributes:

```csharp
[Conditional("UNITY_EDITOR")]
[Conditional("DEVELOPMENT_BUILD")]
[Conditional("LOGLENS_ENABLED")]
```

When **none** of these symbols are defined, every `LogLens.Log/Warning/Error/Exception` call is erased at compile time. Zero overhead in release builds — no string allocations, no method calls, nothing.

**For production:** omit `LOGLENS_ENABLED` from Scripting Define Symbols. Calls made in the Editor and Development Builds still work automatically.

### Log

```csharp
public static void Log(string message, string tag = null)
```

Log an info-level message. Calls `Debug.Log` internally.

| Parameter | Type | Description |
|---|---|---|
| `message` | `string` | The log message. If it starts with `[TAG]`, the tag is extracted automatically. |
| `tag` | `string` | Optional explicit tag (highest priority). Converted to uppercase. |

```csharp
LogLens.Log("Player spawned");                  // no tag
LogLens.Log("[NET] Socket connected");           // tag: NET (from bracket)
LogLens.Log("Auth failed", "AUTH");              // tag: AUTH (explicit)
```

### Warning

```csharp
public static void Warning(string message, string tag = null)
```

Log a warning-level message. Calls `Debug.LogWarning` internally. Same parameters as `Log`.

### Error

```csharp
public static void Error(string message, string tag = null)
```

Log an error-level message. Calls `Debug.LogError` internally. Same parameters as `Log`.

### Exception

```csharp
public static void Exception(Exception exception, string tag = null)
```

Log an exception. Calls `Debug.LogException` internally.

| Parameter | Type | Description |
|---|---|---|
| `exception` | `Exception` | The exception to log. |
| `tag` | `string` | Optional explicit tag. Converted to uppercase. |

### Log\<T\>, Warning\<T\>, Error\<T\>

```csharp
public static void Log<TContext>(string message, string tag = null)
public static void Warning<TContext>(string message, string tag = null)
public static void Error<TContext>(string message, string tag = null)
```

Type-based logging. Resolves the tag from a `[LensLogTag]` attribute on `TContext`. If `tag` is provided, it takes priority over the attribute.

| Parameter | Type | Description |
|---|---|---|
| `message` | `string` | The log message. |
| `tag` | `string` | Optional explicit tag override. When set, overrides the `[LensLogTag]` attribute. |
| `TContext` | type param | The class or struct to resolve `[LensLogTag]` from. Inheritance is supported. |

```csharp
[LensLogTag("NET")]
public class NetworkManager : MonoBehaviour
{
    void OnConnect()
    {
        LogLens.Log<NetworkManager>("Connected");            // tag: NET (from attribute)
        LogLens.Warning<NetworkManager>("Latency spike");    // tag: NET
    }

    void OnAuthFail(string reason)
    {
        LogLens.Error<NetworkManager>(reason, "AUTH");       // tag: AUTH (explicit wins)
    }
}
```

### Jump-to-Source

All `LogLens.Log/Warning/Error` methods capture the caller's file path and line number via `[CallerFilePath]` and `[CallerLineNumber]`. Double-click a log row or press **F4** to open your IDE at the exact line.

`Debug.Log` entries fall back to stack trace parsing (first non-Unity frame).

### SetMaxEntryCap

```csharp
public static void SetMaxEntryCap(int cap)
```

Set the maximum number of log entries retained in memory. Takes effect immediately — oldest entries are dropped if the list exceeds the new cap. This modifies the runtime instance only; the Project Settings asset is not saved. Minimum value: 1.

Conditional — stripped in release builds.

### GetMaxEntryCap

```csharp
public static int GetMaxEntryCap()
```

Returns the current maximum entry cap. Always available (not conditional).

---

## LogLensOverlay

Runtime overlay MonoBehaviour for on-screen logs. Self-installs via `[RuntimeInitializeOnLoadMethod]`. Available when the overlay is enabled in Project Settings.

`Instance` is `null` when the overlay is disabled (`LOGLENS_DISABLE`) or hasn't installed yet (before `AfterSceneLoad`). Always null-check.

**Properties:** [Instance](#instance) | [ToastsMuted](#toastsmuted)
**Visibility:** [Show / Hide / Toggle](#show--hide--toggle)
**Zoom:** [SetZoom](#setzoom) | [ZoomIn / ZoomOut / ResetZoom](#zoomin--zoomout--resetzoom)
**Filtering:** [SetLevelFilter](#setlevelfilter) | [SetTagFilter / ClearTagFilter](#settagfilter--cleartagfilter)
**Layout:** [SetPosition](#setposition) | [SetSize](#setsize) | [ResetPositionAndSize](#resetpositionandsize)
**Actions:** [ClearLogs](#clearlogs) | [ResumeAutoScroll](#resumeautoscroll) | [ExportLogs](#exportlogs)
**Toasts:** [ShowToast](#showtoast) | [MuteToasts](#mutetoasts) | [ToastsMuted](#toastsmuted)

### Instance

```csharp
public static LogLensOverlay Instance { get; }
```

The singleton overlay instance, or `null` if disabled/not installed.

### Show / Hide / Toggle

```csharp
public void Show()
public void Hide()
public void Toggle()
```

Control overlay visibility. `Hide()` also defocuses the command bar.

### SetZoom

```csharp
public void SetZoom(float zoom)
```

Set the zoom level directly. Clamped to 0.5 (50%) – 3.0 (300%). Persisted via PlayerPrefs.

| Parameter | Type | Description |
|---|---|---|
| `zoom` | `float` | Zoom scale factor. 1.0 = 100%, 1.5 = 150%, etc. |

### ZoomIn / ZoomOut / ResetZoom

```csharp
public void ZoomIn()
public void ZoomOut()
public void ResetZoom()
```

`ZoomIn` and `ZoomOut` change the zoom by one step (0.1). `ResetZoom` returns to the project default from Settings.

### SetLevelFilter

```csharp
public void SetLevelFilter(bool log, bool warning, bool error)
```

Set which log levels are visible. The `error` parameter controls both `Error` and `Exception`.

| Parameter | Type | Description |
|---|---|---|
| `log` | `bool` | Show Info-level entries |
| `warning` | `bool` | Show Warning-level entries |
| `error` | `bool` | Show Error and Exception entries |

### SetTagFilter / ClearTagFilter

```csharp
public void SetTagFilter(string tag)
public void ClearTagFilter()
```

`SetTagFilter` toggles a tag in the filter set — call once to add, call again to remove. Pass `null` to clear all tag filters. `ClearTagFilter` removes all tag filters (shows all tags).

| Parameter | Type | Description |
|---|---|---|
| `tag` | `string` | Tag name to toggle, or `null` to clear all. |

### ClearLogs

```csharp
public void ClearLogs()
```

Clear all log entries from the overlay and LogCollector. Also clears the Unity Console. Resets scroll position and selection.

### ResumeAutoScroll

```csharp
public void ResumeAutoScroll()
```

Re-enable auto-scrolling to the latest entry. Auto-scroll is paused when the user scrolls up manually.

### SetPosition

```csharp
public void SetPosition(float x, float y)
```

Move the overlay panel to a specific screen position. Persisted via PlayerPrefs.

| Parameter | Type | Description |
|---|---|---|
| `x` | `float` | X position in screen pixels |
| `y` | `float` | Y position in screen pixels |

### SetSize

```csharp
public void SetSize(float w, float h)
```

Resize the overlay panel. Persisted via PlayerPrefs.

| Parameter | Type | Description |
|---|---|---|
| `w` | `float` | Width in screen pixels |
| `h` | `float` | Height in screen pixels |

### ResetPositionAndSize

```csharp
public void ResetPositionAndSize()
```

Clear persisted position/size from PlayerPrefs and reinitialize from project settings defaults on the next frame.

### ExportLogs

```csharp
public void ExportLogs()
```

Export all visible entries to a timestamped `.txt` file in `Application.persistentDataPath`. The file path is logged to the console.

### ShowToast

```csharp
public void ShowToast(string message, Color? color = null)
```

Show a custom toast notification. Not linked to any log entry — clicking it dismisses it.

| Parameter | Type | Description |
|---|---|---|
| `message` | `string` | The toast message text. Truncated to 120 characters. Rich text tags are stripped. |
| `color` | `Color?` | Optional accent color. Defaults to the overlay accent color. |

### MuteToasts

```csharp
public void MuteToasts(bool muted)
```

Mute or unmute toast notifications for this session. Not persisted — resets each play session.

| Parameter | Type | Description |
|---|---|---|
| `muted` | `bool` | `true` to suppress all new toasts, `false` to resume. |

### ToastsMuted

```csharp
public bool ToastsMuted { get; }
```

Whether toasts are currently muted. Session-only, not persisted.

---

## LogLensCommands (static class)

Registry for overlay command bar commands. Built-in commands (`help`, `clear`, `hide`, `export`, `zoom`) are registered automatically.

**Methods:** [RegisterCommand (No Args)](#registercommand-no-arguments) | [RegisterCommand (With Args)](#registercommand-with-arguments) | [UnregisterCommand](#unregistercommand) | [Execute](#execute) | [GetCommandNames](#getcommandnames) | [FilterByPrefix](#filterbyprefix) | [GetHelpText](#gethelptext)
**Reference:** [Built-In Commands](#built-in-commands)

### RegisterCommand (No Arguments)

```csharp
public static void RegisterCommand(string name, Action action, string description = null, string usage = null)
```

Register a command that takes no arguments.

| Parameter | Type | Description |
|---|---|---|
| `name` | `string` | Command name (case-insensitive, stored lowercase). |
| `action` | `Action` | Callback to execute. |
| `description` | `string` | Short description shown in `help` output. |
| `usage` | `string` | Usage hint (e.g. `"reload"`). Defaults to the command name. |

```csharp
LogLensCommands.RegisterCommand(
    "reload",
    () => SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex),
    "Reload the current scene"
);
```

### RegisterCommand (With Arguments)

```csharp
public static void RegisterCommand(string name, Action<string[]> action, string description = null, string usage = null)
```

Register a command that receives parsed arguments (whitespace-separated).

| Parameter | Type | Description |
|---|---|---|
| `name` | `string` | Command name (case-insensitive, stored lowercase). |
| `action` | `Action<string[]>` | Callback receiving the argument array. |
| `description` | `string` | Short description shown in `help` output. |
| `usage` | `string` | Usage hint (e.g. `"tag <name>"`). Shown in `help` output. |

```csharp
LogLensCommands.RegisterCommand(
    "tag",
    args => {
        if (args.Length > 0)
            LogLensOverlay.Instance?.SetTagFilter(args[0]);
    },
    "Filter overlay by tag",
    "tag <name>"
);
```

### UnregisterCommand

```csharp
public static void UnregisterCommand(string name)
```

Remove a previously registered command.

| Parameter | Type | Description |
|---|---|---|
| `name` | `string` | Command name to remove (case-insensitive). |

### Execute

```csharp
public static bool Execute(string input)
```

Parse and execute a command string. Returns `true` if the command was found and executed.

| Parameter | Type | Description |
|---|---|---|
| `input` | `string` | Raw command input (e.g. `"zoom 150"`). Parsed into command name + arguments. |

### GetCommandNames

```csharp
public static IReadOnlyCollection<string> GetCommandNames()
```

Returns all registered command names, sorted alphabetically. Useful for custom autocomplete implementations.

### FilterByPrefix

```csharp
public static List<string> FilterByPrefix(string prefix)
```

Returns command names matching the given prefix (case-insensitive). Used internally by the overlay autocomplete.

| Parameter | Type | Description |
|---|---|---|
| `prefix` | `string` | Prefix to match. Pass `null` or empty to get all commands. |

### GetHelpText

```csharp
public static string GetHelpText()
```

Returns a formatted help string listing all registered commands with their usage hints and descriptions.

### Built-In Commands

| Command | Action |
|---|---|
| `help` | List all commands |
| `clear` | Clear log entries (also clears Unity Console) |
| `hide` | Hide overlay |
| `export` | Export visible logs to file |
| `zoom <50-300>` | Set zoom percent |
| `toast <message>` | Show a custom toast notification |
| `toastzoom <50-300>` | Set toast zoom percent (independent from overlay) |
| `mute` | Toggle toast mute on/off (session only) |
| `autoshowerror` | Toggle auto-show overlay on error |

---

## LensLogTagAttribute

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct, Inherited = true)]
public class LensLogTagAttribute : Attribute
```

Assign a default tag to all `LogLens.Log<T>()` calls from a class or struct. Inherited by subclasses.

### Constructor

```csharp
public LensLogTagAttribute(string tag)
```

| Parameter | Type | Description |
|---|---|---|
| `tag` | `string` | The tag string. Stored uppercase. `null` or empty is treated as untagged. |

### Properties

| Property | Type | Description |
|---|---|---|
| `Tag` | `string` | The resolved tag (uppercase), or `null` if empty/null was passed. |

```csharp
[LensLogTag("NET")]
public class NetworkManager : MonoBehaviour { }

[LensLogTag("ENEMY")]
public class EnemyBase : MonoBehaviour { }
public class Goblin : EnemyBase { }   // LogLens.Log<Goblin>("hit") -> tag: ENEMY
```

---

## LogCollector (static class)

Read-only access to captured log entries. Main thread only.

**Properties:** [AllEntries](#allentries) | [TagCounts](#tagcounts) | [IsAtCap](#isatcap)
**Methods:** [Clear](#clear)
**Events:** [EntriesChanged](#entrieschanged)

### AllEntries

```csharp
public static IReadOnlyList<LogEntry> AllEntries { get; }
```

All captured log entries, oldest first. Do not modify.

### TagCounts

```csharp
public static IReadOnlyDictionary<string, int> TagCounts { get; }
```

Tag occurrence counts across all entries. Keys are non-null, non-empty tag names only.

### IsAtCap

```csharp
public static bool IsAtCap { get; }
```

`true` when the entry list has reached `MaxEntryCap`. Oldest entries are being silently dropped on each new log.

### Clear

```csharp
public static void Clear()
```

Clear all entries, tag counts, and the pending queue. Also clears the Unity Console (Editor and builds).

### EntriesChanged

```csharp
public static event Action EntriesChanged
```

Fired on the main thread when new entries are added or entries are cleared. Subscribe to refresh UI.

---

## LogEntry (struct)

Immutable struct representing a single log entry. Accessed via `LogCollector.AllEntries`.

| Property | Type | Description |
|---|---|---|
| `Id` | `long` | Unique, monotonically increasing ID |
| `Message` | `string` | Log message text |
| `StackTrace` | `string` | Stack trace (empty string if none) |
| `Type` | `LogType` | `Log`, `Warning`, `Error`, or `Exception` |
| `Timestamp` | `DateTime` | UTC capture time |
| `Frame` | `int` | `Time.frameCount` at capture |
| `Tag` | `string` | Resolved tag, or `null` if untagged |
| `SourceFile` | `string` | Caller file path (LogLens API only; `null` for `Debug.Log`) |
| `SourceLine` | `int` | Caller line number (LogLens API only) |
| `SourceMember` | `string` | Caller member name (LogLens API only) |

---

## Build Stripping

| Define | What It Strips | How to Set |
|---|---|---|
| `LOGLENS_ENABLED` | When **absent**, all `LogLens.Log/Warning/Error/Exception` and `SetMaxEntryCap` calls are erased at compile time. | Add to Scripting Define Symbols for dev builds; omit for release. |
| `LOGLENS_DISABLE` | When **present**, strips all overlay code. `LogLensOverlay` compiles as a no-op stub so references don't break. | Toggled automatically via Project Settings > LogLens > Overlay > Enable Overlay (uncheck). |

### Production Setup

1. **Omit** `LOGLENS_ENABLED` from Scripting Define Symbols
2. **Uncheck** Enable Overlay in Project Settings (adds `LOGLENS_DISABLE`)
3. Result: **zero LogLens code in your release build**

> **Note:** `LogLens.Log` calls are also active when `UNITY_EDITOR` or `DEVELOPMENT_BUILD` is defined — even without `LOGLENS_ENABLED`. This means you get logging in the Editor and in Development Builds automatically, and only need `LOGLENS_ENABLED` if you want logging in non-development builds.
