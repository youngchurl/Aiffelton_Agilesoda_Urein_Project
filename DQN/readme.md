## 5.24 일 기준 PATCH ✒️

### 🌏 환경

1. 보상 : move : -1 obstacle : -2 items : 1000
2. 에이전트 : 0.3 아이템 : 0.6
3. 아이템이 당장 목표 한 개씩만 나오는 형태 (마지막 반환지까지 표현)

### 😗 에이전트


1. 하이퍼 파라미터  
- learning_rate = 0.0005
- gamma = 0.99
- buffer_limit  = 50000
- batch_size = 32
- interval = 50
- a_step = 300
- buffer_size > 4000: train

2. 레이어
- Conv2d 레이어 2계층의 CNN 사용 (Atari reinforcement 논문 참고)

## 5.25 일 기준 PATCH ✒️

### 🌏 환경

1. 보상   
move_reward = -1   
obs_reward = -10   
out_reward = -10   
goal_reward = 1000   
noitem_reward = -0.1 -> 추가.   
- 아이템이 없는곳을 장애물로 취급하는건 그 장소의 보상 점수를 깎는것이라 생각했고 그걸 줄이려고 추가.   

2. 레이어   
- Conv2d 채널 수를 16-32애서 6-16으로 줄여봄.   

3. 처음에만 위로 올라가는 것에서, [9,4]에 에이전트가 들어가면 무조건 위로 올라가도록 수정.   

4. 아이템의 순서를 없앴음.   
5. 아이템을 다 먹고, 다시 출발지로 도착해서 에피소드를 끝냈을 때의 보상 추가.   
6. 1번 에피소드만 반복하는 코드.


## 5.31 일 기준 PATCH ✒️

### 🌏 환경

1. 보상  
move     :  0 
obstacle : -1  
items    : 1  
clear    : 10
- 완수 했을 때 제공되는 clear 데이터 추가
- 아이템이 없는 곳은 장애물 취급

2. 채널 추가
- 장애물 0, 길 1, 아이템 2, 에이전트 3 으로 설정한 뒤 4채널로 원핫인코딩 진행

한동대 학생 프로젝트에서 착안했다 [해당 코드 38줄](https://github.com/choyi0521/snake-reinforcement-learning/blob/master/snake.py)

3. 처음에만 위로 올라가는 것으로 재수정


### 😗 에이전트


1. 하이퍼 파라미터  
- learning_rate = 0.01 (default)
- gamma = 0.99
- buffer_limit  = 100000 **(5만보다 확실히 학습이 잘된다)**
- batch_size = 128 **(64 에서 128 사이가 학습이 잘된다)**
- interval = 100
- a_step = 500
- buffer_size > 4000: train
- epsilon : 0.5 에서 200회당 0.02씩 감소, 최소 0.1 

2. 레이어
- dropout 추가
- 최적화 함수 : Rmsprop, 손실 함수 : MSE 

### ⚠️ <span style='color:red'> 시사점 </span>

모두 동일한 상태에서 

아이템에 순서가 있는 경우 vs 없는 경우

를 비교해 보았다.


| Epi |순서 고려 시 Score 평균| 순서 고려 X Score 평균|
|--|--|--|
|400|-250|-75|
|800|-310|-69|
|1200|-310|-60|
|1600|-320|-34|
|2000|-347|-48|
|2400|-356|-40|
|2800|-370|-35|

순서 고려의 경우 학습 진행도 원활하지 않았다. (오히려 더 못한다)
초반 랜덤한 움직임이 있을 때,  
clear 할 수 있는 가능성이 너무 떨어지기 때문이라고 추측한다.

> **아이템의 순서를 고려하지 않았을 때 학습성능이 좋다 !**


이후에 보상값의 크기가 영향을 주는지 보기 위해  
학습 시 보상값을 100으로 나누어 보았다.  
*(바닥부터 시작하는 딥러닝에서는 cartpole DQN 학습 시 보상을 100으로 나눈다.)*

(두 경우 모두 순서 고려 X)

| Epi (평균 score) |보상/100| 보상 그대로|
|--|--|--|
|400|-87|-75|
|800|-79|-69|
|1200|-67|-60|
|1600|-63|-34|
|2000|-50|-48|
|2400|-51|-40|
|2800|-41|-35|
|3200|-47|-27|

보상을 나눈 경우에는 score 가 더 천천히 줄었다.  
성능 자체는 크게 차이가 없는 듯 하다

Epi 가 하나였으니 문제가 쉬웠지만, 
모든 Epi 를 돌릴 때는 Score 가 천천히 줄어드는 r / 100 방식을  
고려할 법 하다.

> **보상을 나누는 방식은 학습률이 낮아지지만 학습은 한다 !**


다음은 4000번 학습 후 가장 잘 나온 Agent의 모습이다 (epsilon 0.1 )

<img src='./05_31/dqn_4000.gif' width=300px>


추후로는 해당 모델을 그대로 이용해
1. Ep1  을 4000번 학습한 파라미터로 모든 Epi 를 학습
2. 그냥 모든 Epi 를 학습

두 가지 방법을 비교한다.

## 6.1 일 기준 PATCH ✒️
### 😑 1.   
- 에이전트의 시야를 기준으로 상태를 잡아주었다.   
- 에이전트의 시야 범위는 에이전트를 중심으로 7\*7, 9\*9 두 가지를 테스트했다.   
![image](https://user-images.githubusercontent.com/96903347/171789627-6d677829-83e6-425e-922d-b346e87e94dd.png)   

- 레이어도 이에 맞춰서 conv2D 3개로 수정.
- 이 상태로 train 에피소드를 학습시킨 결과, 성능은 그렇게 좋지 못했다.
- 시야에 들어오는 아이템을 먹으러 가는 것은 잘했지만, 시야에 아이템이 없을 때의 문제 때문에 최단 경로로 가는 것이 어려움.

### 😮 2.
- 아이템 개수를 기준으로 train 에피소드들을 나눈 csv파일을 생성.
- 아이템 개수 1개부터 차근차근 훈련을 시켜보자!
- 에이전트 시야를 잡아준 기준으로 훈련을 시켜보았음.
- 7\*7은 아이템 1개부터 결과가 그닥 좋지 않았음. 9\*9는 어느 정도 훈련이 잘 되는 것을 확인.

- 9\*9를 아이템 3개까지 훈련시키고, 저장되는 gif 파일을 확인해 봤을 때,   
  위에서 확인했던 에이전트 시야의 문제점 때문에 우리가 원하는 최단거리에 대한 결과는 얻기 어렵다고 판단.
- 하지만 아이템 개수에 따라 나눠서, 개수를 하나씩 늘려가며 훈련하는 방법이 효과가 있다는 것은 확인!

### 3. 🌏 환경
- 에이전트가 아이템을 다 먹었을 때, 출발지 (9,4)가 아이템으로 변경되도록 추가.

## 6.2 일 기준 PATCH ✒️

- 그리드 전체를 상태로 잡고 (기존의 방식), 아이템 개수를 나눠서 훈련을 시도.

## 6.04 일 기준 PATCH ✒️

### 🌏 환경

1. 보상 : move : -0.1 obstacle : -1 items : 10 goal : 20
2. 에이전트 : 100 아이템 : 200 길 : 1 장애물 : 0
3. 먹어야 할 모든 아이템 표시
4. 아이템을 먹으면 장애물로 변환
5. 모든 아이템을 먹으면 목적지에 아이템 표시
6. item 1개부터 다수까지 순차적 학습

### 😗 에이전트

1. hyper parameter
- [Agent.ipynb](https://github.com/youngchurl/Aiffelton_Agilesoda_Urein_Project/tree/main/DQN/06_04) 참조
2. 레이어
- 3 convolution + 2 FC 

test 데이터 기준 99%의 성공률을 가졌다.
결과적으로 가장 높은 정확도.

A* 최단거리 알고리즘과 비교했을 때, 
1226개 평균 최단 액션은 39 이며, 우리의 모델은 평균 41.7 을 달성했다.

<img src='./06_04/agent.gif' width=300px>

우리는 다음의 요소가 가장 큰 영향을 끼쳤다고 판단했다.
1. CNN 
2. 환경의 단순화
3. train 데이터의 학습 순서.  (난이도 순)

```python

```
