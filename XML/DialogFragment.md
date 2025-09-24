# Dialog Fragment
Dialog를 만들고 호스팅하도록 설계된 Fragment 의 확장 클래스

## Dialog의 사용
DialogFragment는 show라는 함수를 내부적으로 가지고 있다

3가지 show 함수가 존재한다

```java
public void show(@NonNull FragmentManager manager, @Nullable String tag) {
    mDismissed = false;
    mShownByMe = true;
    FragmentTransaction ft = manager.beginTransaction();
    ft.setReorderingAllowed(true);
    ft.add(this, tag);
    ft.commit();
}
```
이것은 void 타입으로 return하는 값은 없다

```java
public int show(@NonNull FragmentTransaction transaction, @Nullable String tag) {
    mDismissed = false;
    mShownByMe = true;
    transaction.add(this, tag);
    mViewDestroyed = false;
    mBackStackId = transaction.commit();
    return mBackStackId;
}
```
이 함수는 동일해보이나 transaction에 add 함수로 추가한 것에 대한 id 값을 필요로 할 때 사용한다      
`mBackStackId`에 commit한 id를 저장해두고, 내부 함수를 더 보면 `dismissInternal()`함수에서 `mBackStackId`값을 가지고 back stack을 관리해주는 것을 확인할 수 있다

```java
public void showNow(@NonNull FragmentManager manager, @Nullable String tag) {
    mDismissed = false;
    mShownByMe = true;
    FragmentTransaction ft = manager.beginTransaction();
    ft.setReorderingAllowed(true);
    ft.add(this, tag);
    ft.commitNow();
}
```
이 함수는 `commitNow()`가 큰 차이로 동기적으로 commit을 사용하기 위해 있는 함수로, showNow를 하게 되면 바로 commit이 적용되게 된다

## 장점
위의 show 함수에서 알 수 있듯이 내부적으로 Fragment manager가 back stack을 관리한다.      
따라서 configuration change가 발생해도 Fragment Manager가 이를 복원하여 화면 전환에도 Dialog가 살아있을 수 있다
그리고 Back Stack을 관리해주는 것으로 뒤로 가기 버튼에도 이를 연동시킬 수 있다      

복잡한 layout을 가진 dialog에 대해서 기본 Dialog보다 Custom 폭이 훨씬 넓다

## 사용 시점
+ Configuration Change에도 살아남아야 하는 경우
+ 복잡한 Layout을 사용하는 경우