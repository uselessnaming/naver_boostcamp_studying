# Vector Resource vs PainterResource
## Vector Resource
Vector Drawable로 타입을 고정하고 세세한 Vector Drawable을 조절하고자 할 때 사용

## Painter Resource
타입을 vector drawable과 bitmap drawable에 따라 적절한 Painter를 반환하여 처리한다      

## 성능 차이
Image Vector는 `@Immutable`로 취급되는 경우가 많기 때문에 최적화에서 아이콘을 많이 사용할 경우 전환하는 것이 좋다

Painter를 반환하는데 `@Unstable`로 간주되어 Recomposition에 영향을 줄 수 있다

## 결론
성능을 고려한다면 vector asset을 처리할 때는 vector resource를 활용하는 부분이 좋고, 만약 단순한 처리가 필요하다면 painter resource를 사용해도 좋다

하지만 이런 성능 차이는 단발적으로 나타나는 부분에서는 알기 어려울 정도로 미미하다고 한다

그래서 편리하게 아무거나 써도 되는데, 이미지 관련해서 recomposition이 반복되는 상황이라면 remember와 vector resource를 사용해서 처리하면 성능상 이득을 볼 수 있다 정도로 생각하면 될 것 같다

> recomposition 과정에서 이미지가 계속 그려진다면 이렇게 하되, 이런 경우가 아니라면 굳이 만들 이유가 없다고 느껴진다