# Calculator (Flet)

Flet で作った iOS 風の電卓アプリです。カスタムボタンコントロールと四則演算を実装しています。

## 必要環境

- Python 3.10 以上
- [uv](https://docs.astral.sh/uv/)

## 実行方法

```bash
uv run main.py
```

起動するとローカルの Web サーバーが立ち上がり、ブラウザで電卓が開きます（通常は `http://127.0.0.1:<port>`）。

Flet CLI のホットリロードを使う場合も、必ず `--web` を付けてください。

```bash
uv run flet run --web main.py
```

`uv run flet run main.py` のように `--web` を付けない場合はネイティブのデスクトップモードで起動し、
この環境では下記の OpenGL エラーで落ちます。

## 実行モードについて（重要）

`ft.run(main)` のデフォルトは**ネイティブのデスクトップモード**（`FLET_APP`）で、
Flutter/GTK のウィンドウを開いて OpenGL コンテキストを使って描画します。

しかし、この環境（Wayland セッション・GPU デバイスノード `/dev/dri` なしのコンテナ/VM）では
OpenGL コンテキストを確保できず、起動時に以下のようなエラーで落ちます。

```
embedder.cc (2575): 'FlutterEngineRemoveView' returned 'kInvalidArguments'.
Failed to cleanup compositor shaders, unable to make OpenGL context current
flet: ../src/dispatch_common.c:872: epoxy_get_proc_address:
  Assertion `0 && "Couldn't find current GLX or EGL context."' failed.
```

これは電卓アプリのコードのバグではなく、**描画環境（GPU）の問題**です。

### 対処

そのため `main.py` では **Web ブラウザモード**で起動するようにしています。
ネイティブウィンドウの代わりにブラウザのタブで描画するため、GPU/OpenGL が無くても動作します。

```python
ft.run(main, view=ft.AppView.WEB_BROWSER)
```

> メモ: この環境には DRI レンダーノードが無いため、ソフトウェアレンダリング
> （`LIBGL_ALWAYS_SOFTWARE=1`）でも解決しません。Web モードが確実な方法です。
> GPU が使える別のマシンでデスクトップモードを使いたい場合は、環境変数で
> 切り替えられるようにすることもできます。

## 機能

- 数字入力・小数点入力
- 四則演算（`+` `-` `*` `/`）
- `AC`（クリア）、`+/-`（符号反転）、`%`（パーセント）
- ゼロ除算時は `Error` を表示
