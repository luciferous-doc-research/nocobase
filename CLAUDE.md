# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

このリポジトリは [nocobase](https://www.nocobase.com/) のドキュメントを調査するためのリポジトリです。  

[nocobaseのリポジトリ](https://github.com/nocobase/nocobase) をsubmoduleとして `repository` のディレクトリで参照できるようにしている。  
これはリポジトリにドキュメントが含まれているためである。  

ドキュメントの調査は [luciferous-plugins](https://github.com/sinofseven/luciferous-plugins-for-claude-code) の `report-kit` というpluginを使う (詳しくはreportワークフロー)。

## セットアップ

クローン直後は `repository/` サブモジュールが空のため、以下を実行して初期化する:

```bash
git submodule update --init --recursive
```

## report-kit

ドキュメントの調査においてはカンバンを使って"知りたいこと", "目的"を定義して調査するために `report-kit` というpluginを使います。

このプラグインは二つのスキルがあり、次の用途に使います。

1. `/add-report`
   - 調査したいことを記載するカンバン用のディレクトリとマークダウンファイルを作成する
1. `/report`
   - カンバン用のマークダウンファイルを読み取って実際に調査を行い、調査結果と詳細なログを残す
   - 実際の調査のワークフローについてはスキルに添付しているマークダウンファイルを参照して欲しい


調査タスクファイルと調査結果は `./reports/` ディレクトリに出力される。

## ドキュメント調査の制限

nocobaseのドキュメントは `./repository/docs/docs` に多言語化されて配置されている。  
利用可能な言語ディレクトリ: `cn`, `de`, `en`, `es`, `fr`, `ja`, `ko`, `pt`, `ru`

ドキュメント調査の起点は `./repository/docs/docs/en` というディレクトリに配置されている英語ドキュメントを元に調査してください。
ただし、調査結果やログにドキュメントのファイルパスを記載する場合には `./repository/docs/docs/ja` という日本語ドキュメントのパスも併記してください。