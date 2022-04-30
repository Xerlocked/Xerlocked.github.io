---
layout: post
title: Unity 3D TileMapCreator 만들기
date: 2021-11-03 02:45 +0900
modified: 2021-11-03 02:45:49 +09:00
description: 3D TileMap
tag:
  - TileMap
  - unity
  - Custom
---
```
Unity 3.14f
```

### 1. 구상

구글에 Level builder 혹은 Map Builder를 검색하면 대부분 2D(Tilemap) 관한 내용이 주를 이루고 있습니다. 혹은 유니티에서 만든 probuilder에 관한 게시글이 나옵니다.  

하지만 기획자가 요구한 느낌의 에디터는 조금 다른 스타일이었습니다.


<figure>
<img src="/assets/img/MapCreator/0.png" alt="0">
<figcaption>2D 타일맵</figcaption>
</figure>

<figure>
<img src="/assets/img/MapCreator/1.png" alt="1">
<figcaption>3D 타일맵</figcaption>
</figure>

제가 구상한 에디터는 `그리드` + `EditorWindow` + `쉬운 컨트롤 패널` 구성이었습니다.  

여기서 EditorWindow는 CustomEditor와 다르게 독립된 하나의 윈도우 패널을 가지는 형태입니다.
해당 내용은 <https://docs.unity3d.com/ScriptReference/EditorWindow.html>

***

### 2. 레이아웃 구성
<figure>
<img src="/assets/img/MapCreator/2.png" alt="2">
<figcaption></figcaption>
</figure>

작업을 들어가기전, 원하는 레이아웃 구성을 찾아보았습니다.
크게 입력부분, 프리팹 구성부분, 기타 설정부분 3가지로 나누어 볼 수 있습니다. 이 형태를 바탕으로 레이아웃을 구성하였습니다.

***

### 3. 실전

> Editor코드를 작성하기 위해선,  
> Asset폴더 밑, 원하는 곳에 Editor폴더를 만들고 작업을 해야 됩니다.

{% gist 989188a0c42c9965bbb2a3adc29ee020 %}

코드를 작성 후, 유니티 에디터 상단 탭을 보면 TileEditor라는 탭이 생겨져 있습니다. 탭을 선택 후, ShowEditor를 선택하면

<figure>
<img src="/assets/img/MapCreator/3.png" alt="3">
<figcaption></figcaption>
</figure>

해당 사진과 같이 패널이 열리게 됩니다.

이후 아래 코드를 넣어서 레이아웃 구성을 마무리 합니다.

{% gist a3da3d2694593b3c4f92479aec3038d7 %}

<figure>
<img src="/assets/img/MapCreator/4.png" alt="4">
<figcaption></figcaption>
</figure>