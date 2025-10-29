---
title: "Lab: Spring AI Chat Application"
author: "Goetz Botterweck"
date: "2025-10-22"
documentclass: article
geometry: margin=1in
colorlinks: true
linkcolor: blue
codeblock-style: github
syntax-highlighting: true
toc: true
toc-depth: 3
---

**Module:** CSU33012 Software Engineering  
**Academic Year:** 2025/2026  
**Duration:** 2 hours (4 x 30-minute phases)  

---

## Learning Objectives

- Intermediate
  - Integrate AI capabilities into Spring Boot applications using Spring AI
  - Build REST APIs for AI-enabled chat functionality
  - Apply error handling for AI services
- Advanced
  - Implement streaming responses with Server-Sent Events (SSE)
  - Manage conversation context and memory
  - Design and use prompt templates for different use cases
  - Support multiple AI model providers

---

## Prerequisites

### Project Repository and Folder

* Create a Coder workspace based on the CSU33012 template (can also work locally).
* Create a new project "aichat" 
  * Go to https://gitlab.scss.tcd.ie/projects/new create new project aichat
    * Private
    * Initialize with Readme = [x]
  * Clone the project so that you get a subdirectory `aichat` below `/home/username/workspaces/` .
  * Open that new folder in Visual Studio Code (File > Open Folder)

```bash
cd ~/workspaces
git clone https://gitlab.scss.tcd.ie/$USER/aichat.git
```

Since you are cloning from your own repository and not startercode provided by another user, you do not need to switch git remotes now.

### AI API Keys

You will need an API key from at least one AI provider. This lab supports OpenAI (GPT), Anthropic (Claude), and Google Gemini. Choose one to get started.

Keep your API keys secure and never commit them to version control!

** IMPORTANT: The instructions below are provided AS-IS and might contain mistakes or code lead to significant costs. You are responsible for any API usage and resulting costs!**

---

#### Option 1: OpenAI (GPT Models) - Recommended for this lab

OpenAI provides the GPT family of models (gtp-5, gpt-5-mini, etc.), More details at https://platform.openai.com/docs/models. For pricing see https://platform.openai.com/docs/pricing.

**Step 1: Create an OpenAI Account**

1. Go to https://platform.openai.com/
2. Click "Sign up" or "Log in" if you already have an account
3. Complete the registration process with your email
4. Verify your email address

**Step 2: Add Payment Method**

1. OpenAI requires a payment method for API access (even for small usage)
2. Navigate to Settings (cog wheel symbol) → Billing → Payment methods
3. Add a credit/debit card
4. Consider setting usage limits and "Pay as you go" to avoid unexpected charges:
   - Go to Settings → Billing → Usage limits
   - Set a monthly budget (e.g., $5-10 for lab purposes)


**Step 3: Generate API Key**

1. Navigate to https://platform.openai.com/api-keys
2. Click "+ Create new secret key"
3. Give it a name (e.g., "Spring AI Lab")
4. Set permissions: "All" or "Restricted" (for this lab, "All" is fine)
5. Click "Create secret key"
6. **IMPORTANT:** Copy the key immediately - you won't be able to see it again!
   - The key starts with `sk-proj-` or `sk-`
7. Store it securely (you'll use it in your `.env` file).

**Step 4: Verify Account Credits**

1. Go to Settings → Billing → Overview
2. Check if you have any free credits
3. Your account needs credits or a payment method to work

**Expected Costs:**

- gpt-5-mini: ~$0.25 per million input tokens, ~$2.00 per million output tokens
- gpt-5-nano: ~$0.05 per million input tokens, ~$0.40 per million output tokens
- A typical chat message uses 100-1000 tokens

---

#### Option 2: Anthropic (Claude Models)

Anthropic provides Claude models (Claude 3.5 Sonnet, Claude 3 Opus, etc.)

**Step 1: Create an Anthropic Account**

1. Go to https://console.anthropic.com/
2. Click "Sign up" or use "Continue with Google"
3. Complete the registration process
4. Verify your email address

**Step 2: Access the Console**

1. Log in to https://console.anthropic.com/
2. You'll land on the Console dashboard
3. New accounts may receive free credits ($5 typically)

**Step 3: Generate API Key**

1. Click on "API Keys" in the left sidebar
   - Or navigate directly to https://console.anthropic.com/settings/keys
2. Click "Create Key" or "+ Create Key"
3. Give it a name (e.g., "Spring AI Lab")
4. Click "Create Key"
5. **IMPORTANT:** Copy the key immediately - you won't be able to see it again!
   - The key starts with `sk-ant-`
6. Store it securely

**Step 4: Add Payment Method (if needed)**

1. Go to Settings → Billing
2. Add a payment method if you've exhausted free credits
3. Set up usage notifications to monitor spending

**Step 5: Check Credit Balance**

1. Go to Settings → Billing
2. View your current credit balance
3. Set usage limits if desired

**Expected Costs:**

- https://claude.com/pricing#api
- Monitor cost at https://console.anthropic.com/workspaces/default/cost

- Claude 4.5 Sonnet: ~$3 per million input tokens, ~$15 per million output tokens
- Claude 4.5 Haiku: ~$1 per million input tokens, ~$5 per million output tokens

---

#### Option 3: Google Gemini

Google provides Gemini models (Gemini 2.5 Pro, Gemini 2.5 Flash, Gemini 2.5 Flash-lite, etc.)

**Step 1: Access Google AI Studio**
1. Go to https://aistudio.google.com/
2. Sign in with your Google account
3. Accept the Terms of Service

**Step 2: Get API Key**

1. Click "Get API Key" in the left sidebar, https://aistudio.google.com/api-keys
2. Click "Create API Key"
3. Select a Google Cloud project or create a new one:
   - If creating new: "Create API key in new project"
   - If using existing: Select your project from the dropdown
4. Your API key will be generated immediately
5. **IMPORTANT:** Copy the key - store it securely
   - The key format: `AIza...`

**Step 3: Enable Billing (for higher quotas)**

1. For free tier: You get limited requests per minute
2. For production use:
   - Go to https://console.cloud.google.com/
   - Select your project
   - Navigate to "Billing"
   - Enable billing and add a payment method

**Step 4: Monitor Usage**

1. Go to https://console.cloud.google.com/
2. Navigate to "APIs & Services" → "Credentials"
3. Click on your API key to view usage statistics
4. Set up quotas and limits if needed

**Expected Costs:**

- Gemini 2.5 Flash: Free tier available with rate limits
- Gemini 2.5 Pro: Paid tier with competitive pricing
- Free tier is sufficient for this lab

---

#### Option 4: Ollama (Local Models) - Free but requires setup

Run AI models locally on your machine - completely free but requires more setup.

Attention:

- We did not really test this variant, use at your own risk. 
- You cannot assume that you can use Ollama for your main application / submission, since the evaluators might not have it available.

**Step 1: Install Ollama**


1. Go to https://ollama.ai/
2. Download for your operating system:
   - macOS: Download .dmg installer
   - Linux: `curl -fsSL https://ollama.ai/install.sh | sh`
   - Windows: Download .exe installer

**Step 2: Install a Model**

```bash
# Install Llama 2 (7B parameters, ~4GB)
ollama pull llama2

# Or install smaller/larger models
ollama pull llama2:13b    # Larger, more capable
ollama pull mistral       # Alternative model
ollama pull codellama     # Optimized for code
```

**Step 3: Verify Installation**

```bash
# Check if Ollama is running
ollama list

# Test the model
ollama run llama2 "Hello, how are you?"
```

**Step 4: Configure Spring AI for Ollama**

No API key needed! Update your `application.properties`:
```properties
spring.ai.ollama.base-url=http://localhost:11434
spring.ai.ollama.chat.options.model=llama2
```

---

### Which Provider Should You Choose?

| Provider | Best For | Cost | Setup Difficulty |
|----------|----------|------|------------------|
| **OpenAI** | General use, best documentation | Low-Medium | Easy |
| **Anthropic** | Long conversations, coding | Medium | Easy |
| **Gemini** | Free tier, experimentation | Free-Low | Easy |
| **Ollama** | Privacy, offline use | Free | Medium |

**Recommendation for beginners:** Start with **Google Gemini** (free tier) or **OpenAI** (best documentation and Spring AI support).

---

### Storing Your API Key Securely

**Never commit API keys to Git!**

**Step 1: Create `.env` file in project root**
```bash
cd ~/workspaces/aichat
touch .env
```

**Step 2: Add your API key to `.env`**

Add the following lines to your .env fikle

For OpenAI:
```bash
export OPENAI_API_KEY=sk-proj-your-key-here
```

For Anthropic:
```bash
export ANTHROPIC_API_KEY=sk-ant-your-key-here
```

For Google Gemini:
```bash
export GOOGLE_API_KEY=AIza-your-key-here
```

**Step 3: Add `.env` to `.gitignore`**
```bash
echo ".env" >> .gitignore
echo "*.env" >> .gitignore
```

**Step 4: Load environment variables**
```bash
source .env
```

**Step 5: Verify (without exposing the key)**
```bash
# Check that variable is set (shows only first few characters)
echo ${OPENAI_API_KEY:0:8}
```

---

#### For Windows 11 Users

Windows handles environment variables differently. Here are some options (not tested):

---

##### Method 1: Using PowerShell with .env file (Recommended)

**Step 1: Create `.env` file in project root**

Open PowerShell in your project directory:
```powershell
cd C:\Users\YourUsername\workspaces\aichat
New-Item .env -ItemType File
```

Or create it manually:

- Right-click in project folder → New → Text Document
- Rename to `.env` (remove .txt extension)
- Windows will warn about changing extension - click "Yes"

**Step 2: Edit `.env` file**

Open `.env` in VS Code and add (without "export"):

For OpenAI:
```
OPENAI_API_KEY=sk-proj-your-key-here
```

For Anthropic:
```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

For Google Gemini:
```
GOOGLE_API_KEY=AIza-your-key-here
```

**Step 3: Add `.env` to `.gitignore`**

Open PowerShell:
```powershell
Add-Content .gitignore "`n.env"
Add-Content .gitignore "*.env"
```

Or manually edit `.gitignore` and add:
```
.env
*.env
```

**Step 4: Create PowerShell script to load environment variables**

Create `load-env.ps1` in your project root:
```powershell
# load-env.ps1
Get-Content .env | ForEach-Object {
    if ($_ -match '^\s*([^#][^=]*?)\s*=\s*(.*?)\s*$') {
        $name = $matches[1]
        $value = $matches[2]
        Set-Item -Path "env:$name" -Value $value
        Write-Host "Set $name" -ForegroundColor Green
    }
}
```

**Step 5: Load environment variables before running**

In PowerShell:
```powershell
# You may need to enable script execution first (one time only)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Load the environment variables
.\load-env.ps1

# Run your application
mvn spring-boot:run
```

**Step 6: Verify**
```powershell
# Check that variable is set
$env:OPENAI_API_KEY.Substring(0,8)
```

---

##### Method 2: Set System Environment Variables (Persistent)

This method sets environment variables permanently for your Windows user account.

**Step 1: Open Environment Variables Dialog**

Option A - Using Run Dialog:

1. Press `Win + R`
2. Type `sysdm.cpl` and press Enter
3. Click "Environment Variables" button

Option B - Using Settings:

1. Press `Win + I` to open Settings
2. Search for "environment variables"
3. Click "Edit environment variables for your account"

Option C - Using Command:

1. Open PowerShell
2. Run: `rundll32.exe sysdm.cpl,EditEnvironmentVariables`

**Step 2: Add New User Variable**

1. Under "User variables", click "New..."
2. Variable name: `OPENAI_API_KEY`
3. Variable value: `sk-proj-your-key-here`
4. Click "OK"

Repeat for other providers:

- Variable name: `ANTHROPIC_API_KEY`, Value: your Anthropic key
- Variable name: `GOOGLE_API_KEY`, Value: your Google key

**Step 3: Apply Changes**

1. Click "OK" on all dialogs
2. **Important:** Restart your terminal/IDE/VS Code for changes to take effect
3. Restart any running applications

**Step 4: Verify**

Open a new PowerShell window:
```powershell
$env:OPENAI_API_KEY.Substring(0,8)
```

Or in Command Prompt (cmd):
```cmd
echo %OPENAI_API_KEY:~0,8%
```

---

### Testing Your API Key

Before starting the lab, verify your API key works:

**For OpenAI:**

List models
```bash
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-5-nano",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

**For Anthropic:**

```bash
curl https://api.anthropic.com/v1/models \
     --header "x-api-key: $ANTHROPIC_API_KEY" \
     --header "anthropic-version: 2023-06-01"
```

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-haiku-4-5-20251001",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

**For Gemini:**

List models
```bash
curl "https://generativelanguage.googleapis.com/v1/models?key=$GOOGLE_API_KEY"
```

```bash
curl "https://generativelanguage.googleapis.com/v1/models/gemini-2.5-flash:generateContent?key=$GOOGLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts":[{"text": "Hello!"}]}]
  }'
```

If you get a valid JSON response, your API key is working correctly!

**Reminder:** Check if .env is showing up in changed files to be added to your commit. That should not be the case!

---

## Project Setup

### Spring Initializr

* Command Palette (Ctrl-Shift-P)
* Spring Initializr: Create a Maven Project
  * Spring Boot, highest stable 3.x
  * group id: ie.tcd.scss
  * artifact id: aichat
  * packaging: Jar
  * java version: Java 21
  * dependencies: Spring Web, Lombok, Spring Boot DevTools and **ONE** of: Spring AI OpenAI, Spring AI Anthropic, Spring AI Vertex Gemini
    * If you pick more than one, you will get error messages or need to annotate the code, since Spring AI cannot decide which model to use.
    * (you can also install more than one, but then need to comment them out in the `pom.xml` file)
  * generate into /home/user/workspaces/   (** !!! NOT /home/user/workspaces/aichat !!! **)
* confirm overwrite "a folder already exists, continue?"
* open folder `/home/user/workspaces/aichat/` (File > Open Folder)

**IMPORTANT: Before you commit, make sure that .env is added to gitignore**

* git commit
  * Commit and push changes to git 
  * Check that changes have reached gitlab server

### Add Pipeline Configuration
(can skip or do later, test might fail due to missing API keys)

In the project root create a new file `.gitlab-ci.yml`

```yaml
image: maven:3.9.11-eclipse-temurin-21-alpine

cache:
  paths:
    - .m2/repository

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"    

stages:
  - build
  - test

build:
  stage: build
  tags:
    - docker 
  script:
    - mvn clean compile
  artifacts:
    paths:
      - target/

test:
  stage: test
  tags:
    - docker 
  script:
    - mvn test
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
```

* Commit ("added pipeline configuration") and push
* Check pipeline status on gitlab server


### Configure API Keys (application.properties)

Create `src/main/resources/application.properties`:

```properties
# Server Configuration
server.port=8080

# OpenAI Configuration
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-5-nano
spring.ai.openai.chat.options.temperature=1.0

# Logging
logging.level.org.springframework.ai=DEBUG
```

**Create a local environment file for API key:**
(can skip if you have done that above)

Create `.env` file in project root (add to .gitignore!):

```bash
export OPENAI_API_KEY=your-api-key-here
```

Add to `.gitignore`:
```
.env
*.env
```

Load environment before running:
```bash
source .env
mvn spring-boot:run
```

* Commit ("API configuration in application.properties") and push
* Check pipeline status

---

### Create Project Structure

```
aichat/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/ie/tcd/scss/aichat/
    │   │   ├── AiChatApplication.java
    │   │   ├── dto/
    │   │   │   ├── ChatRequest.java
    │   │   │   └── ChatResponse.java
    │   │   ├── service/
    │   │   │   └── ChatService.java
    │   │   └── controller/
    │   │       └── ChatController.java
    │   └── resources/
    │       └── application.properties
    └── test/
        └── java/ie/tcd/scss/aichat/
```

### Create Main Application Class

Create `src/main/java/ie/tcd/scss/aichat/AiChatApplication.java`:

```java
package ie.tcd.scss.aichat;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AiChatApplication {
    public static void main(String[] args) {
        SpringApplication.run(AiChatApplication.class, args);
    }
}
```

---

## 🔵 PHASE 1: Basic Chat API (30 minutes)

**Goal:** Create a minimal working AI chat API with simple request/response

### What You'll Build

- Simple DTO classes for requests and responses
- Basic ChatService using Spring AI
- REST controller with single chat endpoint
- Synchronous (blocking) responses

---

### Step 1.1: Create DTO Classes

Create `src/main/java/ie/tcd/scss/aichat/dto/ChatRequest.java`:

```java
package ie.tcd.scss.aichat.dto;

import lombok.Data;

@Data
public class ChatRequest {
    private String message;
}
```

Create `src/main/java/ie/tcd/scss/aichat/dto/ChatResponse.java`:

```java
package ie.tcd.scss.aichat.dto;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class ChatResponse {
    private String response;
    private String model;
}
```

---

### Step 1.2: Create Chat Service

Create `src/main/java/ie/tcd/scss/aichat/service/ChatService.java`:

```java
package ie.tcd.scss.aichat.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.stereotype.Service;

@Service
public class ChatService {
    
    private final ChatClient chatClient;
    
    public ChatService(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
    }
    
    public String chat(String userMessage) {
        return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
    }
}
```

---

### Step 1.3: Create REST Controller

Create `src/main/java/ie/tcd/scss/aichat/controller/ChatController.java`:

```java
package ie.tcd.scss.aichat.controller;

import ie.tcd.scss.aichat.dto.ChatRequest;
import ie.tcd.scss.aichat.dto.ChatResponse;
import ie.tcd.scss.aichat.service.ChatService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/chat")
@RequiredArgsConstructor
public class ChatController {
    
    private final ChatService chatService;
    
    @PostMapping
    public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest request) {
        String response = chatService.chat(request.getMessage());
        return ResponseEntity.ok(new ChatResponse(response, "gpt-5-nano"));
    }
}
```

---

### ✅ Phase 1 Checkpoint: Test Basic Chat

#### Start Application
```bash
source .env
mvn clean install
mvn spring-boot:run
```

#### Test Chat Endpoint
```bash
curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What is Spring Boot?"
  }'
```

**Expected Response:**
```json
{
  "response": "Spring Boot is a framework that simplifies the development of Java applications...",
  "model": "gpt-5-mini"
}
```

#### Test More Examples

**Simple question:**
```bash
curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Explain AI in one sentence"
  }'
```

**Code question:**
```bash
curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Write a Java function to reverse a string"
  }'
```

**🎉 Phase 1 Complete!** You have a working AI chat API!

* Commit and push: "Phase 1: Basic chat API complete"

---

## 🟢 PHASE 2: Streaming Responses (30 minutes)

**Goal:** Add streaming responses using Server-Sent Events (SSE) for better UX

### What You'll Build

- Streaming chat endpoint using SSE
- Real-time token streaming from AI
- Improved user experience for long responses

---

### Step 2.1: Add Streaming to ChatService

Update `src/main/java/ie/tcd/scss/aichat/service/ChatService.java`:

```java
package ie.tcd.scss.aichat.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

@Service
public class ChatService {
    
    private final ChatClient chatClient;
    
    public ChatService(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
    }
    
    public String chat(String userMessage) {
        return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
    }
    
    public Flux<String> chatStream(String userMessage) {
        return chatClient.prompt()
                .user(userMessage)
                .stream()
                .content();
    }
}
```

---

### Step 2.2: Add Streaming Endpoint to Controller

Update `src/main/java/ie/tcd/scss/aichat/controller/ChatController.java`:

```java
package ie.tcd.scss.aichat.controller;

import ie.tcd.scss.aichat.dto.ChatRequest;
import ie.tcd.scss.aichat.dto.ChatResponse;
import ie.tcd.scss.aichat.service.ChatService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/api/chat")
@RequiredArgsConstructor
public class ChatController {
    
    private final ChatService chatService;
    
    @PostMapping
    public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest request) {
        String response = chatService.chat(request.getMessage());
        return ResponseEntity.ok(new ChatResponse(response, "gpt-4o-mini"));
    }
    
    @PostMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chatStream(@RequestBody ChatRequest request) {
        return chatService.chatStream(request.getMessage());
    }
}
```

---

### ✅ Phase 2 Checkpoint: Test Streaming

#### Restart Application
```bash
mvn clean install
mvn spring-boot:run
```

#### Test Streaming Endpoint
```bash
curl -X POST http://localhost:8080/api/chat/stream \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Write a short story about a robot learning to paint"
  }'
```

You should see the response stream in real-time, word by word!

#### Compare Streaming vs Non-Streaming

**Non-streaming (blocks until complete):**
```bash
time curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Explain machine learning in detail"}'
```

**Streaming (starts immediately):**
```bash
time curl -X POST http://localhost:8080/api/chat/stream \
  -H "Content-Type: application/json" \
  -d '{"message": "Explain machine learning in detail"}'
```

**🎉 Phase 2 Complete!** You can now stream AI responses in real-time!

* Commit and push: "Phase 2: Streaming responses with SSE"

---

## 🟡 PHASE 3: Conversation Memory & Context (30 minutes)

**Goal:** Add conversation history and context management

### What You'll Build

- Conversation session management
- Message history storage (in-memory)
- Context-aware responses

---

### Step 3.1: Create Conversation Models

Create `src/main/java/ie/tcd/scss/aichat/dto/ConversationRequest.java`:

```java
package ie.tcd.scss.aichat.dto;

import lombok.Data;

@Data
public class ConversationRequest {
    private String sessionId;
    private String message;
}
```

Create `src/main/java/ie/tcd/scss/aichat/dto/Message.java`:

```java
package ie.tcd.scss.aichat.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@AllArgsConstructor
public class Message {
    private String role; // "user" or "assistant"
    private String content;
    private LocalDateTime timestamp;
}
```

---

### Step 3.2: Create Conversation Service

Create `src/main/java/ie/tcd/scss/aichat/service/ConversationService.java`:

```java
package ie.tcd.scss.aichat.service;

import ie.tcd.scss.aichat.dto.Message;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.messages.SystemMessage;
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.ai.chat.messages.AssistantMessage;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class ConversationService {
    
    private final ChatClient chatClient;
    private final Map<String, List<Message>> conversations = new ConcurrentHashMap<>();
    
    public ConversationService(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
    }
    
    public String chatWithContext(String sessionId, String userMessage) {
        // Get or create conversation history
        List<Message> history = conversations.computeIfAbsent(
            sessionId, 
            k -> new ArrayList<>()
        );
        
        // Add user message to history
        history.add(new Message("user", userMessage, LocalDateTime.now()));
        
        // Build messages list for AI
        List<org.springframework.ai.chat.messages.Message> messages = new ArrayList<>();
        
        // Add system message
        messages.add(new SystemMessage("You are a helpful AI assistant. Maintain context from previous messages in this conversation."));
        
        // Add conversation history
        for (Message msg : history) {
            if (msg.getRole().equals("user")) {
                messages.add(new UserMessage(msg.getContent()));
            } else if (msg.getRole().equals("assistant")) {
                messages.add(new AssistantMessage(msg.getContent()));
            }
        }
        
        // Get AI response
        String response = chatClient.prompt(new Prompt(messages))
                .call()
                .content();
        
        // Add assistant response to history
        history.add(new Message("assistant", response, LocalDateTime.now()));
        
        return response;
    }
    
    public List<Message> getConversationHistory(String sessionId) {
        return conversations.getOrDefault(sessionId, Collections.emptyList());
    }
    
    public void clearConversation(String sessionId) {
        conversations.remove(sessionId);
    }
    
    public Set<String> getActiveSessions() {
        return conversations.keySet();
    }
}
```

---

### Step 3.3: Add Conversation Endpoints to Controller

Update `src/main/java/ie/tcd/scss/aichat/controller/ChatController.java`:

```java
package ie.tcd.scss.aichat.controller;

import ie.tcd.scss.aichat.dto.ChatRequest;
import ie.tcd.scss.aichat.dto.ChatResponse;
import ie.tcd.scss.aichat.dto.ConversationRequest;
import ie.tcd.scss.aichat.dto.Message;
import ie.tcd.scss.aichat.service.ChatService;
import ie.tcd.scss.aichat.service.ConversationService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

import java.util.List;
import java.util.Map;
import java.util.Set;

@RestController
@RequestMapping("/api/chat")
@RequiredArgsConstructor
public class ChatController {
    
    private final ChatService chatService;
    private final ConversationService conversationService;
    
    @PostMapping
    public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest request) {
        String response = chatService.chat(request.getMessage());
        return ResponseEntity.ok(new ChatResponse(response, "gpt-4o-mini"));
    }
    
    @PostMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chatStream(@RequestBody ChatRequest request) {
        return chatService.chatStream(request.getMessage());
    }
    
    @PostMapping("/conversation")
    public ResponseEntity<ChatResponse> conversationChat(@RequestBody ConversationRequest request) {
        String response = conversationService.chatWithContext(
            request.getSessionId(), 
            request.getMessage()
        );
        return ResponseEntity.ok(new ChatResponse(response, "gpt-4o-mini"));
    }
    
    @GetMapping("/conversation/{sessionId}/history")
    public ResponseEntity<List<Message>> getHistory(@PathVariable String sessionId) {
        return ResponseEntity.ok(conversationService.getConversationHistory(sessionId));
    }
    
    @DeleteMapping("/conversation/{sessionId}")
    public ResponseEntity<Map<String, String>> clearConversation(@PathVariable String sessionId) {
        conversationService.clearConversation(sessionId);
        return ResponseEntity.ok(Map.of("message", "Conversation cleared"));
    }
    
    @GetMapping("/conversations")
    public ResponseEntity<Set<String>> getActiveSessions() {
        return ResponseEntity.ok(conversationService.getActiveSessions());
    }
}
```

---

### ✅ Phase 3 Checkpoint: Test Conversation Context

#### Restart Application
```bash
mvn clean install
mvn spring-boot:run
```

#### Test Conversation with Context

**Start a conversation:**
```bash
curl -X POST http://localhost:8080/api/chat/conversation \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "user123",
    "message": "My name is Alice and I love programming"
  }'
```

**Continue the conversation (AI should remember your name):**
```bash
curl -X POST http://localhost:8080/api/chat/conversation \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "user123",
    "message": "What did I just tell you my name was?"
  }'
```

**Ask a follow-up (AI should remember context):**
```bash
curl -X POST http://localhost:8080/api/chat/conversation \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "user123",
    "message": "What do I love?"
  }'
```

#### View Conversation History
```bash
curl http://localhost:8080/api/chat/conversation/user123/history
```

#### Clear Conversation
```bash
curl -X DELETE http://localhost:8080/api/chat/conversation/user123
```

#### View Active Sessions
```bash
curl http://localhost:8080/api/chat/conversations
```

**🎉 Phase 3 Complete!** Your AI can now maintain conversation context!

* Commit and push: "Phase 3: Conversation memory and context"

---

## 🟣 PHASE 4: Prompt Templates & Multi-Model Support (30 minutes)

**Goal:** Add prompt engineering templates and support for multiple AI providers

### What You'll Build

- Reusable prompt templates for different personas
- Configuration for multiple AI models
- Dynamic model selection

---

### Step 4.1: Create Prompt Template Service

Create `src/main/java/ie/tcd/scss/aichat/service/PromptTemplateService.java`:

```java
package ie.tcd.scss.aichat.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.stereotype.Service;

import java.util.Map;

@Service
public class PromptTemplateService {
    
    private final ChatClient chatClient;
    
    public PromptTemplateService(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
    }
    
    public String chatWithPersona(String persona, String userMessage) {
        String systemPrompt = getSystemPromptForPersona(persona);
        
        return chatClient.prompt()
                .system(systemPrompt)
                .user(userMessage)
                .call()
                .content();
    }
    
    public String chatWithTemplate(String templateName, Map<String, Object> variables) {
        String template = getTemplate(templateName);
        String prompt = fillTemplate(template, variables);
        
        return chatClient.prompt()
                .user(prompt)
                .call()
                .content();
    }
    
    private String getSystemPromptForPersona(String persona) {
        return switch (persona.toLowerCase()) {
            case "teacher" -> 
                "You are a patient and knowledgeable teacher. " +
                "Explain concepts clearly with examples. " +
                "Break down complex topics into simple steps.";
            
            case "code_reviewer" -> 
                "You are an experienced code reviewer. " +
                "Analyze code for bugs, performance issues, and best practices. " +
                "Provide constructive feedback with specific suggestions.";
            
            case "translator" -> 
                "You are a professional translator. " +
                "Translate text accurately while preserving tone and context. " +
                "Explain cultural nuances when relevant.";
            
            case "creative_writer" -> 
                "You are a creative writer with vivid imagination. " +
                "Write engaging stories with rich descriptions. " +
                "Use literary techniques to captivate readers.";
            
            default -> 
                "You are a helpful AI assistant.";
        };
    }
    
    private String getTemplate(String templateName) {
        return switch (templateName) {
            case "code_explanation" -> 
                "Explain the following code:\n\n{code}\n\n" +
                "Focus on: {focus}";
            
            case "email_draft" -> 
                "Draft a {tone} email about: {topic}\n" +
                "Recipient: {recipient}";
            
            case "summary" -> 
                "Summarize the following text in {length} words:\n\n{text}";
            
            case "brainstorm" -> 
                "Generate {count} creative ideas for: {topic}\n" +
                "Target audience: {audience}";
            
            default -> "{message}";
        };
    }
    
    private String fillTemplate(String template, Map<String, Object> variables) {
        String result = template;
        for (Map.Entry<String, Object> entry : variables.entrySet()) {
            result = result.replace("{" + entry.getKey() + "}", 
                                   entry.getValue().toString());
        }
        return result;
    }
}
```

---

### Step 4.2: Add Multi-Model Configuration

Update `src/main/resources/application.properties`:

```properties
# Server Configuration
server.port=8080

# OpenAI Configuration (default)
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-4o-mini
spring.ai.openai.chat.options.temperature=0.7

# Alternative: Anthropic Claude Configuration (uncomment to use)
#spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}
#spring.ai.anthropic.chat.options.model=claude-3-5-sonnet-20241022
#spring.ai.anthropic.chat.options.temperature=0.7

# Alternative: Ollama Configuration (for local models)
#spring.ai.ollama.base-url=http://localhost:11434
#spring.ai.ollama.chat.options.model=llama2

# Logging
logging.level.org.springframework.ai=DEBUG
```

---

### Step 4.3: Create Model Configuration Service

Create `src/main/java/ie/tcd/scss/aichat/service/ModelService.java`:

```java
package ie.tcd.scss.aichat.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.ai.openai.OpenAiChatModel;
import org.springframework.ai.openai.OpenAiChatOptions;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.Map;

@Service
public class ModelService {
    
    private final ChatModel defaultChatModel;
    
    @Value("${spring.ai.openai.api-key:}")
    private String openAiApiKey;
    
    public ModelService(ChatModel chatModel) {
        this.defaultChatModel = chatModel;
    }
    
    public String chatWithModel(String modelName, String userMessage, Double temperature) {
        ChatClient.Builder builder = ChatClient.builder(defaultChatModel);
        
        // Configure options based on model
        Map<String, Object> options = Map.of(
            "model", modelName,
            "temperature", temperature != null ? temperature : 0.7
        );
        
        return builder.build()
                .prompt()
                .user(userMessage)
                .options(OpenAiChatOptions.builder()
                        .withModel(modelName)
                        .withTemperature(temperature != null ? temperature : 0.7)
                        .build())
                .call()
                .content();
    }
    
    public Map<String, String> getAvailableModels() {
        return Map.of(
            "gpt-4o", "GPT-4o - Most capable",
            "gpt-4o-mini", "GPT-4o Mini - Fast and efficient",
            "gpt-4-turbo", "GPT-4 Turbo - Balanced performance",
            "gpt-3.5-turbo", "GPT-3.5 Turbo - Legacy model"
        );
    }
}
```

---

### Step 4.4: Add Template and Model Endpoints

Create `src/main/java/ie/tcd/scss/aichat/controller/AdvancedChatController.java`:

```java
package ie.tcd.scss.aichat.controller;

import ie.tcd.scss.aichat.dto.ChatResponse;
import ie.tcd.scss.aichat.service.ModelService;
import ie.tcd.scss.aichat.service.PromptTemplateService;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/api/advanced")
@RequiredArgsConstructor
public class AdvancedChatController {
    
    private final PromptTemplateService promptTemplateService;
    private final ModelService modelService;
    
    @PostMapping("/persona")
    public ResponseEntity<ChatResponse> chatWithPersona(@RequestBody PersonaRequest request) {
        String response = promptTemplateService.chatWithPersona(
            request.getPersona(), 
            request.getMessage()
        );
        return ResponseEntity.ok(new ChatResponse(response, "gpt-4o-mini"));
    }
    
    @PostMapping("/template")
    public ResponseEntity<ChatResponse> chatWithTemplate(@RequestBody TemplateRequest request) {
        String response = promptTemplateService.chatWithTemplate(
            request.getTemplateName(), 
            request.getVariables()
        );
        return ResponseEntity.ok(new ChatResponse(response, "gpt-4o-mini"));
    }
    
    @PostMapping("/model")
    public ResponseEntity<ChatResponse> chatWithModel(@RequestBody ModelRequest request) {
        String response = modelService.chatWithModel(
            request.getModel(),
            request.getMessage(),
            request.getTemperature()
        );
        return ResponseEntity.ok(new ChatResponse(response, request.getModel()));
    }
    
    @GetMapping("/models")
    public ResponseEntity<Map<String, String>> getAvailableModels() {
        return ResponseEntity.ok(modelService.getAvailableModels());
    }
    
    @Data
    public static class PersonaRequest {
        private String persona;
        private String message;
    }
    
    @Data
    public static class TemplateRequest {
        private String templateName;
        private Map<String, Object> variables;
    }
    
    @Data
    public static class ModelRequest {
        private String model;
        private String message;
        private Double temperature;
    }
}
```

---

### ✅ Phase 4 Checkpoint: Test Advanced Features

#### Restart Application
```bash
mvn clean install
mvn spring-boot:run
```

#### Test Different Personas

**Teacher persona:**
```bash
curl -X POST http://localhost:8080/api/advanced/persona \
  -H "Content-Type: application/json" \
  -d '{
    "persona": "teacher",
    "message": "Explain recursion"
  }'
```

**Code reviewer persona:**
```bash
curl -X POST http://localhost:8080/api/advanced/persona \
  -H "Content-Type: application/json" \
  -d '{
    "persona": "code_reviewer",
    "message": "Review this code: public void test() { while(true) { System.out.println(\"test\"); } }"
  }'
```

**Creative writer persona:**
```bash
curl -X POST http://localhost:8080/api/advanced/persona \
  -H "Content-Type: application/json" \
  -d '{
    "persona": "creative_writer",
    "message": "Write a short poem about coding"
  }'
```

#### Test Prompt Templates

**Code explanation template:**
```bash
curl -X POST http://localhost:8080/api/advanced/template \
  -H "Content-Type: application/json" \
  -d '{
    "templateName": "code_explanation",
    "variables": {
      "code": "def factorial(n): return 1 if n <= 1 else n * factorial(n-1)",
      "focus": "how recursion works"
    }
  }'
```

**Email draft template:**
```bash
curl -X POST http://localhost:8080/api/advanced/template \
  -H "Content-Type: application/json" \
  -d '{
    "templateName": "email_draft",
    "variables": {
      "tone": "professional",
      "topic": "project deadline extension",
      "recipient": "project manager"
    }
  }'
```

**Brainstorm template:**
```bash
curl -X POST http://localhost:8080/api/advanced/template \
  -H "Content-Type: application/json" \
  -d '{
    "templateName": "brainstorm",
    "variables": {
      "count": "5",
      "topic": "mobile app features",
      "audience": "university students"
    }
  }'
```

#### Test Different Models

**List available models:**
```bash
curl http://localhost:8080/api/advanced/models
```

**Use GPT-4o (more capable but slower):**
```bash
curl -X POST http://localhost:8080/api/advanced/model \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "message": "Explain quantum computing",
    "temperature": 0.3
  }'
```

**Use GPT-3.5-turbo (faster but less capable):**
```bash
curl -X POST http://localhost:8080/api/advanced/model \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-3.5-turbo",
    "message": "Explain quantum computing",
    "temperature": 0.7
  }'
```

**🎉 Phase 4 Complete!** You now have advanced prompt engineering and multi-model support!

* Commit and push: "Phase 4: Prompt templates and multi-model support"

---

## Complete API Reference

| Method | Endpoint | Description | Example |
|--------|----------|-------------|---------|
| POST | `/api/chat` | Simple chat | See Phase 1 |
| POST | `/api/chat/stream` | Streaming chat | See Phase 2 |
| POST | `/api/chat/conversation` | Chat with context | See Phase 3 |
| GET | `/api/chat/conversation/{id}/history` | Get conversation history | `curl localhost:8080/api/chat/conversation/user123/history` |
| DELETE | `/api/chat/conversation/{id}` | Clear conversation | `curl -X DELETE localhost:8080/api/chat/conversation/user123` |
| GET | `/api/chat/conversations` | List active sessions | `curl localhost:8080/api/chat/conversations` |
| POST | `/api/advanced/persona` | Chat with persona | See Phase 4 |
| POST | `/api/advanced/template` | Use prompt template | See Phase 4 |
| POST | `/api/advanced/model` | Specify AI model | See Phase 4 |
| GET | `/api/advanced/models` | List available models | `curl localhost:8080/api/advanced/models` |

---

## What You've Built

After completing all 4 phases, you have:

✅ **Phase 1 (Basic):** Simple AI chat API with request/response  
✅ **Phase 2 (Streaming):** Real-time streaming responses with SSE  
✅ **Phase 3 (Context):** Conversation memory and context management  
✅ **Phase 4 (Advanced):** Prompt templates and multi-model support  

**Total Features:**

- ✅ AI-powered chat functionality
- ✅ Streaming responses for better UX
- ✅ Conversation context and memory
- ✅ Multiple AI personas (teacher, reviewer, writer)
- ✅ Reusable prompt templates
- ✅ Multi-model support (different AI providers)
- ✅ Temperature control for creativity
- ✅ Session management
- ✅ RESTful API design
- ✅ Error handling

---

## Common Issues & Solutions

### Issue 1: API key not found
**Solution:**

1. Ensure `.env` file exists with correct key
2. Load environment: `source .env`
3. Verify with: `echo $OPENAI_API_KEY`

### Issue 2: Spring AI dependency not resolving
**Solution:**
```bash
mvn clean install -U
# If still failing, check Spring AI milestone repository in pom.xml
```

### Issue 3: Streaming not working
**Problem:** Response arrives all at once  
**Solution:**

- Ensure you're using the `/stream` endpoint
- Use `curl` or a client that supports SSE
- Check that `MediaType.TEXT_EVENT_STREAM_VALUE` is set

### Issue 4: Out of memory with long conversations
**Problem:** Conversation history grows too large  
**Solution:**

- Implement conversation history limits
- Clear old sessions periodically
- Use sliding window approach (keep last N messages)

### Issue 5: Rate limits from AI provider
**Solution:**
```properties
# Add retry logic and exponential backoff
spring.ai.retry.max-attempts=3
spring.ai.retry.backoff.initial-interval=2s
```

---

## Extension Challenges

Try these if you want to explore further:

### Challenge 1: Add Function Calling
Implement AI function calling where the AI can invoke Java methods:

- Weather service
- Calculator
- Database queries

### Challenge 2: Add Vector Store for RAG
Integrate document embeddings and retrieval:

- Upload PDF documents
- Create embeddings using Spring AI
- Query documents with semantic search

### Challenge 3: Add Persistent Storage
Replace in-memory conversation storage:

- Use PostgreSQL or MongoDB
- Store conversation history
- Add user authentication

### Challenge 4: Build a Web UI
Create a simple frontend:

- HTML/CSS/JavaScript chat interface
- Real-time streaming display
- Session management UI

### Challenge 5: Add Cost Tracking
Monitor AI usage costs:

- Track tokens used per request
- Calculate costs based on model pricing
- Add usage analytics dashboard

### Challenge 6: Add Content Moderation
Implement safety features:

- Input validation and sanitization
- Output filtering for inappropriate content
- Rate limiting per user/session

### Challenge 7: Add Image Generation
Extend to multimodal AI:

- Integrate DALL-E or Stable Diffusion
- Image description endpoint
- Image generation from text

---

## Summary

**You learned:**

- Spring AI framework basics
- Integrating AI models into Spring Boot
- Building chat APIs with REST endpoints
- Implementing streaming responses with SSE
- Managing conversation context and memory
- Prompt engineering and templates
- Multi-model provider support
- AI application architecture patterns
- Error handling for AI services
- Best practices for AI integration

---

## Next Steps

What you would normally do, if this was your main project:

1. **Add Unit Tests:** Test services with mock AI responses
2. **Add Integration Tests:** Test full API with test containers
3. **Add Authentication:** Secure endpoints with Spring Security
4. **Add Rate Limiting:** Prevent API abuse
5. **Add Monitoring:** Track AI usage, costs, and performance
6. **Add Vector Store:** Implement RAG for document Q&A
7. **Add Caching:** Cache common queries to reduce costs
8. **Deploy to Cloud:** Deploy with proper API key management

---

## Resources

- [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
- [OpenAI API Documentation](https://platform.openai.com/docs/)
- [Spring Boot Docs](https://docs.spring.io/spring-boot/)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)
- [Server-Sent Events (SSE)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

---

## Security Best Practices

**Important reminders for production applications:**

1. **Never commit API keys** - Use environment variables or secret managers
2. **Implement rate limiting** - Prevent abuse and manage costs
3. **Validate user input** - Sanitize all inputs before sending to AI
4. **Add authentication** - Protect your endpoints
5. **Monitor costs** - Track AI usage to avoid unexpected bills
6. **Add timeout handling** - Prevent long-running requests
7. **Implement retry logic** - Handle transient failures gracefully

