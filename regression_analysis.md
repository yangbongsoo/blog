# 회귀분석

변수들간의 함수관계를 추구하는 통계적 방법을 회귀분석이라고 부른다.
회귀라는 말은 영국의 우생학자 F.Galton 이 처음으로 불렀다고 한다.
그는 아버지의 신장(키)과 아들의 신장의 관계를 조사했다. 그리고 아들들의 신장은
인간 전체의 평균 신장에 되돌아가려는 경향이 있다는걸 밝혔다. 이러한 함수관계를
'regression' 이라는 용어로 처음 표현한 사람이 F.Galton 이고 유래가 되었다.

## 단순회귀분석

먼저 제일 간단한 경우에 해당하는 선형관계에 관한 분석을 다뤄보자.

광고가 상품 판매량에 미치는 관계를 알아보기 위해 10개의 상점 표본(sample)을
추출하여 아래와 같이 표로 정리했다.

|상점번호|광고료 <br> (단위 : 10만원)|총판매액 <br> (단위 : 100만원)|
|------|---|---|
|1|4|9|
|2|8|20|
|3|9|22|
|4|8|15|
|5|8|17|
|6|12|30|
|7|6|18|
|8|10|25|
|9|6|10|
|10|9|20|

파이썬으로 산점도를 그리면 다음과 같다.

![](/assets/regression1.png)

```python
import matplotlib.pyplot as plt

xData = [4,8,9,8,8,12,6,10,6,9] # 광고료
yData = [9,20,22,15,17,30,18,25,10,20] # 총판매액
plt.scatter(xData, yData)
plt.xlabel('ad cost')
plt.ylabel('total')
plt.title('result')
plt.grid(True)
plt.show()
```

여기서 변수 x는 광고료고 y는 총판매액이다.
이 상황에서 단순회귀모형을 표현하면 다음과 같다.

$$y_i = \beta_0 + \beta_1 + \epsilon_i$$

갑자기 수식이 나와서 이게 뭔가 싶지만 하나씩 살펴보면 크게 어려운것은 없다.
수식에 대해 알아보기 전에 먼저 우리가 뭘 하고자 하는지 명확하게 짚고 넘어갈 필요가 있다.

우리는 주어진(위의 10개 샘플) 값을 이용해 10개 값들을 대표할 선형식을 구하고자 한다.
다시말해 1차함수 $$y=ax+b$$ 에서 a와 b를 구하고 싶은거다. 그리고 a는 기울기, b는 절편이다.

주어진 샘플을 이용해 회귀선(아까 말한 1차함수)을 구하면 그 추정치를 통해 다음을 예측할 수 있다.
다시 단순회귀모형으로 돌아와서 하나씩 자세히 살펴보자.

$$y_i = \beta_0 + \beta_1 + \epsilon_i$$ <br>


$$x_i$$ : i번째 주어진 고정된 x 값 <br>
$$y_i$$ : i번째 측정된 y의 값 <br>
$$\beta_0, \beta_1$$ : 모집단의 회귀계수 <br>
$$\epsilon_i$$ : i 번째 측정된 y의 오차항으로 epsilon 이라고 읽는다. 확률분포는 $$N(0,\sigma^2)$$ 이며, 다른 오차항과는 상관관계가 없다. <br>

여기서 $$\epsilon_i$$ 에 대해서 좀 더 자세히 짚고 넘어가자. 먼저 왜 오차의 확률분포가 정규분포일까? <br>
답은 단순하다. 정규분포 자체가 원래 오차에 대한 확률분포다. 정규분포는 여러 수학자에 의해 각기 다른 방식으로
연구되었지만 가우스의 경우 천제 관측 시 발생하는 오차의 성질을 연구하던 중 정규분포를 발견했다.

$$y = \mu_{y*x} + \epsilon$$

$$\mu_{y*x}$$


### 회귀선의 추정

표본 자료(sample data)로부터 선형식을 추정하여 얻은 직선을 다음과 같이 표기한다. <br>
$$\hat y = b_0 + b_1x$$ <br>

이와 같은 직선을 회귀직선 또는 회귀선이라고 부른다. 여기에서 $$b_0, b_1, \hat y$$ 는 각각
$$\beta_0, \beta_1, \mu_{y*x}$$ 의 추정값(estimate) 이다.


**최소 제곱법**<br>

$$y_i = \beta_0 + \beta_1 + \epsilon_i$$ 식에서 오차제곱들의 합은 다음과 같다.

$$S = \sum_{i=1}^n \epsilon_i^2 = \sum_{i=1}^n(y_i-\beta_0-\beta_1x_i)^2$$

이 $$S$$를 최소로 하는 $$\beta_0, \beta_1$$ 값을 구하는 방법이 최소제곱법이다.

$$\beta_0, \beta_1$$ 각각을 편미분해야 한다.

먼저 $$\beta_0$$ 편미분 해보자. <br>
$$\beta_0$$ 로 미분하고 나머지는 상수로 취급한다. $$\partial S \over \partial\beta_0 $$ <br>

$$\sum_{i=1}^n(-\beta_0+ A_i)^2 $$ : $$\beta_0$$ 만 미분하고 나머지는 상수로 취급하니까
나머지를 $$A_i$$ 로 치환했다($$A_i = y_i - \beta_1x_i$$).

시그마는 마지막에 추가해주면 되니까 $$(-\beta_0+ A_i)(-\beta_0+ A_i)$$ 를 미분해야 한다.

cf) $$f(x)g(x)$$ 미분은 $$f^\prime(x)g(x) + f(x)g^\prime(x)$$ 이다.

따라서 $$(-\beta_0+ A_i)(-\beta_0+ A_i)$$ 를 미분하면 <br>
$$(-1)(-\beta_0+ A_i) + (-\beta_0+ A_i)(-1)$$ 된다. <br>
그리고 $$(\beta_0 - A_i) + (\beta_0 - A_i) = 2(\beta_0 - A_i)$$ 이다. <br>
최종적으로는 $$\sum_{i=1}^n2(\beta_0 - A_i) = \sum_{i=1}^n2(\beta_0 - y_i + \beta_1x_i) = -2\sum_{i=1}^n(y_i -\beta_0  - \beta_1x_i)$$ 이 된다.

cf) $$f_1 + f_2 + f_3 + ... + f_n $$ 을 미분하면 $$f^\prime_1 + f^\prime_2 + f^\prime_3 + ... + f^\prime_n $$ 이다.
그러므로 미분하고 시그마만 취해주면 된다.

다음으로 $$\beta_1$$ 편미분 해보자. <br>
$$\beta_1$$ 로 미분하고 나머지는 상수로 취급한다. $$\partial S \over \partial\beta_1 $$ <br>

$$\sum_{i=1}^n(A_i -\beta_1x_i)^2 $$ : $$\beta_1$$ 만 미분하고 나머지는 상수로 취급하니까
나머지를 $$A_i$$ 로 치환했다($$A_i = y_i - \beta_0$$).

$$(A_i -\beta_1x_i)^2$$ 을 $$\beta_1$$ 으로 미분하기 위해선 합성함수 미분이 필요하다. <br>
cf) $$f(g(x))$$ 를 미분하면 $$f^\prime(g(x)) * g^\prime(x)$$ 된다. <br>
$$g(\beta_1) = -x_i\beta_1 + A_i$$ <br>
$$f(x) = x^2$$<br>
$$f(g(\beta_1)) = (-\beta_1x_i + A_i)^2$$ <br>
따라서 미분을 하게 되면 $$-2\Sigma x_i(-x_i\beta_1+A_i)$$ 이 되고 (시그마 첨자 생략) <br>
최종적으로 $$-2\Sigma x_i(-x_i\beta_1+ y_i-\beta_0)$$ 가 된다. <br>

정리하면 다음과 같은 결과를 얻게 된다.

$${\partial S \over \partial\beta_0} = -2\Sigma (y_i -\beta_0  - \beta_1x_i)$$ <br>
$${\partial S \over \partial\beta_1} = -2\Sigma x_i(y_i -\beta_0 -\beta_1x_i)$$ <br>

이 편미분 값을 0으로 만드는 $$\beta_0, \beta_1$$ 을 각각 $$b_0, b_1$$ 으로 대치하여 정리하면
다음과 같고 이 식을 정규방정식(normal equations) 이라고 부른다.

$$b_0n + b_1\Sigma x_i = \Sigma y_i$$ <br>
$$b_0\Sigma x_i + b_1\Sigma x_i^2 = \Sigma x_iy_i$$ <br>

하지만 편미분해서 얻어진 결과를 0으로 놓고 해를 구했다고 해서 그 해가 S 를 최소화한다는 보장은 없다.
미분해서 0이 되는 것만으로는 2차원 그래프의 최대 최소를 알 수 없고 부가적인 정보가 필요하다.

cf) 수학적으로 이에 대한 추가 설명이 있는데 일단 생략(이차 편미분 행렬이 PD 행렬이면 볼록함수고 역도 성립)

다시 돌아와서 <br>
$$b_0n + b_1\Sigma x_i = \Sigma y_i$$ <br>
$$b_0\Sigma x_i + b_1\Sigma x_i^2 = \Sigma x_iy_i$$ <br>

위의 정규방정식을 $$b_0, b_1$$ 에 대해서 풀면 다음과 같다.

$$b_1 = {\Sigma x_iy_i - {(\Sigma x_i)(\Sigma y_i) \over n} \over \Sigma x_i^2 - { (\Sigma x_i)^2  \over n} } = {
\Sigma (x_i - \bar x)(y_i - \bar y) \over \Sigma (x_i - \bar x)^2}$$ <br>

$$b_0 = {\Sigma y_i \over n} - b_1 {\Sigma x_i \over n} = \bar y - b_1 \bar x$$ <br>

수식은 복잡해 보이지만 파이썬으로 구하면 금방이다.

```python
# 최소제곱법 계산(method of least squares)
# x평균, y평균 구하기
xAvg = np.mean(xData)
yAvg = np.mean(yData)

# b1 구하기
numerator = sum((xData - xAvg) * (yData - yAvg)) # 분자
denominator = sum((xData - xAvg) ** 2) # 분모

b1 = numerator/denominator
print(b1)

# b0 구하기
n = len(xData)
b0 = (sum(yData) / n) - (b1 * (sum(xData) / n))
print(b0)
```

그래서 최소제곱법으로 적합된 회귀직선은 다음과 같다. <br>
$$\hat y = -2.2695652173913032 + 2.608695652173913x $$ <br>

구한 회귀선을 위의 산점도와 합쳐서 함께 보면 다음과 같다.

```python
xData = [4,8,9,8,8,12,6,10,6,9] # 광고료
yData = [9,20,22,15,17,30,18,25,10,20] # 총판매액
plt.scatter(xData, yData)
plt.xlabel('ad cost')
plt.ylabel('total')
plt.title('result')
plt.grid(True)

x = np.arange(4, 13, 1)
y = -2.2695652173913032 + 2.608695652173913 * x

plt.plot(x, y)
plt.show(
```

![](/assets/regression-equation1.png)

### 회귀선의 정도

회귀선만 갖고는 이 점들을 어느 정도 잘 대변하여 주고 있는가를 알 수 없다.




참고1 : 회귀분석 박성현 저 <br>
참고2 : https://brunch.co.kr/@gimmesilver/66 <br>