# libE57Format (MeshInspector fork)

This is the [MeshInspector](https://github.com/MeshInspector) fork of [asmaloney/libE57Format](https://github.com/asmaloney/libE57Format), a C++ library that reads and writes [E57](https://www.astm.org/e2807-11r19e01.html) point cloud files. [MeshLib](https://github.com/MeshInspector/MeshLib) uses it for E57 import on all its platforms.

The public API, file-format support and behavior are the same as upstream's. For documentation, the API reference, the changelog and contribution rules, see the [upstream repository](https://github.com/asmaloney/libE57Format) and its [docs](https://asmaloney.github.io/libE57Format-docs/). This README covers only what differs here.

## Base version

`master` = upstream **v3.4.1** + the changes below. To update, rebase those changes onto the next upstream tag.

MeshLib pins exact commits of this repository, in its `thirdparty/libE57Format` submodule, its CPM lock and its vcpkg overlay port. Never force-push `master` or delete a branch whose commits MeshLib still pins.

## Selectable XML parser

Upstream always parses the XML section of an E57 file with [Xerces-C++](https://xerces.apache.org/xerces-c/). Here the CMake option `E57_XML_PARSER` selects the parser, case-insensitively:

| `E57_XML_PARSER` | Dependency | Notes |
|---|---|---|
| `Xerces` (default) | Xerces-C++ >= 3.2 | Same as upstream, including the hardening against external entities and entity expansion. |
| `TinyXML2` | [TinyXML2](https://github.com/leethomason/tinyxml2) | Used by MeshLib on every platform. Reads the whole XML section into memory and then walks it; the XML section holds only metadata, while point data stays binary and is read in streams as before. |
| `Expat` | [Expat](https://libexpat.github.io/) | Streaming SAX parser. |
| `Custom` | none | No parser is built in. Code compiled in-tree must call `E57XmlParser::setCustomImpl()` (in `src/E57XmlParser.h`, not installed) before any file is opened; otherwise opening a file throws `ErrorXMLParserInit`. |

```sh
cmake -B E57-build -DCMAKE_BUILD_TYPE=Release -DE57_XML_PARSER=TinyXML2 libE57Format
cmake --build E57-build --parallel
```

The parser-independent part (`E57XmlProcessor`, which turns XML events into E57 nodes) stays in `src/E57XmlParser.cpp`. Each parser is a small adapter behind the `E57XmlParserImpl` interface in `src/E57XmlParserImpl<Name>.cpp`, and only the selected one is compiled.

The installed `e57format-config.cmake` is generated from `cmake/e57format-config.cmake.in`. It looks up the dependency of whichever parser the library was built with, so consumers keep using `find_package(E57Format CONFIG)` unchanged. For TinyXML2 it falls back to pkg-config when no CMake package is found.

### Why

Xerces is a large dependency (about 2.7 MB as a Windows DLL) and is awkward to build for WebAssembly. MeshLib already uses TinyXML2, so with this option E57 support adds no XML library of its own.

## Testing

Upstream's test suite (`-DE57_BUILD_TEST=ON` with [libE57Format-test-data](https://github.com/asmaloney/libE57Format-test-data)) passes unchanged with the `Xerces`, `TinyXML2` and `Expat` parsers.

## License

Same as upstream: [BSL-1.0](LICENSE.md).
