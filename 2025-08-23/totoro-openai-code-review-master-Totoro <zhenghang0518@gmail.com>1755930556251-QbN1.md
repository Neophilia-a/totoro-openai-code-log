# Totoro项目： OpenAi 代码评审.
### 😀代码评分：60
#### 😀代码逻辑与目的：
该代码块是一个Java类，用于处理代码审查的逻辑，包括代码的检出、评审、日志记录和消息推送。代码使用了OpenAI API进行代码审查，并通过微信API发送通知。

#### 🎯修改建议：
1. 移除不必要的注释和未使用的代码。
2. 使用更安全的方式来处理HTTP请求。
3. 优化异常处理，确保代码的健壮性。
4. 使用日志框架而不是手动处理日志。
5. 优化文件操作和Git操作，减少资源消耗。

#### 💻修改后的代码：
```java
package com.totoro.sdk;

import com.alibaba.fastjson2.JSON;
import com.totoro.sdk.domain.model.Message;
import com.totoro.sdk.domain.service.impl.OpenAiCodeReviewService;
import com.totoro.sdk.infrastructure.git.GitCommand;
import com.totoro.sdk.infrastructure.openai.IOpenAI;
import com.totoro.sdk.infrastructure.openai.dto.ChatCompletionRequestDTO;
import com.totoro.sdk.infrastructure.openai.dto.ChatCompletionSyncResponseDTO;
import com.totoro.sdk.infrastructure.openai.impl.ChatGLM;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Date;

public class OpenAiCodeReview {
    private static final Logger logger = LoggerFactory.getLogger(OpenAiCodeReview.class);

    public static String codeReview(String code) throws IOException {
        String apiKeySecret = "908dcf3234214bb9a2f3db50209ed295.5g2085L8TFIa7TZS";
        String token = BearerTokenUtils.getToken(apiKeySecret);

        URL url = new URL("https://open.bigmodel.cn/api/paas/v4/chat/completions");
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();

        connection.setRequestMethod("POST");
        connection.setRequestProperty("Authorization", "Bearer " + token);
        connection.setRequestProperty("Content-Type", "application/json");
        connection.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)");
        connection.setDoOutput(true);

        ChatCompletionRequestDTO requestDTO = new ChatCompletionRequestDTO();
        requestDTO.setModel(Model.GLM_4_FLASH.getCode());
        requestDTO.setMessages(new ArrayList<ChatCompletionRequestDTO.Prompt>() {{
            add(new ChatCompletionRequestDTO.Prompt("user", "你是一个高级编程架构师，精通各类场景方案、架构设计和编程语言请，请您根据git diff记录，对代码做出评审。"));
            add(new ChatCompletionRequestDTO.Prompt("user", code));
        }});

        try (OutputStream os = connection.getOutputStream()) {
            byte[] input = JSON.toJSONString(requestDTO).getBytes(StandardCharsets.UTF_8);
            os.write(input, 0, input.length);
        }

        int responseCode = connection.getResponseCode();
        logger.info("Response Code: {}", responseCode);

        StringBuilder content = new StringBuilder();
        try (BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream(), StandardCharsets.UTF_8))) {
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                content.append(inputLine);
            }
        }

        connection.disconnect();

        ChatCompletionSyncResponseDTO response = JSON.parseObject(content.toString(), ChatCompletionSyncResponseDTO.class);
        return response.getChoices().get(0).getMessage().getContent();
    }
}
```

#### 🤔问题点：
1. 代码中存在大量的注释和未使用的代码。
2. 使用了手动处理HTTP请求，存在安全风险。
3. 异常处理不够完善，可能会导致程序崩溃。
4. 使用了简单的文件操作和Git操作，效率低下。

#### 🎯修改建议：
1. 移除未使用的代码和注释。
2. 使用成熟的HTTP客户端库，如Apache HttpClient或OkHttp，来处理HTTP请求。
3. 完善异常处理，确保程序的健壮性。
4. 使用Git客户端库，如JGit，来处理Git操作，提高效率。