# libsodium-dart

Fix for allowing https://github.com/firstfloorsoftware/flutter_sodium to run in pure-dart non-flutter applications.

[![Pub](https://img.shields.io/pub/v/libsodium.svg)](https://pub.dartlang.org/packages/libsodium)

## Getting Started

In your dart project add the dependency:

```yml
dependencies:
  ...
  libsodium: ^0.1.6
```

Import the plugin and initialize it. Sodium.init() initializes the plugin and should be called before any other function provided by flutter_sodium.

```dart
import 'package:libsodium/libsodium.dart';

// initialize sodium
Sodium.init();
```

## Usage example

```dart
// Password hashing (using Argon)
final password = 'my password';
final str = PasswordHash.hashStringStorage(password);

print(str);

// verify hash str
final valid = PasswordHash.verifyStorage(str, password);

assert(valid);
```

This project includes an extensive example app with runnable code samples. Be sure to check it out!

## API coverage
The libsodium plugin implements the following libsodium APIs:
- crypto_aead
- crypto_auth
- crypto_box
- crypto_generichash
- crypto_hash
- crypto_kdf
- crypto_kx
- crypto_onetimeauth
- crypto_pwhash
- crypto_scalarmult
- crypto_secretbox
- crypto_secretstream
- crypto_shorthash
- crypto_sign
- crypto_stream
- randombytes
- sodium_version

API coverage is not 100% complete, track the progress in issue #35.

## Dart APIs
The plugin includes a core API that maps native libsodium functions 1:1 to Dart equivalents. The core API is available in the class [`Sodium`](https://github.com/firstfloorsoftware/flutter_sodium/blob/master/lib/flutter_sodium.dart). Dart naming conventions are used for core API function names. A native libsodium function such as `crypto_pwhash_str`, is available in flutter as `Sodium.cryptoPwhashStr`.

Also included in flutter_sodium is a high-level, opinionated API providing access to libsodium in a Dart friendly manner. The various functions are available in separate Dart classes. Password hashing for example is available in the `PasswordHash` class. The high-level API depends on the core API to get things done.

## Migrating to fluttter_sodium FFI
The FFI implementation of flutter_sodium is backwards incompatible with the previous platform channel implementation. The list of changes:
- the entire FFI API is now synchronous, while the previous implementation was entirely asynchronous
- all hardcoded libsodium constants are now available as properties on the Sodium class.
- in the platform channel versions the Android and iOS implementations were not in sync. Some functions were available only in iOS, others only in Android. With the FFI implementation, there is a single API covering both platforms.

## Background threads
Since the entire FFI API is synchronous, you'll need to do some extra work to execute long running crypto function on a background thread. Luckily this is very easy with Flutter's [compute function](https://api.flutter.dev/flutter/foundation/compute.html).

The following code snippet demonstrates running a password hash on the background thread.

```dart
final pw = 'hello world';
final str = await compute(PasswordHash.hashStringStorageModerate, pw);

print(str);
```

## Known issues
- Previous incarnations of flutter_sodium used platform channels for native interop. The latest version has been rewritten to take full advantage of Dart FFI. FFI offers fast native interop and is the obvious choice for flutter_sodium. One minor problem, FFI is still in beta and its API may change. This may affect flutter_sodium.
