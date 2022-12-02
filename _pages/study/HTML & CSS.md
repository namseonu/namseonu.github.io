---
title: HTML & CSS
author: Nam Seonwoo
date: 2022-12-02
category: web
layout: post
---

## 목차
1. [HTML](#HTML)
2. [CSS](#CSS)


## HTML


## CSS
### 줄 간격(행간) 조정:
```css
/* 페이지 전체의 행간 변경 */
body { line-height: 행간값 }

/* 단락 <p></p> 행간 변경 */
p { line-height: 행간값 }

/* 클래스의 행간 변경 */
.className{ line-height: 행간값 }
```

참고로 행간 값을 조절하는 값의 단위는 아래와 같다.
| 단위 | 설명 | 예제 |
| ---- | ---- | ---- |
| % | 브라우저 문자 기준 크기에 대한 % 값 | `body { line-height: 150% }` |
| em | 브라우저 문자 기준을 1em으로 했을 때의 값 | `p { line-height: 0.5em }` |
| px | 픽셀 값을 입력해서 행간 조절 | `.className { line-height: 5px }` |
