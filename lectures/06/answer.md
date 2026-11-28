# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：
  - aws_aws___search_documentation({'topics': ['general'], 'search_phrase': 'Amazon S3 バージョニング 概要'})
  - aws_aws___search_documentation({'search_phrase': 'Using versioning in S3 buckets', 'topics': ['general']})
  - glossary_search_glossary({'query': 'S3 バージョニング'})
- 気づき・疑問：
  - 自作スクリプト（`uv`）、公開パッケージ（`uvx`）、外部リモートサーバー（`url`）という起動場所や通信方式の異なる3種類のMCPサーバーを、`app/mcp_servers.json` に書き足すだけで同じ `<サーバー名>_<ツール名>` の形式でエージェントから統一的に扱える点が便利だと感じた。
  - 一方で、STEP 4 では3つのサーバーを同時に有効化したことでツール数が7個に増え、AWSに関する質問に対しても不要な `glossary_search_glossary` が追加で呼ばれて入力トークン数（15,857 tokens）が増加した。必要なサーバーやツールだけを接続する設計の重要性を実感した。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎
- やったこと：`app/mcp_server.py` の `# TODO(宿題・基礎): get_lecture_summary も公開しよう` の直下に `server.tool()(tools.get_lecture_summary)` の1行だけを追記した。エージェント側の設定（`app/mcp_servers.json`）や本体（`app/mcp_agent.py`）は変更せず、`--list-tools`、`--trace` 付き質問実行、および `uv run pytest -q` で動作を確認した。
- 結果：
  - `--list-tools`：モデルに見えているツールが 7 個から 8 個に増え、`glossary_get_lecture_summary`（引数 `{"lecture": {"title": "Lecture", "type": "integer"}}`）が追加された。
  - `--trace` 実行（「第2回では何を学びますか?」）：`[tool call] glossary_get_lecture_summary({'lecture': 2})` が呼び出され、第2回「ファインチューニングとLoRA」の要点が正しく回答された（入力 5,894 tokens / 出力 280 tokens / モデル応答 2 回）。ツール追加前は `glossary_search_glossary({'query': '第2回'})` で個別用語を検索していたが、追加後は講義全体の要約ツールを適切に選択した。
  - テスト（`uv run pytest -q`）：`21 passed in 8.40s` となり、すべてのテストが通過した（なお、過去の回で `app/agent.py` に追加していた古い関数参照を第6回開始時点の状態に戻したうえでテストを実行した）。
- 気づき・疑問：エージェント側を一切書き換えなくても、起動時に MCP クライアントがサーバーへ `tools/list` を問い合わせる仕組みにより、サーバー側の1行追加だけで新しいツールが自動的にモデルへ公開・選択される疎結合な設計のメリットを実感した。
