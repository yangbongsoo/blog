# 확률과 통계

## 모평균, 모분산, 표본평균, 표본분산
$$X \thicksim N(\mu, \sigma^2) $$ 에서 $$\mu$$ 는 모평균이고 $$\sigma^2$$ 은 모분산이다. <br>
$$X_1, X_2, ... ,X_n \overset{iid}{\thicksim} N(\mu, \sigma^2)$$ 일때 <br>
$$\bar{X} \thicksim N(\mu, {\sigma^2 \over n}) $$ 은 $$\mu$$ 를 추정하기 위해서 사용한다. <br>
$$\bar{X}$$ 는 표본평균이며 $$\bar{X} = {X_1 + X_2 + ... + X_n \over n}$$ 이다. <br>
$$\sigma^2 \over n$$ 은 표본평균의 분산이다. <br>
표본분산 $$S^2 = {1 \over n-1}\sum_{i=1}^n(\bar{X}-X_i)^2$$ 은 모분산을 추정하기 위해 사용한다. <br>
표본분산은 관측된 표본 $$X_i$$ 들이 평균을 기준으로 얼마나 퍼져있는지 나타낸다. <br>

$$E \left[ X \right] = \mu$$ <br>
$$E \left[ \bar{X} \right] = E \left[ {X_1+X_2+...+X_n \over n} \right]$$ <br>
$$E \left[ \bar{X} \right] = {1 \over n}(E \left[ X_1 \right] + E \left[ X_2 \right] + ... + E \left[ X_n \right]) $$ <br>
$$E \left[ \bar{X} \right] = {1 \over n}(nE \left[ X \right]) \quad (identical)$$ <br>
$$E \left[ \bar{X} \right] = E \left[ X \right]$$ <br>
통계량의 기대값($$E \left[ \bar{X} \right]$$)이 모수($$\mu$$)와 일치할 때 $$\bar{X}$$를 모수의 불편추정량(unbiased estimator) 이라고 한다.<br>

$$Var(X) = \sigma^2$$ <br>
$$Var(\bar{X}) = Var({X_1+X_2+...+X_n \over n}) = {1 \over n^2}Var(X_1+X_2+...+X_n)$$ <br>
$${1 \over n^2} nVar(X) = {\sigma^2 \over n}$$ <br>

편차란 기대값 $$\mu$$ 를 기준으로 퍼져있는 정도를 말한다. 부호는 중요하지 않으니까 제곱한다. 그리고 그거의 기대값은 분산이다. <br>
$$E \left[ (x- \mu)^2 \right] = E \left[ X^2 \right] - \{ E \left[ X \right] \}^2 $$ 그리고 이건 모분산이다. <br>

## 표준화
표준화는 기대값을 0, 분산을 1로 만드는 작업이다. <br>
$$Z = {X- E \left[ X \right] \over \sqrt {Var(x)}} = {X- \mu \over \sigma}$$ <br>
분자를 V 로 치환하자. <br>
$$V = X- E \left[ X \right]$$ <br>
$$E \left[ V \right] = E \left[ X \right] - E \left[ E \left[ X \right] \right]$$ <br>
$$E \left[ E \left[ X \right] \right]$$ 를 먼저 구해보자. <br>
$$E \left[ E \left[ X \right] \right] = \int_{-\infty}^{\infty}(\int_{-\infty}^{\infty}xf(x)dx)f(x)dx$$ <br>
$$E \left[ E \left[ X \right] \right] = \int_{-\infty}^{\infty} E \left[ X \right] f(x)dx$$ <br>
$$E \left[ E \left[ X \right] \right] = E \left[ X \right] \int_{-\infty}^{\infty} f(x)dx$$ (기대값은 상수니까 앞으로 나올 수 있다)<br>
$$E \left[ E \left[ X \right] \right] = E \left[ X \right] * 1$$ (pdf) <br>
$$E \left[ V \right] = 0$$ 이다. 그러므로 $$E \left[ Z \right]$$ 는 0이 된다. <br>

$$Var(V) = E \left[ (V - E \left[ V \right])^2 \right]$$ <br>
$$Var(V) = E \left[ V^2 \right]$$ (위에서 $$E \left[ V \right] = 0$$ 임을 알았기 때문) <br>
$$Var(V) = E \left[ (X- E \left[ X \right])^2 \right]$$ <br>
$$Var(V) = Var(X) $$ <br>
그러므로 $$Var(Z) = Var({V \over \sqrt{Var(X)}}) = { 1 \over Var(X)} Var(V) = 1 $$ 이 된다.

## 확률변수
주사위를 예를 들어 보자.

|$$X$$|$$1$$|$$2$$|$$3$$|$$4$$|$$5$$|$$6$$|
|---|---|---|---|---|---|---|
|$$P(X)$$|$$1 \over 6$$|$$1 \over 6$$|$$1 \over 6$$|$$1 \over 6$$|$$1 \over 6$$|$$1 \over 6$$|

이상황에서 확률변수 $$X$$는 주사위를 한번 던졌을 때 나오는 눈의 값이다. <br>
다시말해 주사위를 한번 던졌을 때 나오는 모든 경우(즉 모집단)를 실수로 대응시켜주는 함수(수치화 시켜주는)이다. <br>
다시말해 이벤트를 확률변수 라는 규칙을 통해 숫자화 시키는거고 그게 데이터다. 그리고 이 데이터가 셀 수 있으면 이산(discrete), 셀 수 없으면
연속(continuous) 이다. <br>

cdf : $$P(X \le x) = P(X \in A), A = (- \infty, x] $$ 이다. <br>
확률변수는 함수이므로 좀 더 엄밀히 따지면 $$P(X(w) \le x) = P(X(w) \in A)$$ 이고 <br>
조건제시법으로 나타내면 $$P(\{ w| X(w) \in A \})$$ 이다. <br>
해석하자면 $$\Omega$$ 의 원소 전체를 $$\omega$$ 에 다 넣어서(확률변수 $$X$$라는 함수에) 나오는 결과값이 $$A$$ 라는 집합에 속할 확률이다.<br>
따라서 $$X$$ 는 $$\Omega$$ -> $$R$$ 함수이고 $$\omega$$ 는 $$\Omega$$ 의 원소이고 $$X(w)$$ 결과값은 $$R$$에 속한다.

## 몬테카를로 방법
수학이나 물리 등의 분야에서 답을 찾기 어려운 문제를 만났을 때는, 반복 계산에 능한 컴퓨터를 동원해서 확률 통계적인 방법으로
적절한 추정값을 얻기도 한다. 이 방법은 난수(random number)에 의한 반복 시뮬레이션에 바탕을 둔다.

예를 들어 몬테카를로 방법으로 무리수 $$\sqrt{3}$$ 의 값을 구해보자. <br>
먼저 0과 3 사이의 한 점을 무작위로 택해서 x라 둔다. 이때 x가 $$\sqrt{3}$$ 보다 작을 확률은 얼마일까? <br>
전체 범위의 크기가 3이므로, 구하는 확률은 $$\sqrt{3} \over 3$$ 이 될 것이다. <br>

지금은 $$\sqrt{3}$$ 의 값을 모르는 상태이기 때문에 $$x$$ 가 $$\sqrt{3}$$ 보다 작은지 판단할 수는 없지만, 그 대신
$$x < \sqrt{3}$$ 의 양변을 제곱하면 $$x^2 < 3$$ 이 된다는 것을 이용할 수 있다. 즉, 난수 $$x$$ 를 제곱해서 3보다 작은지 살펴보는 것이다.<br>

이제 이런 테스트를 N번 반복한다고 하자. N번의 시행 중에 난수 $$x$$의 제곱이 3보다 작은 경우가 k번 있었다면, 확률적으로 다음이 성립해야 한다.<br>
$${k \over N} \approx {\sqrt{3} \over 3}$$ <br>
따라서 $$\sqrt{3}$$ 의 대략적인 값은 다음과 같다. <br>
$$\sqrt{3} \approx {3k \over N}$$ <br>

아래는 실제 파이썬 코드이다.
```python
N = 100000 # 반복할 횟수
R = 3
k = 0
for i in range(N):
    x = random.uniform(0, R)
    if x*x < R: k += 1
print(k*R/N)
```


참고 : 수학리부트 강중빈 저

## 확률분포들

### uniform
가장 간단한 분포이다. a, b 구간에서 k 로 y값이 일정한 분포라고 해보자.
$$f(x) = k$$ <br>
$$\int_{a}^{b} f(x) \ dx = \int_{a}^{b}k \ dx = \left[ kx \right]_{a}^{b}$$ <br>
$$= kb-ba=1$$ <br>
$$= k = {1 \over b-a}$$ <br>
$$= f(x) = {1 \over b-a}$$ <br>
$$\therefore X \sim U[a,b]$$

### 베르누이(Bernoulli)

$$\Omega$$ -> $$R \{ 0, 1\}$$


### binomial distribution


## p-value 에 대한 글

"유의성검정은 과학의 작동 방식과 반대의 기능을 수행한다. 상식적으로 과학 이론이 인정받기 위해서는 숱한 도전을 이겨내야 한다.
즉 해당 이론이 틀렸다고 주장하는 수많은 반박 시도를 이겨내고 나서야 학계에서 인정받을 수 있다. 이는 ‘반증주의’로 알려진 과학철학적 입장이기도 하다.
그런데 앞서 언급했듯, 영가설 유의성검정에서 혹독한 검증 절차를 거치는 것은 연구자가 주장하는 ‘연구가설’이 아니라, 그것이 틀렸다고 주장하는 ‘영가설’ 이다.
다시 말해 유의성검정은 연구자의 가설이 아니라, 그것에 대한 반박 시도를 혹독하게 검증하는 절차라 할 수 있다.
이는 반증주의적 과정과는 반대라 할 수 있다."

유의성 검정에 대한 글이다. H0 를 기각하기 때문에 H1을 선택하는게 당연한 flow 라고 생각했었는데 그게 올바르지 않을 수 있다는걸
알려줬다.

원문 : http://scienceon.hani.co.kr/402347

