> 目录已整理：文档在「项目文档」，构建、缓存与暂存输入在「Build」。从仓库根目录运行 `python3 构建.py --build`；如需使用本文原有源码命令，先运行 `python3 构建.py --stage --ci`，再进入 `Build/源码`。暂存会恢复原输入路径。现有版本和历史验证记录按各自提交理解。

# ChromeRelay

**ChromeRelay 1.0.0** is a C++20 MCP-to-Chrome DevTools bridge. It connects to an
existing loopback Chrome and provides 54 canonical browser actions or 75 legacy
names. Native transport, sessions, navigation, inputs, files,
workflows and cancellation run without a Node or Playwright runtime. Small
embedded JavaScript adapters handle DOM access and browser-side value conversion.

Start with [GETTING_STARTED.md](<docs/GETTING_STARTED.md>) for installation, a
separate Chrome profile, MCP host configuration and the C++ library example.
For a concrete first task, [fill a local form, click once and read its result](<docs/LOCAL_WORKFLOW.md>).
The example uses the included page and shows the expected MCP result.

| Task | Canonical tools |
|---|---|
| Open and inspect a page | `tab_create`, `page_navigate`, `page_read`, `page_snapshot` |
| Fill and check a form | `element_fill`, `element_click`, `element_read`, `page_assert` |
| Work within a frame | `frame_list`, `frame_enter`, `frame_leave` |
| Capture or compose actions | `page_capture`, `workflow_batch`, `workflow_steps` |

The current local Release review passed **1,556 checks**, plus 28 installed CLI
checks, on macOS arm64. Debug and ASan/UBSan each passed the 289 affected checks,
not the full matrix. Those runs attach to real Chrome with temporary profiles and
owned pages. They do not cover arbitrary websites or an existing personal profile.
The record is [the protocol review](<validation/protocol-review-2026-09-25.json>).

An earlier hosted Release run,
[36091983364](https://github.com/dhtfish-98/ChromeRelay/actions/runs/36091983364),
passed 1,533 checks at commit `9f585b4c357fd2aa97cd450a4478c3cefdbdb866` on
Chrome 152.0.7977.83. [Its summary](<validation/hosted-run-2026-09-25.json>)
separates installed CLI checks and the catalog-only library consumer. That commit
predates the port-precedence and request-shape fixes. An intermittent
transformed-frame hover timeout remains open; neither run establishes its cause.

```sh
cmake --preset release
cmake --build --preset release
ctest --preset release --verbose
cmake --install build/release --prefix "$PWD/dist/ChromeRelay-macos-arm64"
dist/ChromeRelay-macos-arm64/bin/chrome-relay --port 9222
```

Build dependencies: a C++20 compiler **and a standard library/SDK implementing
`std::stop_source` and `std::stop_token`**, CMake, Ninja, Boost headers and
nlohmann/json. CMake compiles and links a cancellation probe before building;
accepting `-std=c++20` alone is insufficient. If the probe fails, upgrade the
compiler together with its standard library/SDK and configure a fresh build
directory. On macOS, select a newer Xcode or compatible LLVM installation;
changing the compiler executable alone may still leave an older SDK in use.
Local validation used Xcode 27.0; the macOS CI configuration selects Xcode 26.6.
These identify the local and CI toolchains, not a claimed minimum Xcode version.
Validated dependency versions are in `dependencies.lock.json`. The macOS executable uses
system dynamic libraries. Build and install from this source tree. A [validation summary](<validation/README.md>)
records the locally accepted version and its verification boundaries.

Default names describe their domain: `page_navigate`, `element_click`,
`element_type`, `frame_enter`, `page_capture`, `workflow_steps`. Use `--catalog`
to inspect schemas or `--compat-tools` to advertise the original 75 names. The
server exits by disconnecting; it does not close your browser. Repeated
`--allow-root DIR` flags replace default home/temp upload and screenshot roots.

- [COMPATIBILITY.md](<docs/COMPATIBILITY.md>): all names, legacy result formats,
  preserved contracts and intentional differences.
- [CLICKS.md](<docs/CLICKS.md>), [KEYBOARD.md](<docs/KEYBOARD.md>),
  [FRAMES.md](<docs/FRAMES.md>): native input, targeting and frame scope.
- [NAVIGATION.md](<docs/NAVIGATION.md>), [PAGE_SERVICES.md](<docs/PAGE_SERVICES.md>):
  navigation, recovery, dialogs, cookies, storage and accessibility.
- [FILES_AND_CAPTURE.md](<docs/FILES_AND_CAPTURE.md>),
  [WORKFLOWS.md](<docs/WORKFLOWS.md>), [CANCELLATION.md](<docs/CANCELLATION.md>):
  file limits, composition, deadlines and side effects.
- [CHECKPOINT.md](<docs/CHECKPOINT.md>): measured acceptance and exact boundaries.

The 1,556-check result above is the current local review. It fixes port-value
precedence and validates MCP request shapes without conflating protocol errors
with tool execution errors. The 1,533-check figure is the earlier full matrix,
including the hosted run, and is not the current total.

The earlier 2026-09-25 follow-up passed **1,533 checks** in each of Debug, Release and
ASan/UBSan: 155 contracts, 1,146 actual Chrome/MCP checks and 232 socket/keyboard
fault checks. Forty-six new checks cover interrupted typing and remote element
ownership, default/private cookie-context isolation, and asynchronous file
selection. Closing a page during an action stops remaining typing; directory
uploads wait for their own completion event without repeating the selection.
Debug and ASan/UBSan used Chrome 153; Release used Chrome 152. The installed
CLI passed 17 checks; an independently built installed-library consumer passed
seven browser checks. All browser tests used owned temporary Chrome profiles
and local fixtures, with cleanup receipts. The Release MCP suite used the
installed executable.

Historical 2026-09-23 validation includes **549 selected ThreadSanitizer checks**
and a finite protocol fuzz run of **67,587 executions in 31 seconds** without a
crash or sanitizer report. Those two runs were not repeated for the follow-up;
see [validation](<validation/README.md>) for the distinction and recorded scope.

Validated platform: macOS arm64, Chrome 152 and 153. Linux/Windows have not been run;
Windows needs adaptation of POSIX file/stdin handling. Arbitrary page layouts,
extended Playwright selectors and every possible parameter combination are not
claimed equivalent. Drag is within one selected document; snapshots explicitly
use native `ax-yaml`. See the linked guides before migrating a caller.

ChromeRelay connects to a Chrome debug port and can run page script and read or
change cookies and storage. The checks recorded here used temporary profiles on
macOS. They do not cover arbitrary pages.

Copyright 2026 dhtfish98, MIT. Embedded Playwright keyboard mapping data keeps its
Apache-2.0 license. It is not newly authored mapping data or a runtime dependency.

Local validation and publication scope: [validation/README.md](<validation/README.md>).
