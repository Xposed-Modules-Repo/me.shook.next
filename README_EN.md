# HookNext

[中文](README.md) | **English**

This is HookNext's release and feature-description repository in Xposed Modules Repo. Its complete user guide stays synchronized with the HookNext home repository. It does not contain the Android client source code. Feature descriptions track the current HookNext implementation; when an installed version differs, refer to that release's notes and in-app UI.

HookNext is a visual Xposed module manager and runtime analysis tool for Android. It lets users configure method, argument, return-value, and field hooks without writing an Xposed module, then inspect and manage the results from the Android app, a LAN web interface, or an MCP client.

HookNext is the fully upgraded successor to [SimpleHook](https://github.com/littleWhiteDuck/SimpleHook). It keeps SimpleHook's central idea of expressing hooks as configuration while redesigning the configuration model, editor, runtime compatibility layer, record storage, and remote-management experience. It is not merely a renamed APK, and old configurations should not be assumed to work unchanged. HookNext includes an importer for SimpleHook custom configurations, but imported rules must still be reviewed, enabled, and tested.

> Hooks change how target apps run. Use HookNext only on your own devices and apps or in environments where you have explicit authorization. Records may contain accounts, tokens, user input, file contents, or cryptographic material. Protect them accordingly. Do not use HookNext to bypass payment, access-control, anti-cheat, or other security systems.

## Contents

- [HookNext and SimpleHook](#hooknext-and-simplehook)
- [Feature overview](#feature-overview)
- [Requirements and three important concepts](#requirements-and-three-important-concepts)
- [Installation and in-app updates](#installation-and-in-app-updates)
- [First-time setup](#first-time-setup)
- [Custom hook configuration tutorial](#custom-hook-configuration-tutorial)
- [Extension configurations](#extension-configurations)
- [Viewing and managing records](#viewing-and-managing-records)
- [DEX browser and Smali import](#dex-browser-and-smali-import)
- [Backup, migration, and Frida export](#backup-migration-and-frida-export)
- [Web management and MCP](#web-management-and-mcp)
- [Troubleshooting](#troubleshooting)

## HookNext and SimpleHook

HookNext is the next-generation version of SimpleHook, not an unrelated tool. Both provide configurable Java/Smali hooks, while HookNext introduces systematic upgrades:

| Area | What changed in HookNext |
| --- | --- |
| Configuration editor | Hook mode, value type, arguments, conditions, and field data are structured and validated before saving |
| Value types | Values use explicit `BOOLEAN`, `INT`, `STRING`, `JSON_DATA`, `NULL`, and other types instead of relying mainly on text suffixes |
| Member discovery | A DEX/APK/APKS browser and Java/Smali signature import reduce manual entry errors |
| Runtime | Shared hook semantics support modern libxposed API 101/102 and the legacy Xposed API 82 path |
| Configuration storage | Room stores the source of truth, while target-readable JSON files are synchronized copies with retryable failures |
| Record system | Custom and extension records are separated and support search, filters, marks, details, deletion, and export |
| Extensions | Categorized controls cover algorithms, UI, security, network, WebView, clipboard, files, exits, signatures, and HotFix |
| Remote management | An embedded Ktor server provides a Vue web interface and permission-tiered MCP service |
| Data tools | Complete backup/restore, HookNext config import and sharing, SimpleHook migration, and Frida JavaScript export are built in |

Important migration boundaries:

1. HookNext and SimpleHook use different internal configuration models. Do not copy their internal files manually.
2. HookNext can recognize one or more SimpleHook custom configurations copied to the clipboard.
3. Imported configuration groups are disabled by default. Exact duplicates are unselected initially but can still be selected under a stronger warning.
4. SimpleHook extension switches, app settings, and runtime state are not migrated as equivalent settings.
5. Explicit value types, wildcards, and conditions must be reviewed rule by rule and tested in a controlled app.

## Feature overview

### Custom hooks

- Replace method return values.
- Replace one or more arguments before a call.
- Skip a method and return `null`.
- Change static or instance fields before or after a selected method.
- Read or write a static field without a trigger method.
- Record arguments, return values, both, or static/instance fields.
- Gate a rule with argument or return-value conditions.
- Match an exact overload or use method and argument wildcards.
- Produce primitive values, strings, random strings, JSON-created objects, arrays or collections, or `null`.

### Analysis and management

- Browse classes, methods, and fields from installed apps, APK, APKS, or DEX files.
- Search records, group them by type or app, mark, delete, inspect, and export the current group/filter result.
- Synchronize configurations and read records in Root, Shizuku, or Normal mode.
- Back up configurations and settings manually or automatically and restore ZIP snapshots.
- Import configs from the clipboard, files, Android shares, or HTTPS URLs, then resolve conflicts from a preview.
- Select and share one or more HookNext custom configs.
- Export selected custom configurations as Frida JavaScript.
- Manage configurations, extensions, and records from the same device or a trusted LAN browser.
- Expose HookNext tools to compatible clients through read-only, read/write, or full-access MCP tiers.
- Check GitHub stable releases automatically or manually, then install an APK only after verification.

## Requirements and three important concepts

### 1. Android version

HookNext currently requires Android 8.0 (API 26) or later. Actual compatibility also depends on the ROM, Xposed framework, storage restrictions, and target-app implementation.

### 2. Xposed runtime

Hooks are executed by an Xposed runtime. HookNext currently supports:

- Modern libxposed API 101/102.
- A legacy Xposed API 82 compatibility path.

Enable HookNext in a compatible Xposed manager and add each target app to the module scope. Modern service APIs may let HookNext assist with scope synchronization. Legacy frameworks and runtimes without the required service capability need manual scope management in the framework manager.

### 3. Work mode

Work mode controls configuration files, record files, and selected app operations. It does not replace Xposed:

| Mode | Intended environment | Main purpose | Notes |
| --- | --- | --- | --- |
| Root | A device with root access | High-privilege file and process operations through libsu | Root is requested on first use; grant it only to a trusted build |
| Shizuku | Shizuku is installed and running | Authorized file and process operations over IPC | Reconnect or grant permission again after Shizuku restarts or authorization expires |
| Normal | No Root/Shizuku, or ordinary access is preferred | Allowed direct access on Android 10 and below; SAF directory access on Android 11+ | Follow the prompt to grant directories such as `Android/media`; system restrictions still apply |

A hook rule is expected to have the same meaning in every work mode. The modes mainly change how HookNext synchronizes configuration to the target process, reads records, and performs helper operations such as launch or force-stop.

## Installation and in-app updates

For the initial installation, obtain the APK from the HookNext home [Releases](https://github.com/littleWhiteDuck/HookNextHome/releases) page. After installation, you can check manually under Settings → About. Automatic checks are enabled by default and run only when HookNext is opened. The last-check time is stored locally, limiting update requests to once every 24 hours. When a newer stable release is found, Settings and About display an update indicator.

If a GitHub Release contains a verifiable APK, you can download it from GitHub or a third-party mirror such as `wget.la`, `cdn.gh-proxy.org`, or `fastgit.cc`, or open the Release page in a browser. A mirror receives the original download URL, so choose one according to your network conditions and trust requirements. Files returned by mirrors still go through the same verification.

After download, HookNext checks the Release asset metadata, file size, official SHA-256, APK format, application package, signing certificate, and version code. It invokes the Android package installer only after every check passes. Android may first require permission to install unknown apps. A verified cached APK can be reused, while stale APKs and incomplete downloads are cleaned automatically.

Automatic checks do not automatically download or silently install an update. If a Release has no verifiable APK, verification fails, or the installed app uses a different signing certificate, open the Release page and verify the source manually.

## First-time setup

1. Download a trusted APK from the HookNext home [Releases](https://github.com/littleWhiteDuck/HookNextHome/releases) page and install it.
2. Enable the HookNext module in a compatible Xposed manager.
3. Add only the apps you intend to test to HookNext's module scope.
4. Open HookNext, select `Root`, `Shizuku`, or `Normal` in Settings, and complete the requested authorization.
5. Return to Home, create a custom configuration, and select the target app.
6. Start with a record-only rule to verify member matching before adding a rule that changes behavior.
7. Save. If HookNext reports that the data was saved but not written to the configuration file, fix the permission or directory issue and retry synchronization.
8. Fully stop and restart the target app, trigger the relevant feature, and inspect Records.

HookNext saves configurations to its database first and then synchronizes target-readable JSON files. A successful database save is not rolled back if file synchronization fails. This distinction explains why “saved” does not always mean that the target process has already received the new configuration.

## Custom hook configuration tutorial

### Recommended workflow

For an unfamiliar target method:

1. Confirm the class, method, arguments, and return type with the DEX browser or a Smali signature.
2. Start with “Record arguments and return value”; do not change behavior yet.
3. Restart the target app and trigger the feature once.
4. Verify the actual values and stack trace in Records.
5. Narrow the rule to an exact overload and add conditions if needed.
6. Only then switch to return, argument, or field modification.
7. Change one variable at a time. Disable the rule and restart immediately if the app becomes unstable.

All examples below refer to a fictional local test app:

```java
package me.example.demo;

public final class DemoService {
    public static String environment = "production";
    private boolean debugPanelVisible = false;

    public boolean isDemoFeatureEnabled() {
        return false;
    }

    public String buildGreeting(String name, int repeat) {
        return "Hello " + name;
    }

    public Profile loadLocalProfile(String id) {
        return new Profile(id, "Guest");
    }

    public void initialize() {
        debugPanelVisible = false;
    }
}
```

The snippets explain configuration fields only. They are not real apps or Java/Xposed source templates.

### Parts of a rule

| Item | Meaning |
| --- | --- |
| Enabled | Per-rule switch; the app configuration itself must also be enabled |
| Hook mode | Return, argument, field, interception, or record action |
| Class name | Fully qualified owner of the trigger method, such as `me.example.demo.DemoService` |
| Method name | A regular name, constructor `<init>`, or wildcard `*` |
| Argument types | Identify the exact overload; order and count must match |
| Conditions | String conditions evaluated against selected arguments or the return value |
| Replacement value | Explicitly typed value written to an argument, return value, or field |
| Field details | Field owner, name, and hook point; used only in field modes |
| Memo | Maintenance note with no runtime matching effect |

Use the class that actually declares each class member, method, or field. Exact lookup currently targets declared members and does not automatically walk the superclass hierarchy. Obfuscated names must also match the installed target version.

### The ten hook modes

| Mode | Timing | Changes behavior | Main purpose |
| --- | --- | --- | --- |
| Replace return value | Before the call without a return condition; after it with one | Yes | Replace a method result |
| Replace arguments | Before the call | Yes | Replace one or more arguments |
| Break method execution | Before the call | Yes | Skip the original method and return `null` |
| Change static field | Immediately, or before/after a trigger method | Yes | Write a static field |
| Change instance field | Before/after a trigger method | Yes | Write a field on the current object |
| Record return value | After the call | No | Store the method result |
| Record arguments | After the call | No | Store call arguments |
| Record arguments and return value | After the call | No | Store both arguments and result |
| Record static field | Immediately, or before/after a trigger method | No | Read a static field |
| Record instance field | Before/after a trigger method | No | Read a field on the current object |

#### Replace return value

Goal: make the test method `isDemoFeatureEnabled()` return `true`.

```text
Mode: Replace return value
Class: me.example.demo.DemoService
Method: isDemoFeatureEnabled
Arguments: none
Value type: BOOLEAN
Value: true
```

Without a return condition, HookNext sets the result before invocation and skips the original method. This avoids its side effects, but also means that logging, state updates, and I/O in that method never run.

With a return condition, the original method must run first. HookNext checks the original result after the call and replaces it only when the condition matches. One example is replacing the value only when the original result is `false`.

Do not use return replacement on constructors or `void` methods. `NULL` can be valid for reference types, but returning `null` from a primitive-return method may fail during unboxing or crash the target app.

#### Replace arguments

Goal: replace only the second argument of `buildGreeting(String, int)` with `3`.

```text
Mode: Replace arguments
Class: me.example.demo.DemoService
Method: buildGreeting
Argument 0 type: java.lang.String
Argument 0 replacement: not set
Argument 1 type: int
Argument 1 value type: INT
Argument 1 replacement: 3
```

Arguments are changed before the original method runs. An argument without a replacement still identifies the overload but keeps its runtime value. The explicit value type must be compatible with the target parameter.

Add an `Equal / demo` condition to argument 0 to change argument 1 only when the first runtime value is `demo`. Multiple configured conditions must all match.

#### Break method execution

This mode skips the original method before invocation and returns `null`. It has no replacement-value field.

It is most suitable for controlled test methods returning references or `void`. A primitive-return method may fail while unboxing `null`. Skipping constructors, initialization, locks, resource release, or state commits can also corrupt app state. Prefer a correctly typed return replacement and a narrow condition in those cases.

#### Change a static field

Goal: set `DemoService.environment` to `staging`.

An immediate static-field rule has no trigger method:

```text
Mode: Change static field
Hook point: Not set
Field owner: me.example.demo.DemoService
Field name: environment
Value type: STRING
Value: staging
```

HookNext attempts the write once when the configuration is loaded. If the app overwrites the field later, select a trigger method and use the Before or After hook point.

#### Change an instance field

An instance field belongs to an object and therefore requires one of that object's methods or constructors as a trigger. To show the test panel after `initialize()`:

```text
Mode: Change instance field
Class: me.example.demo.DemoService
Method: initialize
Arguments: none
Hook point: After method execution
Field name: debugPanelVisible
Value type: BOOLEAN
Value: true
```

Instance-field modes cannot use an unset hook point. Choose After when the original method initializes the field, or Before when the method itself must observe the replacement.

#### The five record modes

“Record return value,” “Record arguments,” and “Record arguments and return value” write after method execution, so they capture the argument array and result as they exist at the end of the call. A static field can be recorded immediately or around a trigger method. An instance field must be recorded before or after a method on the current object.

Record rules do not intentionally change results, but serializing complex objects, collecting stack traces, and writing high-frequency records still add overhead. Start with an exact method and argument list rather than recording every method in a class.

### Value types

HookNext uses explicit value types:

| Type | Input and purpose |
| --- | --- |
| `BOOLEAN` | `true` or `false` |
| `BYTE` / `SHORT` / `INT` / `LONG` | Integer text such as `42` or `-1` |
| `FLOAT` / `DOUBLE` | Floating-point text such as `3.14` |
| `CHAR` | One character |
| `STRING` | Literal string; empty input means an empty string |
| `RANDOM_STRING` | Charset, length, and every-call, stable-reuse, or timed-refresh strategy |
| `JSON_DATA` | A target class name plus object or array JSON, constructed with the target ClassLoader, reflected type information, and Gson |
| `NULL` | `null`; only use it where the target type permits it |

`JSON_DATA` accepts both object roots `{}` and array roots `[]`. For a structurally simple object that Gson can create, specify its target class directly:

```java
public final class Profile {
    public String id;
    public String name;
}
```

```text
Mode: Replace return value
Target: loadLocalProfile(java.lang.String)
Value type: JSON_DATA
Target class: me.example.demo.Profile
JSON: {"id":"local-001","name":"Test User"}
```

Return values, arguments, instance fields, and static fields can all use `JSON_DATA`. When a target member retains a concrete generic signature such as `List<Profile>`, HookNext reads its reflected `Type` and uses the element type to parse a non-empty collection:

```text
Target method declaration: java.util.List<me.example.demo.Profile>
Value type: JSON_DATA
Target class: java.util.ArrayList
JSON: [{"id":"local-001","name":"Test User"}]
```

The configured class selects a compatible concrete implementation such as `ArrayList` or `LinkedList`, while the target signature supplies `Profile` and nested generic information. If only a raw type is available, a type variable is unresolved, or generic information cannot be read, parsing falls back to the configured raw class. For legacy compatibility, the default empty object `{}` is converted to `[]` only for recognized collection classes; non-empty JSON is never rewritten.

Objects that require custom deserialization, system handles, Binder, Context, or special construction flows still may not be created correctly by Gson. A conversion failure produces an error record; disable the rule and verify the target type, generic signature, and JSON shape.

### Methods, constructors, and wildcards

- `methodName = <init>` matches a constructor.
- `methodName = *` matches any declared method in the class; synthetic methods are excluded.
- `*` in one argument position matches any type at that position, while the argument count still has to match. That position cannot have a replacement, but it may have a condition.
- A single `**` argument entry matches any argument list.
- `<clinit>` cannot be configured.

For example, `*(**)` approximates “all declared methods in this class.” It is a broad, high-risk, high-overhead rule that may hit lifecycle, thread, I/O, and internal bridge methods. Use it only briefly in an isolated test, then replace it with an exact signature.

### Argument and return conditions

Available operators are:

```text
Equal (=)
NotEqual (!=)
Greater (>)
GreaterOrEqual (>=)
Less (<)
LessOrEqual (<=)
MatchRegex (Regex)
```

HookNext converts an argument or result to a string before comparing it. The expression direction is “entered condition value `operator` runtime value,” matching the preview shown by the editor. `Greater` and `Less` currently perform lexicographical string comparison, not numeric comparison. For example, an entered value of `"10"`, operator `<`, and runtime value `"2"` match. Regular expressions use Kotlin `Regex` syntax and are validated before saving.

Every configured condition must match before a rule proceeds. Return conditions apply only to Replace return value, Record return value, and Record arguments and return value. Because an original result is required, a return-replacement rule with a return condition cannot skip the original method.

### Complete example: record first, then modify conditionally

Suppose the test method is:

```java
public String buildGreeting(String name, int repeat) {
    return "Hello " + name;
}
```

First add a record rule:

```text
Mode: Record arguments and return value
Class: me.example.demo.DemoService
Method: buildGreeting
Argument types: [java.lang.String, int]
```

Save, restart the target app, and invoke it once. Verify arguments 0 and 1, the return value, and the stack trace in Records.

Then add a modification rule:

```text
Mode: Replace return value
Class: me.example.demo.DemoService
Method: buildGreeting
Argument types: [java.lang.String, int]
Argument 0 condition: Equal / demo
Value type: STRING
Value: Hello from HookNext
```

The result is now replaced only when `name.toString() == "demo"`. Argument conditions can be evaluated before invocation, so a matching call skips the original method when no return condition is present.

## Extension configurations

Extensions provide predefined hooks for common framework and system APIs. Both the master “Enable extension” switch and each feature switch must be enabled. A child switch has no effect while the master switch is off.

| Category | Configurable capabilities |
| --- | --- |
| Basic | Hook activation toast; extension record switch, cache, per-record size, stack trace, Base64, and hex rendering |
| Algorithms | Base64 encode/decode, message digest, HMAC, and Cipher records, with digest/HMAC/Cipher algorithm-family filters |
| UI | Record or adjust Dialog, Toast, and PopupWindow; block Dialog by keyword or View ID; record click callbacks |
| Security | Filter accelerometer, gyroscope, and motion sensors; block common contact queries; hide common ADB status reads |
| JSON | Record `JSONObject` and `JSONArray` creation or writes |
| Other | Record signature reads, configure signature replacement, record/block/filter clipboard access, monitor files, record Intent and Application entry, intercept exits, and record uncaught exceptions |
| Network | Return “not detected” through common VPN-detection paths |
| WebView | Record `loadUrl` URLs/headers and force WebView debugging on |
| HotFix | Experimental DEX patch loading; failures are always recorded, while Applied and No Patch records are optional |

Extensions target common Android API paths. They cannot guarantee coverage of custom implementations, native code, reflection, or vendor modifications. Broader coverage increases both overhead and compatibility risk.

To use HotFix, enable both the extension master switch and Enable hot fix for the target app, place `.dex` patches in the Dex path shown on that screen, then fully stop and restart the target app. Failures are always recorded. Enable Record normal results only when Applied and No Patch records are also useful.

HotFix is experimental and depends on Android version, ClassLoader behavior, compiler optimization, and target-app structure. During staging, HookNext validates DEX magic, file counts, sizes, and source consistency while copying patches into the target app's private `code_cache`; it then marks them read-only, verifies permissions, and only then attempts injection. The cache also binds a patch to fingerprints of the target's current base and split APKs. If the target app changes while the patch source remains unchanged, loading is rejected until the patch is copied or replaced again to bind it explicitly to the new version. This prevents an old patch from being applied accidentally, but it does not prove that the patch is compatible with the new target.

## Viewing and managing records

Records are divided into Custom Hook and Extension Hook sources. The Records UI can:

- Summarize by record type or app.
- Search by keyword and filter read, unread, marked, or unmarked entries.
- Show structured details, raw content, stack traces, and encoded representations.
- Mark entries and delete one record, a group, all read records, or everything.
- Export the current group/filter result in a readable format or as raw JSON containing all fields.

Custom and extension records have separate output settings. Stack traces, Base64, hex, and large-object serialization can significantly increase storage use. Disable unneeded representations and use a reasonable truncation length for high-frequency methods.

Target processes write records to sharded files, and HookNext reads them into its local database. If no record appears, check both whether the hook matched and whether the current work mode can read the target app's record directory.

## DEX browser and Smali import

The DEX browser reads classes, methods, and fields from installed apps, `.apk`, `.apks`, or `.dex` files and sends a selected member to the editor. This is safer than manual entry for overloaded methods.

The editor also accepts Smali/JVM member signatures such as:

```text
Lme/example/demo/DemoService;->buildGreeting(Ljava/lang/String;I)Ljava/lang/String;
```

It normalizes that signature to:

```text
Class: me.example.demo.DemoService
Method: buildGreeting
Arguments: [java.lang.String, int]
Return type: java.lang.String
```

Common conversions:

| JVM/Smali | Java |
| --- | --- |
| `I` | `int` |
| `Z` | `boolean` |
| `Ljava/lang/String;` | `java.lang.String` |
| `[B` | `byte[]` |
| `[[Ljava/lang/String;` | `java.lang.String[][]` |

Constructors, arrays, object descriptors, field staticness, and `void` are validated during import. A plain field signature does not encode staticness, so the editor asks you to confirm whether the rule should use a static- or instance-field mode.

## Backup, migration, and Frida export

### Backup and restore

After selecting `Documents/HookNext` under Settings → Backup and restore, you can:

- Create a ZIP snapshot of custom configurations, extension configurations, and settings.
- Trigger automatic backups after changes, using automatic, hourly, or daily granularity for the latest backup.
- Select and restore a ZIP from the current directory.

A complete restore first shows a read-only preview, then replaces current custom configurations, extension configurations, and settings. It is not a merge: existing configs absent from the backup are removed. Create a snapshot of the current state first and restore only trusted backups.

### Importing and sharing HookNext configs

On Android, Home imports from the clipboard or an HTTPS URL, while text and files shared by other apps enter the same preview flow. Settings → Backup and restore can also open a JSON or config-exchange ZIP file directly. The web interface accepts pasted JSON or a selected JSON file and uses the same conflict rules. Before import, the preview shows format, app, rule count, installation status, and conflict type:

- Exact duplicates are unselected by default. Select one manually under the stronger warning only when it should be imported anyway.
- When a different config already targets the same package, explicitly choose Keep as new or Replace latest.
- If the target app is not installed, the config is saved only in HookNext's database; no target directory, synchronized file, or scope request is created.

Sharing packages selected custom configs as a config-exchange ZIP containing semantic config data only. It excludes database IDs, app settings, and temporary state. A complete-backup ZIP always follows the replacement restore flow above and cannot be merged as an ordinary config import.

### Migrating from SimpleHook

1. In SimpleHook, copy one or more custom configurations to the clipboard.
2. Open Settings → Backup and restore in HookNext.
3. Select Import SimpleHook configurations.
4. Review recognized items and duplicate markers, then select what to import.
5. Inspect every class, argument list, value type, field, and wildcard after import.
6. Imported configuration groups are disabled by default. Enable and save them manually after review.
7. Confirm Xposed scope, restart the target app, and test a small surface first.

The importer covers SimpleHook custom hook modes, including its JSON-return and random-string forms. Unsupported legacy features, extension configuration, and environment settings must be configured again in HookNext.

### Frida script export

The Backup and restore screen can generate Frida JavaScript from selected custom configurations. The result is editable generated code, not an unconditional compatibility guarantee for every mode or target. Review class-loading timing, overloads, value types, and exception handling before running it.

## Web management and MCP

### Web management

Under Settings → LAN web access, select a port, access scope, and password, then start the foreground service.

- Localhost limits access to the Android device itself.
- LAN exposes the displayed LAN address to computers or phones on the same network.
- The web interface shows status, manages custom and extension configurations, and browses, marks, or deletes records.
- Ports must be in `1024–65535`; changing server settings may require a restart.

Use LAN mode only on a trusted network and enable a password. Do not expose the port directly to the public internet. Android may also require notification permission and unrestricted background execution for reliable service operation.

### MCP

The MCP endpoint is `/mcp` under the selected web address. It supports three permission tiers:

| Tier | Capabilities |
| --- | --- |
| Read-only | Query status, apps, configurations, and records |
| Read/write | Read-only plus enabling, disabling, and updating configurations and extensions |
| Full access | Read/write plus destructive actions such as deleting records, force-stopping apps, and changing work mode |

Copy the generated MCP client configuration from Settings instead of entering tokens manually. Copy a new configuration after the password changes or a token expires. Without a web password, any device able to reach the service may be able to use the selected capabilities. Do not expose MCP on an untrusted network, especially at Full access.

## Troubleshooting

### A saved configuration has no effect

Check these in order:

1. HookNext is enabled in the Xposed manager.
2. The target app is in module scope; legacy runtimes generally require manual scope management.
3. Both the app configuration and individual rule are enabled.
4. Saving did not report a file-write failure or unavailable work mode.
5. Class, method, argument count, and argument types match the exact overload.
6. The target app was fully stopped and restarted, not merely sent to the background.
7. Error records do not report class, method, field, or value-conversion failures.

### Root or Shizuku works, but no hook runs

Root, Shizuku, and Normal are file and helper-operation modes. They do not install or replace Xposed and do not automatically put the target app into module scope.

### A condition compares values unexpectedly

Conditions convert values to strings. `Greater` and `Less` are lexicographical rather than numeric. For numeric ranges, use an unambiguous string representation or implement the logic in controlled test code or a purpose-built hook.

### Break method execution crashes the app

Break returns `null`. Primitive returns, constructors, and methods whose side effects are required may not tolerate it. Use a correctly typed return replacement and narrow argument conditions instead.

### JSON data conversion fails

Confirm that the target process can load the class, the JSON root and fields match the target structure, and the configured class is compatible with the return, argument, or field type. Concrete generic signatures are used to parse collection elements; raw types or unresolved type variables may produce generic values such as maps. System objects, Context, Binder, and objects with special creation flows are usually poor candidates.

### HotFix reports Target app not bound or Target app changed

Target app not bound usually means the patch came from an older cache and has not been bound to the current APK. Target app changed means the base or a split APK changed while the patch source did not. Verify that the patch still targets the installed app version, then copy or replace it again in the Dex path and restart the target. Do not bypass the fingerprint check to keep using an old cache.

### Records grow quickly or the app becomes slow

Avoid `*(**)`, disable unnecessary algorithms, stack traces, Base64, and hex output, target an exact method, reduce per-record size, and clean old records regularly.

### Normal mode cannot read or write files

Android 11+ applies stricter storage rules. Grant the correct SAF directory again when prompted. If access remains unavailable, choose Shizuku or Root only after understanding their permission implications.

### A computer cannot open the web interface

Confirm that the server is running, LAN rather than Localhost is selected, both devices are on the same network, the port is available, and no firewall, VPN, guest-network isolation, or background restriction is blocking access.

## Compatibility and responsible use

- Xposed hooks depend on implementation details. App updates, obfuscation, dynamic loading, native code, and vendor ROMs can invalidate working rules.
- Broad hooks, field modification, signature replacement, exit interception, and HotFix can destabilize a target. Back up first and test in a recoverable environment.
- HookNext can validate only part of the static type relationship. Assignment happens in the target process, and runtime failures are written as error records.
- This guide describes the current HookNext implementation. Names and capabilities may evolve; the installed app and latest release notes are authoritative for a specific version.

When reporting a problem, include the HookNext version, Android version, Xposed framework and API version, work mode, whether the target is in scope, and a sanitized error record. Never publish tokens, signatures, account data, or complete sensitive call contents.
