# 云瞰 YunKan — Unraid Community Applications 模板仓库

云瞰的 Unraid CA（Community Applications）单容器模板仓库。Unraid 用户可在 **Apps** 标签页搜 "YunKan / 云瞰" 一键安装,体验对齐 Home Assistant 加载项。

> 对应 HA 加载项仓库:`yunkan_addons/`(项目根)。
> 对应 docker compose 路径(power user / 带 OTA sidecar):`compose.unraid.yml`(项目根)。

---

## 三个变体(按硬件二/三选一,**端口一致不能同时启用**)

| 模板文件 | 容器名 | 推理 EP | 适用硬件 |
| --- | --- | --- | --- |
| `templates/yunkan-cpu.xml` | `YunKan` | onnxruntime CPU | 任何 amd64,无硬件依赖 |
| `templates/yunkan-openvino.xml` | `YunKan-OpenVINO` | OpenVINO + VAAPI | Intel iGPU(J4125 / N100 / N305 / 12-14 代 Core / Arc) |
| `templates/yunkan-cuda.xml` | `YunKan-CUDA` | onnxruntime CUDA | NVIDIA GPU ≥6GB 显存 |

TRT 变体(`yunkan-trt`)不上 CA——驱动 + 显卡型号 + cuDNN 组合约束太多,不适合"一键装"语义。NVIDIA 用户先用 CUDA 变体,需要 TRT 极致性能再走 `compose.unraid.yml`。

---

## 仓库布局

```
yunkan_unraid_templates/
├── README.md              # 本文档
├── LICENSE                # MIT(CA 提交硬要求:OSI-approved license at repo root)
├── ca_profile.xml         # 仓库级 metadata(Profile / Icon / WebPage / Forum)——CA 提交硬要求
├── icon.png               # 512x512 PNG,被 ca_profile.xml 和各 template 的 <Icon> 引用
└── templates/             # 每个 Docker app 一个 XML(CA starter 约定)
    ├── yunkan-cpu.xml         # CPU 变体
    ├── yunkan-openvino.xml    # Intel iGPU + OpenVINO 变体
    └── yunkan-cuda.xml        # NVIDIA CUDA 变体
```

XML 字段对照 `compose.unraid.yml` 1:1 翻译过来,所以 license 指纹 / iGPU 透传 / nvidia runtime / appdata 习惯 全部对齐。改动一边记得改另一边。

---

## 用户安装路径(目标体验)

### 已上 CA 的用户(理想态)

1. Unraid Web UI → **Apps** 标签页 → 搜索框输入 "yunkan" 或 "云瞰"
2. 找到三个变体里匹配自己硬件的那个 → 点 **Install**
3. CA 弹出表单(本仓库 XML 渲染),默认值即可:
   - `数据目录 appdata` 默认 `/mnt/user/appdata/yunkan/data`(录像写入大时建议改 cache pool)
   - `时区 TZ` 默认 `Asia/Shanghai`
   - **OpenVINO 变体先在 Plugins 装 "Intel GPU TOP"**;**CUDA 变体先装 "NVIDIA Driver" plugin** 并重启 Unraid
4. 点 **Apply** → Unraid 拉镜像、启动容器
5. 容器面板点 **WebUI** → 浏览器打开 `http://<Unraid-IP>:23406/` → Setup 向导 → 选 SQLite → 建管理员账号 → 加摄像头

OTA 升级:Apps 页面看到红点 → 点 **Update**。无需 yunkan-updater sidecar。

### 还没上 CA 的早期用户

CA 仓库登记通过前(见下文"发布到 CA"),用户可以**手动添加**本仓库:

1. Unraid Apps → 右上 **Settings** → **Template Repositories**
2. 粘贴本仓库 GitHub 地址:`https://github.com/mrtian2016/yunkan-unraid-templates`(发布前为占位)
3. 保存 → 回 Apps 搜 "yunkan" → 后续流程同上

---

## 发布到 CA 的完整流程(2026 新版 portal)

> 历史路径(向 `Squidly271/Community-Applications-Moderators` 提 PR 改 `Repositories.json`)在 2026 年已被 deprecate(PR #15 2026-04 被关 "wrong process")。现在必须走官方门户 [ca.unraid.net/submit](https://ca.unraid.net/submit)。

### 0. 仓库前置硬要求(本仓库已满足)

CA portal 校验时会检查:

1. ✅ 仓库 public 且 active
2. ✅ 根目录有 OSI-approved `LICENSE`(本仓库:MIT)
3. ✅ 根目录有 `ca_profile.xml` 含非空 `<Profile>` 段(repo 级元数据)
4. ✅ 有效的 Docker 模板 XML(本仓库:`templates/yunkan-{cpu,openvino,cuda}.xml`)
5. (portal 时)Run **Validate** + **Scan** 通过

参考官方 starter:[unraid/unraid-community-apps-starter](https://github.com/unraid/unraid-community-apps-starter)

### 1. 走 portal 提交

1. 浏览器开 [ca.unraid.net/submit](https://ca.unraid.net/submit) → GitHub OAuth 登录(用 `mrtian2016`)
2. 粘 repo URL:`https://github.com/mrtian2016/yunkan-unraid-templates`
3. 点 **Validate** → portal 检查上述硬要求 + 解析 ca_profile.xml + 解析 templates/*.xml
4. 点 **Scan** → 安全扫描(可能查 Docker image 来源、`:latest` 用法等)
5. 都过了点 **Submit** → 进 moderation 队列

提交前先读 [ca.unraid.net/submit/help](https://ca.unraid.net/submit/help) 当前要求。XML 字段 reference:[ca.unraid.net/submit/help/xml-field-reference](https://ca.unraid.net/submit/help/xml-field-reference)。

### 2.(可选 / 后置)发 Unraid Forum Support Thread

CA 鼓励每个 app 有自己的 forum 支持帖给用户提问,版块通常是 Docker Containers:[forums.unraid.net/forum/52-docker-containers](https://forums.unraid.net/forum/52-docker-containers/)。

发完后:
- 把 thread URL 填进 `ca_profile.xml` 的 `<Forum>` 字段(取消 XML 注释)
- 三个 template XML 的 `<Support>` 字段从 GitHub Issues 改为 forum thread URL(CA 用户在容器面板点 "Support" 跳到帖子)
- commit + push,CA 下次 refresh 自动同步

非强制,审核期间可以暂时只挂 GitHub Issues。

### 3. CA portal 拉取节奏

通过审核后 CA 后台自动拉本仓库,改 XML / icon / ca_profile.xml 直接推到 main,不需要重新提交。**改 `<Repository>` image tag 或新增 template 不需要重审**。

---

## 维护注意

### 改动同步矩阵

| 改了什么 | 同步改 | 原因 |
| --- | --- | --- |
| 端口号(23406 / 23880 / 24214 / 24215 / 23515) | `templates/` 下 3 个 XML + `yunkan_addons/*/config.yaml` + `compose.unraid.yml` + 各 deploy compose + nginx.conf + mediamtx.yml | 端口在镜像里 hardcode,改一处不改全部 = 装上不能用 |
| 镜像 tag(`:latest` ↔ `:0.x.y`) | `templates/` 下 3 个 XML 的 `<Repository>` | CA 用户走 "Force Update" 拉新 tag,pin 版本会让用户漏更新 |
| `SKYVIEW_SELF_CONTAINER_NAME` 默认值 | 必须与 `<Name>` 字段完全一致 | License 心跳用容器名识别自身,不一致会触发"容器漂移"告警 |
| 新加 / 删 volume / device | `templates/` 下 3 个 XML 一起改 + `compose.unraid.yml` + HA addon `config.yaml` | 4 条部署路径必须给出一样的运行时环境 |
| 改 icon | 替换 `icon.png` 后 commit,CA 用户下次 refresh 自动看到新图 | Unraid 不缓存 icon raw URL,加 `?v=2` query 也无效——直接覆盖原文件 |

### 与 HA addon 仓库(`yunkan_addons/`)的差异

- HA 用 `config.yaml`(YAML),Unraid 用 `.xml`,**字段语义对齐但格式不同**——不要试图 share template,各写各的
- HA addon 端口由 supervisor 渲染到 host,Unraid 走 host network 直绑——所以两边 `ports` 行为一样,但 Unraid XML 里写的端口 `<Display>always` 只是给用户看,改它没用
- HA addon 走 supervisor 鉴权 ingress(可选),Unraid 没有 ingress 概念,直接 `http://[IP]:23406/`
- HA addon `SKYVIEW_SELF_CONTAINER_NAME=addon_local_yunkan*`(supervisor 给的容器名前缀),Unraid 用用户起的容器名(本模板默认 `YunKan` / `YunKan-OpenVINO` / `YunKan-CUDA`)

### 不要做的事

- **不要把 yunkan-updater sidecar 也做成 CA 模板**——updater 依赖 `compose.unraid.yml` 在宿主机存在(它 exec `docker compose` 命令),CA 单容器路径下没有 compose 文件,updater 会 boot fail。OTA 走 CA 自己的 "Force Update" 即可。
- **不要拆 setup 模式成独立模板**——容器跑起来就是 setup 模式(`data/database.toml` 不存在),装完直接进向导,不需要二次模板。
- **不要 mask license / 密钥字段**——本模板里所有 `<Config>` 都是 `Mask="false"`,因为没有用户输入的敏感信息;license 是装完进 Setup 向导时贴的,不走容器 env。

---

## License

云瞰是商业化软件,容器镜像本身免费使用但 AI 检测等功能需 license 激活。详见 https://yun-kan.com 定价页。本仓库的 Unraid XML 模板代码 MIT。
