# 어셈블리프로그램 설계 및 실습

# 프로젝트 보고서

## Matrix Convolution

## 학 과: 컴퓨터정보공학부

## 담당교수: 이준환 교수님

## 강의 시간: 화 0, 1, 2, 목 2

## 학 번: 2018202046

## 성 명: 이준휘


## 1. Introduction

## A. Title

```
Matrix convolution
```
## B. Object

### 해당 프로젝트를 통해 현재까지 배운 어셈블리 언어를 활용하여 주어진 과제와 관련한

```
코드를 작성할 수 있다. 해당 Matrix의 Padding, Convolution, Sub-Sampling의 원리를 알
수 있고 이를 코드로 구현할 수 있다. 주어진 조건을 고려하여 state를 최적화하는 방향
으로 코드를 작성함으로써 state를 줄일 수 있는 다양한 방법을 연구한다.
```
## C. Schedule

### 11 / 21 ~11/22 1 1/23~11/24 1 1/2 5 ~11/ 26 1 1/27~11/

### 제안서 작성

### 프로젝트 수행

### 보고서 작성

## 2. Algorithm

## A. FloatingAdd

```
해당 함수는 양 또는 0 끼리의 덧셈을 진행하는 branch다. 이 때 두 피연산자의 값은
Float형으로 표기된다. 해당 함수에서는 r9, r10에 값을 넣으며 r8에 결과를 출력한다. 또
한 r6 ~ r12의 범위의 레지스터를 사용한다.
```
```
피연산자의 값이 0 일 경우는 0x80000000 또는 0 x00000000이다. 이를 판단하기 위해 두
피연산자의 sign bit를 LSL 1 을 통해 제거하여 해당 값이 0 인지를 비교한다. 만약 0 으로
판별될 경우 해당 덧셈의 결과는 0 이 아닌 피연산자의 값으로 반환되기 때문에 0 이 아닌
피연산자를 반환하고 프로그램을 종료한다.
```
```
0 이 아님을 확인한 후에는 LSL과 LSR을 통해 두 피연산자의 exponent랑 Mantissa를 분
리한다. 이 때 Mantissa는 값 손실이 일어났을 경우 이전 비트에서 반올림을 해주어야
하기 때문에 1 : 24 자리에 위치시켜준다. 그 후 mantissa 맨 앞에 생략되었던 1 을 붙어주
고 두 exponent를 비교한다. exponent가 작은 쪽의 Mantissa를 두 exponent의 차만큼
LSR을 통해 위치시켜 두 값을 연산할 수 있도록 도와준다. Mantissa는 단순히 더하기만
진행해도 되는데 이유는 두 값이 조건에서 양수라고 주어졌기 때문이다. 두 덧셈의 연산
```

### 결과가 26 비트 즉, 이전 자리를 기억하기 위한 비트까지 포함한 25 비트를 초과한 경우

LSL을 통해 25 비트로 만들어주면서 동시에 큰 쪽의 exponent의 값을 1 늘려준다.

이후에는 Mantissa의 반올림 과정과 float 값으로 재결합 과정이다. 0 번째 자리의 비트를
따로 때어낸 후 Mantissa를 우측으로 1 번 shift 해준다. 그 후 해당 수를 0 번째 자리를
때어낸 비트와 덧셈 연산을 통해 반올림을 진행해준다. 반올림이 끝나면 해당 값과
exponent, sign(0)을 결합하고 해당 함수를 종료한다.

## B. FloatingMul

### 해당 함수는 양의 자연수와 양의 정수끼리의 곱셈을 진행하는 함수다. 이 때 두 피연산

자의 값은 Float형으로 표기된다. 해당 함수에서는 r9, r10에 값을 넣는다. 이 때 r 9 의 값
에는 자연수를, r 10 자리에는 정수를 넣어주어야 한다. 그 후 연산 결과는 r8에 출력한다.
또한 r6 ~ r12의 범위의 레지스터를 사용한다.

해당 함수에서는 FloatingAdd와 마찬가지로 0 을 확인하는 과정이 있다. 만약 한 쪽의 피
연산자가 0 일 경우 해당 결과를 0 으로 출력하고 함수를 종료한다.

이후 덧셈과 동일하게 Mantissa와 exponent를 분리시켜주는데 Mantissa의 경우에는 자
연수는 상위 8 비트의 Mantissa만 남겨둔다. 이유는 이후의 비트가 0 이기 때문이다. 이후
에 Mantissa에 맨 앞에 1 을 붙어주는 작업을 수행한다.

Exponent끼리 연산은 두 값의 덧셈으로 이루어지는데 이 때 두 값에서 127 을 뺀 값을
더한 후 다시 127 을 더한다. 기본적으로 exponent가 지수에 127 을 더한 값이기 때문에
이러한 과정이 추가되었다.

해당 과정을 마친 후 exponent가 8 비트인 Mantissa에 이전 비트의 패턴인 0 을 뒤에 덧
붙어준다. 그 후 result를 저장할 r8을 0 값으로 초기화 시켜준 후 MulLoop로 넘어간다.

MulLoop에서는 실질적으로 4 - radix Booth algorithm이 구현되는 부분이다. 해당 부분에서
는 Muliplier(9비트)의 마지막 3 개의 비트의 패턴을 확인한다. 그리고 이를 0, 1~2, 3, 4,
5~6, 7의 경우에 따라 Multiplicand를 더해주거나 빼주는 것을 달리한다. 그 후 MulShift
과정으로 넘어간다.

MulShift에서는 result와 multiplier의 비트 조작과 탈출 조건이 명시된 부분이다. 우선


Multiplier를 LSR 2를 진행한다. 만약 해당 결과가 0 일 경우 result는 shift를 진행시키지
않고 아닐 경우에는 result를 ASR 2를 진행하고 MulLoop로 돌아가서 다시 연산을 수행한
다.

Mantissa 결과가 나온 경우 이를 맞추어주고 반올림하는 과정이 필요하다. 우선 연산 결
과로 나온 Mantissa가 26 비트보다 큰 지 확인한다. 만약 작을 경우 이전에 추가했던 맨
앞자리 1 을 제거하고 마지막 비트의 패턴을 따로 저장해둔다.
그 후 비트를 shfit하여 비트 수를 1 개 줄인다. 이후 만약에 아까 비교에서 크거나 같은
결과였을 경우 마지막 비트의 패턴을 지금 저장해두고, shift를 한 번 더 진행한다.
해당 값과 이전 비트의 패턴을 더하여 반올림을 진행하고 exponent와 Sign(0) 값과 함깨
결합시킨 후 함수를 종료한다.

## C. Matrix function

해당 함수의 경우 Padding, Convolution, Sub-Sampling을 동시에 진행한다. 우선 행의
row를 r2, col을 r3로 설정한 후 First_Line_First_Convolution으로 넘어간다.

## D. First_Line_First_Convolution

Padding된 Matrix에서 일어나는 Convolution이 다른 Convolution와 다르다. 특히 첫번째
Convolution은 중간에서 일어나는 Convolution과 다르기(Padding) 때문에 따로 제작된
Convolution이다. 해당 부분에서는 곱을 진행하는 Matrix는 다음과 같다.

A B C
D E F
G H I
Matrix_second

a a b
a a b
c c d
Matrix_first

해당 연산의 결과는 a * A + a * B + a * D + a * E + b * C + b * F + c * G + c * H + d * I
이며 이는 a(A + B + D + E) + b(C + F) + c(G + H) + d * I로 묶어서 연산할 수 있다.

해당 함수는 이 묶은 연산에 해당되며 r 4 에 각 단계의 중간 연산을 누적한다. 연산이 끝
나면 저장할 위치 sp에 이를 저장하고, Matrix_first의 읽는 위치를 4byte만큼 옮긴다. 그
후 col을 2 늘린다. 위치를 4byte, col을 2 늘리는 이유는 Sub_sampling을 한다면 다음 위


### 치가 아니라 그 다음의 위치에서 연산을 수행하기 때문이다. 이후에는

```
First_Line_Middle_Convolution을 진행한다.
```
## E. First_Line_Middle_Convolution

```
해당 행 또한 일반적인 cell과는 다른 연산이 진행되기 때문에(Padding) 따로 분리하여
연산을 시켜주었다.
해당 연산이 진행되는 Matrix는 다음과 같다.(채색된 cell은 현재 위치한 주소를 의미한다.)
```
```
A B C
D E F
G H I
Matrix_second
```
```
a b c
a b c
d e f
Matrix_first
해당 연산의 결과는 a * A + a * D + b * B + b * E + c * C + c * F + d * G + e * H + f * I
이며 이는 a(A + D) + b(B + E) + c(C + F) + d * G + e * H + f * I로 묶어서 연산할 수 있
다.
해당 함수 또한 위의 함수처럼 연산을 수행하고 이를 sp에 저장한다. 그 후 col을 62 와
비교하여 작을 경우에는(해당 줄의 Convolution이 전부 진행되지 않은 경우) col을 2, r0를
8byte 다음으로 위치시킨 후 다시 First_Line_Middle_Convolution을 진행한다. 만약 같을
경우(해당 줄에서 연산이 끝난 경우) 다다음줄의 첫 번째 위치로 이동해야하기 때문에
col값은 0 으로 초기화시켜주고 row 값은 2 늘려준 후 r0가 읽는 주소를 4 * 64 + 4 + 4,
즉 0x10 8 위치로 이동시켜준다. 그 후 Middle_Line_First_Convolution을 진행한다.
```
## F. Middle_Line_First_Convolution

모든 행의 첫 번째 Convolution은 Padding되어있기 때문에 일반적인 상황과는 다른 연산
이 진행된다. 때문에 이를 따로 분리하여 연산을 시켜주었다.

해당 연산이 진행되는 Matrix는 다음과 같다.(채색된 cell은 현재 위치한 주소를 의미한다.)


### A B C

### D E F

### G H I

Matrix_second

a a b
c c d
e e f
Matrix_first

```
해당 연산의 결과는 a * A + a * B + b * C + c * D + c * E + d * F + e * G + e * H + f * I
이며 이는 a(A + B) + c(D + E) + e(G + H) + b * C + d * F + f * I로 묶어서 연산할 수 있
다.
```
해당 연산을 수행한 결과는 sp에 저장되며 First_Line_first_Convolution과 마찬가지로 col
과 r 0 를 조작시킨 후 Middle_Line_Middle_Convolution을 진행시켜준다.

## G. Middle_Line_First_Convolution

해당 연산이 진행되는 Matrix는 다음과 같다.(채색된 cell은 현재 위치한 주소를 의미한다.)

A B C
D E F
G H I
Matrix_second

a b c
d e f
g h i
Matrix_first

```
해당 연산의 결과는 a * A + b * B + c * C + d * D + e * E + f * F + g * G + h * H + i * I
이며 묶을 수 없다.
```
해당 연산을 수행한 결과는 sp에 저장되며 First_Line_Middle_Convolution과 마찬가지로
col이 62 보다 작은지 확인하고 작을 경우 col을 늘리고 주소를 8 옮기는 과정을 수행한다.
만약 col이 62 일 경우 해당 row가 62 보다 작은지 확인한다. 만약 작을 경우 다다음 줄의
첫번째 위치로 이동할 수 있도록 row는 2 늘리고, col은 0 으로 초기화, 그리고 주소는
0x10 8 만큼 이동시켜준 후 Middle_Line_First_Convolution으로 이동한다. 만약 row 또한 62


### 일 경우 프로그램은 종료된다.

## 3. Performance & Result

```
A. FloatingAdd
```
```
해당 사진은 덧셈이 제대로 되는지 확인하는 사진이다 0x3D99999A와 0 x3DFDF3B8의 덧
셈 결과로 0x3E4BC6A8이 나오는 모습을 보인다.
```

### 덧셈의 결과가 정확히 나오는 것을 볼 수 있다.

```
B. FloatingMul
```

해당 결과를 보면 값이 들어가서 r 8 로 나오는 것을 확인할 수 있다. 해당 값이 맞는지 확
인해보면 아래와 같이 동일하게 결과가 나오는 것을 확인할 수 있다.

C. First_Line_First_Convolution


### 해당 결과를 확인해보면 정상적으로 연산 후 저장되는 것을 확인할 수 있다.

D. First_Line_Middle_Convolution


### 해당 값을 보면 다음 인자가 정상적으로 계산된 것을 볼 수 있다.


### 또한 한 줄이 전부 계산되는 모습을 볼 수 있다.

E. Middle_Line_First_Convolution


```
해당 결과를 확인하면 다다음 줄의 첫번째 Convolution이 정상적으로 된 모습을 볼
수 있다.
```
F. Middle_Line_Middle_Convolution


### 다음 인자가 정상적으로 계산된 것을 확인할 수 있다.


### 해당 값은 해당 줄이 전부 계산된 결과다. 정상적으로 출력되는 것을 확인할 수 있

### 다.


### 해당 결과를 보면 기존의 모든 데이터를 읽었고, 32*32*4개의 크기와 맞게 데이터가

### 저장되고 프로그램이 종료되는 모습을 볼 수 있다.

## 4. Conclusion

### 해당 프로젝트를 수행하면서 기본적으로 코드를 주어진 조건에 따라 최적화 시키기 위해

```
여러가지 방법을 사용했다. 우선 Padding, Convolution, Sub_Sampling 과정을 통합함으로
써 불필요한 Convolution을 줄이고 메모리에 결과로는 쓰이지 않은 값을 저장하는 것을
줄였다. 또한 Convolution 과정에서 식을 묶어서 연산을 진행함으로써 불필요한 곱셈이
나 덧셈의 연산 횟수가 많아지는 것을 방지하였다. 또한 Branch를 이동해야 할 상황을
```

되도록 피하도록 프로그램을 작성하고 비교를 할 경우 3 개 이하로 instruction이 들어가
도록 고려하였다. 마지막으로 곱셈의 연산 과정에서 현재 주어진 조건이 자연수와 정수
의 곱셈임을 고려하여 자연수의 패턴을 파악하고 자연수의 Mantissa를 줄여 Loop를 통
한 반복을 최대한 줄이는 것을 고려하였다.

위와 같은 과정을 진행하며 코드를 줄일 수 있는 다양한 방법을 고려하는 코드 작성에
대해 생각할 수 있었고, 알고리즘의 최적화에 대한 여러가지 방안 또한 생각하는 프로젝
트였다. 또한 Matrix를 조작하는 여러가지 방법을 알 수 있는 시간이었다.


