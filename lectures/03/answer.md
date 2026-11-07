# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
    素のLLMは一般的な情報をもとに回答を生成するが、エージェントは情報を検索した上で講座資料の内容をもとに回答を生成する。
- `--trace` で見えたツール呼び出し（ツール名と引数）：
     search_glossary({'query': 'LoRA'})
     search_glossary({'query': 'QLoRA'})
     make_quiz({'lecture': 2})
- 1つの質問での LLM 呼び出し回数（`requests`）：2
- 追加したツールと、docstring に書いた説明:
    ツールはmake_quiz
    説明文は以下
    """
    指定した回の用語から１つを選び、確認問題と答えを返す。
    「第2回の確認問題を出して」のように、復習用の問題を求められた時に使う。
    """
## 宿題

- 取り組んだレベル：基礎
- やったこと：agent.pyのINSTRUCTIONSの記載内容を更新し、更新前後で同じ質問を行った。
    - 更新内容：必ず最初にクイズを作成し、確認問題と回答を生成して下さい。
    - 質問内容：LoRA とQLoRAの違いは?
- 結果：
    - まずはmake_quizツールが実行され、問題と回答が生成された。その後にsearch_glossaryツールが実行され、質問に対する回答が生成された。「必ず最初に」という指示通り、make_quizが最初に実行されたことが確認できた。

    - 更新前の動作:
        - uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"
            [model] gemini-3.5-flash-lite
            LoRAとQLoRAの違いは、**ベースモデルの重みの精度（量子化）**にあります。

            * **LoRA**: 事前学習済みの重みは固定したまま、更新分を表す小さな低ランク行列（アダプタ）のみを学習する手法です。モデルの精度（通常は16ビットなど）はそのままに、学習するパラメータ数を大幅に削減します。
            * **QLoRA**: LoRAの仕組みをベースにしつつ、**ベースモデルの重みを4ビットなどに量子化**してメモリ消費を極限まで抑えた手法です。これにより、より大きなモデルを比較的小さなGPUメモリでも効率よく学習できるようになります。

            第2回

            --- trace ---
            [tool call]   search_glossary({'query': 'LoRA'})
            [tool call]   search_glossary({'query': 'QLoRA'})
            [tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
            [tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
            [usage] RunUsage(cost=Decimal('0.0008688'), details={'text_prompt_tokens': 1346}, input_text_tokens=1346, input_tokens=1346, output_tokens=186, requests=2, tool_calls=2)
    - 更新後の動作:
        - uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"
            [model] gemini-3.5-flash-lite
            まずは第1回からの確認問題です！

            ---
            **【確認問題】**
            * **問題:** [コンテキストウィンドウ]の意味は？
            * **回答:** LLMが1回の呼び出しで扱える入力と出力の合計トークン数の上限。会話履歴やツールの結果もこの中に収める必要があるため、何をモデルに渡すかの設計が重要になる。
            ---

            それでは、LoRAとQLoRAの違いについてご説明します。

            * **LoRA（Low-Rank Adaptation）**
            事前学習済みの重みを固定したまま、重みの更新分を2つの小さな低ランク行列の積で表し、その小さな行列だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習するパラメータが非常に少なく、学習結果（アダプタ）も小さなファイルとして保存できます。
            * **QLoRA（Quantized LoRA）**
            ベースモデルの重みを4ビットなどに量子化してメモリ使用量を大幅に節約した上で、LoRAと同様にパラメータを追加・学習する手法です。これにより、より大きなモデルを比較的小さなGPU環境でも効率よく扱えるようになります。

            つまり、**モデルの重みを量子化してメモリを劇的に節約する仕組み（QLoRA）が加わっているかどうかが大きな違い**です。

            関連する回：第2回

            --- trace ---
            [tool call]   make_quiz({'lecture': 1})
            [tool call]   search_glossary({'query': 'LoRA'})
            [tool call]   search_glossary({'query': 'QLoRA'})
            [tool return] make_quiz -> {"question": "[コンテキストウィンドウ]の意味は？", "answer": "LLMが1回の呼び出しで扱える入力と出力の合計トークン数の上限。会話履歴やツールの結果もこの中に収める必要があるため、何をモデルに渡すかの設計が重要…
            [tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
            [tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
            [usage] RunUsage(cost=Decimal('0.0013013'), details={'text_prompt_tokens': 1471}, input_text_tokens=1471, input_tokens=1471, output_tokens=344, requests=2, tool_calls=3)
- 気づき・疑問：
    - INSTRUCTIONSの内容によってツールの実行順序のコントロールはできるものの、ツールの実行方法については正確な制御ができていないことに気づいた。make_quizは「回を指定すること」を前提としているのに、回を指定せずとも自己判断で第1回を指定してクイズを作成している。
        - tools.pyのdocstring内容:指定した回の用語から１つを選び、確認問題と答えを返す。例「第2回の確認問題を出して」のように、復習用の問題を求められた時に使う。
