# Picture Book Writer Skill (`picture-book-writer`)

专为 **“AI 父母语音克隆 + 有声绘本”** 交付场景打造的儿童绘本分镜脚本创作 Skill。

## 目录结构
```
picture-book-writer/
├── SKILL.md                          # 技能主入口与核心规范
├── README.md                         # 本说明文件
└── references/
    └── ip-overrides-example.md       # IP 角色与世界观覆盖文件模板
```

## 功能特点
1. **交互式提问引导**：启动后对年龄、时长（20/30分钟）、主题、角色进行结构化确认。
2. **分龄定制**：严格区分 2-3 岁感官重复与 4-6 岁逻辑探索两种叙事模型。
3. **TTS 纯净排版**：【绘本文字】段落零 Markdown 干扰，天然适合声音克隆语音合成。
4. **用画面呈现 (Show, Don't Tell)**：内置表达转换表，拒绝成人黑话与说明文腔调。
5. **IP 覆盖机制**：支持配置 `references/ip-overrides.md` 复用固定角色造型与道具。
