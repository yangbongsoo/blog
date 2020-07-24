# 확률과 통계

## 헷갈렸던 부분들 정리

$$X \thicksim N(\mu, \sigma^2) $$ 에서 $$\mu$$ 는 모평균이고 $$\sigma^2$$ 은 모분산이다. <br>
$$X_1, X_2, ... ,X_n \overset{iid}{\thicksim} N(\mu, \sigma^2)$$ 일때 <br>
$$\bar{X} \thicksim N(\mu, {\sigma^2 \over n}) $$ 은 $$\mu$$ 를 추정하기 위해서 사용한다. <br>
$$\bar{X}$$ 는 표본평균이며 $$\bar{X} = {X_1 + X_2 + ... + X_n \over n}$$ 이다. <br>
$$\sigma^2 \over n$$ 은 표본평균의 분산이다. <br>
표본분산 $$S^2 = {1 \over n-1}\sum_{i=1}^n(\bar{X}-X_i)^2$$ 은 모분산을 추정하기 위해 사용한다. <br>
표본분산은 관측된 표본 $$X_i$$ 들이 평균을 기준으로 얼마나 퍼져있는지 나타낸다. <br>

$$E \left[ X \right] = \mu$$ <br>
$$E \left[ \bar{X} \right] = E \left[ {X_1+X_2+...+X_n \over n} \right]$$ <br>
$$\Leftrightarrow {1 \over n}(E \left[ X_1 \right] + E \left[ X_2 \right] + ... + E \left[ X_n \right]) $$ <br>
$$\Leftrightarrow {1 \over n}(nE \left[ X \right]) \quad (identical)$$ <br>
$$\Leftrightarrow E \left[ X \right]$$ <br>

$$Var(X) = \sigma^2$$ <br>
$$Var(\bar{X}) = Var({X_1+X_2+...+X_n \over n}) = {1 \over n^2}Var(X_1+X_2+...+X_n)$$ <br>
$${1 \over n^2} nVar(X) = {\sigma^2 \over n}$$ <br>

## p-value 에 대한 글

"유의성검정은 과학의 작동 방식과 반대의 기능을 수행한다. 상식적으로 과학 이론이 인정받기 위해서는 숱한 도전을 이겨내야 한다.
즉 해당 이론이 틀렸다고 주장하는 수많은 반박 시도를 이겨내고 나서야 학계에서 인정받을 수 있다. 이는 ‘반증주의’로 알려진 과학철학적 입장이기도 하다.
그런데 앞서 언급했듯, 영가설 유의성검정에서 혹독한 검증 절차를 거치는 것은 연구자가 주장하는 ‘연구가설’이 아니라, 그것이 틀렸다고 주장하는 ‘영가설’ 이다.
다시 말해 유의성검정은 연구자의 가설이 아니라, 그것에 대한 반박 시도를 혹독하게 검증하는 절차라 할 수 있다.
이는 반증주의적 과정과는 반대라 할 수 있다."

유의성 검정에 대한 글이다. H0 를 기각하기 때문에 H1을 선택하는게 당연한 flow 라고 생각했었는데 그게 올바르지 않을 수 있다는걸
깨우쳐줬다.

원문 : http://scienceon.hani.co.kr/402347






