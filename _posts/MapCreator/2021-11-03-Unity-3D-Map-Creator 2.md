---
layout: post
title: Unity 3D TileMapCreator 만들기 2
date: 2021-11-04 02:45 +0900
modified: 2021-11-04 02:45:49 +09:00
description: 3D TileMap
tag:
  - TileMap
  - unity
  - Custom
---
```
Unity 3.14f
```

### 1. 위치 표시하기

그리드 형태의 타일맵에서 중요한 것은 바로 현재 마우스 위치가 어디에 있느냐 입니다.  
이번 게시글에서는 그리드에 해당 마우스 위치를 표시해보도록 하겠습니다.

### 2. SceneView에서 마우스 위치 얻기

SceneView에서 마우스 좌표를 얻기 위한 방법 중 가장 쉬운 방법은
Ray를 쏴 마우스 좌표를 얻어오는 것입니다.

```C#

Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);  

Vector3 mousePosition = ray.origin - ray.direction * (ray.origin.y / ray.direction.y);  

```
함수를 통해 얻어온 Ray를 좌표로 변환합니다.

ray.origin으로 ray의 원점을 얻어오고, y값의 direction을 지워줍니다. 그러면 y 값은 0으로 고정됩니다.

이렇게 얻은 마우스 위치를 그리드의 Cell 항목에 대해 Vector2로 변환하여 값을 반올림해줍니다.

```C#

Vector2Int cell = new Vector2Int(Mathf.RoundToInt(mousePosition.x / cellSize.x),
            Mathf.RoundToInt(mousePosition.z / cellSize.y));

return cell * cellSize;

```

### 3. SceneGUI Update

SceneView에서 일어나는 변화를 실시간으로 반영하기 위해
이벤트를 등록하고 사용하도록 합니다.

```C#

private void OnSceneGUI(SceneView view)
{
    if (bPaintMode)
    {
        Vector2 cellCenter = GetSelectedCell();
        
        DisplayHelper(cellCenter);
        
        SceneView.RepaintAll();
    }
}

```

위 함수는 위치를 표시해주는 함수입니다.  
이제 이 함수를 duringSceneGui에 등록하주어야 합니다.  
그럼, Editor 창이 포커스될 때 호출되어집니다.
```
private void OnFocus()
{
    SceneView.duringSceneGui -= this.OnSceneGUI;
    SceneView.duringSceneGui += this.OnSceneGUI;
}
```

### 4. 표시하기
아까 위에서 구한 마우스 좌표로 선을 그려보겠습니다.

```

// Vertices of our square
Vector3 topLeft = cellCenter + Vector2.left * cellSize * 0.5f + Vector2.up * cellSize * 0.5f;  
Vector3 topRight = cellCenter - Vector2.left * cellSize * 0.5f + Vector2.up * cellSize * 0.5f;  
Vector3 bottomLeft = cellCenter + Vector2.left * cellSize * 0.5f  - Vector2.up * cellSize * 0.5f;  
Vector3 bottomRight = cellCenter - Vector2.left * cellSize * 0.5f - Vector2.up * cellSize * 0.5f;  

// Rendering
Handles.color = Color.green;  
Vector3[] lines = { topLeft, topRight, topRight, bottomRight, bottomRight, bottomLeft, bottomLeft, topLeft };  
Handles.DrawLines(lines);  

```

위 코드는 사각형의 정점들을 계산하여 DrawLines를 통해 다시 그려주는 함수입니다.

결과를 확인해봅니다.

<figure>
<img src="/assets/img/MapCreator/5.png" alt="5">
<figcaption></figcaption>
</figure>

잘 나오네요. 그런데 XY로 나오고 있어 제가 원하는 느낌은 아닙니다.
해당 평면을 XZ로 바꿔주겠습니다.

### 4-1. XZ 평면
먼저 위 정점들이 어떤 값을 가지는지 확인해보겠습니다.

<figure>
<img src="/assets/img/MapCreator/6.png" alt="6">
<figcaption></figcaption>
</figure>

로그를 확인해보니, Z값은 0으로 고정되고 Y값이 변화됩니다.
이를 이용하여 Y 와 Z의 Swap을 진행하면 될 것 같습니다.

```

// Vertices of our square
Vector3 topLeft = cellCenter + Vector2.left * cellSize * 0.5f + Vector2.up * cellSize * 0.5f;
Swap(out topLeft.z, ref topLeft.y);

Vector3 topRight = cellCenter - Vector2.left * cellSize * 0.5f + Vector2.up * cellSize * 0.5f;
Swap(out topRight.z, ref topRight.y);
        
Vector3 bottomLeft = cellCenter + Vector2.left * cellSize * 0.5f  - Vector2.up * cellSize * 0.5f;
Swap(out bottomLeft.z, ref bottomLeft.y);
        
Vector3 bottomRight = cellCenter - Vector2.left * cellSize * 0.5f - Vector2.up * cellSize * 0.5f;
Swap(out bottomRight.z, ref bottomRight.y);

// Rendering
Handles.color = Color.green;
Vector3[] lines = { topLeft, topRight, topRight, bottomRight, bottomRight, bottomLeft, bottomLeft, topLeft };
Handles.DrawLines(lines);

```

<figure>
<img src="/assets/img/MapCreator/7.png" alt="7">
<figcaption></figcaption>
</figure>

이제 원하는대로 표시되었습니다. 다음 게시글에선, 해당 위치에 프리팹을 생성해보겠습니다.