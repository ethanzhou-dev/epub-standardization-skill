# 📚 EPUB Metadata Standardization (AI Skill)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

An industrial-grade, highly robust AI Skill designed to automate the process of cleaning, sanitizing, and upgrading EPUB ebook metadata using the Calibre CLI engine. 

这是一款工业级的 AI 技能提示词（Prompt/Skill），旨在驱使 AI 代理（如 Gemini, KI Agent 等）在无需人工干预的情况下，全自动清洗、补全并重构本地 EPUB 电子书的元数据与底层代码。

## ✨ Core Features (核心特性)

本技能不仅仅是简单的数据注入，它逆向攻克了 Calibre 底层 CLI 工具的多个“隐蔽天坑”：

- **🛡️ 纯净度保证**：强制弃用脆弱的正则和 Python 文本替换，全盘接管 Calibre 的 C++ 底层命令行接口（`ebook-meta` & `ebook-polish`）。
- **👥 W3C 多作者规范**：彻底修复多作者排序时 Calibre 自动生成的丑陋 `&&` 乱码，强制锁定单 `&` 的绝对整洁排序。
- **⭐ 满血 10 分制评分**：突破官方参数限制，精准写入网络（如豆瓣）抓取的真实 10 分制评分（绝不降级砍半）。
- **✂️ 防截断机制**：为 Windows 终端特制的字符串逃逸机制，杜绝含有中文全角双引号的百字简介在命令行中被强行腰斩。
- **🚀 自动 EPUB3 跃迁**：自动肃清多余的、冗余的“死代码”CSS样式，并将老旧的 EPUB2 架构无损跃迁至现代 EPUB3 标准。

## 🛠️ Prerequisites (前置依赖)

由于本技能完全依赖专业的工业级软件，在使用前，你的系统环境中必须存在以下依赖：

1. **[Calibre](https://calibre-ebook.com/)**：必须安装并将其所在目录添加到系统的环境变量 `PATH` 中。
2. **终端支持**：确保能够全局调用 `ebook-meta` 和 `ebook-polish` 命令。

## 🚀 How to Install (安装指南)

1. 将本仓库克隆或下载到本地。
2. 找到本仓库核心文件 `SKILL.md`。
3. 将 `SKILL.md` 的内容添加到你所使用的 AI Agent（例如 Gemini 桌面版、Cline、Cursor 等）的自定义系统提示词（System Prompt）或技能库（Skills Directory）中。

## 💻 Workflow Overview (内部执行流览)

当 AI 载入本技能后，对你的书籍会执行以下标准化指令：
1. **重命名清洗**：自动剔除诸如 `(z-library)` 等盗版资源站后缀。
2. **信息检索**：AI 在后台静默抓取 ISBN、准确简介、多标签及真实评分。
3. **排版重构**：优先执行 `ebook-polish` 引擎切除死代码，防止后续元数据重写。
4. **终极注入**：分两步执行 `ebook-meta`，先灌入全面数据，后强制接管修正作者排序。

## 📝 License (开源协议)

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
