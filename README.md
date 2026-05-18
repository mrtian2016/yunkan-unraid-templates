# 云瞰 YunKan — Unraid Community Applications 模板仓库

云瞰的 Unraid CA（Community Applications）单容器模板仓库。Unraid 用户可在 **Apps** 标签页搜 "YunKan / 云瞰" 一键安装,体验对齐 Home Assistant 加载项。

> 对应 HA 加载项仓库:`yunkan_addons/`(项目根)。
> 对应 docker compose 路径(power user / 带 OTA sidecar):`compose.unraid.yml`(项目根)。

---

## 三个变体(按硬件二/三选一,**端口一致不能同时启用**)

| 模板文件 | 容器名 | 推理 EP | 适用硬件 |
| --- | --- | --- | --- |
| `yunkan-cpu.xml` | `YunKan` | onnxruntime CPU | 任何 amd64,无硬件依赖 |
| `yunkan-openvino.xml` | `YunKan-OpenVINO` | OpenVINO + VAAPI | Intel iGPU(J4125 / N100 / N305 / 12-14 代 Core / Arc) |
| `yunkan-cuda.xml` | `YunKan-CUDA` | onnxruntime CUDA | NVIDIA GPU ≥6GB 显存 |

TRT 变体(`yunkan-trt`)不上 CA——驱动 + 显卡型号 + cuDNN 组合约束太多,不适合"一键装"语义。NVIDIA 用户先用 CUDA 变体,需要 TRT 极致性能再走 `compose.unraid.yml`。

---

## 仓库布局

```
yunkan_unraid_templates/
├── README.md              # 本文档
├── icon.png               # 512x512 PNG,Unraid Apps 列表 + 容器面板 icon
├── yunkan-cpu.xml         # CPU 变体模板
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

## 发布到 CA 的完整流程(维护者自用)

### 1. 把本目录推到公开 GitHub 仓库

```bash
# 在云瞰主仓库外,新建 mrtian2016/yunkan-unraid-templates 公开 repo
git init yunkan-unraid-templates
cp -r /home/vision/projects/skyview/yunkan_unraid_templates/* yunkan-unraid-templates/
cd yunkan-unraid-templates
git add . && git commit -m "init: cpu / openvino / cuda templates"
git remote add origin git@github.com:mrtian2016/yunkan-unraid-templates.git
git push -u origin main
```

> **TODO 占位修复**:推之前在三个 XML 里 grep `mrtian2016/yunkan-unraid-templates`,确认 `<Support>` / `<TemplateURL>` / `<Icon>` 三个字段的 GitHub URL 是真实仓库地址。当前是占位,推之前必须 sed 替换。

### 2. 验证模板可装

不等 CA 收录,先让自己 / 内测用户手动加仓库验:

- Unraid → Apps → Settings → Template Repositories → 加 GH URL
- Apps 搜 "yunkan" → 应该看到 3 个 → 任选一个装 → 跑 Setup 向导 → 录像 / 推理跑通

### 3. 向 CA 仓库提 PR 登记

CA 维护者 Andrew "Squid" Zawadzki 接受 PR 把仓库登记进官方索引。流程:

1. Fork `Squidly271/AppFeed` 或 `Squidly271/Community-Applications`(以 CA 官方 README 当前指向的为准)
2. 在 `repositories.json`(或当前要求的文件名)加一行:
   ```json
   {
     "name": "mrtian2016's Repository",
     "url": "https://github.com/mrtian2016/yunkan-unraid-templates"
   }
   ```
3. PR 描述说明:云瞰是商业化 self-hosted VMS,3 个 amd64 变体,maintainer 是 `support@yun-kan.com`
4. 等审核(通常 1-7 天)

> CA 提交规范偶尔变,提交前先看 https://forums.unraid.net/topic/87144-ca-application-policies-please-read/ 当前要求。

### 4. CA 收录后

CA 后台自动每 30 分钟 / 用户 Apps refresh 时拉本仓库根目录所有 `*.xml`。改 XML 推到 main 即可,无需通知 Squid。

---

## 维护注意

### 改动同步矩阵

| 改了什么 | 同步改 | 原因 |
| --- | --- | --- |
| 端口号(23406 / 23880 / 24214 / 24215 / 23515) | 本目录 3 个 XML + `yunkan_addons/*/config.yaml` + `compose.unraid.yml` + 各 deploy compose + nginx.conf + mediamtx.yml | 端口在镜像里 hardcode,改一处不改全部 = 装上不能用 |
| 镜像 tag(`:latest` ↔ `:0.x.y`) | 本目录 3 个 XML 的 `<Repository>` | CA 用户走 "Force Update" 拉新 tag,pin 版本会让用户漏更新 |
| `SKYVIEW_SELF_CONTAINER_NAME` 默认值 | 必须与 `<Name>` 字段完全一致 | License 心跳用容器名识别自身,不一致会触发"容器漂移"告警 |
| 新加 / 删 volume / device | 3 个 XML 一起改 + `compose.unraid.yml` + HA addon `config.yaml` | 4 条部署路径必须给出一样的运行时环境 |
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
