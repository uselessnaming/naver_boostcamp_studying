# Layout
## Constraints Layout
중첩된 뷰 그룹이 없는 플랫 뷰 계층 구조로 크고 복잡한 레이아웃을 만들 수 있다              
제약 조건을 활용하여 구성하는 것으로, 상위 레이아웃 사이의 관계에 따라 모든 레이아웃이 결정되는 점이 Relative Layout과 비슷하지만 훨씬 유연하다                

제약 조건인만큼 좌우 혹은 상하 각각 하나씩은 제약조건이 반드시 필요하다        

중첩 레이아웃이 줄어든다면, View Hierarchy가 얕아져 측정/레이아웃 연산이 빨라진다        
특히 화면이 복잡할수록 성능이 크게 차이가 난다      

**Guideline**

화면에 보이지 않는 선을 기준점으로 사용      
View를 정확한 비율 혹은 픽셀 단위로 배치할 때 유용     

```xml
<androidx.constraints.widget.Guideline
    android:id=""
    android:layout_width="wrap_content"
    android:orientation="vertical"
    app:layout_constraintGuide_percent="0.5"/>
```
와 같은 Guideline을 ConstraintsLayout 내부에 선언해서 사용하게 된다      

해당 Guideline은 실제 View에서는 보이지 않고 layout_constraintGuide_percent 는 begin, end를 설정할 수 있다       

**Barrier**

이는 내부적으로 여러 뷰의 가장자리 위치를 모아 만든 가상 기준선이다      

```xml
<androidx.constraintlayout.widget.Barrier
    android:id="@+id/barrier"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:barrierDirection="end"
    app:constraint_referenced_ids="textView1,textView2,textView3"/>
```
와 같이 Barrier를 선언해 뷰의 크기가 변할 때 따라 움직이도록 함        

barrierDirection 방향을 기준으로 constraint_referenced_ids 내부 뷰들이 정렬된다

**Constraint Group**
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView 1"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"/>

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView 2"
        app:layout_constraintTop_toBottomOf="@id/textView1"
        app:layout_constraintStart_toStartOf="parent"/>

    <androidx.constraintlayout.widget.Group
        android:id="@+id/myGroup"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="visible"
        app:constraint_referenced_ids="textView1, textView2"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```
위 코드처럼 Constraint Layout 내부에 group이 존재하는 데, group에서 constraint_referenced_ids 필드를 활용해 여러 id를 묶어 놓을 수 있다     
이렇게 하면 group id로 한 번에 visible 혹은 elevation 을 조절할 수 있다

다만 방금 말한 속성을 제외하고는 조절이 불가능하다        

Group 위젯은 실제로 존재한다기 보다는 일종의 가상 View이다.

**Flow**
Flow란 참조된 위젯을 체인과 유사하게 가로 혹은 세로로 배치할 수 있는 View 요소       
이 또한 가상의 object이며 chain을 간소화하여 사용하기 좋다      
속성 중 flow가 포함된 속성들을 조절하여 각 view들의 간격을 조절하거나, 배치 방향 등을 조절하여 사용할 수 있다

## Linear Layout
말 그대로 선형 Layout을 말한다        
여기서 핵심 속성은 orientation 속성으로 해당 Layout이 Vertical일지 Horizontal일지 정해야 한다       

정하게 되면 기본적으로 orientation 방향대로 View가 배치된다        

가장 단순하고 직관적이지만 복잡한 UI에서는 중첩이 꽤 필요하기 때문에 성능 자체가 저하될 수 있다     
따라서 적절히 섞어서 사용하는 것이 좋다

## Constraint Layout VS. LinearLayout
Constraint Layout은 하나의 flat한 계층으로 유지하여 효율이 좋다.

LinearLayout은 단방향으로 View를 나열하는 layout이다.

전체적으로 Constraint Layout이 범용성도 좋을 뿐더러 중첩 구조를 사용하지 않기 때문에 더 효율적이다.