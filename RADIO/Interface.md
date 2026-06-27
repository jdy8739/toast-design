# 🔌 Interface

> 토스트 라이브러리의 공개 API 및 사용 패턴

---

## 📦 코어 로직

토스트 기능은 완전 다른 도메인과 분리하여 라이브러리 형식의 feature로 모듈화 및 다른 도메인과 격리시킨다.

```js
const TOAST_QUEUE = [];

const pushToQueue = (config) => {
    TOAST_QUEUE.push(config);
};

const removeFromQueue = (id) => {
    const idx = TOAST_QUEUE.findIndex(t => t.id === id);
    if (idx !== -1) TOAST_QUEUE.splice(idx, 1);
};

const openToast = ({ id, duration = 3000, ...props }) => {
    const config = { id: id ?? crypto.randomUUID(), ...props };

    pushToQueue(config);

    setTimeout(() => removeFromQueue(config.id), duration);
    return config.id;
};
```

> 실제 렌더링은 큐를 구독하는 전용 컨테이너가 ReactDOM.createRoot 로 수행한다.

---

## 🧩 사용 패턴

### 정적 사용

도메인 로직에서 직접 `openToast()`를 호출하여 정적으로 토스트를 띄운다.

```js
import { openToast } from "@toast";

function ProfileSaved() {
    const handleSave = async () => {
        await saveProfile();
        openToast({ message: "프로필 저장 완료", meaning: "success", position: "bottom-right" });
    };
}

function SessionExpired() {
    const handleAction = () => {
        openToast({ message: "세션이 만료되었습니다", meaning: "error", exposeCloseButton: true, position: "top-right" });
    };
}
```

### 동적 사용

#### Axios 인터셉터

```js
import { openToast } from "@toast";

axios.interceptors.response.use(
    (response) => response,
    (error) => openToast({ message: error.message, duration: 2000, exposeCloseButton: true, position: "top-right" })
);
```

#### TanStack Query 캐시 훅

```js
import { openToast } from "@toast";

const queryClient = new QueryClient({
    queryCache: new QueryCache({
        onError: (error) =>
            openToast({ message: error.message, meaning: "error", position: "top-right" }),
        onSuccess: () =>
            openToast({ message: "데이터 갱신 완료", meaning: "success", position: "bottom-right" }),
    }),
    mutationCache: new MutationCache({
        onError: (error) =>
            openToast({ message: error.message, meaning: "error", position: "top-right" }),
        onSuccess: () =>
            openToast({ message: "변경사항 저장 완료", meaning: "success", position: "bottom-right" }),
    }),
});
```
