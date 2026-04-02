# Unity 비동기 처리 & 오브젝트 풀링

## 비동기 처리

```csharp
// Awaitable (Unity 6 권장 — 메인 스레드 유지, Task.Delay 대체)
public async Awaitable DelayedAction(float seconds, CancellationToken ct)
{
    await Awaitable.WaitForSecondsAsync(seconds, ct);
    Execute();
}

// 씬 로드 — Coroutine
private IEnumerator LoadSceneCoroutine(string sceneName)
{
    AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);
    while (!op.isDone) { yield return null; }
}
```

> `Task.Delay` 사용 금지 → `Awaitable.WaitForSecondsAsync` 대체 (Unity 6).
> CancellationToken 미전달 시 씬 전환 후 계속 실행됨 — Warning.

---

## 오브젝트 풀링 (UnityEngine.Pool)

반복 생성·삭제 오브젝트는 `ObjectPool<T>` 필수 (Unity 6 내장):

```csharp
using UnityEngine.Pool;

private ObjectPool<Bullet> m_pool;

private void Awake()
{
    m_pool = new ObjectPool<Bullet>(
        createFunc:      () => Instantiate(m_bulletPrefab),
        actionOnGet:     b  => b.gameObject.SetActive(true),
        actionOnRelease: b  => b.gameObject.SetActive(false),
        actionOnDestroy: b  => Destroy(b.gameObject),
        defaultCapacity: 10, maxSize: 50
    );
}
```
