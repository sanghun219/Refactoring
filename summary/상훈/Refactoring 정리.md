## Chapter 01

- Chapter 01 예제 코드

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

- **코드를 처음 본 소감**
  - 매직 넘버가 너무 많다.
  - 계산식이 마구 포함되어 코드 읽기가 수월하지 않다.
  - switch-case 문을 hash-map 형태로 바꿀 수 없는가.. (case는 계속 추가될 수 있기에)
  - 중복 코드를 함수로 변환할 수 있을듯 하다.

- **프로그램의 구조가 빈약한 경우**

  - 프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면, 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.

- **리팩터링의 첫 단계**

  - 리팩터링할 코드 영역을 꼼꼼하게 검사해줄 **테스트 코드**들부터 마련해야 한다.
    - 성공/실패를 스스로 판단하는 자가진단 테스트를 만든다.
    - 테스트 코드를 만드는게 번거롭지만, 디버깅 시간을 줄여주기에 전체 개발 시간을 줄여준다.
    - **<Chapter 4> 에서 다룰 예정**

- **함수 쪼개기**

  - 변경전

  ```javascript
  // 함수 statement 일부 -> 함수 추출하기 기법으로 빼낸다. 
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
  ```

  - 변경후

  ```javascript
  // 1단계 : 함수 추출하기로 코드 블록을 함수로 치환
  function amountFor(perf, play) {
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
      return thisAmount;
      }
  }
  
  // 2단계 : 변수의 이름을 명확하게 변환
  function amountFor(aPerformance, play) { // perf => aPerformance로 변환
      let result = 0; // thisAmount => result로 변환됨
      switch (play.type) {
              case "tragedy" : // 비극
                  result = 40000;
                  if (perf.audience > 30) {
                      result += 1000 * (perf.audience - 30);
                  }
                  break;
              case "comedy": // 희극
                  result = 30000;
                  if (perf.audience > 20) {
                      this.Amount += 10000 + 500 * (perf.audience -20);
                  }
                  result += 300 * perf.audience;
                  break;
              default:
                  throw new Error(`알수 없는 장르: ${play.type}`);
          }
      return result;
      }
  }
  
  // 3단계 : 필요 없는 매개변수 제거하기
  // statement() 함수..
  function playFor(aPerformance) {
      return plays[aPerformance.playID];
  }
  // 최상위
  function statement(invoice, plays) {
  	let totalAmount = 0;
  	let volumeCredits = 0;
  	let result = `청구 내역 (고객명: ${invoice.customer})\n`;
      const format = new Intl.NumberFormat("en-US",
                                          {style: "currency", currency: "USD", minimumFractionDigits: 2}).format;
      
      for (let perf of invoice.performances) {
          const play = playFor(perf); // 변경된 부분
          let thisAmount = 0;
           
          volumeCredits += Math.max(perf.audience -30, 0);
      if("comedy" === play.type) volumeCredits += Math.floor(perf.audience /5);
      
      result += ` ${play.name}: ${format(thisAmount/100)} (${perf.audience}석)\n`;
      totalAmount += thisAmount;
  }
  	result += `총액: ${format(totalAmount/100)}\n`;
  	result += `적립 포인트: ${volumeCredits}점\n`;
  	return result;
  }
      }
  
  // 4단계 : 변수 인라인하기
  let totalAmount = 0;
  	let volumeCredits = 0;
  	let result = `청구 내역 (고객명: ${invoice.customer})\n`;
      const format = new Intl.NumberFormat("en-US",
                                          {style: "currency", currency: "USD", minimumFractionDigits: 2}).format;
      
  for (let perf of invoice.performances) {
      let thisAmount = amountFor(perf, playFor(perf)); // 변경된 부분
         
      volumeCredits += Math.max(perf.audience -30, 0);
      if("comedy" === playFor(perf).type)
          volumeCredits += Math.floor(perf.audience /5);
      
      result += ` ${playFor(perf).name}: ${format(thisAmount/100)} (${perf.audience}석)\n`;
      totalAmount += thisAmount;
  }
  	result += `총액: ${format(totalAmount/100)}\n`;
  	result += `적립 포인트: ${volumeCredits}점\n`;
  	return result;
  }
  
  // 5단계 : 함수 선언 바꾸기로 명확한 의미 전달
  function amountFor(aPerformance) { // play 제거
      let result = 0; 
      switch (playFor(aPerformance).type) {
              case "tragedy" : // 비극
                  result = 40000;
                  if (aPerformance.audience > 30) {
                      result += 1000 * (aPerformance.audience - 30);
                  }
                  break;
              case "comedy": // 희극
                  result = 30000;
                  if (aPerformance.audience > 20) {
                      this.Amount += 10000 + 500 * (aPerformance.audience -20);
                  }
                  result += 300 * aPerformance.audience;
                  break;
              default:
                  throw new Error(`알수 없는 장르: ${playFor(aPerformance).type}`);
          }
      return result;
      }
  }
  
  // 이후 나머지 작업도 대부분 위에 사용된 기법으로 해결해나간다. (너무 길어서 생략)
  ```

  - 함수 추출하기
    - code block을 의미있는 이름 (Code의 목적)의 함수로 분리 (chapter 6에서 다룸)
  - 변수 인라인하기
    - 굳이 필요 없는 변수 선언을 제거하고 해당 변수에 들어갈 연산을 그대로 이용한다. (chapter 6에서 다룸)
  - 함수 선언 바꾸기
    - 함수 추출하기 이후 명확한 이름으로 변경 (chapter 6에서 다룸)
  - 문장 슬라이드하기
    - 비슷한 목적의 코드나 서로 관계가 있는 코드는 가까운 곳에 위치시킨다. (chapter 8 에서 다룸)

- 성능 관련 이슈
  
- 성능 문제와 리팩터링은 전혀 다른 문제다. 리팩터링 할 때는 성능 문제를 고려하지 않고, 이후에 문제가 생기면 성능 문제를 해결한다.
  
- Chapter1에서 다루려고 했던 것들

  - 절차
    - 함수를 중첩 함수 여러개로 나누기
    - 단계 쪼개기를 이용하여 계산 코드와 출력 코드 분리
    - 계산 로직을 다형성으로 바꾸기

  - 다루지 못한 이유
    - 코드 위주의 설명인데 생각보다 부피가 커서 요약하더라도 읽기 용이하지 않다고 판단
    - Chapter1은 맛보기로 이후 Chapter에서 크게 다루기에 해당 장에서 자세하게 요약할 것



## Chapter 02

- **리팩터링 정의**
  - 소프트웨어의 **겉보기 동작**은 그대로 유지한 채, 코드를 이해하고 수정하기 쉽도록 내부 구조를 변경하는 기법
- **겉보기 동작**
  - 리팩터링하기 전과 후의 코드가 똑같이 동작해야 함
  - 리팩터링 전에 발견된 버그는 리팩터링 후에도 그대로 남아있어야 함
- **두 개의 모자**
  - 기능 추가
    - 기능을 추가할 때는 기존 코드는 건드리지 않고 새 기능만을 추가
  - 리팩터링
    - 기능 추가는 절대 하지 않고 코드 재구성에만 전념
- **리팩터링하는 이유**
  - 소프트웨어 설계가 좋아진다
    - 단기 목표만을 위해 코드를 수정하다 보면 기반 구조가 무너지기 쉽다
      - 코드만 봐서는 설계를 파악하기 어려움
      - 같은 일을 하더라도 코드가 더 길어짐
    - 중복 코드 제거를 통하여 모든 코드가 언제나 고유한 일을 수행함을 보장할 수 있다
  - 소프트웨어를 이해하기 쉬워진다
    - 컴퓨터에게 시키려는 일과 이를 표현한 코드의 차이를 최대한 줄여야 한다
    - 본인을 포함하여 다른 개발자들을 위함이다
  - 리팩터링하면 버그를 쉽게 찾을 수 있다
    - 코드가 하는 일을 깊이 파악하게 되면서 버그를 쉬이 파악할 수 있다
  - 리팩터링하면 프로그래밍 속도를 높일 수 있다
    - 오히려 리팩터링 하는 시간이 더 든다고 생각할 수 있지만, 잘 리팩터링된 코드는 추후에 추가되는 코드들이 잘 녹아들어갈 수 있어 개발 속도를 증진시킨다
- **리팩터링을 언제 해야하는가**
  - 3의 법칙
    - 처음에는 그냥 한다
    - 비슷한 일을 두 번째로 하게 되도 일단 계속 진행한다
    - **비슷한 일을 세 번째 하게 되면 리팩터링한다**
  - 코드 베이스에 새 기능을 추가할 때
    - 중복 코드를 사전에 방지하여 추후 코드 수정을 피할 수 있다
  - 코드를 파악해야 할 때
    - 특정 코드를 파악할 때 리팩터링 할 부분이 있는지 찾아본다
    - 코드 분석시 리팩터링을 하면, 더 깊은 수준까지 이해하게 된다
    - 하려던 일과 관련이 없을 경우, 쉽게 처리할 수 있으면 바로 처리하고, 없다면 메모 후 일을 끝마치고 리팩터링을 한다
- **리팩터링을 하지 말아야 할 때**
  - 외부 API 다루듯 호출해서 쓰는 코드라면 지저분해도 그냥 둔다
    - 내부 동작을 이해해야 할 시점에 리팩터링해야 효과를 제대로 볼 수 있다
- **리팩터링 시 고려할 문제**
  - 리팩터링은 '클린 코드'의 목적 보다는 경제적인 이유를 목적으로 둔다
  - 코드 소유권이 남에게 있는 경우 기존 API를 유지한 채로 내부 코드를 수정한다
  - 브랜치를 사용할 경우 마스터 브랜치와의 통합 주기를 짧게 가진다
  - 리팩터링에 따른 자가 진단 테스트를 마련해야 한다
  - 테스트 시스템이 없는 레거시 코드는 진단 테스트를 추가하며 리팩토링 한다
  - 데이터베이스 리팩터링은 여러 단계로 나누어 진행한다
    - 필드 이름을 바꿀 때 첫 번째 커밋에서는 새로운 데이터베이스 필드를 추가만 하고 사용하지 않는다
    - 기존 필드와 새 필드를 동시에 업데이트하도록 설정한다
    - 데이터베이스를 읽는 클라이언트들을 새 필드를 사용하는 버전으로 조금씩 교체한다

- **YAGNI (you aren't going to need it)**
  - 당장에 필요없는 기능을 확장 유연성을 위해서 추가하지 말자
  - 현재 요구사항만을 완벽히 해내고 추후에 리팩토링 하는 기법
  - 테스트 주도 개발 (자가 테스트 코드 + 지속적 통합)과 리팩터링을 통해 YAGNI 설계 방식으로 개발을 진행할 수 있다
- **리팩터링과 성능**
  - 리팩터링은 소프트웨어 속도 저하의 원인이 될 수도 있지만, 그와 동시에 성능을 튜닝하기는 더 쉬워진다
  - 하드 리얼타임 시스템을 제외하고.. (게임은 여기에 속할듯)
  - 섣불리 추측 하지 말고 성능을 측정해보자

## Chapter 03

- **기이한 이름**
  - 코드는 단순하고 명료하게 작성해야 한다
  - 이름만 잘 지어도 나중에 문맥을 파악하느라 헤매는 시간을 크게 절약할 수 있다
- **중복 코드**
  - 한 클래스에 딸린 두 메서드가 똑같은 표현식을 사용하는 경우
    - 함수 추출하기 사용
  - 코드가 비슷하긴 한데 완전히 똑같지 않은 경우
    - 문장 슬라이드 이후 함수 추출하기 사용
  - 같은 부모로부터 파생된 서브 클래스들에 코드가 중복되어 있는 경우
    - 메서드 올리기 적용 (chapter 12.1)
- **긴 함수**
  - 짧은 코드는 끝없이 위임하는 방식으로 작성되어 연산하는 부분이 없어 보인다
  - 예전 언어는 서브루틴 호출 비용때문에 짧은 함수를 꺼렸지만, 요즘 언어는 호출 비용이 거의 없다.
  - 짧은 함수는  depth가 깊어지나, 좋은 IDE를 사용하면 쉽게 서브루틴을 순회하며 코드 파악이 가능하다.
  - 함수를 잘게 쪼개고 좋은 이름을 부여해야 한다
    - 주석을 달아야 하는 부분은 함수로 만든다
    - 함수 이름은 동작 방식이 아닌 의도가 드러나게 짓는다
    - 원래 코드보다 길어지더라도 함수로 뽑는다
  - 함수가 매개변수와 임시 변수를 많이 사용하면 추출 작업에 방해가 된다
    - 임시 변수를 질의 함수로 바꾸기(chapter 7.4) 이용
    - 매개변수 객체 만들기(chapter 6.8) 이용
    - 객체 통째로 넘기기(chapter 11.4) 이용
    - 함수를 명령으로 바꾸기(chapter 11.9) 이용
- **긴 매개변수 목록**
  - 매개변수를 많이 넣는건 전역 변수 사용을 방지하는 이유였지만, 오히려 함수 해석을 어렵게한다
    - 클래스로 변수를 묶어 사용하는게 효율적이다
- **전역 데이터**
  - 전역 변수, 클래스 변수, 싱글톤은 변수 캡슐화하기(chapter 6.6)으로 해결한다
- **뒤엉킨 변경**
  - 코드를 수정할 때는 시스템에서 교쳐야 할 딱 한 군데를 찾아서 그 부분만 수정하자
    - 이렇지 못한 경우 뒤엉킨 변경과 산탄총 수술중 하나가 발생함
  - 단일 책임 원칙이 제대로 지켜지지 않았을 경우 발생함
- **산탄총 구슬**
  - 코드를 변경할 때마다 자잘하게 수정해야 하는 클래스가 많은 경우에 발생
  - 함수 옮기기(chapter 8.1)와 필드 옮기기(chapter 8.2)를 이용하자
  - 어설프게 분리된 로직을 함수 인라인하기(chapter6.2)나 클래스 인라인하기(chapter 7.6)를 이용해 합치자
- **기능 편애**
  - 자기가 속한 모듈의 함수나 데이터보다 다른 모듈의 함수나 데이터와 상호작용을 많이하는 것을 의미
  - 프로그램 모듈화시 영역 안에서 이뤄지는 상호작용은 최대로 늘리고 영역 사이에서 이뤄지는 상호 작용은 최소로 줄인다
- **데이터 뭉치**
  - 데이터 항목 여러개가 항상 붙어 다니는 경우, 데이터 항목 하나를 삭제 했을때 나머지 항목만으로 의미가 없다면 데이터 뭉치로 판단하여 클래스화 시켜준다
- **추측성 일반화**
  - 나중에 필요할 것이라는 생각에 작성해둔 코드
- **임시 필드**
  - 특정 상황에서만 값이 설정되는 필드를 가진 클래스들
    - 클래스 추출하기(chapter 7.5)를 이용한다
    - 함수 옮기기(chapter 8.1)를 이용한다
- **메시지 체인**
  - getter의 getter... 계속 이어지는 코드
    - 얻고자 하는 데이터를 함수화시켜 중간 객체들을 숨기자
- **중개자**
  - 객체의 캡슐화를 통한 위임을 이용할 때, 클래스의 메서드 중 절반이 다른 클래스에 구현을 위임한다면 중개자 제거하기(chapter 7.8)를 이용한다
- **상속 포기**
  - 계층 구조 설계를 잘못 했을 때, 일부 데이터만 상속 받기를 원하는 경우
  - 같은 계층에 서브클래스를 하나 만들고, 메서드 내리기(chapter 12.4)와 필드 내리기(chapter 12.5)를 활용해 물려받지 않을 부모 코드를 모조리 새로 만든 서브클래스로 넘긴다
    - 엄~청 좋은 방식은 아니지만, 실무에서 유용함
  - 서브클래스를 위임으로 바꾸기(chapter 12.10)나 슈퍼클래스를 위임으로 바꾸기(chapter 12.11)를 이용해보자
- **주석**
  - 주석을 남겨야겠다는 생각이 들면, 가장 먼저 주석이 필요 없는 코드로 리팩터링 해보자



## Chapter 04

- **자가 테스트 코드의 가치**
  - 실제 코드 작성 시간은 길지 않다.
  - 대부분에 시간을 디버깅에 쓴다.
  - 테스트 자동화를 구축하면 디버깅 시간을 대폭 줄여준다.
    - 기능을 추가하기 전에 테스트를 먼저 작성하는 것이 좋다
- **테스트 유의사항**
  - 실패해야 할 상황에서는 반드시 실패하게 만들자
    - 일시적인 오류 주입을 통해 실패를 확인한다
  - 자주 테스트하자. 작성 중인 코드는 최소한 몇 분 간격으로 테스트하고, 적어도 하루에 한 번은 전체 테스트를 돌려보자.
- **테스트 추가하기**
  - 테스트는 위험 요인을 중심으로 작성해야 한다.
    - 테스트의 목적은 어디까지나 현재 혹은 향후에 발생하는 버그를 찾는 데 있다.
    - 단순히 필드를 읽고 쓰기만 하는 접근자는 테스트 할 필요가 없다.
  - 완벽하게 만드느라 테스트를 수행하지 못하느니, 불완전한 테스트라도 작성해 실행하는게 낫다.

- **경계 조건 검사하기**
  - 문제가 생길 가능성이 있는 경계 조건을 생각해보고 그 부분을 집중적으로 테스트하자.



## Chapter 06

- **함수 추출하기**

  - ```javascript
    // 변경전
    function printOwing(invoice) {
        printBanner();
        let outstanding = calculateOutstanding();
        
        //세부 사항 출력
        console.log(`고객명: ${invoice.customer}`);
        console.log(`채무액: ${outstanding}`);
    }
    
    // 변경후
    function printOwing(invoice) {
        printBanner();
        let outstanding = calculateOutStanding();
        printDetails(outstanding);
        
        function printDetails(outstanding) {
        //세부 사항 출력
        console.log(`고객명: ${invoice.customer}`);
        console.log(`채무액: ${outstanding}`);
        }
    }
    ```

  - 코드 조각을 찾아 무슨 일을 하는지 파악한 다음, 독립된 함수로 추출하고 목적에 맞는 이름을 붙인다.

  - **재사용성**을 기준으로, 두 번 이상 사용될 코드는 함수로 만들고, 한 번만 쓰이는 코드는 인라인 상태로 놔둔다.

    - 재사용성도 좋지만, 코드를 보고 무슨 일을 하는지 파악하기 어렵다면 그 부분을 함수로 추출한 뒤 '무슨 일'에 걸맞는 이름을 짓는 방식을 선호한다.
    - 이 원칙을 사용하면, 함수의 길이들이 굉장히 짧게 바뀐다.
    - 함수가 짧으면 캐싱하기가 더 쉽기 때문에 컴파일러가 최적화하는 데 유리할 때가 많다.

  - 절차

    - 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다.('어떻게'가 아닌 '무엇을' 하는지가 드러나야 한다.)
    - 추출한 코드를 원본 함수에서 복사하여 새 함수에 붙여놓는다.
    - 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
    - 변수를 다 처리했으면 컴파일한다.
    - 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다.
    - 테스트한다.
    - 다른 코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다. 있다면 방금 추출한 새 함수를 호출하도록 바꿀지 검토한다.

    

- **함수 인라인하기**

  - ```javascript
    // 변경전
    function getRating(driver) {
        return moreThanFiveLateDeliveries(driver) ? 2 : 1;
    }
    
    function moreThanFiveLateDeliveries(driver) {
        return driver.nmuberOfLateDeliveries > 5;
    }
    
    // 변경후
    function getRating(driver) {
        return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
    }
    ```

  - 함수 추출하기와 반대되는 리팩터링

    - 함수 본문이 이름만큼 명확한 경우 사용
    - 함수 본문을 이름만큼 깔끔하게 리팩터링한 경우 사용

  - 절차

    - 다형 메서트인지 확인한다. (서브클래스에서 오버라이드하는 메서드는 인라인하면 안 된다.)
    - 인라인할 함수를 호출하는 곳을 모두 찾는다.
    - 각 호출문을 함수 본문으로 교체한다.
    - 하나씩 교체할 때마다 테스트한다.
    - 함수 정의를 삭제한다.

    

- **변수 추출하기**

  - ```javascript
    // 변경전
    return order.quantity * order.itemPrice - Math.max(0,order.quantity - 500) * order.itemPrice * 0.05 + Math.min(order.quantity * order.itemPrice * 0.1, 100);
    
    // 변경후
    const basePrice = order.quantity * order.itemPrice;
    const quantityDisCount = Math.max(0,order.quantity - 500) * order.itemPrice * 0.05;
    const shipping = Math.min(basePrice * 0.1, 100);
    return basePrice - quantityDiscount + shipping;
    ```

  - 표현식이 너무 복잡해서 이해하기 어려울 때 지역 변수를 활용하여 쉽게 만든다.

  - 디버거에 중단점 지정이나 상태 출력 문장을 추가할 수 있어 디버깅이 쉬워진다.

  - 현재 함수 안에서만 의미가 있다면 변수로 추출하는 것이 좋다.

  - 절차

    - 추출하려는 표현식에 부작용은 없는지 확인한다.
    - 불변 변수를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입한다.
    - 원본 표현식을 새로 만든 변수로 교체한다.
    - 테스트한다.
    - 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체한다. 하나 교체할 때마다 테스트 한다.

- **변수 인라인하기**

  - ```javascript
    // 변경전
    let basePrice = anOrder.basePrice;
    return (basePrice > 1000);
    
    // 변경후
    return anOrder.basePrice > 1000;
    ```

  - 변수 추출하기와 반대되는 리팩터링

    - 이름이 원래 표현식과 다를 바 없을때 사용

  - 절차

    - 대입문의 우변에서 부작용이 생기지는 않는지 확인한다.
    - 변수가 불변으로 선언되지 않았다면 불변으로 만든 후 테스트한다.
    - 이 변수를 가장 처음 사용하는 코드를 찾아서 대입문 우변의 코드로 바꾼다.
    - 테스트한다.
    - 변수를 사용하는 부분을 모두 교체할 때까지 이 과정을 반복한다.
    - 변수 선언문과 대입문을 지운다.
    - 테스트한다.

    

- **함수 선언 바꾸기**

  - ```javascript
    // 변경전
    function circum(radius) {...}
    
    // 변경후
    function circumference(radius) {...}
    ```

  - 이름이 좋으면 함수의 구현 코드를 살펴볼 필요 없이 호출문만 보고도 무슨 일을 하는지 파악할 수 있다.

  - 좋은 이름을 떠올리는 법

    - 주석을 이용해 함수의 목적을 설명해보는 것.

  - 절차

    - 매개변수를 제거하려거든 먼저 함수 본문에서 제거 대상 매개변수를 참조하는 곳은 없는지 확인한다.
    - 메서드 선언을 원하는 형태로 바꾼다.
    - 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정한다.
    - 테스트한다.

    

- **변수 캡슐화하기**

  - ```javascript
    // 변경전
    let defaultOwner = {firstName: "마틴", lastName: "파울러"}
    
    // 변경후
    let defaultOwnerData = {firstName: "마틴", lastName: "파울러"};
    export function defaultOwner()	{return defaultOwnerData;}
    export function setDefaultOwner(arg) {defaultOwnerData = arg;}
    ```

  - 함수는 데이터보다 다루기가 수월하다.

    - 이름을 바꾸거나 다른 모듈로 옮기기 쉽다.
    - 기존함수를 그대로 둔 채 전달함수로 활용할 수도 있다.

  - 접근할 수 있는 범위가 넓은 데이터를 옮길 때는 먼저 그 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 가장 좋은 방법이다.

  - 데이터 캡슐화는 데이터를 변경하고 사용하는 코드를 감시할 수 있는 확실한 통로가 되어주기 때문에 데이터 변경 전 검증이나 변경 후 추가 로직을 쉽게 끼워 넣을 수 있다.

  - 데이터의 유효범위가 넓으면 캡슐화 해야한다.

  - 절차

    - 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 만든다.
    - 정적 검사를 수행한다.
    - 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 바꾼다. 하나씩 바꿀 때마다 테스트 한다.
    - 변수의 접근 범위를 제한한다.
    - 테스트한다.
    - 변수 값이 레코드라면 레코드 캡슐화하기를 적용할지 고려해본다.



- **변수 이름 바꾸기**

  - ```javascript
    // 변경전
    let a = height * width;
    
    // 변경후
    let area = height * width;
    ```

  - 함수 호출 한 번으로 끝나지 않고 값이 영속되는 필드의 경우 이름에 더 신경써야한다.

  - 절차

    - 폭넓게 쓰이는 변수라면 변수 캡슐화하기를 고려한다.

    - 이름을 바꿀 변수를 참조하는 곳을 모두 찾아서, 하나씩 변경한다.

    - 테스트한다.

      

- **매개변수 객체 만들기**

  - ```javascript
    // 변경전
    function amountInvoiced(startDate, endDate) {...}
    function amountReceived(startDate, endDate) {...}
    function amountOverdue(startDate, endDate) {...}
    
    // 변경후
    function amountInvoicde(aDateRange) {...}
    function amountReceived(aDateRange) {...}
    function amountOverdue(aDateRange) {...}
    ```

  - 데이터 항목 여러 개가 여러 함수에서 몰려다니며 사용되는 경우 데이터 구조 하나로 묶어준다.

  - 데이터 구조화 리팩터링을 통해 데이터 구조를 이용하는 형태로 프로그램이 재구성된다.

  - 데이터 구조화를 통해 문제 영역을 훨씬 간결하게 표현하는 새로운 추상 개념으로 격상되면서, 코드의 개념적인 그림을 다시 그릴수도 있다.

  - 절차

    - 적당한 데이터 구조가 아직 마련되어 있지 않다면 새로 만든다.

    - 테스트한다.

    - 함수 선언 바꾸기로 새 데이터 구조를 매개변수로 추가한다.

    - 테스트한다.

    - 함수 호출 시 새로운 데이터 구조 인스턴스를 넘기도록 수정한다. 하나씩 수정할 때마다 테스트한다.

    - 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 바꾼다.

    - 다 바꿨다면 기존 매개변수를 제거하고 테스트한다.

      

- **여러 함수를 클래스로 묶기**

  - ```javascript
    // 변경전
    function base(aReading) {...}
    function taxbleCharge(aReading) {...}
    function calculateBaseCharge(aReading) {...}
    
    // 변경후
    class Reading {
    	base() {...}
    	taxbleCharge() {...}
    	calculateBaseCharge() {...}
    }
    ```

  - 공통 데이터를 중심으로 긴밀하게 엮여 작동하는 함수 무리를 발견하면 클래스로 묶는다.

  - 클래스로 묶을 때의 두드러진 장점은 클라이언트가 객체의 핵심 데이터를 변경할 수 있고, 파생 객체들을 일관되게 관리할 수 있다.

  - 절차

    - 함수들이 공유하는 공통 데이터 레코드를 캡슐화한다.

    - 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮긴다.

    - 데이터를 조작하는 로직들은 함수로 추출해서 새 클래스로 옮긴다.

      

- **여러 함수를 변환 함수로 묶기**

  - ```javascript
    // 변경전
    function base(aReading) {...}
    function taxableCharge(aReading) {...}
    
    // 변경후
    function enrichReading(argReading) {
    	const aReading = _.cloneDeep(argReading);
    	aReading.baseCharge = base(aReading);
    	aReading.taxableCharge = taxableCharge(aReading);
    	return aReading;
    }
    ```

  - 어떤 정보가 사용되는 곳마다 같은 도출 로직이 반복되는 경우, 도출 작업들을 한데로 모아 검색과 갱신을 일관된 장소에서 처리하고 중복도 막을수 있다.

  - 변환 함수를 이용하여 원본 데이터를 입력 받아서 필요한 정보를 모두 도출한 뒤, 각각을 출력 데이터의 필드에 넣어 반환한다.

  - 여러 함수를 클래스로 묶기를 대용으로 사용 가능하다.

    - 원본 데이터가 코드 안에서 갱신될 때는 클래스로 묶는게 낫다. 변환 함수로 묶으면 가공한 데이터를 새로운 레코드에 저장하므로, 원본데이터가 수정되면 일관성이 깨질 수 있기 때문이다.

  - 절차

    - 변환할 레코드를 입력받아서 값을 그대로 반환하는 변환 함수를 만든다.

    - 묶을 함수 중 함수 하나를 골라서 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드에 새 필드로 기록한다. 그런 다음 클라이언트 코드가 이 필드를 사용하도록 수정한다.

    - 테스트한다.

    - 나머지 관련 함수도 위 과정에 따라 처리한다.

      

- **단계 쪼개기**

  - ```javascript
    // 변경전
    const orderData = orderString.split(/\s+/);
    const productPrice = priceList[orderData[0].split("-")[1]];
    const orderPrice = parseInt(orderData[1]) * productPrice;
    
    // 변경후
    const orderRecord = parseOrder(order);
    const orderPrice = price(orderRecord, priceList);
    
    function parseOrder(aString) {
    	const values = aString.split(/\s+/);
    	return ({
    		productID: values[0].split("-")[1],
    		quantity: parseInt(values[1]),
    	});
    }
    function price(order, priceList) {
    	return order.quantity * priceList[order.productID];
    }
    ```

  - 서로 다른 두 대상을 한꺼번에 다루는 코드에 적용

    - 동작을 연이은 두 단계로 쪼개는 것이 가장 편리한 방식
      - 각 단계는 자신만의 문제에 집중하기 때문에 나머지 단계에 관해서는 자세히 몰라도 이해가능
    - 규모에 관계없이 여러 단계로 분리하면 좋을만한 코드를 발견할 때마다 기본적인 단계쪼개기 리팩터링을 진행한다.

  - 절차

    - 두 번째 단계에 해당하는 코드를 독립 함수로 추출한다.
    - 테스트한다.
    - 중간 데이터 구조를 만들어서 앞에서 추출한 함수의 인수로 추가한다.
    - 테스트한다.
    - 추출한 두 번째 단계 함수의 매개변수를 하나씩 검토한다. 그중 첫 번째 단계에서 사용되는 것은 **중간 데이터 구조**(매개변수 줄이는용)로 옮긴다. 하나씩 옮길 때마다 테스트한다.
    - 첫 번째 단계 코드를 함수로 추출하면서 중간 데이터 구조를 반환하도록 만든다.



## Chapter 7

- **레코드 캡슐화하기**

  - ```javascript
    organization = {name: "애크미 구스베리", country: "GB"};
    =>
    class Organization {
    	constructor(data) {
    	this._name = data.name;
    	this._country = data.country;
    	}
    	get name() {return this._name;}
    	set name(arg) {this._name = arg;}
    	get country() {return this._country;}
    	set country(arg) {this._country = arg;}
    }
    ```

  - 레코드 형식은 데이터를 직관적으로 묶을수 있다는 장점이 있지만, 계산해서 얻을수 있는 값과 그렇지않은 값을 구분해 저장하는 점이 번거롭다

  - 클래스를 이용하면 데이터를 캡슐화하여 제공할 수 있다

  - (c++에 레코드 타입이 있나..?)

    

- **컬렉션 캡슐화하기**

  - ```javascript
    class Person {
    	get courses() {return this._courses;}
    	set courses(aList) {this._courses = aList;}
    }
    =>
    class Person {
    	get courses() {return this._courses.slice();}
    	addCourse(aCourse) {...}
    	removeCourse(aCourse) {...}
    }
    ```

  - 컬렉션 변수로의 접근을 캡슐화하면서 게터가 컬렉션 자체를 반환하도록 한다면, 그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀔수 있다

    - javascript만 그런가..?

  - 컬렉션 게터를 제공하되 내부 컬렉션의 복사본을 반환하여 해결할 수 있다.

    

- **기본형을 객체로 바꾸기**

  - ```javascript
    ordrs.filter(o=> "high" === o.priority || "rush" === o.priority);
    =>
    orders.filter(o => o.priority.higherThan(new Priority("normal")))
    ```

  - 단순한 출력 이상의 기능이 필요해지는 순간 그 데이터를 표현하는 전용 클래스를 정의하자



- **임시** **변수를 질의 함수로 바꾸기**

  - ```javascript
    const basePrice = this._quantity * this._itemPrice;
    if (basePrice > 1000)
    	return basePrice * 0.95;
    else
    	return basePrice * 0.98;
    =>
    get basePrice() {this._quantity * this._itemPrice;}
    ...
    if (this.basePrice > 1000)
    	return this.basePrice * 0.95;
    else
    	return this.basePrice * 0.98;
    ```

  - 변수 대신 함수로 만들어두면 비슷한 계산을 수행하는 다른 함수에서도 사용할 수 있어서 코드중복이 줄어든다.

    - 클래스 안에서 적용할 때 효과가 크다. 클래스는 추출한 메서드들에 공유 컨텍스트를 제공하기 때문이다.



- **클래스 추출하기**

  - ```javascript
    class Person {
    	get officeAreaCode() {return this._officeAreaCode;}
    	get officeNumber() {return this._officeNumber;}
    }
    =>
    class Person {
    	get officeAreaCode() {return this._telephoneNumber.areaCode;}
    	get officeNumber() {return this._telephoneNumber.number;}
    }
    class TelephoneNumber {
    	get areaCode() {return this._areaCode;}
    	get number() {return this._number;}
    }
    ```

  - 메서드와 데이터가 너무 많은 클래스는 이해하기가 쉽지 않으니 잘 살펴보고 적절히 분리하는것이 좋다.



- **클래스 인라인하기** (반대 리팩터링 => 클래스 추출하기)

  - ```javascript
    class Person {
    	get officeAreaCode() {return this._telephoneNumber.areaCode;}
    	get officeNumber() {return this._telephoneNumber.number;}
    }
    class TelephoneNumber {
    	get areaCode() {return this._areaCode;}
    	get number() {return this._number;}
    }
    =>
    class Person {
    	get officeAreaCode() {return this._officeAreaCode;}
    	get officeNumber() {return this._officeNumber;}
    }
    ```

  - 제 역할을 못하는 클래스는 인라인한다.

  - 두 클래스의 기능을 지금과 다르게 배분하고 싶을때 인라인한다.



- **위임 숨기기**
  - ```javascript
    manager = aPerson.department.manager;
    =>
    manager = aPerson.manager;
    
    class Person {
    	get manager() {return this.department.manager;}
    }
    ```

  - 모듈화 설계를 제대로 하는 핵심은 캡슐화이다. 캡슐화는 모듈들이 시스템의 다른 부분에 대해 알아야 할 내용을 줄여준다.

  - 위임 객체를 숨기면 수정 사항이 발생했을 때 의존성이 제거되어 클라이언트 코드를 수정하지 않아도 된다.



- **중개자 제거하기** (반대 리팩터링 => 중개자 제거하기)

  - ```javascript
    manager = aPerson.manager;
    
    class Person {
    	get manager() {return this.department.manager;}
    }
    =>
    manager = aPerson.department.manager;
    ```

  - 클라이언트가 위임 객체의 또 다른 기능을 사용하고 싶을 때마다 서버에 위임 메서드를 추가해야하는데, 이렇게 **기능을 추가하다 보면 단순히 전달만 하는 위임 메서드들**이 점점 성가셔진다.



- **알고리즘 교체하기**
  - 문제를 더 확실히 이해하고 훨씬 쉽게 해결하는 방법을 발견하면 교체한다.
  - 같은 코드의 신뢰성있는 라이브러리가 있어도 교체한다.