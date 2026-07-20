# Golden Helix Automation Registry

A catalog of workflows, applications, and plugins that can be imported into the
Golden Helix VSWarehouse server and run on its workflow/application/plugin
runtime.

- **Workflows** — Workflows > Actions > Add Workflow from Repository.
- **Applications** — Applications > Actions > Add Application from Repository.
- **Plugins** — Plugins > Add from Repository.

## Manifest files

| File | Read by | Status |
|---|---|---|
| `workflow-repository.json` | all servers | active |
| `application-repository.json` | **v3.0 servers** | **deprecated / frozen** |
| `application-registry.json` | **v3.1+ servers** | **go-forward** |
| `plugin-registry.json` | v3.1+ servers | go-forward |

### Why two application manifests

v3.1 added **version selection**: an application/plugin YAML is version-agnostic
and references its image as `${SOME_DOCKER_IMAGE}`; the workspace pins a concrete
image for that variable and resolves it at launch. v3.0 servers have none of
that machinery — handed a `${VAR}`-based YAML they cannot resolve the image and
the app won't start.

A v3.0 client also predates the `minServerVersion` filter (below), so it would
ignore that guard and try to install anyway. The only boundary a v3.0 client
respects is the URL it fetches. So the cut is by **file**:

- `application-repository.json` is **frozen** as the v3.0 manifest. Leave its
  entries and their `applications/*.yaml` + `images/*` files as-is. **Do not add
  version-selection entries here.**
- `application-registry.json` is the **go-forward** manifest (v3.1+). New and
  version-selectable apps live here, under `apps/<name>/` (YAML + icon
  co-located). Plugins are v3.1-only, so there is a single `plugin-registry.json`
  with no frozen predecessor.

This file split was a **one-time** step for the 3.0→3.1 transition. Every later
transition is handled per-entry with `minServerVersion` — no new manifest files.

## Directory layout

```
application-repository.json      # frozen v3.0 app manifest
application-registry.json        # go-forward app manifest (v3.1+)
plugin-registry.json             # plugin manifest (v3.1+)
workflow-repository.json         # workflow manifest
applications/  images/           # frozen v3.0 app YAMLs + icons
apps/<name>/                     # go-forward app YAML + icon, co-located
plugins/<name>/                  # plugin YAML (+ icon if not using icon_name)
```

## Application / plugin entry schema

Application entries live in the `workflows` array (legacy key name); plugin
entries in the `plugins` array.

| Field | Required | Notes |
|---|---|---|
| `name`, `description`, `version` | yes | `version` is legacy display; `versions[]` is the selectable list |
| `applicationYamlUrl` / `pluginYamlUrl` | yes | relative to the manifest URL, or absolute |
| `iconUrl` | app: yes / plugin: no | plugins may use a built-in `icon_name` and ship no icon file |
| `websiteUrl` | app: yes / plugin: no | |
| `tags` | yes | filter chips |
| `imageVar` | no | the `${VAR}` the YAML references, e.g. `IGV_DOCKER_IMAGE` |
| `imageRepository` | no | repo the tags belong to when all versions share one repo |
| `versions` | no | `[{ label, tag, image?, default?, minServerVersion? }]`; when present the dialog shows a version picker |
| `minServerVersion` | no | server semver required to run the entry; newer clients hide it on older servers |

### Versions

Each version pins a concrete image. Either:

- share `imageRepository` on the entry and give each version a `tag` (pin =
  `imageRepository:tag`) — used by **VSAgent**; or
- give each version a full `image` when the repository itself changes across
  versions — used by **IGV** (`ghdesktop-igv:2.19.7` vs
  `ghdesktop-igv3:3.0.0-beta.1`), consolidating what used to be two entries into
  one version-selectable app.

The chosen version is pinned into the workspace `${imageVar}` and resolved at
launch and pre-pull. The YAML is always refreshed to the latest on checkout, so
**it must run against every tag offered in `versions[]`.** Drop a version from
`versions[]` to desupport its image.

### `minServerVersion` — worked example

Use it to offer a version/entry only newer servers can run. Clients filter out
anything their server predates (the client's build version is the server
version — they ship together).

```json
{
  "name": "VSAgent",
  "applicationYamlUrl": "apps/vsagent/VSAgent.application.yaml",
  "iconUrl": "apps/vsagent/vsagent.svg",
  "imageVar": "VSAGENT_DOCKER_IMAGE",
  "imageRepository": "registry.goldenhelix.com/public/vsagent",
  "minServerVersion": "3.1.0",
  "versions": [
    { "label": "v0.6.3 (latest)", "tag": "v0.6.3", "default": true },
    { "label": "v0.6.2", "tag": "v0.6.2" }
  ]
}
```

A 3.1.0 server sees and installs it; a 3.0 server (still on
`application-repository.json`) never sees it. To ship a next-version app to
insiders only, add it here with `minServerVersion` set to the upcoming version:
insiders builds report that version and see it, released builds are one version
behind and filter it out — no separate branch required.

## Compatibility rules

- **Additive-only**: only ever add *optional* fields; never remove, rename, or
  retype a required field. Clients validate with a non-strict schema, so unknown
  fields are ignored and an entry without `versions`/`imageVar` still works (no
  picker).
- **Frozen manifest is frozen**: never add `${VAR}`/`versions` entries to
  `application-repository.json`.

## About Golden Helix

Golden Helix provides genomic analysis software, with VarSeq as the flagship
product for variant analysis and interpretation.

- [Golden Helix Website](https://www.goldenhelix.com/)
- [VarSeq Product Page](https://www.goldenhelix.com/products/VarSeq/)
