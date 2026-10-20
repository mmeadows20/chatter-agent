# Bounty Objective: ChatterAgent を有料にするなら — ソースは公開のまま、署名・公証済みのビルド済みを売る形を検討する
Target: schwarz9791/chatter-agent Issue #195
URL: https://github.com/schwarz9791/chatter-agent/issues/195

## Issue Description
**ChatterAgent を有料にする場合の売り方を検討する。** マスコット（Mac / XR）は無料のまま。課金とライセンスキーの処理は自前で書かない。

★ 今すぐ有料にするのではなく、決めるときの材料をここに集める。下の料金や各社の状況は 2026-10 時点の調査で、変わりうるので決める段で確かめ直すこと。

## 前提（決まっていること）

- **Mac App Store / TestFlight は使えない。** TestFlight は App Store 向けのビルドだけが対象で、App Sandbox が要る。ChatterAgent は次の点で審査ガイドラインに当たる
  - 任意のフォルダの node を子プロセスで起動する（2.4.5(iv)、2.5.2）
  - コンテナの外（`~/.config/chatter-agent/`）を読み書きする（2.5.2）
  - 別のアプリ（マスコット、合成エンジン）を起動し、終了後も残りうる（2.4.5(iii)）
  - そもそも Claude Code に hooks を入れてもらう必要があり、自己完結しない
- **配るなら Developer ID 署名と公証。** Apple Developer Program（年 99 ドル）が要る。公証には Hardened Runtime が必須で、今は切っている（`docs/knowledge/agent.md`。lldb などで attach できなくなるため）
- **node の同梱（#169 の別案）は頑張らない。** 合成エンジン（AivisSpeech / Kokoro）はどのみち別に入れてもらうので、同梱しても入れるものが1つ減るだけ

## ソースの扱い

ChatterAgent は Apache-2.0 で公開している。何もしなければ「有料」は「**自分でビルドすれば無料、ビルド済みは有料**」の意味になる。

| 方針 | 中身 |
|---|---|
| **A. 公開のまま、ビルド済みを売る（軸）** | 買う理由は、署名と公証済みですぐ動くことと、更新が届くこと |
| B. ChatterAgent を非公開に移す | 今後の版は別のライセンスにできる（公開済みの版は Apache-2.0 のまま）。機能で売るならこちら |
| C. 無料にして寄付を受ける | GitHub Sponsors など |

### A の前例

- Ardour（DAW）: ソースは無料、公式のビルド済みは最低 1 ドルか月額
- Synergy: GPL の無料版と、有料のビルド済み
- Conversations（Android の XMPP クライアント）: F-Droid 版は無料、Play 版は有料
- Aseprite: 2016 年までこの形。その後ライセンスを絞り、フォーク（LibreSprite）が生まれた

Polar を使った「OSS + macOS の署名済みバイナリを有料」の前例は見つからなかった。

### A の注意点

- **無料のビルド済みを第三者が配るのは止められない**（Apache-2.0 で合法。Ardour で実際に揉めている）。止められるのは名前だけなので、「フォークを配るなら ChatterAgent の名前とアイコンを使わない」商標の方針を先に書いておく
- **後からライセンスを絞らない。** Aseprite も Synergy も、そこでフォークが割れた
- 「ペイウォール」への反発には、最低価格つきの Pay What You Want が効く（Ardour の形）

## 配り方: A と A′

| | A: バイナリを売る | A′: 無料で配り、キーで解除する |
|---|---|---|
| 配り方 | 購入者ページから `.dmg` | GitHub Releases で誰でも |
| 未購入のとき | 手に入らない（ビルドすれば別） | 動くが、購入を促す表示が出る |
| 自動更新 | 作りにくい（取り直してもらう） | Tauri の updater がそのまま使える |
| Homebrew の cask | 載せられない（公開の URL が要る） | 載せられる |
| 自前で書くもの | なし | キーの入力欄と、検証 API の呼び出し1回（秘密鍵は要らない） |

- A′ の購入を促す表示はソースから消せるが、「自分でビルドする人は無料」と同じ扱いなので筋は崩れない
- **更新を知らせる仕組みは、最初に配る版から入れる。** 入っていない版を持っている人には、知らせる手段が無い
  - A′ は、公開の Release と Tauri の updater で自動更新まで通る
  - A は、自動で落とすには購入者だけに渡す口（認証つきのダウンロード）が要り、自前のサーバーかライセンスの検証に戻る。なので「新しい版があります」と購入者ページを開くところまでにする
  - Tauri の updater は専用の署名鍵で更新を検証する。失うとその版の利用者に更新を配れなくなるので、APK の鍵（#189）と同じく保管を決める
- A なら、アプリが決済サービスに縛られないので**同じものを複数の店で売れる**（海外向けと国内向けの二本立て）。A′ はキーの検証先が店ごとに分かれ、キーの API が無い店（BOOTH）では使いにくい

## 決済サービス

世界に売るなら Merchant of Record（MoR。各国の VAT・売上税を代わりに納める）が前提。

| | MoR | 手数料（基本） | 日本の個人で出金 | ライセンスキー API（秘密鍵なし） | 新しい版の再ダウンロード | PWYW | 届く層 |
|---|---|---|---|---|---|---|---|
| Polar | ○ | 5% + 50¢ | 日本は対象国（Stripe Connect Express）。個人で通るかは未確認 | ○ | 未確認 | ○ | 開発者・OSS |
| Dodo Payments | ○ | 4% + 40¢ | 個人で登録できる。日本の銀行で受け取れるかは未確認 | ○ | 未確認 | ○ | 開発者 |
| Lemon Squeezy | ○ | 5% + 50¢ | ○ 銀行振込 | ○ | ○（差し替えると既存の購入者にも届く） | ○ | 開発者 |
| Gumroad | ○ | 10% + 50¢ | 日本円で銀行振込（二次情報） | ○（端末ごとの有効化は無い） | 未確認 | ○ | クリエイター全般 |
| BOOTH | 未確認（海外の税の扱い） | 5.6% + 22円 | ◎ 国内の銀行 | ✕ | 未確認 | 未確認 | 日本の VRM / VTuber 界隈 |

- 海外カードは多くで +1.5%。出金・両替の費用は各社で別
- 安い商品では固定費（50¢ など）が料率の差より効く
- **Lemon Squeezy は先行きが弱い。** 2024 年に Stripe が買収し、運営自身が停滞を認めて Stripe Managed Payments へ移す方針
- **候補は、海外向けに Polar か Dodo（OSS との相性なら Polar、手数料なら Dodo）、国内向けに BOOTH**

### 外したもの

| | 理由 |
|---|---|
| Stripe Managed Payments | Stripe の通常手数料に 3.5% 上乗せ。ファイルの再配布とライセンスキーの仕組みが無い。明細が `LINK.COM` になる |
| Paddle | 今の Paddle Billing にライセンスキーの機能が無い |
| FastSpring | 料金が非公開で、小規模向きでない |
| Creem | ライセンス API に秘密鍵が要り、配布アプリから呼べない |
| Payhip | 日本の消費税は自己申告。公開アプリでのキー検証に秘密鍵が要る |
| itch.io | 米国の源泉徴収 30% の問題 |
| Setapp | 専用 SDK と別のバンドル ID が要り、独自のライセンスと更新機構を持てない。ニッチな開発者ツールは却下されうる |
| Keygen | ライセンス専用で、決済が別に要る。有料プランは月額固定 |

## 決めること

- ソースの扱い（A / B / C）と、配り方（A / A′）
- 店（海外向け・国内向け）と値段（固定か、最低価格つきの PWYW か）
- 商標の方針の文言と置き場所（README / NOTICE）
- Developer ID の取得と、Hardened Runtime を有効にしたときの開発体験（lldb などの attach）

## 確かめること

- Polar / Dodo: 日本の個人でオンボーディングと出金まで通るか（実際にアカウントを作って確かめる）
- BOOTH: ソフトウェアの販売の可否、海外の購入者の税の扱い、新しい版の再ダウンロード
- Gumroad: 決済手数料が基本の手数料とは別に掛かるか
- Homebrew: Gatekeeper を通らない cask を締め出す方針の現状（A′ で cask に載せるなら、署名と公証が前提になる）

## 関連

- #191（GitHub Release で配る）— この検討より先に動く。A′ を採るなら、#191 の配り方がそのまま土台になる
- #188（外部テスター）/ #169（core の npm 公開）

## 出典

- App Review ガイドライン: https://developer.apple.com/app-store/review/guidelines/
- TestFlight: https://developer.apple.com/help/app-store-connect/test-a-beta-version/testflight-overview
- 公証: https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution
- Ardour: https://ardour.org/faq.html
- Polar: https://polar.sh/docs/merchant-of-record/fees / https://polar.sh/docs/merchant-of-record/supported-countries / https://polar.sh/docs/features/benefits/license-keys
- Dodo Payments: https://dodopayments.com/pricing / https://docs.dodopayments.com/features/license-keys
- Lemon Squeezy: https://docs.lemonsqueezy.com/help/getting-started/fees / https://www.lemonsqueezy.com/blog/2026-update
- Gumroad: https://gumroad.com/pricing
- BOOTH の手数料: https://www.pixiv.co.jp/news/information/article/10522/
- Setapp: https://docs.setapp.com/docs/faq


## Instructions for Kuro (Autonomous Solver):
1. Use skills: `systematic-debugging` and `code-review-and-quality`.
2. Inspect the repository structure, locate relevant files, and understand the bug/feature.
3. Apply the minimal, robust fix or implementation.
4. Run any existing test suites or linters in the workspace.
5. Ensure the fix adheres to the project's existing code style.
6. When done, output a summary of files modified and verification results.
