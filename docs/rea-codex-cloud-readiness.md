# REA readiness in this Codex cloud session

Checked on 2026-10-13. These findings describe this session's Linux container;
they are not a guarantee for every Codex Cloud task or a persistent installation.

## Skill source and preservation

- Requested source: https://github.com/odogwu-7g/rea
- Destination selected by the user: `odogwu-7g/rea`.
- Source commit: `d13c9f840d36a321a56917e75c48afcb8d72920e`.
- Source package version: `6.4.0`.
- Skill path: `.agents/skills/reverse-engineer-anything/`.

The destination is the source repository itself. The complete skill already
exists at the requested path: `SKILL.md` and six reference files. Copying these
bytes to the same path would make no change. All existing skills are preserved;
this PR adds only this provenance and readiness report. No skill files,
personal agent configuration, providers, or project dependencies are changed.

The following Git blob IDs record the exact source files at that commit.
They are Git object IDs, not raw-file SHA-256 checksums.

| File relative to the skill folder     | Git blob ID                              |
| ------------------------------------- | ---------------------------------------- |
| SKILL.md                              | de9f6b7c12ac065361ee6429b937046146a83ed3 |
| references/android-applications.md    | 896f15f85efaa03503eacfd598bed7db6bf791ea |
| references/connection-and-recovery.md | ed8393bd9a020f7a31adfa050e47b3d673db7580 |
| references/evidence-workflows.md      | c651287f3e8f7aa5b368c2ec9356b0c6f4a8807e |
| references/javascript-applications.md | 29b4718fefdc0abcba74910199be83b8f7242393 |
| references/native-and-artifacts.md    | 71fe3e95081d218b89be987fd32ddb07b93afa61 |
| references/runtime-observation.md     | 2d660d675d8adb7b11777d39aa7ce39b6eb533cb |

## Separate availability results

| Component                      | Observed result                                                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Project skill                  | Present and complete in the repository. Not newly loaded into this chat's advertised skill catalog.                                                                                  |
| CLI                            | Built successfully from the recorded commit; version and PE resource inspection passed. Published npm 6.4.0 help also passed separately. No global CLI installation was made.        |
| MCP server                     | Local stdio server started; initialization, 191 tool schemas, six prompts, server identity, and binary_session passed. PE resource inspection also passed through a real MCP client. |
| MCP tools exposed to this chat | No REA connector tools are registered in the chat's tool catalog. A shell-launched test server is distinct from an agent integration.                                                |

The audit-wide `doctor` report is unhealthy because native engines, the
personal installed-skill identity, and the Codex REA registration are absent.
This does not negate the repository skill, working CLI commands, or successful
local MCP protocol tests. No `rea setup` configuration writes were performed.

## Runtime and build

- Host: Linux, Ubuntu 24.04.
- Node.js: `v24.19.0`; accepted by package engines
  `^22.19.0 || ^24.11.0 || >=26.0.0`.
- npm: `11.9.0`.
- Repository development pins: Node `24.18.0` and npm `11.16.0`.
  The available toolchain is runtime-supported but differs from these pins.
- `npm ci --ignore-scripts --no-audit --no-fund --fetch-retries=0 --fetch-timeout=20000`
  passed, installing 408 packages.
- `TURBO_TELEMETRY_DISABLED=1 DO_NOT_TRACK=1 npm run build:cached`
  passed: three successful tasks.
- An earlier build triggered Turborepo telemetry and automatic approval review
  rejected continuing that invocation. The successful retry disabled telemetry.
- Build outputs, test samples, dependencies and raw diagnostic results are
  temporary and are excluded from this PR.

## Functional checks

The repository documentation gate also passed with telemetry disabled:
`npm run docs:check` completed six build/check tasks and both README consistency
tests. The report was formatted with the repository's oxfmt, and all seven
recorded skill blob IDs were verified against the working files. Existing skill
files have no diff against the source commit.

Commands below were run from the source checkout unless noted.

| Check                                                                                                 | Result                                                                     |
| ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| node scripts/rea.mjs --version                                                                        | Exit 0, 6.4.0                                                              |
| npm exec --yes --package=rea-agents@6.4.0 --fetch-retries=0 --fetch-timeout=15000 -- rea --help       | Exit 0; tests published release help, not equivalence to the source commit |
| node scripts/rea.mjs doctor --json                                                                    | healthy=false; missing registrations and native engines                    |
| node scripts/rea.mjs mcp doctor --json                                                                | healthy=true; 191/191 tools, 6/6 prompts; binary_session request passed    |
| node scripts/rea.mjs inspect-pe-resources /tmp/rea-smoke.exe --json                                   | Exit 0; five resource payloads from the repository's inert PE fixture      |
| MCP inspect_pe_resources on the same sample                                                           | Success, no MCP error                                                      |
| node scripts/rea.mjs analyze-javascript-application tests/conformance/readiness/javascript-cli --json | Exit 1, capability_unavailable; parsing worker could not launch            |
| node scripts/rea.mjs inspect-sqlite-database /tmp/rea-smoke.sqlite --table smoke --json               | Exit 1, capability_unavailable; database worker could not launch           |
| MCP inspect_sqlite_database on the same sample                                                        | Same process-ownership failure                                             |
| ps -axo pid=,ppid=,pgid=,uid=,stat=,command=                                                          | Fails with fatal library error, lookup self                                |

The PE sample was generated from `src/native/pe/PeResources.fixture.ts`; it
contains resources rather than a running application. The SQLite sample was
created locally with one ordinary table and one row. Neither sample executes
an untrusted target.

Successful CLI PE Evidence:
`ev_928910a11152d2cb4e59443a1e8145865f3ebf3abefee52b463073703b49892a`.
Sample SHA-256:
`7b6c790f8a865899f70e16fc12780d47a9ef630c5f5d62121cd49a5d995cc7f3`.

The JavaScript failure includes partial Evidence
`ev_b75e7849d94d6a784726263944de5b5c3b44220b74b5d37703d55177587ad6de`.
Partial inventory does not establish complete parsing or reconstruction.

## Tool support boundaries

REA's catalog advertises operations before all execution-time prerequisites
are known. A listed tool is not evidence that its full workflow succeeds.

| Tool family                                                                      | Status in this session                                                                                                                     |
| -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Session/catalog and PE resources                                                 | Verified through real local MCP; PE also verified through CLI                                                                              |
| JavaScript application and SQLite inspection                                     | Tested and blocked by Linux process ownership inspection; supported Node alone is insufficient                                             |
| Go metadata, managed .NET static inspection, retained Evidence views/comparisons | Advertised by binary_session; no representative functional test performed                                                                  |
| Native disassembly/decompilation                                                 | Unavailable: Hopper, Ghidra and IDA not installed/configured                                                                               |
| Android JADX/ADB/Apktool workflows                                               | Missing providers; no device or emulator workflow tested                                                                                   |
| Offline ELF/core inspection, firmware, WASM, signal decoding                     | Missing caller-supplied providers/configuration                                                                                            |
| Packet analysis                                                                  | TShark and capinfos absent from PATH; catalog advertisement alone does not verify execution                                                |
| Browser/Electron/Frida/process capture                                           | Some tools advertised, but no target/browser/runtime attachment tested; process-launch workflows face the observed ownership-check failure |
| macOS native UI and assetutil workflows                                          | Unsupported host: this container is Linux                                                                                                  |

`java` (OpenJDK 17) and `prlimit` are present, but these do not supply the
missing providers. No native engine, browser, hardware integration, or remote
debugging endpoint was installed or assumed.

## Rechecking in a future project environment

Build from the recorded commit with a supported runtime and telemetry disabled.
Then run `rea mcp doctor --json` and a representative command for the intended
target. In particular, confirm the exact `ps` command above succeeds before
expecting worker-based JavaScript or SQLite inspection to work.

A project skill supplies instructions. CLI installation and MCP registration
are separate steps. This PR neither registers a server with Codex nor asserts
that a repository file can dynamically add tools to an already-running chat.
