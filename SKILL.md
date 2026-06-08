---
name: epub-metadata-standardization
description: 通过提取 EPUB 内部 HTML 版权页的原始数据并辅以网络检索数据，对 EPUB 元数据进行标准化清洗与完善。
---

# EPUB 元数据标准化

## 概述
本技能定义了清理和标准化 EPUB 元数据的标准操作程序。其核心逻辑是：严格依赖 EPUB 内部 HTML 版权页作为出版方和 ISBN 的唯一真实数据源。提取 ISBN 后，通过网络检索（如豆瓣）获取其余缺失的元数据（评分、简介、标签等）进行互补填充。

## 适用场景
- 收到清理、格式化或标准化 EPUB 元数据的指令时。
- 导入的 EPUB 文件包含不准确的元数据、字段缺失或包含推广标签（如 z-library, 1lib）时。

## 核心工作流

### 特殊分支：极劣质结构的降维重构 (Pandoc)
- **触发条件**：如果发现 EPUB 毫无结构（例如：将整本书几十万字塞入一个单独的 HTML 文件，无 `.ncx` 目录树，或由低级爬虫生成的纯垃圾标签代码）。
- **执行逻辑**：放弃直接修改 HTML，转而调用 `run_command` 执行 Pandoc 进行降维重构。
  1. 将 EPUB 降维为纯文本：`pandoc 烂书.epub -t markdown -o temp.md`
  2. 使用 Python 脚本或正则在 `temp.md` 中识别“第 X 章”并自动加上 Markdown 的 `#` 标题语法。
  3. 重新编译为原生结构的全新电子书：`pandoc temp.md -o 全新重构书.epub --epub-chapter-level=1`
- **注意**：重构完成生成拥有完美原生目录结构的新 EPUB 后，再进入下述标准洗稿流程。

### 阶段 1：提取内部版权数据
1. **解包 EPUB**：使用 `zipfile` 模块将 EPUB 内容提取到临时目录。
2. **定位版权页**：遍历解压出的 `.html`、`.xhtml` 或 `.htm` 文件（优先检查 `part0000.xhtml` 或文件列表前序的页面）。搜索包含“版权信息”、“出版发行”、“版次”或“ISBN”的文本内容。
3. **提取真实数据**：解析 HTML 文本，提取以下精准字符串：
   - `ISBN`（13 位数字格式）
   - `Publisher`（出版方，如“北京联合出版公司”）
   - `Date`（出版年月，如“2017年8月”）

### 阶段 2：检索网络数据
1. **通过 ISBN 查询**：使用 `search_web` 工具，以阶段 1 提取的 ISBN 作为主要搜索关键字（例如：`search_web("<ISBN> 豆瓣读书")`）。
2. **提取补充数据**：从搜索结果中提取以下字段：
   - **简介 (`<dc:description>`)**：官方内容简介，去除所有推广链接。
   - **标签 (`<dc:subject>`)**：提取 3-5 个标准的分类标签（如心理学、社会学）。
   - **评分 (`calibre:rating`)**：获取网络 10 分制数字评分（**严禁除以2**！无论官方文档怎么写，实测底层直接写入该数字，例如豆瓣 8.3 就传 8.3）。

### 阶段 3：排版洗稿与底层升级 (Calibre ebook-polish)
**【警告】**：必须**先执行排版洗稿，后执行元数据注入**！因为 `ebook-polish` 会彻底重构 EPUB 底层，这会触发 Calibre 的内部重算机制，将所有人工强制干预的元数据（例如多作者排序）全部格式化归零！

优先利用本地环境已有的专业命令行工具（CLI）进行排版大清洗，取代脆弱的正则匹配：
1. **清理冗余样式与重构**：使用 `run_command` 调用 Calibre 底层工具 `ebook-polish` 对 EPUB 进行一键格式标准化。
   - `ebook-polish --remove-unused-css <文件名>`：全自动剥离所有无用垃圾代码和未引用的 CSS 样式。
   - `ebook-polish --upgrade-book <文件名>`：将老旧的底层格式强制升级为现代标准。
2. **无障碍阅读 (A11y) 声明注入**：
   - 必须遍历所有被 `ebook-polish` 重构过的 `.xhtml`/`.html` 文件。
   - 强制在每一个文件的 `<html>` 根标签中注入 `lang="zh-CN" xml:lang="zh-CN"` 属性，以满足国际视障无障碍辅助阅读标准。
3. **精准安全的 CSS 注入**：
   - 禁止使用暴力且危险的全局 `p { text-indent: 2em; }` 污染排版。
   - 必须通过解析 HTML 找到真正的“正文段落类名”（如 `.calibre1`, `.body-text`），并在清洗后的 CSS 文件中仅对该特定 Class 注入：`text-indent: 2em; text-align: justify; margin-bottom: 0;`。
   - 确保图片安全响应式：`img { max-width: 100%; height: auto; }`。

### 阶段 4：元数据安全注入 (Calibre ebook-meta 终端工具)
在排版重构完毕后，强制要求通过 `run_command` 调用 Calibre 成熟的官方 CLI 工具 `ebook-meta` 进行安全注入，这样数据才会被永久封存在最终的 OPF 中。
**执行命令示例**：
```bash
ebook-meta <文件名.epub> -t "<书名>" -a "<作者A & 作者B>" --author-sort="<作者A>" -p "<出版社>" --isbn "<13位ISBN码>" -d "<YYYY-MM-DD>" --tags "<标签1,标签2>" -c "<纯净简介>" -l zh-CN -r <10分制评分>
```
**技术规范防呆注意（极易踩坑）**：
- **多作者排序**：`-a` 参数传入 `A & B` 后，系统会生成恶心的 `A && B` 排序。**绝对不要**在同一条命令中同时使用 `-a` 和 `--author-sort`（会导致相互覆盖失败）。**必须分两步执行**：
  1. 先执行完整参数包含 `-a "A & B"` 的注入命令。
  2. 随后再单独执行一条 `ebook-meta <文件名> --author-sort="<作者A> & <作者B>"`，强行接管底层的排序修正。
- **评分星级**：无视官方文档，`-r` 参数**必须直接传入 10 分制评分**（如 `-r 8.3`），它才能在 UI 显示为正确的 4 星半。
- **终端截断保护**：简介内容（`-c`）中如果包含中文全角双引号 `“ ”`，极易导致 Windows 命令行截断报错。**必须将简介中所有的全角双引号替换为单引号 `' '` 或去除**。
- **标识符**：工具底层会自动将传入的纯数字 ISBN 映射为完美的 EPUB3 `urn:isbn:` 协议格式。

   - 确保图片安全响应式：`img { max-width: 100%; height: auto; }`。

### 阶段 5：重新打包与官方质检 (EpubCheck)
1. **规范封包**：创建新的 `.epub` 时，`mimetype` 必须是首个被添加的文件，强制设定为 `compress_type=zipfile.ZIP_STORED`，其余文件采用 `ZIP_DEFLATED`。
2. **终极质检 (EpubCheck)**：打包完成后，若环境中配置了 `epubcheck.jar`，AI 需通过 `run_command` 运行 `java -jar epubcheck.jar <最终文件名>.epub`。
3. **排错循环**：读取 EpubCheck 输出的 W3C 标准体检报告。如果存在严重的结构错误或 HTML 闭合错误，必须使用 BeautifulSoup 针对报错行数进行精确的微调修复，直至验证通过。
4. **清理**：删除所有临时工作目录。
