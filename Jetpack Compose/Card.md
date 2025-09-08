# Card

[Material Design 자료](https://m3.material.io/components/cards/overview)

## Material2와 달라진 점
+ Color: dynamic color를 활용함
+ Elevation: default에 shadow를 삭제하고, elevation을 낮춤
+ Types: Elevated, Filled, Outlined 3종류

## Specs
### Card States
3종류의 타입 모두 공통된 State를 가짐

+ Hovered
+ Focused
+ Pressed
+ Dragged
+ Disabled

### Measurements
**default**
+ shape: 12dp corner radius
+ horizontal padding: 16dp
+ padding between cards: 8dp(max)
+ label text alignment: Start-aligned

## Guidelines
### 지향
+ 하나의 주제를 그리는 것

### 지양
+ 카드를 최소한으로 써라(?)
> 이 말의 의문이 들어서 좀 더 보니, 안써도 될 부분에서 굳이 Card를 남용하지 말아라. 라는 의미

### Type 분기
background로부터 분리 정도 (강조되는 정도가 다르다)
outlined cards >> elevated cards >> filled cards

### Anatomy
![alt text](image-3.png)
1. Container
2. Headline
3. Subhead
4. Supporting Text
5. Image
6. Button

### Divider
Divider를 활용해서 Card의 확장될 수 있는 부분을 구분지을 수 있다

혹은 내용을 분리하면 좋은 부분도 Divider를 활용할 수 있다

### 주의사항: 텍스트, 아이콘, 이미지의 Layering
권장 사항은 아님        
다만 필요하다면, background 이미지와 충분한 대비를 제공해 접근성 표준을 충족하도록 해야 한다