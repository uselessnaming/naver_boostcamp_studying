# Single Thread vs. Multi Thread
## Single Thread
Thread가 하나로 운영됩니다. 일반적으로 main thread 위에서 돌아가기 때문에, 다른 작업을 하는 동안은 동작이 멈추게 됩니다. 

## Multi Thread
Thread가 여러 개로 운영됩니다. 요즘은 multi-core이기 때문에 core당 하나의 thread를 관리합니다. 여기서 core의 개수보다 많이 thread가 생성되면, 작업하는 과정에서 core에 context switch가 발생됩니다. 이 과정에서 overhead가 발생할 수 있습니다.

<img width="662" height="330" alt="image" src="https://github.com/user-attachments/assets/5779dac4-8a2f-4c4e-a6b5-c1ae5959563c" />

<img width="654" height="276" alt="image" src="https://github.com/user-attachments/assets/c2f4b2b8-4ba6-44ae-91da-fb7c6304051d" />

<img width="666" height="324" alt="image" src="https://github.com/user-attachments/assets/75f09f49-6d00-4dbe-9cfa-1bdf1f0d3b87" />

<img width="664" height="332" alt="image" src="https://github.com/user-attachments/assets/c8b833df-1992-4f3e-8561-be9ca1791ba9" />

사진들은 사용 thread에 따른 각 Core의 사용량을 나타내는 그래프입니다. 사용량을 보면 사용 중인 thread에 비례해서 그래프의 상승폭이 높은 것을 확인할 수 있습니다. 

<img width="401" height="442" alt="image" src="https://github.com/user-attachments/assets/f2c7dbe6-fcf4-4c2d-a610-b2705aee1b4c" />

thread의 개수에 따라 소요 시간이 늘어나는 것을 확인할 수 있습니다. task들이 core 개수를 넘어가지 않는 범위에서는 task들이 추가되어도 비슷한 처리 속도를 가진 것을 볼 수 있습니다. 하지만 core 개수를 넘어가는 순간부터 thread를 교체하는 데 비용이 들기 때문에 소요 시간이 크게 차이나는 것을 확인할 수 있습니다.
