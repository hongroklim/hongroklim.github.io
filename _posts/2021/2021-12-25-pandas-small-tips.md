---
date: 2021-12-25 14:00:00 +0900
title: Pandas를 써본 티를 내는 방법
excerpt: 어느날 Jupyter Notebook을 열었을 때 기억했으면 하는 습관들
header:
    overlay_image: https://user-images.githubusercontent.com/59322692/147387245-c06d4f6b-dd5f-471e-be5d-fa2a3caa3266.png
categories: bigdata
---

지금까지 Python IDE만 쓰다가 이번 학기를 통해 Jupyter Notebook을 처음 써보게 되었다. 너무 편했다.
앞으로 간단히 데이터 분석할 일이 있을 때 자연스럽게 Pandas를 쓸 것 같으니 소소한 습관들을 정리하고자 한다.

## Notebook을 쓰기 위한 준비

### Python 설정

일단 설치부터 해야 한다. 검색해보면 Jupyter Lab과 Notebook 두 개가 나온다.
Jupyter Lab은 Notebook의 확장판이라고 생각하면 되니 Notebook만 설치해도 된다.

```sh
# .venv/에 가상환경부터 설정한다
$ python -m venv .venv
$ source .venv/bin/activate

# Notebook을 설치한다
$ pip install notebook

# 실행하면 된다
$ jupyter notebook
```

Notebook을 열고 kernel을 연 다음에 필요한 package를 설치한다.

```python
# dependencies 설치
!pip install -r requirements.txt

# 특정 package 설치
%pip install seaborn
```

마지막으로 경로설정까지 하면 준비완료.

```python
import os

os.chdir('.')
print(f"Current directory is {os.getcwd()}")
```

### Pandas 설정

본격적으로 Pandas를 가지고 놀기 전에 실행할 명령어들이다.

```python
import pandas as pd

# Grid 출력 시 ... 으로 나오지 않고 모두 보여주기
pd.options.display.max_columns = None

# float Datatype 설정
pd.set_option('precision', 4)
pd.options.display.float_format = '{:.2f}'.format
```

## csv 불러오고 저장하기

데이터셋을 읽고 저장하는 건 간단하다.

```python
import pandas as pd

# (1) 불러오기
df_data = pd.read_csv(LOAD_CSV_PATHNAME, header=None, skiprows=0, nrows=100)
df_data.head(n=10)

# (2) 여러 CSV 이어서 불러오기
df_order = pd.concat([pd.read_csv(FST_ORDERBOOK_PATHNAME),
                      pd.read_csv(SND_ORDERBOOK_PATHNAME) ],
                     ignore_index=True)

# 저장하기
df_data.to_csv(SAVE_CSV_PATHNAME, index=False,
               columns=['timestamp', 'price', 'midprice', 'bfeature', 'alpha', 'side'])
```

## Result를 잘 보여주기

### Print

변수를 Code Cell에 입력만 해도 어떤 값인지는 볼 수 있다. 하지만 print 함수를 썼을 때와 출력 결과와 다르므로 그냥 print 함수 붙여서 쓰는게 더 낫다.

### Logging

print 함수와 다른건 없지만, logging을 사용한다면 Console 출력 내용을 다른 print 결과와 구분할 수 있다. 그리고 자연스럽게 timestamp도 남길 수 있다는 점도 편리하다.

```python
import logging

# Logging Configuration
logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter('%(asctime)s [%(levelname)s] %(message)s'))
logger.addHandler(stream_handler)

# Logging Example
logging.debug(f"Start Drawing …")
```

## Dataframe Column 다루기

별거 아닌거 가지고 매번 google에 검색하지 말자. 한번에 정리해본다.

```python
# 새로 만들기
df_order['timestamp'] = pd.to_datetime(df_order['timestamp'], format='%Y-%m-%d %H:%M:%S.%f')

# 지우기
df_trade.drop(['quantity', 'amount', 'fee'], axis=1, inplace=True)

# 이름바꾸기
df_trade.rename(columns={'0':'buy', '1':'sell'}, inplace=True)

# Type Cast
df_trade.columns = df_trade.columns.map(str)
```

Aggregate나 Pivot 후에 column이 Multi-index로 바뀌게 되는데, 일반적인 column과는 다르게
다뤄줘야 한다.

```python
# column 이름 없애기
df_trade.columns.name = None

# Multi-index Access하기
df_trade[('price', 'min', 1)] // 또는 df_order['quantity', 'mean', 0]

# 1D로 만들기
df_x.columns = [f'{x[0]}_{x[1]}_{x[2]}' for x in df_x.columns.to_flat_index()]
```

## SQL같이 쓰는 기능

Pandas는 데이터 분석할 때 사랑받는 라이브러리답게 SQL에서 쓸 수 있는 function들을 다 갖추고 있다.

### GROUP BY

기본적으로 다양한 Aggregate function을 쓸 수 있다.

```python
# COUNT
df_data.groupby(['timestamp', 'type']).size().to_frame(name='count')

# AVG, MAX, MIN
df_data.groupby(['timestamp', 'type']).agg({'price': ['mean', 'min'], 'quantity': ['mean']})
```

### PIVOT

Pivoting을 하고 난 후에는 index parameter들이 index가 되는데, `reset_index`로 풀어주면 된다.

```python
pd.pivot_table(data=df_data, index='timestamp', columns='type')

pd.pivot_table(data=df_data, index=['timestamp'], columns='side',
	       aggfunc=len, fill_value=0)

df_data.reset_index(inplace=True)
```

Unpivot은 `melt`로 한다. `var_name`은 기존 column 이름, `value_name`은 cell data에 대한 새로운 column 명을 넣으면 된다.

```python
df_trade = df_trade.melt(id_vars=['timestamp'], var_name='div', value_name='counts')
```

### MERGE

Column들을 이어 붙이는건 간단하다. Pandas에는 신기한 `merge_asof` 기능도 지원한다.

```python
# (1) Concatenate
pd.concat([df_order, df_trade], axis=1)

# (2) Column 이름 붙여서 Concatenate 
dic_dfs = {'single': df_x, 'window': df_win, 'diff': df_diff}
df_x = pd.concat(dic_dfs.values(), keys=dic_dfs.keys(), axis=1)

# Join
df_y.value_counts().to_frame('count').join(df_y.value_counts(normalize=True).to_frame('%'))

# Sort Merge
pd.merge_asof(df_trade, df_order,
              on='timestamp', direction='backward',
              tolerance=pd.Timedelta("1min"))
```

## Multiprocessing

오래 걸리는 작업을 처리할 때는 학부생 때 OS를 배워본 사람답게 Multiprocess도 써먹을 줄 알아야 한다.

```python
def preprocess_data(seq, skips, rows):
    # Skip Condition
    if os.path.isfile(f'amazon_review_full_csv/train/train_{str(seq).zfill(3)}.csv'):
        print(f'{datetime.datetime.now()} - {str(seq).zfill(3)} is skipped')
        return

    # Load Data
    data = pd.read_csv('amazon_review_full_csv/train.csv', header=None, skiprows=skips, nrows=rows)
    
    # ...
    
    # Save
    data.to_csv(f'amazon_review_full_csv/train/train_{str(seq).zfill(3)}.csv', index=False)
    print(f'{datetime.datetime.now()} - {str(seq).zfill(3)} is completed')
    
    
# Main
from multiprocessing import Pool

TOTAL_ROWS = 3000000
BATCH_SIZE =   10000

if __name__ == '__main__':
    print(f'{datetime.datetime.now()} - start preprocessing. {TOTAL_ROWS//BATCH_SIZE} times.')
    with Pool(10) as p:
        p.starmap(preprocess_data,
                  [(id+1, id*BATCH_SIZE, BATCH_SIZE)
                    for id in range(TOTAL_ROWS//BATCH_SIZE)])
```
