# 契约与门禁

## 输入

调用方提交纯内容 `CreationBriefV1`，再由 `WriterRequestV1` 指定工作模式、摘要和审批状态。只接受受控文字，不接受儿童照片。`characters` 表示全册常驻角色，最多 3 个；临时背景人物不得成为新的连续性锚点。

`briefDigest` 是持久化 `CreationBriefV1` 文件原始 UTF-8 字节的 SHA-256，格式为 `sha256:<64 位小写十六进制>`。摘要存放在 `WriterRequestV1`，不得写回被计算摘要的简报本体。请求中内嵌的 `brief` 必须与该持久化简报内容一致。

`must`、`prefer`、`avoid` 的优先级依次降低，但任何客户约束都不能覆盖儿童安全、原创性、Schema 和审批门禁。

## 模式与审批

- `outline`：生成完整分页纲要。`approvedOutlineDigest` 和 `sourceSpecDigest` 必须为 `null`。
- `finalize`：必须携带调用方写入的 `approvedOutlineDigest` 和对应纲要的 `sourceSpecDigest`。
- `revise`：必须携带 `approvedOutlineDigest`、被修订版本的 `sourceSpecDigest` 和至少一条 `revisionInstructions`。

调用方必须同时提供与 `sourceSpecDigest` 对应的既有 `StorybookSpecV1`。`finalize` 中 `sourceSpecDigest` 必须等于已批准纲要的实际文件摘要。Skill 只能读取审批摘要，不能自行创建、猜测或宣布客户审批。输出的 `briefDigest`、`sourceSpecDigest` 必须与请求及父版本一致。

## 时长与分页

两个年龄段均开放 20、30、45 分钟。Skill 自主选择 12–64 页，并在 `durationEstimate` 中记录估算版本、估算时长、依据和置信度。

当前阶段没有真实音频，因此 `estimatedMinutes` 只是文字体量估算，不是播放时长承诺。不得使用固定总字数冒充已校准结果。

若无法在 64 页内兼顾目标时长和单页可读性，返回 `capacity_conflict`。不得静默改变客户选择。

## 正文与字幕分段

- `outline`：每页提供 `pageSummary` 和 `visualBrief`；`narration` 为 `null`。
- `finalize`、`revise`：每页提供完整 `narration` 和至少一个 `captionSegments`。
- `captionSegments` 按顺序直接拼接后必须与 `narration` 完全一致，包括标点。
- 不输出时间码、音频标记、发音提示或图片内文字。

## 语义分镜

每页 `visualBrief` 必须与正文同一事件，只包含一个主要动作和一个情绪焦点，并记录：

- 出场角色 ID
- 场景、动作、情绪和必要道具
- 构图焦点和连续性引用
- 安全提示
- 样页风险等级及原因

语义分镜不包含最终图片提示词。下游提示词编译器负责加入画风、参考图、尺寸、边缘安全区、字幕留白和禁止图片内文字等固定条件。

全册使用顶层 `continuityAnchors` 定义角色、场景、道具和世界观锚点。每页 `continuityRefs` 只能引用该表中的 ID。

全册必须标出 2–4 个 `sampleRisk=high` 的代表性样页候选。`high` 只用于相对全册最容易发生角色比例、场景关系、道具数量或连续性漂移的页面，并必须填写 `sampleReasons`；其余页面按相对风险降为 `medium` 或 `low`。

## 修订与失效提示

`revise` 只列出实际变化的 `changedPageNumbers`。调用方据此处理下游失效：

- 正文变化：字幕文字及未来音频轨失效。
- 单页语义分镜变化：该页图片失效。
- 角色锚点或全册画风变化：全册图片失效。

Skill 不删除或覆盖既有资产，只返回新 revision。

## 输出一致性

成功输出必须满足：

- 页码从 1 连续递增。
- `pageCount` 等于 `pages.length`。
- 每个 `actId`、`characterIds` 和 `continuityRefs` 都引用已定义对象。
- `finalize` 和 `revise` 的每页正文非空。
- 封面语义分镜不安排可见标题；标题由下游确定性排版。
