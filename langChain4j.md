## 简介

LangChain4j 的目标是简化将 LLM 集成到 Java 应用程序中的过程。官方一开始支持`python`，后面支持了java。

官方文档地址：[在这里](https://docs.langchain4j.info/intro)

LangChain4j 提供[与多种 LLM 提供商的集成](https://docs.langchain4j.info/integrations/language-models/)。 每种集成都有自己的 Maven 依赖项。 这里使用阿里的通义千问大模型，模型参考文档[在这里](https://help.aliyun.com/zh/model-studio/get-api-key?spm=a2c4g.11186623.0.0.21da11fcWrqkMc)

记忆管理、历史记录、索引增强(RAG)、Agent是不断深入使用和学习的必经之路，加油！

## 开始使用

### Hello World

`maven`依赖

```xml
<!--这是 LangChain4j 的核心模块，提供所有通用抽象和基础组件，不包含任何大模型的实现-->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>1.0.0-beta3</version>
</dependency>
<!--扩展适配器 针对 OpenAI 兼容 API 的具体实现模块-->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>1.0.0-beta3</version>
</dependency>
```

调用大模型接口交互代码，很简单。

```java
// 构建模型对象
ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1") // 关键！
                .apiKey("sk-3606fcd9cb2a4f639f850fbcb84f0183") // 从通义千问官网获取
                .modelName("qwen3-vl-plus") // 或 qwen-plus, qwen-turbo, qwen-vl-plus（多模态）
                .build();
// 这里开始聊天
String answer = chatModel.chat("Say 'Hello World'");
System.out.println(answer); // Hello World
```

### SpringBoot 使用

放在`SpringBoot`中使用，大概如下。可以使用`SpringBoot`集成

创建模型

```java
@Configuration
public class AiConfig {
    /**
     * 普通模型
     */
    @Bean
    public ChatLanguageModel chatModel() {
        return OpenAiChatModel.builder()
                .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1") // 关键！
                .apiKey("sk-123456") // 从 https://dashscope.console.aliyun.com 获取
                .modelName("qwen3-vl-plus") // 或 qwen-plus, qwen-turbo, qwen-vl-plus（多模态）
                .build();
    }

    /**
     * 响应流式模型
     */
    @Bean
    public StreamingChatLanguageModel streamingChatModel() {
        return OpenAiStreamingChatModel.builder()
                .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1") // 关键！
                .apiKey("sk-123456") // 从 https://dashscope.console.aliyun.com 获取
                .modelName("qwen3-vl-plus") // 或 qwen-plus, qwen-turbo, qwen-vl-plus（多模态）
                .build();
    }
}
```

Restful接口调用

```java
@Slf4j
@RestController
@RequestMapping("/qa")
public class QaController {

    @Resource
    private ChatLanguageModel chatLanguageModel;

    @Resource
    private StreamingChatLanguageModel streamingChatModel;

    /**
     * 简单测试
     *
     * @param prompt 提示词
     * @return 结果
     */
    @GetMapping("/test")
    public String test(@RequestParam String prompt) {
        log.info("prompt: {}", prompt);
        String result = chatLanguageModel.chat(prompt);
        log.info("result: {}", result);
        return result;
    }

    /**
     * 图片测试
     *
     * @param image  图片
     * @param prompt 提示词
     * @return 结果
     */
    @PostMapping("/image")
    public String image(@RequestPart MultipartFile image, @RequestParam String prompt) {
        log.info("prompt: {}", prompt);
        // 转为 Base64
        String base64Image;
        try {
            base64Image = "data:image/png;base64," + Base64.getEncoder().encodeToString(image.getBytes());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        // 传递给 LangChain4j 的多模态模型
        UserMessage userMessage = UserMessage.from(
                TextContent.from(prompt),
                ImageContent.from(base64Image)
        );

        ChatResponse response = chatLanguageModel.chat(userMessage);
        String result = response.aiMessage().text();
        log.info("result: {}", result);
        return result;
    }

    /**
     * 简单测试
     *
     * @param prompt 提示词
     * @return SSE 对象
     */
    @GetMapping("/streamingChat")
    public SseEmitter streamingChat(@RequestParam String prompt) {
        log.info("prompt: {}", prompt);
        SseEmitter sseEmitter = new SseEmitter(60_000L);
        this.streamingChatModel.chat(prompt, new StreamingChatResponseHandler() {
            @Override
            public void onPartialResponse(String token) {
                try {
                    log.info("token: {}", token);
                    sseEmitter.send(SseEmitter.event().data(token));
                } catch (Exception e) {
                    sseEmitter.completeWithError(e);
                }
            }

            @Override
            public void onError(Throwable error) {
                sseEmitter.completeWithError(error);
            }

            @Override
            public void onCompleteResponse(ChatResponse response) {
                sseEmitter.complete();
            }
        });
        return sseEmitter;
    }
}
```

