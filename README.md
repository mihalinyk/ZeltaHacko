# Zelta - Hacko

**Zelta - Hacko** is a lightweight Android security library designed to protect your applications from tampering, reverse engineering, and running on rooted devices.

---

## Features

- **Signature Verification (SHA-256)**
- **Tamper Detection** (APK re-signing)
- **Root Detection** (Native-level)
- **Custom Payload Execution** (upon tampering detection)
- **Fallback Activity Launch** (for recovery or lockout)

---

## 🛠️ Installation

1. **Add as Git Submodule**

If you're using Git, it's recommended to add this library as a submodule from your project root directory:

```bash
$ git submodule add https://github.com/mihalinyk/ZeltaHacko ZeltaHacko
$ git submodule update --init --recursive
```

2. **Add the library to your project**

You must add this (in your `settings.gradle` or `settings.gradle.kts`):

```gradle
//...

include(":ZeltaHacko")
```

In your main application in your `build.gradle` or `build.gradle.kts`, you must add:

```kotlin
implementation project(":ZeltaHacko") // if modularized
// OR
implementation(project(":ZeltaHacko"))
```

---

## 📦 Usage

### 1. **Initialization**

```kotlin
val zelta = ZeltaHackoMimi(context)
zelta.init(FallbackActivity::class.java) // Optional: for level 1 handling
```

### 2. **Launch Protection**

```kotlin
zelta.launch(level = 1) {
    if (it) {
        Log.i("Zelta", "Signature verified.")
    } else {
        Log.w("Zelta", "Tampering detected!")
    }
}
```

---

## Security Levels

| Level | Behavior |
|-------|----------|
| `1`   | Launch fallback Activity if signature is invalid |
| `2`   | Execute native C++ payload (e.g., crash, block, exit app) |

### ✔ Root Detection
The `isDeviceRooted()` method uses native code to detect common signs of rooted devices.

---

## 🔧 API Reference

### `ZeltaHackoMimi(context: Context)`
Constructor to initialize the protection engine.

### `fun init(activity: Class<*>)`
Set the fallback activity to be launched on detection (for level 1).

### `fun launch(level: Int = 1, callback: (Boolean) -> Unit)`
Launch the protection check. Calls back `true` if tampered.

### `external fun isDeviceRooted(): Boolean`
Returns `true` if the device is rooted.

---

## Native Integration

You must implement the following **native functions** in C++ (`zeltamimi.cpp`):

```cpp
extern "C"
JNIEXPORT jstring JNICALL
Java_id_seraphyne_zeltamimi_ZeltaHackoMimi_sin(JNIEnv* env, jobject /* this */) {
    std::string sin = "YOUR_SHA256";
    return env->NewStringUTF(sin.c_str());
}
```

## Example

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val zelta = ZeltaHackoMimi(applicationContext)
        zelta.init(FallbackActivity::class.java)
        zelta.launch(level = 1) {
            if (it) {
                setContentView(R.layout.activity_main)
            }
        }
    }
}
```

---

### 3. Remove the Submodule (if needed)

To remove the submodule completely, run this one-liner:

```bash
$ git submodule deinit -f ZeltaHacko
$ git rm -f libs/ZeltaHacko
$ rm -rf .git/modules/ZeltaHacko
$ rm -rf ZeltaHacko
```

---

## 🧪 Testing

- Sign your app with the correct release keystore
- Run on an emulator/device
- Try modifying the APK or re-signing it → it should trigger detection

## 🧙‍♂️ Tips

- Combine this with ProGuard/R8 and native obfuscation.
- Store secrets and hash logic in C++ to make reverse engineering harder.
- For critical apps (banking, fintech), use multiple layers (debug checks, hook detection, etc.)

## 🤝 Contributing

Pull requests and ideas are welcome! Feel free to fork and improve the system.

---

## License

This project is licensed under the MIT License — see the [LICENSE](./LICENSE) file for details.
