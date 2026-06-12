# YunKan — Unraid Community Applications templates

Single-container Unraid CA (Community Applications) templates for **YunKan** (云瞰). Once approved, Unraid users can install it from the **Apps** tab by searching "YunKan" — the experience is on par with a Home Assistant add-on.

> Sibling repos:
> - HA add-on: `yunkan_addons/` (project root)
> - docker compose (power user, with OTA sidecar): `compose.unraid.yml` (project root)

---

## Four variants (pick one — they share ports and cannot run together)

| Template file | Container name | Inference EP | Target hardware |
| --- | --- | --- | --- |
| `templates/yunkan-cpu.xml` | `YunKan` | onnxruntime CPU | any amd64 host, no hardware dependency |
| `templates/yunkan-openvino.xml` | `YunKan-OpenVINO` | OpenVINO + VAAPI | Intel iGPU (J4125 / N100 / N305 / 12-14th gen Core / Arc) |
| `templates/yunkan-cuda.xml` | `YunKan-CUDA` | onnxruntime CUDA | NVIDIA GPU with ≥6 GB VRAM |
| `templates/yunkan-trt.xml` | `YunKan-TRT` | TensorRT | NVIDIA GPU with ≥6 GB VRAM (highest throughput) |

The TRT variant builds GPU-specific inference engines at **first start, on the user's own GPU** (a few minutes, one time; cached under the data directory and reused across restarts / image updates). That keeps the per-GPU engine artifact out of the image, so the same CA template works across NVIDIA cards — the bundled CUDA / cuDNN / TensorRT all ride inside the container. The known failure mode is an older GPU / driver the bundled TensorRT can't target: the engine build errors out. The template's Overview steers those users back to **YunKan-CUDA** (same features, no engine build, no-wait first start), so CUDA stays the safe default and TRT is the opt-in throughput tier.

---

## Repository layout

```
yunkan_unraid_templates/
├── README.md              # this file
├── LICENSE                # MIT (CA requires an OSI-approved license at the repo root)
├── ca_profile.xml         # repo-level metadata (Profile / Icon / WebPage / Forum) — CA hard requirement
├── icon.png               # 512x512 PNG referenced by ca_profile.xml and every template's <Icon>
└── templates/             # one XML per Docker app (CA starter convention)
    ├── yunkan-cpu.xml         # CPU variant
    ├── yunkan-openvino.xml    # Intel iGPU + OpenVINO variant
    ├── yunkan-cuda.xml        # NVIDIA CUDA variant
    └── yunkan-trt.xml         # NVIDIA TensorRT variant (highest throughput)
```

The XML fields are a 1:1 translation of `compose.unraid.yml`, so the license fingerprint mounts, iGPU passthrough, NVIDIA runtime and appdata conventions are all aligned. **Change one side → change the other.**

---

## User install flow (the target experience)

### Once approved on CA (the steady state)

1. Unraid Web UI → **Apps** tab → search "yunkan"
2. Pick the variant that matches your hardware → click **Install**
3. CA renders the install form from this repo's XML — the defaults are sane:
   - `Data directory` defaults to `/mnt/user/appdata/yunkan/data` (point this at a cache pool if you record a lot)
   - `Time zone` defaults to `Etc/UTC` — change it to your local zone if you care about log timestamps and quiet-hours cron
   - **OpenVINO variant**: install the "Intel GPU TOP" plugin first; **CUDA variant**: install the "NVIDIA Driver" plugin first, then reboot
4. Click **Apply** → Unraid pulls the image and starts the container
5. Click **WebUI** on the container card → open `http://<Unraid-IP>:23406/` in a browser → first-run Setup wizard → pick SQLite → create an admin → add cameras

OTA updates: a red dot appears on the Apps page → click **Update**. No `yunkan-updater` sidecar is needed for this path.

### Early users (before CA approval)

While the CA submission is in review, users can add this repo manually:

1. Unraid Apps → top right **Settings** → **Template Repositories**
2. Paste this repo's URL: `https://github.com/mrtian2016/yunkan-unraid-templates`
3. Save → go back to **Apps** → search "yunkan" → flow continues as above

---

## Publishing to CA (current portal, 2026)

> The old route (PR against `Squidly271/Community-Applications-Moderators` editing `Repositories.json`) is **deprecated** as of 2026 — PR #15 was closed in April 2026 with "wrong process". Submissions now go through the official portal at [ca.unraid.net/submit](https://ca.unraid.net/submit).

### 0. Repository preflight (this repo already meets these)

The CA portal validates:

1. ✅ Repo is public and active
2. ✅ Root contains an OSI-approved `LICENSE` (this repo: MIT)
3. ✅ Root contains a `ca_profile.xml` with a non-empty `<Profile>` section
4. ✅ Valid Docker template XMLs (here: `templates/yunkan-{cpu,openvino,cuda,trt}.xml`)
5. (in portal) **Validate** + **Scan** both pass

Reference starter: [unraid/unraid-community-apps-starter](https://github.com/unraid/unraid-community-apps-starter)

### 1. Submit via the portal

1. Open [ca.unraid.net/submit](https://ca.unraid.net/submit) → sign in with GitHub OAuth (`mrtian2016`)
2. Paste the repo URL: `https://github.com/mrtian2016/yunkan-unraid-templates`
3. Click **Validate** → portal checks the preflight items above and parses `ca_profile.xml` + every `templates/*.xml`
4. Click **Scan** → security scan (looks at Docker image source, `:latest` usage, etc.)
5. All green → click **Submit** → enters the moderation queue

Read the current requirements first: [ca.unraid.net/submit/help](https://ca.unraid.net/submit/help). XML field reference: [ca.unraid.net/submit/help/xml-field-reference](https://ca.unraid.net/submit/help/xml-field-reference).

### 2. (Optional, after the fact) open an Unraid Forum support thread

CA encourages every app to have its own forum support thread. The usual section is Docker Containers: [forums.unraid.net/forum/52-docker-containers](https://forums.unraid.net/forum/52-docker-containers/).

After opening it:
- Fill the thread URL into `ca_profile.xml`'s `<Forum>` field (uncomment it)
- Change each template XML's `<Support>` from the GitHub Issues URL to the forum thread URL (so the "Support" button on the container page jumps to the forum)
- Commit + push — CA will pick up the change on its next refresh

Not required for initial approval; GitHub Issues is fine in the meantime.

### 3. CA refresh cadence

Once approved, CA auto-pulls from this repo. Edits to XMLs / icon / `ca_profile.xml` go straight to `main` — no resubmission needed. **Changing the `<Repository>` image tag or adding a new template does not trigger re-review.**

---

## Maintenance

### Change-sync matrix

| Changed | Also change | Why |
| --- | --- | --- |
| Ports (23406 / 23880 / 24214 / 23515) | all 4 XMLs in `templates/` + `yunkan_addons/*/config.yaml` + `compose.unraid.yml` + every deploy compose + nginx.conf + mediamtx.yml | Ports are hardcoded inside the image; changing one place and not the others = container installs but doesn't work |
| Image tag (`:latest` vs `:0.x.y`) | `<Repository>` in all 4 templates | CA users get new versions via "Force Update"; pinning a version means they miss updates |
| `SKYVIEW_SELF_CONTAINER_NAME` default | Must exactly match the template's `<Name>` field | The license heartbeat uses the container name to identify itself; a mismatch triggers a "container drift" alert |
| Added / removed volumes or devices | all 4 XMLs + `compose.unraid.yml` + the HA add-on `config.yaml` | All deploy paths must present the same runtime environment |
| Changed icon | Replace `icon.png` and commit; CA users see the new icon on the next refresh | Unraid does not cache the icon raw URL, and adding a `?v=2` query is a no-op — just overwrite the file |

### Differences from the HA add-on repo (`yunkan_addons/`)

- HA uses `config.yaml` (YAML); Unraid uses `.xml`. **Field semantics align but the format does not** — don't try to share templates between the two; write each separately.
- HA add-on ports are rendered through the supervisor onto the host; Unraid uses host networking directly. The runtime port behavior is identical, but the Unraid XML's `<Display>always` on ports is purely cosmetic — editing those values does nothing.
- HA add-on uses supervisor auth/ingress (optionally); Unraid has no ingress concept — you hit `http://[IP]:23406/` directly.
- HA add-on `SKYVIEW_SELF_CONTAINER_NAME=addon_local_yunkan*` (supervisor's prefix); Unraid uses whatever the user named the container (this repo defaults to `YunKan` / `YunKan-OpenVINO` / `YunKan-CUDA` / `YunKan-TRT`).

### Don'ts

- **Don't ship `yunkan-updater` as a CA template** — the updater expects `compose.unraid.yml` to exist on the host (it `exec`s `docker compose` commands). The CA single-container path has no compose file, so the updater would just fail to boot. OTA on CA is "Force Update", and that's enough.
- **Don't split Setup mode into a separate template** — the container is _already_ in Setup mode when it first starts (`data/database.toml` doesn't exist yet). The user enters the wizard right after install; no second template needed.
- **Don't mask license or secret fields** — every `<Config>` here uses `Mask="false"`, because there is no user-input secret. The license itself is pasted in the Setup wizard inside the running container; it never lives in a container env var.

---

## License

YunKan is commercial software. The container image is free to use, but AI detection features require a license to activate — see https://yun-kan.com pricing. The Unraid XML templates in **this** repo are MIT.

---

## 简介(中文)

云瞰是面向家庭用户的自托管摄像头系统,本地优先,支持人 / 车 / 人脸 / 车牌 / 姿态 / 跌倒检测,视频不出局域网。本仓库提供 4 个 Unraid CA 单容器模板(CPU / Intel iGPU OpenVINO / NVIDIA CUDA / NVIDIA TensorRT),通过审核后可在 Unraid Apps 标签页搜 "yunkan" 一键安装。完整中文安装文档:[yun-kan.com/zh-CN/docs/install-unraid](https://yun-kan.com/zh-CN/docs/install-unraid)。

模板字段与项目根 `compose.unraid.yml` 一一对应,改一边记得改另一边——具体清单见上方 **Change-sync matrix**。
