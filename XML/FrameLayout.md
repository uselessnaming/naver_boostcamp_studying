# Frame Layout

Frame Layout은 하나의 View를 배치하기 위한 Container다

물론 여러 개의 자식을 추가할 수는 있다      
`android:layout_gravity`를 지정해 자식들의 gravity를 별도로 설정해줄 수 있다        

자식 View는 Stack처럼 나중에 추가한 View를 위쪽에 그리도록 함

> Compose Box Layout과 비슷한 것 같은데?

[공식 문서](https://developer.android.com/reference/android/widget/FrameLayout)

**Q 하나의 자식을 배치한다면 왜 사용할까? 감싸지 않고 써도 될 것 같은데..**     
이는 사용 방법을 보면 확실히 이해할 수 있다     
FrameLayout은 하나의 자식을 갖는 것이 의도이니만큼 하나의 자식을 두되, 그 자식을 교체해가면서 사용하는 것이다       
결과적으로 Fragment를 표시할 Container로써 사용하는 경우가 많다     

**Q CardView나 NestedScrollView가 FrameLayout을 상속받는 것으로 갖는 특징**      

FrameLayout을 상속받는 View를 사용할 때 내부에 Layout을 하나 배치한 후에 사용해야 한다     
그렇지 않는다면 배치한 순서대로 겹겹이 겹쳐지며 원하는 layout을 볼 수 없다