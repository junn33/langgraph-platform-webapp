# 에이전트 채팅 UI

에이전트 채팅 UI는 `messages` 키를 가진 모든 LangGraph 서버와 채팅 인터페이스를 통해 대화할 수 있는 Next.js 애플리케이션입니다.

> [!NOTE]
> 🎥 설정 가이드 비디오를 [여기](https://youtu.be/lInrwVnZ83o)에서 시청하세요.

## 설정

> [!TIP]
> 앱을 로컬에서 실행하지 않으시겠습니까? 배포된 사이트를 사용하세요: [agentchat.vercel.app](https://agentchat.vercel.app)!

먼저 저장소를 클론하거나 [`npx` 명령어](https://www.npmjs.com/package/create-agent-chat-app)를 실행하세요:

```bash
npx create-agent-chat-app
```

또는

```bash
git clone https://github.com/langchain-ai/agent-chat-ui.git

cd agent-chat-ui
```

의존성을 설치하세요:

```bash
pnpm install
```

앱을 실행하세요:

```bash
pnpm dev
```

앱은 `http://localhost:3000`에서 사용할 수 있습니다.

## 사용법

앱이 실행되면 (또는 배포된 사이트를 사용하는 경우) 다음 정보를 입력하라는 메시지가 표시됩니다:

- **배포 URL**: 대화하고 싶은 LangGraph 서버의 URL입니다. 이는 프로덕션 또는 개발 URL일 수 있습니다.
- **어시스턴트/그래프 ID**: 채팅 인터페이스를 통해 실행을 가져오고 제출할 때 사용할 그래프의 이름 또는 어시스턴트의 ID입니다.
- **LangSmith API 키**: (배포된 LangGraph 서버에 연결할 때만 필요) LangGraph 서버로 전송된 요청을 인증할 때 사용할 LangSmith API 키입니다.

이러한 값을 입력한 후 `계속하기`를 클릭하세요. 그러면 LangGraph 서버와 대화를 시작할 수 있는 채팅 인터페이스로 리디렉션됩니다.

## 환경 변수

다음 환경 변수를 설정하여 초기 설정 양식을 우회할 수 있습니다:

```bash
NEXT_PUBLIC_API_URL=http://localhost:2024
NEXT_PUBLIC_ASSISTANT_ID=agent
```

> [!TIP]
> 프로덕션 LangGraph 서버에 연결하려면 [프로덕션으로 전환](#프로덕션으로-전환) 섹션을 읽어보세요.

이러한 변수를 사용하려면:

1. `.env.example` 파일을 `.env`라는 새 파일로 복사하세요
2. `.env` 파일에 값을 입력하세요
3. 애플리케이션을 다시 시작하세요

이러한 환경 변수가 설정되면 애플리케이션은 설정 양식 대신 이 변수들을 사용합니다.

## 채팅에서 메시지 숨기기

에이전트 채팅 UI에서 메시지의 가시성을 두 가지 주요 방법으로 제어할 수 있습니다:

**1. 실시간 스트리밍 방지:**

LLM 호출에서 메시지가 스트리밍되는 동안 표시되지 않도록 하려면 채팅 모델의 구성에 `langsmith:nostream` 태그를 추가하세요. UI는 일반적으로 `on_chat_model_stream` 이벤트를 사용하여 스트리밍 메시지를 렌더링합니다. 이 태그는 태그가 지정된 모델에 대해 이러한 이벤트가 발생하지 않도록 합니다.

_Python 예시:_

```python
from langchain_anthropic import ChatAnthropic

# .with_config 메서드를 통해 태그 추가
model = ChatAnthropic().with_config(
    config={"tags": ["langsmith:nostream"]}
)
```

_TypeScript 예시:_

```typescript
import { ChatAnthropic } from "@langchain/anthropic";

const model = new ChatAnthropic()
  // .withConfig 메서드를 통해 태그 추가
  .withConfig({ tags: ["langsmith:nostream"] });
```

**참고:** 이렇게 스트리밍을 숨겨도 메시지는 LLM 호출이 완료된 후 그래프의 상태에 추가 변경 없이 저장되면 여전히 나타납니다.

**2. 메시지 영구 숨기기:**

메시지가 채팅 UI에 절대 표시되지 않도록 하려면 (스트리밍 중이든 상태에 저장된 후든) 그래프의 상태에 추가하기 전에 `id` 필드에 `do-not-render-` 접두사를 붙이고, 채팅 모델의 구성에 `langsmith:do-not-render` 태그를 추가하세요. UI는 이 접두사로 시작하는 ID를 가진 모든 메시지를 명시적으로 필터링합니다.

_Python 예시:_

```python
result = model.invoke([messages])
# 상태에 저장하기 전에 ID에 접두사 추가
result.id = f"do-not-render-{result.id}"
return {"messages": [result]}
```

_TypeScript 예시:_

```typescript
const result = await model.invoke([messages]);
// 상태에 저장하기 전에 ID에 접두사 추가
result.id = `do-not-render-${result.id}`;
return { messages: [result] };
```

이 접근 방식은 메시지가 사용자 인터페이스에서 완전히 숨겨지도록 보장합니다.

## 프로덕션으로 전환

프로덕션으로 전환할 준비가 되면 배포에 연결하고 요청을 인증하는 방법을 업데이트해야 합니다. 기본적으로 에이전트 채팅 UI는 로컬 개발을 위해 설정되어 있으며 클라이언트에서 LangGraph 서버에 직접 연결합니다. 이는 프로덕션에서는 불가능합니다. 모든 사용자가 자신의 LangSmith API 키를 가지고 LangGraph 구성을 직접 설정해야 하기 때문입니다.

### 프로덕션 설정

에이전트 채팅 UI를 프로덕션화하려면 LangGraph 서버에 대한 요청을 인증하는 두 가지 방법 중 하나를 선택해야 합니다. 아래에서 두 가지 옵션을 설명하겠습니다:

### 빠른 시작 - API 통과

에이전트 채팅 UI를 프로덕션화하는 가장 빠른 방법은 [API 통과](https://github.com/langchain-ai/langgraph-nextjs-api-passthrough) 패키지를 사용하는 것입니다. 이 패키지는 LangGraph 서버로의 요청을 프록시하고 인증을 처리하는 간단한 방법을 제공합니다.

이 저장소에는 이 방법을 사용하기 위해 필요한 모든 코드가 이미 포함되어 있습니다. 필요한 유일한 구성은 적절한 환경 변수를 설정하는 것입니다.

```bash
NEXT_PUBLIC_ASSISTANT_ID="agent"
# 이는 LangGraph 서버의 배포 URL이어야 합니다
LANGGRAPH_API_URL="https://my-agent.default.us.langgraph.app"
# 이는 웹사이트 URL + "/api"여야 합니다. 이것이 API 프록시에 연결하는 방법입니다
NEXT_PUBLIC_API_URL="https://my-website.com/api"
# API 프록시 내부에서 요청에 주입되는 LangSmith API 키
LANGSMITH_API_KEY="lsv2_..."
```

각 환경 변수의 역할을 살펴보겠습니다:

- `NEXT_PUBLIC_ASSISTANT_ID`: 채팅 인터페이스를 통해 실행을 가져오고 제출할 때 사용할 어시스턴트의 ID입니다. 이는 클라이언트에서 요청을 제출할 때 사용되므로 여전히 `NEXT_PUBLIC_` 접두사가 필요합니다.
- `LANGGRAPH_API_URL`: LangGraph 서버의 URL입니다. 이는 프로덕션 배포 URL이어야 합니다.
- `NEXT_PUBLIC_API_URL`: 웹사이트 URL + `/api`입니다. 이것이 API 프록시에 연결하는 방법입니다. [에이전트 채팅 데모](https://agentchat.vercel.app)의 경우 이는 `https://agentchat.vercel.app/api`로 설정됩니다. 프로덕션 URL로 설정해야 합니다.
- `LANGSMITH_API_KEY`: LangGraph 서버로 전송된 요청을 인증하는 데 사용할 LangSmith API 키입니다. 다시 한 번, 이는 비밀이므로 `NEXT_PUBLIC_` 접두사를 붙이지 마세요. API 프록시가 배포된 LangGraph 서버로 요청을 보낼 때만 서버에서 사용됩니다.

자세한 문서는 [LangGraph Next.js API 통과](https://www.npmjs.com/package/langgraph-nextjs-api-passthrough) 문서를 참조하세요.

### 고급 설정 - 사용자 정의 인증

> [!WARNING]
> 현재 이 기능은 Python LangGraph 배포에서만 사용할 수 있으며, TypeScript 배포에서는 곧 사용할 수 있을 예정입니다.

LangGraph 배포에서 사용자 정의 인증은 LangGraph 서버에 대한 요청을 인증하는 고급적이고 더 강력한 방법입니다. 사용자 정의 인증을 사용하면 LangSmith API 키 없이도 클라이언트에서 요청을 할 수 있습니다. 또한 요청에 대한 사용자 정의 액세스 제어를 지정할 수 있습니다.

LangGraph 배포에서 이를 설정하려면 [LangGraph 사용자 정의 인증 문서](https://langchain-ai.github.io/langgraph/tutorials/auth/getting_started/)를 읽어보세요.

배포에서 설정한 후 에이전트 채팅 UI에 다음과 같은 변경을 해야 합니다:

1. 클라이언트에서 요청을 인증하는 데 사용할 인증 토큰을 LangGraph 배포에서 가져오는 추가 API 요청을 구성하세요.
2. `NEXT_PUBLIC_API_URL` 환경 변수를 프로덕션 LangGraph 배포 URL로 설정하세요.
3. `NEXT_PUBLIC_ASSISTANT_ID` 환경 변수를 채팅 인터페이스를 통해 실행을 가져오고 제출할 때 사용할 어시스턴트의 ID로 설정하세요.
4. [`useTypedStream`](src/providers/Stream.tsx) (`useStream`의 확장) 훅을 수정하여 인증 토큰을 헤더를 통해 LangGraph 서버로 전달하세요:

```tsx
const streamValue = useTypedStream({
  apiUrl: process.env.NEXT_PUBLIC_API_URL,
  assistantId: process.env.NEXT_PUBLIC_ASSISTANT_ID,
  // ... 다른 필드들
  defaultHeaders: {
    Authentication: `Bearer ${여기에토큰추가}`, // 여기에 인증 토큰을 전달합니다
  },
});
```
