# 影之刃零存档被覆盖后的恢复指南

《影之刃零》（Phantom Blade Zero）在 Steam 上会把存档同步到云端。共用账号、换设备、重开新档、或者云同步状态不同步时，**新的云存档可能直接顶掉你本地的旧进度**，看起来像"存档丢了"。

实际上，游戏自己会在本地保留三组历史备份。只要备份还在，绝大多数情况都能按时间回档，把进度找回来。

本仓库记录排查与恢复流程，并附一个可复用的 Claude Code skill：[`skill/save-rescue/SKILL.md`](skill/save-rescue/SKILL.md)。不包含任何游戏文件或第三方代码。

> 非官方文档，与游戏开发商、发行商无关。任何写存档的操作都有风险，操作前务必备份。

## 先判断：你是哪一种"丢档"

| 现象 | 大概率原因 | 去哪一步 |
|---|---|---|
| 进游戏只剩新档，旧进度不见了 | 云存档被另一台设备/共号覆盖 | 第 2 节 |
| 游戏提示云存档冲突 | 本地和云端版本不一致 | 第 5 节 |
| 游戏里连"继续游戏"都没有 | 当前存档损坏或被换成空档 | 第 2、6 节 |
| 存档目录整个不见了 | 换了 Steam 账号或 Windows 用户 | 先确认账号目录 |

## 1. 存档和备份在哪

Windows + Steam 版，存档根目录：

```
<Steam 库>\steamapps\common\PBZ\ProjectPBZ\Saved\SaveGames\Steam\<SteamID>\
```

`<SteamID>` 是 17 位数字，一般一个账号一个目录。目录里长这样：

```
PBZSaveGame.sav          当前正在用的存档
1RealTime\               按真实时间留下的备份（最容易救命）
2Hourly\                 按小时备份
3Daily\                  按天备份
```

备份文件名类似 `20261101-224501-344.sav`，格式是 `年月日-时分秒-毫秒`。**这是备份被写入的时间，不一定等于存档里的游戏进度时间**，两者可能差很多。

一个 `.sav` 是打包容器，里面有多个槽位：

- `AutoSave0` / `AutoSave1` / `AutoSave2`：自动存档
- `ManualSave1` 到 `ManualSave11`：手动存档
- `__Advanced__`、`__Achievements__`：全局数据和成就

槽位内容经过 Oodle 压缩，解压需要游戏自带的 Oodle DLL。想直接看内容，可以用 [pbz-save-manager](https://github.com/Orisland/pbz-save-manager) 的源码做离线解析和校验。

## 2. 第一时间止损

看到进度不对，**先别启动游戏，更别联网启动 Steam**。

1. 完全退出 Steam：主菜单 → 退出，并检查托盘和任务管理器里没有 `steam.exe`。
2. 把整个 `<SteamID>` 目录复制一份到别的盘，三组备份和当前档一起拷，不要只拷 `PBZSaveGame.sav`。
3. 确认复制完成后再动游戏目录。

云同步是双刃剑：它救回进度，也会把错误版本再覆盖回来。止损阶段的目标是**先让本地状态停下来**。

## 3. 找出真正的最新进度

不要只按文件名排序。最靠后的备份文件，可能是新开档的备份，而不是你的旧进度。

判断顺序：

1. 看备份目录的覆盖范围：`1RealTime` 最密，`2Hourly` 次之，`3Daily` 最粗。
2. 逐个检查候选文件，重点看**存档内时间、游戏时长、关卡名、当前主线任务**。
3. 只有解压后看到的内容才作准。文件名时间和文件修改时间都只是辅助。

如果不会解码，也可以间接判断：把候选文件按修改时间排列，结合你还记得的关卡/时长，挑出最像最后进度的一份。但恢复前最好还是做一次解码校验。

示例候选表（本次恢复，已脱敏）：

| 候选文件 | 文件时间 | 存档内时间 | 关卡 / 主线 | 游戏时长 | 判断 |
|---|---|---|---|---|---|
| `2Hourly/20261101-224501-344.sav` | 11-01 22:45 | 11-01 22:45 | 磐镇 / `MainCh2-SiZhen` | 15h29m | 匹配，选它 |
| `20261103-172658-417.sav` | 11-03 17:26 | 11-03 17:25 | 西古 / 新档 | 3h06m | 最新，但不是目标进度 |

这张表的重点不是文件名，而是最后两列：**关卡/主线**和**游戏时长**。文件名最新的那份，往往属于覆盖进来的新档。

## 4. 回档

1. 确认 Steam 已完全退出。
2. 把选中的备份文件复制为当前存档：

   ```powershell
   Copy-Item "<备份文件>" "<Steam 库>\steamapps\common\PBZ\ProjectPBZ\Saved\SaveGames\Steam\<SteamID>\PBZSaveGame.sav" -Force
   ```

3. 不要改文件名，不要手动改时间戳。保持备份文件原样复制即可。
4. 用 MD5/SHA 校验两份文件一致：

   ```powershell
   Get-FileHash "<备份文件>" -Algorithm MD5
   Get-FileHash "...\PBZSaveGame.sav" -Algorithm MD5
   ```

## 5. 防止启动时又被云覆盖

回档完成后，云端还是那份错误存档。直接联网启动，有概率再次被覆盖。

推荐顺序：

1. 打开 Steam → 左上角 Steam 菜单 → **进入离线模式**。
2. 启动游戏 → 继续游戏 → 确认关卡、游戏时长、存档点都对。
3. 确认无误后退出游戏。
4. 回到 Steam，右键游戏 → 属性 → 通用 → 暂时关闭「Steam 云存档」。
5. 需要恢复云同步时，再打开云存档；如果弹出冲突，选**保留本地文件**，让本地这份上传。

如果游戏目录里的备份还在，也可以先不恢复云同步，把本地备份再留几天。

## 6. 恢复后怎么确认是对的

进游戏看三件事：关卡名、游戏时长、当前主线任务。这三项和备份解码结果一致，基本就对了。

解码校验可以用 pbz-save-manager 的源码：

```powershell
$env:PYTHONPATH = "<pbz-save-manager>\src"
python -c "from pathlib import Path; from pbz_save_manager.container import parse_save; from pbz_save_manager.codec import load_slot_json; from pbz_save_manager.oodle_host import get_host; ..."
```

具体调用方式看该仓库的 README。它只读本地文件，不会联网，也不会改游戏进程；但请自行审查源码后再运行。

## 7. 常见坑

- **只备份了 `PBZSaveGame.sav`**：不够。三组历史备份才是回档的原料。
- **文件名最新 = 进度最新**：不成立。新开档的备份也会是最新文件名。
- **回档后直接联网开游戏**：云端可能立刻把新档又推回来。
- **手动改文件时间戳来"骗过"同步**：没必要，而且容易把备份也弄乱。先断同步，再回档。
- **共号/共享账号**：确认另一边没有正在运行游戏，退出后再同步；否则会反复互相覆盖。
- **备份会被游戏清理**：三组备份都有保留上限，发现丢档后越早复制越好。

## 8. 预防

- 定期把整个 `<SteamID>` 目录复制到其他盘或网盘。
- 关掉这个游戏的 Steam 云存档，改用本地/手动备份；或者接受云同步但每次换设备前先确认状态。
- 共号时约定好谁在玩，避免两台设备同时推进度。
- 重要节点留一个手动存档，不要只依赖自动存档。

## 适用边界

- **实测环境**：Windows 11 + Steam 版《影之刃零》（2026-11）。其他游戏/平台沿用同一思路，但路径和备份机制需要自行确认。
- **适用**：本地存档、平台自带的轮转备份、可以复制的存档文件（Steam / Epic / GOG / 独立游戏）。
- **不适用**：纯服务器侧存档（在线游戏）、Xbox / PlayStation 云存档（容器不透明或加密）、有反作弊或账号绑定的存档。
- **不是自动化工具**：本仓库提供流程和 skill，所有写操作都需要你确认，恢复前必须整目录备份。
- **不保证一定找回**：备份越早复制，成功率越高；备份已被游戏清理时可能无解。

## 作为 Claude Code skill 使用

[`skill/save-rescue`](skill/save-rescue/SKILL.md) 是一个通用版技能，适用于任何平台的存档覆盖/丢失，不只《影之刃零》。它的核心约束是：先确认游戏已退出、恢复前整目录备份、不盲覆盖、恢复后断云同步校验。

触发示例：

> 我的存档被覆盖了：影之刃零，Steam，Windows。游戏和 Steam 都已经退出。原本进度是第二章磐镇，15 小时左右，死亡 60 多次。

然后 skill 会按 intake → 定位 → 证据 → 候选表 → 安全恢复 → 防再覆盖的顺序处理，而不是直接让你覆盖文件。

安装（个人技能目录）：

```powershell
# Windows
Copy-Item -Recurse ".\skill\save-rescue" "$env:USERPROFILE\.claude\skills\save-rescue"
```

```bash
# macOS / Linux
cp -R ./skill/save-rescue ~/.claude/skills/save-rescue
```

重新打开 Claude Code，然后说「我的存档被覆盖了，游戏是 XXX」，它会按 intake → 定位 → 证据 → 候选表 → 安全恢复 → 防再覆盖的顺序处理。

## 案例

一次真实的共用账号覆盖事故、时间线和恢复过程，见 [docs/case-2026-11-steam-cloud-overwrite.md](docs/case-2026-11-steam-cloud-overwrite.md)。

## English summary

A safety-first guide and a Claude Code skill for recovering game saves that were overwritten by a new save or a cloud sync conflict. Tested on Windows 11 with the Steam version of *Phantom Blade Zero*; the workflow applies to other local-save games, but save paths must be verified per game. It does not cover server-side saves, console cloud containers, or anti-cheat-protected data.

The skill lives in [`skill/save-rescue`](skill/save-rescue/SKILL.md). Copy it to `~/.claude/skills/save-rescue`, then ask: "My save was overwritten, the game is X, platform Y, and the game is closed." It gathers evidence, builds a candidate table, backs up the entire save folder, and only then restores.

## 相关

- [pbz-save-manager](https://github.com/Orisland/pbz-save-manager)：可回档、查看槽位、在备份后修改少量数值的本地工具。

## 许可

MIT，见 [LICENSE](LICENSE)。
