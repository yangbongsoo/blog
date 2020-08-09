# Multivariate Distributions

### EXERCISES 2.1.10
Let $$X_1$$ and $$X_2$$ have the joint pdf $$f(x_1, x_2) = 15x_1^2x_2$$,
$$0 < x_1 < x_2 < 1$$, zero elsewhere. Find the marginal pdfs and compute
$$P(X_1 + X_2 \le 1)$$. <br>

문제에서 joint pdf가 주어졌다. 이걸 가지고 marginal pdf 들을 구해보자.
먼저 $$x_2$$ 의 마지널을 구해보자. $$x_1$$ 을 적분해야 하고, $$x_2$$ 는 상수취급한다.
따라서 $$x_1$$ 의 적분 범위는 $$0 < x_1 < x_2$$ 이다. <br>

$$\int_{0}^{x_2} 15x_1^2x_2 \ dx_1$$ <br>
$$= 5x_2 \int_{0}^{x_2} 3x_1^2 \ dx_1$$ <br>
$$= 5x_2 \left[ x_1^3 \right]_{0}^{x_2} $$ <br>
$$= 5x_2^4 \quad support : 0< x_2 < 1$$ <br>

support 가 $$0< x_2 < 1$$ 이 되는 이유는 $$x_1$$ 에 대해 적분을 해서 다 구했으므로
$$0 < x_1 < x_2 < 1$$ 여기서 $$x_1$$ 을 빼도 된다. <br>

다음 $$x_1$$ 의 마지널을 구해보자. $$x_2$$ 를 적분해야 하고, $$x_1$$ 은 상수취급한다.
따라서 $$x_2$$ 의 적분 범위는  $$x_1 < x_2 < 1$$ 이다. <br>

$$\int_{x_1}^{1} 15x_1^2x_2 \ dx_2$$ <br>
$$= 15x_1^2 \int_{x_1}^{1} x_2 \ dx_2$$ <br>
$$= 15x_1^2 \left [ {1 \over 2}x_2^2 \right ]_{x_1}^{1}$$ <br>
$$= 15x_1^2({1\over2} - {1\over2}x_1^2) \quad support : 0 < x_1 < 1$$ <br>

다음  $$P(X_1 + X_2 \le 1)$$ 을 구해보자.
적분 구간을 정리해서 한번에 적분해 줘야 하는데 주어진 부등식($$X_1 + X_2 \le 1$$)과 각 확률변수들의 support($$0 < x_1 < x_2 < 1$$) 들을 적절히 조합해줘야 한다. <br>

$$0 < X_1 < X_2$$ <br>
$$X_2 \le 1-X_1$$ <br>

이걸 통해서 다음을 구할 수 있다.

$$X_1 < X_2 \le 1-X_1$$ <br>

그리고 이걸 통해서 $$X_1 < 1-X_1$$ 을 알 수 있고 $$X_1 < {1 \over 2}$$ 니까 <br>
$$0 < X_1 < {1 \over 2}$$ 이다. <br>

$$X_2$$ 를 먼저 적분하고 $$X_1$$ 을 적분해서 최대한 바깥쪽에 상수가 있게 한다.<br>

$$\int_{0}^{{1\over2}} \int_{x_1}^{{1-x_1}} 15x_1^2x_2 \ dx_2 \ dx_1$$ <br>
$$= \int_{0}^{{1\over2}} 15x_1^2 \int_{x_1}^{{1-x_1}} x_2 \ dx_2 \ dx_1$$ <br>
$$= \int_{0}^{{1\over2}} 15x_1^2 \{{1\over2}(1-x_1)^2 - {1\over2}x_1^2\} \ dx_1$$ <br>
$$= \int_{0}^{{1\over2}} 15x_1^2 ({1\over2} - x_1) \ dx_1$$ <br>
$$= \left[ {5 \over2}x_1^3 \right]_{0}^{1\over2} - \left[ 5x_1^4 \right]_{0}^{1\over2} = 0$$ <br>

### EXERCISES 2.1.13
Let $$X_1, X_2$$ be two random variables with joint pdf
$$f(x_1, x_2) = 4x_1x_2, \ 0 < x_1 < 1, \ 0 < x_2 < 1$$ zero elsewhere.
Compute $$E(X_1), E(X_1^2), E(X_2), E(X_2^2), E(X_1X_2)$$. Is $$E(X_1X_2) = E(X_1)E(X_2)$$?
Find $$E(3X_2 - 2X_1^2 + 6X_1X_2)$$. <br>

먼저 $$E(X_1)$$ 을 구해보자. <br>
$$\int_{0}^{1}x_1 f(x_1) \ dx_1 $$ 이다 <br>
내가 알고 있는건 joint pdf 이고 $$f(x_1)$$ 을 적분하려면 $$X_1$$ 의 마지널 pdf 를 구해야한다.
다시 말해 $$X_2$$ 를 적분하고 $$X_1$$ 을 상수취급한다. <br>

$$\int_{0}^{1} 4x_1x_2 \ dx_2 $$ <br>
$$= 4x_1 \int_{0}^{1} x_2 \ dx_2 $$ <br>



참고 : [7th Edition] Robert V. Hogg, Joeseph McKean, Allen T Craig - Introduction to Mathematical Statistics (2012, Pearson)
