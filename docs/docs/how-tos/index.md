# 사용 방법 가이드

LangGraph 사용 방법 가이드에 오신 것을 환영합니다! 이 가이드들은 LangGraph에서 주요 작업을 수행하기 위한 실용적이고 단계별 지침을 제공합니다.

## 제어 가능성

LangGraph는 높은 제어 가능성으로 유명한 에이전트 프레임워크입니다.
이 사용 방법 가이드는 그러한 제어 가능성을 달성하는 방법을 보여줍니다.

- [서브그래프 생성 방법](subgraph.ipynb)
- [병렬 실행을 위한 브랜치 생성 방법](branching.ipynb)
- [병렬 실행을 위한 맵-리듀스 브랜치 생성 방법](map-reduce.ipynb)

## 상태 지속성

LangGraph는 그래프 실행 간 상태를 쉽게 지속할 수 있습니다. 아래 가이드는 그래프에 지속성을 추가하는 방법을 보여줍니다.

- [그래프에 지속성("메모리") 추가 방법](persistence.ipynb)
- [대화 기록 관리 방법](memory/manage-conversation-history.ipynb)
- [메시지 삭제 방법](memory/delete-messages.ipynb)
- [대화 요약 메모리 추가 방법](memory/add-summary-conversation-history.ipynb)
- [Postgres 체크포인터를 사용한 지속성 추가 방법](persistence_postgres.ipynb)
- [MongoDB를 사용하여 사용자 정의 체크포인터 생성 방법](persistence_mongodb.ipynb)
- [Redis를 사용하여 사용자 정의 체크포인터 생성 방법](persistence_redis.ipynb)

## 인간이 개입하는 루프

LangGraph의 주요 이점 중 하나는 인간이 개입하는 워크플로우를 쉽게 만들 수 있다는 점입니다.
이 가이드들은 그러한 일반적인 예시들을 다룹니다.

- [중단점 추가 방법](human_in_the_loop/breakpoints.ipynb)
- [그래프 상태 편집 방법](human_in_the_loop/edit-graph-state.ipynb)
- [사용자 입력을 기다리는 방법](human_in_the_loop/wait-user-input.ipynb)
- [과거 그래프 상태 보기 및 업데이트 방법](human_in_the_loop/time-travel.ipynb)
- [도구 호출 검토 방법](human_in_the_loop/review-tool-calls.ipynb)

## 스트리밍

LangGraph는 스트리밍을 우선적으로 설계했습니다.
이 가이드들은 다양한 스트리밍 모드를 사용하는 방법을 보여줍니다.

- [그래프의 전체 상태를 스트리밍하는 방법](stream-values.ipynb)
- [그래프 상태 업데이트를 스트리밍하는 방법](stream-updates.ipynb)
- [LLM 토큰을 스트리밍하는 방법](streaming-tokens.ipynb)
- [LangChain 모델 없이 LLM 토큰을 스트리밍하는 방법](streaming-tokens-without-langchain.ipynb)
- [임의로 중첩된 콘텐츠를 스트리밍하는 방법](streaming-content.ipynb)
- [동시에 여러 스트리밍 모드를 구성하는 방법](stream-multiple.ipynb)
- [도구 내에서 이벤트를 스트리밍하는 방법](streaming-events-from-within-tools.ipynb)
- [LangChain 모델 없이 도구 내에서 이벤트를 스트리밍하는 방법](streaming-events-from-within-tools-without-langchain.ipynb)
- [최종 노드에서 이벤트를 스트리밍하는 방법](streaming-from-final-node.ipynb)

## 도구 호출

- [ToolNode를 사용하여 도구 호출 방법](tool-calling.ipynb)
- [도구 호출 오류 처리 방법](tool-calling-errors.ipynb)
- [도구에 그래프 상태를 전달하는 방법](pass-run-time-values-to-tools.ipynb)
- [도구에 설정을 전달하는 방법](pass-config-to-tools.ipynb)
- [많은 수의 도구를 처리하는 방법](many-tools.ipynb)

## 상태 관리

- [Pydantic 모델을 상태로 사용하는 방법](state-model.ipynb)
- [상태에 컨텍스트 객체를 사용하는 방법](state-context-key.ipynb)
- [입력 및 출력 스키마를 분리하는 방법](input_output_schema.ipynb)
- [그래프 내부에서 노드 간에 비공개 상태를 전달하는 방법](pass_private_state.ipynb)

## 기타

- [그래프를 비동기로 실행하는 방법](async.ipynb)
- [그래프를 시각화하는 방법](visualization.ipynb)
- [그래프에 런타임 설정을 추가하는 방법](configuration.ipynb)
- [노드 재시도를 추가하는 방법](node-retries.ipynb)

## 사전 구축된 ReAct 에이전트

이 가이드는 사전 구축된 ReAct 에이전트를 사용하는 방법을 보여줍니다.
여기에서는 **사전 구축된 에이전트**를 사용할 것입니다. LangGraph의 큰 이점 중 하나는 자신만의 에이전트 아키텍처를 쉽게 만들 수 있다는 것입니다. 따라서 빠르게 에이전트를 구축하려면 여기서 시작하는 것도 좋지만, LangGraph의 모든 기능을 최대한 활용할 수 있도록 자신만의 에이전트를 만드는 방법을 배우는 것이 좋습니다.

- [ReAct 에이전트 생성 방법](create-react-agent.ipynb)
- [ReAct 에이전트에 메모리 추가 방법](create-react-agent-memory.ipynb)
- [ReAct 에이전트에 사용자 정의 시스템 프롬프트 추가 방법](create-react-agent-system-prompt.ipynb)
- [ReAct 에이전트에 인간이 개입하는 프로세스 추가 방법](create-react-agent-hitl.ipynb)