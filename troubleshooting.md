
# Quarto + TinyTeX で日本語 PDF 出力トラブルシューティング記録

このドキュメントは、Quarto で日本語を含む PDF を生成する際に発生した問題とその解決プロセスをまとめたものです。特に、R コンソールで実施する作業とターミナル（cmd）で実施する作業を分けて記載しています。

---

## 1. 背景

- **目的**: Quarto を使って日本語を含む物理解説サイトを作成し、PDF 出力時に日本語が正しく表示されるようにする。
- **使用ツール**:
  - **Quarto**
  - **TinyTeX**（R 経由でインストール）
  - **R**（RStudio や R コンソール）
  - **LuaLaTeX / XeLaTeX**（PDF コンパイル用）
- **主なトラブル内容**:
  - PDF 出力時に日本語フォントが見つからない
  - `fontspec` で `IPAGothic`（または `IPAexGothic`）が見つからないというエラー
  - ロケールの問題により、一部エラーが発生

---

## 2. R コンソールで実施する作業

### 2.1 TinyTeX のインストール・リインストール
- **TinyTeX を初期インストールする**  
  R コンソールで実行：
  ```r
  install.packages("tinytex")
  tinytex::install_tinytex()
  ```
- **TinyTeX の再インストール（トラブル発生時）**  
  R コンソールで実行：
  ```r
  tinytex::reinstall_tinytex()
  ```

### 2.2 必要な LaTeX パッケージのインストール
- **IPAex や日本語関連パッケージのインストール**  
  R コンソールで実行：
  ```r
  tinytex::tlmgr_install(c("ipaex", "bxjscls", "luatexja"))
  ```
  ※ インストール中に `xfun` のバージョンエラーが出た場合は、`install.packages("xfun")` でアップデートしてください。

### 2.3 フォントキャッシュの更新
- **luaotfload のキャッシュ更新**  
  R コンソールで実行：
  ```r
  system("luaotfload-tool --update")
  system("luaotfload-tool --cache=erase")
  ```

### 2.4 PDF のフォント設定確認
- **Quarto の設定ファイル（_quarto.yml）に以下を記述**  
  例:
  ```yaml
  format:
    pdf:
      pdf-engine: lualatex
      documentclass: bxjsbook
      include-in-header:
        text: |
          \usepackage{luatexja}
          \usepackage{luatexja-fontspec}
          \setmainjfont{IPAexMincho}      % 本文用フォント
          \setsansjfont{IPAexGothic}       % サンセリフ用
          \setmonojfont{IPAexGothic}        % 等幅フォント用
  ```
- **TinyTeX のパッケージが最新であるか確認**  
  R コンソールで：
  ```r
  tinytex::tlmgr_update()
  ```

---

## 3. ターミナル（cmd）で実施する作業

### 3.1 一時的なロケール設定
- **管理者権限の cmd を開き、以下のコマンドを実行**
  ```cmd
  set LANG=en_US.UTF-8
  set LC_ALL=en_US.UTF-8
  ```
  ※ これらの設定はその cmd セッション内だけ有効。

### 3.2 R の起動（CMD から）
- **R のインストール場所の確認（例）**  
  `R.home()` の結果が `C:/PROGRA~1/R/R-43~1.3` となっているので、実際のパスは  
  `C:\Program Files\R\R-4.3.3\bin\x64\R.exe` などになります。
- **cmd から R を起動する**：
  ```cmd
  "C:\Program Files\R\R-4.3.3\bin\x64\R.exe"
  ```

### 3.3 TinyTeX 関連コマンドの実行
- **R が起動した状態で上記の R コンソールの手順（TinyTeX の再インストール、パッケージ更新など）を実行**

### 3.4 フォントキャッシュやシステムコマンドの実行
- **必要に応じて、cmd から以下を実行してキャッシュ更新を試す**
  （TinyTeX に付属の `luaotfload-tool` が PATH に通っていれば）
  ```cmd
  luaotfload-tool --update
  luaotfload-tool --cache=erase
  ```

※ TinyTeX を使っている場合、通常は R コンソールから `tinytex::tlmgr_install()` や `system("luaotfload-tool --update")` を使うことで十分です。

---

## 4. まとめと今後の対策

- **R コンソール側**  
  - TinyTeX のインストールや再インストール、必要なパッケージの管理は R コンソール内で行う。
  - フォント設定（`luatexja-fontspec` を使った `\setmainjfont{IPAexMincho}` など）は _quarto.yml の `include-in-header` で管理する。

- **ターミナル（cmd）側**  
  - 一時的なロケール設定（`set LANG=en_US.UTF-8` など）は cmd セッションで実施する。
  - cmd から R を起動して、R コンソールで TinyTeX やフォントキャッシュの更新を行う。

- **今後のトラブルシューティング**
  - **エラーログの確認**: PDF 出力時の `index.log` をチェックし、具体的なエラー箇所を確認する。
  - **フォントの確認**: Windows の `C:\Windows\Fonts` や `fc-list :lang=ja` でシステムフォントが正しく認識されているか確認する。
  - **TinyTeX の更新**: 定期的に `tinytex::tlmgr_update()` を実行して最新状態に保つ。

---

このドキュメントを元に、今後の作業やトラブル発生時の対策として活用してください。  
何か疑問があれば、都度この資料や公式ドキュメントを参照すると良いでしょう。
