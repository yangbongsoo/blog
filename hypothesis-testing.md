# 가설 검정(Hypothesis-Testing)

### EXERCISES 4.5.5

Let $$X_1, X_2$$ be a random sample of size n = 2 from the distribution having <br>
pdf $$ f(x;\theta) = (1/\theta)e^{-x/\theta}, \ 0 < x < \infty$$ zero elsewhere. <br>
We reject $$H_0 : \theta = 2$$ and accept $$H_1 : \theta = 1$$ if the observed values of $$X_1, X_2$$, <br>
say $$x_1, x_2$$ are such that $${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$. <br>
Here $$\Omega = \{ \theta:\theta = 1,2 \}$$. <br>
Find the significance level of the test and the power of the test when $$H_0$$ is false. <br>

먼저 문제에서 기각역을 알려줬다. $$x_1, x_2$$ 의 observed values 가 $${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$
를 만족할 때 H0를 reject 하고 H1을 accept 한다고 했다. 기각역을 먼저 구하면 아래와 같다. <br>

$${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$ <br>
$$\Leftrightarrow {{1\over2}e^{-x_1 \over 2}{1\over2}e^{-x_2 \over 2} \over e^{-x_1}e^{-x_2}} \le {1\over2}$$ <br>
$$\Leftrightarrow {e^{-x_1 \over 2}e^{-x_2 \over 2} \over e^{-x_1}e^{-x_2}} \le 2$$ <br>
$$\Leftrightarrow {e^{-x_1-x_2 \over 2}  \over e^{-x_1-x_2}} \le 2$$<br>
$$\Leftrightarrow {e^{{-x_1-x_2 \over 2} - ({-x_1-x_2})} } \le 2$$<br>
$$\Leftrightarrow {e^{{x_1+x_2 \over 2} }} \le 2$$<br>
$$\Leftrightarrow \log_{e}{e^{{x_1+x_2 \over 2} }} \le \log_{e}2$$<br>
$$\Leftrightarrow {x_1+x_2 \over 2} \le \ln2$$<br>
$$\Leftrightarrow {x_1+x_2} \le 2\ln2$$(기각역)<br>

**power를 구해라**<br>
power(검정역) 는 1 - Type2 Error 이다. Type2 Error 는 H0를 선택했는데 H1이 참인 경우다. <br>
power 가 클수록 Type2 Error 는 작아지니까 올바른 의사결정을 내릴 확률이 커진다. <br>
power = $$1 - \Pr_{H_1}(H_0 accept)$$ <br>
power = $$\Pr_{H_1}(H_0 reject)$$ <br>
즉, H1이 참인 상황이고, 데이터도 기각역 안에 들어있을 확률을 말한다. <br>

$$\theta = 1$$ 일 때 관측된 데이터 $$x_1, x_2$$ 가 $${x_1+x_2} \le 2\ln2$$ 에 들어있을 확률 <br>
$$\Pr_{\theta=1}({X_1+X_2} \le 2\ln2)$$ 를 구하면 된다. <br>

우선, 확률의 기본적 개념은 P(A), 즉 집합의 형태로 표현된 A에 0에서 1 사이의 수를
부여하는 것이다. 현재 A는 두 관측값 x1과 x2의 합이 2ln2 보다 작게 되는 모든 (x1, x2)의 집합이다.
그러니까 적분도 x1, x2에 대한 중적분이 된다(x1, x2 범위를 명확히 정리해 주어야 적분값을 계산할 수 있다).

확률을 계산하는 방법은 크게 2가지이다. (1) $$X_1 + X_2 = U, X_2 = V$$ 로의 transformation 후 U 에 대한
marginal pdf를 구해 계산하는 방식. (2) 기존에 주어진 것처럼, 적분 구간을 정리해서 한번에 적분해 주는 방식.
(2) 는 주어진 부등식과 각 확률변수들의 support 들을 적절히 조합해주는 작업이 수반된다.

이 문제에서는 $$X_1, X_2$$ 2개의 확률변수를 다루고, 이 두 변수가 독립이기 때문에
(2) 의 방법이 좀 더 계산이 적어서 (1) 보다 낫다. 그리고 $$X_1, X_2$$ 가 지수분포이므로 이 둘은 positive임이 명백하다.<br>

적분 구간을 정리하면 다음과 같다. <br>
$$X_1 + X_2 \le 2\ln2, \quad X_1 >0, \quad X_2 >0$$ <br>
$$\Leftrightarrow 0 < X_1 < X_1 + X_2 \le 2ln2$$ <br>
$$\Leftrightarrow 0 < X_1 < 2ln2, \quad 0 < X_2 \le 2ln2 - X_1$$ <br>

$$X_1, X_2$$ 가 독립이기 때문에 joint pdf 가 각각의 marginal로 분리되고 이로 인해 계산이 수월해진다(분리가 안되는 경우 계산이 복잡해 지는 경우가 많음). <br>
$$\Pr_{\theta=1}({X_1+X_2} \le 2\ln2)$$ <br>
$$\Leftrightarrow P(0 < X_1 < 2ln2 \quad AND \quad 0 < X_2 \le 2ln2 - X_1)$$ <br>
$$\Leftrightarrow \int_{0}^{2ln2} \int_{0}^{2ln2-x_1} f_{1,2}(x_1,x_2) \ dx_2, dx_1$$ <br>
$$\Leftrightarrow \int_{0}^{2ln2} \int_{0}^{2ln2-x_1} f_1(x_1)f_2(x_2) \ dx_2, \ dx_1 (independence)$$ <br>
$$\theta=1$$ 일때 계산하기 때문에 문제 pdf $$f(x;\theta) = (1/\theta)e^{-x/\theta} 에 1을 넣으면 된다.<br>
$$\Leftrightarrow \int_{0}^{2ln2} \int_{0}^{2ln2-x_1} e^{-x_1}e^{-x_2} \ dx_2, \ dx_1 $$ <br>
$$\Leftrightarrow \int_{0}^{2ln2} e^{-x_1} \int_{0}^{2ln2-x_1} e^{-x_2} \ dx_2, \ dx_1 $$ <br>
안쪽 정적분부터 정리해야 한다. <br>
$$\left[ -e^{-x_2} \right]_{0}^{2ln2-x_1} = 1-e^{x_1-2ln2}$$ <br>
$$\Leftrightarrow \int_{0}^{2ln2} e^{-x_1} (1-e^{x_1-2ln2}) \ dx_1 $$ <br>
$$\Leftrightarrow \int_{0}^{2ln2} (e^{-x_1} -e^{-2ln2}) \ dx_1 $$ <br>
$$\Leftrightarrow \int_{0}^{2ln2} (e^{-x_1} - {1 \over 4}) \ dx_1 $$ <br>
$$\Leftrightarrow \left[ -e^{-x_1} - {1 \over 4}x_1 \right]_{0}^{2ln2} = -e^{-2ln2} -{1 \over 4}2ln2 + 1 $$ <br>
$$\Leftrightarrow {3 \over 4} - {1 \over 2}ln2 = 0.40342640972002736 $$ <br>

cf1) 지수함수 $$y = e^x $$ 를 미분하면 $$y \prime = e^x$$ 똑같다. <br>
cf2) $$\int e^{f(x)} = {1 \over f\prime(x)}e^{f(x)} + C$$ <br>



cf) support 은 확률값이 0이 아닌 것들을 말한다. <br>
area where the value of pdf(pmf) is none-zero $$\{x | p(x) > 0\}$$ <br>

**유의수준을 구해라**<br>
유의수준(significance level)은 Type1 Error 의 최대 허용범위다. Type1 Error는 H1을 선택했는데 H0가 참인 경우다. <br>
Type1 Error 는 H1을 선택했고 H0가 참일 때다. <br>
H1을 선택했다는 말은 H0를 기각했다는 말이다. 다시말해 기각역안에 들어왔다는 말이다. <br>
결론적으로 유의수준은 H0가 참인 상황이고, 데이터가 기각역 안에 들어있을 확률을 말한다. <br>

$$\theta = 2$$ 일 때 관측된 데이터 $$x_1, x_2$$ 가 $${x_1+x_2} \le 2\ln2$$ 에 들어있을 확률 <br>
$$\Pr_{\theta=2}({X_1+X_2} \le 2\ln2)$$ 를 구하면 된다. <br>









참고 : [7th Edition] Robert V. Hogg, Joeseph McKean, Allen T Craig - Introduction to Mathematical Statistics (2012, Pearson)