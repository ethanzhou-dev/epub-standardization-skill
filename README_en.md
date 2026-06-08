# EPUB Standardization Skill

English | [中文版](README.md)

An automated AI skill that utilizes Calibre's official CLI tools to clean, format, and upgrade EPUB ebook metadata and styling.

## Core Features

*   **Format Upgrading**: Automatically strips unused CSS and upgrades EPUB2 to EPUB3.
*   **Metadata Injection**: Accurately writes ISBNs, tags, and descriptions based on web scraping.
*   **True Ratings**: Supports 10-point scale ratings (bypassing Calibre's native UI star conversion limits).
*   **Author Sorting Standard**: Prevents Calibre's default `&&` artifacts, generating W3C-compliant single `&` sorting.
*   **Terminal Anti-Truncation**: Automatically replaces full-width quotes to prevent command-line truncation during long description injection.

## Prerequisites

*   Install [Calibre](https://calibre-ebook.com/) and add its installation directory to your system's `PATH`.
*   Ensure that the `ebook-meta` and `ebook-polish` commands are globally available in your terminal.

## Usage

Simply copy the contents of `SKILL.md` and import it into your AI Agent's (e.g., Gemini, Cline, Cursor) system prompt or skills directory.

## Internal Execution Logic

1. Retrieves target files and strips pirate/redundant filename suffixes.
2. Scrapes and completes accurate metadata.
3. Executes `ebook-polish` for styling cleanup and format upgrading.
4. Executes `ebook-meta` to inject core metadata.
5. Executes `ebook-meta` a second time to explicitly override and fix multi-author `author-sort` fields.
