> `이전 단계의 정보`와 `새로 들어온 측정값`을 적절한 비율로 섞어 최적의 상태 값을 추정하는 것

칼만 필터 시스템은 환경의 실제 상태를 완전히 알아낼 수 없다고 가정한다. 센서 등 측정 장비를 통해 현재 상태의 트렌드가 어떤지 간접적으로 파악할 수 있고, 센서 조차 노이즈가 있으므로 실제 환경을 정확히 투영할 수 없다.

## 변수 및 시스템 모델 정의

- **상태 변수 및 데이터**
	- $z_k$: $k$ 시간의 측정 값 (Sensor Measurement) 
	- $\hat{x}_k$: $k$ 시간의 추정 값 (State Estimate)
	- $P_k$: 오차 공분산 (Error Covariance), 추정 값의 불확실성(Uncertainty)
- **시스템 모델 매개변수**
	  - $A$: 상태 전이 행렬 (State Transition Matrix), 이전 상태에서 현재 상태로 변화하는 규칙을 기술
	  - $H$: 관측 행렬 (Obsevation Matrix), 상태 변수와 측정 값 사이의 관계를 기술
	  - $Q$: 시스템 노이즈 공분산 (Process Noise), 모델 자체의 노이즈(불확실성)
	  - $R$: 측정 노이즈 공분산 (Measurement Noise), 센서 자체의 노이즈(불확실성)

## 알고리즘 루프

칼만 필터는 크게 예측과 보정의 두 단계가 반복되는 피드백 루프다.

## 1. 예측 단계 (Prediction / Time Update)

직전 시간 ($k - 1$)의 정보를 바탕으로 현재 시간($k$)의 상태를 추정

**1-1. 상태 변수 예측 ($\hat{x}_k^-$):**
- 시스템 모델 중 상태 전이 행렬($A$)만 사용해서 다음 $k$ 시간의 상태를 예측함.

$$
\hat{x}_k^- = A\hat{x}_{k-1}
$$

**1-2. 예측 오차 공분산 계산 ($P_k^-$):**
- 시간이 흐름에 따라 불확실성($P$)이 시스템 노이즈 공분산($Q$)만큼 커진다.

$$
P_k^- = AP_{k-1}A^T + Q
$$

## 2. 보정 단계 (Correction / Measurement Update)
실제 측정 값($z_k$)을 받아서 예측 값을 수정하고 불확실성을 줄이는(보정하는) 단계

**2-1. 칼만 게인 계산 ($K_k$):**
- 예측 값의 불확실성($P^-$)과 센서의 불확실성($R$)을 비교

$$
K_k = P_k^-H^T \left(HP_k^-H^T + R\right)^{-1}
$$

**2-2. 추정 값 보정 ($\hat{x}_k$)**
- 예측 값 ($\hat{X}_k^-$)에 이노베이션 $\tilde{y}_k$(실제 측정 값($z_k$)과 예측 값의 차이)를 칼만 게인만큼 곱하고 더한다.
- 이노베이션(Innovation) 또는 잔차(Residual)라고 부른다.

$$
\tilde{y}_k = z_k - H\hat{x}_k^- \\
$$

$$
\hat{x}_k = \hat{x}_k^- + K_k\tilde{y}_k
$$

**2-3. 오차 공분산 보정 ($P_k$):
- 새로운 측정 정보를 반영했으므로, 추정 값의 불확실성($P$)을 낮출 수 있다.
- $(I - K_kH)$는 항상 1보다 작은 값을 가지므로, 측정 값($z_k$)를 반영하는 순간 불확실성($P$)는 무조건 줄어들게 된다.
- $k$ 시간 측정을 건너뛰면 $P$는 계속 커지게 된다.

$$
P_k = (I - K_kH)P_k^-
$$

## 3. 센서 스케쥴링 로직

**스케쥴링 지점 판단**:
- $T_{max}$: 최대 샘플링 주기
- $2\sqrt{P_k^-}$는 통계적으로 약 95%의 신뢰 수준을 의미한다.

$$
\begin{aligned}
&\text{IF } (2\sqrt{P_k^-} \le Threshold) \text{ AND } (\text{Age} \lt T_{max}) \text{ THEN} \\
&\quad \hat{x}_k = \hat{x}_k^- \\
&\quad P_k = P_k^- \\
&\text{ELSE} \\
&\quad \hat{x}_k = \hat{x}_k^- + K_k(z_k - H\hat{x}_k^-) \\
&\quad P_k = (I - K_kH)P_k^- \\
&\text{END IF}
\end{aligned}
$$

또는 아래와 같이 표현 가능

$$
\hat{x}_k, P_k = 
\begin{cases} 
\hat{x}_k^-, P_k^- & \text{if } (2\sqrt{P_k^-} \le \text{Threshold}) \text{ and } (\text{Age} \lt T_{max}) \\
\text{Update by } z_k & \text{otherwise}
\end{cases}
$$
