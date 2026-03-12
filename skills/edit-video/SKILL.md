---
name: edit-video
description: 撮影済み動画に台本のテロップマーカーを自動焼き込み（Remotion）
user_invocable: true
argument_description: "<raw.mp4> [script.html] [output.mp4]"
---

# /edit-video

撮影済みの動画ファイルに、台本HTMLの `[テロップ: ...]` マーカーを自動でオーバーレイして焼き込む。
Whisper APIで発話タイミングを検出し、テロップ表示タイミングを自動整列。Remotionでレンダリング。

## Multi-Pass Design

既存パイプライン（5ステージ）を多段階原則にマッピング。

| Pass | 内容 | 対応ステージ |
|------|------|-------------|
| **Pass 1: 素材収集** | 動画ファイル + 台本HTML読込、Whisper音声認識、無音検出、テロップ/セクション抽出 | prepare |
| **Pass 2: 整列 + 自己批判** | テロップ×トランスクリプト照合（align.ts）。「overlap解決できているか？」「low confidence箇所はないか？」を確認 | prepare後半 |
| **Pass 3: 生成** | Remotion bundle → renderMedia（h264, CRF18）+ チャプターファイル生成 | chapters, bundle, render |
| **Pass 4: 品質ゲート** | renderStillで5フレーム自動出力→目視確認 | render前 |
| **Pass 5: 修正** | FAIL箇所のEDL手動調整→再レンダリング。目視確認なしのバッチ処理は厳禁 | render再実行 |

### Pass 4 品質ゲート項目（動画編集固有）

| # | チェック項目 | FAIL基準 |
|---|------------|---------|
| 1 | テロップ表示タイミング | 発話と2秒以上ずれている |
| 2 | テロップ重なり | 複数テロップが同時表示で読めない |
| 3 | ジャンプカット | カット境界で映像が不自然に飛ぶ |
| 4 | confidence分布 | low confidence が全体の20%超 |
| 5 | セクションタイトル | 表示されない / タイミングがずれている |

## アーキテクチャ

**Remotion v2方式**。shorts-generatorプロジェクト内のVideoEditorコンポジションを使用。

- **前処理スクリプト**: `shorts-generator/scripts/edit-video/` (prepare, parse-script, transcribe, align)
- **Remotionコンポジション**: `shorts-generator/src/compositions/VideoEditor/` (6コンポーネント)
- **エントリポイント**: `scripts/edit-video/index.ts` -- prepare -> bundle -> render -> chapters

台本なし = ジャンプカットのみ（無音区間+フィラーワード自動カット）。

## パイプライン（5ステージ）

1. **prepare**: whisper.cpp + 無音検出 + フィラーカット + テロップ/セクション抽出 -> EDL
2. **chapters**: YouTubeチャプター用タイムスタンプファイル生成
3. **bundle**: Remotionのbundle（動画をpublic/にhard link）
4. **render**: Remotion renderMedia（VideoEditor, h264）
5. **cleanup**: symlinkの後片付け + 結果レポート

## 使い方

```
/edit-video /path/to/raw.mp4 /path/to/script.html
```

台本なしでジャンプカットのみ:
```
/edit-video /path/to/raw.mp4
```

スピーカー名指定（Lower Third表示）:
```
SPEAKER_NAME="{author_name}" SPEAKER_TITLE="{title}" /edit-video /path/to/raw.mp4 /path/to/script.html
```

## 実行手順

1. 引数からファイルパスを取得。output未指定なら `raw-edited.mp4` とする
2. shorts-generatorプロジェクトでパイプラインを実行:

```bash
cd <YOUR_VIDEO_PROJECT>
SPEAKER_NAME="{author_name}" SPEAKER_TITLE="{title}" \
  npx ts-node --project scripts/tsconfig.json scripts/edit-video/index.ts "$RAW_VIDEO" "$SCRIPT_HTML" "$OUTPUT"
```

3. 完了後、結果を報告:
   - テロップ数、confidence分布（high/medium/low）
   - セクション数、チャプターファイルの内容
   - 出力ファイルパスとサイズ
   - 各ステージの所要時間
   - low confidenceが多い場合は手動確認を推奨

4. Discord通知（任意）:

```bash
node <YOUR_PROJECT_ROOT>/scripts/discord-notify.mjs \
  --channel team \
  --persona soga \
  --message "動画編集完了: $(basename $OUTPUT), テロップ${COUNT}個"
```

## 前提条件

- `OPENAI_API_KEY` が環境変数に設定されていること（Whisper用、whisper.cppローカルモードでも音声抽出にffmpegが必要）
- `ffmpeg` / `ffprobe` がインストール済み
- Remotion依存: `shorts-generator` の `node_modules` が揃っていること
- フォント: `FOT-筑紫B見出ミン Std E`（なければ `Hiragino Mincho ProN` にフォールバック）

## YouTube動画編集機能（6レイヤー構成）

| レイヤー | コンポーネント | 機能 |
|---------|--------------|------|
| 1 | JumpCutVideo | 無音+フィラーカットされた動画を連結再生 |
| 2 | SectionTitleBar | 左上にセクションタイトル常時表示（スライドイン） |
| 3 | TelopOverlay | 5種テロップ（center-impact/red/lower-third/upper-keyword/quote） |
| 4 | SpeakerLowerThird | 左下にスピーカー名+肩書き（冒頭数秒のみ） |
| 5 | VideoProgressBar | 動画下部に薄いプログレスバー |
| 6 | FadeTransition | 開始0.5秒フェードイン、終了0.8秒フェードアウト |

## テロップ分類

| パターン | TelopType | 表示位置 | アニメーション |
|---------|-----------|---------|--------------|
| 数字+単位（%、億、万、店舗） | center-impact | 画面中央・大 | scale-fade (spring) |
| 赤文字/赤枠指定 | center-impact-red | 画面中央・赤大 | scale-fade (spring) |
| 「」付き引用+出典 | quote | 中央 | fade-italic |
| / 区切りの情報列挙 | lower-third | 下1/3 | fade |
| それ以外 | upper-keyword | 上部 | fade |

## 確定テロップスタイル（1920x1080基準）

- フォント: FOT-筑紫B見出ミン Std E（見出し専用明朝、Extra Bold）
- 色: 白文字 + 薄グレーシャドウ（赤文字廃止。どの動画背景にも合う）
- CenterImpact: 210px, UpperKeyword: 160px, LowerThird: 140px, Quote: 130px
- 縁取り: 極薄2px (WebkitTextStroke)
- シャドウ: 5px 6px 8px rgba(80,80,80,0.35)

## 中間成果物

workDir（動画と同じディレクトリの `edit-video-work/`）に保存:
- `transcript.json` -- Whisper結果
- `edl.json` -- EDL全データ（keeps, telops, sections, speaker）
- `chapters.txt` -- YouTube概要欄用チャプタータイムスタンプ

## ファイル構成

```
shorts-generator/
  src/compositions/VideoEditor/
    index.tsx              -- メインcomposition (6レイヤー統合)
    JumpCutVideo.tsx       -- EDLのkeepセグメント連結再生
    TelopOverlay.tsx       -- テロップSequence表示（5スタイル+アニメーション）
    SectionTitleBar.tsx    -- 左上セクションタイトル（スライドイン/アウト）
    SpeakerLowerThird.tsx  -- 左下スピーカー名表示
    VideoProgressBar.tsx   -- 下部プログレスバー
    FadeTransition.tsx     -- フェードイン/アウト
    types.ts               -- Remotion側型定義
  scripts/edit-video/
    index.ts         -- パイプラインオーケストレータ（prepare -> chapters -> bundle -> render）
    prepare.ts       -- whisper.cpp + 無音検出 + フィラー検出 -> EDL生成
    parse-script.ts  -- 台本HTML -> テロップマーカー + セクション情報抽出
    transcribe.ts    -- Whisper API -> 単語タイムスタンプ（旧方式、prepare.tsは@remotion/install-whisper-cpp使用）
    align.ts         -- テロップ x トランスクリプト照合
    md-to-html.ts    -- Markdown台本 -> Part別HTML分割
    generate-ass.ts  -- ASSファイル生成（旧FFmpeg方式、スタンドアロン用）
    render.ts        -- FFmpegレンダリング（旧方式、スタンドアロン用）
    types.ts         -- パイプライン型定義
  scripts/tsconfig.json  -- CommonJS, node moduleResolution
```

## Phase 2（将来）

- BGM自動挿入 + 音声ダッキング
- 図表の自動差し込み（NotebookLMスライド連携）
- 効果音の自動配置
- 縦ショート版レイアウト（同一EDLから1080x1920に変換）
