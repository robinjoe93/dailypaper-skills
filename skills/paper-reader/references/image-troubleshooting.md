# 图片获取排错指南

## ar5iv 图片 URL 的坑

ar5iv 的 asset 编号（x1.png, x2.png...）**不一定对应论文的 Figure 编号**！

常见问题：
- 小 icon/箭头/符号图片也占编号，容易下载到错误图片
- 同一 Figure 的子图（a/b/c）可能用不连续的编号
- 某些 Figure 是由多个小图拼成的

**解决方法**:
1. 用 WebFetch 获取页面时，提取每个 Figure 标题和对应的 img src
2. **不要下载图片**，直接使用外链 URL

## 图片获取策略（只用外链）

**重要原则**: 所有图片使用在线外链 URL，不下载到本地，避免 git history 中出现图片文件。

### 来源 A: arXiv HTML（首选）
- WebFetch `https://arxiv.org/html/{arxiv_id}` → 提取 `<figure>` 的 img src
- 先统计论文 Figure 总数，确保提取完整

### 来源 B: 项目主页（补充）
- 从论文摘要/HTML 中查找项目主页（关键词：`project page`、`github.io`、`our website`）
- WebFetch 项目主页，提取展示图片（teaser / demo 图）
- 适合获取 arXiv HTML 中缺失的方法概览图

## 图片 URL 规范化（防止路径重复）

WebFetch 返回的图片路径可能是**相对路径**（如 `2603.05312v1/x1.png`），也可能是**已解析的绝对路径**。
拼接 URL 时极易出现路径重复 bug（如 `.../2603.05312v1/2603.05312v1/x1.png`）。

**铁律**: 写入笔记前，必须对每个图片 URL 执行以下检查：

1. 如果 URL 已经是 `https://arxiv.org/html/...` 的完整形式，直接使用，不要再拼接
2. 如果是相对路径，**只用** `https://arxiv.org/html/` 作为 base，不要再加 `{arxiv_id}/`
   - 因为相对路径通常已经包含 `{arxiv_id}/`（如 `2603.05312v1/x1.png`）
3. **最终验证**: 检查 URL 中是否存在连续两段相同的 arxiv_id（如 `2603.05312v1/2603.05312v1/`），如果有，删除重复段

示例：
```
✗ https://arxiv.org/html/2603.05312v1/2603.05312v1/x1.png  ← 重复了
✓ https://arxiv.org/html/2603.05312v1/x1.png               ← 正确
```

## 笔记中的图片引用格式

**只用外链**（禁止本地下载）:
```markdown
![Figure 1](https://arxiv.org/html/xxxx/x1.png)
![Figure 2](https://project-page.io/images/teaser.png)
```

**禁止格式**:
```markdown
![[{方法名}_fig1_overview.png]]  ← 不允许本地图片
```

## frontmatter 中记录图片来源

```yaml
---
image_source: online  # 始终为 online，不使用 mixed 或 local
arxiv_id: "2501.12345"
---
```