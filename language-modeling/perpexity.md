# 언어모델의 평가 방법

좋은 언어모델이란 실제 우리가 쓰는 언어에 대해서 최대한 비슷하게 확률 분포를 근사하는 모델(또는 파라미터, $\theta$)이 될 것입니다. 많이 쓰이는 문장 또는 표현일수록 높은 확률을 가져야 하며, 적게 쓰이는 문장 또는 이상한 문장 또는 표현일수록 확률은 낮아야 합니다. 즉, 우리가 (공들여) 준비한 테스트 문장을 잘 예측 해 낼 수록 좋은 언어모델이라고 할 수 있습니다. 문장을 잘 예측 한다는 것은, 다르게 말하면 주어진 테스트 문장이 언어모델에서 높은 확률을 가진다고 할 수 있습니다. 또는 문장의 앞부분이 주어지고, 다음에 나타나는 단어의 확률 분포가 실제 테스트 문장의 다음 단어에 대해 높은 확률을 갖는다면 더 좋은 언어모델이라고 할 수 있을 것 입니다. 이번 섹션은 언어모델의 성능을 평가하는 방법에 대해서 다루도록 하겠습니다.

## Perplexity

Perplexity(퍼플렉시티, PPL) 측정 방법은 정량 평가(explicit evaluation) 방법의 하나입니다. PPL을 이용하여 언어모델 상에서 테스트 문장들의 점수를 구하고, 이를 기반으로 언어모델의 성능을 측정합니다. PPL은 문장의 확률에 길이에 대해서 normalization한 값이라고 볼 수 있습니다.

$$
\begin{aligned}
\text{PPL}(w_1,w_2,\cdots,w_n)=&P(w_1,w_2,\cdots,w_n)^{-\frac{1}{n}} \\
=&\sqrt[n]{\frac{1}{P(w_1,w_2,\cdots,w_n)}}
\end{aligned}
$$

문장이 길어지게 되면 문장의 확률은 굉장히 작아지게 됩니다. 이는 체인룰에 따라서 조건부 확률들의 곱으로 바꿔 표현하여 보면 알 수 있습니다. 따라서 우리는 문장의 길이 n으로 제곱근을 취해 기하평균을 구하고, 문장 길이에 대해서 normalization을 해 주는 것을 볼 수 있습니다. 문장의 확률이 분모에 들어가 있기 때문에, 확률이 높을수록 PPL은 작아지게 됩니다.

따라서 테스트 문장에 대해서 확률을 높게 예측할 수록 좋은 언어모델인 만큼, 해당 테스트 문장에 대한 PPL이 작을 수록 좋은 언어모델이라고 할 수 있습니다. 즉, PPL은 수치가 낮을수록 좋습니다. 또한, n-gram의 n이 클 수록 보통 더 낮은 PPL을 보여주기도 합니다.

위 PPL의 수식은 다시한번 체인룰에 의해서

$$
=\sqrt[n]{\frac{1}{\prod_{i=1}^{n}{P(w_i|w_1,\cdots,w_{i-1})}}}
$$

라고 표현 될 수 있고, 여기에 n-gram이 적용 될 경우,

$$
\approx\sqrt[n]{\frac{1}{\prod_{i=1}^{n}{P(w_i|w_{i-n+1},\cdots,w_{i-1})}}}
$$

로 표현 될 수 있습니다.

### Perplexity의 해석

![주사위 두 개](../assets/lm_rolling_dice.png)

<stop>

Perplexity의 개념을 좀 더 짚고 넘어가도록 해 보겠습니다. 이것은 Perplexity의 수치를 해석하는 방법에 대해서라고 할 수 있습니다. 예를 들어 우리가 6면 주사위를 던져서 나오는 값을 통해 수열을 만들어낸다고 해 보겠습니다. 따라서 1부터 6까지 숫자의 출현 확률은 모두 같다(uniform distribution)고 가정하겠습니다. 그럼 $N$번 주사위를 던져 얻어내는 수열에 대한 perplexity는 아래와 같습니다.

$$
PPL(x)=({\frac{1}{6}}^{N})^{-\frac{1}{N}}=6
$$

매 time-step 가능한 경우의 수인 6이 PPL로 나왔습니다. 즉, PPL은 우리가 뻗어나갈 수 있는 branch(가지)의 숫자를 의미하기도 합니다. 다른 예를 들어 만약 $20,000$개의 어휘로 이루어진 뉴스 기사에 대해서 PPL을 측정한다고 하였을 때, 단어의 출현 확률이 모두 동일하다면 PPL은 $20,000$이 될 것입니다. 하지만 3-gram을 사용한 언어모델을 만들어 측정한 PPL이 30이 나왔다면, 우리는 이 언어모델을 통해 해당 신문기사에서 매번 기사의 앞 부분을 통해 다음 단어를 예측 할 때, 평균적으로 30개의 후보 단어 중에서 선택할 수 있다는 얘기가 됩니다. 따라서 우리는 perplexity를 통해서 언어모델의 성능을 단순히 측정할 뿐만 아니라 실제 어느정도인지 가늠 해 볼 수도 있습니다.

### Entropy와 Perplexity의 관계

엔트로피(Entropy)는 물리학에서 중요하게 사용되는 개념이지만, 정보이론(Information Theory)에서도 굉장히 중요하게 사용 됩니다. 정보이론에서 엔트로피는 어떤 정보의 불확실성을 나타내는 수치로 사용 될 수 있습니다. 정보량과 불확실성은 어떤 관계를 가질까요? 불확실성은 일어날 것 같은 사건(likely event)의 확률로 정의할 수 있습니다. 따라서 불확실성과 정보량은 아래와 같은 관계를 가집니다.

- 자주 발생하는(일어날 확률이 높은) 사건은 낮은 정보량을 가진다.
- 반대로 드물게 발생하는(일어날 확률이 낮은) 사건은 높은 정보량을 가진다.

위의 예를 실제 사례를 들어 적용 해 보겠습니다.

|번호|문장|
|-|-|
|1|내일 아침에는 해가 동쪽에서 뜬다.|
|2|내일 아침에는 해가 서쪽에서 뜬다.|
|3|대한민국 올 여름의 평균 기온은 섭씨 28도로 예상 된다.|
|4|대한민국 올 여름의 평균 기온은 섭씨 5도로 예상 된다.|

누군가에게 위와 같은 말을 들었을 때, 어떤 말이 우리에게 큰 도움이 될까요? 몇십억년 동안 반복되온 아침 해가 뜨는 위치가 내일은 바뀐다는 정보는 정말 천지 개벽의 정보가 될 겁니다. 따라서 우리는 이처럼 확률이 낮을 수록 그 안에 포함된 정보량은 높다고 생각 할 수 있습니다.

섀넌(Claude Elwood Shannon)은 위와 같이 정보 엔트로피라는 개념과 함께 아래의 수식을 제시하였습니다. 확률 변수(Random Variable) $X$의 값이 $x$인 경우의 정보량은 아래와 같이 표현 할 수 있습니다.

$$
I(x) = -\log{P(x)}
$$

$-\log{}$를 취하였기 때문에, 0과 1 사이로 표현되는 확률은 0에 가까워질수록 지수적으로 높은 정보량을 가짐을 알 수 있습니다. 언어모델 관점에서 적용시켜 보면, 흔히 나올 수 없는 문장(확률이 낮은 문장)일수록 더 높은 정보량을 가질 것이라고 생각 할 수 있습니다.

우리는 이러한 정보량의 기대값을 엔트로피(entropy)라고 부릅니다.

$$
H(P) = -\mathbb{E}_{X \sim P}[\log{P(x)}] = -\sum_{\forall x}{P(x)\log{P(x)}}
$$

위 식을 해석 해 보면, 우리는 확률 분포 $P$로부터 발생한 사건 $X$의 정보량에 대한 기대값을 구하는 것이라고 할 수 있습니다.

### Cross Entropy Loss

크로스 엔트로피(Cross entropy)는 entropy로부터 한 걸음 더 나아가, 우리가 구하고자 하는 ground-truth 확률분포 $P$를 통해 우리가 학습중인 확률분포 $Q$의 정보량의 기대값을 이릅니다. 크로스 엔트로피의 수식은 아래와 같습니다.

$$
H(P,Q)=-\mathbb{E}_{X \sim P}[\log{Q(x)}]=-{\sum_{\forall x}{P(x)\log{Q(x)}}}
$$

이것을 $Q$ 대신, 우리의 모델($P_\theta$)을 최적화 하기 위해 최소화(minimize)해야 하는 loss(손실)함수로 적용하여 보면 아래와 같습니다.

$$
\mathcal{L}=H(P,P_\theta)=-\mathbb{E}_{Y|X \sim P}[\log{P_\theta(y|x)}]
$$

여기서 우리는 N번의 Sampling 하는 과정을 통해 expectation을 제거할 수 있습니다. <comment> 강화학습(Reinforcement Learning) 챕터 참고 </comment>

$$
\begin{aligned}
H(P, P_\theta)&=-\mathbb{E}_{Y|X \sim P}[\log{P_\theta(w_i|w_{<i})}] \\ 
&\approx -\frac{1}{N}\sum_{i=1}^{N}{\log{P_\theta(w_i|w_{<i})}}
\end{aligned}
$$

여기서 $Y=\{y_1,y_2,\cdots,y_N\}$는 $N$개의 단어로 이루어진 문장(word sequence)로 생각하고, 한 문장에 대한 cross entropy는 아래와 같이 표현할 수 있습니다.

$$
\begin{aligned}
\mathcal{L}&=-\frac{1}{N}\sum_{i=1}^{N}{\log{P_\theta(w_i|w_{<i})}} \\
&=\log{\Big(\prod_{i=1}^{N}{P_\theta(w_i|w_{<i})}\Big)^{-\frac{1}{N}}} \\
&=\log{\sqrt[N]{\frac{1}{\prod_{i=1}^{N}{P_\theta(w_i|w_{<i})}}}} \\
\end{aligned}
$$

여기에 PPL 수식을 다시 떠올려 보겠습니다.

$$
\begin{gathered}
\text{PPL}(W)=P(w_1, w_2, \cdots, w_N)^{-\frac{1}{N}}=\sqrt[N]{\frac{1}{P(w_1,w_2,\cdots,w_N)}} \\
\text{by chain rule},\\
\text{PPL}(W)=\sqrt[N]{\prod_{i=1}^{N}{\frac{1}{P(w_i|w_1,\cdots,w_{i-1})}}}
\end{gathered}
$$

앞서 정리했던 Cross Entropy와 수식이 비슷한 형태임을 알 수 있습니다. 따라서 PPL과 Cross Entropy의 관계는 아래와 같습니다.

$$
\text{PPL}=\exp(\text{Cross Entropy})
$$

따라서, 우리는 Maximum Likelihood Estimation(MLE)을 통해 parameter($\theta$)를 배울 때, cross entropy를 통해 얻은 ($P_\theta$의 로그 확률 값) loss 값에 $\exp$를 취함으로써, perplexity를 얻어 언어모델의 성능을 나타낼 수 있습니다.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU2MDkxOTY2NywyMTAzODM5MzgxXX0=
-->