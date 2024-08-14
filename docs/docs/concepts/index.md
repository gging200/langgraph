# 개념 가이드

이 가이드에서는 LangGraph를 사용하여 에이전트 기반 및 다중 에이전트 시스템을 구축하는 데 필요한 개념을 탐구합니다. 이 가이드는 [소개 튜토리얼](../tutorials/introduction.ipynb)에서 다룬 기본 내용을 이미 숙지했으며, LangGraph의 기본 설계와 내부 작동 방식에 대한 이해를 심화하고자 하는 사용자를 대상으로 합니다.

이 개념 가이드는 세 부분으로 나뉩니다. 먼저, 에이전트 기반이 무엇을 의미하는지 높은 수준에서 논의합니다. 다음으로, 자신만의 에이전트 시스템을 구축하는 방법을 이해하는 데 핵심이 되는 LangGraph의 저수준 개념을 살펴봅니다. 마지막으로, 일반적인 에이전트 패턴과 LangGraph로 이를 어떻게 구현할 수 있는지 논의할 것입니다. 이 가이드는 주로 개념적인 내용에 집중하며, 더 기술적이고 실질적인 가이드가 필요하다면 [사용 방법 가이드](../how-tos/index.md)를 참고하십시오.

## 에이전트 기반 애플리케이션을 위한 LangGraph

- [에이전트 기반이란 무엇을 의미하는가?](high_level.md#what-does-it-mean-to-be-agentic)
- [왜 LangGraph인가](high_level.md#why-langgraph)
- [배포](high_level.md#deployment)

## 저수준 개념

- [그래프](low_level.md#graphs)
    - [StateGraph](low_level.md#stategraph)
    - [MessageGraph](low_level.md#messagegraph)
    - [그래프 컴파일하기](low_level.md#compiling-your-graph)
- [상태](low_level.md#state)
    - [스키마](low_level.md#schema)
    - [리듀서](low_level.md#reducers)
    - [MessageState](low_level.md#working-with-messages-in-graph-state)
- [노드](low_level.md#nodes)
    - [`START` 노드](low_level.md#start-node)
    - [`END` 노드](low_level.md#end-node)
- [엣지](low_level.md#edges)
    - [일반 엣지](low_level.md#normal-edges)
    - [조건부 엣지](low_level.md#conditional-edges)
    - [진입점](low_level.md#entry-point)
    - [조건부 진입점](low_level.md#conditional-entry-point)
- [전송](low_level.md#send)
- [체크포인터](low_level.md#checkpointer)
- [스레드](low_level.md#threads)
- [체크포인터 상태](low_level.md#checkpointer-state)
    - [상태 가져오기](low_level.md#get-state)
    - [상태 기록 가져오기](low_level.md#get-state-history)
    - [상태 업데이트](low_level.md#update-state)
- [구성](low_level.md#configuration)
- [시각화](low_level.md#visualization)
- [스트리밍](low_level.md#streaming)

## 일반적인 에이전트 패턴

- [구조화된 출력](agentic_concepts.md#structured-output)
- [도구 호출](agentic_concepts.md#tool-calling)
- [메모리](agentic_concepts.md#memory)
- [인간이 개입하는 루프](agentic_concepts.md#human-in-the-loop)
    - [승인](agentic_concepts.md#approval)
    - [입력 대기](agentic_concepts.md#wait-for-input)
    - [에이전트 작업 편집](agentic_concepts.md#edit-agent-actions)
    - [시간 여행](agentic_concepts.md#time-travel)
- [맵-리듀스](agentic_concepts.md#map-reduce)
- [다중 에이전트](agentic_concepts.md#multi-agent)
- [계획 수립](agentic_concepts.md#planning)
- [반성](agentic_concepts.md#reflection)
- [상용 ReAct 에이전트](agentic_concepts.md#react-agent)