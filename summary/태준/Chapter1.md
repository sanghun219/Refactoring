<br>

# Chapter1 리팩터링: 첫 번째 예시

<br>

---

<br>

다양한 연극을 외주로 받아서 공연하는 극단이 공연 요청이 들어오면 연극의 장르와 관객 규모를 기초로 비용을 책정한다. 현재 이 극단은 두 가지 장르, 비극과 희극만 공연한다. 그리고 공연료와 별개로 포인트를 지급해서 다음번 의뢰 시 공연료를 할인받을 수도 있다.

다음은 공연료 청구서를 출력하는 코드이다.

<br>

 ```javascript
  function statement (invoice, plays){
  	let totalAmount = 0;
  	let volumeCredits = 0;
  	let result = `청구 내역 (고객명: ${invoice.customer})\n`;
      const format = new Intl.NumberFormat("en-US",
                                          {style: "currency", currency: "USD", minimumFractionDigits: 2}).format;
      
      for (let perf of invoice.performances) {
          const play = plays[perf.playID];
          let thisAmount = 0;
          
          switch (play.type) {
              case "tragedy" : // 비극
                  thisAmount = 40000;
                  if (perf.audience > 30) {
                      thisAmount += 1000 * (perf.audience - 30);
                  }
                  break;
              case "comedy": // 희극
                  thisAmount = 30000;
                  if (perf.audience > 20) {
                      this.Amount += 10000 + 500 * (perf.audience -20);
                  }
                  thisAmount += 300 * perf.audience;
                  break;
              default:
                  throw new Error(`알수 없는 장르: ${play.type}`);
          }
      }
      
      volumeCredits += Math.max(perf.audience -30, 0);
      if("comedy" === play.type) volumeCredits += Math.floor(perf.audience /5);
      
      result += ` ${play.name}: ${format(thisAmount/100)} (${perf.audience}석)\n`;
      totalAmount += thisAmount;
  }
  	result += `총액: ${format(totalAmount/100)}\n`;
  	result += `적립 포인트: ${volumeCredits}점\n`;
  	return result;
  }
  ```

<br>

프로그램이 잘 작동하는 상황에서 그저 코드가 '지저분하다'는 이유로 불평하는 것은 프로그램의 구조를 너무 미적인 기준으로만 판단하는 것은 아닐까? 컴파일러는 코드가 깔끔하든 지저분하든 개의치 않으니 말이다. 하지만 그 코드를 수정하려면 사람이 개입되고, 사람은 코드의 미적 상태에 민감하다. **설계가 나쁜 시스템은 수정하기 어렵다.**

**프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면, 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.**

<br>

### 리팩터링의 첫 단계

- 리팩터링할 코드 영역을 꼼꼼하게 검사해줄 테스트 코드의 마련
  - 실수를 최소화 시켜주며 디버깅 시간을 줄여준다.

<br>

### 함수 쪼개기

- statement()처럼 긴 함수를 리팩터링할 떄는 먼저 전체 동작을 각각의 부분으로 나눌 수 있는 지점을 찾는다.
  - switch문
  - 함수 추출하기 (6.1절)
- 수정 후에는 곧바로 컴파일하고 테스트해보기
  - 한 가지를 수정할 때마다 테스트하면, 오류가 생기더라도 변경 폭이 작기 때문에 살펴볼 범위도 좁아서 문제를 찾고 해결하기가 쉽다.
    - 리팩터링 절차의 핵심

**리팩터링은 프로그램 수정을 작은 단계로 나눠 진행한다. 그래서 중간에 실수하더라도 버그를 쉽게 찾을 수 있다.**

- 함수 추출 후에는 추출된 함수 코드를 자세히 들여다보면서 지금보다 명확하게 표한할 방법이 있는지 확인한다.
  - 변수 이름

**컴퓨터가 이해하는 코드는 바보도 작성할 수 있다. 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자다.**

- 임시 변수를 질의 함수로 바꾸기 (7.4절)
- 변수 인라인하기 (6.4절)
- 반복문 쪼개기 (8.7절)
- 문장 슬라이드하기 (8.6절)

<br>

### 마치며

이번 장에서는 리펙터링을 크게 세 단계로 진행했다. 
1. 원본 함수를 중첩 함수 여러 개로 나눔
2. 단계 쪼개기 (6.11절)을 적용해서 계산 코드와 출력 코드를 분리
3. 계산 로직을 다형성으로 표현

리팩터링은 대부분 코드가 하는 일을 파악하는 데서 시작한다.

**졸은 코드를 가늠하는 확실한 방법은 '얼마나 수정하기 쉬운가'다.**

