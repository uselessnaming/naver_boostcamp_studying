# Single Thread vs. Multi Thread
## Single Thread
Thread가 하나로 운영됩니다. 일반적으로 main thread 위에서 돌아가기 때문에, 다른 작업을 하는 동안은 동작이 멈추게 됩니다. 

## Multi Thread
Thread가 여러 개로 운영됩니다. 요즘은 multi-core이기 때문에 core당 하나의 thread를 관리합니다. 여기서 core의 개수보다 많이 thread가 생성되면, 작업하는 과정에서 core에 context switch가 발생됩니다. 이 과정에서 overhead가 발생할 수 있습니다.

![alt text](<thread 6개 사용 cpu 사용 현황.png>)

![alt text](<thread 9개 사용 cup 사용 현황.png>)

![alt text](<thread 11개 사용 cpu 사용 현황.png>)

![alt text](<thread 12개 사용 cpu 사용 현황.png>)

사진들은 사용 thread에 따른 각 Core의 사용량을 나타내는 그래프입니다. 사용량을 보면 사용 중인 thread에 비례해서 그래프의 상승폭이 높은 것을 확인할 수 있습니다. 

![alt text](<사용 thread 개수에 따른 task 처리 시간 변동 추이.png>)
thread의 개수에 따라 소요 시간이 늘어나는 것을 확인할 수 있습니다. task들이 core 개수를 넘어가지 않는 범위에서는 task들이 추가되어도 비슷한 처리 속도를 가진 것을 볼 수 있습니다. 하지만 core 개수를 넘어가는 순간부터 thread를 교체하는 데 비용이 들기 때문에 소요 시간이 크게 차이나는 것을 확인할 수 있습니다.