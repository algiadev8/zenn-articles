---
title: "Obsidianサーバーを建てよう"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# Obsidianサーバーを建てよう
[docker-obsidian](https://github.com/linuxserver/docker-obsidian)を使ってObsidianが使えばObsidianを使えるサーバーが1コマンドで建てれるよという紹介をします．

## きっかけ

## インストール方法


## 動作の仕組み

## メリット
- ブラウザでObsidianが使える．
- (Windowsだと)ネイティブアプリより動作が軽い
	- 私の環境だけかもしれませんが...
## デメリット
残念ながら以下のような欠点があります
- 日本語入力がちょっと不安定
	- 使えないことはない．たまに文字が飛ぶ/IMEの位置がおかしい
	- Selkies起因だと思われる．Linux特有の日本語の不安定さなので我慢
- 画面の綺麗さはLinux仕様
	- WindowsとかMacには劣る

## 今後の展望
- リモートサーバーにObsidianコンテナーを立てる．
	- リバースプロキシの設定を追加する必要あり．
## 参考リンク