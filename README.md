# Dashcam Rust Viewer Layout

要件定義書に合わせた、ローカルWebビューア連携用のドラレコ録画システムです。

今まで作った `dashcam_rust_no_crates` は残したまま、こちらは別版として作っています。

## 目的

Webビューアのデフォルト動画フォルダ `videos/` に、フロント/リアの録画をカテゴリ別に保存します。
常時録画は10分単位のラップファイルとして保存します。

```text
videos/
  front/
    continuous/  フロント常時録画 10分ラップ
    event/       フロント事故イベント録画
    manual/      フロント手動録画
  rear/
    continuous/  リア常時録画 10分ラップ
    event/       リア事故イベント録画
    manual/      リア手動録画
```

## セットアップ

Mac:

```sh
brew install rust ffmpeg
```

Raspberry Pi 5:

```sh
sudo apt update
sudo apt install -y ffmpeg rustc cargo v4l-utils
```

## カメラ確認

Mac:

```sh
ffmpeg -f avfoundation -list_devices true -i ""
```

Raspberry Pi 5:

```sh
v4l2-ctl --list-devices
ls /dev/video*
```

## 実行

Macで試す例:

```sh
cd /Users/kojiyoshi/Documents/Codex/2026-07-01/ma/outputs/dashcam_rust_viewer_layout
FRONT_DEVICE=1 REAR_DEVICE=0 cargo run --offline
```

Raspberry Pi 5でUSBカメラ2台を使う例:

```sh
cd /path/to/dashcam_rust_viewer_layout
FRONT_DEVICE=/dev/video0 REAR_DEVICE=/dev/video1 DASHCAM_FPS=30 DASHCAM_SIZE=1280x720 cargo run --offline
```

要件定義書の標準値に合わせ、初期値は以下です。

- 解像度: `1280x720`
- FPS: `30`
- 動画形式: `.mp4`
- 映像コーデック: H.264 `libx264`
- 映像ビットレート: `3M`
- ラップ長: 10分
- 容量上限: フロント30GB、リア30GB、合計約60GB

変更したい場合:

```sh
export DASHCAM_FPS=27.5
export DASHCAM_VIDEO_BITRATE=3M
export DASHCAM_VIDEO_CODEC=libx264
export DASHCAM_MAX_BYTES_PER_CAMERA=32212254720
```

## 操作

- `a`: 事故イベントとして保存
- `e`: 手動イベントとして保存
- `i`: フロント/リアのカメラ初期化
- `f`: 録画停止

## Webビューアとの接続

Webビューア側の動画フォルダを、このプロジェクトの `videos/` に向けます。

このシステムは、常にインターネットへ接続する前提ではありません。
カメラ録画をスマートフォン等で確認したい時だけ、ローカル接続または転送モードを使います。
通常走行中は録画を優先し、Webビューアや転送用の接続は必要時のみ起動する想定です。

例:

```json
{
  "video_dir": "videos"
}
```

ビューアを別フォルダで動かす場合は、録画側の保存先を絶対パスで指定できます。

```sh
DASHCAM_MEDIA_DIR=/home/pi/videos FRONT_DEVICE=/dev/video0 REAR_DEVICE=/dev/video1 cargo run --offline
```

この場合、Webビューア側も `/home/pi/videos` を参照してください。

## 要件定義書との対応

実装済み:

- フロント/リア2カメラ常時録画
- 10分ラップ管理
- `.mp4` 保存
- H.264 3Mbps指定
- 720p初期設定
- 手動記録
- イベント記録
- トリガー時点の前1ラップ、現在1ラップ、後1ラップ保存
- フロント/リア合計60GB相当のループ削除
- ローカルWebビューア向けフォルダ構成

未実装または別プロセス想定:

- YOLO/OpenCVによるADAS画像診断
- TTC衝突予知
- IMU/GPIOによる自動イベントトリガー
- 音声AAC 64kbpsの同時多重化
- レンズ汚れ、豪雨、霧などの自己診断
- 録画確認時だけWeb転送モードへ切り替える親プロセス制御
