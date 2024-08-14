# 🦜🕸️LangGraph

![버전](https://img.shields.io/pypi/v/langgraph)
[![다운로드](https://static.pepy.tech/badge/langgraph/month)](https://pepy.tech/project/langgraph)
[![오픈 이슈](https://img.shields.io/github/issues-raw/langchain-ai/langgraph)](https://github.com/langchain-ai/langgraph/issues)
[![](https://dcbadge.vercel.app/api/server/6adMQxSpJS?compact=true&style=flat)](https://discord.com/channels/1038097195422978059/1170024642245832774)
[![문서](https://img.shields.io/badge/docs-latest-blue)](https://langchain-ai.github.io/langgraph/)

⚡ 언어 에이전트를 그래프로 구축 ⚡

> [!NOTE]
> JS 버전을 찾고 있나요? [여기를 클릭](https://github.com/langchain-ai/langgraphjs) ([JS 문서](https://langchain-ai.github.io/langgraphjs/))하세요.

> [!TIP]
> LangGraph 애플리케이션을 배포하려고 하시나요? [대기자 명단에 등록](https://www.langchain.com/langgraph-cloud-beta)하여 [LangGraph Cloud](https://langchain-ai.github.io/langgraph/cloud/)에서 관리되는 서비스를 사용해 보세요.

## 개요

[LangGraph](https://langchain-ai.github.io/langgraph/)는 LLM을 사용하여 상태를 유지하고 다중 액터 애플리케이션을 구축하는 라이브러리로, 에이전트 및 다중 에이전트 워크플로우를 만드는 데 사용됩니다. 다른 LLM 프레임워크와 비교할 때, 주기, 제어 가능성 및 지속성을 포함한 주요 이점이 있습니다. LangGraph는 대부분의 에이전트 아키텍처에 필수적인 주기를 포함하는 흐름을 정의할 수 있어 DAG 기반 솔루션과 차별화됩니다. 또한, 애플리케이션의 흐름과 상태를 세밀하게 제어할 수 있는 저수준 프레임워크로서 신뢰할 수 있는 에이전트를 만드는 데 중요합니다. LangGraph에는 지속성이 내장되어 있어 고급 인간 개입 및 메모리 기능을 지원합니다.

LangGraph는 [Pregel](https://research.google/pubs/pub37252/)과 [Apache Beam](https://beam.apache.org/)에서 영감을 받았으며, 공용 인터페이스는 [NetworkX](https://networkx.org/documentation/latest/)에서 영감을 받았습니다. LangGraph는 LangChain Inc, LangChain의 제작자가 개발했으며, LangChain 없이도 사용할 수 있습니다.

### 주요 기능

- **주기 및 분기**: 앱에서 반복문과 조건문을 구현할 수 있습니다.
- **지속성**: 그래프의 각 단계 후 상태를 자동으로 저장합니다. 그래프 실행을 중단하고 다시 시작할 수 있어 오류 복구, 인간 개입 워크플로우, 타임 트래블 등을 지원합니다.
- **인간 개입**: 그래프 실행을 중단하고 에이전트가 계획한 다음 동작을 승인하거나 편집할 수 있습니다.
- **스트리밍 지원**: 각 노드에서 생성되는 출력물을 실시간으로 스트리밍할 수 있습니다(토큰 스트리밍 포함).
- **LangChain과의 통합**: LangGraph는 [LangChain](https://github.com/langchain-ai/langchain/) 및 [LangSmith](https://docs.smith.langchain.com/)와 원활하게 통합되지만, 반드시 필요한 것은 아닙니다.

## 설치

```shell
pip install -U langgraph
```

## 예제

LangGraph의 중심 개념 중 하나는 상태입니다. 각 그래프 실행은 실행되는 그래프의 노드 간에 전달되는 상태를 생성하며, 각 노드는 실행 후 이 내부 상태를 반환 값으로 업데이트합니다. 그래프가 내부 상태를 업데이트하는 방식은 선택한 그래프 유형 또는 사용자 정의 함수에 의해 정의됩니다.

다음은 검색 도구를 사용할 수 있는 간단한 에이전트의 예입니다.

```shell
pip install langchain-anthropic
```

```shell
export ANTHROPIC_API_KEY=sk-...
```

선택적으로, [LangSmith](https://docs.smith.langchain.com/)를 설정하여 최고 수준의 가시성을 확보할 수 있습니다.

```shell
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=lsv2_sk_...
```

```python
from typing import Annotated, Literal, TypedDict

from langchain_core.messages import HumanMessage
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.checkpoint import MemorySaver
from langgraph.graph import END, StateGraph, MessagesState
from langgraph.prebuilt import ToolNode


# 에이전트가 사용할 도구 정의
@tool
def search(query: str):
    """웹 검색을 실행합니다."""
    # 이 부분은 플레이스홀더이지만 LLM에게는 비밀로 하세요...
    if "sf" in query.lower() or "san francisco" in query.lower():
        return ["현재 60도이며 안개가 끼어 있습니다."]
    return ["현재 90도이며 맑습니다."]


tools = [search]

tool_node = ToolNode(tools)

model = ChatAnthropic(model="claude-3-5-sonnet-20240620", temperature=0).bind_tools(tools)

# 계속할지 여부를 결정하는 함수 정의
def should_continue(state: MessagesState) -> Literal["tools", END]:
    messages = state['messages']
    last_message = messages[-1]
    # LLM이 도구 호출을 실행한 경우 "tools" 노드로 라우팅
    if last_message.tool_calls:
        return "tools"
    # 그렇지 않은 경우 중지(사용자에게 응답)
    return END


# 모델 호출 함수 정의
def call_model(state: MessagesState):
    messages = state['messages']
    response = model.invoke(messages)
    # 목록을 반환합니다. 이 목록은 기존 목록에 추가됩니다.
    return {"messages": [response]}


# 새로운 그래프 정의
workflow = StateGraph(MessagesState)

# 주기적으로 반복할 두 개의 노드 정의
workflow.add_node("agent", call_model)
workflow.add_node("tools", tool_node)

# 진입점을 `agent`로 설정
# 이는 이 노드가 첫 번째로 호출된다는 것을 의미합니다.
workflow.set_entry_point("agent")

# 조건부 엣지 추가
workflow.add_conditional_edges(
    # 먼저 시작 노드를 정의합니다. `agent`를 사용합니다.
    # 이는 `agent` 노드가 호출된 후에 취해지는 엣지임을 의미합니다.
    "agent",
    # 다음으로, 어느 노드가 다음에 호출될지 결정하는 함수를 전달합니다.
    should_continue,
)

# `tools`에서 `agent`로의 일반적인 엣지 추가
workflow.add_edge("tools", 'agent')

# 그래프 실행 간 상태를 지속시키기 위한 메모리 초기화
checkpointer = MemorySaver()

# 마지막으로 컴파일합니다!
# 이것을 LangChain Runnable로 컴파일합니다.
# 이는 일반적인 Runnable처럼 사용할 수 있음을 의미합니다.
# 메모리를 컴파일할 때 메모리를 (선택적으로) 전달한다는 점에 유의하세요.
app = workflow.compile(checkpointer=checkpointer)

# Runnable 사용
final_state = app.invoke(
    {"messages": [HumanMessage(content="sf의 날씨는?")]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```

```
"검색 결과에 따르면, 현재 샌프란시스코의 날씨는 다음과 같습니다:\n\n온도: 60도 화씨\n조건: 안개\n\n샌프란시스코는 미세한 기후와 특히 여름철 자주 발생하는 안개로 유명합니다. 60°F(약 15.5°C)의 온도는 이 도시에서는 매우 일반적입니다. 안개, 특히 아침과 저녁 시간에 샌프란시스코의 날씨의 특징입니다.\n\n샌프란시스코의 날씨나 다른 장소의 날씨에 대해 더 알고 싶으신가요?"
```

이제 동일한 `"thread_id"`를 전달하면 대화 컨텍스트가 저장된 상태(저장된 메시지 목록)를 통해 유지됩니다.

```python
final_state = app.invoke(
    {"messages": [HumanMessage(content="뉴욕의 날씨는?")]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```

```
"검색 결과에 따르면, 현재 뉴욕시의 날씨는 다음과 같습니다:\n\n온도: 90도 화씨 (약 32.2도 섭씨)\n조건: 맑음\n\n이는 샌프란시스코에서 본 것과는 매우 다른 날씨입니다. 현재 뉴욕은 훨씬 더 따뜻한 기온을 경험하고 있습니다. 다음은 주목할 만한 몇 가지 사항입니다:\n\n1. 90°F의 온도는 매우 더운 편으로, 뉴욕시의 여름 날씨에 일반적입니다.\n2. 맑은 날씨는 하늘이 맑다는 것을 의미하며, 야외 활동에 적합하지만 직사광선으로 인해 더 더워질 수 있습니다.\n3. 뉴욕의 이러한 날씨는 종종 높은 습도를 동반하여 실제 온도보다 더 따뜻하게 느껴질 수 있습니다.\n\n샌프란시스코의 온화하고 안개 낀 날씨와 뉴욕의 더운 맑은 날씨의 현저한 차이를 보는 것은 흥미롭습니다. 이는 미국의 다른 지역에서 동일한 날에 날씨가 얼마나 다양할 수 있는지를 보여줍니다.\n\n뉴욕의 날씨나 다른 장소에 대해 더 알고 싶으신가요?"
```

### 단계별 분석

1. <details>
    <summary>모델 및 도구 초기화</summary>

    - `ChatAnthropic`을 LLM으로 사용합니다. **참고:** 모델이 사용할 도구를 알고 있어야 합니다. `.bind_tools()` 메서드를 사용하여 LangChain 도구를 OpenAI 도구 호출 형식으로 변환함으로써 이 작업을 수행할 수 있습니다.
    - 사용할 도구를 정의합니다. 우리의 경우 검색 도구입니다. 사용자 지정 도구를 만드는 것은 매우 쉽습니다. 자세한 내용은 [여기](https://python.langchain.com/docs/modules/agents/tools/custom_tools)에서 확인하세요.
   </details>

2. <details>
    <summary>상태와 함께 그래프 초기화</summary>

    - 상태 스키마(`MessagesState`)를 전달하여 그래프(`StateGraph`)를 초기화합니다.
    - `MessagesState`는 하나의 속성 - LangChain `Message` 객체의 목록과 각 노드에서 상태 업데이트를 병합하는 로직이 있는 사전 빌드된 상태 스키마입니다.
   </details>

3. <details>
    <summary>그래프 노드 정의</summary>

    주요 노드는 다음과 같습니다:

      - `agent` 노드: 수행할 동작을 결정하는 역할을 합니다.
      - `tools` 노드: 에이전트가 동작을 결정한 경우, 해당 동작을 실행합니다.
   </details>

4. <details>
    <summary>진입점과 그래프 엣지 정의</summary>

      먼저, 그래프 실행을 위한 진입점을 설정해야 합니다 - `agent` 노드.

      그런 다음 하나의 일반 엣지와 하나의 조건부 엣지를 정의합니다. 조건부 엣지는 그래프 상태(`MessageState`)의 내용에 따라 목적지가 달라진다는 것을 의미합니다. 우리의 경우, 목적지는 에이전트(LLM)가 결정할 때까지 알려지지 않습니다.

      - 조건부 엣지: 에이전트가 호출된 후, 우리는 다음을 실행해야 합니다:
        - a. 에이전트가 동작을 수행하기로 결정한 경우 도구 실행, 또는
        - b. 에이전트가 도구 실행을 요청하지 않은 경우 종료(사용자에게 응답)
      - 일반 엣지: 도구가 호출된 후, 그래프는 항상 다음에 수행할 작업을 결정하기 위해 에이전트로 돌아가야 합니다.
   </details>

5. <details>
    <summary>그래프 컴파일</summary>

    - 그래프를 컴파일하면 LangChain [Runnable](https://python.langchain.com/v0.2/docs/concepts/#runnable-interface)로 변환되며, 자동으로 `.invoke()`, `.stream()` 및 `.batch()`를 입력과 함께 호출할 수 있습니다.
    - 그래프 실행 간 상태를 지속시키기 위해 체크포인터 객체를 선택적으로 전달할 수 있습니다. 우리의 경우 간단한 메모리 체크포인터인 `MemorySaver`를 사용합니다.
    </details>

6. <details>
   <summary>그래프 실행</summary>

    1. LangGraph는 입력 메시지를 내부 상태에 추가한 후, 상태를 진입점 노드인 `"agent"`로 전달합니다.
    2. `"agent"` 노드가 실행되며, 챗 모델을 호출합니다.
    3. 챗 모델은 `AIMessage`를 반환합니다. LangGraph는 이를 상태에 추가합니다.
    4. 그래프는 더 이상 `AIMessage`에 `tool_calls`이 없을 때까지 다음 단계를 반복합니다:

        - `AIMessage`에 `tool_calls`이 있는 경우 `"tools"` 노드가 실행됩니다.
        - `"agent"` 노드가 다시 실행되며 `AIMessage`를 반환합니다.

    5. 실행은 특별한 `END` 값으로 진행되며 최종 상태를 출력합니다.
    결과적으로 우리는 모든 채팅 메시지 목록을 출력물로 받습니다.
   </details>

## 문서

* [튜토리얼](https://langchain-ai.github.io/langgraph/tutorials/): LangGraph를 사용하여 빌드하는 방법을 안내합니다.
* [How-to 가이드](https://langchain-ai.github.io/langgraph/how-tos/): 스트리밍부터 메모리 및 지속성 추가, 일반적인 디자인 패턴(분기, 서브그래프 등)까지 LangGraph에서 특정 작업을 수행하는 방법을 설명합니다.
* [개념 가이드](https://langchain-ai.github.io/langgraph/concepts/): 노드, 엣지, 상태 등 LangGraph의 주요 개념과 원리에 대한 심층적인 설명을 제공합니다.
* [API 레퍼런스](https://langchain-ai.github.io/langgraph/reference/graphs/): 그래프 및 체크포인팅 API 사용 방법에 대한 간단한 예제, 고수준의 사전 빌드된 구성 요소 등을 검토할 수 있습니다.
* [클라우드(베타)](https://langchain-ai.github.io/langgraph/cloud/): 클릭 한 번으로 LangGraph 애플리케이션을 LangGraph Cloud에 배포할 수 있습니다.

## 기여

기여 방법에 대한 자세한 내용은 [여기](https://github.com/langchain-ai/langgraph/blob/main/CONTRIBUTING.md)를 참조하세요.