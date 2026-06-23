# BLEUnlock Security Model

## What BLEUnlock Does

BLEUnlock locks, wakes, and optionally unlocks your Mac based on proximity to a Bluetooth Low Energy (BLE) device.

## Auto-Unlock: Important Limitations

When auto-unlock is enabled, BLEUnlock types your macOS login password into the lock screen using synthetic keyboard events (CGEvent). This approach has significant security implications:

### What Auto-Unlock Is NOT

- **Not equivalent to Apple Watch Auto Unlock.** Apple's implementation uses secure hardware attestation and encrypted key exchange. BLEUnlock sends your password as simulated keystrokes.
- **Not resistant to BLE spoofing.** Anyone with a BLE scanner can detect your device's UUID and broadcast the same identifier to impersonate your device.
- **Not resistant to keyboard event interception.** Other apps with Accessibility permissions could observe the synthetic keystrokes.

### Default Behavior

As of v1.14, auto-unlock is **disabled by default**. BLEUnlock ships in lock-and-wake-only mode. You must explicitly opt in to auto-unlock through the menu, and you will see a security warning before enabling it.

## Password Storage

If you enable auto-unlock, your macOS login password is stored in the macOS Keychain with the following attributes:

- **Access control:** `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` — the password is only accessible when the device is unlocked, and cannot be transferred to other devices via Keychain sync.
- **Service:** Tied to the app's bundle identifier.
- **Account:** Tied to the current macOS user account.

The password is never logged, displayed in the UI, included in crash reports, or transmitted over the network.

## Accessibility Permission

BLEUnlock requires Accessibility permission to:

1. Post synthetic keyboard events (for auto-unlock).
2. Detect screen lock/unlock state (via `CGSessionCopyCurrentDictionary`).
3. Send the Escape key to dismiss the screensaver (to show the login panel).

Without Accessibility permission, BLEUnlock cannot auto-unlock. Lock and wake functionality still works without it.

## BLE Device Identity

- BLEUnlock identifies devices by their CoreBluetooth UUID, which is assigned by macOS and is stable for paired devices.
- Some devices (especially iPhones) may use rotating BLE addresses. BLEUnlock reads `/Library/Preferences/com.apple.Bluetooth.plist` and `/Library/Bluetooth/*.db` as a best-effort fallback to resolve device names and MAC addresses. These files require admin access and may not be available on all macOS versions.
- **BLE proximity is not a secure authentication factor.** RSSI values can be affected by interference, reflections, and antenna orientation. A device may appear "close" or "far" due to environmental conditions.

## Recommendations

1. **Use lock-and-wake mode** (default) unless you specifically need auto-unlock.
2. If you enable auto-unlock, use a strong, unique password for your macOS account.
3. Be aware that auto-unlock reduces the physical security of your Mac — anyone who can bring a spoofed BLE device within range can unlock it.
4. Consider using "Wake without Unlocking" mode as a compromise: BLEUnlock wakes the screen but you still enter your password manually.

## Reporting Security Issues

Please report security vulnerabilities via GitHub Issues with the `security` label, or contact the maintainer directly.
