# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
　エージェントはまとまった回答をしてきた。

- `--trace` で見えたツール呼び出し（ツール名と引数）：
　[tool call]   search_glossary({'query': 'LoRA'})
　[tool call]   search_glossary({'query': 'QLoRA'})　別々に呼び出されていた

　
- 1つの質問での LLM 呼び出し回数（`requests`）：requests=2, tool_calls=2

- 追加したツールと、docstring に書いた説明：＜ここの部分はAIから答えをもらっています＞
※app/tools.pyに追加

import random
import json

def make_quiz(lecture: int) -> str:
    """指定した回の用語から1つを選び、確認問題と答えを返す。
    「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。
    """
    glossary = load_glossary()
    # 1. lecture が一致する用語を集める
    target_terms = [item for item in glossary if item.get("lecture") == lecture]
    
    if not target_terms:
        return "指定された回の用語はありませんでした。"
        
    # 2. 1つ選ぶ
    chosen = random.choice(target_terms)
    
    # 3. 問題と答えを文字列で返す
    result = {
        "question": f"「{chosen['term']}」の意味は？",
        "answer": chosen["definition"]
    }
    return json.dumps(result, ensure_ascii=False)


    ※ツール登録　app/agent.py
    agent.tool_plain(tools.make_quiz)





## 宿題

- 取り組んだレベル：基礎 / 
- やったこと：
- 結果：

　INSTRUCTIONSの「必ず search_glossary で用語集を確認する」という指示の一文を削除
　→内容変わらず　また、requests=2, tool_calls=2のまま　予測ではrequests＝1となる予定だった

　AI曰く、
　app/tools.py の search_glossary ツールの説明文にはまだ「答える前に使うこと」という指示が残っているため、賢いモデルは「このツールは使うべきなんだな」と判断して自発的に呼び出してしまったと考えられ　とのこと。

app/tools.pyのdocstringを書き換えた
「query には調べたい用語を1つだけ入れる」→「query には調べたい用語をまとめて入れる」
結果：requests=2, tool_calls=1　になった。
「LORAとQLORAを別々に検索するのではなく、まとめて1回で検索」と行動を変えた

- 気づき・疑問：
INSTRUCTIONSの「必ず search_glossary で用語集を確認する」という指示の一文を削除したにも関わらず
挙動が変わらなかったことがどうも腑に落ちない。
