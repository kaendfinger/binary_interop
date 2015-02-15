binary_interop
=====

Binary interop is a library that allows load shared libraries, invoke their functions and get access to their data.

Version: 0.0.19

[Donate to binary interop for dart](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=binary.dart@gmail.com&item_name=binary.interop.for.dart&currency_code=USD)

**Interrelated (binary) software**

- [Binary declarations](https://pub.dartlang.org/packages/binary_declarations)
- [Binary generator](https://pub.dartlang.org/packages/binary_generator)
- [Binary interop](https://pub.dartlang.org/packages/binary_interop)
- [Binary types](https://pub.dartlang.org/packages/binary_types)

**Limitations**

- Binary type "long double" not supported
- Returning the packed structures not supported
- Passing by the value a packed structures not supported

**Supportedd platforms**

- X86 Linux
- X86 Windows
- X86_64 Linux
- X86_64 Windows

Mac OS X will be added soon.

Binary interop is a low-level way of interacting with dynamic loadable libraries.  
It support interaction only through the low level binary types and binary objects.

**Examples**

```dart
import "dart:io";

import "package:binary_interop/binary_interop.dart";
import "package:unittest/unittest.dart";

import "libc.dart";

final BinaryTypes _t = new BinaryTypes();

void main() {
  var libc = loadLibc();
  var helper = new BinaryTypeHelper(_t);

  // Strlen
  var string = "0123456789";
  var length = libc.strlen(string);
  expect(length, string.length, reason: "Call 'strlen'");

  string = "Hello Dartisans 2015\n";

  // printf
  length = libc.printf("Hello %s %i\n", ["Dartisans", 2015]);
  expect(length, string.length, reason: "Wrong length");

  // sprintf
  var buffer = alloc(_t["char[500]"]);
  length = libc.snprintf(buffer, 500, "Hello %s %i\n", ["Dartisans", 2015]);
  var string2 = helper.readString(buffer);
  expect(length, string.length, reason: "Wrong length");
  expect(string, string2, reason: "Wrong string");
}

Libc loadLibc() {
  String libname;
  var operatingSystem = Platform.operatingSystem;
  switch (operatingSystem) {
    case "macos":
      libname = "libSystem.dylib";
      break;
    case "android":
    case "linux":
      libname = "libc.so.6";
      break;
    case "windows":
      libname = "msvcr100.dll";
      break;
    default:
      throw new UnsupportedError("Unsupported operating system: $operatingSystem");
  }

  var library = DynamicLibrary.load(libname, types: _t);
  if (library == null) {
    throw new StateError("Failed to load library: $libname");
  }

  return new Libc(library);
}

BinaryObject alloc(BinaryType type, [value]) => type.alloc(value);

```

Library wrapper for `libc`.

```dart
// This code was generated by a tool.
// Processing tool available at https://github.com/mezoni/binary_generator

import "package:binary_interop/binary_interop.dart";

class Libc {
  String _header = '''
int printf(const char *format, ...);
#if OS == windows
int snprintf(char *s, size_t n, const char *format, ...) __attribute__((alias("_sprintf_p")));
#else
int snprintf(char *s, size_t n, const char *format, ...);
#endif
size_t strlen(const char *s);''';

  DynamicLibrary _library;

  /**
   *
   */
  Libc(DynamicLibrary library) {
    if (library == null) {
      throw new ArgumentError.notNull("library");
    }

    library.declare(_header);
    _library = library;
  }

  /**
   * int printf(const char* format, ...)
   */
  dynamic printf(format, [List params]) {
    var arguments = [format];
    if (params != null) {
      arguments.addAll(params);
    }

    return _library.invoke("printf", arguments);
  }

  /**
   * int snprintf(char* s, size_t n, const char* format, ...)
   */
  dynamic snprintf(s, int n, format, [List params]) {
    var arguments = [s, n, format];
    if (params != null) {
      arguments.addAll(params);
    }

    return _library.invoke("snprintf", arguments);
  }

  /**
   * size_t strlen(const char* s)
   */
  dynamic strlen(s) {
    return _library.invoke("strlen", [s]);
  }

}


```
