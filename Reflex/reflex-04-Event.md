# [Reflex Event]
> Reflex로 만들어보는 파이썬 웹애플리케이션(https://wikidocs.net/book/16592)

## 핵심 개념 (Insight)
### 1. Event Trigger
> 버튼 클릭, API 요청 발생, 파일 업로드 시작, DB 변경 등
- 이벤트를 발생시키는 것
- `on_change`: 실시간으로 모든 값을 계속해서 백엔드로 보냄 (트래픽 多)
- `on_value_commit`: 마우스 버튼을 놓는 시점의 최종 값 하나만 백엔드로 전송

#### 이벤트 트리거 하나에 여러 이벤트 핸들러를 연결하는 방법
- `list` 활용: `on_click` 파라미터에 리스트를 넣고, 리스트 안에 여러 개의 핸들러를 붙이는 방법 (`async`)
- 이벤트 핸들러를 **묶은** 이벤트 핸들러 (`yield`)

### 2. Event Handler
> 트리거에 반응해서 실제 일을 수행
- 이벤트가 발생했을 때 실행되는 로직
- `set_`: State의 특정 값을 바꾸는 작업은 빈번한 편이라, **미리 정의 된** 이벤트 핸들러를 제공

#### 임의의 값을 이벤트 핸들러에 추가로 보내는 방법
- `lambda` 활용: 트리거의 람다함수가 보내는 인자 갯수와 받는 인자 갯수가 동일해야 함
- `Events with Partial Arguments`: 이벤트 트리거에 이벤트 핸들러를 호출`()`

#### 이벤트 도중에도 이벤트를 보내는 방법: `yield`
- 이벤트 핸들러가 다른 이벤트 핸들러를 **비동기**로 실행하기 위해서는 `return 이벤트` 활용

### 3. 내장된 특수 이벤트
- `rx.console_log`
- `rx.scroll_to`
- `rx.redirect`
- `rx.set_clipboard`
- `rx.set_value`
- `rx.window_alert`
- `rx.download`: `download`는 이벤트 핸들러지만, `upload`는 컴포넌트임

### 4. 백그라운드 작업(Background Task)
> 오랜 시간 실행되어야 하는 작업을 별도의 비동기 작업으로 수행하여, UI가 평소처럼 작동할 수 있도록(블로킹이 되지 않는) 해주는 이벤트
- 비동기는 잠재적인 오류를 피하기 위해 **동일한 작업을 수행 할 때, task의 수를 제한하는 등** 추가 기능도 필요함

### 5. 이벤트액션 적용 방법
- `prevent_default`: 의도적으로 브라우저상의 DOM 요소의 기본 동작 차단 or 변경
```js
// 폼 제출 방지
document.getElementById('form').addEventListener('submit', (event) => {
    event.preventDefault();  // 기본 폼 제출 방지
    console.log('폼 제출 대신 JavaScript로 처리');
});

// 링크()의 기본 내비게이션 방지
document.querySelector('a').addEventListener('click', (event) => {
    event.preventDefault();  // 기본 링크 이동 방지
    console.log('링크 대신 커스텀 동작 실행');
});
```

- `stop_propagation`: 이벤트 전파를 중단
- `throttle`: 특정 시간 내에 한 번만 이벤트가 발생하도록 조절
- `debounce`: 주기가 아닌 **지연 시간** 적용 → 최종 값이 이벤트 핸들러로 넘어갈 수 있게 보장


## 코드 / 예시
```python
# 컴포넌트의 값을 다시 백엔드로 보내는 이벤트
class State(rx.State):
    value: int = 0

    @rx.event  # 이벤트 핸들러: 이벤트 실행시 실행되는 로직
    def update_value(self, slide_val: list):
        # 버전 업데이트로 list 기대함
        self.value = slide_val[0]

@rx.page()
def index():
    return rx.vstack(
        rx.heading(State.value),
        rx.slider(
            default_value=State.value,
            on_change=State.update_value                  # 이벤트 트리거: 이벤트 발생
            # on_change=State.update_value.throttle(100)  # 스로틀을 추가해서, 몇 밀리초마다 값을 전송할지 결정
            # on_change=State.set_value                   # 제공되는 사전 이벤트 핸들러 활용
            ),
        align="center"
    )
```

```python
# lambda 활용
class State(rx.State):
    colors: list[str] = [
        "rgba(245, 168, 152, 0.9)",
        "MediumSeaGreen",
        "#DEADE3"
    ]

    @rx.event
    def change_color(self, color: str, index: int):  # 이벤트핸들러에서는 자료형을 꼭 명시해야 함
        self.colors[index] = color

@rx.page()
def index():
    return rx.hstack(
        rx.input(
            default_value=State.colors[0],
            on_blur=lambda c: State.change_color(c, 0),
            bg=State.colors[0]
        ),
        rx.input(
            default_value=State.colors[1],
            on_blur=lambda c: State.change_color(c, 1),
            bg=State.colors[1]
        ),
        rx.input(
            default_value=State.colors[2],
            on_blur=lambda c: State.change_color(c, 2),
            bg=State.colors[2]
        )
    )
```

```python
# list를 활용해 트리거에 여러개의 핸들러를 연결
@rx.page("/")
def index():
    return rx.vstack(
        rx.hstack(
            rx.text("A", color=State.color_a),
            rx.text("B", color=State.color_b),
            rx.text("C", color=State.color_c),
        ),
        rx.button("Change", on_click=[
                  State.change_a, State.change_b, State.change_c])
    )

# 세 개의 이벤트 핸들러 메서드를 같이 실행하는 핸들러 추가
class State(rx.State):
    ... 생략 ...
    @rx.event
    def change_all(self):
        # self.change_a()
        # self.change_b()
        # self.change_c()
        yield State.change_a
        yield State.change_b
        yield State.change_c
```

```python
# prevent_default 적용 예시
@rx.page("/")
def index():
    return rx.link(
        "This Link Does Nothing",
        href="https://reflex.dev/",
        on_click=rx.prevent_default,
    )

@rx.page("/")
def index():
    return rx.link(
        "This Link Does Nothing But Changes Color Itself.",
        href="https://reflex.dev/",
        color_scheme=State.color,
        on_click=State.change_color.prevent_default
    )
```
