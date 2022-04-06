## [목차로](./readme.md)

# 해쉬 함수 및 메시지 인증

## 메시지 무결성
원본 메시지의 변조를 방지하기 위한 서비스

대칭키, 공개키 암호 시스템은 `비밀성`을 제공함

`무결성`을 위해 Modification Detection Code(MDC) 사용. 변조 감지 코드(MDC)는 해쉬함수로 생성함.

```
정보보호서비스에서
기밀성(비밀성), 무결성, 가용성
```

# 메시지 변조 감지 코드 MDC

메시지 m과 MDC h(m)은 분리될 수 있음. h(m)은 변조되지 않아야 함.

m은 일반 채널로 전송하는데 h(m)은 안전한 채널로 전송함

예시) 클라우드 서비스에서 원본데이터 무결성 제공

# 해쉬함수(hach function)
- 임의의 크기의 데이터를 입력 값 // 제한이 없다.
- 고정된 크기의 출력 값

하드웨어, 소프트웨어적으로 적용이 쉬워야하고 어떤 입력이 오더라도 출력값 계산하기 쉬워야함. 빨라야 함.

공개된 함수이고 키가 사용되지 않음(MDC에서는). MAC에서는 키 사용함

전자서명 생성하기 위해 사용 -> ch9 에서 더 다룸

## 암호학적으로 안전한 해쉬함수 성질
1. 역상 저항성
> given h, infeasible to find `x` s.t. H(x) = h
2. 제2역상 저항성
> given x, infeasible to find `y` s.t. H(y)=H(x)
3. 충돌 저항성
> infeasible to find any `(x, y)` s.t. H(y)=H(x)

해쉬함수로 이제 암호시스템을 설계할건데 그거로 만든게 과연 안전할까? 그 암호시스템의 수학적 성질을 제공하는 구현 가능한 함수를 모르는 경우에 사용함 랜덤 오라클이라는 개념을 사용함.

오라클이란 이론적인 추상머신임. 랜덤 오라클은 이상적인 해쉬함수임. 해쉬함수를 랜덤 오라클이라고 추상화해서 증명할 때 사용함. 랜덤 오라클은 임의의 입력에 대해 랜덤 값을 리턴해주는데 이전에 같은 입력값에 대해 답한 것이 있으면 그것으로 리턴해 줌.


## 생일문제
1. 역상 저항성
> n 비트인 H(x)가 주어진 경우 50%이상의 확률로 x를 찾는 문제<br>
> 평가해야하는 회수: 0.69 * n
2. 제2역상 저항성
>  n비트인 H(x)가 주어진 경우 50%이상의 확률로 H(x)=H(y)인 y를 찾는 문제<br>
> 0.69 * n + 1
3. 충돌 저항성
> 50%이상의 확률로 H(x)=H(y)인 쌍 (x,y)를 찾는 문제<br>
> 1.18 * sqrt(n)
4. 두 집합에서 같은 해쉬값을 찾을 수 있는가
> 서명공격. (두 집합에서 맞나?) m!=m'이면서 H(m)=H(m')인 쌍 (m, m')을 발견하여 공격<br>
> 0.83 * sqrt(n)

## SHA(Secure Hash Algorithm)

SHA-1은 해쉬값으로 160비트를 리턴함.

충돌 저항성의 측면에서 2^80정도의 역상(preimage), 또는 입력데이터가 있으면 충돌쌍을 찾을 수 있다는게 이론인데

만약 2^80보다 작은 데이터로 충돌쌍을 찾는게 가능하면 이 알고리즘은 깨진것 임.

그리고 깨졌음.

# 메시지 인증 코드 (MAC)
메시지 근원 인증을 제공함

MDC는 메시지의 출처를 알 수 없음.

MAC는 메시지의 출처와 데이터의 무결성도 함께 제공함.

MAC는 해쉬함수에 쓰이는 키를 사전에 공유하고 있음. 그래서 m과 H(m)은 안전하지 않은 채널로 전송해도 됨.

- MAC 생성 방법

해쉬함수 기반: Nested MAC, HMAC

블록암호 알고리즘 기반: CBC-MAC, CMAC

## Nested MAC
거의 사용하지 않음.

`key를 2`개 사용하고, 단순히 해쉬함수 2번 돌린 것임.

## HMAC (Hashed MAC)
`key 1`개. 메시지 m을 `b비트` 사이즈로 분할.

키를 `b비트`로 맞추기 위해 앞에 0으로 패딩함. 그것을 k'이라 할 것임.

ipad(input padding)하고 opad(output padding) 두 상수 값이 존재함.

1. (k' XOR ipad)||M 을 첫번째 해쉬함수에 넣고 `b비트`짜리 출력(I)을 받음
2. (k' XOR opad)||I 를 다시 해쉬함수에 넣고 그 출력이 HMAC 값임.

- 안전성

사용된 해쉬함수의 안전성에 기반함.

공격은 사용된 키 전수공격과 생일공격이 있는데 생일공격하려면 키가 2^n/2개 필요한데 MAC는 키가 사용되기때문에 자유롭게 키 수집이 어려움. 그래서 사실상 생일공격은 불가능.

## CBC-MAC (Cipher Block Chaining MAC)
`키 2개`. 블록암호 운영모드중에 CBC모드를 사용함.

IV=0이고 P로 들어갔던 것에 메시지들이 들어감. chain의 마지막 출력이 CBC-MAC값

- `가변길이의 메시지`에 대한 CBC-MAC를 생성하는 경우 **제2역상을 계산할 수 있음**

공격자가 (M, T), (M', T') 두개의 메시지, 해쉬쌍을 가지고 있다고 할 때(M=m1m2m3...mr, M'=m'1m'2m'3...,m'r),

M''= M||(m'1 XOR T)||m'2m'3...m'r 이라고 하면, T' = H(M'')인 것을 알 수 있음.

=> 그래서 `고정된길이의 메시지`만을 `사용`해야함

## CMAC (Cipher-based MAC)
고정된 길이의 메시지만을 사용해야 하는 CBC-MAC의 단점 해결 -> k2를 추가해서 가변길이 입력 사용 가능!

`2개의 키` 사용 k1, k2

`CBC-MAC하고 유사`한데 `모든 라운드 함수에는 k1`을 사용하고

마지막 라운드에 `input에` (이전함수의 값 XOR Pn `XOR k2`)를 사용함.


## 암호학적 해쉬함수 설계 원리
**반복 구조의 해쉬함수**

<img src="./8/iterated_hash.png">

1. n비트의 배수가 되도록 원본 메시지 m을 패딩한다
2. 패딩한 메시지를 한 블록이 n비트가 되도록 분할한다. m1, m2, m3, ..., mi
3. f(H0, m1) = H1
4. i번 수행해서 H 생성

위 구조는 SHA1, SHA2에서 쓰였고 MD구조임

SHA3은 스펀지 구조를 사용함










## [목차로](./readme.md)