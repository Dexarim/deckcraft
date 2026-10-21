# KDE Plasma / Linux desktop integration

Updated 2026-10-21. Integration checks and remaining limitations are recorded separately from
features implemented in code; a successful build is not a Plasma GUI test.

## Global Menu

The desktop host exports the existing text-menu catalogue through the shared
`craft-desktop` backend. X11/XWayland uses KDE window properties and the AppMenu registrar;
native Wayland uses KDE's appmenu protocol on the existing winit surface. The bundled additive
winit patch supplies the safe same-connection surface accessor; no raw pointers are rebuilt.
Local ribbon controls stay available without Plasma or its Global Menu widget.

Enable Plasma's Global Menu widget in the desired panel manually. To disable export for one
launch, set `DECKCRAFT_NO_GLOBAL_MENU=1`; `KDE_NO_GLOBAL_MENU=1` is also respected. Display-server
selection is intentionally unchanged: use `DECKCRAFT_WAYLAND=1` to test the native Wayland
window separately from the default X11/XWayland path when `DISPLAY` is present.

The root editing window exports the menu; fullscreen slideshow and secondary presenter windows
do not receive the editor menu. Stale-document events and events during modal dialogs are
discarded. Application-specific live Plasma verification remains separate from backend tests.

## Appearance and window layout

Fresh desktop profiles select system appearance. Existing `ui.json` files without
`systemTheme` keep their explicit Light/Dark choice. Choose **System**, **Light**, or **Dark**
under File → Account → Appearance, or enable **Follow system appearance** in Preferences.
The dark-mode shortcut/button selects an explicit theme. Cancel in Preferences restores the
previous choice. Application colour tokens are retained; these are not a pixel-for-pixel Breeze
theme or a promise to import every KDE colour role.

Interface size multiplies the monitor scale reported by eframe; it does not override the
monitor's physical scale. Ribbon tabs and command groups can scroll horizontally when they do
not fit. Use a horizontal touchpad gesture or Shift+wheel over the corresponding strip. The
local ribbon remains available independently of desktop menu support.

Linux keeps native window decorations. The integrated macOS titlebar hints are not applied on
Linux. Clipboard services retain their selection owner while the application is running and
enable arboard's Wayland data-control support on Linux and FreeBSD. Desktops without that
protocol can still use the toolkit's ordinary clipboard path; rich-format support depends on
the available backend.

## Reproducible verification

Use a synthetic sample and isolated settings; never overwrite working presentations or the
user's Plasma configuration. The repository-wide gate is `CARGO_BUILD_JOBS=2 cargo xtask ci`.
Targeted regressions can be run with:

```sh
CARGO_BUILD_JOBS=2 cargo test -p deckcraft -p deckcraft-ui-egui desktop_
```

The optional sibling `craft-fonts` checkout is absent in the current development environment.
Cargo reports the existing fallback-font warning; no fonts or manifests were substituted for
verification. Its build script watches missing manifest paths, so consecutive Cargo commands
may rebuild the font-dependent crates even when no source changed.

Local automated result (2026-10-21): `cargo test --offline -p deckcraft -p deckcraft-ui-egui`
passed all 36 host tests and 227 UI tests; four opt-in screenshot/long-soak tests stayed ignored.
The final offline desktop build and workspace/all-targets Clippy with `-D warnings` also passed.
This includes old-preference migration, live appearance choice changes, cancel restoration,
menu state/dispatch/stale-document guards (including deferred Copy/Cut/Paste), image-only paste without a duplicate text paste,
retained clipboard ownership, and narrow-ribbon reachability at five raster scales. The existing
Replace Font GPU screenshot test was also run explicitly; its normal ribbon and dialog images
were visually inspected.

Native Wayland smoke in the current Plasma session (2026-10-21) confirmed the same-surface menu
binding without an X11 window ID, calls from the real `org.kde.plasmashell` consumer, Ruler's
checked-state change, disabled actions while About is open, and dirty-close confirmation.
System appearance initially matched the real portal's dark answer. Normal and narrow Insert
ribbon screenshots were inspected. Physical mixed-DPI, live desktop-theme changes, all portal
dialog cases, and cross-application clipboard formats remain separate manual checks.

For manual checks use a temporary `XDG_CONFIG_HOME`, `DECKCRAFT_NO_RECOVERY=1`, and `--sample`.
Check native Wayland (`DECKCRAFT_WAYLAND=1`) separately from X11/XWayland; a Wayland desktop
session does not prove the application's window is native Wayland.

1. At 100%, 125%, 150%, 175%, and 200%, resize the window and scroll both ribbon strips;
   the last tab and the last Insert control must remain reachable.
2. Move the window between monitors with different scales. Verify text, icons, popovers,
   hit testing, resizing, native window buttons, fullscreen show, and presenter view.
3. With System selected, switch Plasma light/dark appearance. Repeat with explicit Light and
   Dark and confirm those are not overridden. Test Preferences → Cancel.
4. Copy synthetic plain/rich text and a picture between DeckCraft and another application;
   test both immediate and delayed paste while DeckCraft remains open. Do not use private data.
5. Open/Save As use rfd's XDG portal backend. Check cancellation, selected filters, overwrite
   confirmation, parent/focus behaviour, and unchanged unsaved documents after cancellation.

Automated UI input tests exercise five fractional raster scales, not physical mixed-DPI
monitors. Native compositor, portal dialog, and clipboard interoperability need live-session
checks; they must not be inferred from these unit tests.
