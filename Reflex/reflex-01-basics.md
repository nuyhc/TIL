# [Reflex Basics]
> Reflex로 만들어보는 파이썬 웹애플리케이션(https://wikidocs.net/book/16592)

## 핵심 개념 (Insight)
- 페이지 안에서 동적인 정보(변수)들은 전부 ```State```클래스 안에 담아야 함
- 이벤트: GUI 라이브러리에서 사용자가 특정 동작을 했을 때, 동작에 반응하는 동작 → 메서드 (이벤트트리거)
  * 컴포넌트 이벤트(클릭, 마우스오버, 키입력 등)
  * 이벤트핸들러(이벤트 발생시 실행되는 메서드)
- ```State``` 클래스의 변수를 ```Var```라고 부름
  * ```State``` 내에서 정의하는 모든 멤버변수(```Var```)마다 **```State.set_변수명```**이라는 메서드가 자동으로 만들어짐

## 코드 / 예시
```python
# Make it count!
import reflex as rx

class State(rx.State):
    num = 0

    def 감소(self):
        self.num -= 1

    def 증가(self):
        self.num += 1


def index():
    return rx.hstack(
        rx.button("감소", on_click=State.감소, color_scheme="ruby"),
        rx.text(State.num, font_size="1.5em"),
        rx.button("증가", on_click=State.증가, color_scheme="grass")
    )


app = rx.App()
app.add_page(index)
```

```python
rx.foreach(iterable, render_fn)  # list
```

```python
# TODO-list w/ CRUD

import reflex as rx


class State(rx.State):
    todo: str
    todo_list: list[str] = ["영어공부", "수학공부", "코딩연습"]
    completed: list[bool] = [False, False, False]
    on_edit: list[bool] = [False, False, False]

    def add_todo(self):
        self.todo_list.insert(0, self.todo)
        self.completed.insert(0, False)
        self.on_edit.insert(0, False)

    def pop_todo(self, idx: int):
        self.todo_list.pop(idx)
        self.completed.pop(idx)
        self.on_edit.pop(idx)

    def update_todo(self, x: dict):
        """
        x는 {"0": "new_todo"} 형식의 dict 타입.
        dict 키인 "0"을 정수로 바꿔서 인덱스로 활용하고 값으로 "할일" 문자열을 업데이트함
        """
        i, val = x.popitem()
        i = int(i)
        self.todo_list[i] = val
        self.on_edit[i] = False

    def toggle(self, idx: int):
        self.completed[idx] = not self.completed[idx]

    def toggle_edit(self, idx: int):
        self.on_edit[idx] = not self.on_edit[idx]


def render_fn(item: str, idx: int):
    return rx.hstack(
        rx.cond(
            State.completed[idx],
            rx.icon("circle-check-big", size=10, on_click=lambda: State.toggle(idx)),
            rx.icon("circle", size=10, on_click=lambda: State.toggle(idx))
        ),
        rx.cond(
            State.completed[idx],
            rx.text(item, style={"text-decoration": "line-through"}),
            rx.cond(
                State.on_edit[idx],
                rx.form(
                    rx.input(default_value=item, autofocus=True, on_change=State.set_todo, name=idx.to_string()),
                    on_submit=State.update_todo,
                    reset_on_submit=True
                ),
                rx.text(item, on_click=lambda: State.toggle_edit(idx))
            )
        ),
        rx.icon("trash", size=10, on_click=lambda: State.pop_todo(idx)),
        align="center"
    )


def index():
    return rx.container(
        rx.form(
            rx.input(on_change=State.set_todo),
            on_submit=lambda x: State.add_todo(),
            reset_on_submit=True
        ),
        rx.foreach(State.todo_list, lambda item, idx: render_fn(item, idx))
    )


app = rx.App()
app.add_page(index)
```
