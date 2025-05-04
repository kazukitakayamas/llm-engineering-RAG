# AI エンジニアリング講座　第三回　任意課題

本リポジトリは、AIエンジニアリング講座第3回の任意課題において出された、RAGの検証実験の結果を書き示したものである。
<br>
なお、Google Colabで実装確認する際は、下記のボタンをクリックいただきたい。

**[![Open In Colab (コード翻訳)][(https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kazukitakayamas/llm-code-translation-tasks/blob/main/BELU-score-vllm-inference.ipynb)](https://github.com/kazukitakayamas/llm-engineering-RAG/blob/main/rag.ipynb)**
<br>


検証結果として結論から申すと、文脈を考慮したチャンク化の導入（炎症ノートブックの3）が人から見ても、スコア上でも精度が高かった。
<br>
今回の評価は人目によるものとOpenAIのAPIを使った評価を行い、定性と定量の両面から評価を行っている。

### 評価コード
```
# @title 5点満点評価実装

from openai import OpenAI
from google.colab import userdata
import re
import time

gold_answer = "DeepSeekショックとは、中国浙江省杭州市を拠点とするAI開発企業が提供する次世代AI推論エンジン「DeepSeek」の登場により引き起こされた、業界全体の急激な変化とインパクトを指します。この技術は、膨大なデータを短時間で解析し、適切な予測や意思決定を支援する能力を持つことで、従来の業務や市場構造に劇的な影響を与えています。DeepSeekの3つの特徴超高速なデータ処理 DeepSeekは、大量のデータをリアルタイムで解析する能力を持ち、迅速な意思決定を可能にします。汎用性の高さ 金融、医療、製造、マーケティングなど、多様な分野に応用可能な柔軟性を持っています。操作のシンプルさ 高度な技術がバックエンドに存在しながら、直感的で使いやすいインターフェースを提供。"

client = OpenAI(api_key=userdata.get("OPENAI_API_KEY"), max_retries=5, timeout=60)

def openai_generator(query, retry_attempts=3, delay=2):
    """拡張されたOpenAI API呼び出し関数"""
    for attempt in range(retry_attempts):
        try:
            messages = [
                {
                    "role": "user",
                    "content": query
                }
            ]

            response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=messages,
                temperature=0.1  # より一貫性のある回答のために低温度を設定
            )
            return response.choices[0].message.content
        except Exception as e:
            print(f"API呼び出し失敗 (試行 {attempt+1}/{retry_attempts}): {e}")
            if attempt < retry_attempts - 1:
                time.sleep(delay)  # 次の試行前に待機
                
    print("すべてのAPI呼び出し試行が失敗しました")
    return None

def extract_numeric_score(text):
    """テキストから数値スコアを抽出する補助関数"""
    if not text:
        return 0
        
    # 単一の数字（0-5）を探す
    matches = re.findall(r'\b[0-5]\b', text)
    if matches:
        return int(matches[-1])  # 最後に見つかった数字を使用（結論が最後に来ることが多い）
    
    # 文字で書かれた数字も探す
    number_words = {
        'zero': 0, 'one': 1, 'two': 2, 'three': 3, 'four': 4, 'five': 5,
        '０': 0, '１': 1, '２': 2, '３': 3, '４': 4, '５': 5  # 全角数字
    }
    
    for word, value in number_words.items():
        if word in text.lower():
            return value
    
    return 0  # デフォルト値

def evaluate_answer_accuracy(query, response, reference):
    """元の関数シグネチャを維持しながら内部実装を改善"""
    
    # 改善されたテンプレート1：より明確な日本語の評価基準を含む
    template_accuracy1 = (
        "以下の質問と回答を分析してください。\n"
        "ユーザー回答が基準回答の内容とどれだけ一致しているかを0から5の尺度で評価してください：\n"
        "- 0: ユーザー回答は質問の要点を完全に外しているか、全く関連がない\n"
        "- 1: ユーザー回答には基準回答と一致する小さな部分があるが、大部分は的外れまたは不正確\n"
        "- 2: ユーザー回答は基準回答と部分的に一致するが、約半分の内容が基準から逸脱している\n"
        "- 3: ユーザー回答はおおむね基準回答と一致するが、顕著な逸脱や省略がある\n"
        "- 4: ユーザー回答は基準回答とほぼ一致しており、ごくわずかな不一致や省略がある\n"
        "- 5: ユーザー回答は基準回答とすべての重要な側面で完全に一致している\n"
        "あなたの評価は数字（0、1、2、3、4、または5）のみを回答してください。説明は不要です。\n\n"
        "質問: {query}\n"
        "ユーザー回答: {sentence_inference}\n"
        "基準回答: {sentence_true}\n"
        "評価:"
    )
    
    # 改善されたテンプレート2：評価基準を明確にし、逆順での評価
    template_accuracy2 = (
        "あなたは質問への回答を評価する専門家です。\n"
        "ユーザー回答が基準回答と比較してどれだけ正確かを0〜5のスケールで評価してください：\n"
        "- 0: ユーザー回答は完全に不正確か質問と無関係\n"
        "- 1: ユーザー回答には正しい点もあるが、ほとんどは基準回答から逸脱している\n"
        "- 2: ユーザー回答は内容の約半分が基準回答と一致するが、重大な逸脱がある\n"
        "- 3: ユーザー回答は基準回答と部分的に一致するが、一部の矛盾や欠落要素がある\n"
        "- 4: ユーザー回答は軽微な不一致を除いて基準回答とほぼ一致している\n"
        "- 5: ユーザー回答はすべての重要な点で基準回答と一貫している\n"
        "評価は数字のみで回答し、説明は付けないでください。\n\n"
        "質問: {query}\n"
        "基準回答: {sentence_inference}\n"
        "ユーザー回答: {sentence_true}\n"
        "評価:"
    )
    
    # 第3のテンプレート：より直接的で簡潔
    template_accuracy3 = (
        "質問に対する2つの回答を比較評価してください。\n"
        "ユーザー回答が基準回答の内容をどれだけ正確に反映しているかを0〜5で評価：\n"
        "0=まったく不正確、5=完全に一致\n\n"
        "質問: {query}\n"
        "ユーザー回答: {sentence_inference}\n"
        "基準回答: {sentence_true}\n"
        "評価点数（0〜5の数字のみ）:"
    )
    
    scores = []
    
    # テンプレート1での評価
    result1 = openai_generator(
        template_accuracy1.format(
            query=query,
            sentence_inference=response,
            sentence_true=reference
        )
    )
    score1 = extract_numeric_score(result1)
    scores.append(score1)
    
    # テンプレート2での評価（順序入れ替え）
    result2 = openai_generator(
        template_accuracy2.format(
            query=query,
            sentence_inference=reference,
            sentence_true=response
        )
    )
    score2 = extract_numeric_score(result2)
    scores.append(score2)
    
    # テンプレート3での評価（追加の視点）
    result3 = openai_generator(
        template_accuracy3.format(
            query=query,
            sentence_inference=response,
            sentence_true=reference
        )
    )
    score3 = extract_numeric_score(result3)
    scores.append(score3)
    
    # 外れ値を除外するため、中央値を使用
    scores.sort()
    if len(scores) >= 3:
        final_score = scores[1]  # 3つの値の中央値
    else:
        # スコアが不足している場合は平均を使用
        final_score = sum(scores) / len(scores) if scores else 0
    
    return final_score
```  
