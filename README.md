# EPUB Standardization Skill

通过调用 Calibre 官方命令行工具，全自动清理、格式化并升级 EPUB 电子书的元数据与排版。

## 核心功能

*   **格式升级**：自动剥离无用 CSS 并将 EPUB2 升级至 EPUB3。
*   **元数据注入**：基于网络抓取，精准写入 ISBN、标签及简介。
*   **真实评分**：支持写入 10 分制评分（解决 Calibre UI 星级折算问题）。
*   **多作者规范**：规避默认 `&&` 乱码，自动生成符合 W3C 标准的单 `&` 排序。
*   **终端防截断**：自动替换全角双引号，防止长简介导致的命令行截断。

## 环境依赖

*   必须安装 [Calibre](https://calibre-ebook.com/) 并将安装目录添加至系统环境变量 `PATH`。
*   确保终端可全局调用 `ebook-meta` 和 `ebook-polish`。

## 使用方法

将本仓库中的 `SKILL.md` 内容复制并导入至你的 AI Agent（如 Gemini、Cline、Cursor）的系统提示词或技能库中即可生效。

## 内部执行逻辑

1. 获取文件并去除冗余的盗版资源后缀。
2. 检索并补全准确的元数据。
3. 执行 `ebook-polish` 进行排版清洗与格式升级。
4. 执行 `ebook-meta` 写入核心元数据。
5. 二次执行 `ebook-meta` 覆盖修正多作者的 `author-sort` 排序字段。
