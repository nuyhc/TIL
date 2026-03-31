# [Reflex Basics]
> Reflex로 만들어보는 파이썬 웹애플리케이션(https://wikidocs.net/book/16592)

## 핵심 개념 (Insight)
- ```app.add_page(함수이름, route="/주소")```
  * 함수 이름이 ```index```인 경우 ```route```의 기본값은 ```"/"```가 됨
  * ```route``` 파라미터를 지정하지 않으면 **함수 이름**이 자동 할당 됨
- ```@rx.page()```로도 페이지 지정 가
- ```rx.heading```을 통해 URL의 경로 파라미터를 보여주는 **동적 라우팅** 가능
- 다운로드 시점에 파일을 생성하여 제공하기 위해 ```data``` 파라미터 사

## 코드 / 예시
```python
# 동적 라우팅
@rx.page(route="/[id]")  # "/[id]" 대신에 "/[q]", "/[val]" 등 아무 변수명이든 입력해도 됨
def index():
    return rx.heading(State.id)  # 단, 대괄호 안의 이름과 State변수명은 일치해야 함.

# 점을 세 개 붙이면 경로파라미터가 몇 개든지 State.param 변수에 각 파라미터의 문자열 리스트가 저장
@rx.page(route="/[...id]/")
def index():
    return rx.vstack(
        rx.heading(rx.State.id)
    )
```

```python
# 다운로드 시점 파일 생성
class State(rx.State):
    def download_random_data(self):
        return rx.download(
            data=",".join(
                [str(random.randint(0, 100)) for _ in range(10)]
            ),
            filename="random_numbers.csv",
        )

@rx.page()
def index():
    return rx.vstack(
        rx.heading("Download random numbers"),
        rx.button(
            "Download",
            on_click=State.download_random_data,
        )
    )
```
