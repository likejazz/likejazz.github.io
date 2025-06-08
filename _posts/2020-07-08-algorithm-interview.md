---
layout: post
title: ! '알고리즘 코딩 테스트 책 출간 안내'
tags: ["Books"]
last_modified_at: 2025/06/08 19:25:15
---

<div class="message">
《파이썬 알고리즘 인터뷰》 출간에 부쳐, 지난 일 년여 간의 집필을 마무리하며 소회를 남겨봅니다.
</div>

<small>
*2020년 7월 19일 과정 업데이트*  
*2020년 7월 8일 초안 작성*
</small>

<!-- TOC -->

- [들어가며](#들어가며)
- [집필 과정](#집필-과정)
- [SNS](#sns)
- [서평에 대한 답글](#서평에-대한-답글)
- [YouTube](#youtube)
- [학습 노트](#학습-노트)
- [판매처](#판매처)
- [참고](#참고)

<!-- /TOC -->

## 들어가며

<p class="bg-danger">
★ 안내: 2023년 9월, <a href="/java-coding-test">《자바 알고리즘 인터뷰 with 코틀린》</a> 신간이 출간되었습니다.
</p>

2017년 8월, 카카오는 흥미로운 실험을 진행했습니다. 신입 개발자 채용을 이름과 연락처만 제시하면 누구나 지원할 수 있는 블라인드 방식의 공개 채용으로 진행한 것입니다. 과감한 시도였고 당시 IT 업계에서조차 보기 드문 사례였습니다.

좋은 개발자를 선발하기 위한 출제 위원회의 움직임도 바빠졌습니다. 주요 알고리즘을 녹여 내면서도 응시자들에게 친근하게 다가서기 위해 카카오 캐릭터를 이용한 수십 항목의 문제를 만들었고, 출제 위원들이 각자 돌아가며 꼼꼼히 문제를 풀이해 나갔습니다. 피어 리뷰를 통해 충분히 논의를 진행했고, 다수결로 출제 여부를 확정했습니다. 지루한 절차가 반복됐지만 꼭 필요하고도 합리적인 과정이었습니다. 그렇게 여러 단계를 거쳐 최종적으로 총 7개 문제를 엄선했고, 이 문제들은 지금까지도 좋은 평가를 받으며 블라인드 채용의 모범 사례가 되었습니다. 당시의 성공 덕분에 이후에도 블라인드 채용은 계속 이어지고 있으며 다른 기업의 채용에까지 영향을 미쳤으니, 돌이켜 보면 그 자랑스러운 역사의 현장에 일원으로서 함께할 수 있었다는 사실이 그저 영광스러울 따름입니다.

당시 개발자 공채가 끝난 후 저는 카카오 기술 블로그에 [카카오 신입 공채 1차 코딩 테스트 문제 해설](https://tech.kakao.com/2017/09/27/kakao-blind-recruitment-round-1/)을 직접 정리하여 작성했습니다. 그 전까지만 해도 모든 기업들은 코딩 테스트 이후에 문제는 비공개로 처리하여 문제 유출을 방지하는 것이 원칙이었습니다. 그러나 당시 카카오의 출제 위원회에는 문제를 공개하여 업계의 발전을 도모하고자 하는 도전적이고 진취적인 생각을 가진 사람들로 가득했고, 결국 모든 문제를 공개하고 직접 해설까지 곁들인 [국내 기업 최초의 코딩 테스트 해설](https://tech.kakao.com/2017/09/27/kakao-blind-recruitment-round-1/)을 발행하기에 이르렀습니다. 이후에도 카카오는 물론 네이버, 라인 등 많은 기업들이 제 글을 참고하여 코딩 테스트 해설을 게시하고 있으니, 업계의 발전과 좋은 문화 정착에 기여했다는 점에 자부심과 뿌듯함을 느낍니다.

[좋은 개발자란 어떤 개발자인가](http://likejazz.com/senior-developers/) 라는 주제는 사실 오랜 저의 고민이기도 합니다.  

지난 몇 년간은 좋은 개발자를 채용하기 위해 누구보다 노력해왔으며, 또한 제 자신이 여전히 현업에 종사하는 개발자로서 더 좋은 개발자가 되기 위해 꾸준히 노력해왔습니다. 이 책에는 수많은 면접자들을 대상으로 코딩 테스트를 출제한 경험, 기술 인터뷰를 수행한 경험, 그리고 면접을 더 잘하기 위해 수많은 회사의 코딩 면접, 기술 면접 과정을 면밀히 살펴본 경험을 담았습니다. 그리고 이를 바탕으로, 저 또한 이직을 시도하면서 겪은 여러 가지 다양한 경험까지도 녹여내어 종합했습니다. 면접관으로서, 또는 면접자로서 겪은 다양한 경험은 한동안 면접관의 입장에서는 미처 느끼지 못했던 잘못된 관행들을 다시금 면접자의 입장에서 깨닫게 되는 좋은 계기가 됐고, 이직 후에는 더 좋은 면접관으로 거듭나게 됐으며, 이 책을 통해 그간의 경험들을 모두 잘 정리해 담아냈음은 물론입니다.

이 책에는 다음 마인드맵과 같이 파이썬, 자료구조, 알고리즘, 크게 세 가지 주제로 분류하고 각각 풍부한 설명과 함께 기업에서 기술 인터뷰시 가장 선호하는 문제를 추려내 관련된 리트코드 문제를 풀이했습니다. 지난 카카오 공채 문제를 포함해 총 95문제를 각각 풀이하였습니다.

![마인드맵](https://user-images.githubusercontent.com/1250095/86745916-a62e9a00-c075-11ea-9aa5-8455e2527f87.png)

이 책은 기본적으로 코딩 인터뷰를 위한 책이지만 코딩 테스트는 물론, 알고리즘을 익혀 문제 풀이 능력을 키우고 싶은 모든 개발자를 위한 책이기도 합니다.

> "코딩 인터뷰를 준비하는 개발자나 채용 담당자는 물론, 문제 해결력을 키우려는 개발자에게 유용하며, 코딩을 취미로 삼은 모든 코딩 덕후에게 즐거움을 줄 책이다."
>
> "코딩 테스트를 직접 만들고 진행한 저자의 경험과 지식을 바탕으로 핵심 내용을 잘 짚어주므로, IT 기업을 희망하는 분들께 많은 도움이 될 것이다. 비단 지원자뿐만 아니라 컴퓨터를 활용해 문제를 해결하고 싶은 모든 분께 이 책을 추천한다."

이를 통해 현업에서 문제 해결 능력을 높이고, 궁극적으로 더 좋은 개발자가 되기 위한 길잡이를 제시합니다.

이 책을 위해 지난 일 년여 간은 거의 매일 밤 알고리즘 문제를 풀이해 왔습니다. 이 책은 매 주말 집필에 몰두해온 노력과 고민의 결실입니다. 뿐만 아니라 이 책은 IT 업계 최고의 개발자들인 많은 분들이 공헌해주신 결실이기도 합니다. 그간 오랜 여정에 도움을 주신 많은 분들께 진심으로 감사의 말씀을 전합니다.

## 집필 과정
그간 알고리즘 인터뷰를 진행해오면서 배우고 싶을 정도로 뛰어난 분들도 있었지만, 아예 손도 못대고 서로 어색한 시간만 보내는 경우도 적지 않았습니다. 무엇이 문제이길래 구글, 페이스북은 그렇게 잘 한다는 알고리즘을 우리는 손도 못대고 있는 것일까요.

물론 알고리즘 인터뷰에 대한 반박은 잘 알고 있습니다. 당장 실무에 필요한 역량이 훨씬 더 중요하다는 점도 충분히 잘 알고 있습니다. 그러나 좋은 식당에 가장 중요한 것은 기본기 라고 백종원은 늘 강조합니다. 좋은 개발자에게 가장 중요한 역량 또한 기본기 라고 저는 생각합니다. 알고리즘은 기초이자 곧 기본입니다. 기본기가 튼튼해야 훌륭한 제품을 만들 수 있습니다. 마이크로소프트가 수십 년간 최고의 자리를 지킬 수 있었던 것도, 구글, 페이스북이 계속해서 좋은 제품으로 시장을 선도하는 이유도, 튼튼한 기본기 때문이라고 믿어 의심치 않습니다.

우리말로 된 가이드를 제시하고 싶었습니다. 영어책과 겨뤄도 손색이 없을만한 우리말로 된 코딩 인터뷰 책을 만들고 싶었습니다. 줄곧 알고리즘 인터뷰를 진행하며 느꼈던 기쁨과 슬픔의 경험들은, 최고의 알고리즘 책을 위한 밑거름이 될거라 확신했습니다. 해마다 꾸준히 독서를 하며 쌓아올린 좋은 책의 기준과 눈높이는 스스로에게도 동일하게 적용할 수 있을거라 생각했습니다.

<img src="https://user-images.githubusercontent.com/1250095/87880557-8dd35d80-ca2d-11ea-9d54-3191dc06b42b.jpg" width="80%">

알고리즘 인터뷰를 진행한 날에는 밤 늦게까지 사무실에 남아 그 날의 인터뷰 문제를 다시 한 번 풀이해보면서 복기하고, 어떤 점이 좋았고 어떤 부분이 잘못되었는가를 꼼꼼히 기록해두곤 했습니다.

<img src="https://user-images.githubusercontent.com/1250095/87880226-f10fc080-ca2a-11ea-8df3-43e76bae5b96.jpg" width="50%">

주말에는 오롯이 집필에 시간을 투자하였습니다. 지난 일 년여 간은 더 좋은 책을 위해 주말을 포기할 수 밖에 없었습니다.

<img src="https://user-images.githubusercontent.com/1250095/87857527-767b6e00-c962-11ea-98de-bad7188f4dbe.jpg" width="50%">

한 문제라도 틀리지 않기 위해 같은 문제도 수십번씩 반복해서 풀이하였습니다.

<img src="https://user-images.githubusercontent.com/1250095/87857245-387d4a80-c960-11ea-89e6-6f867719999b.jpg" width="60%">

그림 하나도 허투루 그리고 싶지 않았습니다. 유료앱도 구매하고, 유료 그림 강좌도 수강해보았지만 아쉽게도 그림은 결과가 좋지 못했습니다.

<img src="https://user-images.githubusercontent.com/1250095/87857240-34512d00-c960-11ea-9f62-65439a0070b4.jpg" width="60%">

다행인 것은 최고의 작가를 알고 있었다는 점이었고, 제가 그린 똥손 그림은 전문가의 손길을 통해 작품으로 거듭날 수 있었습니다.

<img src="https://user-images.githubusercontent.com/1250095/87868470-d7da2600-c9d0-11ea-81b6-72369ccc94a2.jpg" width="60%">

그 결과, 책에는 최고의 모습으로 실릴 수 있게 되었습니다.

<img src="https://user-images.githubusercontent.com/1250095/87881725-1bb34680-ca36-11ea-95f8-cc7f3e903168.jpg" width="80%">

<img src="https://user-images.githubusercontent.com/1250095/87880669-819bd000-ca2e-11ea-9408-db5e57861de3.jpg" width="50%">

탈고를 하면 모든게 끝인줄 알았습니다. 그러나 마지막까지 치열한 교정 작업을 거쳐야만 했습니다. 이 과정에서 전체 문제를 다시 한 번 풀이하고, 틀린 부분이 없나 꼼꼼히 살펴보았습니다.

<img src="https://user-images.githubusercontent.com/1250095/87880732-05ee5300-ca2f-11ea-9ee0-5100aef2f20c.jpg" width="80%">

드디어 인쇄 작업에 들어가는 모습입니다.

<img src="https://user-images.githubusercontent.com/1250095/87880779-71382500-ca2f-11ea-97e1-01e34d2b0197.jpg" width="80%">

이제 이쁜 모습으로 책으로 탄생할 준비를 하고 있습니다.

<img src="https://user-images.githubusercontent.com/1250095/87880903-6d58d280-ca30-11ea-85ca-398a745fcd35.jpg" width="60%">

마침내 전국 교보문고 매장에서 만나볼 수 있게 됐습니다. 교보문고 광화문점에 전시되어 있는 모습입니다.

<img src="https://user-images.githubusercontent.com/1250095/88868148-58f3b180-d24a-11ea-9354-d800910b066e.jpg" width="60%">

뿐만 아니라 컴퓨터 분야 종합베스트 1위에 올라 교보문고 베스트셀러 코너 1위에 놓여 있습니다.

<img src="https://user-images.githubusercontent.com/1250095/88133338-c38a6900-cc1c-11ea-874d-7d23e93ebc63.jpg" width="60%">

온라인에서도 4대 서점 판매 1위를 석권하며 순항 중에 있습니다. 여전히 초급 실용서가 판매 순위 상위를 차지하는 국내 도서 시장에서는 매우 고무적인 일입니다. 무엇보다 《파이썬 알고리즘 인터뷰》로 인해 우리나라에 기본기가 튼튼한 좋은 개발자가 더 늘어날 수 있다면 그것으로 소임을 다한 것에 여한이 없습니다. 나아가 이 책이 여러분의 커리어에 좋은 길잡이가 되길 기원합니다.

## SNS
《파이썬 알고리즘 인터뷰》를 소개한 SNS 글을 소개합니다.
<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Fzzsza%2Fposts%2F2706265252808533&width=500" width="500" height="636" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Fkanghun.ahn%2Fposts%2F3541612815906406&width=500" width="500" height="670" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Fphoto.php%3Ffbid%3D10159100101597125%26set%3Da.10150688345107125%26type%3D3&width=500" width="500" height="777" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Fsstrato.kong%2Fposts%2F3208482612569863&width=500" width="500" height="452" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Fresious%2Fposts%2F3411057615620193&width=500" width="500" height="790" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Ftobyilee%2Fposts%2F10219441469561987&width=500" width="500" height="834" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

<iframe src="https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Ffupfin.geek%2Fposts%2F1443546749367637&width=500" width="500" height="739" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>

## 서평에 대한 답글
<img src="https://user-images.githubusercontent.com/1250095/89048945-74b2a100-d38b-11ea-85ec-65226dfa2f07.png" width="60%">

요즘은 독자분들이 인터넷 서점에는 리뷰를 잘 올리지 않는 것 같습니다. 개인적으로 안타깝게 생각하며, 그러다 보니 몇 안되는 댓글로 인해 잘못된 정보를 접할 수 있어 예스24에 올라온 댓글에 대해 이 자리를 통해 답변을 드립니다. 《파이썬 알고리즘 인터뷰》는 《코딩 인터뷰 완전 분석》을 짜깁기 한 책이 아닙니다. 지난 수 년간 직접 실험하고 풀이한 내용, 그 동안의 면접관으로서의 오랜 경험을 정리하여 직접 집필한 책입니다. 동일한 '코딩 인터뷰'를 주제로 했다는 점을 제외하면 《파이썬 알고리즘 인터뷰》는 《코딩 인터뷰 완전 분석》과는 완전히 다른 책이며, 한글로 된 더 좋은 책을 만들기 위해 최선을 다한 책입니다. 몇 페이지만 열어봐도 완전히 다른 책임은 금방 알아차릴 수 있을텐데, 단순히 포맷이 유사하다는 이유로 저자와 출판사의 지난 수 년간의 노력을 헛되이 하는 이런 근거 없는 댓글은 매우 안타깝습니다.

## YouTube
저자의 책 소개 영상입니다.
<iframe width="560" height="315" src="https://www.youtube.com/embed/fNyGHpSWhTA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

네이버 클로바 리더이신 김성훈 교수님과 인터뷰한 내용입니다.
<iframe width="600" height="338" src="https://www.youtube.com/embed/51CmqUdLGe4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

이외에도 《파이썬 알고리즘 인터뷰》에서 제공하는 문제는 [Chris AI Holic 유튜브 채널](https://www.youtube.com/user/ChrissSong)을 운영하고 있는 송호연님의 유튜브를 통해 풀이를 영상으로 보실 수 있습니다.
- 문제 풀이 채널: <https://bit.ly/algorithm-interview-youtube>

실제로 온라인 코딩 테스트 시에 어떤 식으로 풀이하는지 영상을 통해 살펴볼 수 있습니다. 전문가는 어떻게 풀이 알고리즘을 설계하고, 어떤 식으로 문제를 풀이하고, 어떤 풀이 기법들을 사용하는지 유심히 살펴본다면 실제로 코딩 인터뷰 시에 문제를 제대로, 잘 풀이하는 데 많은 도움이 될 것입니다.

아울러 몇 가지 [코딩 테스트 문제를 저자가 직접 함께 풀이하는 영상](https://likejazz.tumblr.com/post/630859218303500288/유튜브에는-이미-네이버-카카오-코딩-테스트-기출-문제-풀이-영상들이-많이-있는데요)을 소개합니다.
<iframe width="560" height="315" src="https://www.youtube.com/embed/H5RMk2cOz88" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

> 유튜브에는 이미 네이버, 카카오 코딩 테스트 기출 문제 풀이 영상들이 많이 있는데요. 하지만 중요한 부분은 건너 뛰거나 미리 정답을 준비해서 빠르게 풀이하는 영상들이 많아서, 실제로 많은 분들에게 도움이 될 수 있도록 처음부터 차근차근 풀이하는 영상을 한 번 만들어 봤습니다.
>
> - 1편 영상: <https://youtu.be/8DrkNmN9TGM>  
> - 2편 영상: <https://youtu.be/H5RMk2cOz88>
>
> ** 알림: 원래 훨씬 더 문제를 잘 풀이하는데 일반인을 위한 코딩 테스트 문제 풀이 컨셉에 맞게 질의 응답 형태로 각본대로 진행하였습니다. 행여나 잘못된 오해 없으시길 바랍니다. **
>
> 차근차근 풀다보니 영상이 길어져서 두 개로 나누었습니다. 1부에서는 문제에 대한 설명과 프로그래머스를 이용한 문제 풀이 방법, 실제 코딩 테스트시 문제가 풀리지 않을때 대처 방안 등을 설명하고 List로 문제를 풀이하였습니다. 2부에서는 1부에 이어 보다 효율적인 자료형, OrderedDict와 deque를 활용하여 문제를 풀이하는 방법을 소개하였습니다.

## 학습 노트
《파이썬 알고리즘 인터뷰》를 보면서 정말 열심히 공부하고, 코드로 풀이하고 계신 분의 깃헙을 소개해드립니다. 

<img src="https://user-images.githubusercontent.com/1250095/90315196-75832f80-df54-11ea-828a-0c114c156a99.png" width="70%">

- 깃헙 소개: <https://github.com/jiukeem/python_algorithm/tree/master/02_leetcode>

코드 뿐만 아니라 주석에도 꼼꼼하게 학습 노트를 기입해두셨네요. 정말 알차게 책을 보면서 열심히 공부하고 계시네요. 책을 보면서 공부하는 다른 분들도 이 곳의 코드와 주석을 참고하면 많은 도움이 될 것 같습니다. 적극 추천합니다!

## 판매처
![책](/images/2020/book-cover.jpg)  
《파이썬 알고리즘 인터뷰》는 다음 판매처에서 구매하실 수 있습니다.
- [교보문고](https://bit.ly/3feaYxi)
- [YES24](https://bit.ly/3iGw8Gw)
- [알라딘](http://aladin.kr/p/2fU2N)
- [인터파크](https://bit.ly/3f9SyOg)

및 전국 교보문고 매장

## 참고
문제 풀이 코드는 깃허브, 책에 대한 상세한 소개는 출판사 블로그를 참고하세요.  
- [onlybooks/algorithm-interview - GitHub](https://github.com/onlybooks/algorithm-interview)
- [파이썬 알고리즘 인터뷰 - 책만](https://www.onlybook.co.kr/entry/algorithm-interview)
