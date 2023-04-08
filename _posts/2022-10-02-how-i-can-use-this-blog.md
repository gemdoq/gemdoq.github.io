---
layout: single
title: "블로그 활용 방법"
date: 2022-10-02 10:25:12 +0900
categories: knowledge
tags: [Blog, Jekyll]
typora-root-url: ../
---


## 블로그(GitHub Pages) 활용 방법
> - GitHub pages를 꾸미고 각 요소를 설정하는 방법들 정리
> - 포스팅할 때 활용할 수 있는 기능 정리

<br>

## 알아야 하는 기본 개념

### 개념0 필요한 것

1. 구글링 실력
   - 궁금할 때 구글링을 해서 쉽게 바로 얻을 수 있는 정보를 기록하는 건 낭비
1. vim 사용 지식
2. 리눅스 기본명령어 지식

### 개념1 설치 순서

- 루비설치 

- 디렉토리생성: mkdir 디렉토리 

- 디렉토리명은 관리하기 쉽게 깃허브아이디.github.io

- 디렉토리이동: cd 디렉토리

- 지킬설치: gem install jekyll 

- 번들러설치: `생략가능` 

- 지킬사이트생성: jekyll new --skip-bundle . 

- 젬파일설정(Gemfile): vi Gemfile

- 번들러실행: bundle install

- 지킬사이트테마설정(_config.yml): vi _config.yml

### 개념2 Jekyll theme

추천하는 테마는 [Just-The-Docs](https://github.com/just-the-docs/just-the-docs){:target="_blank"}, [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/){:target="_blank"}가 있다.

현재 사용중인 Jekyll Theme : [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/){:target="_blank"}

### 개념3 로컬에서 지킬사이트 구동

- 번들로 지킬사이트 구동: bundle exec jekyll serve

- 로컬 사이트 주소: localhost:4000

- 번들러 종료: Ctrl+C

<br>

## 깃허브 블로그에서 특수문자 사용하는 방법

GitHub Pages는 몇가지 특수문자들을 구현해버린다.

이런 특수문자들을 conflicting syntax라고 한다.

그래서 내가 표현하고자 하는 텍스트가 conflicting syntax여서 

표현하지 못할 때가 있다.

liquid에서는 conflicting syntax를 사용할 수 있게 하기 위한 

다음과 같은 raw문법을 제공한다.

```
{% raw %}{%{% endraw %} raw %}
conflicting syntax
{% raw %}{%{% endraw %} endraw %}
```

<br>

## 머릿말 블록(front matter block)

YAML 형식의 머릿말 블록(front matter block)을 

가지고 있는 모든 파일은 지킬이 특별한 파일 취급을 한다.

### 형식

이하 코드박스와 같이 삼중선 안에 내용을 넣는 형식이며, 

사전 정의된 변수를 설정하거나 사용자 지정 변수를 만들어 

설정을 적용할 수 있다.

```yaml
---
layout: post
title: blogging like article
---
```

### 사전정의된 전역변수

layout, permalink, published

### 사전 정의된 포스트 변수

date, categories, tags

### 커스텀 변수

예를 들어 food라는 변수명에 pizza를 정의하면 

```yaml
---
food: pizza
---
{% raw %}
<h3> {{ food.pizza }} </h3>
{% endraw %}
```

해당 페이지에서 중괄호를 겹쳐서 사용 가능

<br>

## 카테고리별 보기 기능(아카이브)

_pages에서 category-archive.md를 만들고, 

front matter에 다음과 같이 넣는다.

```yaml
---
title: "Posts by Category"
layout: categories
permalink: /categories/
author_profile: true
sidebar_main: true
---
```

_data에서 navigation.yml을 연다.

navigation.yml의 main은 상단 네비게이션 바의 요소들을 의미한다.

다음과 같이 추가한다.

```yaml
main:
  - title: "Category 🗂️"
    url: /categories/
```

이제 포스트 작성 시 카테고리를 추가하고 싶으면, front matter에 

category-archive.layout값인 categories를 key로 가지는 

원하는 카테고리 value를 넣어주면 포스트에 카테고리가 할당된다. 

<br>

## 태그별 보기 기능(아카이브)

_pages에서 tag-archive.md를 만들고, 

front matter에 다음과 같이 넣는다.

```yaml
---
title: "Posts by Tag"
layout: tags
permalink: /tags/
author_profile: true
sidebar_main: true
---
```

_data에서 navigation.yml을 연다.

navigation.yml의 main은 상단 네비게이션 바의 요소들을 의미한다.

다음과 같이 추가한다.

```yaml
main:
  - title: "Tags 🏷️"
    url: /tags/
```

이제 포스트 작성 시 태그를 추가하고 싶으면, 

front matter에 tag-archive.layout값인 tag를 key로 가지는 

원하는 태그 value를 넣어주면 포스트에 태그가 할당된다.

태그는 카테고리와 다르게 태그 value로 여러 태그값이 담긴

배열을 value로 넣어 한번에 여러 태그를 할당할 수 있다. 

<br>

## 연도별 보기 기능(아카이브)

_pages에서 year-archive.md를 만들고, 

front matter에 다음과 같이 넣는다.

```yaml
---
title: "Posts by Year"
layout: posts
permalink: /year/
entries_layout: grid
author_profile: true
---
```

_data에서 navigation.yml을 연다.

navigation.yml의 main은 상단 네비게이션 바의 요소들을 의미한다.

다음과 같이 추가한다.

```yaml
main:
  - title: "Year 📅"
    url: /year/
```

이제 포스트를 작성하면 자동으로 작성된 연도별로 분류된다. 

<br>

## 좌측사이드바 기능

_data에서 navigation.yml을 연다.

navigation.yml의 docs는 좌측 네비게이션 바의 요소들을 의미한다.

다음과 같이 추가한다.

```yaml
docs: 
  - title: "대목차1 📚"
    children:
      - title: "Category 🗂️"
        url: /categories/

  - title: "대목차2 📚"
    children:
      - title: "Tag 🏷️"
        url: /tags/
```

이제 포스트 작성 시 좌측사이드바를 추가하고 싶으면,

front matter에 다음 설정을 넣어준다.

```yaml
sidebar:
	nav: "docs"
```

이제 포스트를 클릭하면 좌측 사이드에 네비게이션으로 

docs를 활용할 수 있다.

<br>

## 공지사항 기능

{: .notice--danger}
markdown 형식의 문서에서 적용되더라

텍스트 밑에 `중괄호열고콜론띄어쓰기닷설정값중괄호닫고`를 넣어주면 된다.

예시코드는 다음과 같다.

```markdown
{: .notice--danger/success/warning/info/primary}
```

<br>

## 버튼 기능

{: .notice--danger}
markdown 형식의 문서에서 적용되더라

다음의 예시코드처럼 텍스트를 작성하면 된다.

```markdown
[Text](#link){: .btn--inverse/danger/success/warning/info/primary}
```

<br>

## 유튜브 영상추가 기능

유튜브의 영상 주소에는 영상식별번호가 있다.

`https://www.youtube.com/watch?v=HasadgwdthH`

위 주소에서 `HasadgwdthH`가 영상식별번호다.

다음 코드에 넣으면 영상식별번호에 해당되는 유튜브 영상이 첨부된다.

```markdown
{% raw %}
{% include video id="영상식별번호" provider="youtube" %}
{% endraw %}
```

<br>

## 구글 드라이브 영상 추가기능

구글 드라이브 영상에도 영상식별번호가 있다.

`https://drive.google.com/file/d/1iDkrmC75d3jDk8kSJfgHasa38dg6th/preview`

위 주소에서 `1iDkrmC75d3jDk8kSJfgHasa38dg6th`가 영상식별번호다.

다음 코드에 넣으면 영상식별번호에 해당되는 유튜브 영상이 첨부된다.

```markdown
{% raw %}
{% include video id="영상식별번호" provider="google-drive"}
{% endraw %}
```

<br>

## 이미지추가 기능

### Typora로 이미지 넣기

typora는 마크다운문서를 효과적으로 작성하는 데 유용하다.

typora 설정 - 이미지 탭 - 다음과 같이 설정한다.

![typora_config](/images/2022-10-02-how-i-can-use-this-blog/typora_config.png)

typora로 이미지를 드래그앤드롭으로 넣으면, 

블로그 디렉토리의 images라는 폴더에도 함께 복사된다.

다만 주의해야 할 점이 있다. 

블로그가 구성될 때 포스트의 카테고리를 참조하여 url을 만들기 때문에, 

이미지의 위치가 제대로 특정되지 않아 이미지가 깨지는 경우가 있다.

매 포스트를 작성시마다 front matter에 다음 설정을 해주자.

```yaml
---
typora-root-url: ../
---
```

### 기타 방법으로 이미지 넣기

typora가 유료여서 사용하는 게 부담된다면,

1. 블로그 디렉토리(루트경로)에 images 폴더를 만들고
2. 그 안에 각 포스트명과 같은 이름의 폴더를 만들고
3. 그 안에 해당 포스트에서 참조하는 이미지를 넣고
4. 해당 이미지를 참조하는 마크다운 문법을 넣는다

이미지를 참조하는 마크다운 문법은 아래와 같다.

```markdown
![이미지별명](루트경로/images/YYYY-mm-dd-포스트명/이미지명)
```

루트경로는 다른 개발환경에서도 이미지가 깨지지 않고 호환될 수 있도록 

`중괄호` 안에 다음 코드를 넣어 표현한다.

```markdown
{% raw %}
{{ site.url }}
{% endraw %}
```

<br>

## 리다이렉트 기능

포스트 정보가 바뀌었을 때 포스트 주소를 리다이렉트를 걸어주어 

연결을 유지할 수 있는 기능이다.

포스트를 작성할 때 front matter에 다음과 같이 넣어준다.

```yaml
redirect_from:
	- 원래주소 (/원래카테고리/원래_포스트_파일_제목)
```

예를 들어 예전 포스트의 카테고리가 `original`이고, 

포스트파일의 제목이 `2009-10-11-past-post.md`였다면,

바뀐 포스트의 front matter에는 이렇게 들어가야 

redirect가 정상적으로 작동한다.

```yaml
redirect_from:
	- /original/past-post
```

<br>

## 심화) 커스텀 CSS 스타일 기능

minimal mistakes 테마에는 여러 장점이 있다. 

무엇보다 sass를 다 미리 꾸며놔서 사용자가 간편하게 사용할 수 있다.

게다가 커스텀 CSS를 만들고, 필요에 따라 markdown 문서에 

간단한 코드사용으로 적용할 수 있다.

우선 _sass 폴더의 minimal-mistakes 폴더를 들어간다.

여러 scss 파일들이 있는데, 그 중 _utilities.scss파일을 열어본다.

_utilities.scss는 사용자가 필요에 따른 서식을 정의할 수 있는 파일이다.

예를 들어 내가 이미지를 삽입했는데, 크기가 너무 커서 조절하고 싶다면 

다음과 같은 순서로 커스텀 스타일을 적용하면 된다.

1. 유틸리티 서식파일(_utilities.scss)을 연다.

2. 최하단에 아래와 같이 커스텀 서식을 정의한다.
   ```scss
   // custom style
   .img-width-half {
   	width: 50%;
   }
   ```

3. 삽입한 이미지의 뒤에 커스텀 서식을 아래와 같이 적용한다.
   ```markdown
   <!-- .align-center는 이미 테마 제작자에 의해 정의되어 있음 -->
   ![이미지명](이미지주소){: .align-center .img-width-half}
   ```

<br>

## LaTeX(수식문법) 기능

포스트 작성 시, front matter에 다음과 같이 설정해주면 된다.

```yaml
use_math: true
```

위 코드가 있는 포스트에서는 사이트가 구동 시, 

_includes/head에 있는 custom.html에서 liquid문법에 따라 

_includes에 있는 mathjax-support.html파일이 호출되고, 

LaTeX 기능이 활성화된다.

테스트 문구는 다음과 같다.

$\[ x^n + y^n = z^n \]$

$y = f(x)^2$

<br>
