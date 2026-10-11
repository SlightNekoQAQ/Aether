# Android Agent Mode display orientation

Agent Mode virtual displays now support a saved orientation in **Settings → Agent Mode → Virtual Display**:

- **Device default** preserves the physical display dimensions used by previous versions.
- **Portrait** puts the shorter edge on the width axis.
- **Landscape** puts the longer edge on the width axis, useful for landscape-only games.

This is an Android-only feature: iOS does not implement Android virtual displays. The preference is represented in the shared settings model so settings serialization and import/export preserve it.

Changing the saved orientation stops the managed display. Relaunch the target application in Agent Mode afterwards. No main-device display rotation, resolution or global Android settings are changed.

The agent can also explicitly request a session orientation:

```json
{"action":"launch","target":"com.miniclip.plagueinc","orientation":"landscape"}
```

`orientation` is accepted only for `start` and `launch`, with values `device`, `portrait`, or `landscape`. A session override does not change the saved setting and lasts until the managed display is stopped. Changing orientation recreates the display rather than rotating only its output surface. A `start` call changing orientation must be followed by `launch` to put the target app on the new display.

The saved setting is also exposed by `aether_config_get` under `agent_mode.display_orientation`, and can be changed using `aether_config_set` with category `agent_mode` and settings `{"display_orientation":"landscape"}`.

## Geometry contract

The virtual display and screenshot ImageReader are created with the same dimensions. Display state supplies those dimensions to the live preview and normalized touch mapping. The live TextureView is recreated with a new display ID/geometry so its callbacks do not retain old buffer dimensions.

Tap/swipe coordinates remain normalized 0–1000 independently on each axis. Do not send screenshot pixels as coordinates. Convert using `x / image_width * 1000` and `y / image_height * 1000`.

## Validation

Automated tests cover default/portrait/landscape/square dimensions, invalid dimensions, screenshot sizing, normalized coordinates, tool schema, legacy settings defaults and serialization round-trip.

Hardware validation is still necessary for both Root and Shizuku:

1. Start with Device default; confirm existing portrait behavior.
2. Choose Landscape, relaunch a landscape-only app, and confirm reported width > height, landscape screenshot and undistorted live preview.
3. Tap visible controls and verify cursor and input alignment.
4. Capture screenshots while live preview is attached; ensure preview resumes.
5. Switch to Portrait, relaunch, and repeat input/capture checks.
6. Restart Aether and confirm the saved selection persists.
7. Test explicit launch orientation, then a screenshot without orientation: geometry should stay unchanged. Stop and launch again: the saved setting should apply.

The optional Build Agent Mode APK workflow runs unit tests and produces an **unsigned release APK** with application ID `com.baimoqilin.aether`. Standard Android devices will not install an unsigned APK; sign it appropriately for normal installation. Back up application data before replacing an existing installation.
