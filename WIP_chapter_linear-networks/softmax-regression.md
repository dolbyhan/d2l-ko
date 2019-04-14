# Softmax 회귀

앞 두 절, [*from scratch*](linear-regression-scratch.ipynb) 와 [using Gluon](linear-regression-gluon.ipynb)을 통해서 선형 회귀 모델을 직접 구현해보기도 했고, Gluon을 이용해서 구현해보았습니다. Gluon을 이용하면 파라미터 정의나 초기화, 손실 함수(loss function) 정의, optimizer 구현과 같은 반복된 일을 자동화할 수 있었습니다.

회귀(regression)는 몇 개인지, 얼마인지 등에 대한 답을 구할 때 사용하는 도구로, 예를 들면 집 가격이 얼마인지, 어떤 야구팀이 몇 번 승리를 할 것인지 등을 예측하는데 사용할 수 있는 방법입니다. 다른 예로는, 환자가 몇 일 만에 퇴원할 것인지 예측하는 것도 회귀(regression) 문제입니다. 

하지만, 현실에서는 어떤 카테고리에 해당하는지를 예측하는 문제를 더 많이 접하게 됩니다.

* 메일이 스팸인지 아닌지
* 고객이 구독 서비스에 가입할지 아닐지
* 이미지에 있는 객체가 무엇인지 (원숭이, 강아지, 고양이, 닭 등)
* 고객이 어떤 물건을 구매할 것인지

카테고리별로 값을 할당하거나, 어떤 카테고리에 속할 확률이 얼마나 되는지를 예측하는 것은 분류(classification)라고 부릅니다. 앞 절들에서 살펴본 모델은 확률을 예측하는 문제에 적용하기 어렵습니다.

## 분류 문제들

입력 이미지의 높이와 넓이가 2 픽셀이고, 색은 회색인 이미지를 입력으로 다루는 간단한 문제부터 시작해보겠습니다. 이미지의 4개 픽셀의 값은  $x_1, x_2, x_3, x_4$ 으로 표현하고, 각 이미지의 실제 레이블(label)는 "고양이", "닭", "강아지" 중에 하나로 정의되어 있다고 하겠습니다. (4 픽셀로 구성된 이미지가 3개 동물 중에 어떤 것인지를 구별할 수 있다고 가정합니다.)

이 레이블(label)들을 표현하는데 두가지 방법이 있습니다. 첫번째 방법은  {강아지, 고양이, 닭}을 각각  $y \in \{1, 2, 3\}​$ 으로 정의합니다. 이 방법은 컴퓨터에 정보를 저장하는 좋은 방법이지만, 이 방법은 회귀 문제에 적합합니다. 더구나 이 숫자들의 순서가 분류의 문제에서는 의미가 없습니다. 우리의 간단한 예제에서는 적어도 수학적으로는 고양이가 강아지보다는 닭과 더 비슷하다는 것을 의미할 수도 있게 됩니다. 하지만, 실제 문제들에서 이런 비교가 잘되지 않습니다. 그렇기 때문에, 통계학자들은 원-핫-인코딩(one hot encoding) 을 통해서 표현하는 방법을 만들었습니다.

$$y \in \{(1, 0, 0), (0, 1, 0), (0, 0, 1)\}$$

즉, $y$ 는 3차원 벡터로 (1,0,0)은 고양이를, (0,1,0)은 닭은, (0,0,1)은 강아지를 의미합니다.

## 네트워크 아키텍처

여러 클래스들에 대한 분류를 예측할 때는 카테고리 개수와 같은 수의 출력들이 필요합니다. 이점이 회귀 문제와 가장 다른 점입니다. 4개 특성(feature)들과 3개의 동물 카테고리 출력(output)들이 있으니, 가중치($w$)는 12개의 스칼라들로 구성되고 편향(bias) ($b$)는 3개의 스칼라로 정의됩니다. 각 입력에 대해서 3개의 출력 ($o1, o2, o3$)는 다음과 같이 계산됩니다.
$$
\begin{aligned}
o_1 &= x_1 w_{11} + x_2 w_{21} + x_3 w_{31} + x_4 w_{41} + b_1,\\
o_2 &= x_1 w_{12} + x_2 w_{22} + x_3 w_{32} + x_4 w_{42} + b_2,\\
o_3 &= x_1 w_{13} + x_2 w_{23} + x_3 w_{33} + x_4 w_{43} + b_3.
\end{aligned}
$$

아래 뉴럴 네트워크 다이어그램은 위 연산을 표현하고 있습니다. 선형 회귀처럼, softmax 회귀는 단일층의 뉴럴 네트워크로 구성됩니다. 출력 ($o1, o2, o3$) 는 모든 입력 ($x1, x2, x3, x4$) 값들과 연관되서 계산되기 때문에, softmax 회귀의 출력층은 완전 연결층입니다.

![Softmax 회귀는 단일층 뉴럴 네트워크입니다.](../img/softmaxreg.svg)

## Softmax 연산

위 표기법은 다소 장황해 보입니다. 이를 벡터 표현으로 하면  $\mathbf{o} = \mathbf{W} \mathbf{x} + \mathbf{b}$ 와 같이 쓰기도 간단하고 코딩하기도 간단해집니다. 하지만, 분류 문제는 이산(discrete) 예측 결과가 필요하기 때문에, $i$ 번째 카테고리에 대한 확신 수준(confidence level)을 표현하기 위해서 출력을 $o_i$ 로 표현하는 간단한 방법을 사용합니다. 이렇게 구성하면, 어떤 카테고리에 속하는지를 결과 값들 중에 가장 큰 값의 클래스로 선택하면 되고,  $\operatorname*{argmax}_i o_i$ 로 간단히 계산할 수 있습니다. 예를 들면, 결과 $o1, o2, o3$ 가 각 각 0.1, 10, 0.1 이라면, 예측된 카테고리는 2, 즉 "닭"이 됩니다.

하지만, 출력층의 값을 직접 사용하기에는 두 가지 문제가 있습니다. 첫번째는 출력값의 범위가 불확실해서, 시각적으로 이 값들의 의미를 판단하기 어렵다는 것입니다. 예를 들어, 이전 예에서 결과 10은 주어진 이미지가 "닭" 카테고리에 속할 것이라고 "매우 확신"한다는 것을 의미합니다. 왜냐하면, 다른 두 카테고리들의 값보다 100배 크기 때문입니다. 만약에 $o_1=o_3=10^3$ 이라면, 10이라는 출력값은 이미지가 "닭" 카테고리에 속할 가능성이 매우 낮다는 것의 의미하게 됩니다. 두번째 문제는 실제 레이블(label)은 이산(discrete) 값을 갖기 때문에, 불특정 범위를 갖는 출력값과 레이블 값의 오류를 측정하는 것이 매우 어렵다는 것입니다.

출력값들이 확률값으로 나오도록 해볼 수 있겠지만, 새로운 데이터가 주어졌을 때 확률값이 0 또는 양수이고, 전체 합이 1이 된다는 것을 보장할 수는 없습니다. 이런 이산 값(discrete value) 예측 문제를 다루기 위해서 통계학자들은 (softmax) 로지스틱 회귀(logistic regression)이라는 분류 모델을 만들었습니다. 선형 회귀(linear regression)과는 다르게, softmax 회귀(regression)의 결과는 모든 결과값들의 합이 1이 되도록 하는 비선형성에 영향을 받고, 각 결과 값는 0 또는 양수값을 갖습니다. 비선형 변환은 다음 공식으로 이뤄집니다.
$$
\hat{\mathbf{y}} = \mathrm{softmax}(\mathbf{o}) \text{ where }
\hat{y}_i = \frac{\exp(o_i)}{\sum_j \exp(o_j)}
$$

모든 $i$ 에 대해서  $0 \leq \hat{y}_i \leq 1$  이고  $\hat{y}_1 + \hat{y}_2 + \hat{y}_3 = 1$ 를 만족하는 것을 쉽게 확인할 수 있습니다. 따라서, $\hat{y}$ 은 적절한 확률 분포이고, $o$ 값은 쉽게 측정할 수 있는 값으로 간주할 수 있습니다. 아래 공식은 가장 가능성 있는 클래스를 찾아줍니다.
$$
\hat{\imath}(\mathbf{o}) = \operatorname*{argmax}_i o_i = \operatorname*{argmax}_i \hat y_i
$$

즉, softmax 연산은 예측하는 카테고리의 결과를 바꾸지 않으면서, 결과 $o$ 에 대한 적절한 의미를 부여해줍니다. 이것을 벡터 표현법으로 요약해보면, get ${\mathbf{o}}^{(i)} = \mathbf{W} {\mathbf{x}}^{(i)} + {\mathbf{b}}$,  ${\hat{\mathbf{y}}}^{(i)} = \mathrm{softmax}({\mathbf{o}}^{(i)})$ 이 됩니다.

## 미니 배치를 위한 벡터화

연산 효율을 더 높이기 위해서, 데이터의 미니 배치에 대한 연산을 벡터화합니다. 차원이 $d$ 이고 배치 크기가 $n$ 인 데이터들의 미니 배치  $\mathbf{X}$ 가 있고, 결과로 $q$ 개의 카테고리가 있다고 가정하겠습니다. 그러면, 미니 배치 feature  $\mathbf{X}$ 는  $\mathbb{R}^{n \times d}$ 에 속하고, 가충치들 $\mathbf{W}$ 는 $\mathbb{R}^{d \times q}$ 에, 편향(bias)  $\mathbf{b}$ 는 $\mathbb{R}^q$ 에 속합니다.
$$
\begin{aligned}
\mathbf{O} &= \mathbf{X} \mathbf{W} + \mathbf{b} \\
\hat{\mathbf{Y}} & = \mathrm{softmax}(\mathbf{O})
\end{aligned}
$$

이렇게 정의하면 가장 많이 차지하는 연산을 가속화할 수 있습니다. 즉,  $\mathbf{W} \mathbf{X}$ 이 형렬-벡터의 곱에서 행렬-행렬의 곱으로 변환됩니다. softmax는 결과  $\mathbf{O}$ 의 모든 항목에 지수 함수를 적용하고, 지수 함수들의 값의 합으로 정규화(normalize) 하는 것으로 계산됩니다.

## 손실 함수(loss function)

확률 결과를 출력하는 방법을 정의했으니, 이 값이 얼마나 정확한지를 측정하는 값으로 변환하는 것이 필요합니다. 즉, 손실 함수(loss function)가 필요합니다. 선형 회귀에서 사용했던 것과 동일한 개념을 사용하는데, 이는 가능도 최대화(likelihood maxmization_이라고 합니다.

### 로그 가능도(Log-Likelihood)

softmax 함수는 결과  $\mathbf{o}$ 를 여러 결과들에 대한 확률, $p(y=\mathrm{cat}|\mathbf{x})$, 들의 벡터로 변환합니다. 이는, 예측된 값이 얼마나 잘 예측하고 있는지를 확인하는 것으로 실제 값과 예측 결과에 대한 비교를 할 수 있습니다.
$$
p(Y|X) = \prod_{i=1}^n p(y^{(i)}|x^{(i)})
\text{ 이고, 그러므로 }
-\log p(Y|X) = \sum_{i=1}^n -\log p(y^{(i)}|x^{(i)})
$$

잘 예측하는 것은 $-\log p(Y|X)$ 를 최소화하는 것을 의미합니다. 이를 통해서 손실 함수(loss function)를 다음과 같이 정의할 수 있습니다. (표기를 간단하게 하기 위해서 $i$ 는 제외했습니다.)
$$
l = -\log p(y|x) = - \sum_j y_j \log \hat{y}_j
$$

여기서  $\hat{y} = \mathrm{softmax}(\mathbf{o})$ 이고, 벡터 $\mathbf{y}$ 는 해당하는 레이블이 아닌 위치에는 모두 0을 갖습니다. (예를 들면 (1,0,0)). 따라서, 모든 $j$ 에 대한 합을 하면, 하나의 항목만 남게 됩니다. 모든 $\hat{y}_j$ 는 확률값이기 때문에, 이에 대한 로그를 적용한(logarithm) 값은 0보다 커질 수 없습니다. 그 결과, 주어진 x에 대해서 y를 잘 예측하는 경우라면 (즉,  $p(y|x) = 1$), 손실 함수(loss function)는 최소화될 것입니다.

## Softmax와 미분(derivative)

Softmax와 이에 대한 손실(loss)는 많이 사용되기 때문에, 어떻게 계산되는지 자세히 살펴볼 필요가 있습니다.  $o$ 를 손실 $l$ 의 정의에 대입하고, softmax의 정의를 이용하면, 다음과 같이 표현을 얻습니다.
$$
l = -\sum_j y_j \log \hat{y}_j = \sum_j y_j \log \sum_k \exp(o_k) - \sum_j y_j o_j
= \log \sum_k \exp(o_k) - \sum_j y_j o_j
$$

어떤 일이 일어나는지 더 살펴보기 위해서, 손실 함수(loss function)를 $o$ 에 대해서 미분을 해보면 아래 공식을 유도할 수 있습니다.
$$
\partial_{o_j} l = \frac{\exp(o_j)}{\sum_k \exp(o_k)} - y_j = \mathrm{softmax}(\mathbf{o})_j - y_j = \Pr(y = j|x) - y_j
$$

다르게 설명해보면, 그래디언트(gradient)는 모델이  $p(y|x)$ 확률 표현식으로 예측한 것과 실제 값  $y_j$ 의 차이입니다. 이는 회귀 문제에서 보았던 것과 아주 비슷합니다. 회귀 문제에서 그래디언트(gradient)가 관찰된 실제 값 $y$ 와 예측된 값 $\hat{y}$ 의 차이로 계산되었습니다. 이는 너무 우연으로 보이는데, 사실은 그렇지 않습니다. [exponential 계열](https://en.wikipedia.org/wiki/Exponential_family)의 모델의 경우에는, 로그 가능도(log-likelihood)의 그래디언트(gradient)는 정확하게 이 항목으로 주어집니다. 이로 인해서 그래디언트(gradient)를 구하는 것이 실제 적용할 때 매우 간단해집니다.

### 크로스-엔트로피 손실(cross-entropy loss)

자 이제는 하나의 결과에 대한 관찰을 하는 경우가 아니라, 결과들에 대한 전체 분포를 다루는 경우를 생각해봅시다.  $y$ 에 대한 표기를 이전과 동일하게 사용할 수 있습니다. 오직 다른 점은 (0,0,1) 과 같이 이진(binary) 값을 갖는 것이 아니라 (0.1, 0.2, 0.7)과 같이 일반적인 확률 벡터를 사용한다는 것입니다. 손실 $l$ 의 정의도 동일한 수학을 사용하지만, 이에 대한 해석은 조금 더 일반적입니다. 레이블들의 분포에 대한 손실의 기대값을 의미합니다.
$$
l(\mathbf{y}, \hat{\mathbf{y}}) = - \sum_j y_j \log \hat{y}_j
$$

이렇게 정의된 손실는 크로스-엔트로피 손실(cross-entropy loss)이라고 부릅니다. 이것은 다중 클래스 분류에 가장 흔히 사용되는 손실 입니다. 이 이름에 대해서 알아보기 위해서는 정보 이론(information theory)에 대한 설명이 필요하며, 지금부터 설명하겠습니다. 다음 내용은 넘어가도 됩니다.

## 정보 이론(Information theory) 기초

정보 이론(information theory)는 정보 (또는 데이터)를 가능한 한 간결한 형식으로 인코딩, 디코딩, 전송, 및 변조하는 문제를 다룹니다.

### 엔트로피(Entropy)

데이터 (또는 난수)에 몇개의 정보 비트들이 담겨있는지가 중요한 개념입니다. 이는 분표 $p$ 의 [entropy](https://en.wikipedia.org/wiki/Entropy)로 다음과 같이 수치화할 수 있습니다.
$$
H[p] = \sum_j - p(j) \log p(j)
$$

정보 이론의 근본적인 이론 중에 하나로 분포 $p$ 로부터 임의로 추출된 데이터를 인코드하기 위해서는 최소  $H[p]$ 개의 'nat'이 필요하다는 것이 있습니다. 여기서 'nat'은 비트와 동일하나, 베이스(base) 2가 아니라 베이스(base) $e$ 를 이용합니다. 즉, 1 nat은 $\frac{1}{\log(2)} \approx 1.44$  비트이고,  $H[p] / 2$ 는 종종 이진 앤트로피(binary entropy)라고 불립니다.

조금 더 이론적으로 들어가보겠습니다. $p(1) = \frac{1}{2}$ 이고,  $p(2) = p(3) = \frac{1}{4}$ 인 분포를 가정하겠습니다. 이 경우, 이 분포에서 추출한 데이터에 대한 최적의 코드를 굉장히 쉽게 설계할 수 있습니다. 즉, 1의 인코딩은 `0`, 2와 3에 대한 인코딩은 각 각 `10`, `11` 로 정의하면 됩니다. 예상되는 비트 개수는  $1.5 = 0.5 * 1 + 0.25 * 2 + 0.25 * 2$ 이고, 이 숫자는 이진 앤트로피(binary entropy) $H[p] / \log 2​$ 와 같다는 것을 쉽게 확인할 수 있습니다.

### Kullback Leibler Divergence

두 분포간에 차이를 측정하는 방법 중에 하나로 앤트로피(entropy)를 이용하는 방법이 있습니다. $H[p]$ 는 분포 $p$를 따르는 데이터를 인코드하는데 필요한 최소 비트 수를 의미하기 때문에, 틀린 분포 $q$ 에서 뽑았을 때 얼마나 잘 인코딩이 되었는지를 물어볼 수 있습니다. $q$ 를 인코딩하는데 추가로 필요한 비트 수는 두 분표가 얼마나 다른지에 대한 아이디어를 제공합니다. 직접 계산해보겠습니다. 분포 $q$ 에 대해 최적인 코드를 이용해서 $j$ 를 인코딩하기 위해서는 $-\log q(j)$ nat이 필요하고,  $p(j)$ 인 모든 경우에서 이를 사용하면, 다음 식을 얻습니다.
$$
D(p\|q) = -\sum_j p(j) \log q(j) - H[p] = \sum_j p(j) \log \frac{p(j)}{q(j)}
$$

$q$ 에 대해서  $D(p\|q)$ 를 최소화하는 것은 크로스-엔트로피 손실(cross-entropy loss)을 최소화하는 것과 같습니다. 이는 $q$ 에 의존하지 않는 $H[p]$ 를 빼버리면 바로 얻을 수 있습니다. 이를 통해서 우리는 softmax 회귀(regression)는 예측된 값  $\hat{y}$ 이 아니라 실제 레이블 $y$ 를 봤을 때 얻는 놀라움(비트 수)을 최소화하려는 것임을 증명했습니다.

## 모델 예측 및 평가

학습된 softmax 회귀(regression) 모델을 사용하면, 새로운 특성(feature)가 주어졌을 때, 각 결과 카테고리에 속할 확률값을 예측할 수 있습니다. 일반적으로는 가장 크게 예측된 확률값을 갖는 카테고리를 결과 카테고리라고 정의합니다. 실제 카테고리 (label)와 일치하는 경우에 예측이 정확하다고 합니다. 다음에는 모델의 성능을 평가하는 방법으로 accuracy 정확도를 사용할 예정입니다. 이는 정확하게 예측한 개수와 전체 예측의 개수의 비율과 같습니다. 

## 요약

* 벡터를 확률로 변환하는 softmax 연산을 알아봤습니다.
* softmax 회귀(regression)은 분류의 문제에 적용할 수 있습니다. softmax 연산을 이용해서 얻은 결과 카테고리의 확률 분포를 이용합니다.
* 크로스 엔트로피(cross entropy)는 두 확률 분포의 차이를 측정하는 좋은 방법입니다. 이는 주어진 모델이 데이터를 인코드하는데 필요한 비트 수를 나타냅니다.

## 문제

1. Kullback-Leibler divergence $D(p\|q)$ 가 모든 음수가 분포 $p$, $q$ 에 대해서 음수가 아님을 증명하세요. 힌트 - Jensen의 부등식을 이용하세요. 예를 들여,  $-\log x$ 가 볼록 함수(convex function)이라는 사실을 사용하세요.
1. $\log \sum_j \exp(o_j)$ 가  $o$에서 볼록 함수(convex function)임을 증명하세요.
1. 지수승 집단과 softmax를 관계를 심도있게 알아볼 수 있습니다.
    * softmax에 대한 크로스 엔트로피 손실 $l(y,\hat{y})$ 의 이차 미분을 계산하세요.
    *  $\mathrm{softmax}(o)$ 로 주어지는 분포의 분산을 계산하고, 위에서 계산한 이차 이분과 같음을 증명하세요.
1. 동일한 확률로 일어나는 3개 클래스가 있다고 가정합니다. 즉, 확률 벡터가  $(\frac{1}{3}, \frac{1}{3}, \frac{1}{3})$ 입니다.
    * 이 문제를 위한 이진 코드(binary code)를 설계하고자 하면 어떤 문제가 있을까요? 앤트로피의 하한(lower bound)를 필요한 비트수로 같게할 수 있나요?
    * 더 좋은 코드를 설계할 수 있나요? 힌트 - 두 독립적인 관찰을 인코딩하려면 어떤일이 생기나요? $n$개의 관찰을 연관해서 인코드하면 어떨까요?
1. Softmax는 위에서 소개된 매핑에 대한 잘못된 이름입니다 (하지만 딥러닝에서 많은 사람들이 쓰고 있습니다.) 실제 softmax는 $\mathrm{RealSoftMax}(a,b) = \log (\exp(a) + \exp(b))$ 로 정의됩니다.
    * $\mathrm{RealSoftMax}(a,b) > \mathrm{max}(a,b)$ 임을 증명하세요.
    * $\lambda > 0$ 일 경우, 모든  $\lambda^{-1} \mathrm{RealSoftMax}(\lambda a, \lambda b)$ 에 대해서 이것이 성립함을 증명하세요
    * $\lambda \to \infty$ 이면,  $\lambda^{-1} \mathrm{RealSoftMax}(\lambda a, \lambda b) \to \mathrm{max}(a,b)$ 임을 증명하세요.
    * soft-min은 어떻게 생겼을까요?
    * 이를 두개 이상의 숫자들로 확장해보세요.

## QR 코드를 스캔해서 [논의하기](https://discuss.mxnet.io/t/2334)

![](../img/qr_softmax-regression.svg)