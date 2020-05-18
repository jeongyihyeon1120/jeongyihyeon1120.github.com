---
layout: post
title:  "Installing tensorflow"
date:   2020-05-18 10:35:00 +0700
categories: [python, tensorflow]
---

#### TensorFlow 설치

**방법1.** Python3버전 설치: ["https://www.python.org/downloads/windows/"]
`주의 사항: ` `2020.05.18 기준 3.7.x 버전 아래 설치`

cmd창 킨 후 아래 명령어 입력

```
pip3 install tensorflow
```
이후 

```
$ pip3 list
```

**방법2.** Anaconda3 설치: ["https://www.anaconda.com/products/individual#download-section"]
`주의사항: `기존 파이썬 파일과 충돌이 될 수 있음 (가능하면 파이썬 파일 지우고 설치)

**Anaconda Prompt로 TensorFlow 실행하기** 


**1.** pip 업그레이드

```
python -m pip install --upgrade pip
```

**2.** conda 환경 구축

```
conda create -n tensorflow python=3.7
```

`주의사항: `버전에 맞게 설치
ex) 파이썬 버전이 만약 3.6버전이면 (버전에 맞게 설치하자)

```
conda create -n tensorflow python=3.6
```

-n 뒤에는 tensorflow를 돌릴 가상환경의 이름을 적는다.
ex)

```
conda create -n yhTensorflow python=3.7
```

**3.** Tensorflow 설치

```
activate 가상환경이름
```
→ (가상환경이름)으로 바뀐다

### tensorflow 다운

```
pip install tensorflow
```

### 파이썬 터미널

```
python
```

### tensorflow 패키지 improt

```
import tensorflow as tf
```

이후 코딩








