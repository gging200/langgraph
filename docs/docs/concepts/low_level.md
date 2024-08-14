# 저수준 개념 가이드

## 그래프

LangGraph의 핵심은 에이전트 워크플로를 그래프로 모델링하는 것입니다. 에이전트의 동작을 정의하기 위해 세 가지 주요 구성 요소를 사용합니다:

1. [`State`](#state): 애플리케이션의 현재 스냅샷을 나타내는 공유 데이터 구조입니다. 이는 Python의 어떤 타입이든 될 수 있지만, 일반적으로 `TypedDict` 또는 Pydantic의 `BaseModel`을 사용합니다.

2. [`Nodes`](#nodes): 에이전트의 로직을 인코딩하는 Python 함수입니다. 이 함수들은 현재 `State`를 입력으로 받아서 일부 계산이나 부수 효과를 수행하고, 업데이트된 `State`를 반환합니다.

3. [`Edges`](#edges): 현재 `State`를 기반으로 다음에 실행할 `Node`를 결정하는 Python 함수입니다. 이 함수들은 조건부 분기일 수도 있고 고정된 전환일 수도 있습니다.

`Nodes`와 `Edges`를 조합하여 시간이 지남에 따라 `State`가 발전하는 복잡하고 반복적인 워크플로를 만들 수 있습니다. LangGraph의 진정한 힘은 `State`를 관리하는 방식에서 나옵니다. 강조하자면, `Nodes`와 `Edges`는 Python 함수에 불과합니다 - 여기에는 LLM이 포함될 수도 있고, 일반적인 Python 코드일 수도 있습니다.

간단히 말하자면: _노드는 작업을 수행하고, 엣지는 다음에 수행할 작업을 지시합니다_.

LangGraph의 기본 그래프 알고리즘은 [메시지 전달](https://en.wikipedia.org/wiki/Message_passing)을 사용하여 일반적인 프로그램을 정의합니다. 노드가 작업을 완료하면, 하나 이상의 엣지를 따라 다른 노드로 메시지를 보냅니다. 이러한 수신 노드는 기능을 실행하고, 결과 메시지를 다음 노드 집합으로 전달하며, 이 과정은 계속됩니다. Google의 [Pregel](https://research.google/pubs/pregel-a-system-for-large-scale-graph-processing/) 시스템에서 영감을 받아, 프로그램은 개별적인 "슈퍼 스텝"으로 진행됩니다.

슈퍼 스텝은 그래프 노드를 한 번 반복하는 것으로 간주할 수 있습니다. 병렬로 실행되는 노드는 동일한 슈퍼 스텝의 일부이며, 순차적으로 실행되는 노드는 별도의 슈퍼 스텝에 속합니다. 그래프 실행이 시작되면 모든 노드는 `비활성` 상태로 시작합니다. 노드는 하나 이상의 수신 엣지(또는 "채널")에서 새 메시지(상태)를 수신하면 `활성` 상태가 됩니다. 활성화된 노드는 기능을 실행하고 업데이트로 응답합니다. 각 슈퍼 스텝이 끝나면, 수신 메시지가 없는 노드는 자신을 `비활성`으로 표시하여 `중단`에 투표합니다. 모든 노드가 `비활성` 상태가 되고 메시지가 전송되지 않으면 그래프 실행이 종료됩니다.

### StateGraph

`StateGraph` 클래스는 사용하는 주요 그래프 클래스입니다. 이는 사용자 정의 `State` 객체로 매개변수화됩니다.

### MessageGraph

`MessageGraph` 클래스는 특별한 유형의 그래프입니다. `MessageGraph`의 `State`는 단순히 메시지 목록입니다. 이 클래스는 챗봇 외의 대부분의 애플리케이션에서 사용되지 않으며, 대부분의 애플리케이션은 메시지 목록보다 더 복잡한 `State`를 필요로 합니다.

### 그래프 컴파일

그래프를 구축하려면 먼저 [상태](#state)를 정의하고, [노드](#nodes)와 [엣지](#edges)를 추가한 후, 그래프를 컴파일해야 합니다. 그렇다면 그래프를 컴파일하는 것이 정확히 무엇이며, 왜 필요할까요?

컴파일은 비교적 간단한 단계입니다. 이는 그래프의 구조(고아 노드가 없는지 등)에 대한 몇 가지 기본적인 검사를 제공합니다. 또한 런타임 인수(예: [체크포인터](#checkpointer) 및 [중단점](#breakpoints))를 지정할 수 있는 곳이기도 합니다. 그래프를 컴파일하려면 `.compile` 메서드를 호출합니다:

```python
graph = graph_builder.compile(...)
```

그래프를 사용하기 전에 **반드시** 컴파일해야 합니다.

## State

그래프를 정의할 때 가장 먼저 해야 할 일은 그래프의 `State`를 정의하는 것입니다. `State`는 [그래프의 스키마](#schema)와 `State`에 대한 업데이트를 적용하는 방법을 지정하는 [`리듀서` 함수](#리듀서)로 구성됩니다. `State`의 스키마는 그래프의 모든 `Nodes` 및 `Edges`의 입력 스키마가 되며, 이는 `TypedDict` 또는 `Pydantic` 모델이 될 수 있습니다. 모든 `Nodes`는 `State`에 업데이트를 출력하며, 이는 지정된 `리듀서` 함수를 사용하여 적용됩니다.

### 스키마

그래프의 스키마를 지정하는 주요 문서화된 방법은 `TypedDict`을 사용하는 것입니다. 그러나 **기본값** 및 추가 데이터 유효성 검사를 추가하기 위해 [Pydantic BaseModel을 사용하는 것](../how-tos/state-model.ipynb)도 지원합니다.

### 리듀서

리듀서는 노드의 업데이트가 `State`에 어떻게 적용되는지를 이해하는 데 중요합니다. `State`의 각 키에는 고유한 독립적인 리듀서 함수가 있습니다. 리듀서 함수가 명시적으로 지정되지 않은 경우, 해당 키에 대한 모든 업데이트가 이를 덮어쓰도록 간주됩니다. 몇 가지 예제를 살펴보며 이를 더 잘 이해해보겠습니다.

**예제 A:**

```python
from typing import TypedDict

class State(TypedDict):
    foo: int
    bar: list[str]
```

이 예에서는 어떤 키에 대해서도 리듀서 함수가 지정되지 않았습니다. 그래프의 입력이 `{"foo": 1, "bar": ["hi"]}`라고 가정해 봅시다. 그런 다음 첫 번째 `Node`가 `{"foo": 2}`를 반환한다고 가정합니다. 이는 상태에 대한 업데이트로 처리됩니다. 노드가 전체 `State` 스키마를 반환할 필요는 없으며, 업데이트만 반환하면 됩니다. 이 업데이트를 적용한 후 `State`는 `{"foo": 2, "bar": ["hi"]}`가 됩니다. 두 번째 노드가 `{"bar": ["bye"]}`를 반환하면 `State`는 `{"foo": 2, "bar": ["bye"]}`가 됩니다.

**예제 B:**

```python
from typing import TypedDict, Annotated
from operator import add

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]
```

이 예제에서는 `Annotated` 타입을 사용하여 두 번째 키(`bar`)에 대한 리듀서 함수(`operator.add`)를 지정했습니다. 첫 번째 키는 변경되지 않습니다. 그래프의 입력이 `{"foo": 1, "bar": ["hi"]}`라고 가정해 봅시다. 그런 다음 첫 번째 `Node`가 `{"foo": 2}`를 반환한다고 가정합니다. 이는 상태에 대한 업데이트로 처리됩니다. 노드가 전체 `State` 스키마를 반환할 필요는 없으며, 업데이트만 반환하면 됩니다. 이 업데이트를 적용한 후 `State`는 `{"foo": 2, "bar": ["hi"]}`가 됩니다. 두 번째 노드가 `{"bar": ["bye"]}`를 반환하면 `State`는 `{"foo": 2, "bar": ["hi", "bye"]}`가 됩니다. 여기서 `bar` 키는 두 리스트를 더하여 업데이트됩니다.

### 메시지 상태

`MessageState`는 LangGraph에서 사용하기 쉽게 설계된 몇 안 되는 구성 요소 중 하나입니다. `MessageState`는 상태에서 메시지 목록을 키로 사용하기 쉽게 만들어진 특별한 상태입니다. 구체적으로, `MessageState`는 다음과 같이 정의됩니다:

```python
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages
from typing import Annotated, TypedDict

class MessagesState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
```

이것은 `TypedDict`를 생성하고 단일 키인 `messages`를 포함합니다. 이는 `Message` 객체의 목록이며, `add_messages`를 리듀서로 사용합니다. `add_messages`는 기본적으로 메시지를 기존 목록에 추가합니다(또한 OpenAI 메시지 형식을 표준 LangChain 메시지 형식으로 변환하고, 메시지 ID를 기반으로 업데이트를 처리하는 등의 기능도 수행합니다).

우리는 종종 메시지 목록이 상태의 핵심 구성 요소가 되는 것을 보며, 이 사전 구축된 상태는 메시지를 쉽게 사용할 수 있도록 설계되었습니다. 일반적으로 메시지 외에도 추적할 상태가 더 많이 있으므로, 우리는 이 상태를 서브클래싱하고 더 많은 필드를 추가하는 것을 봅니다:

```python
from langgraph.graph import MessagesState

class State(MessagesState):
    documents: list[str]
```

## 노드

LangGraph에서 노드는 일반적으로 Python 함수(동기 또는 `async`)이며, 첫 번째 위치 인수는 [상태](#state)이고, (선택적으로) 두 번째 위치 인수는 `thread_id`와 같은 선택적 [구성 가능한 매개변수](#configuration)를 포함하는 "구성"입니다.

`NetworkX`와 유사하게, [add_node][langgraph.graph.StateGraph.add_node] 메서드를 사용하여 이 노드를 그래프에 추가합니다:

```python
from langchain_core.runnables import RunnableConfig
from langgraph.graph import StateGraph

builder = StateGraph(dict)


def my_node(state: dict, config: RunnableConfig):
    print("In node: ", config["configurable"]["user_id"])
    return {"results": f"Hello, {state['input']}!"}


# 두 번째 인수는 선택 사항입니다.
def my_other_node(state: dict):
    return state


builder.add_node("my_node", my_node)
builder.add_node("other_node", my_other_node)
...
```

백그라운드에서 함수는 [RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html#langchain_core.runnables.base.RunnableLambda)로 변환되어, 함수에 배치 및 비동기 지원을 추가하며, 기본적인 추적 및 디버깅 기능을 제공합니다.

이름을 지정하지 않고 그래프에 노드를 추가하면, 기본적으로 함수 이름과 동일한 이름이 지정됩니다.

```python
builder.add_node(my_node)
# 그런 다음 이 노드에서 또는 이 노드로의 엣지를 생성하여 참조할 수 있습니다.
```

### `START` 노드

`START` 노드는 사용자 입력을 그래프에 보내는 노드를 나타내는 특별한 노드입니다. 이 노드를 참조하는 주요 목적은 어떤 노드를 먼저 호출할지 결정하는 것입니다.

```python
from langgraph.graph import START

graph.add_edge(START, "node_a")
```

### `END` 노드

`END` 노드는 종료 노드를 나타내는 특별한 노드입니다. 작업이 완료된 후에 동작이 없는 엣지를 나타내고자 할 때 이 노드를 참조합니다.

```python
from langgraph.graph import END

graph.add_edge("node_a", END)
```

## 엣지

엣지는 논리를 라우팅하고 그래프가 중단되는 방식을 정의합니다. 이는 에이전트가 작동하는 방식과 서로 다른 노드가 서로 통신하는 방식을 크게 좌우합니다. 몇 가지 주요 엣지 유형이 있습니다:

- 일반 엣지: 한 노드에서 다른 노드로 직접 이동합니다.
- 조건부 엣지: 함수를 호출하여 다음에 어떤 노드로 이동할지 결정합니다.
- 시작 지점: 사용자 입력이 도착할 때 처음 호출할 노드를 결정합니다.
- 조건부 시작 지점: 사용자 입력이 도착할 때 처음 호출할 노드를 함수로 결정합니다.

노드에는 여러 개의 출력 엣지가 있을 수 있습니다. 노드에 여러 개의 출력 엣지가 있는 경우, **모든** 대상 노드가 다음 슈퍼스텝의 일부로 병렬로 실행됩니다.

### 일반 엣지

항상 노드 A에서 노드 B로 이동하려면 [add_edge][langgraph.graph.StateGraph.add_edge] 메서드를 직접 사용할 수 있습니다.

```python
graph.add_edge("node_a", "node_b")
```

### 조건부 엣지

1개 이상의 엣지로 선택적으로 라우팅(또는 선택적으로 종료)하려면 [add_conditional_edges][langgraph.graph.StateGraph.add_conditional_edges] 메서드를 사용할 수 있습니다. 이 메서드는 노드 이름과 해당 노드가 실행된 후 호출할 "라우팅 함수"를 인수로 받습니다:

```python
graph.add_conditional_edges("node_a", routing_function)
```

노드와 마찬가지로 `routing_function`은 그래프의 현재 `state`를 받아 값을 반환합니다.

기본적으로, `routing_function`의 반환 값은 상태를 다음에 보낼 노드(또는 노드 목록)의 이름으로 사용됩니다. 이러한 모든 노드는 다음 슈퍼스텝의 일부로 병렬로 실행됩니다.

`routing_function`의 출력을 다음 노드의 이름에 매핑하는 사전을 선택적으로 제공할 수 있습니다.

```python
graph.add_conditional_edges("node_a", routing_function, {True: "node_b", False: "node_c"})
```

### 시작 지점

시작 지점은 그래프가 시작될 때 실행되는 첫 번째 노드입니다. 가상 [`START`][start] 노드에서 첫 번째로 실행할 노드로 진입점을 지정하기 위해 [`add_edge`][langgraph.graph.StateGraph.add_edge] 메서드를 사용할 수 있습니다.

```python
from langgraph.graph import START

graph.add_edge(START, "node_a")
```

### 조건부 시작 지점

조건부 시작 지점은 사용자 정의 로직에 따라 다른 노드에서 시작할 수 있도록 합니다. 가상 [`START`][start] 노드에서 [`add_conditional_edges`][langgraph.graph.StateGraph.add_conditional_edges]를 사용하여 이를 수행할 수 있습니다.

```python
from langgraph.graph import START

graph.add_conditional_edges(START, routing_function)
```

`routing_function`의 출력을 다음 노드의 이름에 매핑하는 사전을 선택적으로 제공할 수 있습니다.

```python
graph.add_conditional_edges(START, routing_function, {True: "node_b", False: "node_c"})
```

## `Send`

기본적으로 `Nodes`와 `Edges`는 사전에 정의되며 동일한 공유 상태에서 작동합니다. 그러나 경우에 따라 정확한 엣지를 미리 알 수 없거나, 동시에 서로 다른 `State` 버전이 존재하기를 원할 수도 있습니다. 일반적인 예는 `맵-리듀스(map-reduce)` 설계 패턴입니다. 이 설계 패턴에서 첫 번째 노드는 객체 목록을 생성할 수 있으며, 이 목록의 모든 객체에 대해 일부 다른 노드를 적용하고자 할 수 있습니다. 객체의 수는 미리 알 수 없으며(즉, 엣지의 수를 미리 알 수 없음), 하위 `Node`에 대한 입력 `State`는 다릅니다(생성된 각 객체에 대해).

이 설계 패턴을 지원하기 위해, LangGraph는 조건부 엣지에서 [`Send`](../reference/graphs.md#send) 객체를 반환하는 것을 지원합니다. `Send`는 두 개의 인수를 받습니다: 첫 번째는 노드의 이름이고, 두 번째는 그 노드로 전달할 상태입니다.

```python
def continue_to_jokes(state: OverallState):
    return [Send("generate_joke", {"subject": s}) for s in state['subjects']]

graph.add_conditional_edges("node_a", continue_to_jokes)
```

## 체크포인터

LangGraph는 [체크포인터][basecheckpointsaver]를 통해 구현된 내장된 영속성 계층을 가지고 있습니다. 그래프에 체크포인터를 사용하는 경우, 그래프의 상태와 상호작용하고 관리할 수 있습니다. 체크포인터는 그래프 상태의 _체크포인트_를 각 슈퍼스텝에서 저장하여 여러 강력한 기능을 제공합니다:

첫째, 체크포인터는 인간이 그래프의 상태를 어느 시점에서든지 확인하고, 중단하고, 단계를 승인할 수 있도록 하여 [사람이 개입하는 워크플로](agentic_concepts.md#human-in-the-loop)를 촉진합니다. 인간이 그래프의 상태를 확인하고, 상태에 대한 업데이트를 수행한 후 그래프 실행을 재개해야 하기 때문에 이러한 워크플로에는 체크포인터가 필요합니다.

둘째, 체크포인터를 사용하여 상호작용 간에 ["메모리"](agentic_concepts.md#memory)를 허용할 수 있습니다. 체크포인터를 사용하여 스레드를 생성하고 그래프 실행 후 스레드의 상태를 저장할 수 있습니다. 반복되는 인간 상호작용(예: 대화)에서는 후속 메시지를 해당 체크포인트로 보낼 수 있으며, 이전 메시지를 기억합니다.

그래프에 체크포인터를 추가하는 방법에 대한 자세한 내용은 [이 가이드](../how-tos/persistence.ipynb)를 참조하세요.

## 스레드

스레드는 여러 다른 실행의 체크포인트를 가능하게 하여, 다중 테넌트 채팅 애플리케이션 및 별도의 상태 유지를 필요로 하는 다른 시나리오에서 필수적입니다. 스레드는 체크포인터에 의해 저장된 체크포인트 시리즈에 할당된 고유 ID입니다. 체크포인터를 사용할 때, 그래프를 실행할 때 `thread_id` 또는 `thread_ts`를 지정해야 합니다.

`thread_id`는 스레드의 ID입니다. 이는 항상 필요합니다.

`thread_ts`는 선택적으로 전달할 수 있습니다. 이 식별자는 스레드 내의 특정 체크포인트를 참조합니다. 이를 통해 스레드 중간에서 그래프 실행을 시작할 수 있습니다.

이들을 설정하려면 구성 가능한 설정의 일부로 그래프를 호출할 때 전달해야 합니다.

```python
config = {"configurable": {"thread_id": "a"}}
graph.invoke(inputs, config=config)
```

스레드를 사용하는 방법에 대한 자세한 내용은 [이 가이드](../how-tos/persistence.ipynb)를 참조하세요.

## 체크포인터 상태

체크포인터 상태와 상호작용할 때는 [스레드 식별자](#threads)를 지정해야 합니다. 체크포인터에 의해 저장된 각 체크포인트에는 두 가지 속성이 있습니다:

- **values**: 이 값은 이 시점에서의 상태 값입니다.
- **next**: 그래프에서 다음에 실행할 노드의 튜플입니다.

### 상태 가져오기

`graph.get_state(config)`를 호출하여 체크포인터의 상태를 가져올 수 있습니다. 설정에는 `thread_id`가 포함되어야 하며, 해당 스레드에 대한 상태가 검색됩니다.

### 상태 기록 가져오기

`graph.get_state_history(config)`를 호출하여 그래프의 기록 목록을 가져올 수도 있습니다. 설정에는 `thread_id`가 포함되어야 하며, 해당 스레드에 대한 상태 기록이 검색됩니다.

### 상태 업데이트

상태와 직접 상호작용하고 업데이트할 수도 있습니다. 이는 세 가지 구성 요소로 구성됩니다:

- 구성
- 값
- `as_node`

**구성**

구성에는 업데이트할 스레드를 지정하는 `thread_id`가 포함되어야 합니다.

**값**

이 값들은 상태를 업데이트하는 데 사용됩니다. 이 업데이트는 노드의 업데이트로 간주됩니다. 이는 이 값들이 상태의 일부인 [리듀서](#reducers) 함수에 전달됨을 의미합니다. 따라서 상태를 자동으로 덮어쓰지 않습니다. 예제를 통해 살펴보겠습니다.

그래프 상태를 다음과 같이 정의했다고 가정해 보겠습니다:

```python
from typing import TypedDict, Annotated
from operator import add

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]
```

그래프의 현재 상태가 다음과 같다고 가정해 보겠습니다:

```
{"foo": 1, "bar": ["a"]}
```

아래와 같이 상태를 업데이트하면:

```
graph.update_state(config, {"foo": 2, "bar": ["b"]})
```

그래프의 새로운 상태는 다음과 같습니다:

```
{"foo": 2, "bar": ["a", "b"]}
```

`foo` 키는 완전히 변경됩니다(해당 키에 리듀서가 지정되지 않았기 때문에 덮어씁니다). 그러나 `bar` 키에는 리듀서가 지정되어 있으므로, `bar` 상태에 `"b"`가 추가됩니다.

**`as_node`**

`update_state`를 호출할 때 마지막으로 지정하는 것은 `as_node`입니다. 이 업데이트는 노드 `as_node`에서 온 것처럼 적용됩니다. `as_node`가 제공되지 않으면, 모호하지 않은 경우 상태를 마지막으로 업데이트한 노드로 설정됩니다.

이것이 중요한 이유는 그래프에서 다음에 실행할 단계가 마지막으로 업데이트를 제공한 노드에 따라 다르기 때문에, 이를 사용하여 다음에 실행할 노드를 제어할 수 있습니다.

## 구성

그래프를 만들 때 그래프의 특정 부분이 구성 가능하다는 것을 표시할 수도 있습니다. 이는 모델 또는 시스템 프롬프트 간에 쉽게 전환할 수 있도록 하기 위해 일반적으로 수행됩니다. 이를 통해 단일 "인지 아키텍처"(그래프)를 생성할 수 있지만, 여러 다른 인스턴스를 사용할 수 있습니다.

그래프를 생성할 때 `config_schema`를 선택적으로 지정할 수 있습니다.

```python
class ConfigSchema(TypedDict):
    llm: str

graph = StateGraph(State, config_schema=ConfigSchema)
```

이 구성을 `configurable` 구성 필드를 사용하여 그래프에 전달할 수 있습니다.

```python
config = {"configurable": {"llm": "anthropic"}}

graph.invoke(inputs, config=config)
```

그런 다음 노드 내에서 이 구성을 액세스하고 사용할 수 있습니다:

```python
def node_a(state, config):
    llm_type = config.get("configurable", {}).get("llm", "openai")
    llm = get_llm(llm_type)
    ...
```

구성에 대한 전체 설명은 [이 가이드](../how-tos/configuration.ipynb)를 참조하세요.

## 중단점

일부 노드가 실행되기 전이나 후에 중단점을 설정하는 것이 유용할 수 있습니다. 이는 계속하기 전에 인간의 승인을 기다리는 데 사용될 수 있습니다. 이러한 중단점은 ["그래프 컴파일"](#compiling-your-graph) 시 설정할 수 있습니다. `interrupt_before`를 사용하여 노드가 실행되기 **전** 또는 `interrupt_after`를 사용하여 노드가 실행된 **후**에 중단점을 설정할 수 있습니다.

중단점을 사용할 때는 [체크포인터](#checkpointer)를 **반드시** 사용해야 합니다. 이는 그래프가 실행을 재개할 수 있어야 하기 때문입니다.

실행을 재개하려면 입력으로 `None`을 사용하여 그래프를 호출하면 됩니다.

```python
# 그래프의 초기 실행
graph.invoke(inputs, config=config)

# 그래프가 중단점에서 멈췄다면, None을 전달하여 재개할 수 있습니다.
graph.invoke(None, config=config)
```

중단점을 추가하는 방법에 대한 전체 설명은 [이 가이드](../how-tos/human_in_the_loop/breakpoints.ipynb)를 참조하세요.

## 시각화

그래프가 더 복잡해짐에 따라 시각화하는 것이 유용할 수 있습니다. LangGraph에는 그래프를 시각화하는 여러 가지 내장된 방법이 있습니다. 자세한 내용은 [이 가이드](../how-tos/visualization.ipynb)를 참조하세요.

## 스트리밍

LangGraph는 스트리밍에 대한 일급 지원을 제공합니다. LangGraph는 여러 가지 스트리밍 모드를 지원합니다:

- [`"values"`](../how-tos/stream-values.ipynb): 그래프의 각 단계 후 상태의 전체 값을 스트리밍합니다.
- [`"updates"`](../how-tos/stream-updates.ipynb): 그래프의 각 단계 후 상태 업데이트를 스트리밍합니다. 동일한 단계에서 여러 업데이트가 이루어진 경우(예: 여러 노드가 실행된 경우) 이 업데이트는 별도로 스트리밍됩니다.
- `"debug"`: 그래프 실행 전체에서 가능한 많은 정보를 스트리밍합니다.

또한, [`astream_events`](../how-tos/streaming-events-from-within-tools.ipynb) 메서드를 사용하여 노드 내부에서 발생하는 이벤트를 스트리밍할 수 있습니다. 이는 [LLM 호출의 토큰 스트리밍](../how-tos/streaming-tokens.ipynb)에 유용합니다.