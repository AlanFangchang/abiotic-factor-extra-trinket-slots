# 《非生物因素》额外挂饰槽

[English](README.md) · [实现原理](docs/ARCHITECTURE.zh-CN.md) · 作者：[AlanFangchang](https://github.com/AlanFangchang)

使用官方 Modkit 制作的挂饰槽扩展。这里保留最终可编辑资产，没有中途测试工程、生成脚本、Python 队列或其他开发工具。

## 实现原理概览

Mod 扩展玩家的原生装备库存，把额外挂饰保存在索引 13–16，然后将四个自定义槽位控件插入原版背包画布。状态组件读取已装备的密码盘，控制槽位解锁；服务器根据有效槽位协调 Buff 和温度抗性。

想了解完整流程，包括如何过滤原生装备变化事件以避免地面出现复制物品，请阅读[实现原理](docs/ARCHITECTURE.zh-CN.md)。

## 功能

原版两个槽位不变。装备 T3 / T4 / T5 / Gatekey 时，分别开放 1 / 2 / 3 / 4 个额外槽位，总有效槽位为 3 / 4 / 5 / 6。

打开背包时始终显示四个额外槽位，关闭背包时一起隐藏。锁定槽位变暗，内部挂饰保留、效果停止，可以取出，但不能重新放入。解锁槽位提供图标、空槽提示和挂饰放置高亮。

支持挂饰的装备 Buff，以及可累加的抗寒 `ColdResist` 和抗热 `HeatResist`。本快照已包含 **2026-10-31 的抗性修复**。

## 如何阅读与编辑

完整 Mod 源码位于 `Mods/TrinketSlotProbes2QpQ/`。其中 `.uasset` 保存了可编辑蓝图和默认值，不是仅能运行的烘焙产物。本项目没有单独的 `.uproject`，也没有必须编译的 C++ 源码。

需要官方 Abiotic Factor Modkit，通过其本地 Mod 管理流程打开或导入。保留 `TrinketSlotProbes2QpQ` 目录名、插件名和内部资源路径，避免引用失效。名字中的 Probe 是历史命名，为兼容已有引用而保留。

建议从以下文件开始阅读：

1. `Content/DataTables/DT_ClassManager/Class_TrinketSlotsBootstrap.uasset`：官方入口注册。
2. `Content/Blueprints/BP_TrinketSlots_Bootstrap.uasset`：为玩家安装运行组件。
3. `Content/Blueprints/BP_TrinketSlots_State.uasset`：等级、库存槽位、Buff 和温度抗性。
4. `Content/Blueprints/BP_TrinketSlots_UI.uasset`：把四格插入原生背包界面。
5. `Content/Blueprints/WBP_TrinketSlot_Runtime.uasset`：每一格的显示、提示和交互。

详细流程见[实现原理](docs/ARCHITECTURE.zh-CN.md)。引用的原版类、控件、贴图及字符串表由官方 Modkit 提供，未复制进本仓库。

## 打包与发布

发布副本中的 `.abfmod` 已将 `ModId` / `TestingModId` 清零，避免复用原作者的发布绑定。自行发布前，在官方编辑器中关联自己的 mod.io 项目。原开发工程和其发布绑定没有改动。显示名称为 **Extra Trinket Slots**；内部插件名仍保留，以维持蓝图引用。

游戏安装包需通过官方云端烘焙生成；不要把本仓库的编辑器资产直接放进游戏。本次源码整理没有上传或重新烘焙。插件描述符中的旧版本字段未用作新的公开版本号。

## 已知边界

- 联机玩家都需安装并启用 Mod。
- 真实物品使用原生装备库存的 **13–16 号索引（从 0 开始）**，数组至少扩展到 17 格。与使用相同索引的 Mod 或未来官方变更可能冲突，尚无自动分配或迁移机制。
- 卸载前请取出额外槽位内的全部挂饰；没有实现停用 Mod 后自动退还。
- 额外挂饰不会显示为人物身上的附挂模型；未实现 Shift 快速装入额外槽位。
- 本版处理装备 Buff、抗寒和抗热，未扩展直接护甲值及伤害类型减免。
- 本次整理不代表主机平台或独立服务器已完成实机验证。具体测试边界见[验证记录](docs/VALIDATION.zh-CN.md)。

原创 Mod 逻辑及说明以 MIT 许可证开放。原版游戏、引擎及第三方内容不属于该授权范围，详见 [LICENSE](LICENSE) 和[第三方说明](THIRD_PARTY_NOTICES.zh-CN.md)。
