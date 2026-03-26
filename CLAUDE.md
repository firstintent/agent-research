# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概述

AI Agent 行业调研报告库，纯 Markdown 文档，无可运行代码。

## 文件命名规范

`{主题}-{YYYYMMDD}.md`，主题需能反映报告核心内容，例如：`openai-agent-sdk-20260326.md`、`langchain-20260326.md`。

## 新报告结构

概述 → 技术架构 → 核心能力 → 市场数据 → 竞争格局 → 风险 → 总结，头部注明报告日期与数据截止日期。

## WebFetch 权限

常用数据源域名已预配置在 `.claude/settings.local.json`，新增域名在其 `permissions.allow` 中添加 `WebFetch(domain:xxx)`。
