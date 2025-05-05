# AI エンジニアリング講座　第三回　任意課題

本リポジトリは、AIエンジニアリング講座第3回の任意課題において出された、RAGの検証実験の結果を書き示したものである。
<br>
なお、Google Colabで実装確認する際は、下記のボタンをクリックいただきたい。
<br>
**[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kazukitakayamas/llm-engineering-RAG/blob/main/rag.ipynb)**

<br>


検証結果として結論から申すと、文脈を考慮したチャンク化の導入（演習ノートブックの3）が人から見ても、スコア上でも精度が高かった。
<br>
今回の評価は人目によるものとOpenAIのAPIを使った評価を行い、定性と定量の両面から評価を行っている。


## 1. 質問設計の観点と意図
<br>
今回の質問はLlama3がリリースされた以降のニューストピックスを題材として考え、設計しました。
具体的に、以下の５つである。
<br>

1~2は高レベルの専門知識の取得が出来ているか否かを判断するために、最新の論文を利用。3以降はML分野に問わず(問わず問いつも、2/3がMLトピックとなってしまっている。)一般的なニュース記事を利用。当然だが、Llama3リリース(2024年4月)以降のトピックを使っている。
<br>
<br>
1. LLMのアラインメントの新手法に関する論文発表について（T-SPMO）※2025/4以降に発表の論文の内容を抜粋
<br>
ソース: https://arxiv.org/abs/2504.20834
<br>
<br>
2. 表現学習の新手法に関する論文発表について(RKDO(Recursive KL Divergence Optimization: A Dynamic Framework for Representation Learning))※2025/4以降に発表の論文の内容を抜粋
<br>
ソース: https://arxiv.org/abs/2504.21707
<br>
<br>
3. 2025年4月時点の日本の総理大臣について。※当然だが、Llama3リリース時点(2024年4月)では、岸田文雄が総理大臣のため、検証がしやすい
<br>
<br>
4. DeepSeekショックとは何かについて。2025年早々に起きたML業界へのショッキングニュースを扱ってみた。これをRAGで適切に取ってこれるかを見たい。
<br>
<br>
5. MCP(Model Context Protocol)とは何かについて。Anthoropicが出したAIが様々なものに繋がるためのルールのようなもので、これも比較的新しい概念のため、RAGにより正しく取れるかを確認したい。
<br>

## 2. RAGの実装方法と工夫点について
<br>
使用した手法は主に６つ(※1~5は講義のノートブック)で、１つだけ自分で調べた手法の「Fusion-in-Decoder (FiD)」というものである。

FiDは、各取得パッセージを独立してエンコードし、デコーダーで“融合”して回答を生成する手法で、複数のパッセージが相互作用するクロスアテンションにより、長くても分散した文脈を効果的に捉えられるため、従来の単一連結より高い性能を発揮する。
<br>
冒頭に記載の通り、今回色々な手法を検証した中で最も精度が高かったのが、「文脈を考慮したチャンク化の導入」であった。

## 3. 結果の分析と考察について
<br>
1. RAGありとRAGなしについて
<br>
これは、当然明らかな差が出ており、RAGを行うことで適切に最新の情報をLLMが取得出来ていることが確認された。今回扱ったのはLlama3がリリースされた以降の情報のため、Llama3の知識としては備えている訳もなく、RAGなしの場合だと当然ハルシネーションが生じた。一方RAGを行うことで手法は様々あれど最低限あるべき情報の取得はどの手法でもできていた。
<br>
<br>
2. RAGによって回答が改善したケースと悪化したケースについて
 <br>
1)改善したケースについて
 <br>
 冒頭記載の通り、手法を問わずRAGを行うことで、最低限あるべき回答を含んだ生成は出来ていた。
 <br>
 2)悪化したケースについて
 <br>
 各種手法によって生成される細かいニュアンス（人にとっての読みやすさ）に差が生じているのが散見された。例えば、Rerankでは、ソースとなる文章をそのまま引っ張て来ており、正しいと言えば正しいが、人にとって読みやすいものではなかった。
 <br>
 <br>
3. 結果に基づいて、RAGの有効性と限界についての考察
 <br>
 1)有効性について
 <br>
 まず、１つ言えるのは、SFTやAlignmentのようなPosttrainingと異なり、そこまで手間をかけずに最新の情報をモデルに出力させる期待が持てるのはRAGの大きな利点として改めて思った。
 最新の知識を備えていない場合であっても、モデルのベース性能が今回のLlamaのようにある程度担保されているものであれば、最新の情報をハルシネーションリスクを抑えながら運用出来ると感じた。
 <br>
 2)限界について
 <br>
 最新の情報をアップデートするために、それ用のデータベースを常に高い品質のデータとしての信頼性を保ちながら運用することに高い難易度があると感じた。
 例えば、新聞記者や株価のアナリストのような常に最新情報を必要とするような職種の人にRAGでの運用は今の技術上ハードルがまだ、一定程度あるように思える。
　<br>
 <br>
　3)その他感想※特段求められているものではない
 <br>
　ノーマルRAGについてだが、これは意外と精度が高く、今回の論文やニュース記事から持ってくるケースにかなり高い精度で合致していた。ただ、比較すると「文脈を考慮したチャンク化の導入」より、一歩劣るものだった。
　続いて、「Rerank」についてだが、そもそも解答に近似したものを判断するのが、Llama３のため信頼性がそこまで高くない可能性があるのが１点、関連性というのが、定性要素が高くうまく取得できないといのが２点。これらが要因となり、あまり良い精度が出なかったと思料。
　最後に、「意味的チャンク化」について。意味的なまとまり（トピック、議論、例示など）に基づいてテキストを分割をするものだが、これもかなり、抽象度が高く、あまり意味のある分割が出来ていないと思料。
　これらのことから、「文脈を考慮したチャンク化の導入」が一番シンプルで高い精度が出たと考えられる。



## 4. 評価指標の設計について（発展的な宿題）※詳細は、以下のコードに記載の通り、変更を加え、より細かい粒度感で評価を行えるようにしている。
<br>
具体的には下記の通り、改善案として挙げた。
<br>
1. 英語から日本語に評価項目を書き換えた
<br>
2. 評価項目を３つに増やした
<br>
3. 平均値ではなく、中央値で計算


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



