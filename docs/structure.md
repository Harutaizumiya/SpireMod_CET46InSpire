# Project Structure

## Overview
这是一个面向 Slay the Spire 的 ModTheSpire/BaseMod 模组，核心目标是在出牌前插入词汇测验，并用测验得分影响攻击、格挡和部分附加效果。项目以 `CET46Initializer` 为启动入口，运行时通过 BaseMod 注册遗物、字符串、配置面板和自定义测验屏幕，通过 ModTheSpire 补丁接入原游戏事件、出牌流程、存档流程、字体与图片初始化。

## Modules
- 模组启动与注册：读取 `ModTheSpire.json` 配置，在 `CET46Initializer` 中注册遗物、本地化字符串、词库资源、配置面板、自定义屏幕和开发控制台命令。
- 游戏流程补丁：通过 `patches` 包拦截涅奥事件、Downfall 心脏事件、玩家出牌、原生行动、图片初始化、字体初始化和读档定位，把 CET 事件、测验前置出牌和兼容性修正接入原游戏。
- CET/JLPT 入口事件：`CallOfCETEvent` 和 `CallOfCETRoom` 在第 0 层引导玩家选择词库遗物，按已启用的书籍配置生成事件选项并发放对应遗物。
- 测验遗物与战斗效果：`QuizRelic` 及 `CETRelic`、`JLPTRelic` 维护答题状态、分数倍率、连胜计数、错题本和奖励药水，并在战斗开始提供 `PerfectAnsPower`。
- 测验动作与屏幕：`QuizAction`、`GeneralQuizAction`、`CorrectAction` 负责构造题目并打开 `QuizScreen`；`screens` 与 `ui` 组件负责题目展示、选项选择、手柄导航、核对答案和返回战斗。
- 词库配置与取题策略：`BookConfig` 管理书籍与词库枚举、词库容量和遗物权重；`BuildQuizDataRequest`、`IQuizDataStrategy` 负责按 CET/JLPT 规则生成题目与干扰项。
- Anki 学习调度模型：`com.stemlaur.anki.domain` 提供牌组、学习会话、卡片进度和内存仓储，供 `FSRSFactory` 为词库创建临时牌组并按学习进度选取下一题。
- 配置面板与资源管理：`ModConfigPanel` 管理显示、字体、快速模式、作答模式、答案数量、词库启用和词库权重；`ImageElements`、`CNFontHelper` 与资源目录提供图片、字体、本地化和词库 JSON。
- 错题与存档：`CorrectionNote` 记录本局错题权重和已移除项目；`SaveData` 将错题数据注入 Slay the Spire 存档并在读档后恢复到遗物。
- 测试与词库工具：`JLPTRelicTest` 使用本地 UIString 读取器遍历 JLPT 题库；`vocabulary` 目录中的脚本和 `.apkg` 文件支持词库来源处理。

## Data Flow
游戏加载模组时，ModTheSpire 读取 `ModTheSpire.json` 并调用 `CET46Initializer.initialize()`；初始化器建立书籍配置，BaseMod 加载本地化与词库 JSON，随后注册配置面板、遗物、自定义测验屏幕和 `quiz` 命令。开局第 0 层结束涅奥事件或 Downfall 心脏事件时，补丁切换到 `CallOfCETRoom`，事件根据已启用书籍发放 CET 或 JLPT 测验遗物。

玩家战斗中使用卡牌时，`AbstractPlayerPatch` 先暂停原出牌流程并把当前卡牌、目标和能量暂存到补丁静态字段；`QuizRelic` 选择配置权重中的词库，调用 `FSRSFactory` 从内存学习会话取题，再交给 CET 或 JLPT 策略生成 `QuizData` 并打开 `QuizScreen`。玩家答题后，屏幕把分数、完美作答和错题结果写回遗物；遗物恢复原出牌流程，并在真正使用卡牌前按分数倍率修改伤害、格挡和部分 magicNumber。错题可通过右键遗物进入 `CorrectAction` 回顾；存档时 `SaveData` 将 `CorrectionNote` 内容写入原游戏保存参数，读档后再还原。

## Next Steps
- 核对 `CallOfCETEvent` 中 CET 日期常量是否仍符合当前版本预期。
- 补充 `com.stemlaur.anki.domain` 代码来源、改动边界和许可证说明，避免后续维护时误判为主模组业务代码。
- 为错题存档恢复、Downfall 跳转和 CET/JLPT 题目生成补充最小验证清单。
