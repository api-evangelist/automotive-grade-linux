# LAVA API Skill

Reference for any LLM/agent working against a LAVA instance (e.g.
`lava.automotivelinux.org`). Do not embed credentials in prompts, code, or
committed files — this toolkit gives you three ways to reach LAVA, all of
which keep the token out of the model's context:

| Method | Where the token lives | Best for |
|---|---|---|
| **MCP tool** (this repo's `go/`, `python/`, `rust/`, `cpp/`) | Server-side env var or `users.json`, never in the conversation | An agent driving LAVA autonomously across a session |
| **`lavacli`** (third-party CLI) | `~/.config/lavacli.yaml` | A human or script preferring a polished CLI, XML-RPC under the hood |
| **Direct `curl`** (`scripts/`) | Env var or `~/.config/lava/token` file | One-off terminal use, CI steps, or when nothing else is installed |

All three talk to the same LAVA instance and (mostly) the same REST API —
pick based on what's available in your environment, not because one is more
"correct." See `docs/deployment.md` for a fuller decision table on which MCP
language/variant to use.

## Instance
- Base: `https://lava.automotivelinux.org`
- REST API root: `/api/v0.2/` (DjangoRestFramework, browsable at that URL in a real browser)
- XML-RPC root: `/RPC2` (legacy but still the only way to reach a few admin/device-mapping calls, and what `lavacli` uses)
- Help/introspection: `/api/help/` (REST), XML-RPC method list via `system.listMethods()`

## Auth
- Token-based. Same token works for REST and XML-RPC.
- Obtain: create one in the web UI under API → Authentication Tokens (preferred
  over `POST /api/v0.2/token/` with a password from automation).
- REST: header `Authorization: Token <token>` on every call.
- XML-RPC / lavacli: `https://<user>:<token>@lava.automotivelinux.org/RPC2`
- No OAuth, no refresh flow. Tokens are static until revoked in the UI.

### Where the token lives, per method
- **MCP stdio** (`<lang>/stdio`): `LAVA_TOKEN` environment variable, set by
  whatever launches the process (MCP client config, systemd, CI runner).
- **MCP server** (`<lang>/server`): inside `users.json`, one LAVA token per
  mapped identity; callers authenticate with a *different* token
  (`Authorization: Bearer <mcp_token>`) that the server exchanges server-side.
- **curl scripts** (`scripts/`): `$LAVA_TOKEN` env var, or `$LAVA_TOKEN_FILE`,
  or `~/.config/lava/token` (see `scripts/lava-env.sh`).
- **lavacli**: `~/.config/lavacli.yaml` (see `scripts/lavacli.yaml.example`).

Full threat-model / permission notes: `docs/security.md`.

## REST endpoints (v0.2, all under `/api/v0.2/`)
| Endpoint | Methods | Notes |
|---|---|---|
| `jobs/` | GET, POST | list/submit. POST body: `{"definition": "<yaml>"}` |
| `jobs/{id}/` | GET | job detail |
| `jobs/{id}/logs/` | GET | job log (YAML log lines), supports `?start=N` |
| `jobs/{id}/cancel/` | POST | cancel a running/queued job |
| `jobs/{id}/definition/` | GET | original submitted definition |
| `jobs/{id}/resubmit/` | POST | resubmit |
| `devices/` | GET, POST | device list / register |
| `devices/{hostname}/` | GET, PUT | device detail |
| `devicetypes/` | GET | device type list |
| `workers/` | GET | worker list |
| `tags/` | GET, POST | job/device tags |
| `aliases/` | GET | device type aliases |

Standard DRF conventions apply: pagination (`?limit=&offset=` or `?page=`), filtering via query
params, `?format=json` to force JSON over the browsable HTML view, `?ordering=field`.

## XML-RPC-only calls (no REST equivalent)
- `system.api_version()`
- `run_query(query_name, limit=200, username=None)` — requires the caller to own/administer
  the named saved query.
- Switch/network-map admin calls, `revoke_perm_device(perm, device, group)`, and other
  permission-management calls.
- Only reach for XML-RPC (or `lavacli`, which wraps it) when the table above doesn't cover it.

## Common workflows
**Submit a job and poll:**
1. `POST jobs/` with definition → returns `job_ids` (a multinode submission returns several).
2. `GET jobs/{id}/` → poll `state` (`Submitted`/`Scheduling`/`Scheduled`/`Running`/`Canceling`/`Finished`)
   and `health` (`Unknown`/`Complete`/`Incomplete`/`Canceled`).
3. `GET jobs/{id}/logs/?start=N` incrementally once `Running`.

**Find recent failures for a device type:**
- `GET jobs/?device_type=<n>&health=Incomplete&ordering=-submit_time&limit=50`

**Check device availability before submit:**
- `GET devices/?device_type=<n>&health=Good&state=Idle`

## Error conventions
- 401/403 → bad or missing token, or insufficient permission on the object (LAVA has
  per-device/per-devicetype submit permissions, this is *not* just an auth bug).
- 404 on `jobs/{id}/` → wrong id or no view permission on that job.
- 400 on job submit → definition YAML failed schema/device-compat validation; body contains
  the validation error, surface it verbatim to the user, don't guess.

## Job definitions (the `definition` YAML you POST to `jobs/`)

A job definition is one YAML document with these top-level keys:

```yaml
device_type: qemux86-64          # or a specific hostname via `target:`
job_name: agl-smoke-test
timeouts:
  job:
    minutes: 30
  action:
    minutes: 10
  connection:
    minutes: 2
priority: medium                  # low|medium|high, or an integer
visibility: public                # public|personal|group
context: {}                       # device-config overrides, rarely needed
actions:
  - deploy: {...}
  - boot: {...}
  - test: {...}
```

`actions` is an ordered pipeline. Minimum viable job = one `deploy`, one `boot`, one
`test`. Multiple `test` blocks are allowed (e.g. one per test suite); each runs after
the preceding boot unless another `deploy`/`boot` pair resets the device first.

**deploy** (varies by device/method — common QEMU/AGL example):
```yaml
- deploy:
    to: tmpfs
    images:
      rootfs:
        image_arg: -drive format=raw,file={rootfs}
        url: https://example.org/agl-images/agl-demo-platform-qemux86-64.wic.xz
      kernel:
        image_arg: -kernel {kernel}
        url: https://example.org/agl-images/bzImage
```
For real hardware it's typically `to: tftp` or `to: nbd`/`to: sata`/`to: usb`, each with
its own required sub-keys — check the device-type template on the instance
(`GET /devicetypes/{name}/`) rather than guessing.

**boot:**
```yaml
- boot:
    method: qemu
    media: tmpfs
    prompts: ["root@agl:~#"]
    timeout:
      minutes: 5
```
`prompts` must match the real shell prompt string the DUT emits, or LAVA will time out
waiting for boot to complete — this is the single most common job-definition mistake.

**test** — this is where a *test shell definition* is referenced (see next section):
```yaml
- test:
    timeout:
      minutes: 15
    definitions:
      - repository: https://gerrit.automotivelinux.org/gerrit/src/qa-testdefinitions
        from: git
        path: automated/agl-healthcheck/agl-healthcheck.yaml
        name: agl-healthcheck
        params:
          SUITE_ARG: "--verbose"
```

### Creating a job from a template

Rather than writing a job definition from scratch, start from
`templates/job-qemu-agl.yaml` (QEMU/emulated device),
`templates/job-generic-device.yaml` (physical hardware, NBD-root boot), or
`templates/job-generic-device-tftp-nfsroot.yaml` (physical hardware,
NFS-root boot) and fill in the `<PLACEHOLDER>` values. All are validated
YAML you can copy directly; `templates/README.md` explains which deploy
pattern fits which situation, and `templates/examples/` has two verbatim
known-working jobs the templates were genericized from. For physical
hardware specifically, fetch the real device-type template first
(`GET /devicetypes/{name}/`, or `lavacli device-types template get <name>`)
rather than guessing deploy/boot parameters — they vary significantly by
board, including which of NBD-root/NFS-root/TFTP the lab actually supports.

### NBD-root vs. NFS-root vs. QEMU/tmpfs booting

Three deploy shapes show up across the templates, and they're not
interchangeable — which one works depends on the device type and the LAVA
lab's configuration, not on preference:

- **QEMU / emulated (`to: tmpfs`)**: used for `device_type: qemu` and
  similar. Despite not being a hardware NBD-root boot, LAVA's QEMU deploy
  action still uses the `lava-xnbd` protocol internally to coordinate the
  image transfer — you'll see a `protocols: {lava-xnbd: [...]}` block under
  `deploy` even here. Images are referenced under `images:` with
  device-specific keys (`kernel`, `rootvd`, ...), each with its own
  `image_arg` describing how it's wired into the QEMU command line.
- **NBD-root, physical hardware (`to: nbd`)**: the kernel+initrd boot over
  the network as normal, then the actual root filesystem image is served
  to the device over NBD (Network Block Device) and mounted as the real
  root — distinct from copying the whole rootfs onto local storage first.
  Needs a top-level `protocols: {lava-xnbd: {port: auto}}` block *and* a
  matching one under `deploy.protocols`, plus `kernel`/`initrd`/`nbdroot`/
  `dtb` keys (note: `nbdroot`, not `rootfs`) and typically
  `boot.method: minimal` / `boot.commands: nbd`.
- **NFS-root, physical hardware (`to: tftp` + NFS)**: the more traditional
  pattern — kernel over TFTP, rootfs mounted live over NFS on every boot,
  `boot.commands: nfs`, usually `boot.method: u-boot` or similar.

A structural detail worth knowing about because it fails silently rather
than with an error: **YAML mappings can't have two keys with the same
name.** A job definition with two separate top-level `context:` blocks (or
any other duplicated key) will have most parsers — including LAVA's —
silently keep only the *last* one and discard everything in the first, with
no warning. This is a real mistake found while preparing
`templates/examples/agl-qemuarm64-flutter-snapshot.yaml` from a live job
that had exactly this bug (a `context.extra_kernel_args` value was being
silently dropped) — merge everything for one key into a single block rather
than writing it twice.

## Writing a test shell definition (qa-testdefinitions)

A **test shell definition** is a *different* YAML document from the job definition —
it's the actual test recipe, checked into a git repo (for AGL: `qa-testdefinitions`,
hosted at `gerrit.automotivelinux.org/src/qa-testdefinitions` / mirrored on
`git.automotivelinux.org/src/qa-testdefinitions`). The job's `test.definitions[].path`
points at one of these files inside that repo. Start from
`templates/testdef-template.yaml`.

Minimal shape:
```yaml
metadata:
  format: "Lava-Test Test Definition 1.0"   # mandatory, literal string
  name: agl-healthcheck                      # mandatory
  description: "Basic AGL platform healthcheck"  # mandatory
  os:
    - oe                                     # optional but conventional for AGL/Yocto
  scope:
    - functional
  devices:
    - qemux86-64
  environment:
    - lava-test-shell

params:
  SUITE_ARG: "--default"    # optional, becomes a shell env var in `run.steps`

run:
  steps:
    - "cd automated/agl-healthcheck"
    - "./agl-healthcheck.sh $SUITE_ARG"
    - "lava-test-case dbus-check --shell dbus-send --system --print-reply --dest=org.freedesktop.DBus / org.freedesktop.DBus.ListNames"

parse:
  pattern: "^(?P<test_case_id>[\\w-]+)\\s+:\\s+(?P<r>PASS|FAIL)$"
```

Rules that matter:
- `metadata.format`, `metadata.name`, `metadata.description` are **mandatory**. If the
  file isn't in a git/bzr repo, `metadata.version` becomes mandatory too — always true
  for anything actually checked into `qa-testdefinitions`, so not usually an issue.
- Each `run.steps` entry is exactly one shell line — no pipes, no `&&`, no functions, no
  redirects. Put real logic in a separate script committed alongside the YAML in the
  repo, and call that script from a single step (this is the AGL convention: each test
  category under `qa-testdefinitions` has its own subdir with a `.sh` script + a `.yaml`
  wrapper of this shape).
- Use `lava-test-case <case-name> --shell <command>` to record each sub-result as a
  separate pass/fail test case in LAVA's results view, rather than relying only on the
  `parse.pattern` regex against stdout.
- `parse.pattern` is a Python-flavored regex, and Python-string-quoting rules apply in
  YAML, so backslashes must be doubled (`\\s`, `\\w`, ...). It must capture named groups
  `test_case_id` and `result` if you want it to auto-classify plain stdout lines instead
  of / in addition to explicit `lava-test-case` calls.
- Test scripts should be portable POSIX shell (`/bin/sh` semantics) unless the AGL image
  is known to have bash — LAVA test-shell helpers themselves only assume POSIX.
- Inline test definitions (embedding the whole `metadata`/`run`/`parse` block directly
  under `repository:` in the job YAML instead of a `path:`+git repo) are fine for
  one-off/throwaway jobs, but for anything reusable, commit it to `qa-testdefinitions`
  and reference it by `path` — that's what makes it show up for other AGL test writers
  and survive job history.

## Submitting a job — three ways

Pick whichever access method fits your context (see the table at the top of
this document); the underlying LAVA call is the same in all three.

### 1. MCP tool (agent-driven)
1. (optional) `lava_list_devices` with `device_type=<type>&health=Good&state=Idle` to
   confirm capacity before submitting.
2. Assemble the full job YAML (from a template, or from scratch per the sections above)
   as a single string.
3. `lava_submit_job` with that string as `definition`. Response body (echoed via the
   tool's `HTTP <status>` + JSON) contains the new job id(s).
4. `lava_get_job` with the returned id to poll `state`/`health`.
5. `lava_job_logs` (optionally with `start=<line>`) once `state` is `Running`, to stream
   output incrementally rather than re-fetching the whole log each poll.
6. On `state: Finished`, check `health`: `Complete` = ran to completion (still check
   individual `lava-test-case`/`parse` results for pass/fail — `Finished`+`Complete` only
   means the job pipeline didn't crash), `Incomplete` = infra/pipeline failure — pull
   `lava_job_logs` and surface the failing action verbatim rather than guessing.

Equivalent raw call if you need to bypass the dedicated tool (via `lava_raw_request`):
`method=POST path=jobs/ body={"definition": "<the yaml string>"}`.

### 2. lavacli (human/CLI-driven)
```
lavacli -i agl jobs submit job.yaml
lavacli -i agl jobs show <job-id>
lavacli -i agl jobs logs <job-id>
lavacli -i agl jobs cancel <job-id>
```
See `scripts/lavacli.yaml.example` for the `-i agl` identity config.

### 3. Direct curl (terminal/CI-driven)
```
scripts/lava-submit.sh templates/job-qemu-agl.yaml
scripts/lava-poll.sh <job-id> --watch
```
Both scripts resolve the token from env or `~/.config/lava/token` — see
`scripts/README.md`. Or by hand:
```
curl -sS -X POST -H "Authorization: Token $LAVA_TOKEN" \
     -H "Content-Type: application/json" \
     --data "$(jq -Rs '{definition: .}' < job.yaml)" \
     "$LAVA_URL/api/v0.2/jobs/"
```

## Rules for agents
- Never print or log the token. If a tool result would echo it back, redact.
- Treat `definition` YAML as untrusted-but-yours: validate device_type/priority fields exist
  before POSTing; don't fabricate device hostnames — look them up via `devices/` first.
- Prefer REST (MCP tools / curl) over XML-RPC (`lavacli`) unless the call is XML-RPC-only
  (see table above).
- Respect pagination — don't assume `jobs/` returns everything in one page.
- Start from a template (`templates/`) rather than free-hand YAML when creating a new
  job or test definition — it's faster and avoids missing mandatory fields.

## Toolkit layout
```
lava-toolkit/
├── LICENSE, NOTICE, SECURITY.md, CONTRIBUTING.md, CHANGELOG.md
├── docker-compose.yml, Caddyfile, .env.example, config/   containerized server + optional HTTPS
├── sbom/                  software bill of materials, per language
├── lava-skill.md          this file
├── docs/                  architecture, deployment, docker, testing, security, licensing, CRA, troubleshooting
├── templates/             job + test-definition YAML templates
├── scripts/               curl-based helpers + lavacli config example
├── go/{stdio,server}      MCP servers, Go
├── python/{stdio,server}  MCP servers, Python
├── rust/{stdio,server}    MCP servers, Rust (server variant also runs in docker-compose.yml)
└── cpp/{stdio,server}     MCP servers, C++
```
Each language/variant folder has its own `README.md` with build/run/test
instructions specific to it. `docs/deployment.md` has the decision table for
which one to pick; `docs/docker.md` covers the containerized `rust/server`
setup specifically.
