---
date: 2022-11-08 12:00:00 +0900
title: 이더리움 창시자가 들려주는 Merge 이야기 
excerpt: 비탈릭 부테린이 왜 NTU에서 나와?
categories: blockchain 
---

## 세미나 개요

- 제목: The merge and what’s next for Ethereum
- 일시: 2022.11.07 16:30
- 장소: LT8, Nanyang Technological University, Singapore
- 연사: Vitalik Buterin, **Creator of Ethereum**
- 주최: CCTF (Centre of Computational Technologies in Finance)

## 참석하기까지

NTU 블록체인 동아리에서 세미나 참석 링크를 올렸다. 평소 블록체인 관련 강연이나
교육 프로그램, 인턴 구인 글들이 텔레그램에 자주 나오곤 했다. 그런데 이번에는
좀 달랐다. 평소 이런 글은 모든 사람이 볼 수 있는 Public 채팅방과 동아리
Sub-committee에 속한 사람만 모인 Private 채팅방 둘 다 올라왔다. 하지만 이번
세미나는 특이하게도 Private 채팅방에만 올라왔고 왜 그런가 자세히 읽어보았다.

<figure>
  <img src="https://i.imgur.com/DZCyxfG.jpg"
       alt="poster">
  <figcaption>
    응? 이더리움을 만든 사람이 여기로 온다고?
  </figcaption>
</figure>

비탈릭이라는 이더리움 창시자가 연사인 것을 보고 화들짝 놀랐다. 그 영어 이름을
복사해서 [구글에 검색](https://www.google.com/search?q=비탈릭+부테린)해봤는데
진짜였다. 그래도 긴가민가 싶었지만 일단 Register 부터 했다. 역시나 짧은 시간
안에 Offline 등록은 마감되었다. 세미나 당일 일찍 LT에 찾아갔다. 평소에는 시작
5분 뒤 도착이었지만, 이번에는 시작 30분 전에 도착했다. 이미 문 앞에는 긴 줄이
늘어섰고, 사전 등록한 사람만 입장하도록 통제했으며 내부는 거의 다 찬 상태였다.
다행히도 나는 빈자리를 찾았고 앉아서 세미나를 들었다.

<figure>
  <img src="https://i.imgur.com/pGzWd0N.png"
       alt="my_photo">
  <figcaption>
    세미나를 열심히 듣는 나. 가방에 쿠키앤크림 빼빼로 넣고 하나씩 빼먹으면서 들었다.
  </figcaption>
</figure>

비탈릭이 등장하자 마자 사람들은 연신 카메라를 켜서 그를 찍기 시작했다. 처음
나는 시큰둥했지만, 그에 대해 구글링을 하던 중 그의 이더리움 잔고(조
단위더라)를 보고 바로 인증샷을 남겼다. 그 때 그는 싱가포르에서 가장 돈이
많은 사람이었을 것이다. 본격적으로 세미나가 시작된 이후 빈자리는 사라졌고 계단
조차 빈 공간이 거의 없었다. 세미나가 끝난 후에 사람들은 자신들이 들어온 입구가
아닌 비탈릭이 나가는 입구 쪽으로 줄지어서 이동하기 시작했다. 구름같은 군중을
이끌고 다니는 그의 인기는 실로 엄청났다.

<figure>
  <img src="https://i.imgur.com/IVivhzq.jpg"
       alt="vitalik_by_me">
  <figcaption>
    진짜 LT8에 온 비탈릭. NTU 총장님이 와도 이 정도 인기는 아닐 것이다.
  </figcaption>
</figure>

## 세미나 요약

다시 본론으로 돌아오자. 제목에서 알 수 있듯 이더리움 2.0 The Merge와 관련된
세미나였다. Merge까지 있었던 일들을 언급했고, Merge 이후 나타났던 성과들을
홍보하였다. 그리고 이더리움이 앞으로 해결해야 할 문제들을 설명하였다. 

## Merge 과정

이번 Merge의 핵심은 PoW에서 PoS로의 전환이다. 이는 한 순간에 일어난 일이
아니다. 수 년 전부터 PoS 도입에 관한 논의와, 계획 그리고 개발이 있었다.

<figure>
  <img src="https://i.imgur.com/May4gze.jpg"
       alt="the_road_to_merge">
  <figcaption>
    Merge 까지의 여정. 과연 2014년에 이더리움이 PoS로 돌아가리라 누가 예상할 수 있었을까?
    &emsp;/&emsp;
    <a href="https://www.ntu.edu.sg/cctf/news-events/news/details/cctf-distinguished-lecture-series-by-vitalik-buterin-creator-of-ethereum">
      출처: NTU CCTF
    </a>
  </figcaption>
</figure>

### 초기 PoS에 대한 생각

PoS에 관한 블로그 포스트를 통해 당시 비탈릭의 견해를 엿볼 수 있다. 2014년
1월에 올라온
[’Slasher’](https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm)
제목의 글에서는 PoS가 도입될 것이라는 느낌은 없었고, 그저 PoW에 대한 흥미로운
대안 중 하나라고 생각하고 있었다. PoS의 장점을 언급하면서도 ‘Nothing at stake’
문제를 지적했는데, 이를 해결해보려는 자신만의 방법을 설명하는 것으로 끝났다.

2014년 11월에 올라온 [‘Weak
Subjectivity’](https://blog.ethereum.org/2014/11/25/proof-stake-learned-love-weak-subjectivity)에서는
‘Nothing at stake’ 문제에 대해 좀 더 심화된 논의를 하였다. Fork 상황을
Short-range와 Long-range로 나누고 두 상황 모두 해결 가능함을 설명했다. 그리고
이러한 해결책들이 한계 비용 측면에서 보았을때도 실현 가능하다고 하였다.

### 구현 과정

PoS를 구현하는 과정도 단계적으로 이루어졌다. 우선 PoS 기반으로 돌아가는 새로운
블록체인인 [Casper FFG](https://arxiv.org/abs/1710.09437)가 개발되었다. 이는
이더리움과 독립적이며, 실제 운영 경험을 쌓으면서 PoS로의 전환을 준비하는
역할이었다. 그리고 [Beacon
Chain](https://ethereum.org/en/upgrades/beacon-chain/)이 개발되었는데 이는 PoS
형식으로 블록의 검증을 담당하는 역할이고, 이더리움의 차세대 합의 알고리즘이
담겨 있다. [Altair
fork](https://github.com/ethereum/consensus-specs/blob/dev/specs/altair/fork.md)에서는
이더리움에 상대적으로 작은 업데이트가 있었는데, light client들도 검증에
참여하는 기능이 추가되었다. 마지막으로 대망의 [‘The
Merge’](https://ethereum.org/en/upgrades/merge/)에서는 PoW로 운영되는 메인넷이
PoS기반의 Beacon Chain과 합쳐졌다.

## Merge 결과

<figure>
  <img src="https://i.imgur.com/lViSmFk.jpg"
       alt="power_comsuption">
  <figcaption>
    Merge 이후 획기적인 전력사용량 감소. 저런 차트는 다단계가 아니고선 믿기 어렵다.
    &emsp;/&emsp;
    <a href="https://www.ntu.edu.sg/cctf/news-events/news/details/cctf-distinguished-lecture-series-by-vitalik-buterin-creator-of-ethereum">
      출처: NTU CCTF
    </a>
  </figcaption>
</figure>

비탈릭이 제시한 Merge의 가장 큰 영향은 바로 전력 사용량이었다. 무려 99.95%나
감소했다고 한다. 이 정도 수치면 사기라고 느껴질만큼 엄청난 성과였고,
암호화폐를 비판하는 이유가 하나 줄겠구나라는 생각을 하게 되었다. 그 다음
성과로는 합의 디자인의 발전 있었는데, parallel confirmations의 도입으로 더
많은 검증을 빠른 시간에 달성할 수 있게 되었다. 마지막으로 Fork 발생의 감소가
있었다. 이는 빨라진 검증 때문에 블록끼리 경합이 발생할 일 자체가
줄어들었으며 블록체인의 안정성이 강화되었다고 할 수 있다.

## Merge 이후

### Remaining problems

다음으로 비탈릭은 Merge 이후 이더리움에 남아있는 문제들에 대해 소개했다. 여러
개선할 점들이 있었는데, 크게 정리하자면 다음과 같다.

- User Experience & Scalability
- Security Issues & Privacy

우선 거래가 너무 느리다는 것이다. 이는 여러 차례 업데이트를 통해 꾸준히, 또는
획기적으로 개선되었다고는 하지만 일상생활에서 사용하기에는 여전히 부족하다.
비탈릭은 실제로 이더리움으로 결제해 본 경험을 들려주었는데, 자신도 불편하다는
것을 알고 있었다. 더 빠른 거래를 위해 Sharding으로 검증 건수와 시간을 개선하고
있다고 한다.

두 번째 문제점은 보안에 대한 걱정이었다. 비트코인의 흥행이 PoW가 안전하다는
것을 보장하고 있음에도 이더리움은 PoS 방식을 도입했다. 그리고
[Sharding](https://ethereum.org/en/upgrades/sharding/#main-content) 또한
이더리움과 같이 인기 많은 블록체인 생태계에 새롭게 도입되었기 때문에 숨겨진
버그가 있을 수도 있다. 보안은 블록체인의 기본이기 때문에 여러 이더리움
이해관계자들이 잘 해결하리라고 기대해본다.

### Danksharding

Danksharding은 The Merge 이후 이더리움 코어 개발자들이 가장 집중해서 하는
일이라고 한다. 이를 도입하는 이유를 알기 위해선 일반적인 블록체인 상식 위에
Rollup에 대한 이해가 필요하다.

<figure>
  <img src="https://i.imgur.com/3IM8dn6.png"
       alt="rollup">
  <figcaption>
    Rollup 설명. Mainnet에는 간단하게 기록된다.
    &emsp;/&emsp;
    <a href="https://vitalik.ca/general/2021/01/05/rollup.html">
      출처: Vitalik
    </a>
  </figcaption>
</figure>

최근 이더리움은 거래를 기록하기 위해
[Rollup](https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/#top)
방식을 사용하고 있다. 이는 이더리움 외부에서 거래들을 잘게 나누어 처리하고, 그
결과만을 이더리움 내부에 기록하는 것이다. 이 방식의 장점은 노드마다 다루는
데이터가 작으니 처리 속도가 획기적으로 개선되며, 더 많은 노드들이 블록생성에
참여할 수 있으니 탈중앙화도 나아진다 반면에 단점으로는 거래 내용이 외부에
있어서 Data Availability Attack이 발생할 수 있다. 해커가 잘못된 정보를
이더리움 외부에서 숨겨둔 상태이지만, 이더리움 내부에선 이를 모르고 정상적인
정보로 간주할 수 있다는 것이다.

[Danksharding](https://medium.com/@coiniseasy/danksharding을-최대한-이해해보자-a57e1a82e1e7)은
기존 Rollup 방식과 차원이 다르다. 변화의 골자는 다음과 같다.

- 거래 내용을 다시 이더리움 안으로 가져오자
    - 이건 아마 이더리움의 보안성을 외주화하지 않겠다는 개발자들의 결연한
    의지로 보인다.
- 그리고 블록을 생성할 때 많은 데이터를 한 번에 처리하자
    - 그렇게 된다면 블록 생성은 성능 좋은 소수의 노드들만 할 수 있게 되버린다.

<figure>
  <img src="https://i.imgur.com/AU8TQi0.jpg"
       alt="danksharding">
  <figcaption>
    Danksharding 설명. 데이터들이 굵직하게 메인넷 블록에 담겨있다.
    &emsp;/&emsp;
    <a href="https://www.ntu.edu.sg/cctf/news-events/news/details/cctf-distinguished-lecture-series-by-vitalik-buterin-creator-of-ethereum">
      출처: NTU CCTF
    </a>
  </figcaption>
</figure>

소수의 노드들만 관여할 수 있다는 것은 확장성과 탈중앙화에 역행하는 것이다.
이를 보완하고자 Danksharding은 블록 생성 이후 블록 검증이라는 새로운 일자리를
만들었다. 이 검증 작업은 블록 생성에 비해 상대적으로 적은 컴퓨팅 파워를
사용하니 여러 작은 노드들도 참여할 수 있다. Danksharding은 기존 방식을 보완한
게 아니라, 보안성과 확장성, 탈중앙화를 모두 지키고자 완전히 새로운 방식으로
접근해보자는 것이다.

### Account Abstraction

Account Abstraction은 사용자 경험에 관한 이더리움 개발자들의 배려라고 볼 수
있다. 이는 하나의 Wallet을 여러 Private Key로 관리하자는 것이다. 원래 하나의
Wallet은 하나의 Key로 관리하는 것이었고, 그 Key를 잃어버리는 순간 Wallet의
잔고는 영영 못 쓰게 된다는 문제가 있었다. 이를 보완하고자 이더리움 거래 상에서
Wallet의 인터페이스는 유지하되, 내부적으로 여러 Contract들을 합쳐서
운영해보자는 것이다. 물론 이것도 이더리움이 확장성을 개선한 이후 많은 사람들이
개인 Wallet을 이용해봐야 빛을 발휘할 것이다.

<figure>
  <img src="https://i.imgur.com/X3BpyKE.png"
       alt="account_abstraction">
  <figcaption>
    Account Abstraction 설명. Transaction 상으로는 변화를 눈치채지 못 한다.
    &emsp;/&emsp;
    <a href="https://eips.ethereum.org/EIPS/eip-4337">
      출처: Ethereum EIP
    </a>
  </figcaption>
</figure>

## Q&A

그렇게 Merge와 Merge 이후 이더리움에 남겨진 과제들에 대한 설명이 끝났다. 그
다음으로 Q&A 시간이 있었다. 세미나를 들을 당시에는 배경지식이 부족해서 솔직히
이해가 힘들었는데, 그래도 Q&A는 좀 알아들을 수 있었다. 질문할 사람 손 들라고
했는데 나빼고 다 손을 들었던 것 같다. 기억에 남는 내용 몇 개를 정리해보겠다.

- **Q. ZKP(Zero Knowledge Proof)의 활용에 대해 어떻게 생각하는가?**
    - 최근 5년 새 주목받고 있고 아주 좋은 기술이라고 생각한다. 현재 가장 큰
    문제는 costly 하다는 점이다. Privacy와 Scalability의 Trade-off를
    고려해봐야 한다.
- **Q. 클라우드로 채굴 많이하던데, CSP들이 이더리움의 탈중앙화에 미치는 영향에
대해 어떻게 생각하는가?**
    - 우리도 이 문제에 대해 심각하게 생각하고 있다. 그래서 패넡티를 주는
    방안도 검토하고 있다. 근데 이런 문제는 Evil Game Theory이랑, 해커의
    공격이랑, 확장성 등이 함께 논의되어야 한다. 또한 Penalty 대신 Benefit을
    주는 등 다른 방법들도 생각해봐야 한다.
- **Q. PoW에서 의미 없는 연산을 하는데, 이를 의미 있는 연산으로 바꿀 수 있지
않을까?**
    - “Meaningful”이라는 단어를 정의하기가 참 애매하다.
- **Q. 요즘 P2E games들이 늘어난 것에 대해 어떻게 생각하는가?**
    - 재밌자고 하는 게임인데, 그런 것들은 힘든 노동이지 더 이상 게임이 아닌 것
같다. (웃음)

## 느낀점

### 요즘 이더리움을 이해하는 방법

이번 세미나의 내용은 블록체인의 기본적인 내용과 결이 달랐다. 이 블로그에서
합의 알고리즘과 같은 블록체인의 바탕이 되는 내용을 공부할 때는 탈중앙화와
보안성을 달성하는 매커니즘 위주로 이해했다. 하지만 이번 The Merge와 앞으로
이더리움이 개발하려는 기능들의 대부분은 확장성과 관련이 되어있었다. 이들
대부분은 탈중앙화와 보안성에 취약할 수 밖에 없는 구조이다. 이와 같은 Trade-off
관계에서 창의적으로 균형을 찾는 것이 요즘 이더리움 발전 과정에서 일어나는
일들이다.

### 근본 과목을 열심히 공부하자

이더리움에 적용되는 신기술 대부분이 확장성과 관련있다보니 DB와 OS 시간에
들어본 내용들이 등장했다. 이더리움의 Rollup을 이해하면서 Optimistic과 관련된
내용이 나왔는데 DB에서 Concurrency Control과 관련해 이와 비슷한 개념이 있었다.

- **Pessimistic CC**: Update 전에 먼저 Lock을 잡아서 Conflict 방지. Hotspot에서는
성능이 잘 나오지만 Deadlock과 Lock Table 관리가 문제.
- **Optimistic CC**: Commit 전에 Conflict가 발생했는 지 확인. Conflict가 드문
상황에서만 유리한 성능.

그리고 Sharding과 관련된 내용에서 Block 사이즈에 관해 나왔는데, 이 역시도 OS
시간 Memory Management를 배우면서 들어본 내용이었다.

- **Coarse-Grained**: 관리가 쉬운 대신 Internal fragmentation이 발생할 수 있음
- **Fine-Grained**: Fragmentation 이슈는 해결하지만 관리가 까다로움

그리고 또 개인적인 경험으로 많은 양의 데이터라면 분산처리가 더 쉽지만,
데이터가 적은 경우라면 오히려 분산처리 성능이 반감된다. (Spark로 개발할 때
열심히 RDD를 썼지만 오히려 bottleneck이 생겨서 single-thread 코드로 다시
바꿨던 추억이 있다.)

구글링하다가 전공핵심 과목에서 배웠던 개념들이 등장할 때 반갑고 신기했지만,
앞으로 열심히 수업을 들어야겠다는 생각도 들었다.

## References

- [세미나 관련 뉴스자료, NTU CCTF](https://www.ntu.edu.sg/cctf/news-events/news/details/cctf-distinguished-lecture-series-by-vitalik-buterin-creator-of-ethereum)
- [비탈릭 개인 블로그](https://vitalik.ca)
- [Beacon Chain 업그레이드 설명, 이더리움](https://ethereum.org/en/upgrades/beacon-chain/)
- [Beacon Chain 이해하기, 블로그](https://medium.com/decipher-media/이더리움-[비콘-체인-이해하기-c0d6a80f3ecf)
- [Sharding+DAS 설명, 비탈릭 블로그](https://hackmd.io/@vbuterin/sharding_proposal#An-explanation-of-the-sharding--DAS-proposal)
- [Rollup 설명, 비탈릭 개인 블로그](https://vitalik.ca/general/2021/01/05/rollup.html)
- [Rollup 로드맵, 비탈릭 작성글](https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698)
- [Danksharding 이해하기, 블로그](https://medium.com/@coiniseasy/danksharding을-최대한-이해해보자-a57e1a82e1e7)
- [Danksharding F&Q, 비탈릭 작성글](https://notes.ethereum.org/@vbuterin/proto_danksharding_faq)

