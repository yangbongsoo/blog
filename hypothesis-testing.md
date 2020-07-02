# 가설 검정(Hypothesis-Testing)

### EXERCISES 4.5.5

Let $$X_1, X_2$$ be a random sample of size n = 2 from the distribution having <br>
pdf $$ f(x;\theta) = (1/\theta)e^{-x/\theta}, 0 < x < \infty$$ zero elsewhere. <br>
We reject $$H_0 : \theta = 2$$ and accept $$H_1 : \theta = 1$$ if the observed values of $$X_1, X_2$$, <br>
say $$x_1, x_2$$ are such that $${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$. <br>
Here $$\Omega = \{ \theta:\theta = 1,2 \}$$. <br>
Find the significance level of the test and the power of the test when $$$$ is false. <br>

먼저 문제에서 기각역을 알려줬다. $$x_1, x_2$$ 의 observed values 가 $${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$
를 만족할 때 H0를 reject 하고 H1을 accept 한다고 했다. <br>

$${f(x_1;2)f(x_2;2) \over f(x_1;1)f(x_2;1)} \le {1\over2} $$ <br>
$${{1\over2}e^{-x_1 \over 2}{1\over2}e^{-x_2 \over 2} \over e^{-x_1}e^{-x_2}} \le {1\over2}$$ <br>
$${e^{-x_1 \over 2}e^{-x_2 \over 2} \over e^{-x_1}e^{-x_2}} \le 2$$ <br>
$${e^{-x_1-x_2 \over 2}  \over e^{-x_1-x_2}} \le 2$$<br>
$${e^{{-x_1-x_2 \over 2} - ({-x_1-x_2})} } \le 2$$<br>
$${e^{{x_1+x_2 \over 2} }} \le 2$$<br>
$$\log_{e}{e^{{x_1+x_2 \over 2} }} \le \log_{e}2$$<br>
$${x_1+x_2 \over 2} \le \ln2$$<br>
$${x_1+x_2} \le 2\ln2$$<br>







참고 : [7th Edition] Robert V. Hogg, Joeseph McKean, Allen T Craig - Introduction to Mathematical Statistics (2012, Pearson)