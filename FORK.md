# Fork: the Netflix button on a Xiaomi stick opens an app you choose

This branch of [EnnaKenT/cleanyourtv](https://github.com/EnnaKenT/cleanyourtv) adds one thing to the
reference launcher: the **Netflix** button on a Xiaomi MiTV-AESP0 stick's remote opens any app you map
to it in `app_keys`, once Netflix is disabled. It was offered upstream as Gathin23/cleanyourtv#4418 and
lives here instead, as a row in [SHOWCASE.md](https://github.com/Gathin23/cleanyourtv/blob/main/SHOWCASE.md).

## Why a normal key remap cannot do it

The stick's firmware gives the Netflix button keycode 1001 and consumes it in
`interceptKeyBeforeQueueing` without passing it on, so no accessibility service (the launcher's
`HomeKeyService`, Button Mapper) ever sees the key. Instead the system sends
`VIEW nflx://www.netflix.com/`. With Netflix disabled nothing answers that link, and the button does nothing.

## What this fork adds

- `NetflixLinkActivity`: an activity with no UI (`Theme.NoDisplay`) that answers `nflx://` links and
  opens the `app_keys` target of keycode 1001.
- It is **exported**, because the system's link has to reach it, so any app or web page can start it
  with an `nflx://` link; all it can do is open the app you mapped to 1001. It ships **disabled** in the
  manifest, and `MainActivity` switches it on only while `app_keys` maps 1001. Without a 1001 entry the
  launcher leaves `nflx://` links alone, so a TV with Netflix enabled never sees a chooser.
- No new permission, no network.

## Use it

1. Build and install this branch's launcher as the upstream [launcher guide](docs/launcher.md) describes.
2. Disable Netflix, which the [MiTV-AESP0 list](examples/xiaomi-mitv-aesp0/debloat.txt) leaves to you:
   `pm disable-user --user 0 com.netflix.ninja` (`pm enable com.netflix.ninja` brings it back).
3. Map the button in `launcher/app/src/main/res/values/config.xml`, or `"app_keys"` in a theme's
   `config.json`:

   ```xml
   <string-array name="app_keys" translatable="false">
       <item>1001:com.example.app</item>
   </string-array>
   ```

   Any `app_keys` target works, including actions such as `settings` or `library`.
4. Open Home once so the launcher switches the link handler on, then press the real Netflix button.

## Tested on

Xiaomi MiTV-AESP0 stick (PI.2055 firmware), Android TV 9, with the original remote, 23 Nov 2026. The
logcat (keycode 1001 → `VIEW nflx://www.netflix.com/` → `NetflixLinkActivity` → the mapped app) is in
Gathin23/cleanyourtv#4418. Other Xiaomi sticks and TVs that send `nflx://` for the button may work too,
but are untested.

Issues: [EnnaKenT/cleanyourtv issues](https://github.com/EnnaKenT/cleanyourtv/issues).
