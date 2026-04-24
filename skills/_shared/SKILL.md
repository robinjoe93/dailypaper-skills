---
name: image-hosting
description: |
  图片托管规范。当 md 文件需要图片时，统一使用 GitHub raw URL。
  适用于所有 Obsidian Vault 中的 markdown 文件图片引用。
context: fork
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# 图片托管规范

## 核心规则

**所有 md 文件中的图片必须使用 GitHub raw URL，不使用本地图片文件。**

## 图片 URL 格式

```
https://raw.githubusercontent.com/{user}/{repo}/main/{path}/{filename}
```

当前仓库配置：
- User: `robinjoe93`
- Repo: `autoPaper`
- Path: `assets/images/`

完整 URL 模板：
```
https://raw.githubusercontent.com/robinjoe93/autoPaper/main/assets/images/{filename}
```

## Markdown 图片语法

标准格式：
```markdown
![{description}]({url})
```

示例：
```markdown
![Hyperagents_fig1_overview](https://raw.githubusercontent.com/robinjoe93/autoPaper/main/assets/images/Hyperagents_fig1_overview.png)
```

## Obsidian 格式转换

如果已有 Obsidian wikilink 格式图片：
```markdown
![[filename.png]]
![[filename.png|600]]
```

转换为标准 markdown URL：
```markdown
![filename](https://raw.githubusercontent.com/robinjoe93/autoPaper/main/assets/images/filename.png)
```

转换规则：
- 描述文本使用文件名（去掉扩展名）
- 忽略 Obsidian 的 `|size` 参数
- URL 使用完整 GitHub raw URL

## 图片上传流程

1. **存放位置**: 所有图片统一存放到 `assets/images/` 目录
2. **命名规范**: 使用 `{方法名}_{类型}_{编号}` 格式，如 `Hyperagents_fig1_overview.png`
3. **提交到 Git**: 图片文件必须提交到仓库
4. **URL 引用**: md 文件中只使用 GitHub raw URL

## 批量转换脚本

```python
import re
import os

base_url = "https://raw.githubusercontent.com/robinjoe93/autoPaper/main/assets/images/"

def replace_obsidian_images(content):
    pattern = r'!\[\[([^\|\]]+)(?:\|[^\]]+)?\]\]'

    def replacer(match):
        filename = match.group(1)
        name_without_ext = os.path.splitext(filename)[0]
        url = base_url + filename
        return f'![{name_without_ext}]({url})'

    return re.sub(pattern, replacer, content)
```

## YAML Frontmatter 设置

笔记文件的 `image_source` 字段设置：
- `online`: 图片全部来自在线源（arXiv HTML、项目主页等）
- `local`: 图片全部来自本地 `assets/images/`
- `mixed`: 混合使用在线和本地图片

使用 GitHub raw URL 时设置为 `online` 或 `mixed`。

## Git 历史清理

如果删除了大量本地图片，需清理 Git 历史：

```bash
# 使用 git-filter-repo 删除路径的历史
git-filter-repo --path '论文笔记/assets/' --invert-paths --force

# 重新添加 remote
git remote add origin git@github.com:robinjoe93/autoPaper.git

# 强制推送
git push --force origin main
```

清理后 `git clone` 不会下载已删除图片的历史记录。