# 180 XR Video Stream

A free, local-first DLNA/UPnP media server for VR180, 3D SBS, top/bottom, and
regular 4K video. Point Meta Quest, a TV, or any DLNA client at your PC or
phone and stream the original files over your own network — no transcoding,
no cloud, no ads.

<p align="center">
  <img src="assets/images/app_logo.png" width="240" alt="VRstream logo">
</p>

## Why this app

- **Your files stay yours.** Nothing is uploaded, copied, or re-encoded.
- **Made for headsets.** Share VR180 and 3D folders and browse them from
  DeoVR, Skybox, Pigasus, or any DLNA/UPnP player.
- **No monetization.** Test ads only (production ads disabled by default); premium purchase integration present but product setup required. No trackers, no accounts.

## Features

- **Direct file streaming** — original bytes over HTTP with full byte-range
  support, so seeking works on large 4K/8K files.
- **SSDP discovery** — automatic detection by DLNA clients on the local
  network (`239.255.255.250:1900`), including `ssdp:alive` / `ssdp:byebye`
  notifications.
- **UPnP MediaServer** — device description, ContentDirectory, and
  ConnectionManager endpoints with DIDL-Lite metadata.
- **Browse and search** — folder tree, virtual playlist container, paging,
  and standard UPnP sort/search capability responses.
- **Real previews** — cached JPEG thumbnails for image files and video frames
  on Android and Windows; the app renders previews on demand, and DLNA artwork
  is advertised whenever a cached JPEG exists.
- **Playlists** — build `.m3u` playlists from library items and expose them to
  DLNA clients.
- **Cross-platform** — Android, Windows, macOS, and Linux. iOS/iPadOS project
  scaffolding is included.
- **Android background serving** — foreground service with a persistent
  notification while the server is running.
- **Android Storage Access Framework** — share folders and individual files
  without broad storage permissions; original files are never modified.
- **Auto-start** — optionally start serving when the app opens.
- **Two themes** — neon purple interface and an OLED-black dark mode.
- **Diagnostics** — server address, port, platform, CPU, and memory summary.

## Requirements

- Flutter 3.47 or newer (stable channel), Dart 3.13 or newer.
- Windows 10/11 with Visual Studio Desktop C++ workload for Windows builds.
- Android SDK/NDK for Android builds; macOS and Xcode for Apple platforms.
- A local network that allows multicast traffic for SSDP discovery. If
  discovery is blocked, clients can usually connect to the server address
  manually.

## Quick start

```bash
flutter pub get
flutter run -d windows      # or: -d macos, -d linux, or an Android device
```

In the app:

1. Add one or more folders (or individual videos) with **Add Folder** /
   **Add Videos**.
2. Press **Start Server**.
3. Open your DLNA client on the same network and select
   **180 XR Video Stream**.
4. Browse to the shared folder and play a file.

The default port is `8200`. The address and port are shown in the app after
the server starts. Stop the server when you are finished.

## Build

```bash
flutter analyze
flutter test

flutter build windows --release
flutter build apk --release
flutter build appbundle --release
flutter build macos --release
flutter build linux --release
```

Artifacts:

| Platform | Output |
| --- | --- |
| Windows | `build/windows/x64/runner/Release/vrstream.exe` |
| Android | `build/app/outputs/flutter-apk/app-release.apk` / `build/app/outputs/bundle/release/app-release.aab` |
| macOS | `build/macos/Build/Products/Release/180 XR Video Stream.app` |
| Linux | `build/linux/x64/release/bundle/vrstream` |

Android release signing is configured through `android/key.properties`; copy
`android/key.properties.example` and provide your keystore. See
[docs/RELEASING.md](docs/RELEASING.md) for the full checklist, and
[docs/STORE_LISTING.md](docs/STORE_LISTING.md) for store metadata.

Launcher icons and splash images are generated from
`assets/images/topbar_logo_cyan.png`:

```bash
python tool/generate_icons.py    # requires Pillow
```

## Project layout

```text
lib/
├── main.dart                 # App shell and server screen
├── core/
│   ├── config/               # App configuration
│   ├── errors/               # Exception types
│   ├── logging/              # Console logger
│   ├── platform/             # Platform bridges, paths, thumbnails
│   └── ui/                   # Shared visual widgets
├── diagnostics/              # Device and server diagnostics
├── dlna/
│   ├── http/                 # HTTP server, range handling, MIME dispatch
│   ├── ssdp/                 # SSDP discovery and notifications
│   └── upnp/                 # Device/service descriptions, ContentDirectory
├── media/                    # Media library, scanner, playlists, MIME types
├── security/                 # Local-only and path-validation guards
└── settings/                 # Persistence and settings UI

assets/images/                # Runtime branding assets
docs/                         # Release docs, store copy, design references
tool/generate_icons.py        # Reproducible launcher-icon generation
tools/media_probe_harness/    # Standalone native probe harness
test/                         # Unit, integration, widget, and platform tests
```

## How it works

1. The scanner indexes only the folders and files the user shares.
2. `DlnaHttpServer` serves device descriptions, SOAP control endpoints, media
   bytes, thumbnail JPEGs, and playlists on the configured port.
3. `SsdpServer` advertises the MediaServer on the local network and answers
   `M-SEARCH` requests.
4. `ContentDirectory` turns the in-memory library into DIDL-Lite so clients
   can browse, search, and resolve playable resources.
5. Media URLs resolve to bytes on disk (or through the Android SAF bridge);
   nothing is transcoded.

Security boundaries:

- The HTTP handler rejects absolute paths, `..` segments, malformed
  percent-encoding, NUL bytes, and symlinks that escape a shared root.
- The app does not make outbound internet requests.
- There is no authentication, so only run the server on networks you trust.
  See [PRIVACY.md](PRIVACY.md).

## Testing

```bash
flutter analyze
flutter test
```

The suite covers HTTP range parsing and streaming, SSDP discovery, UPnP
device descriptions, ContentDirectory/DIDL output, path-traversal rejection,
thumbnail generation, media scanning, SAF ID mapping, settings persistence,
playlists, multi-select UI, and Windows-specific paths.

## Documentation

- [Building and releasing](docs/RELEASING.md)
- [Store listing and metadata](docs/STORE_LISTING.md)
- [Changelog](CHANGELOG.md)
- [Privacy policy](PRIVACY.md)
- [Design references](docs/design/README.md)

## License

[MIT](LICENSE) © 2026 XR180.

180 XR Video Stream is an independent project and is not affiliated with Meta,
DeoVR, Skybox, or any DLNA/UPnP vendor. All trademarks belong to their
respective owners.
