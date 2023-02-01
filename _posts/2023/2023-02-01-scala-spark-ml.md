---
date: 2023-02-01 10:00:00 +0900
title: 머신러닝이 분산컴퓨팅을 만난다면
excerpt: Scala + Spark + ML 이 조합 찬성이오
categories: bigdata
---

## 아무 것도 모르던 시절

2021년 '인공지능및응용'을 수강하며 [NLP 관련 프로젝트](https://2021hyt6-techblog.github.io/projects-blog/ai/)
를 진행했다. Amazon 리뷰와 별점을 가지고 문장의 긍정/부정을 가리는 모델을
개발하고 있었다. 36만건의 학습 데이터셋을 다루었는데 이게 보통 일이 아니었다.
Python에서 [pandas](https://pandas.pydata.org)와 [spaCy](https://spacy.io)
라이브러리 등을 써서 전처리를 했는데 (이미지도 아닌 것이) 엄청난 시간이
소요되었다.

당시 나는 이 문제를 해결하기 위해 OS 시간에 배운 [Multi-processing](https://en.wikipedia.org/wiki/Multiprocessing)을 써보기로
했다. Disk I/O 문제가 걸리긴 했지만 메모리 사용량을 최대한으로 끌어올리고
싶었다. pandas로 csv를 읽을 때 skiprows 옵션이 있길래 여러 범위로 나누어
데이터를 읽어보기로 했다.

```python
from multiprocessing import Pool
import pandas as pd

TOTAL_ROWS = 3000000
BATCH_SIZE =   10000

def preprocess_data(seq, skips, rows):
    data = pd.read_csv('amazon_review_full_csv/train.csv',
                header=None, skiprows=skips, nrows=rows)
    # ...


if __name__ == '__main__':
    with Pool(10) as p:
        p.starmap(preprocess_data,
                  [(id+1, id*BATCH_SIZE, BATCH_SIZE)
                   for id in range(TOTAL_ROWS//BATCH_SIZE)])
```

효과는 엄청났다. 정확하게 비교해보진 않았지만 Single-process 대비 20배 이상
빨랐던 것 같다. '수업시간에 배운 내용을 이렇게 써먹을 줄이야' 하면서 혼자
좋아했던 기억이 난다. 2023년 [SparkML](https://spark.apache.org/docs/latest/ml-guide.html)
을 공부하던 나는 분산컴퓨팅 기술이 머신러닝에 아주 적극적으로 활용되고 있다는
것을 알게 되었다. 그러면서 이 프로젝트가 떠올랐고, 데이터 전처리 과정을
개선해보고 싶은 마음이 생겼다.

## Scala 좀 써볼까나

Python으로 개발한 프로젝트를 이번엔 [Scala](https://www.scala-lang.org)
를 통해 개발하려고 한다. pyspark로 Python에서 Spark를 쓸 수 있었지만, 같은
언어로 개발하게 되면 단순하게 복붙만 할까봐 일부러 Scala로 개발하며 데이터
전처리 과정을 온전히 생각해보고 싶었다.

### Kernel 추가

[almond](https://almond.sh) 를 통해 [Jupyter Notebook](https://jupyter.org)
에서 개발해보려 한다. Scala + Spark 개발은 [zeppelin](https://zeppelin.apache.org)
으로 하고 있었다. 데이터 시각화는 아주 훌륭했지만, 그 뭐랄까 타이핑이 손에
감기지 않는다고 해야 하나... Notebook 만큼의 개발 생산성이 나오질 않았다.
(도구 탓을 하는 걸 보니 나는 아직 장인이 아닌가보다) [Coursier](https://get-coursier.io)
만 설치되어 있다면 Scala Kernel을 추가하는 건 일도 아니었다.

```sh
coursier launch --fork almond:0.13.2 --scala 2.12 -- --install
# 참고: https://almond.sh/docs/quick-start-install
```

저 버전도 참 중요한데, [이 설명](https://almond.sh/docs/usage-spark)에 따르면
최신 버전의 almond는 Spark 2.4.X만 지원하는 데, 이는 Scala 2.12.X만 지원하니
저 Scala 버전을 무조건 선택해야 한다.

### spark 잡기

이후 Jupyter Notebook을 열어 다음과 같이 설정하면 된다.

```scala
// Dependencies
import $ivy.`org.apache.spark::spark-sql:2.4.8`
import $ivy.`org.apache.spark::spark-mllib:2.4.8`

// Logging
import org.apache.log4j.{Level, Logger}
import org.apache.spark.sql._

Logger.getLogger("org").setLevel(Level.OFF)

// Init
val spark = {
  NotebookSparkSession.builder()
    .master("local[*]")
    .getOrCreate()
}

def sc = spark.sparkContext
```

개발 환경설정은 끝났다. 이제 익숙한 UI에서 개발을 이어나가자.

## csv to parquet

먼저 csv 형식의 데이터를 [parquet](https://parquet.apache.org)으로 변환하고
나서 시작한다. pandas로 개발할 때는 모든 데이터를 한 번에 읽어 작업하므로 Disk
I/O는 크게 상관 없었다. 하지만 Spark 에서는 내가 필요한 특정 부분만 읽어올 수
있으므로 이에 걸맞게 좀 더 효율적인 저장 포맷이 필요하다.
([참고자료](https://blog-tech.tadatada.com/2018-05-23-parquet-and-spark))

```scala
import org.apache.spark.sql.types.{StructType, StructField, IntegerType, StringType}

// Define schema
val schema = new StructType(Array(
    StructField("score", IntegerType, true),
    StructField("title", StringType, true),
    StructField("body", StringType, true),
))

// Load
val trainDf = spark.read.options(Map("header" -> "false")).schema(schema)
            .csv("amazon_review_full_csv/train.csv")

val testDf = spark.read.options(Map("header" -> "false")).schema(schema)
            .csv("amazon_review_full_csv/test.csv")

// Union
val df = trainDf.union(testDf)

// Save
df.write.parquet("review")

// Read
val df = spark.read.parquet("review")
```

Schema를 정의하고 train, test csv 파일을 각각 읽어들여 하나의 parquet 형식의
파일로 저장했다. 1.64GB 파일이 1.01GB로 61%나 압축되었다. 별일 안 했는데도 효율성
개선을 실감하다니 벌써부터 기분이 좋아진다.

## Stages

데이터 파이프라인에는 총 4단계의 stage가 있다. 다른 라이브러리의 도움 없이
다음의 객체를 활용해서 Spark 자체적으로도 할 수 있다.

1. 유효하지 않은 데이터 거르고 열 합치기: `SQLTransformer`
1. 문장을 단어 단위로 분해하기: `RegexTokenizer`
1. 불용어 제거하기: `StopWordsRemover`
1. 단어들을 벡터로 표현하기: `CountVectorizer`

이미 다 구현되어 있는 지라 사용법도 간단하다.

### SQLTransformer

데이터 파이프라인에서 가장 먼저 유효하지 않은 데이터들을 걸러주어야 한다.
그리고 후속 단계에서의 원활한 핸들링을 위한 간단한 `map` 함수도 적용해야 한다.
이러한 기본적인 작업들은 `SQLTransformer` 에서는 `SELECT`와 `WHERE` clause를
통해 구현할 수 있다.

```scala
import org.apache.spark.ml.feature.SQLTransformer

val sqlTrans = new SQLTransformer().setStatement("""
    SELECT CASE WHEN score in (1, 2) THEN 0
                WHEN score = 3       THEN 1
                WHEN score in (4, 5) THEN 2
           END as label
         , CONCAT(title, ' ', body) AS text
      FROM __THIS__
     WHERE score BETWEEN 1 AND 5
       AND CONCAT(title, ' ', body) IS NOT NULL
""")
```

이처럼 간단한 작업을 SQL Syntax를 통해 구현하니 파이프라인을 들여다보기가
편해졌다. 복잡한 `map` 함수를 적용하고 싶다면 SparkSQL의 `udf`를 이용하면 된다.

### RegexTokenizer

그 다음으로 문장을 단어 단위로 분해하는 일이다. 이 때는 숫자나 특수문자, 공백
등도 걸러내어 알파벳만으로 이루어진 연속된 문자열을 뽑아내야 한다.
`RegexTokenizer`에서 정규식을 통해 작업해보자.

```scala
import org.apache.spark.ml.feature.RegexTokenizer

val regexTokenizer = new RegexTokenizer().setInputCol("text").setOutputCol("token").setPattern("\\W")
```

### StopWordsRemover

문장 속에는 'I', 'the', 'a'와 같이 자주 사용되지만 크게 의미가 없는 단어들도
있다. 이들을 `StopWordsRemover`를 통해 걸러낼 수 있다.

```scala
import org.apache.spark.ml.feature.StopWordsRemover

val remover = new StopWordsRemover().setInputCol("token").setOutputCol("tokenRemoved")
```

### CountVectorizer

모든 단어를 모델 학습에 사용할 수는 없다. 자주 쓰이는 일부 단어만을 골라내어
학습을 진행시켜야 한다. 출현 빈도를 구하고 이를 인코딩하는 과정을 떠올리니
머리가 복잡해지지만 `CountVectorizer`를 쓰면 한 번에 해결된다.

```scala
import org.apache.spark.ml.feature.CountVectorizer

val countVectorizer = new CountVectorizer()
  .setInputCol("tokenRemoved")
  .setOutputCol("features")
  .setVocabSize(100)
  .setMinDF(2)
```

모든 Stage를 구현했으니. 이를 합치기만 하면 된다.

## Pipeline

### Overview

원본 데이터는 다음과 같은 형식이었다.

| score | title | body |
| ----- | ----- | ---- |
| 2     | Good for what it tries to do but... | The book is a very good exploration (생략)|

`score` 열은 1점부터 5점까지 점수가 매겨져 있는데, 이를 (1, 2) / (3) / (4, 5)
총 세 개의 카테고리로 나누고 `label`이라는 이름을 붙였다.

`title`과 `body` 열은 모델에서 `features`로 쓰인다. 이에 대한 데이터
파이프라인을 요약하면 다음과 같다.

| # | Stage | Role | Column | Example |
| - | ----- | ---- | ------ | ------- |
| 0 |  |  | title \| body | \"Good for what it…\" \| \"The book...\" |
| 1 | SQLTransformer | filtering 및 projection | text | \"Good for what it...\" |
| 2 | RegexTokenizer | tokenize | token | [good, for, what, ...] |
| 3 | StopWordsRemover | remove stopwords | tokenRemoved | [good, tries, book, ...] |
| 4 | CountVectorizer | convert to vector | features | (100,[0,2,36,55,56,75],[1,2,1,1,1,1]) |

두 개의 String이 담긴 열에서 시작해 4개의 Stage를 거쳐 마지막에는 정수 벡터
하나로 변환되었다.

### Integration

지금까지 만든 Stage들을 하나의 파이프라인에 모으는 법은 다음과 같다.

```scala
import org.apache.spark.ml.Pipeline

val pipeline = new Pipeline()
    .setStages(Array(sqlTrans, regexTokenizer, remover, countVectorizer))
```

## ML Application

간단히 RandomForest 모델을 만들어보자. 그리고 나서 파이프라인 마지막에
붙여주면 된다.

```scala
import org.apache.spark.ml.classification.RandomForestClassifier

val rf = new RandomForestClassifier()
  .setFeaturesCol("features")
  .setLabelCol("label")
  .setNumTrees(30)

val pipeline = new Pipeline()
    .setStages(Array(sqlTrans, regexTokenizer, remover, countVectorizer, rf))
```

### Fit

train, test 데이터로 다시 나누고 `fit`과 `transform`을 진행하면 된다.

```scala
val Array(trainingData, testData) = df.randomSplit(Array(0.7, 0.3))

val model = pipeline.fit(trainingData)
val predictions = model.transform(testData)
```

`predictions`에는 `df`에 추가적으로 `rawPrediction`, `probability` 그리고
`prediction` column이 생긴다.

### Evaluation

모델의 정확도를 알아보는 것도 Spark 내에서 가능하다. 총 3개의 label이 있으니
`MulticlassClassificationEvaluator`를 사용한다.

```scala
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator

val evaluator = new MulticlassClassificationEvaluator()
  .setLabelCol("label")
  .setPredictionCol("prediction")
  .setMetricName("accuracy")
```

그리고 `evaluator.evaluate(predictions)`를 실행하게 된다면 정확도가 나온다.
여기선 0.54 정도의 값이 나왔다. 이것 말고도 Hyperparameter Tuning을 할 수
있다고 하는데 생략하겠다.

<figure>
  <img src="https://i.imgur.com/RM3Mbwp.jpg"
       alt="but_it_was_fast">
  <figcaption>
    정확도가 54%밖에 안 된다고? 하지만 빨랐죠
    &emsp;/&emsp;
    <a href="https://www.inven.co.kr/board/lostark/4811/5037482">
      출처: 인벤
    </a>
  </figcaption>
</figure>

## 결론: 빠르다 빨라 현대사회

데이터 전처리 부터 모델 평가까지 이렇게 빨리 할 줄 몰랐다. 만약에 클라우드에서
여러 인스턴스를 띄워놓았으면 거의 실시간으로 했을 지도 모른다. 2021년에
프로젝트를 진행할 때는 모든 단계가 오래걸리다 보니 Tving 구독하고 ['유미의
세포들 시즌1'](https://www.cjenm.com/ko/featured-contents/유미의-세포들/)
을 결과값이 나올 때 까지 틈틈이 봤다. 하지만 이번에 Spark로 개발했을 때는
한눈을 팔 틈도 없었다. [Spark UI](https://spark.apache.org/docs/latest/web-ui.html)
에서 각 Thread 별 진척을 살펴보다보면 어느 새 모든 Job이 끝나있었다. 앞으로
Spark를 애용할 것 같은 예감이 든다.
