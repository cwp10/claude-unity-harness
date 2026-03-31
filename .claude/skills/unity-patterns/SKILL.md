---
name: unity-patterns
description: >
  Provides Unity C# pattern code references for State Machine, Factory, Command, Observer,
  Strategy, Flyweight, Dirty Flag, Object Pool, SO Event Channel, UniTask, and MVP patterns.
  Loads automatically when writing, designing, or refactoring Unity C# code.
  Do NOT use for web, TypeScript, or non-Unity projects.
allowed-tools: Read, Glob, Grep
user-invocable: false
---

# Unity 패턴 코드 레퍼런스

## 패턴 선택 가이드
| 상황 | 패턴 |
|------|------|
| 런타임 다양한 오브젝트 생성 | 팩토리 |
| 총알·이펙트 반복 생성 | 오브젝트 풀 |
| 실행 취소·재실행·리플레이 | 커맨드 |
| 상태 3개 이상 (AI, 캐릭터) | 상태 머신 |
| 씬 간 느슨한 이벤트 통신 | 관찰자 (SO Event) |
| 런타임에 동작 교체 | 전략 |
| 대량 오브젝트 메모리 최적화 | 플라이웨이트 |
| 비용 높은 연산 최소화 | 더티 플래그 |
| 밸런스 데이터 분리 | SO_Config |
| UI와 로직 분리 | MVP |


---

## 패턴 코드 참조
각 패턴의 코드 예시:
- 상태 머신, SO Event Channel, C# 이벤트, 팩토리, 오브젝트 풀, UniTask:
  → `unity-patterns/references/advanced-patterns.md`
- 커맨드·전략·플라이웨이트·더티플래그·SO_Config·MVP·SOLID:
  → `unity-patterns/references/solid-principles.md`
