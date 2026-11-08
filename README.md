# xihuuworld_set
# XiHuu — Paper 1.21.8 多世界保护 / PvP / 饱腹 / 无限血量插件

一个轻量级的世界级规则管理插件，**完整兼容 Multiverse-Core**（识别 MV 世界名与别名），
所有设置都可以在游戏内用 `/xihuu` 命令即时修改并自动保存。

## 功能一览

| 功能 | 说明 |
| --- | --- |
| 世界保护 | 禁止在该世界放置方块、破坏方块（管理员可用 `xihuu.bypass` 绕过） |
| 禁止 PvP | 玩家之间、箭矢、TNT 等造成的玩家伤害全部取消 |
| 取消保护 / 取消禁 PvP | 随时一键恢复 |
| 永久饱腹 | 饥饿值与饱和度锁定在满值，不再消耗 |
| 无限血量（不掉血） | 取消该世界玩家受到的一切伤害，血量始终保持满值 |

> 所有功能都按**世界**独立生效，支持 Multiverse-Core 的多世界与别名。

## 环境要求

- 服务端：**Paper 1.21.8**（已在 Paper `1.21.8-40` 实测通过，Paper API `1.21.8-R0.1-SNAPSHOT`）
- Java：**21 及以上**（实测 Java 21 / 25 均可运行）
- 可选：**Multiverse-Core**（已在 MV-Core `5.8.1` 实测通过；内部同时兼容 MV4 旧版 API。未安装时插件照常工作，只是不识别 MV 别名）

## 安装

1. 将 `xihuu-1.0.0.jar` 放入服务器 `plugins/` 目录。
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
- 支持世界名大小写不敏感与 Multiverse 别名（如 MV 别名 `hub` 会解析到真实世界名）。
- 支持 `on/off/true/false/enable/disable/yes/no/开/关` 等开关写法。

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

```bash
# 需要 JDK 21+ 与 Maven 3.8+
mvn -q package
# 产物：target/xihuu-1.0.0.jar
```

## 实测记录

已在真实 Paper 服务端做端到端测试（Paper 1.21.8-40 + Multiverse-Core 5.8.1）：

- 插件正常加载、启用、停用，无报错；
- `/xihuu protect|pvp|saturation|health` 开启与关闭、`unprotect|unpvp` 取消均正常；
- 设置写入 `worlds.yml`，重启服务器后仍然生效；
- 带颜色的消息文本、`/xihuu reload` 热重载正常；
- Multiverse 世界别名解析正常（例如 MV 别名 `hub` → 真实世界 `testworld`）；
- 控制台缺少世界参数时正确提示用法，未知子命令正确报错。

## 实现说明 / 注意事项

- 保护监听使用 `HIGHEST` 优先级并 `ignoreCancelled`，兼容领地类插件（如 Dominion 等先处理的插件）。
- 无限血量通过取消 `EntityDamageEvent` 实现（摔落、火焰、PvP、怪物、虚空等伤害都会被拦截）；`/kill` 这类直接执行指令造成的死亡不受事件拦截。
- 与其它「禁用饥饿/无敌」类插件可能功能重复，建议只保留一个。
- 若使用 Multiverse 的 `world` 别名（alias），请确认 MV 已正常加载；插件会通过反射安全读取，MV 缺失时自动降级。
- 建议先在测试服验证，再部署到正式服。

