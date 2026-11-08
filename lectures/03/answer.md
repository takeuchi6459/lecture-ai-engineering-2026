# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：エージェントは用語集（search_glossary）を検索してから答えるため、「3レベル」を「基礎・実装・発展の3段階で、どれか1つに取り組めば完了、原則採点なし」と用語集どおりに答え、最後に関係する回（第1回）も示した。
- `--trace` で見えたツール呼び出し（ツール名と引数）：「LoRAとQLoRAの違いは？」で `search_glossary({'query': 'LoRA'})` と `search_glossary({'query': 'QLoRA'})`。docstring の「query には用語を1つだけ入れる」どおり、用語を1つずつ2回に分けて検索していた。
- 1つの質問での LLM 呼び出し回数（`requests`）：2（tool_calls=2）
- 追加したツールと、docstring に書いた説明：`make_quiz(lecture: int)`。docstring は「指定した回の用語から確認問題を1問作り、問題と答えを返す。『第2回の確認問題を出して』のように、確認問題・復習問題・クイズを求められたときに使う」。「第2回の内容から確認問題を1問出して」で `make_quiz({'lecture': 2})` が呼ばれ、pytest も 15 passed。該当する用語がない回（第0回など）はメッセージを返すようにした。

## 宿題

- 取り組んだレベル：基礎
- やったこと：`app/agent.py` の INSTRUCTIONS から「用語や講座に固有のことを答える前に、必ず search_glossary で用語集を確認する。」の1行を消し、同じ質問「LoRAとQLoRAの違いは？」を `--trace` 付きで実行して、変更前と比べた。
- 結果：
  - 変更前：search_glossary を LoRA・QLoRA で1回ずつ（tool_calls=2、requests=2）
  - 変更後：変わらず search_glossary を LoRA・QLoRA で1回ずつ（tool_calls=2、requests=2）。回答の内容もほぼ同じで、最後に「関係する回：第2回」も出た
  - 入力トークンは 1196 → 1346 に増えたが、これは指示の変更ではなく、間に make_quiz を登録してツールの説明文が増えたためと考えられる
- 気づき・疑問：
  - INSTRUCTIONS から検索の指示を消しても、search_glossary の docstring に「答える前に使うこと」と書いてあるため、モデルはそちらを読んで検索を続けた。指示はエージェント全体の指示文だけでなく、ツールの説明文からも効いている
  - 本当に「検索しない」ふるまいを見るには、docstring 側の「答える前に使うこと」も外す必要がありそう
  - make_quiz では、INSTRUCTIONS の「宿題の答えそのものは書かず、ヒントで導く」が効いて、答えをそのまま出さずにヒントだけ出していた