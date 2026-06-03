# lrcmd Future Experience

## Vision

`lrcmd` should feel like a small, trustworthy macOS utility for developers who use a US keyboard and want left/right Command single-taps to switch input sources.

The ideal experience is not "install a daemon and hope it works." It is:

1. Install with one clear command.
2. Run setup.
3. Choose the input sources for left and right Command.
4. Grant Accessibility permission in System Settings.
5. Start using the keyboard naturally.

The user should always understand what changed on their machine, where files were installed, how to verify the result, and how to undo it.

## Target User Journey

### Install

The developer starts with:

```sh
curl -fsSL https://install.ultrahope.dev/lrcmd | sh
```

The installer should:

- detect macOS and CPU architecture
- download the matching release archive
- verify the archive checksum when release metadata is available
- install `lrcmd` and bundled `inctl` into a stable user-local location
- print every installed path
- avoid surprising service changes before setup is complete

Expected installed shape:

```text
~/Applications/lrcmd/
  bin/lrcmd
  bin/inctl
```

### Setup

After install, the developer runs:

```sh
lrcmd setup
```

Setup should guide the user through input-source selection instead of assuming every machine has the same IDs enabled.

The flow should:

- show available input sources using bundled `inctl`
- offer sensible defaults when `ABC` and Japanese Hiragana are available
- let the user choose left Command and right Command mappings
- create `~/.config/lrcmd/config.json` only after showing what will be written
- preserve existing config unless the user explicitly chooses to replace it
- generate or update the LaunchAgent plist
- start or restart the service with `launchctl`
- show the Accessibility permission step clearly

The setup output should be concrete and calm:

```text
Installed:
  /Users/you/Applications/lrcmd/bin/lrcmd
  /Users/you/Applications/lrcmd/bin/inctl

Configured:
  left Command  -> ABC
  right Command -> Hiragana

Created:
  /Users/you/.config/lrcmd/config.json
  /Users/you/Library/LaunchAgents/dev.ultrahope.lrcmd.plist

Next:
  Open System Settings > Privacy & Security > Accessibility
  Enable lrcmd
  Then run: lrcmd restart
```

### Accessibility Permission

`lrcmd` cannot fully automate Accessibility permission, so the product should make this limitation feel expected rather than broken.

The CLI should:

- detect whether Accessibility permission is currently granted
- explain that macOS requires manual approval
- identify the exact binary that should be enabled
- offer a command to re-check status
- avoid hiding config or LaunchAgent problems behind permission errors

Useful commands:

```sh
lrcmd status
lrcmd doctor
lrcmd restart
```

### Daily Use

After setup and permission approval:

- tapping left Command switches to the configured left input source
- tapping right Command switches to the configured right input source
- holding Command with another key behaves like normal Command usage
- pressing both Command keys together does not trigger accidental switching

The utility should stay quiet during normal operation. When it fails, errors should be visible in predictable logs and diagnosable with `lrcmd status` or `lrcmd doctor`.

### Maintenance

The developer should be able to manage the service without remembering `launchctl` details:

```sh
lrcmd status
lrcmd restart
lrcmd stop
lrcmd uninstall
```

These commands should describe exactly which files and services they touch.

`lrcmd uninstall` should:

- stop and unload the LaunchAgent
- remove the LaunchAgent plist
- optionally remove config after confirmation
- optionally remove installed binaries after confirmation
- leave unrelated input-source settings untouched

## Trust And Safety

Because the installation path starts with `curl | sh`, the project should over-communicate safety.

The installer and README should provide:

- a way to inspect the script before running it
- release archive URLs
- checksum verification
- a clear list of installed files
- a clear list of service changes
- an uninstall path
- no background service registration before the user runs setup

No command should silently overwrite an existing config, LaunchAgent, or installed binary without telling the user.

## Distribution Shape

The release archive should be simple:

```text
lrcmd-v0.1.0-macos-arm64.tar.gz
  bin/lrcmd
  bin/inctl
  README.md
  LICENSE
```

The hosted install script at `https://install.ultrahope.dev/lrcmd` should be thin:

- detect platform
- choose release archive
- download and verify
- install files
- tell the user to run `lrcmd setup`

The main setup behavior should live in `lrcmd`, not in the hosted shell script, so that install, reinstall, repair, and local development all share the same logic.

## Near-Term Feature Direction

The next implementation work should move toward this future in small slices:

- add `lrcmd setup`
- add LaunchAgent plist generation
- add `launchctl bootstrap`, `kickstart`, `bootout`, and status handling
- add interactive input-source selection
- add `lrcmd status`, `restart`, `stop`, `doctor`, and `uninstall`
- add release archive packaging
- add the hosted installer script

The important part is the experience: developers should feel that `lrcmd` is small, inspectable, reversible, and respectful of macOS permissions.
