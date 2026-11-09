# Operit 技能 · APK 逆向改包

让 Operit 里的 AI 在**手机本机**完成 APK 反编译、改默认值/开关/常量/资源/标识、回编译、重签名、验证。
不需要电脑，也不需要 adb —— 设备就是目标机。

适合这些活：改默认设置、看某段逻辑怎么走的、重打包后装不上、签名不匹配。

技能沉淀了三条硬规则（详见 `SKILL.md`）：**先判断要改的在哪一层**（dex / resources / manifest / shared_prefs）、
**一次只改一个变量**、**没在真机验证过就不算完成**。第 0 步是一棵决策树：
改 shared_prefs 直接 root 改、不破签名；改代码或资源才走 `decode → 改 → build → sign`。

## 安装

装到哪：`/sdcard/Download/Operit/skills/APK逆向改包/` —— Operit 的技能目录，放好即被识别。

### 方式一：让 Operit 的 AI 自己装（推荐）

把下面这句话直接发给 Operit 里的 AI：

> 安装技能：APK 逆向改包，仓库是 https://github.com/LinZeroZero/operit-skill-apk-reverse

AI 会去 Release 取包、解开、校验 `SKILL.md` 是否到位。

### 方式二：一行命令（Android 自带 curl + tar，不需要 root）

```sh
curl -fL https://github.com/LinZeroZero/operit-skill-apk-reverse/releases/latest/download/apk-reverse.tar.gz \
  | tar -xz -C /sdcard/Download/Operit/skills
```

### 方式三：安装脚本（带回滚、验包、`--check`）

```sh
curl -fL -o /data/local/tmp/apk-install.sh \
  https://raw.githubusercontent.com/LinZeroZero/operit-skill-apk-reverse/main/install.sh

sh /data/local/tmp/apk-install.sh --check   # 先看现在装的是什么，不下载
sh /data/local/tmp/apk-install.sh           # 安装
```

脚本流程：下载 → `tar -tzf` 验包 → 备份旧版 → 解包 → 校验 `SKILL.md`。
已存在同名目录时**不会直接覆盖**，而是改名为 `APK逆向改包.bak.<时间戳>`；确定要硬覆盖就加 `--force`。

先试装到别处、不影响现有技能：

```sh
sh /data/local/tmp/apk-install.sh /sdcard/tmp/skills
```

### 方式四：git clone（想用 `git pull` 跟进更新）

```sh
git clone --depth 1 https://github.com/LinZeroZero/operit-skill-apk-reverse.git \
  "/sdcard/Download/Operit/skills/APK逆向改包"
```

## 装完验证

```sh
ls "/sdcard/Download/Operit/skills/APK逆向改包/SKILL.md"
```

然后在 Operit 新对话里说「用 APK逆向改包 技能帮我看看某个 APK」，AI 会读 `SKILL.md` 并按决策树选路线。

技能本身不含二进制工具，靠这三样：Operit 内置的 `apk_reverse` 包（`inspect / decode / jadx / search / sign`）、
proot 里的 apktool 与 jadx（回编译只能走这条路，原因见 `references/toolchain-apk_reverse.md`）、root 权限。

## 目录

| 路径 | 内容 |
|---|---|
| `SKILL.md` | 技能主文件：第 0 步决策树、三条硬规则、命令清单 |
| `references/patch-layers.md` | 改动分五层：常量 / 开关 / 资源 / 标识 / 包名 |
| `references/toolchain-apk_reverse.md` | 工具链：哪些能跑、哪些被 SELinux 拦住、怎么绕过 |
| `references/symptoms.md` | 症状 → 原因（装不上、闪退、签名冲突…） |
| `scripts/apkw.sh` | 工作区管理：`new / list / clean / drop` |

## 更新

- 方式一/三：重跑 `install.sh`（旧目录自动备份成 `.bak.<时间戳>`）
- 方式四：`git -C "/sdcard/Download/Operit/skills/APK逆向改包" pull`

## 卸载

```sh
mv "/sdcard/Download/Operit/skills/APK逆向改包" "/sdcard/Download/APK逆向改包.bak"
```

不提供 `install.sh --remove`：删目录不可逆，交给使用者自己决定更安全。

## 依赖

| 依赖 | 是否有 | 说明 |
|---|---|---|
| `curl`、`tar`、`gzip` | Android 自带 | `/system/bin`，toybox 0.8.12，装技能零额外依赖 |
| `apk_reverse` 包 | Operit 内置 | 反编译、搜索、重签名 |
| apktool + jadx | 需自备 | 放在 proot Ubuntu 里；回编译必须用 apktool，App 进程内执行 aapt2 会被 SELinux 拒绝 |
| root | 需要 | 改 `/data/data/<包名>/shared_prefs` 这类“不破签名”的改动必须有 |

## 说明

技能里的示例路径、包名一律是占位写法（`com.example.app`、`/sdcard/your-project`）。
自己的改包工程不要提交到仓库里 —— APK 与应用资源可能含第三方版权内容。