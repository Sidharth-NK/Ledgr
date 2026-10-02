# SETUP

- **Gradle wrapper download timeouts.** The wrapper's default 10 s timeout is too short on a slow route to the download host. If it recurs, set
  `networkTimeout=120000` *inside* `gradle/wrapper/gradle-wrapper.properties`
  (setting it in the shell does nothing).
- **Phone not visible to adb** ("no permissions ... missing udev rules"). Fixed by `sudo apt install android-sdk-platform-tools-common`, then replug.
- **Use `./gradlew`, never the system Gradle** (apt version is 4.4.1 and irrelevant). Do not use the apt `sdkmanager` package either; use the one from `cmdline-tools` under `~/Android/Sdk`.
- **SDK licenses:** `yes | sdkmanager --licenses`.

