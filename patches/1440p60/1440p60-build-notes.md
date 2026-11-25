# POCO X5 Pro 1440p60 camera patches

These patches target normal Video mode on the main rear camera of the POCO X5 Pro (`redwood`), crDroid 13.1 build `v13.1-20261107`, with Xiaomi Camera `5.1.001370.1`. They are ROM build inputs. The phone used for the investigation still runs its original camera package and recording profiles.

## Build integration

1. Apply `0001-redwood-add-1440p60-system-profile.patch` in the redwood device tree. The modified XML is copied to `/vendor/etc/media_profiles_yupik_v1.xml`. It adds one profile for camera 0, using H.264 at 32 Mbit/s and the same AAC settings as stock 1080p. All other profiles were compared with the original.
2. Add `0011-redwood-add-1440p60-to-Xiaomi-Camera.patch` after the existing patches under `vendor/xiaomi/redwood-miuicamera/patches/`. Regenerate the patched prebuilt through the repository's APK extraction and split step, or apply the patch to the decoded matching prebuilt and regenerate its split APK inputs. An ordinary build that only concatenates the existing parts will use the old app. The device tree and camera extraction code are available from [crDroid's redwood sources](https://github.com/crdroidandroid/android_device_xiaomi_redwood/blob/1e2c4c54580abf77d188bdc1ab341230a1faca2e/device.mk) and [the camera repository](https://gitlab.com/crdroidandroid/proprietary_vendor_xiaomi_miuicamera/-/blob/1dd6cc28df8dd5c4368cb4b5644f3f808e98921c/extract-files.py).
3. Rebuild the camera and package both changes in the ROM. The installed camera carries crDroid's platform certificate, SHA-256 `73fdfbf41fe6255a3fbc3cfda8b6666f4124f940f499bdd82f1566da68e45732`. An in-place signed update to this app requires a compatible signing identity. The camera module uses the build's platform key. [Camera build definition](https://gitlab.com/crdroidandroid/proprietary_vendor_xiaomi_miuicamera/-/blob/1dd6cc28df8dd5c4368cb4b5644f3f808e98921c/vendor/Android.bp)

The menu patch adds the value `11,60`. Xiaomi Camera parses its first number as Android's `QUALITY_QHD` and its second as the recording frame rate. The patch checks the camera ID, normal rear Video mode, the QHD system profile, the vendor's 2560×1440 HFR entry, and the availability of 1080p60 and 4K30 in the current configuration. The entry uses the existing 60 fps icon and the label **1440p 60 fps**.

## Verification

| Check | Result |
| --- | --- |
| Eight-second rear-camera test, one-second warm-up excluded | The regular Camera2 recording session and Qualcomm H.264 encoder produced 2560×1440 output. Sensor timestamps measured 60.069 fps across 402 frames. Encoded timestamps measured 60.069 fps across 399 frames. The encoded median frame interval was 16.647 ms and the longest interval was 18.315 ms. The probe discarded the video and did not request microphone access. |
| Profile parsing on the handset | In a separate process, the ROM's own `libmedia.so` read both the stock XML and a temporary copy of the patched XML. The stock file had no QHD profile. The patched file returned 2560×1440, 60 fps, 32,000,000 bit/s and stereo 48 kHz for camera 0. Camera 1 continued to report no QHD profile. |
| Patch application and code generation | Both diffs applied to clean inputs. The device-tree XML matched the installed file by SHA-256. Apktool 3.0.3 assembled the patched Xiaomi Camera code. Reinspection of the rebuilt DEX verified the new menu entry, capability checks and selection value. |
| Installed Xiaomi Camera and a saved clip | Pending a compatible signed build. The short capture test used the actual camera and encoder with stabilization off. Stock-app preview, audio/video sync, stabilization and sustained recording still need testing together. |

For the first signed-build test, open rear Video at 1×, select **1440p 60 fps**, record a moving subject, and inspect the saved MP4 dimensions and frame timestamps. Also check 1080p60 and 4K30. Any eventual deployment should preserve the phone's current vendor customizations and use a rollback for the exact active system and vendor partitions.

The original `MiuiCamera.apk` SHA-256 was `5e8d4de460be66ffd2bd24f40dbbd58ed1a30dd8394ff8259f11aef8b392a601`. The original profile XML SHA-256 was `8abef69fff85e655c5d19edd3333cf6556e2e0f1666c75e6577cd8c73017c1d9`. The menu analysis used REA evidence `ev_29554ffd880939c11f8aaa1d8c4d57bc5b32fe4dfafa0776bcb290ee72bdf060` and rebuilt-method evidence `ev_35c843c9aa029f63d69860f443d7d85a2c3a037eb3fd0ee156bccd0837790125`.
