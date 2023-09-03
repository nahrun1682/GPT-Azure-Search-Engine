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

| **アイテム**                   | **説明**                                                                                                       | **リンク**                                                                                                                                                |
|----------------------------|--------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| VBD SKU情報とデータシート      | CSAMはこれを"Customer Invested"としてUnified Support Contractのクレジット/時間に対してディスパッチする必要があります。顧客が3日か5日を選びます。  | [ESXP SKU ページ](https://esxp.microsoft.com/#/omexplanding/services/14486/geo/USA/details/1)                                                         |
| VBD CSA認定                  | CSAがワークショップを提供するために必要な認定を取得するリンク                                                                                 | [リンク 1](https://learningplayer.microsoft.com/activity/s9261799/launch), [リンク 2](https://learningplayer.microsoft.com/activity/s9264662/launch)   |
| VBD 3-5日 POC資産 (IP)       | 提供されるべきMVP（このGitHubリポジトリ）                                                                                                     | [Azure-Cognitive-Search-Azure-OpenAI-Accelerator](https://github.com/MSUSAzureAccelerators/Azure-Cognitive-Search-Azure-OpenAI-Accelerator)             |
| VBD ワークショップデッキ      | ワークショップを紹介し説明するスライドデッキ                                                                                                  | [Intro AOAI GPT Azure Smart Search Engine Accelerator.pptx](https://github.com/MSUSAzureAccelerators/Azure-Cognitive-Search-Azure-OpenAI-Accelerator/blob/main/Intro%20AOAI%20GPT%20Azure%20Smart%20Search%20Engine%20Accelerator.pptx) |
| CSAトレーニングビデオ         | マイクロソフトのCSA向けの2時間のトレーニング                                                                                                  | [POC VBD トレーニング録画](https://microsoft-my.sharepoint.com/:v:/p/jheseltine/EbxoBjWHJ-NJsnAM3qWXvVQBTK28SW7hgIrn7KaAJ77yaA?e=abiunn)       |

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
# Architecture 
![Architecture](./images/GPT-Smart-Search-Architecture.jpg "Architecture")

## Flow
1. The user asks a question.
2. In the app, an OpenAI GPT-4 LLM uses a clever prompt to determine which source to use based on the user input
3. Four types of sources are available:
   * 3a. Azure SQL Database - contains COVID-related statistics in the US.
   * 3b. Azure Bing Search API - provides access to the internet allowing scenerios like: QnA on public websites .
   * 3c. Azure Cognitive Search - contains AI-enriched documents from Blob Storage (10k PDFs and 90k articles).
      * 3c.1. Uses OpenAI to vectorize the top K document chunks
      * 3c.2. Fills up the vector-based indexes on-demand.
      * 3c.3. Gets the Top N Chunks by doing a vector search on vector-based indexes.
   * 3d. CSV Tabular File - contains COVID-related statistics in the US.
4. The app retrieves the result from the source and crafts the answer.
5. The tuple (Question and Answer) is saved to CosmosDB to keep a record of the interaction and further analysis.
6. The answer is delivered to the user.

---
## Demo

https://gptsmartsearch.azurewebsites.net/

To open the Bot in MS Teams, click [HERE](https://teams.microsoft.com/l/chat/0/0?users=28:5d583679-8196-4673-9d77-c294c010bca5)

---

## 🔧**Features**

   - Uses [Bot Framework](https://dev.botframework.com/) and [Bot Service](https://azure.microsoft.com/en-us/products/bot-services/) to Host the Bot API Backend and to expose it to multiple channels including MS Teams.
   - 100% Python.
   - Uses [Azure Cognitive Services](https://azure.microsoft.com/en-us/products/cognitive-services/) to index and enrich unstructured documents: Detect Language, OCR images, Key-phrases extraction, entity recognition (persons, emails, addresses, organizations, urls).
   - Uses Vector Search Capabilities of Azure Cognitive Search to provide the best semantic answer.
   - Creates vectors on-demand as users interact with the system. (versus vectorizing the whole datalake at the beginning)
   - Uses [LangChain](https://langchain.readthedocs.io/en/latest/) as a wrapper for interacting with Azure OpenAI , vector stores, constructing prompts and creating agents.
   - Multi-Lingual (ingests, indexes and understand any language)
   - Multi-Index -> multiple search indexes
   - Tabular Data Q&A with CSV files and SQL flavor Databases
   - Uses [Azure AI Document Intelligence SDK (former Form Recognizer)](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-3.0.0) to parse complex/large PDF documents
   - Uses [Bing Search API](https://www.microsoft.com/en-us/bing/apis) to power internet searches and Q&A over public websites.
   - Uses CosmosDB as persistent memory to save user's conversations.
   - Uses [Streamlit](https://streamlit.io/) to build the Frontend web application in python.
   

---

## **Steps to Run the POC/Accelerator**

Note: (Pre-requisite) You need to have an Azure OpenAI service already created

1. Fork this repo to your Github account.
2. In Azure OpenAI studio, deploy these models: **Make sure that the deployment name is the same as the model name.**
   - "gpt-35-turbo"
   - "gpt-35-turbo-16k"
   - "gpt-4"
   - "gpt-4-32k"
   - "text-embedding-ada-002"
3. Create a Resource Group where all the assets of this accelerator are going to be. Azure OpenAI can be in different RG or a different Subscription.
4. ClICK BELOW to create all the Azure Infrastructure needed to run the Notebooks (Azure Cognitive Search, Cognitive Services, etc):

[![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpablomarin%2FGPT-Azure-Search-Engine%2Fmain%2Fazuredeploy.json) 

**Note**: If you have never created a `Cognitive services Multi-Service account` before, please create one manually in the azure portal to read and accept the Responsible AI terms. Once this is deployed, delete this and then use the above deployment button.

5. Clone your Forked repo to your AML Compute Instance. If your repo is private, see below in Troubleshooting section how to clone a private repo.

6. Make sure you run the notebooks on a **Python 3.10 conda enviroment**
7. Install the dependencies on your machine (make sure you do the below pip comand on the same conda environment that you are going to run the notebooks. For example, in AZML compute instance run:
```
conda activate azureml_py310_sdkv2
pip install -r ./common/requirements.txt
```

You might get some pip dependancies errors, but that is ok, the libraries were installed correctly regardless of the error.

8. Edit the file `credentials.env` with your own values from the services created in step 4.
9. **Run the Notebooks in order**. They build up on top of each other.

---

<details>

<summary>FAQs</summary>
  
## **FAQs**

1. **Why use Azure Cognitive Search engine to provide the context for the LLM and not fine tune the LLM instead?**

A: Quoting the [OpenAI documentation](https://platform.openai.com/docs/guides/fine-tuning): "GPT-3 has been pre-trained on a vast amount of text from the open internet. When given a prompt with just a few examples, it can often intuit what task you are trying to perform and generate a plausible completion. This is often called "few-shot learning.
Fine-tuning improves on few-shot learning by training on many more examples than can fit in the prompt, letting you achieve better results on a wide number of tasks. Once a model has been fine-tuned, you won't need to provide examples in the prompt anymore. This **saves costs and enables lower-latency requests**"

However, fine-tuning the model requires providing hundreds or thousands of Prompt and Completion tuples, which are essentially query-response samples. The purpose of fine-tuning is not to give the LLM knowledge of the company's data but to provide it with examples so it can perform tasks really well without requiring examples on every prompt.

There are cases where fine-tuning is necessary, such as when the examples contain proprietary data that should not be exposed in prompts or when the language used is highly specialized, as in healthcare, pharmacy, or other industries or use cases where the language used is not commonly found on the internet.
</details>

<details>

<summary>Troubleshooting</summary>
  
## Troubleshooting

Steps to clone a private repo:
- On your Terminal, Paste the text below, substituting in your GitHub email address. [Generate a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key).
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
- Copy the SSH public key to your clipboard. [Add a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key).
```bash
cat ~/.ssh/id_ed25519.pub
# Then select and copy the contents of the id_ed25519.pub file
# displayed in the terminal to your clipboard
```
- On GitHub, go to **Settings-> SSH and GPG Keys-> New SSH Key**
- In the "Title" field, add a descriptive label for the new key. "AML Compute". In the "Key" field, paste your public key.
- Clone your private repo
```bash
git clone git@github.com:YOUR-USERNAME/YOUR-REPOSITORY.git
```
</details>

# Disclaimer: The attached diagrams and sample code are provided AS IS without warranty of any kind, and should not be interpreted as an offer or commitment on the part of Microsoft, and Microsoft cannot guarantee the accuracy of any information presented. MICROSOFT MAKES NO WARRANTIES, EXPRESS OR IMPLIED, IN THIS CODE SAMPLE.

