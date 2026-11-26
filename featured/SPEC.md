# featured/ データ定義 (schema_version 1, FINAL 2026-11-27)

受け渡し先: VolVoX123/-karasu-data リポジトリ直下 featured/ (既存 data/・index.json とは分離)
時刻はすべて JST・ISO 8601 (+09:00)。CSV と index.json は同一コミットで公開する。
文字コードは UTF-8。CSV の空欄は「不明」、0 は「観測した 0」。
キー: runs = snapshot_id / snapshots = snapshot_id + auction_id / results = auction_id。
訂正は同じキーで更新し、再取込で増殖させない。

## snapshot_runs.csv (観測記録: 失敗・部分・0件も1行残す)
- snapshot_id: YYYYMMDD-HHMM
- scheduled_at / executed_at
- category_id: 2084009286
- pages_planned / pages_fetched
- items_observed: 一覧で観測した件数
- items_identified: auction_id を特定できた件数
- seller_identified_count: seller_id まで特定できた件数
- status: success / partial / failed
- note

## snapshots/YYYY-MM-DD.csv (商品観測: その日の全観測を追記)
- snapshot_id, observed_at, category_id
- rank: 注目順の順位 / badge_rank: 注目度バッジ (1-3、なければ空)
- auction_id, seller_id (aucUserId), seller_name, title
- start_price: 開始価格 (現在価格から代用しない)
- current_price, buy_now_price, bids (観測時点)
- watchers, watchers_checked_at (取得した場合のみ)
- featured_price_raw: その観測で取得した候補値の生値 / fp_status: candidate / verified
- fp_checked_at: 候補値の実取得時刻 (一覧観測と別なら分ける。不明なら空)
- start_time, end_time
- fetch_status: ok / partial / failed
- 候補値の履歴は snapshots が正本。item_results の値は最新観測の要約にすぎない

## item_results.csv (終了結果: auction_id 単位で累積)
- auction_id, seller_id, seller_name
- first_observed_at, last_observed_at, best_rank, badge_rank
- featured_price_raw (最新観測の要約) / featured_price_after_end: 終了後の値
- fp_status: candidate (意味・単位が未確定の間) / verified
- ended_status: live (出品中) / ended_confirmed (終了を確認) / ended_unknown (終了を確認できない)
- ended_checked_at: 終了を確認した時刻
- result: sold / unsold / unconfirmed (取得失敗・ページ消失・終了不明は unconfirmed)
- result_basis: ended_page / daily_collection / ended_page+daily_collection / none
- result_evidence: sold/unsold を判断した明示的な証拠 (該当フィールド/値や表示)
- final_price, bids, watchers, watchers_checked_at
- start_time, end_time (実績)
- tracking_status: tracking / done / expired (結果の判定とは別。7日再試行の超過で expired、結果は unconfirmed のまま)
- result_checked_at, note

## index.json
- schema_version
- publication_status: draft / pending / published (未公開・承認待ちは published と扱わない)
- generated_at: 生成時刻
- published_at: 公開した版のみ実日時、未公開は null
- last_snapshot_at, last_result_check_at
- files[]: name (featured/ からの相対パス), rows, sha256

## 検証メモ (2026-11-27)
- featuredPrice は、今回確認した 3 件では終了後に 0 へ戻った。終了後ページからは復元不可。
  → 識別は出品中の観測が必須。終了済み無作為 3 件の 0 は「非利用」ではなく、この範囲では終了後は 0。
- featuredPrice の意味・単位、出品中の非利用品での値は未検証 (fp_status=candidate の間は利用判定に使わない)。
- 日次売上の収録が遅れている場合、注目観測が最新でも売上を 0 と扱わず、収録範囲/未到着を表示する。
