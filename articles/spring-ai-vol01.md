---
title: "Spring AIを使ったChatアプリの作成 その1"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["spring", "springai", "ollama"]
published: true
---

ChatGPT4がリリースされて以来、生成AIを使用して業務を支援するアプリケーションが増えています。
実際の実装としては[LangChain](https://www.langchain.com/)で実装されているもしくは、サービス（OpenAO、Azure OpenAI Service）が提供しているAPIを直接使用しているケースが多いのではないかと思います。

サービスが提供するAPIを使用する場合は、APIを簡単に呼び出すことができるため、既存の技術スタックに組み込むことが容易です。一方で、RAGなど複雑な要件に対応することが難しい場合もあります。
LangChainはPythonもしくはNode.jsで実装されているため、エンタープライズシステムで多く使われているJavaアプリケーションに組み込むのは難しいです。また、技術スタックが大きく異なるために、かなりのコストが必要とされることが予想されます。

そこで、Spring AIを使って既存の技術スタック（Java & Spring Boot）の中に生成AIを組み込む方法を紹介します。

:::message alert
Spring AIは現時点では1.0.0-M1であり、開発中となっています。そのため、今後のアップデートで仕様が変更される可能性があります。
:::

## Goal

今回のGoalは以下の通りです。

1. REST APIをたたくとllama3を使って返答を返す。

    今回はOpenAIではなく、ローカルでFoundation Modelを使用できる[Ollama](https://ollama.com/)を使用します。

## 手順

基本的にSpring AIのドキュメントを参照して作成していきます。

@[card](https://docs.spring.io/spring-ai/reference/api/chat/ollama-chat.html)

### Spring Bootプロジェクトの作成

[Spring Initializr](https://start.spring.io/)を使ってプロジェクトを作成します。
この際に依存関係として以下を設定します。

- Spring Web
- Ollama

[Spring AIのドキュメント](https://docs.spring.io/spring-ai/reference/getting-started.html)ではSpring Webの依存関係が記載されていませんが、Spring AIが依存しているので追加します。

:::message
Spring Boot AI Starterを使用していますが、現時点ではSpring Webへの依存関係が設定されていないため、手動での追加が必要です。
:::

Spring Initializrでプロジェクトを作成すると以下のようなプロジェクトが作成されます。

![Generated Project](/images/articles/spring-ai-starter/01-init-project.png)

### Chat APIの作成

ここからChat用のAPIを作成していきます。基本的にはSpring Webの知識があれば実装できる内容となっています。

#### ChatControllerの作成

Chat用のAPIとしてChatControllerを作成し、その中に以下のように実装します。

```kotlin:ChatController.kt
package com.example.demo.controller

import org.springframework.ai.chat.messages.UserMessage
import org.springframework.ai.chat.model.ChatModel
import org.springframework.ai.chat.model.ChatResponse
import org.springframework.ai.chat.prompt.Prompt
import org.springframework.ai.ollama.api.OllamaOptions
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.bind.annotation.RestController

@RestController
class ChatController(private val chatModel: ChatModel) {
    private val options: OllamaOptions = OllamaOptions()

    init {
        options.model = "llama3.1"
    }

    @GetMapping("/chat")
    fun generate(
        @RequestParam(value = "message", defaultValue = "Tell me a joke") message: String
    ): String {
        val userMessage = UserMessage(message)
        val prompt = Prompt(userMessage, options)
        return  chatModel.call(prompt).result.output.content
    }
}
```

Ollamaを使用するときの肝としては、`OllamaOptions`で使用するモデルを指定することです。今回は先日発表された`llama3.1`を使用しています。

:::details Ollamaで使用できるモデル
Ollamaでは多くのモデルを使用することができます。詳細は[OllamaのModels](https://ollama.com/library)を参照してください。
:::

この例ではシステムプロンプトは指定していませんが、以下のように`SystemMessage`を指定することでシステムプロンプトを追加することができます。
`Prompt`は複数のメッセージを設定できるため、Chatのようにこれまでの履歴をDBなどで保持しておき、`Prompt`に設定することでより自然な会話を実現することができます。

```kotlin
val userMessage = UserMessage(message)
val systemMessage = SystemMessage("システムプロンプト")
val prompt = Prompt(listOf(userMessage, systemMessage), options)
```

#### Ollamaの準備

Ollamaを使用する際には事前にモデルをダウンロードしておく必要があります。
`ollama pull`コマンドを使用して必要なモデルをダウンロードできます。

```shell
ollama pull llama3.1
```

#### APIのテスト

作成したAPIをテストするために、以下のように`curl`コマンドを使用してAPIを叩いてみます。
今回は試しに現在の首相の名前を聞いてみましょう。

```shell
curl http://localhost:8080/chat?message=Who%20is%20japanese%20prime%20minister
```

すると以下のようなレスポンスが返ってきます。

```text
As of my last update, the Prime Minister of Japan is Fumio Kishida. He has been in office since October 2021. However, it's essential to verify this information through reliable sources for any updates or changes that might have occurred after my knowledge cutoff date.
```

きちんと岸田首相の名前が返ってきましたね。（さすがllama3.1）

## まとめ

Spring AIを使ってChatのREST APIを作成することができました。
既存の技術スタックの中に生成AIを組み込むことができるので、既存システムを活かしながらAIを活用した機能を追加することができます。
また、既存の技術スタックと同様なので、これまでの開発経験を活かすことができるので、開発コストを抑えることができますね。

今回はOllamaを使用しましたが、OpenAIやAzure OpenAI Service、Watsonx.AIなども利用できるので、企業で利用しているAIサービスをそのまま利用することも可能です。

今後もSpring AIを使って、より複雑なチャットやRAGを使った検索APIなどを作成していきたいと思います。
