# [Reflex State]
> Reflex로 만들어보는 파이썬 웹애플리케이션(https://wikidocs.net/book/16592)

## 핵심 개념 (Insight)
- 리플렉스 웹앱에서 `State`를 제외한 다른 코드는 **고정** 됨
    - 동적으로 변경되는 모든 내용에 대한 코드는 `State` 안에 정의되어 있어야 함
- 변수를 바꿔주는 함수 = `이벤트 핸들러`
- `@rx.event`: State 클래스의 이벤트핸들러 메서드 위에 붙이는 데코레이터 → 명시적 & 매개변수 갯수나 타입 체크 등 정적 검사 기능 수행
- 모든 접속자의 환경에서 State 클래스의 인스턴스가 독립적으로 생성
    - 모든 이벤트는 서버상의 개별 State 인스턴스로 전송
- `Var`: State 클래스 안에서 정의된 변수를 통틀어 부름
    - `Base Var`: 파이썬의 원시 자료형 → `json`으로 직렬화할 수 있는 자료형
    - `Computed Var`: 어떤 연산이나 가공을 거쳐 결과가 만들어지는 `Var` → 함수의 모양
    - `Backend Var`: State 클래스 안에서만 사용되고, 프론트엔드와 통신하지 않는 변수들, `_변수명`으로 선언
        - **캐시 적용** 기능: `@rx.var(cache=True)`
    - `Custom Var`

## 코드 / 예시
```python
# Backend Var
class State(rx.State):
    # init_int: int = 0
    _init_int: int = 0

    @rx.event
    def plus_one(self):
        # self.init_int += 1
        # print(self.init_int)
        self._init_int += 1
        print(self._init_int)

@rx.page()
def index():
    return rx.vstack(
        # rx.heading(State.init_int),
        rx.heading(State._init_int),               # 프론트엔드와 동기화되지 않음,
        rx.button("+1", on_click=State.plus_one),  # 임의의 이벤트핸들러를 통해 값을 변경해도 화면에서는 변경되지 않음
        align="center"
    )
```

```python
# Computed Var
class State(rx.State):
    raw_text: str = "hi"

    @rx.var  # Computed Var
    def upper_text(self) -> str:
        return self.raw_text.upper() if self.raw_text else "EMPTY"

@rx.page()
def index():
    return rx.vstack(
        rx.heading(State.upper_text),
        rx.input(
            on_change=State.set_raw_text,
            placeholder="Type here"
        ),
        align="center"
    )
```

```python
# Custom Var
class Translation(rx.Base):
    original_text: str
    en_text: str
    es_text: str
    ja_text: str

# @dataclass
# class Translation:
#     original_text: str
#     ...

class State(rx.State):
    input_text = "안녕하세요"
    cur_tr: Translation(
        original_text = "",
        en_text = "",
        es_text = "",
        ja_text = ""
    )

    @rx.event
    def translate(self):
        en_text = (
            ~~
            .translate(self.input_text, dest="en"),
            .text
        ),
        es_text = ~~

        # 중략

        self.cur_tr = Translation(
            orginal_text=self.input_text,
            en_text=en_text,
            ...
        )
```
