# 开发与兼容性验证

## 当前适配目标

正式服 12.1.0（Midnight），TOC Interface 为 `120100`。
版本依据为 2026-09-13 获取的 [Masque 上游源码](https://github.com/SFX-WoW/Masque/tree/3b1ae9a74bf57452ffe0e0c08c4a62b5d6eebd04)：

- `Masque.toc` 和 `CHANGELOG.md` 声明正式服 `120100` / 12.1.0。
- `Skins/Skins.lua` 仍兼容 `Masque_Version`、`Border.Enchant` 和原有 `AddSkin` 调用。
- `Core/Regions/Mask.lua` 仍支持字符串形式的 `Icon.Mask` 贴图路径。

本插件只注册静态皮肤，并使用 `C_AddOns.GetAddOnMetadata` 读取版本；
不读取战斗状态、光环或秘密值。保留旧客户端元数据回退和现有皮肤定义。
`Masque_Version` 是皮肤使用的 Masque API 版本，不等同于 WoW Interface，
不要随客户端版本机械递增。怀旧服声明保持原值，本次不承诺其当前版本兼容性。

## 分支和发布

`main` 为发布主线，`develop` 为集成分支。任务从最新 `develop` 建立独立分支和工作树，
本机验证后通过 PR 合并到 `develop`，完成集成和游戏内验证后再通过 PR 合并到 `main`。
现有正式版和预发布工作流由版本标签触发；发布前必须完成主线检查并取得发布授权。
本项目未配置专用远程游戏测试环境，游戏客户端验证需要可运行正式服的环境。

## 本机检查

从仓库根目录运行 `luacheck . -q` 和 `luac5.1 -p Skins.lua`。
皮肤注册验证应加载上述 Masque 版本的 `Skins/Defaults.lua`、`Skins/Regions.lua`、
`Skins/Skins.lua`，提供元数据 API 和 LibStub 桩，再加载本插件。
检查四个皮肤名称、遮罩与冷却贴图路径、旧 Border 字段的映射，并确认所有引用贴图存在。
分别验证正式服元数据 API、旧元数据 API、缺失 LibStub 和缺失 Masque 时的加载行为。
执行 `git diff --check`，检查 TOC 所列文件和安装包目录结构。

2026-09-13 本机已通过 Lua 5.1.5 语法检查、Luacheck 1.2.0（零警告、零错误），
以及上述四种加载场景的注册测试。四套皮肤的自有贴图引用均存在，
当前 Masque 的旧字段映射符合预期。尚未执行游戏内验收。

## 游戏内验收（必须人工执行）

1. 安装正式服当前版 Masque、支持 Masque 的按钮/光环插件和 `Masque_Pointy`。
2. 使用 `/dump GetBuildInfo()` 确认客户端 Interface 为 `120100`；关闭“加载过期插件”后确认正常加载。
3. 开启 `/console scriptErrors 1`，执行 `/reload`，确认无 Lua 错误。
4. 在 `/msq` 中逐个选择四套 Pointy 皮肤，检查图标裁切、边框、层数、持续时间和冷却遮罩。
5. 进入和退出战斗，检查增益/减益刷新、物品边框、鼠标高亮和重载后配置恢复。

静态和桩测试不覆盖游戏渲染、战斗保护或第三方插件实际集成；游戏内检查通过前不得标记已实机验证。
