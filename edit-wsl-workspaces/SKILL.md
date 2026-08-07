---
name: edit-wsl-workspaces
description: Safely inspect and edit repositories stored inside WSL from a Windows-hosted Codex session. Use when Codex works with a path beginning with \\wsl.localhost\ or \\wsl$\, Windows tools fail with error 267 or an invalid-directory error on a UNC working directory, or repository operations must run through wsl.exe while preserving existing Git changes.
---

# WSLワークスペース編集

WSLのUNCパスをWindows側の `workdir` へ渡さない。ディストリビューション名とLinuxパスへ変換し、`wsl.exe` 経由で操作する。

## パスを変換する

次の形式をWSLワークスペースとして扱う。

```text
\\wsl.localhost\<distribution>\<linux-path>
\\wsl$\<distribution>\<linux-path>
```

例えば `\\wsl.localhost\NixOS\home\nixos\project` は次のように変換する。

```text
distribution: NixOS
Linux path: /home/nixos/project
```

Windows側のシェルには、利用が許可されたWindowsネイティブの作業ディレクトリを指定する。

## WSL経由で操作する

読み取り、Git操作、試験は次の形式で実行する。シェル構文が必要な場合だけ `bash -lc` を使う。

```powershell
wsl.exe -d <distribution> --cd <linux-path> <command>
wsl.exe -d <distribution> --cd <linux-path> bash -lc '<command>'
```

最初に現在地とGit状態を確認する。

```bash
pwd
git status --short --branch
git diff -- <target-files>
```

既存の未コミット変更は利用者の作業として保持する。`git reset --hard`、`git checkout --`、無関係なファイルの復元を行わない。

## ファイルを編集する

まず通常の `apply_patch` を試す。UNCパスでエラー267または「ディレクトリ名が無効です」が発生した場合は、同じ指定を繰り返さない。

組み込み編集がUNCパスを扱えない場合は、次の手順でパッチを適用する。

1. Windowsネイティブの書き込み可能な一時領域へ、`apply_patch` で統一差分を作成する。
2. パッチ内のパスをリポジトリ相対の `a/<path>` と `b/<path>` にする。
3. Base64でパッチのバイト列を保持し、`git apply --check` と適用を別々の呼び出しで実行する。
4. 適用後に差分と対象ファイルを確認する。

PowerShellの文字列パイプでパッチを直接渡さない。改行や文字コードが変化する場合があるため、検証時は次のように渡す。

```powershell
$patchBase64 = [Convert]::ToBase64String(
  [IO.File]::ReadAllBytes('<windows-patch-path>')
)
wsl.exe -d <distribution> --cd <linux-path> bash -lc `
  "printf '%s' '$patchBase64' | base64 -d | git apply --check -"
```

検証に成功した後、別の呼び出しでBase64を作り直して適用する。

```powershell
$patchBase64 = [Convert]::ToBase64String(
  [IO.File]::ReadAllBytes('<windows-patch-path>')
)
wsl.exe -d <distribution> --cd <linux-path> bash -lc `
  "printf '%s' '$patchBase64' | base64 -d | git apply -"
```

Base64文字列がコマンド長の上限へ近づく場合は、パッチを分割するか、明示的に許可された共有一時パスを使う。

パッチが適用できない場合は、対象を行番号付きで読み直す。

```bash
nl -ba <target-file> | sed -n '<start>,<end>p'
```

現在の内容に合わせてパッチを作り直す。`--unidiff-zero` は通常の文脈付きパッチを作れない場合だけ使い、先に `git apply --check --unidiff-zero` を実行する。

## エラーを分類する

- エラー267または「ディレクトリ名が無効です」: Windows側のUNC作業ディレクトリ処理を疑う。
- `Access denied` または `Permission denied`: ファイル権限やサンドボックス権限を確認する。
- `corrupt patch`: 差分の形式、行数情報、文字コードを確認する。
- `patch does not apply`: 対象ファイルの現在の内容とパッチの前提を確認する。

シェル起動がサンドボックス制限で失敗した場合は、実行環境の承認手順に従う。リポジトリの権限問題とは断定しない。

## 検証する

変更後は最低限、次を実行する。

```bash
git diff --check
git diff -- <target-files>
git status --short
```

対象言語の構文検査と関連試験も実行する。例えばシェルスクリプトでは `sh -n <script>` と `bash -n <script>` を使う。

一時パッチは保存先を確認してから削除する。利用者へ変更ファイル、検証結果、未実行の確認事項を報告する。
