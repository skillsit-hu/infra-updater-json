# infra-updater: local code changes

infra-updater is going to be forked and rewritten to read the JSON config. Until then this repo runs a patched copy of the original in [infra-updater-src/](infra-updater-src/). This file records every code change made to that copy, so it can be checked against (or reapplied to) the fork.

The patched code was copied on 2026-10-14 from `ghcr.io/gyorfimarcell/infra-updater:latest` (image ID `sha256:8b4d57bf13655496e0f8582fcd37f2095fc525a084d7f839697979dffcb3f4ec`, built 2026-07-02).

## Changes

There is only one, in `update_infra.py` (`update_infra()`): the config is read as JSON instead of the old line-based text file. `main.py`, `settings.py` and the `Dockerfile` are unchanged.

```diff
+import json
 import re
 ...
     with open(settings.config_path, "r", encoding="utf8") as file:
         print(f"Reading main config at {settings.config_path}")
-        lines = [x.strip() for x in file.readlines()]
+        config = json.load(file)

-        base_domain = lines[0]
-        root_username = lines[2]
-        root_password = lines[3]
-        competitors = [x.split(" ")[0] for x in lines[5:]]
+    base_domain = config["domain"]
+    root_username = config["username"]
+    root_password = config["password"]
+    competitors = [x["username"] for x in config["competitors"]]
```

Without this change, the old code takes `{` (the first line of the JSON) as the domain and every Gitea call fails with `Failed to resolve 'git.%7b'`.

### Fields read from `config/main-config.json`

| Field                    | Replaces (old text format)             | Used as                                                               |
| ------------------------ | -------------------------------------- | --------------------------------------------------------------------- |
| `domain`                 | line 1                                 | Base domain for the Gitea API, the registry and the competitor routes |
| `username`               | line 3                                 | Gitea admin username                                                  |
| `password`               | line 4                                 | Gitea admin password                                                  |
| `competitors[].username` | first word of each line from line 6 on | Competitors whose repos get a container                               |

## When the fork arrives

Check that the fork reads these four fields from the JSON file. If it doesn't, apply the diff above.
