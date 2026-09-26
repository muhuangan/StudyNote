# StudyNote

个人中文学习笔记仓库(Obsidian vault,Syncthing 同步)。纯 Markdown,无构建/测试/lint 脚本,也无 CI。改动后的验证手段: prettier 格式化、链接可达性、GitHub/Obsidian 渲染检查。

## 结构

- `README.md` — 总目录,链接到各书索引。
- `notes/<书名>.md` — 8 个索引文件(高等数学、概率论、电磁场与波、复变函数、大学物理、数据结构、STM32、数电); `notes/` 下其余约 410 个 `.md` 是原子子笔记,`提问的智慧.md` 是单文件笔记。
- `resources/` — 图片,子笔记中以 `../resources/...` 引用。
- `templates/笔记模板.md` — Obsidian 模板,只含 frontmatter 骨架。
- `.obsidian/`、`.stfolder/`、`.stignore`、`.workbuddy/` 被 gitignore,其改动不入库。

## 笔记模板

`templates/笔记模板.md` 的完整内容:

```markdown
---
created_at: "{{date}}"
updated_at: "{{date}}"
tags: []
archived: false
---
```

每篇笔记的完整结构(frontmatter 之后依次是面包屑、一级标题、正文):

```markdown
---
created_at: "2026-09-26"
updated_at: "2026-09-26"
tags:
    - 数理基础
archived: false
---

[高等数学](./高等数学.md) / 5. 定积分 / 5.6 定积分应用 / 5.6.1 面积

# 面积
```

- frontmatter 固定四个字段: `created_at`、`updated_at`、`tags`、`archived`,日期为 `YYYY-MM-DD`,列表缩进 4 空格。
- 面包屑在 frontmatter 之后、`# 标题` 之前; 每段带完整层级序号: 根段 `[索引](./索引.md)`,其后依次为章段 `N. 章名`、节段 `N.M 名称`、叶段 `N.M.K 名称`(到笔记自身所在层级为止),末段与本文 `# 标题` 对应。
- 被两个索引共用的笔记写两行面包屑,每行是一条从对应索引出发的完整路径(如 `库仑定律.md` 同时属于《电磁场与波》和《大学物理》)。

## 索引与编号约定

索引行与子笔记面包屑必须成套维护: 新增、删除、移动或调整顺序时两边同时更新,序号一致。

- 索引结构: frontmatter → `[README](../README.md)` → `# 书名` → `## N. 章名` → 条目列表,无 `## 目录` 包裹层。
- **序号必须正确且连续无空缺**: 章号连续(允许从 0 起); 每章条目从 `N.1` 起依次编号; 子条目从 `N.M.1` 起依次编号。不留教材原书的编号空缺,序号只有一层的条目(如 `1.`)不存在,一律为 `N.M` / `N.M.K`。
- 条目行的序号写在链接外: `- 1.1 [标题](./文件.md)`,链接文本不含序号。
- 有子条目的分组行写纯文本组名,不带链接,如 `- 3.1 中值定理是什么`; 子条目缩进 4 空格。
- 不使用任何共用标记; 共用关系只体现在子笔记的两行面包屑和 `notes/大学物理.md` 的复用关系表里。

## 文风

- Markdown 风格为 GFM: 列表标记统一 `-`,表格用 GFM 表格语法。
- 中文正文用半角标点,不用全角标点; `,` `.` `:` `;` 后跟文字时加一个空格,行尾无文字时不加。
- `"` 与 `'` 前后紧贴文字,不加空格。
- `+ - * / =` 实际用于运算时前后各加一个空格,公式内外一致(`1 + 1 = 2`、`$f(x) = x^2$`); 作为名称或记号的组成部分时不加(`0-1分布`、`B+树`、`D/A 转换器`)。
- 行内公式 `$...$`,独立公式 `$$...$$`,写法必须能在 GitHub 正常渲染。
- 不手动重排正文换行(保持原有软换行)。

## updated_at

对笔记内容做**实质性改变**时,更新该文件 frontmatter 的 `updated_at`。纯结构性调整(改序号、移动面包屑、增删链接、格式化)不更新。

## 格式化

- 仓库使用 prettier,配置为根目录 `.prettierrc`(`tabWidth: 4`,与 4 空格列表缩进对应)。
- **每个文件写完都要用 prettierd 格式化一遍**,并确认再次运行输出 0 diff(幂等)。本机 prettierd 由 nvim mason 提供,不在 PATH 中,且不支持 `--write`,用法为 stdin 管道:

```bash
cat 文件 | ~/.local/share/nvim/mason/bin/prettierd 文件 > tmp && [ -s tmp ] && mv tmp 文件
```

## 提交

Conventional Commits + 中文描述,scope 用文件名,例如: `docs(泰勒公式.md): 调整公式以适配 GitHub 渲染`。

## Shell 注意

- 文件名含中文及特殊字符(`!`、`Σ-Δ`、`+`),shell 中一律加引号; 批量处理用 `find -print0` / `git -c core.quotepath=off ls-files -z`。
- 本机 shell 为 zsh,字符串变量不会自动单词拆分,循环列表要用数组。
- `.gitattributes` 强制所有文本与 `*.md` 为 LF。
