![image](https://user-images.githubusercontent.com/113465005/226238596-cc76039e-67c2-46b6-b0bb-35d037ae66e1.png)

# 3 or 5 days POC VBD powered by: Azure Search + Azure OpenAI + Bot Framework + Langchain + Azure SQL + CosmosDB + Bing Search API

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/MSUSAzureAccelerators/Azure-Cognitive-Search-Azure-OpenAI-Accelerator?quickstart=1)
[![Open in VS Code Dev Containers](https://img.shields.io/static/v1?style=for-the-badge&label=Remote%20-%20Containers&message=Open&color=blue&logo=visualstudiocode)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/MSUSAzureAccelerators/Azure-Cognitive-Search-Azure-OpenAI-Accelerator)

## 日本語訳版

あなたの組織では、さまざまな場所に点在する多種多様なタイプのデータを理解できる検索エンジンとマルチチャンネル対応のスマートチャットボットが必要です。さらに、会話型のチャットボットは、質問に対する回答を提供するだけでなく、その回答がどのようにして、どこから得られたのかというソースと説明も提供する能力が求められます。言い換えれば、**組織内でプライベートかつセキュアなChatGPTがビジネスデータに関する質問を解釈、理解、回答できる**ようにしたいと考えています。

このPOC（プルーフ・オブ・コンセプト）の目的は、Azureサービスを使用して構築されたGPTバーチャルアシスタントの価値を、独自のデータを使用して自分自身の環境で示す/証明することです。納品物は以下の通りです：

1. Bot Frameworkで構築され、複数のチャンネル（Webチャット、MS Teams、SMS、Email、Slackなど）に公開されたバックエンドBot API
2. 検索とBot UIを備えたフロントエンドのWebアプリケーション。

このリポジトリは、OpenAIベースのスマート検索エンジンの構築方法を段階的に教えるために作成されました。各ノートブックは互いに積み重ねられ、最終的には2つのアプリケーションの構築に終わります。

**マイクロソフトのFTE（フルタイム雇用者）向け：** これは顧客が資金提供するVBD（Value-Based Delivery）であり、以下は納品のための資産です。

| **アイテム**        | **説明**                                                                                                                                   | **リンク**                                                                                                                                                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| VBD SKU情報とデータシート | CSAMはこれを"Customer Invested"としてUnified Support Contractのクレジット/時間に対してディスパッチする必要があります。顧客が3日か5日を選びます。 | [ESXP SKU ページ](https://esxp.microsoft.com/#/omexplanding/services/14486/geo/USA/details/1)                                                                                                                                           |
| VBD CSA認定               | CSAがワークショップを提供するために必要な認定を取得するリンク                                                                                    | [リンク 1](https://learningplayer.microsoft.com/activity/s9261799/launch), [リンク 2](https://learningplayer.microsoft.com/activity/s9264662/launch)                                                                                       |
| VBD 3-5日 POC資産 (IP)    | 提供されるべきMVP（このGitHubリポジトリ）                                                                                                        | [Azure-Cognitive-Search-Azure-OpenAI-Accelerator](https://github.com/MSUSAzureAccelerators/Azure-Cognitive-Search-Azure-OpenAI-Accelerator)                                                                                             |
| VBD ワークショップデッキ  | ワークショップを紹介し説明するスライドデッキ                                                                                                     | [Intro AOAI GPT Azure Smart Search Engine Accelerator.pptx](https://github.com/MSUSAzureAccelerators/Azure-Cognitive-Search-Azure-OpenAI-Accelerator/blob/main/Intro%20AOAI%20GPT%20Azure%20Smart%20Search%20Engine%20Accelerator.pptx) |
| CSAトレーニングビデオ     | マイクロソフトのCSA向けの2時間のトレーニング                                                                                                     | [POC VBD トレーニング録画](https://microsoft-my.sharepoint.com/:v:/p/jheseltine/EbxoBjWHJ-NJsnAM3qWXvVQBTK28SW7hgIrn7KaAJ77yaA?e=abiunn)                                                                                                |

---

**3-5日間POCの前提条件（クライアント向け）**

* Azureサブスクリプション
* Azure Open AIへの承認済みアプリケーション、GPT-4を含む。もし顧客がGPT-4の承認を持っていない場合、MicrosoftのCSAがワークショップ中に貸し出すことができます
* できれば、MicrosoftのメンバーをクライアントのAzure ADにゲストとして追加してください。もし不可能な場合、顧客はMicrosoftのメンバーに企業IDを発行できます
* このワークショップPOCのために、顧客のAzureテナント内でリソースグループ（RG）を設定する必要があります
* 顧客チームとMicrosoftチームは、このリソースグループに対して「Contributor」の権限を持つ必要があり、ワークショップの2週間前までにすべてを設定できるようにする必要があります
* RG内でストレージアカウントを設定する必要があります
* 顧客のデータ/ドキュメントは、ワークショップの日程の少なくとも2週間前に、blobストレージアカウントにアップロードする必要があります
* ワークショップ中のIDEの協力と標準化のために、AMLコンピュートインスタンスとJupyter Labが使用されます。これには、RG内にAzure Machine Learning Workspaceがデプロイされている必要があります
  * 注：Azure Machine Learningワークスペースで十分なコアコンピュートのクォータがあることを確認してください


---

# アーキテクチャ

![アーキテクチャ](./images/GPT-Smart-Search-Architecture.jpg "アーキテクチャ")

## フロー

1. ユーザーが質問をします。
2. アプリ内で、OpenAI GPT-4 LLMが賢いプロンプトを使用して、ユーザー入力に基づいて使用するソースを決定します。
3. 4種類のソースが利用可能です：
   * 3a. Azure SQLデータベース - アメリカのCOVID関連の統計情報を含む。
   * 3b. Azure Bing Search API - 公共のウェブサイトでのQnAのようなシナリオを可能にするインターネットへのアクセスを提供。
   * 3c. Azure Cognitive Search - BlobストレージからのAIによって豊富なドキュメント（10kのPDFと90kの記事）を含む。
     * 3c.1. OpenAIを使用して上位Kのドキュメントチャンクをベクトル化
     * 3c.2. 需要に応じてベクトルベースのインデックスを埋める。
     * 3c.3. ベクトルベースのインデックスでベクトル検索を行い、上位Nのチャンクを取得。
   * 3d. CSV形式の表ファイル - アメリカのCOVID関連の統計情報を含む。
4. アプリはソースから結果を取得し、答えを作成します。
5. タプル（質問と回答）は、対話の記録とさらなる分析のためにCosmosDBに保存されます。
6. 答えがユーザーに提供されます。

---

## デモ

https://gptsmartsearch.azurewebsites.net/

MS Teamsでボットを開くには、[こちら](https://teams.microsoft.com/l/chat/0/0?users=28:5d583679-8196-4673-9d77-c294c010bca5)をクリックしてください。

---

## 🔧**機能**

- [Bot Framework](https://dev.botframework.com/)と[Bot Service](https://azure.microsoft.com/en-us/products/bot-services/)を使用して、Bot APIバックエンドをホストし、MS Teamsを含む複数のチャンネルに公開。
- 100% Python。
- [Azure Cognitive Services](https://azure.microsoft.com/en-us/products/cognitive-services/)を使用して非構造化ドキュメントをインデックス化および強化：言語の検出、OCR画像、キーフレーズの抽出、エンティティ認識（人物、メール、住所、組織、URL）。
- Azure Cognitive Searchのベクトル検索機能を使用して最良のセマンティック回答を提供。
- ユーザーがシステムと対話するにつれて、オンデマンドでベクトルを作成（最初に全体のデータレイクをベクトル化するのとは対照的）。
- [LangChain](https://langchain.readthedocs.io/en/latest/)を使用して、Azure OpenAI、ベクトルストア、プロンプトの構築、エージェントの作成との対話のためのラッパーとして。
- 多言語対応（任意の言語を摂取、インデックス化、理解）。
- マルチインデックス -> 複数の検索インデックス。
- CSVファイルおよびSQLフレーバーデータベースを使用した表形式のデータQ&A。
- [Azure AIドキュメントインテリジェンスSDK（旧Form Recognizer）](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-3.0.0)を使用して複雑な/大規模なPDFドキュメントを解析。
- [Bing Search API](https://www.microsoft.com/en-us/bing/apis)を使用してインターネット検索と公共のウェブサイトでのQ&Aを強化。
- CosmosDBを使用してユーザーの会話を永続的に保存。
- [Streamlit](https://streamlit.io/)を使用して、PythonでフロントエンドWebアプリケーションを構築。

もちろんです、以下はそのMarkdownテキストを日本語に翻訳したものです。

---

## **POC/アクセラレーターを実行する手順**

注:（前提条件）すでにAzure OpenAIサービスを作成している必要があります。

1. このリポジトリを自分のGithubアカウントにフォークします。
2. Azure OpenAIスタジオで、以下のモデルをデプロイします：**デプロイメント名はモデル名と同じであることを確認してください。**
   - "gpt-35-turbo"
   - "gpt-35-turbo-16k"
   - "gpt-4"
   - "gpt-4-32k"
   - "text-embedding-ada-002"
3. このアクセラレーターのすべての資産が配置されるリソースグループを作成します。Azure OpenAIは異なるRGまたは異なるサブスクリプションにすることができます。
4. 以下をクリックして、ノートブックを実行するために必要なすべてのAzureインフラストラクチャを作成します（Azure Cognitive Search、Cognitive Servicesなど）：

[![Azureへデプロイ](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpablomarin%2FGPT-Azure-Search-Engine%2Fmain%2Fazuredeploy.json)

**注**: 以前に「Cognitive services Multi-Service account」を作成したことがない場合は、Azureポータルで手動で作成し、Responsible AIの条件を読んで同意してください。これがデプロイされたら、上記のデプロイメントボタンを使用してこれを削除してください。

5. フォークしたリポジトリをAML Compute Instanceにクローンします。リポジトリがプライベートの場合、トラブルシューティングセクションでプライベートリポジトリをどのようにクローンするかを参照してください。
6. ノートブックは**Python 3.10 conda環境**で実行することを確認してください。
7. 依存関係を自分のマシンにインストールします（以下のpipコマンドを、ノートブックを実行するのと同じconda環境で実行してください。例えば、AZMLコンピュートインスタンスで実行する場合：

```zsh
conda activate azureml_py310_sdkv2
pip install -r ./common/requirements.txt
```

いくつかのpip依存関係エラーが出るかもしれませんが、それは問題ありません。エラーに関わらず、ライブラリは正しくインストールされています。

8. `credentials.env`ファイルを編集して、ステップ4で作成されたサービスから自分自身の値に更新します。
9. **ノートブックを順番に実行してください**。それぞれが前のものに基づいています。

---

<details>

<summary>FAQs</summary>

## **よくある質問**

1. **LLMにコンテキストを提供するためにAzure Cognitive Searchエンジンを使用する理由は何ですか？なぜLLMを微調整しないのですか？**

A: [OpenAIのドキュメント](https://platform.openai.com/docs/guides/fine-tuning)に引用すると：「GPT-3は、オープンインターネットからの膨大なテキストデータで事前にトレーニングされています。少数の例だけでプロンプトが与えられると、多くの場合、何を試みているのかを直感的に理解し、適切な完成を生成できます。これは一般に「フューショット学習」と呼ばれています。
微調整は、プロンプトに収まるよりも多くの例でトレーニングを行うことで、フューショット学習を改善し、多くのタスクでより良い結果を得ることができます。モデルが微調整されると、プロンプトで例を提供する必要はもうありません。これにより**コストを節約し、低レイテンシのリクエストが可能になります**」

ただし、モデルを微調整するには、数百または数千のプロンプトと完成のタプル（基本的にはクエリとレスポンスのサンプル）を提供する必要があります。微調整の目的は、LLMに企業のデータの知識を与えるのではなく、各プロンプトで例を必要とせずにタスクを非常にうまく実行できるようにするための例を提供することです。

プロンプトで公開すべきでない独自のデータが含まれている場合や、使用される言語が医療、薬局、またはインターネット上で一般的に見られない言語が使用されている他の業界やユースケースなど、微調整が必要なケースもあります。

</details>

<details>

<summary>Troubleshooting</summary>

## Troubleshooting

## トラブルシューティング

プライベートリポジトリをクローンする手順：

- ターミナルを開き、以下のテキストを貼り付けて、自分のGitHubのメールアドレスに置き換えます。[新しいSSHキーを生成する](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)。

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- SSH公開キーをクリップボードにコピーします。[新しいSSHキーを追加する](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)。

```bash
cat ~/.ssh/id_ed25519.pub
# ターミナルに表示されるid_ed25519.pubファイルの内容を
# 選択してクリップボードにコピーします
```

- GitHubで、**設定-> SSHとGPGキー-> 新しいSSHキー**に移動します。
- 「タイトル」フィールドには、新しいキーにわかりやすいラベルを追加します。例：「AML Compute」。"Key"フィールドには、公開キーを貼り付けます。
- プライベートリポジトリをクローンします。

```bash
git clone git@github.com:YOUR-USERNAME/YOUR-REPOSITORY.git
```

</details>

# 免責事項：添付された図やサンプルコードは、一切の保証なしで「現状のまま」提供されます。これらはMicrosoftの提供または約束と解釈すべきではありません。また、Microsoftは提供される情報の正確性を保証するものではありません。このコードサンプルに対して、MICROSOFTは明示または暗黙を問わず、一切の保証を行いません.
