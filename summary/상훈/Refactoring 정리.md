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