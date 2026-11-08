# XiHuu — 多世界保护 / PvP / 饱腹 / 无限血量（跨版本）

一个轻量级的世界级规则管理插件：按世界（完整兼容 Multiverse-Core 多世界与别名）开关
**方块保护、PvP 限制、永久饱腹、无限血量**，全部通过游戏内 `/xihuu` 指令实时配置并自动持久化。

同一套源码提供两个构建：

- **通用版**（推荐）：一份 jar 覆盖 **Paper / Spigot / Purpur 1.13.2 – 1.21.11**
- **原生版**：针对 Paper 1.21.8 的 Paper API + Java 21 构建

## 功能一览

| 功能 | 说明 |
| --- | --- |
| 世界保护 | 禁止在该世界放置方块、破坏方块（管理员可用 `xihuu.bypass` 绕过） |
| 禁止 PvP | 玩家之间、箭矢、TNT 等造成的玩家伤害全部取消 |
| 取消保护 / 取消禁 PvP | 随时一键恢复 |
| 永久饱腹 | 饥饿值与饱和度锁定在满值，不再消耗 |
| 无限血量（不掉血） | 取消该世界玩家受到的一切伤害，血量始终保持满值 |

> 所有功能都按**世界**独立生效，支持 Multiverse-Core 的多世界与别名。

## 支持版本

| 构建 | 支持范围 | 已实测 | 说明 |
| --- | --- | --- | --- |
| `xihuu-1.0.0-universal.jar`（推荐） | Paper / Spigot / Purpur **1.13.2 – 1.21.11** | 1.18.2、1.20.4、1.21.8、1.21.11 | 编译基线 Spigot API 1.13.2，Java 8 字节码，`api-version: '1.13'` |
| `xihuu-1.0.0-1.21.8.jar` | Paper **1.21.8** | 1.21.8（含 Multiverse-Core 5.8.1） | 编译基线 Paper API 1.21.8，Java 21 字节码，`api-version: '1.21'` |

- 通用版基于 1.13.2 API 编译，理论上覆盖 1.13.2–1.17.x，但实测范围为 1.18.2–1.21.11。
- **不支持 1.8–1.12 及 Folia**（Folia 的调度器模型不同）。
- Multiverse-Core：已在 **5.8.1** 实测；内部同时兼容 MV4 旧版 API；未安装 MV 时插件照常工作，只是不识别 MV 别名。
- 服务端需 Java 8+ 运行通用版（推荐 17/21）；原生版需 Java 21+。

## 安装

1. 把 `xihuu-1.0.0-universal.jar` 放入服务器 `plugins/` 目录（只支持 Paper 1.21.8 时也可用原生版）。
2. 重启服务器（或 `/reload confirm`，推荐重启）。
3. 首次启动会生成：
   - `plugins/XiHuu/config.yml` —— 消息文本配置（可自由改颜色/文字）
   - `plugins/XiHuu/worlds.yml` —— 世界设置数据库（插件自动维护，**不建议手动编辑**）

## 命令

权限节点：`xihuu.admin`（默认仅 OP）。

```
/xihuu protect    <世界> [on|off]    世界保护：禁止放置/破坏方块
/xihuu unprotect  <世界>             取消世界保护
/xihuu pvp        <世界> [on|off]    禁止 PvP
/xihuu unpvp      <世界>             取消禁止 PvP
/xihuu saturation <世界> [on|off]    永久饱腹
/xihuu health     <世界> [on|off]    无限血量（不掉血）
/xihuu info       [世界]             查看某世界当前设置
/xihuu list                         列出所有已配置的世界
/xihuu reload                       重载 config.yml / worlds.yml
/xihuu help                         查看帮助
```

**快捷用法**

- 玩家执行时省略世界名，默认操作**自己所在的世界**：
  `/xihuu protect`  → 给当前世界开保护
  `/xihuu pvp off`  → 当前世界允许 PvP
- 支持世界名大小写不敏感与 Multiverse 别名（例如 MV 别名 `hub` 会解析到真实世界名）。
- 支持 `on/off/true/false/enable/disable/yes/no/开/关` 等开关写法，命令别名 `/xh`。

**示例：给主城世界开保护并禁止 PvP、开启饱腹与无限血量**

```
/xihuu protect spawn
/xihuu pvp spawn
/xihuu saturation spawn
/xihuu health spawn
/xihuu info spawn
```

## 权限

| 权限 | 默认 | 说明 |
| --- | --- | --- |
| `xihuu.admin` | OP | 允许使用 `/xihuu` 全部管理命令 |
| `xihuu.bypass` | OP | 绕过世界保护与 PvP 限制（建筑管理员用） |

> **测试提示**：OP 默认拥有 `xihuu.bypass`，会绕过保护与 PvP 限制。
> 想验证保护效果，请用**非 OP 玩家**测试，或在 LuckPerms 中移除该玩家的 `xihuu.bypass` 权限。

## 配置文件

`config.yml` 只保存**提示消息**，所有世界规则通过命令修改并写入 `worlds.yml`。
想修改提示文字/颜色，编辑 `config.yml` 的 `messages:` 节点后执行 `/xihuu reload`。
如果删除了 `config.yml`，重启服务器会自动重新生成默认配置。

`worlds.yml` 结构示例（自动生成）：

```yaml
worlds:
- world: world
  protect: false
  pvp-disabled: false
  saturation: false
  no-damage: false
- world: spawn
  protect: true
  pvp-disabled: true
  saturation: true
  no-damage: true
```

## 从源码编译

需要 Maven 3.8+ 与 JDK 8+（推荐 17/21；JDK 25 亦可）。

```bash
# 通用版（默认）：Spigot API 1.13.2 + Java 8 字节码
mvn -q package
# 产物：target/xihuu-1.0.0-universal.jar

# Paper 1.21.8 原生版：Paper API 1.21.8 + Java 21 字节码
mvn -q -Pnative package
# 产物：target/xihuu-1.0.0-1.21.8.jar
```

## 实测记录

跨版本端到端测试（每个版本均安装通用版 jar，通过 RCON 执行真实指令）：

| 服务端版本 | Java | 加载/启用 | 四类开关+取消 | 配置持久化 | 异常日志 |
| --- | --- | --- | --- | --- | --- |
| Paper 1.18.2 | 17 | ✅ | ✅ | ✅ | 无 |
| Paper 1.20.4 | 17 | ✅ | ✅ | ✅ | 无 |
| Paper 1.21.8 | 25 | ✅ | ✅ | ✅ | 无 |
| Paper 1.21.11 | 25 | ✅ | ✅ | ✅ | 无 |

另外在 Paper 1.21.8 + Multiverse-Core 5.8.1 上验证了原生版：MV 检测、别名解析、指令与持久化均正常；
此前也验证了通用版在 Paper 1.21.8 + MV 5.8.1 下的别名解析（`hub` → `testworld`）。

## 实现说明 / 注意事项

- 保护监听使用 `HIGHEST` 优先级并 `ignoreCancelled`，兼容领地类插件（如 Dominion 等先处理的插件）。
- 无限血量通过取消 `EntityDamageEvent` 实现（摔落、火焰、PvP、怪物、虚空等伤害都会被拦截）；`/kill` 这类直接执行指令造成的死亡不受事件拦截。
- 与其它「禁用饥饿/无敌」类插件可能功能重复，建议只保留一个。
- 通用版为兼容旧服务端使用 Java 8 字节码与 1.13.2 API 基线，因此代码中不依赖任何新版本 API；在 1.21.x 上同样完整可用。
- 若使用 Multiverse 的 `world` 别名（alias），请确认 MV 已正常加载；插件会通过反射安全读取，MV 缺失时自动降级。
- 建议先在测试服验证，再部署到正式服。
