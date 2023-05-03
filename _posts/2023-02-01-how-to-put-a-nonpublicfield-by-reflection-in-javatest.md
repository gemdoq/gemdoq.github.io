---
layout: single
title: "GitHub README 파일에 이미지나 동영상 첨부하는 방법"
date: 2023-01-28 11:32:29 +0900
categories: knowledge
tags: [README, jpg, png, gif]
typora-root-url: ../
---

## GitHub README 파일에 이미지나 동영상 첨부하는 방법
> - 링크 생성
> - 마크다운 양식
> - mp4파일을 gif파일로 변환

<br>

GitHub의 README 파일에 이미지나 동영상을 첨부하는 방법은 다음과 같다.

## 링크 생성

해당 미디어의 링크를 생성해야 하는데 가장 간편한 방법은 Repository-Issues-New issue에서 Leave a comment라는 부분에 이미지를 드래그하면 된다.

![link](/images/2023-01-28-about-github-upload-media/link.png){: width="560"}

위와 같이 링크가 생성되면 해당 링크를 복사하여 README 파일에 마크다운 양식에 맞게 붙여넣는다.

<br>

## 마크다운 양식

```markdown
<img width="{해상도 비율}" src="{이미지 경로}"/>
```

위와 같이 양식에 맞춰 해상도 비율을 정하고, 이미지 경로에 해당 링크를 붙여넣으면 정상적으로 첨부된다.

만약 README 파일에 간단한 동영상을 첨부하고 싶다면, MP4 파일을 GIF 파일로 변환하여 이미지 첨부하듯이 첨부하면 된다.

<br>

## mp4파일을 gif파일로 변환

mp4파일을 gif파일로 변환해주는 웹사이트가 있다.

🔗 웹사이트 링크 : [cloudconvert.com/mp4-to-gif](https://cloudconvert.com/mp4-to-gif)

위 사이트로 들어가면 다음과 같이 input파일 형식과 output파일 형식을 정할 수 있다. 

![index](/images/2023-01-28-about-github-upload-media/index.png){: width="560"}

파일을 추가하면 다음과 같이 전환되고, Convert를 눌러서 변환을 시작할 수 있다.

![convert](/images/2023-01-28-about-github-upload-media/convert.png){: width="560"}

변환이 완료되면 다음과 같이 다운로드 받을 수 있게 바뀐다.

![down](/images/2023-01-28-about-github-upload-media/down.png){: width="560"}

gif파일을 다운받고, 위의 방법대로 링크를 생성하여 양식에 맞게 첨부하면 된다.

<br>
