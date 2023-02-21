# 上級者チーム開発 - Git を使った開発の流れ

## 概要

チーム開発で Git を活用する上で効率よく作業を進められるよう、git-flow のルールをベースとしてリポジトリを管理し、主な作業の流れとしては:
1. Issue を使って課題のタスク分けを行う
2. Issue に対してその機能追加用の ```feature``` ブランチを ```develop``` から切る
3. 機能追加完了後に ```develop``` へプルリクを作成する
4. 最低一人のメンバーにプルリクに対するレビューを行ってもらい、その後 ```develop``` へマージする

---

## 1. git-flow とは？

git-flow とは Git におけるリポジトリの分岐モデルであり、ルールのことを指します。
それぞれのブランチを明確に定義し、複数人での開発時にそれぞれが好き勝手にブランチを作成し混乱することを防ぎます。

下図はその概念図です。

一般的に使用する各ブランチの定義は、

```main (master)```: リリース用のブランチであり、このブランチ上での作業は行わない

```develop```: 開発用のブランチであり、コードが安定し、リリースへの準備ができたら ```main``` へマージする

```feature```: 機能追加用のブランチであり、```develop``` から分岐し、```develop``` へマージする

---

## 2. 開発の流れ