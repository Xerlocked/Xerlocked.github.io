---
layout: post
title: Unity 3D TileMapCreator 만들기 3
date: 2021-11-05 02:45 +0900
modified: 2021-11-05 02:45:49 +09:00
description: 3D TileMap
tag:
  - TileMap
  - unity
  - Custom
---
```
Unity 3.14f
```

### 0. 개요

프리팹을 통한 생성은 3단계로 나누었습니다.  
1. 폴더를 지정하고 해당 폴더 안에 있는 프리팹 목록을 가져옵니다.
2. 가져온 프리팹 정보를 GUI 창에 표시합니다.
3. 선택된 프리팹을 마우스 위치의 SceneView에 생성합니다.


### 1. 폴더 지정 후 프리팹 가져오기

프리팹은 삽입 삭제가 자유로운 List 형태로 저장하고, 폴더의 경로는 문자열이므로 다음과 같은 코드를 사용하겠습니다.  

```C#
[SerializeField] 
private List<GameObject> palette = new List<GameObject>();  
string path = "Assets/MapCreator/Prefabs";
```

생성한 리스트에 자료를 넣어주기 위해 다음과 같은 메소드를 작성합니다.  

```C#
private void RefreshPalette()
{
    palette.Clear();

    string[] prefabFiles = System.IO.Directory.GetFiles(path, "*.prefab");

    foreach (string prefabFile in prefabFiles)
    {
        palette.Add(AssetDatabase.LoadAssetAtPath(prefabFile, typeof(GameObject)) as GameObject);
    }
}
```

1. 리스트를 초기화합니다.
2. 경로의 있는 .prefab확장자를 가진 파일들을 가져옵니다.
3. 리스트에 하나씩 추가해줍니다.

> `AssetDatabase.LoadAssetAtPath` 는 경로에 있는 에셋을 지정된 타입으로 로드합니다.

### 2. 프리팹 정보를 GUI에 표시하기

유니티 에디터에서는 사용자가 GUI를 커스텀할 수 있도록 `OnGUI`라는 메소드를 제공하고 있습니다.  

이를 이용해 GUI창을 꾸며보도록 하겠습니다.

먼저 우리가 가져온 프리팹은 하나의 정보 덩어리입니다. 이는 사용자에게 직관적으로 보이지 않습니다. 그러므로 정보를 가공하도록 하겠습니다.  

먼저 썸네일을 저장할 리스트를 만듭니다.  

```C#
List<GUIContent> paletteIcons = new List<GUIContent>();

foreach (GameObject prefab in palette)
{
	Texture2D texture = AssetPreview.GetAssetPreview(prefab);
	paletteIcons.Add(new GUIContent(texture));
}
```

List를 GUIContent 형태로 저장하는 이유는 나중에 사용할 SelectGrid라는 메소드가 GUIContent를 인자로 받고 있기 때문입니다.  

<figure>
<img src="/assets/img/MapCreator/3-1.png" alt="1">
<figcaption></figcaption>
</figure>

이 메소드를 사용하는 이유는 콘텐츠를 표시할 뿐아니라 우리가 선택한 콘텐츠의 인덱스를 반환해주기 때문에 사용하게 되었습니다.  

```C#

[SerializeField] 
private int paletteIndex;
...
paletteIndex = GUILayout.SelectionGrid(paletteIndex, paletteIcons.ToArray(), 6, GUILayout.Height(60f));

```

인덱스 변수하나를 만든 후, 선택된 프리팹을 가져옵니다.

<figure>
<img src="/assets/img/MapCreator/3-2.png" alt="2">
<figcaption></figcaption>
</figure>

GUI에 가져온 프리팹이 나타나는 것을 볼 수 있습니다.

### 3. 선택된 프리팹을 SceneView에 올리기

생성을 하기 위해서는 사용자가 마우스를 눌렀는지 판단해야 합니다.
이는 Event 클래스를 이용하여 판단할 수 있습니다.

```C#
if (Event.current.type == EventType.Layout)
{
	HandleUtility.AddDefaultControl(0);
}
```
해당 코드를 통해 SceneView에 마우스의 입력이 들어왔는지 판단하게 됩니다.

```C#
if (paletteIndex < palette.Count && 
            Event.current.type == EventType.MouseDown && 
            Event.current.button == 0)
{
    GameObject go = PrefabUtility.InstantiatePrefab(palette[paletteIndex]) as GameObject;
    go.transform.position = new Vector3(cellCenter.x, 0, cellCenter.y);
    go.transform.parent = TileSpawner;
}
```

이후, 마우스가 눌러졌고 왼쪽버튼일 경우 해당 위치에 프리팹을 생성하게 됩니다.


<figure>
<img src="/assets/img/MapCreator/3-3.gif" alt="3">
<figcaption></figcaption>
</figure>