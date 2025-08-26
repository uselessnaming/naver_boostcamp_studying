# Property will not be serialized into a 'Parcel'
이는 ```Parcelize``` 어노테이션 사용 시 Parcelable로 만드려는 속성이 기본 생성자에서 선언되지 않았기 때문에 발생하는 Warning이다.

## 해결
```@IgnoredOnParcel``` 어노테이션을 사용해서 직렬화하지 않도록 직접 지정하거나, 속성 값을 생성자로 옮겨서 직렬화될 수 있도록 수정해야 한다

**이것보다 좋은 방법**  
Custom Class에 대해 직접 Saver를 지정하여 rememberSaveable을 사용하는 것이 좋다