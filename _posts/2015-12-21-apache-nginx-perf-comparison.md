---
layout: post
title: Apache와 Nginx의 PHP 성능 비교
tags:  ["Network Programming"]
last_modified_at: 2025/07/06 18:23:10
---

<div class="message">
이벤트 방식인 Nginx 는 프로세스/쓰레드 방식인 Apache 에 비해 월등한 성능을 보이는 것으로 알려져 있다. 실제로 Static 파일들 CS, JSS 의 경우엔 두드러져 보이는데, 그렇다면 CGI 도 이에 해당되는지 특히 PHP 의 경우를 예로 들어 살펴본다.
</div>

- [서론](#서론)
- [테스트](#테스트)
  - [성능](#성능)
- [전통적인 CGI](#전통적인-cgi)
- [결론](#결론)

## 서론

PHP 를 웹으로 서빙하는 케이스는 크게 3 종류로 나눌 수 있다.

1. PHP Built-in 웹 서버
1. Apache w/ mod_php
1. Nginx w/ FastCGI

1번의 경우 개발시에 웹 서버를 별도로 셋팅하기 번거로울 경우 쉽게 사용할 수 있는 방법이고, 실제로 흔히 사용한다. 그러나 편의상 사용되며 성능과는 거리가 멀다. 따라서 여기서는 더 이상 언급하지 않기로 한다.

2번의 경우 10 여년 이상 사용해온 전통적인 방식이며 **LAMP 스택(Linux, Apache, MySQL, PHP)**이란 이름으로 지금의 아파치와 PHP 를 있게 한 장본인이다.

3번의 경우 PHP-FPM 과 함께 최근에 주로 쓰이는 방식인데, FPM 은 FastCGI Process Manager 의 약자이며 PHP-FPM 은 PHP FastCGI 의 대안 구현으로 2010년 PHP 코어에 팩키징되고 2011년 실험적인(experiment) 딱지를 뗌으로써 사실상 PHP FastCGI 표준 구현으로 자리 잡았다. 특히 아파치에 비해 월등한 성능을 보이는 Nginx 와 손쉽게 연동되며 최근 **LEMP 스택(Linux, Nginx, PHP, MySQL/MariaDB)**이란 이름으로 새로운 트렌드를 주도하고 있다.

그렇다면 3번 구현은 과연 새로운 트렌드로써 월등한 성능을 보여주고 있는지 직접 확인해보도록 한다.

## 테스트

테스트 서버의 사양은 아래와 같다.

- Intel Xeon E312xx (Sandy Bridge) x 2
- 4G RAM
- OS: Ubuntu 14.04
- Apache, Nginx 모두 apt 를 이용한 기본 설치, 설정 변경 없음

요즘 기준으로 보면 웹 서버로 쓸 수 있는 거의 최소 사양인데 테스트 PHP 코드가 과연 얼만큼의 성능을 보여주는지 확인해본다.

### 성능

코드는 간단한 HTML 을 변수에 삽입한 다음 템플릿을 구성해서 내려주는 형태이며, 별도의 외부 서버나 DB 호출은 하지 않는다.

<img src="https://github.com/user-attachments/assets/fed33f70-9902-402f-937a-c9f2e1429060" width="700" />

**carrotw** 는 사내에서 만든 브라우저로 직접 부하 테스트가 가능한 성능 테스트 도구(외부에 공개되지 않은 사내 시스템이라 부득이하게 주소를 가림)다. 전통적인 쓰레드 방식으로 동작하며 여기서는 쓰레드를 200개 주어 최대 성능을 측정했다. 200개 쓰레드가 동시에 요청을 보내 응답을 받아오는 초당 평균 횟수/속도를 측정하는 방식이며 10초간 측정한 결과는 약 7,600 TPS 에, 평균 응답속도 25ms 수준이다.

아주 좋은 성능의 서버가 아님에도 불구하고 초당 7,600회 처리라는 준수한 성능을 보여준다. 그렇다면 과연 아파치는 Nginx 에 비해 어느 정도 성능이 나오는지 확인해본다.

<img src="https://github.com/user-attachments/assets/fba6b5e7-1a33-401b-bc62-dbce72bd19ae" width="700" />

동일한 200개 쓰레드에서 7,100 TPS, 27ms 가 나왔다. 이를 표로 정리하면 아래와 같다.

<table>
  <thead>
    <tr>
      <th>방식</th>
      <th>TPS</th>
      <th>응답 속도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Nginx w/ FastCGI</td>
      <td>7,600 TPS</td>
      <td>25ms</td>
    </tr>
    <tr>
      <td>Apache w/ mod_php</td>
      <td>7,100 TPS</td>
      <td>27ms</td>
    </tr>
  </tbody>
</table>

단순 TPS 만 비교하면 Nginx 가 아파치 보다 약 7% 정도 성능이 더 좋은것으로 나온다. CSS, JS 등의 Static 파일이 압도적인 성능을 보여주는 것에 비하면 성능이 약간 높긴 하나 다소 실망스런 결과다.

[Why is FastCGI /w Nginx so much faster than Apache /w mod_php?](http://www.eschrade.com/page/why-is-fastcgi-w-nginx-so-much-faster-than-apache-w-mod_php/) 를 보면 비슷한 얘기를 하고 있다. 아파치 설정을 튜닝하고 불필요한 시스템 콜을 없애자 실제로는 비슷한 성능을 보여주며 큰 파일(100KB)인 경우 오히려 아파치가 더 나은 성능을 보여준다고 한다.

## 전통적인 CGI

따라서 CGI(PHP) 용도로 아파치나 Nginx 나 별 차이가 없다는 결론에 이른다. 그러나 전통적인 CGI(Traditional CGI) 방식과는 압도적인 성능 차이가 있음에 유의해야 한다. 아래는 여러가지 언어로 전통적인 CGI 를 구현하고 아파치 cgi-bin 에서 구동한 결과다.


<table>
  <thead>
    <tr>
      <th>방식</th>
      <th>TPS</th>
      <th>코드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Bash</td>
      <td>870 TPS</td>
      <td>
{% highlight bash %}
#!/bin/bash
echo "Content-type: text/html"
echo ''
echo '<html>'
echo '<body>'
echo '<h1>CGI Bash Example</h1>'
echo '</body>'
echo '</html>'
{% endhighlight %}
      </td>
    </tr>
    <tr>
      <td>Python</td>
      <td>190 TPS</td>
      <td>
      {% highlight python %}
#!/usr/bin/python

print "Content-type: text/html\n\n"
print '''<html>
<body>
<h1>CGI Python Example</h1>
</body>
</html>'''
      {% endhighlight %}
      </td>
    </tr>
    <tr>
      <td>C</td>
      <td>1,580 TPS</td>
      <td>
      {% highlight c %}
#include <stdio.h>

int main(void) {
  printf("Content-Type: text/html \n\n");
  printf("<html>\n");
  printf("<body>\n");
  printf("<h1>CGI C Example</h1>\n");
  printf("</body>\n");
  printf("</html>\n");

  return 0;
}
      {% endhighlight %}
      </td>
    </tr>
    <tr>
      <td>C++</td>
      <td>1,100 TPS</td>
      <td>
      {% highlight cpp %}
#include <iostream>
using namespace std;

int main(void) {
  cout << "content-type: text/html" << endl << endl;
  cout << "<html>" << endl;
  cout << "<body>" << endl;
  cout << "<h1>CGI C++ Example</h1>" << endl;
  cout << "</body>" << endl;
  cout << "</html>" << endl;

  return 0;
}
      {% endhighlight %}
      </td>
    </tr>
    <tr>
      <td>PHP</td>
      <td>250 TPS</td>
      <td>
      {% highlight php %}
#!/usr/bin/php
<?php
echo "Content-type: text/html\n\n";
echo <<<EOF
<html>
<body>
<h1>CGI PHP Example</h1>
</body>
</html>
EOF
?>
      {% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

예상했듯이 C 가 가장 빠르고 그 다음 C++ > Bash > PHP > Python 순이다. 전통적인 CGI 방식은 프로세스를 직접 실행한 결과를 보여주는 방식이기 때문에 미리 바이너리를 만들고 사이즈가 가장 작은 C 가 가장 빠르다.

그러나 이 역시도 mod_php 에 비하면 1/7 수준에 불과하다. 따라서 특수한 경우를 제외하곤 굳이 C 로 전통적인 CGI 를 만들어야 할 이유가 전혀 없다. 재밌는 점은 PHP 의 경우인데, 전통적인 CGI 에서는 250 TPS 밖에 나오지 않지만 mod_php/FastCGI 에서 돌리면 7,100 ~ 7,600 TPS 가 나온다. 거의 30배 이상 성능 차이가 난다.

## 결론

Static 파일과 달리 CGI(PHP) 방식에서 Apache w/ mod_php 와 Nginx w/ FastCGI 의 성능 차이는 크지 않다. 따라서 각자에게 편한 웹 서버를 사용해도 무방하다. 그러나 전통적인 CGI 방식과는 성능 차이가 매우 크므로 특수한 경우를 제외하곤 전통적인 CGI 방식은 사용하지 않는게 좋다.
