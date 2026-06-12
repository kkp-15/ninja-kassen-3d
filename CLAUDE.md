# 合戦3D俯瞰デモ プロジェクト（第1作：箱館戦争）

## 概要
歴史上の合戦をテレビ特番風の3D俯瞰アニメで見せる単一HTMLデモ。
第1作が箱館戦争（hakodate-1869.html）。今後、関ヶ原・厳島などへシリーズ展開予定。
クライアントはモバイルブラウザ閲覧が主。Claude.aiのチャットで原型を制作し、本リポジトリへ引き継いだ。

## 絶対に守る制約
- 単一HTMLファイル完結。ビルドツール・フレームワーク・npm依存は使わない
- Three.js は r128 を cdnjs から読み込む（バージョンを変えない。OrbitControls等のアドオンは使えない前提で書く）
- Cloudflare Pages（アカウント: kkp-15 / ドメイン: kkpwebninja.com のサブドメイン）にそのまま置ける静的構成を維持（GitHub Pagesは使わない。KKP標準: push→GitHub Actions deploy.yml→CF Pages自動デプロイ）
- UIに絵文字を使わない
- フォントは Noto Serif JP + Cormorant Garamond（Google Fonts）
- 兵力は「約」表記。パネル注記「※地形・布陣は様式化した再現。兵力は諸説ある概数、日付は旧暦。」を削除しない
- クレジット表記「VOICEVOX:ずんだもん」は音声またはずんだもん解説ON時に表示される。消さない

## ファイル構成
- hakodate-1869.html … 本体（HTML/CSS/JSすべて入り）
- voice/phase1.wav 〜 phase6.wav … ずんだもん音声（**未生成**。下記仕様参照）
- voice/phase1.txt 〜 phase6.txt … 読み上げ原稿（textZと同一。VOICEVOXに貼り付ける用）
- voice/README.md … VOICEVOX書き出し手順
- .claude/launch.json … ローカル確認用サーバー定義（python3 http.server :8765。デプロイ対象外）
- CLAUDE.md … このファイル

## コード構造（hakodate-1869.html 内の script）
- `terrainH(x,z)` … 渡島半島の様式化地形。ガウシアンblob＋ridgeの合成。海面は y=0、実標高データではない
- `groundY(x,z)` / `P(x,z,extra)` … 地表追従ヘルパー（矢印・部隊の高さ決め）
- 五稜郭 … `starShape()` で星形ポリゴンを生成し Extrude。中心座標 `FORT=[75,-70]`
- `makeArrow(points,color,r)` … 進軍矢印。TubeGeometry の drawRange を伸ばして成長アニメ
- `makeTroop` / `makeShip` / `makeFlag` … 部隊・艦船・旗。`makeUnitLabel(名前,兵力,色)` で頭上ラベル（Canvasスプライト）
- `fitLabels()` … ラベルの近接巨大化防止。カメラ距離が `LABEL_NEAR`(=140) 未満では見かけサイズを一定に抑える（地名=placeLabels／部隊=unitLabels、unitLabels は clearDyn() でリセット）。毎フレーム animate() から呼ぶ。一律サイズは従来どおり `sy=h/8.4` で調整
- `makeCombat(x,z,scale)` … 交戦エフェクト（発砲閃光・硝煙・地面リング）
- `makeFirefight(a,b)` … 曳光弾の撃ち合い ／ `makeExplosion(pos)` … 爆発（朝陽轟沈で使用）
- `phases[]` … 全6局面。各要素は `{date,title,text,textZ,mood,camA,camB,build()}`
  - `build()` 内で `dyn` グループにオブジェクト追加し、`updaters.push(function(t,time){...})` で局面内アニメを登録
  - `t` = 局面内進行 0..1、`time` = 経過秒。`seg(t,a,b)` で区間正規化
  - 局面切替時は `clearDyn()` が dyn を破棄（共有リソース boxGeo / smokeTex は dispose しない）
- カメラ … camA→camB を ease 補間＋慣性 lerp。`mood` で背景色と霧濃度が局面ごとに変化
- UI … 下部パネル（「たたむ」で折りたたみ可）、ずんだもん解説トグル、音声トグル、局面ドット、キー操作（←→/Space）

## ずんだもん音声の仕様
- `voice/phase1.wav` 〜 `phase6.wav` を置くと、音声ON時に局面切替で自動再生（無ければ無音でスキップ、エラーにしない）
- 読み上げ原稿は `phases[].textZ`（ずんだもん口調版）をそのまま使う
- VOICEVOX（無料・Mac版GUIアプリあり）の話者「ずんだもん」で書き出す
- 公開前に VOICEVOX とずんだもん（東北ずん子・ずんだもんプロジェクト）の公式利用ガイドラインを確認すること（広告収益サイトでの利用条件含む）

## 残タスク
1. 実機スマホでの最終確認（体感のみ）
   - 2026-06-11 モバイルビューポート(375px)でシミュレータ検証済み: 負荷は全局面で最大64draw call・約4.1万トライアングル、局面切替30回×2でGPUリソースリークなし、コンソールエラーなし
   - 近接時にラベルが画面を覆う問題は `fitLabels()` 導入で修正済み。局面3「開陽丸」(y17→12)・局面6「榎本武揚ら」(+26→+10)のラベル高さも画面内に収まるよう修正済み
   - 実機確認はローカルサーバー（.claude/launch.json または `python3 -m http.server 8765`）を立て、同一Wi-FiのスマホからMacのIPで開く
2. voice/ 音声ファイルの生成と再生確認（モバイルは初回タップ後でないと再生されない仕様に注意）
   - 原稿は voice/phase1.txt〜phase6.txt に書き出し済み。手順は voice/README.md
   - 再生はAudio要素1つを使い回す方式（iOSの自動再生制限対策）に変更済み
3. サブドメイン公開：例 `kassen-3d.kkpwebninja.com`
   - KKP標準フロー（new-ninja-siteスキルあり）: リポジトリ作成 → .github/workflows/deploy.yml 設置 → Cloudflare Pagesプロジェクト作成 → DNSにCNAME追加 → カスタムドメイン設定
   - JSON-LD等のauthor名は「web忍者の砦」固定（個人名を入れない）。apex掲載・sitemap更新は別タスクに分ける
4. シリーズ2作目（関ヶ原 or 厳島）。`terrainH` と `phases[]` を差し替える設計をそのまま踏襲する

## 開発スタイル（KKP共通ルール）
- 戦略は市場イン×ランチェスター弱者の戦略（ニッチ特化）
- デプロイは Cloudflare Pages（deploy.yml経由の自動デプロイ）、DNS操作は Cloudflare Dashboard > DNS > Records
- 環境は Mac + Claude Code（GUI操作のみ）。パスは /Users/ 配下を前提にする
