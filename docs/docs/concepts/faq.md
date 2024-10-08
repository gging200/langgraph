# 자주 묻는 질문 (FAQ)

자주 묻는 질문과 그에 대한 답변입니다!

## LangGraph를 사용하려면 LangChain도 사용해야 하나요?

아닙니다! LangGraph는 범용 프레임워크로, 노드와 엣지는 파이썬 함수에 불과합니다. LangChain을 사용할 수도 있고, 원시 HTTP 요청이나 다른 프레임워크를 노드와 엣지 내부에서 사용할 수도 있습니다.

## LangGraph는 도구 호출을 지원하지 않는 LLM과도 작동하나요?

네! LangGraph는 어떤 LLM과도 함께 사용할 수 있습니다. 도구 호출을 지원하는 LLM을 사용하는 주된 이유는, LLM이 무엇을 해야 할지 결정하는 가장 편리한 방법 중 하나이기 때문입니다. 만약 사용 중인 LLM이 도구 호출을 지원하지 않더라도, LLM의 문자열 응답을 기반으로 할 작업을 결정하는 약간의 로직을 작성하면 계속 사용할 수 있습니다.

## LangGraph는 OSS LLM과도 작동하나요?

네! LangGraph는 내부적으로 어떤 LLM을 사용하는지에 상관하지 않습니다. 대부분의 튜토리얼에서 비공개 LLM을 사용하는 주된 이유는 도구 호출을 원활하게 지원하기 때문입니다. 그러나 도구 호출은 필수적인 기능이 아니므로(해당 [섹션](#does-langgraph-work-with-llms-that-dont-support-tool-calling)을 참고하세요), OSS LLM과 함께 LangGraph를 완전히 사용할 수 있습니다.
