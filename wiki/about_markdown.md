

<h1>MarkDown 문법 정리 (with Typora)</h1>

> 2019.10.14 - 강민 작성
>
> 2019.10.31 - 강민 업데이트



## 문법

- 제목
  - `<h1>제목</h1>`
  - `<h2>제목</h2>`
  - `<h3>제목</h3>`
  - `<h4>제목</h4>`
  - `<h5>제목</h5>`
  - `<h6>제목</h6>`
  
- 이미지 첨부
  - `![](이미지 url 입력)`
  - `<img src="이미지 url 입력")`
  - 이미지 url 입력시 `./` 쓰는 것을 권장함 (개인적으로) 그 이유는 현재주소를 기준으로 이미지 경로를 파악할 수 있기 때문이다. 괜히 `\` 쓰다가 md파일에 이미지가 안 나오는 불상사가 벌어질 수 있다.
  
- 링크 걸기

  - HTML의 `<a>`를 이용하여 링크걸기

    예) `<a href="../README.md">README</a>` = > <a href="../README.md">README</a>

- 코드 펜스

  - Typora에서의 단축키는 shift+ctrl+k

  - 코드로 짠 부분을 넣어주고 해당 언어를 설정하면 해당 언어 방식으로 보여줌

  - ```c++
    printf("Hello world!");
    ```

- 표

  - ```
    | 항목1   | 항목2   | 항목3   |
    | ------ | ------ | ------ |
    | 내용1-1 | 내용1-2 | 내용1-3 |
    | 내용2-1 | 내용2-2 | 내용2-3 |
    | 내용3-1 | 내용3-2 | 내용3-3 |
    ```

  - | 항목1   | 항목2   | 항목3   |
    | ------- | ------- | ------- |
    | 내용1-1 | 내용1-2 | 내용1-3 |
    | 내용2-1 | 내용2-2 | 내용2-3 |
    | 내용3-1 | 내용3-2 | 내용3-3 |

