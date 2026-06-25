---
aliases: [capture-{{date}}-{{slug}}]
created: {{date}}
source:
  - tool: {{tool}}
    session: {{sessionId}}
    file: {{sourceFile}}
    lines: [{{lineStart}}, {{lineEnd}}]
capture_type: {{decision|fix|architecture|method}}
status: inbox
promote_target: {{建议去向房间，如 40.notes/permanent}}
---

# {{标题：一句话点出这个认知单元}}

{{正文：认知浓缩，第三人称陈述。保留关键决策的理由。
不要写"用户说…我说…"，直接陈述结论。
长度适中，一段话 ~ 一个认知点。}}

<!-- 捕获自 {{tool}} 会话 {{sessionId}}，行 {{lineStart}}-{{lineEnd}}。
     详细原文见 {{sourceFile}}。本文件是认知浓缩，非原文。 -->
