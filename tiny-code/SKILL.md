---
name: tiny-code
description: Design, implement, refactor, repair, test, or review software so external dependencies remain replaceable behind small boundaries and solutions stay minimal. Use for coding, test-design, and installation or deployment-script tasks where Codex should understand the real flow, prefer no implementation, reuse, standard features, or installed dependencies before new code, isolate plugins, SDKs, and external APIs, and add only non-redundant tests for required important or hard-to-observe behavior and external contracts.
---

# Tiny Code

変更理由が異なる処理を疎結合にし、変更の影響を局所化することを最優先にする。その条件を満たす範囲で作らずに済ませ、必要な変更可能性を少ない構造で保つ。

## 参照手順

作業内容に応じて次のファイルを読む。

- 設計または構造の判断では [references/design.md](references/design.md) を読む。
- テストの設計、追加、変更または確認では [references/testing.md](references/testing.md) を読む。

実装、修正、リファクタリングまたは導入・配備処理の変更では、コードやテストを変更する前に設計、テストの順で両方を読む。レビューでは設計を読み、テストや検証方法も対象に含む場合はテストも読む。

## 応答

変更結果を先に示す。設けた境界、意図して残した冗長性、確認を閉じた範囲のうち、判断に必要な内容だけを説明する。確認範囲を広げた場合は根拠も示す。

少ない行数、少ない結合で変更可能性を高く保つ
