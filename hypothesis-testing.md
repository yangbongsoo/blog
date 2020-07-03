# 가설 검정(Hypothesis-Testing)

### EXERCISES 4.5.5

Let $$X_1, X_2$$ be a random sample of size n = 2 from the distribution having <br>
pdf $$ f(x;\theta) = (1/\theta)e^{-x/\theta}, 0 < x < \infty$$ zero elsewhere. <br>
We reject $$H_0 : \theta = 2$$ and accept $$H_1 : \theta = 1$$ if the observed values of $$X_1, X_2$$, <br>
say $$x_1, x_2$$ are such that $${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$. <br>
Here $$\Omega = \{ \theta:\theta = 1,2 \}$$. <br>
Find the significance level of the test and the power of the test when $$H_0$$ is false. <br>

먼저 문제에서 기각역을 알려줬다. $$x_1, x_2$$ 의 observed values 가 $${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$
를 만족할 때 H0를 reject 하고 H1을 accept 한다고 했다. 기각역을 먼저 구하면 아래와 같다. <br>

$${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$ <br>
$${{1\over2}e^{-x_1 \over 2}{1\over2}e^{-x_2 \over 2} \over e^{-x_1}e^{-x_2}} \le {1\over2}$$ <br>
$${e^{-x_1 \over 2}e^{-x_2 \over 2} \over e^{-x_1}e^{-x_2}} \le 2$$ <br>
$${e^{-x_1-x_2 \over 2}  \over e^{-x_1-x_2}} \le 2$$<br>
$${e^{{-x_1-x_2 \over 2} - ({-x_1-x_2})} } \le 2$$<br>
$${e^{{x_1+x_2 \over 2} }} \le 2$$<br>
$$\log_{e}{e^{{x_1+x_2 \over 2} }} \le \log_{e}2$$<br>
$${x_1+x_2 \over 2} \le \ln2$$<br>
$${x_1+x_2} \le 2\ln2$$(기각역)<br>

**유의수준을 구해라**<br>
유의수준(significance level)은 Type1 Error 의 최대 허용범위다. Type1 Error는 H1을 선택했는데 H0가 참인 경우다. <br>
Type1 Error 는 H1을 선택했고 H0가 참일 때다. <br>
H1을 선택했다는 말은 H0를 기각했다는 말이다. 다시말해 기각역안에 들어왔다는 말이다. <br>
결론적으로 유의수준은 H0가 참인 상황이고, 데이터가 기각역 안에 들어있을 확률을 말한다. <br>

$$\theta = 2$$ 일 때 관측된 데이터 $$x_1, x_2$$ 가 $${x_1+x_2} \le 2\ln2$$ 에 들어있을 확률 <br>
$$\Pr_{\theta=2}({X_1+X_2} \le 2\ln2)$$ 를 구해라. <br>


**power를 구해라**<br>
power(검정역) 는 1 - Type2 Error 이다. Type2 Error 는 H0를 선택했는데 H1이 참인 경우다. <br>
power 가 클수록 Type2 Error 는 작아지니까 올바른 의사결정을 내릴 확률이 커진다. <br>
power = $$1 - \Pr_{H_1}(H_0 accept)$$ <br>
power = $$\Pr_{H_1}(H_0 reject)$$ <br>
즉, H1이 참인 상황이고, 데이터도 기각역 안에 들어있을 확률을 말한다. <br>

$$\theta = 1$$ 일 때 관측된 데이터 $$x_1, x_2$$ 가 $${x_1+x_2} \le 2\ln2$$ 에 들어있을 확률 <br>
$$\Pr_{\theta=1}({X_1+X_2} \le 2\ln2)$$ 를 구해라. <br>


참고 : [7th Edition] Robert V. Hogg, Joeseph McKean, Allen T Craig - Introduction to Mathematical Statistics (2012, Pearson)