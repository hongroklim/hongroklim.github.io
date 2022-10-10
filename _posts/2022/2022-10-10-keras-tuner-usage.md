---
date: 2022-10-10 23:00:00 +0900
title: Keras Tuner 사용법
excerpt: 이것만 알아도 HP 튜닝은 어렵지 않다
categories: dev
---

## 직접 튜닝하기

작년에 '인공지능및응용' 과목을 통해 태어나서 처음으로 딥러닝을 배워봤다.
IC-PBL 수업이라서 이론적인 내용은 크게 중요하지 않았기 때문에 딥러닝을
배웠다고 말하기 민망할 정도로 공부했다. 대신에 딥러닝 관련 프로젝트는 열심히
준비했다 ([링크](https://2021hyt6-techblog.github.io/projects-blog/ai/)).

그 때 프로젝트에서 만들어 본 것은 가장 기초적인 NLP 관련 Classification
모델이었는데, 만들었던 레이어 중에서 `WORD_COUNT`와 `VOCAB_COUNT` 관련 변수를
사용하는 녀석이 있었다. 그냥 예제파일에 나온 숫자 그대로 사용하기보다 저
변수들의 최적값을 한번 찾아보았다.

```python
ret_stats = []

# 데이터셋을 샘플링하고 Vocab Length를 고정
sample_length = 100000
vocab_length = 700

# Word Count 찾기
for length in range(10, 100+1, 5):
    # 모델을 학습시키고
    stats = model_training(df_data.sample(sample_length), vocab_length, length, 5)
    
    # 해당 조합에 대한 최적의 loss, accuracy를 저장한다
    stats['param'] = (sample_length, vocab_length, length)
    stats['min_loss'] = get_peak_with_index(min, stats['val_loss'])
    stats['max_acc'] = get_peak_with_index(max, stats['val_accuracy'])
    ret_stats.append(stats)
```

샘플링 데이터에 epochs는 50 정도 잡고 변수를 조합하면서 모델을
학습시켜보았다. 그리고 이를 그래프로 시각화한 다음에 유의미한 수준의 값을
어림잡아 보았다. 

<figure>
  <img src="https://i.imgur.com/HH62YYf.png" alt="manual_analysis">
  <figcaption>그래프 그려서 눈대중으로 때려 맞춘 경우 / <a href="https://2021hyt6-techblog.github.io/projects-blog/ai/">출처 : HYU Team 6</a></figcaption>
</figure>

시간이 지나고 나서 그 때 내가 했던 일들을 바로 'Hyperparameter Tuning (이하 HP)'
이라고 부른다는 것을 알게 되었다. 그리고 한땀 한땀 코드를 만들 필요 없이 이미
Keras에서 관련 기능을 제공하고 있었다.

## 하이퍼파라미터 튜닝?

하이퍼파라미터튜닝 없이도 딥러닝 모델을 돌리는 데 전혀 문제없다. 이와 비슷한
맥락으로 머신러닝과 신경망 관련 수학적 지식이 없어도 Tensorflow와 Keras는
사용할 수 있다. 단지 딥러닝 모델을 사용하고 싶다면 구글링해서 나오는 코드를 잘
복붙하기만 하면 충분하다.

더 나은 딥러닝 모델을 만들고 싶다면 이론적인 이해가 필요하다. '어떤 Optimizer로
바꿔야하는가', '어떻게 Learning Rate를 바꿔야하는가'와 같은 물음에는 지금 내가
만든 모델에 대한 수학적인 접근이 필요하다. 코드 뒤에 숨겨진 관련된 이론들을
꿰뚫어 볼 수 있어야 내가 만든 모델을 자유롭게 다룰 수 있을 것이다.

추가적으로 Simulation 기법을 적용해야 한다. 수학적 모델링은 무수히 많거니와
불확실하기까지 한 변수들을 모두 담아내기에 부족하다. 설정 가능한 변수들을
요리조리 바꿔보는 Simulation 기법을 통해 최선의 변수값들을 찾아보고자 한다.

## 문제 상황

### Data Preparation

Linear Regression 문제를 딥러닝으로 해결해보고자 한다. 우선 다음과 같이 샘플
데이터를 준비해준다.

```python
# Simple linear regression
x = np.linspace(start=0, stop=100, num=10000)
delta = np.random.uniform(low=-10, high=10, size=x.size)
y = x + delta

# Zip
df_xy = pd.DataFrame(np.column_stack((x, y)), columns=["x", "y"])
df_xy = df_xy.sample(frac=1).reset_index(drop=True)
```

`x`에는 0 부터 1 사이에 있는 10,000개의 숫자를 담고 있다. `delta`에는 -10 ~ 10
사이의 오차를 만들었고, 이 두 배열을 합쳐 `y`를 만들었다.

### DL Modeling

그 다음에는 구글을 뒤지면서 이와 관련된 모델에 대한 소스코드를 찾는다.

```python
model = keras.Sequential()
model.add(keras.Input(shape=(1, ), name="x", dtype="float64"))

# Normalization
normalizer = Normalization()
feature_ds = train_ds.map(lambda x, y: tf.expand_dims(x["x"], -1))
normalizer.adapt(feature_ds)
model.add(normalizer)

# Hidden layer
model.add(layers.Dense(32, activation="relu"))

model.add(layers.Dense(1))

# Optimizer
model.compile(optimizer=keras.optimizers.Adam(learning_rate=0.001),
              loss=keras.losses.MeanSquaredError(),
              metrics=["mae"])
```

딥러닝 모델을 만든다는 것은 굉장히 주관적인 일이다. 구글링을 하다보면 같은
일을 하고 있음에도 가지각색의 형태와 숫자를 가지는 모델들을 마주하게 된다.
돌아가는 모델 아무거나 가져다써도 상관은 없다지만, 과연 어떤 모델을 써야 잘
썼다고 소문이 날지 생각해보게 된다.

위의 모델에서는 다음과 같은 질문을 떠올릴 수 있다.

1. Input에 대한 Normalization이 꼭 필요한가?
1. Hidden Layer의 32라는 unit 개수는 적당한가?
1. Optimizer는 Adam이 SGD보다 더 나은가?
1. Learning Rate는 0.001이 적당한가?

생각할거리도 많고, 어떤 데이터셋으로 학습할 지도 모르는 상황에서 수학적으로
이러한 문제들을 해결하기란 쉽지 않다. 이 때 사용해볼만한 라이브러리는 Keras
Tuner이다.

## Keras Tuner

무려 Keras 공식 문서에 올라와있는 [Keras Tuner](https://keras.io/keras_tuner/).
여러가지 경우의 수를 따져가면서 가장 최적의 모델을 찾는 것을 도와준다.
사용법도 어렵지 않다. 이미 만들어둔 코드를 바탕으로 Keras Tuner를 활용해보자.

### build_model

기존에 모델을 생성하는 코드를 `keras_tuner.HyperParameters`를 Argument로
받고 `Model` 인스턴스를 Return하는 `build_model` 함수로 감싸줘야 한다. 그리고
여러 경우의 수를 적용해보고 싶은 곳들을 Argument를 활용해 바꿔준다.

```python
def build_model(hp):
    model = keras.Sequential()
    model.add(keras.Input(shape=(1, ), name="x", dtype="float64"))
    
    # Hyperparameter 1.
    is_normalize = hp.Boolean("normalize")
    if is_normalize:
        normalizer = Normalization()
        feature_ds = train_ds.map(lambda x, y: tf.expand_dims(x["x"], -1))
        normalizer.adapt(feature_ds)
        model.add(normalizer)
    
    # Hyperparameter 2.
    neurons = hp.Int("neurons", min_value=4, max_value=32, step=4)
    model.add(layers.Dense(neurons, activation="relu"))
    
    model.add(layers.Dense(1))
    
    # Hyperparameter 3.
    learning_rate = hp.Float("learning_rate", min_value=1e-4, max_value=2e-1, sampling="log")
    
    # Hyperparameter 4.
    optimizer_name = hp.Choice("optimizer", ["adam", "sgd"])
    if optimizer_name == "adam":
        optimizer = keras.optimizers.Adam(learning_rate=learning_rate)
    elif optimizer_name == "sgd":
        optimizer = keras.optimizers.SGD(learning_rate=learning_rate * 10)
    
    model.compile(optimizer=optimizer,
                  loss=keras.losses.MeanSquaredError(),
                  metrics=["mae"])
    
    return model
```

위의 `build_model` 안에서는 다음과 같은 하이퍼파라미터를 설정하였다.

1. `hp.Boolean("normalize")`
1. `hp.Int("neurons", min_value=4, max_value=32, step=4)`
1. `hp.Float("learning_rate", min_value=1e-4, max_value=2e-1, sampling="log")`
1. `hp.Choice("optimizer", ["adam", "sgd"])`

`hp.<메소드>` 형식으로 바꿔보고자 하는 값들을 넘겨주는데, Signature만 봐도 저
코드가 어떤 값을 Return할 지 대충 감이 온다.

### RandomSearch

이제 이 `build_model`을 가지고 Simulation을 실행해주는 `keras_tuner.Tuner`
인스턴스를 만들어준다.

```python
import keras_tuner

tuner = keras_tuner.RandomSearch(
    hypermodel=build_model,
    objective="val_loss",
    max_trials=10)
```

`RandomSearch` 알고리즘 이외에도 `BayesianOptimzation`과 `Hyperband`를 이용할
수 있다.

`tuner.search_space_summary()`를 통해 하이퍼파라미터들이 잘 잡혔는지
확인해보자.

```
// ...
normalize (Boolean)
{'default': False, 'conditions': []}
neurons (Int)
{'default': None, 'conditions': [], 'min_value': 4, 'max_value': 32, 'step': 4, 'sampling': None}
learning_rate (Float)
{'default': 0.0001, 'conditions': [], 'min_value': 0.0001, 'max_value': 0.2, 'step': None, 'sampling': 'log'}
optimizer (Choice)
{'default': 'adam', 'conditions': [], 'values': ['adam', 'sgd'], 'ordered': False}
```

`build_model`을 제대로 읽어들였음을 확인할 수 있다.

### search

평소같았으면 `model.fit(...)`을 실행시켰겠지만, 이번에는 `search(...)`
메소드를 실행하면된다.

```python
early_stop = keras.callbacks.EarlyStopping(monitor='val_loss', patience=10)
tuner.search(train_ds, validation_data=val_ds, epochs=50, callbacks=[early_stop])
```

파라미터는 `model.fit`에 쓰던 것과 완전히 같다. 실행하게 되면 다음과 같이
실시간으로 Simulation 상황을 지켜볼 수 있다.

```
Trial 6 Complete [00h 00m 18s]
val_loss: 33.460365295410156

Best val_loss So Far: 33.374664306640625
Total elapsed time: 00h 01m 35s

Search: Running Trial #7

Value             |Best Value So Far |Hyperparameter
False             |False             |normalize
24                |16                |neurons
0.0019608         |0.04053           |learning_rate
adam              |adam              |optimizer
// ...
```

### results_summary

Simulation이 끝난 후 `tuner.results_summary()`로 세부 Trial 기록을 확인할 수
있다.

```
Results summary
Results in ./untitled_project
Showing 10 best trials
<keras_tuner.engine.objective.Objective object at 0x14e3f2980>
Trial summary
Hyperparameters:
normalize: False
neurons: 16
learning_rate: 0.04053025640180924
optimizer: adam
Score: 33.374664306640625
Trial summary
Hyperparameters:
normalize: False
neurons: 24
learning_rate: 0.0019608369359383425
optimizer: adam
Score: 33.38059616088867
// ...
```

### get_best_hyperparameters

tuner가 찾은 최적의 값들을 확인하는 방법은 다음과 같다.

```python
best_hp = tuner.get_best_hyperparameters()[0]
best_hp.get("learning_rate") # 0.04053025640180924
```

### get_best_models

위의 `best_hp`로 다시 모델을 만들 수 있지만, best trial의 best epoch에서
checkpoint로 저장된 모델을 불러올 수 있다.

```python
best_model = tuner.get_best_models()[0]
best_model.evaluate(val_ds)
```

## 결론

지금까지 Keras Tuner 사용법을 알아보았다. 그냥 딥러닝 모델을 만드는 코드에
`build_model`을 감싸주는 일 밖에 하지 않았다. 이렇게 간편하고도 강력한
라이브러리가 있었다니, 작년에 일일이 그래프 그려보면서 최적의 HP를 찾던
나날들이 주마등처럼 지나갔다.

아무리 Simulation을 사용한다고 하지만 이론적인 지식은 여전히 필요함을 느꼈다.
구글링해서 복붙한 코드일지라도 어느 부분을 바꿔볼 지에 대한 결정은 여전히 나
자신에게 달려있다. 그리고 Learning Rate를 1 ~ 1,000 사이로 잡았다면
Simulation을 하는 의미가 없을 것이다.
