# TouchEn nxKey: 키로깅 방지를 위해 키로깅하는 솔루션

```
이 글은 영문으로 된 원문을 저자의 허락을 받고 한국어로 번역한 글입니다.. 
번역 오류 및 잘못된 내용에 대한 수정은 Pull Request를 부탁드려요.
다음 글도 번역에 참여하실 분은 코멘트 남겨주세요 (혼자 다 하려니 쉽지 않네요 ^^)
```

###### 번역관련 정보
- 아래 글 번역에 참여한 이: 
[@alanleedev](https://github.com/alanleedev)
[@MerHS](https://github.com/MerHS)

- 원글 작성일: 2023-01-09 [TouchEn nxKey: The keylogging anti-keylogger solution0](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/)
- 번역글 최초 작성일: 2023-01-15 (PR통해 계속 업데이트 중)

------
일주일 전에 [한국에서 필수설치를 해야하는 소위 보안 프로그램이라 불리는 것](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/)에 대해 글을 썼다. 이 여정은 라온시큐어에서 만든 TouchEn nxKey부터 관심을 갖게되면서 시작되었고, 이 브라우저 확장기능을 천만명이 넘게 사용하고 있어서이다. 사실 크롬 웹 스토어에서 보여주는 다운로드 수는 최대 천만이지만, 대한민국에서 사용되는 대다수의 컴퓨터에 설치되기 때문에 실제 사용자 수는 훨씬 더 많을 것으로 추정된다.

이렇게나 사용자가 많은 것은 사람들이 이 플러그인을 너무 좋아해서가 아니며, 실제로 평이 매우 안 좋아 별점 5점 만점에 평균 1.3점을 받았고 많은 사용자들이 플러그인을 없애버리라고 요청하고 있다. 하지만 한국에서 온라인 뱅킹 등을 하려면 해당 플러그인의 사용이 필수적이다.


TouchEn nxKey를 설치하도록 유도하는 은행에서는 이 플러그인을 통해 보안을 향상한다고 주장한다. 하지만 사용자들은 이것을 [악성 소프트웨어 (malware) 혹은 키로거(keylogger) 라고 부른다](https://www.reddit.com/r/korea/comments/9qwucv/comment/e8ch6yn/). 시간을 투자해 이 제품의 내부 동작을 분석해보니 사실 후자의 주장이 진실에 더 가까웠다. 애플리케이션에는 키로깅 기능이 설계되어 있지만 외부에서 이 기능을 접근하는 것을 제대로 막고 있지 않다. 거기에 더헤 간단한 서비스 거부부터 원격 코드 실행을 비롯한 여러 버그들도 산재한다. 이를 모두 포함하여 나는 7가지 보안 취약점에 대해 신고하였다.

#### 목차

-   [배경](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EB%B0%B0%EA%B2%BD)
-   [TouchEn nxKey는 실제 어떻게 동작하는가?](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#touchen-nxkey%EB%8A%94-%EC%8B%A4%EC%A0%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80)
-   [웹사이트는 어떻게 TouchEn nxKey와 통신하는가?](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EC%9B%B9%EC%82%AC%EC%9D%B4%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-touchen-nxkey%EC%99%80-%ED%86%B5%EC%8B%A0%ED%95%98%EB%8A%94%EA%B0%80)
-   [TouchEn 확장 기능을 악용하여 은행 웹사이트 공격하기](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#touchen-%ED%99%95%EC%9E%A5-%EA%B8%B0%EB%8A%A5%EC%9D%84-%EC%95%85%EC%9A%A9%ED%95%98%EC%97%AC-%EC%9D%80%ED%96%89-%EC%9B%B9%EC%82%AC%EC%9D%B4%ED%8A%B8-%EA%B3%B5%EA%B2%A9%ED%95%98%EA%B8%B0)
    -   [Side-note: TouchEn과 유사한 브라우저 확장기능](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#side-note-touchen%EA%B3%BC-%EC%9C%A0%EC%82%AC%ED%95%9C-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%ED%99%95%EC%9E%A5%EA%B8%B0%EB%8A%A5)
-   [웹사이트에서 키로깅 기능 사용하기](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EC%9B%B9%EC%82%AC%EC%9D%B4%ED%8A%B8%EC%97%90%EC%84%9C-%ED%82%A4%EB%A1%9C%EA%B9%85-%EA%B8%B0%EB%8A%A5-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
-   [애플리케이션 자체를 공격하기](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%9E%90%EC%B2%B4%EB%A5%BC-%EA%B3%B5%EA%B2%A9%ED%95%98%EA%B8%B0)
-   [도우미 (helper) 애플리케이션 악용하기](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EB%8F%84%EC%9A%B0%EB%AF%B8-helper-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%95%85%EC%9A%A9%ED%95%98%EA%B8%B0)
-   [드라이버의 키로깅 기능을 직접 접근하기](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84%EC%9D%98-%ED%82%A4%EB%A1%9C%EA%B9%85-%EA%B8%B0%EB%8A%A5%EC%9D%84-%EC%A7%81%EC%A0%91-%EC%A0%91%EA%B7%BC%ED%95%98%EA%B8%B0)
    -   [Side-note: 드라이버가 죽다 (crash)](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#side-note-%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84%EA%B0%80-%EC%A3%BD%EB%8B%A4-crash)
-   [과연 문제가 수정될까?](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#%EA%B3%BC%EC%97%B0-%EB%AC%B8%EC%A0%9C%EA%B0%80-%EC%88%98%EC%A0%95%EB%90%A0%EA%B9%8C)
    -   [Side-note: 정보 누수 (information leak)](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#side-note-%EC%A0%95%EB%B3%B4-%EB%88%84%EC%88%98-information-leak)
-   [nxKey에 적용된 개념이 제대로 동작할까?](https://github.com/alanleedev/KoreaSecurityApps/blob/main/01_touchen_nxkey.md#nxkey%EC%97%90-%EC%A0%81%EC%9A%A9%EB%90%9C-%EA%B0%9C%EB%85%90%EC%9D%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C)


## 배경


[한국의 현재 상황에 대해서 요약한 글](https://palant.info/2023/01/02/south-koreas-online-security-dead-end/)을 쓴 후 다양한 한국 웹사이트에서 내 글에 대해서 토론하기 시작하였다. [특히 댓글 하나](https://www.clien.net/service/board/news/17827726?c=true#140131396)에서 내가 놓쳤던 꽤 중요한 정보를 발견하였는데 2005년에 한국외환은행에서 발생했던 해킹 사건이었다 [[1]](https://news.kbs.co.kr/news/view.do?ncd=735696) [[2]](https://news.kbs.co.kr/news/view.do?ncd=735697). 이 기사는 기술적인 내용에 대해서 자세히 서술하지 않았지만 이 사건에 대해 내가 이해한 바를 설명하도록 하겠다.


해당 사건은 2005년 당시의 한국에선 매우 중요했던 것으로 보인다. 사이버 범죄집단이 5000만원 (당시 약 5만 달러)을 고객의 은행계좌에서 원격 접속 트로전([Remote Access Trojan](https://www.malwarebytes.com/blog/threats/remote-access-trojan-rat))을 사용해서 빼돌렸고, 이를 통해 고객의 계정 정보뿐만 아니라 보안카드에 대한 정보도 같이 얻었다. 내가 알기로는 보안카드는 유럽연합 (EU)에서 2차 인증절차로 사용하던 indexed TAN과 유사하며, 유럽연합에서는 2012년에 정확히 같은 이유로 트로전을 통해 쉽게 뚫리기 때문에 폐기되었다.


어떤 경로를 통해 고객의 컴퓨터가 악성 프로그램에 감염이 되었을까? 설명에 따르면, 브라우저의 보안 취약점을 공략하는 악성 웹사이트에 방문했을 때에 감염이 되는 드라이브 바이 다운로드([drive-by download](https://en.wikipedia.org/wiki/Drive-by_download))가 사용된 것으로 보인다. 아니면 사용자가 직접 애플리케이션을 설치하도록 속였을 수도 있다. 사용된 브라우저의 이름은 명시되어 있지 않지만 인터넷 익스플로러일 것이 뻔하다. 당시 한국에서 다른 브라우저는 거의 사용되지 않았기 때문이다.


위 기사는 고객이 온라인 뱅킹 정보를 분실하거나 누구에게 준 적이 없으므로 고객은 잘못한 것이 없다고 강조하고 있다. 대신 온라인 뱅킹 자체의 전반적인 안전성에 대해서 의문을 제시하고, 은행이 충분한 수준의 보안조치를 구현하지 않았다고 비판하였다.


2005년 당시 다른 나라에서도 유사한 이야기들이 많이 나돌았다. 이러한 문제가 완전히 사라졌다고 말하기는 어렵지만 오늘날에는 찾아보기 어렵다. 웹 브라우저들의 보안이 많이 향상되기도 하였지만, 한편으로는 은행에서 더 나은 2차 인증 방법을 도입하였기 때문이다. 적어도 유럽에서는 2차 인증 장치가 있어야 거래를 완료할 수 있으며, 거래 승인 시에 거래의 세부내역을 볼 수 있기 때문에 실수로 악성 거래를 승인하기는 힘들다.

하지만 그 당시의 대한민국은 다른 길을 걸었고, 분노한 대한민국의 대중들은 빠른 해결책을 요구하였다. 위의 두 번째 기사가 당시의 문제점을 조명하고 있는데, 보안 애플리케이션을 통해 공격을 막을 수는 있지만 유저는 이를 필수적으로 실행할 필요는 없었고 은행은 이를 허용했다. 결과적으로 은행은 "해킹 방지" 애플리케이션을 유저에게 제공하고 고객들이 이를 필수적으로 실행하도록 하기로 하였다.

그렇기 때문에 TouchEn Key에 대한 첫번째 정보가 2006/2007년도 경에 나오기 시작하는 것은 결코 우연이 아니었다. TouchEn Key는 웹페이지에 취급에 주의가 필요한 정보를 입력할 때 이를 보호해 줄 수 있다고 주장한다. 시간이 지나 마이크로소프트 유래의 브라우저가 아닌 타 브라우저의 지원이 추가되었고, 내가 살펴본 제품도 이것이다.


## TouchEn nxKey는 실제 어떻게 동작하는가?

대중에 공개된 모든 TouchEn nxKey에 대한 자료는 해당 어플리케이션이 키로거를 방지하기 위해 모종의 방법으로 키보드 입력을 암호화한다고 주장한다. 내가 찾을 수 있는 기술적인 내용은 이것이 전부였기에 동작 방식을 스스로 알아내야 했다.


TouchEn nxKey에 의존하는 웹사이트는 nxKey SDK를 사용하는데, 이는 웹사이트에서 실행되는 자바스크립트 코드와 서버에서 실행되는 코드 2가지로 나뉘어져 있다.

동작 방식은 다음과 같다.

1.  유저가 nxKey SDK를 사용하는 웹사이트에서 암호를 입력한다.
2.  nxKey SDK 내의 자바스크립트 코드가 이를 인식하고 로컬에서 실행 중인 nxKey 애플리케이션에 알려준다.
3.  nxKey 애플리케이션이 윈도우 커널 상의 디바이스 드라이버를 활성화한다.
4.  디바이스 드라이버가 모든 키보드 입력을 가로채 시스템이 입력을 처리하는 것을 막는다. 키보드 입력은 nxKey 애플리케이션으로 전송된다.
5.  nxKey 애플리케이션은 키보드 입력을 암호화하여 nxKey SDK의 자바스크립트 코드로 보낸다.
6.  자바스크립트 코드는 암호화된 데이터를 숨겨진 폼의 필드에 입력한다. 실제 암호 입력란은 더미 텍스트만을 받는다.
7.  로그인 정보 입력을 마치고 "로그인" 을 클릭한다.
8.  암호화된 키보드 입력 정보가 다른 데이터와 함께 서버로 보내진다.
9.  서버 상에서 동작하는 nxKey SDK는 데이터를 복호화해 평문으로 된 암호를 얻고 일반적인 로그인 절차로 넘어간다.


웹사이트에 입력된 데이터를 가로채려는 키로거는 암호화된 데이터만 볼 수 있다는 이론이다. 웹사이트가 사용하는 공개키를 볼 수는 있지만, 거기에 상응하는 비공개키를 얻을 수는 없다. 그러니까 복호화는 불가능하고 암호는 안전할 것이다.


그렇다. 이론적으로는 꽤 괜찮은 아이디어이다.


## 웹사이트는 어떻게 TouchEn nxKey와 통신하는가?


웹사이트는 어떻게 특정 애플리케이션이 컴퓨터에 설치가 되어 있는지 알 수 있을까? 그리고 어떻게 그것과 통신을 하는 것일까?


이와 관련되서는 패러다임(역자 주: 구현 혹은 동작방식)의 변화가 일어나고 있는 것처럼 보인다. 원래 TouchEn nxKey는 자체 브라우적 확장기능을 설치하는 것이 필수였다. 브라우저 확장 기능이 웹사이트의 요청 내용을 [native messaging](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Native_messaging) 을 사용해서 전달 하였다. 


하지만 브라우저 확장기능을 사용하는 것은 더 이상 최신 방식이 아니다. 현재 방식은 웹사이트가 웹소켓 API ([WebSockets API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API))를 사용해 애플리케이션과 직접 통신하는 것이다. 더 이상 브라우저 확장 기능이 필요하지 않다.


[busanbank.co.kr 웹시아트는 touchenex_nativecall()을 사용해서 TouchEn 브라우저 확장기능과 통신하는 것으로 보인다. 이 확장 기능은 Native Messaging을 통하여 CrossEXChrome이라는 애플리케이션과 통신한다. 다른 한편으로 citibank.co.kr 웹사이트는 웹소켓 주소127.0.0.1:34581을 사용해 CrossEXService 애플리케이션과 직접 통신한다. ](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/website_communication.png)


이런 동작방식의 변화가 언제 발생했는지 모르겠지만 아직 완성과는 거리가 멀다. 한국 씨티은행 같은 경우 웹소켓을 사용한 방식만을 취하고 있지만 부산은행과 같은 다른 웹사이트는 아직 브라우저 확장 기능만을 사용하는 옛방식을 유지하고 있다.


이것은 사용자가 여전히 브라우저 확장 기능을 설치해야한다는 것만을 의미하지 않고 소프트웨어에 대한 잦은 불만에 대해서 전혀 대응하지 않고 있다는 의미도 된다. 여기 사용자들은 웹소켓 통신을 지원하지 않는 옛날 버전의 소프트웨어를 사용하고 있다. 자동업데이트 기능도 없다. 일부 은행은 옛날 버전을 다운로드 하게 하고 있는데 나도 처음에 잘 모르고 옛날 버전을 다운로드하였다.


## TouchEn 확장 기능을 악용하여 은행 웹사이트 공격하기


TouchEn 브라우저 확장 기능은 매우 작고 기능도 최소한의 것만 있다. 그렇기 때문에 악용할 소지가 별로 없어야 하지만 코드를 보면 다음과 같은 코멘트가 있다.


```js
result = JSON.parse(result);
var cbfunction = result.callback;

var reply = JSON.stringify(result.reply);
var script_str = cbfunction + "(" + reply + ");";
//eval(script_str);
if(typeof window[cbfunction] == 'function')
{
  window[cbfunction](reply);
}
```


어떤 사람이 무언가를 하기 위해 매우 나쁜 방식 (사실 위험하다는 뜻)의 설계를 하였다. 그리고 나서 `eval()`을 사용하지 않고도 기능 구현이 가능한 것을 깨달았거나 누군가 그것을 지적한 것처럼 추정된다. 하지만 나쁜 코드를 삭제하는 대신에 혹시 몰라서 코멘트에 살려두었다. 솔직히 이것은 자바스크립트, 보안, 버전관리에 대해 제대로 된 이해가 부족하다고 객인적으로 해석된다. 나라면 코드를 작성한 사람에게 감독없이 보안 소프트웨어를 만들지 못하도록 할 것이다.


어찌되었던 위험한 `eval()` 호출은 이미 브라우저 확장 기능에서 제거되었다.  하지만 은행 웹사이트에서 사용하는 nxKey SDK의 자바스크리트에서 완전히 제거되지 않았고 아직 큰문제가 발생하지는 않았다. 이렇게 낮은 품질의 코드를 보면 다른 문제들도 더 많을 것으로 예상된다.


역시 유사한 문제를 콜백처리 방식에서도 발견하였다. 웹사이트에서 이벤트 등록을 위해 애플리케이션에게 `setcallback` 요청을 호출할 수 있는데 애플리케이션은 확장기능에 해당 페이지에 등록된 콜백을 호출할 수 있도록 지시한다. 간단히 말하면 해당 페이지에 있는 어떤 글로벌 함수도 이름으로 호출 할 수 있다.


만약 악성 웹사이트가 다른 웹사이트를 대신해 콜백을 등록할 수 있을까? 이것을 위해서는 두가지 장애물이 있다:


1.  목표 웹페이지가 `id="setcallback"` 를 포함한 앨리먼트 (element)를 가지고 있어야 한다.
2.  콜백은 특정 탭으로만 전달된다.


첫 번째 장애물은 nxKey SDK를 사용하는 웹사이트만을 공격할 수 있다는 의미이다. 브라우저 확장 기능을 통해서 통신한다면 이것을 통해 필요한 앨리먼트를 생성할 수 있다. 웹소켓을 사용한 통신은 이 앨리먼트를 생성하지 않는다. 즉, 더 최신 nxKey SDK를 사용하는 웹사이트는 영향이 없다는 뜻이다.


두 번째 장애물은 현재 브라우저 탭에서 로딩된 페이지만 공격할 수 있다는 뜻이다 (예. frame 안에 로딩된 페이지). 예외적으로 nxKey 애플리케이션을 속여서 응답에 잘못된 `tabid`를 설정하지 않는다면.


이 공격방식은 생각보다 매우 쉬웠다. 애플리케이션은 제대로된 JSON 파서를 사용해 입력되는 데이터를 처리하지만 응답은 [sprintf_s()](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/sprintf-s-sprintf-s-l-swprintf-s-swprintf-s-l) 를 호출해서 생성한다. 이스케이프 처리(escaping)를 하지 않는다. 그렇기 때문에 응답에 포함된 요소(property)를 조작해서 물음표를 추가해서 임의의 JSON 요소를 삽입할 수 있다.

```js
touchenex_nativecall({
  …
  id: 'something","x":"y'
  …
});
```

`id` 요소를 애플리케이션의 응답에 복사해 넣을 수 있다. 이것은 응답에 갑자기 없더 JSON요소인 `x`가 추가된다는 뜻이다. 이 취약점을 통해서 `tabid`에 아무 값이나 넣을 수 있다.


그렇다면 악성 페이지는 은행 웹사이트를 접속한 브라우저 탭을 어떻게 알 수 있을까? 자신의 tab ID (TouchEn 확장기능에서 노출해 줌)를 사용해서 다른 tab ID를 추측해볼 수 있다. 아니면 그냥 공란으로 남길 수도 있다. 이럴 경우 브라우저 확장 기능이 의도치 않은 도움을 준다.


```js
tabid = response.response.tabid;
if (tabid == "")
{
  chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    chrome.tabs.sendMessage(tabs[0].id, response, function(res) {});
  });
}
```


`tabid`의 값이 비었다면 현재 활성화된 탭에 메시지를 전달하다.


이것으로 다음과 같은 공격이 가능할 수 있다:
1.  은행웹사이트를 새 탭에서 연다. 그럼 그것이 활성화된 탭이 된다.
2.  해당 페이지가 로딩되도록 기다린다 `id="setcallback"` 앨리먼트가 존재할 수 있도록.
3.  TouchEn 확장기능을 통해 `setcallback` 메시지를 보내어 어떤 함수를 콜백으로 등록하면서 동시에 다음과 같은 JSON 요소인 `"tabid":""` 와 `"reply":"malicious payload"`를 덮어쓴다. 


첫번째 콜백 호출은 즉시 일어난다. 그렇게되면 은행 웹사이트가 콜백 함수를 호출하면서 파라미터로 악성 응답에 포함된 `reply` 요소를 파라미터로 사용한다.


거의 끝까지 왔으니 조금만 더 따라오시라. 가능한 콜백 함수로 `eval`을 사용할 수 있지만 마지막 장애물이 있다. TouchEn이 콜백으로 넘기기 전에 `reply`요소를 `JSON.stringify()`로 처리한다. 그래서 실제로는 `eval("\"malicious payload\"")` 을 받게되고 아무 동작도 하지 않는다.


다른 한 편으로 만약 목표 페이지가 jQuery를 사용한다면? `$('"<img src=x onerror=alert(\'Hi,_this_is_JavaScript_code_running_on_\'+document.domain)>"')` 를 호출하면 원하는 결과를 얻을 수 있다.


![gbank.busanbank.co.kr says: Hi,_this_is_JavaScript_code_running_on_busanbank.co.kr](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/xss.png)


공격이 성공하기위해 jQuery가 있어야 한다는 가정은 비현실적일까? 그렇지 않다. TouchEn nxKey를 사용하는 웹사이트는 TouchEn Transkey (화상 키보드)도 같이 사용할 가능성이 높고 이 기능은 jQuery를 사용하고 있다. 전반적으로 대한민국 은행 사이트들은 jQuery에 대한 의존도가 높아 보이는데 이것은 [좋지 않다 (bad idea)](https://palant.info/2020/03/02/psa-jquery-is-bad-for-the-security-of-your-project/).


하지만 nxKey SDK에서 사용하는 지정된 콜백인 `update_callback`을 통해 JSON 문자열로 변환된 임의의 자바스크립트 코드를 실행하도록 악용할 수 있다. 다음을 호출할 경우  `update_callback('{"FaqMove":"javascript:alert(\'Hi, this is JavaScript code running on \'+document.domain)"}')`,  `javascript:` 링크로 리다이렉트 시도를 할 것이면 부수효과로 임의의 코드를 실행할 수 있다. 

![gbank.busanbank.co.kr says: Hi, this is JavaScript code running on busanbank.co.kr](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/xss2.png)


이 공격으로 악성 웹사이트가 TouchEn 확장기능을 사용하는 임의의 웹사이트를 탈취(compromise)할 수 있다. 그리고 한국의 은행에서 필수로 설치하도록 하는 어떤 "보안" 애플리케이션도 이 공격을 감지하거나 막을 수 없다.


### Side-note: TouchEn과 유사한 브라우저 확장기능

내가 테스트를 시작했을 무렵 크롬 웹스토어에는 TouchEn 확장 기능이 2개가 있었다. 대체적으로 유사하지만 사용자가 더 적은 확장기능은 이 후로 삭제되었다.


이것이 이야기의 끝은 아니다. 이것과 유사한 확장기능 3개를 더 발견했다:  이니세이프 (INISAFE)에서 만든 CrossWeb EX 와 Smart Manager EX 그리고 이니라인(iniLINE)이 만든 CrossWarpEX가 있다.


내 처음 예상은 라온시큐어와 이니세이프가 같은 동일한 그룹내 계열사로 생각했다. 하지만 그래 보이지는 않는다.


그리고 나서 iniLINE 소프트웨어 개발회사의 [이 페이지](http://www.iniline.co.kr/about/about_04.jsp)를 보게되었다.


![이니테크 및 라온시큐어의 로고를 보여주는 페이지](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/partners.png)


이 목록에서는 이니테크와 라온시큐어가 파트너로 명시되어있다. 이로 보건대 이니라인이 문제가되는 브라우저 확장 기능의 개발사로 보인다. 또 하나의 흥미로는 사실은 국방부에 명시된 주요 고객 목록에 첫 업체로 나온다. 적어도 국방관련된 소프트웨어는 다른 파트너들에 건네 주는 코드보다는 더 높은 품질의 코드이길 바란다.


## 웹사이트에서 키로깅 기능 사용하기

만약 악성 웹사이트가 있다고 가정하자. 그리고 그 웹사이트에서 TouchEn nxKey에 다음과 같이 이야기한다고 하자: "안녕, 사용자가 암호 입력란에 있어, 그가 입력하는 데이터를 받고 싶어". 그렇다면 그 웹사이트가 모든 키보드 입력을 가져갈 수 있을까?


그렇다. 사용자가 입력하는 모든 값을 가져간다. 어떤 브라우저 탭이 활성화 되어 있든 아니면 브라우저가 비활성화 되어 있다고 해도 그렇다. nxKey 애플리케이션은 단순히 요청을 그대로 들어준다. 그 요청이 말이 되는지는 전혀 확인하지 않는다. 사실 [User Access Control prompt](https://en.wikipedia.org/wiki/User_Account_Control) 에 입력된 관리자 암호까지 넘겨줄 수 있다.


하지만 넘어야할 장애물이 있긴하다. 먼저 해당 웹사이트는 유요한 라이센스가 필요하다. 그 라이센스를 `get_versions` 호출시 넘겨줘야 하고 이건 애플리케이션의 어떤 기능을 사용하기 전에 먼저 일어나야한다.


```js
socket.send(JSON.stringify({
  "tabid": "whatever",
  "init": "get_versions",
  "m": "nxkey",
  "origin": "https://www.example.com",
  "lic": "eyJ2ZXJzaW9uIjoiMS4wIiwiaXNzdWVfZGF0ZSI6IjIwMzAwMTAxMTIwMDAwIiwicHJvdG9jb2xfbmFtZSI6InRvdWNoZW5leCIsInV1aWQiOiIwMTIzNDU2Nzg5YWJjZGVmIiwibGljZW5zZSI6IldlMkVtUDZjajhOUVIvTk81L3VNQXRVd0EwQzB1RXFzRnRsTVQ1Y29FVkJpSTlYdXZCL1VCVVlHWlY2MVBGdnYvVUJlb1N6ZitSY285Q1d6UUZWSFlCcXhOcGxiZDI3Z2d0bFJNOUhETzdzPSJ9"
}));
```


이 라이센스는 `www.example.com`에서만 유효하다. 그럭기 때문에  `www.example.com` 웹사이트에서만 사용이 가능하다. 아니면 `www.example.com`라고 주장하는 웹사이트이거나.


위 코드에서 `origin` 속성이 보이는가? TouchEn nxKey는 HTTP 헤더의 `Origin`을 확인하는 대신에 위에 주어진 값을 그대로 받아들인다. 그래서 nxKey를 사용하는 어떤 웹사이트에서 라이센스를 가져온 다음 그 웹사이트인 것처럼 행세할 수 있다. 가짜 라이센스를 만들 필요조차 없다.


다른 장애물: 악성 웹사이트에서 받은 데이터는 암호화 되지 않나? 그걸 어떻게 복호화 할까? 비공개키가 알려진 별도의 공개키를 사용하는 것이 가능하다. 그렇다면 암호화 알고리즘만 알면 데이터를 복호화 할 수 있다.


하지만 이 중 어느것도 필요하지 않다. TouchEn nxKey가 공개키를 받지 않는다면 암호화 자체를 하지 않는다. 웹사이트는 평범한 텍스트 (clear text)로 키보드 입력값을 받는다.


내가 만든 PoC (proof of concept; 개념증명) 페이지이다 (HTML 보일러플레이트를 포함해도 3 kB 미만이다)


![웹페이지 스크린 샷: 이페이지는 당신이 다른 애플리케이션으로 입력하는 것을 다 알 수 있다. 아무 애플레케이션에서 입력한 내용이 여기 뜨는 것을 확인해 보라: I AM TYPING THIS INTO A UAC PROMPT](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/typing_page.png)


여전히 세번째 장애물이 있지만 이건 취약점의 심각성을 오히려 약화한다. 악성 웹페이지에서 가로챈 키보드 입력은 원래 목적지에 전달되지 않는다. 사용자가 암호를 입력하기 시작했는데 아무것도 입력된 것이 안보이면 뭔가 의심을 할 것이다. 내가 nxKey 애플리케이션을 분석한 결과 키보드 입력은 웹페이지로 가거나 아니면 원래 목표점으로 전달되지만 절대 둘 다 동시에 전달되지는 않는다.


## 애플리케이션 자체를 공격하기

앞에서 살펴본바로는 이 제품의 자바스크립트 코드를 쓴 사람이 언어를 능숙히 다루지는 못한 것 같다. 어쩌면 작성자의 전문분야가 C++ 이라면?  우린 이런 문제를 [전에도 본적이 있다](https://palant.info/2018/11/30/maximizing-password-manager-attack-surface-leaning-from-kaspersky/). 개발자가 가능한 빨리 자바스크립트를 벗어나서 모든 동작을 C++코드로 위임하려는 것을.


아쉽게도 이건 내가 확인할 수 없는 추측일 뿐이다. 난 바이너리 코드보다는 자바스크립트를 분석하는게 훨씬 익숙하다. 하지만 애플리케이션 자체도 유사한 문제들이 많아 보인다. 사실 C++ 보다는 C에서 많이 사용하는 기법들을 사용한다. 그래서 수동으로 메모리관리를 하는 부분이 많다.


앞에서 이미 `sprintf_s()` 사용에 대해선 언급했다. `sprintf_s()` 나`strcpy_s()` 같은 함수의 흥미로운 점은 이들이 `sprintf()` or `strcpy()` 의 “memory safe” 버전이라 버퍼 오버플로우를 방지하지만 여전히 사용하기 까다로운 함수이다. 만약 충분히 큰 버퍼를 할당하지 않는다면 invalid parameter handler를 호출하고 이건 디폴트로 애플리케이션을 죽인다 (crash).


그걸 아는가: nxKey 애플리케이션은 버퍼 크기가 충분한지 거의 대부분 확인하지 않는다. 그리고 디폴트 동작을 변경하지도 않는다. 그리서 아주 큰 값을 보내면 대부분 애플리케이션이 죽어버린다. 버퍼오버플로우보다 애플리케이션이 죽는 것이 낫지만. 애플리케이션이 죽어버리면 원래 의도한 동작을 하지 못한다. 그것의 일반적인 결과로 온라인 뱅킹 로그인은 정상 동작하는 것처럼 보이지만 암호를 평범한 문자열(clear text)로 받는다. 입력폼을 제출할 때 에러가 나면서 그제야 뭔가 이상하다는 것을 발견한다. 이 취약점을 활용해서 서비스 거부 공격([Denial-of-Service attacks](https://owasp.org/www-community/attacks/Denial_of_Service))을 할 수 있다.


다른 예: 여러 JSON 파서가 있음에도 불구하고 nxKey 개발자는 C언어로 만들어진 파서를 채택했다. 그 뿐 아니라 2014년에 있던 임의의 버전을 가져와서 사용하고 그 후로 업데이트를 하지도 않았다. 2014년에 수정된 이 [널포인터 참조오류](https://github.com/json-parser/json-parser/commit/dec8f04414d2b4a754b2309147665ef341c5f90b)가 여전히 존재한다. 그래서 JSON 데이터 대신에 `]` (괄호닫기) 문자 하나를 보내면 애플리케이션이 죽는다. DoS 공격을 허용하는 또 하나의 취약점인 셈이다.


웹사이트가 접속하는 웹소켓 서버의 경우 OpenSSL을 사용한다. 어떤 OpenSSL 버전일까? OpenSSL 1.0.2c 이다. 모든 보안 전문가들이 한 숨 쉬는 소리가 여기서 들리는 듯 하다. OpenSSL 1.0.2c 는 7년전 버전이다. 사실 1.0.2 브랜치에 지원은 3년 전인 2020년 1월 1일에 종료되었다. 해당 버전의 마지막 릴리즈는 OpenSSL 1.0.2u 이며 이것은 앞의 버전보다 18번 더 버그 및 보안문제 수정을 거쳤다는 뜻이다. 이것이 모두 nxKey 애플리케이션에서는 빠져있다.


애플리케이션이 죽는 것보다 좀 더 흥미로운 내용을 살펴보자. 위에서 언급한 애플리케이션 라이센스는 base64로 인코딩된 데이터이다. 애플리케이션은 이것을 디코딩을 해야한다. 디코딩 함수는 아래와 유사하다:


```c
size_t base64_decode(char *input, size_t input_len, char **result)
{
  size_t result_len = input_len / 4 * 3;
  if (str[input_len - 1] == '=')
    result_len--;
  if (str[input_len - 2] == '=')
    result_len--;
  *result = malloc(result_len + 1);

  // Decoding input in series of 4 characters here
}
```

이 함수를 어디서 가져왔는지는 모르겠지만 CycloneCRYPTO 라이브러리의 base64디코더와 유사하다. 하지만 CycloneCRYPTO는 결과를 사전할당된 버퍼에 쓴다. 여기의 버퍼 할당 로직은 nxKey 개발자가 추가했을 수도 있다.


하지만 그 로직에 문제가 있다. `input_len`이 4의 배수라고 가정하고 있다. `abcd==`와 같은 입력값의 경우 코드에 의해 2바이트 버퍼가 할당이 되지만 실제 아웃풋은 3바이트 크기이다.


한 바이트의 힙 오버플로(heap overflow)를 악용할 수 있는가? 그렇다 여기 블로그 글 [Project Zero blog post](https://googleprojectzero.blogspot.com/2014/08/the-poisoned-nul-byte-2014-edition.html) 혹은 [this article by Javier Jimenez](https://sensepost.com/blog/2017/linux-heap-exploitation-intro-series-the-magicians-cape-1-byte-overflow/) 에서 설명하는 것처럼 악용가능하다. 그러한 취약점을 공격하는 코드를 작성하는 것 내 스킬을 벗어나는 일이다.


대신의 내 PoC는 랜덤으로 생성된 라이센스 문자열을 nxKey 애플리케이션에게 보냈다. 이것 만으로도 애플리케이션을 몇 초만에 죽일 수 있었다. 디버거를 연결해보면 메모리 오류(corruption)의 명확한 증거들을 확인할 수 있었다. 애플리케이션이 죽은 이유는 가짜 주소에 읽기나 쓰기를 시도했기 때문이다. 어떤 경우에 이 메모리 주소는 내 웹사이트가 제공한 데이터에서 왔다. 그래서 분명히 충분한 기술과 열정이 있는 사람이라면 취약점을 이용해 원격 코드 실행을 할 수도 있다.


현대 운영체제는 버퍼오버플로를 기반의 코드 실행 취약점을 악용하는 것을 더 어렵게 만든다. 하지만 관련 기능을 사용했을 때 그렇다는 이야기다. nxKey 개발자는 애플리케이션이 로딩하는 2개의 DLL에서 [Address space layout randomization](https://en.wikipedia.org/wiki/Address_space_layout_randomization)  기능을 껐다. 4개의 DLL에서는 [Data Execution Prevention](https://learn.microsoft.com/en-us/windows/win32/memory/data-execution-prevention) 기능을 껐다.


## 도우미 (helper) 애플리케이션 악용하기


여기 까지는 웹기반 공격에 대한 내용이다. 하지만 악성 프로그램이 이미 시스템에 설치가 되어있고 자기 권한을 확장하기 위한 방법을 찾고 있다면 어떨까? TouchEn nxKey는 이런 악성 프로그램을 막고 프로그램의 기능을 다른 곳에 사용하지 못하도록 하는데는 엉망이다.


nxKey가 키보드 입력을 가로챌 때마다 `CKAgentNXE.exe` 라는 도우미 애플리케이션이 실행된다. 이것의 목적은  nxKey가 키입력을 처리하려고 하지 않을 때에 목표 애플리케이션에 그 데이터가 전달될 수 있도록 한다. 주 애플리케이션에서 사용하는 `TKAppm.dll` 라이브러리의 로직은 다음과 같다:

```c
if (IsAdmin())
  keybd_event(virtualKey, scanCode, flags, extraInfo);
else
{
  AgentConnector connector;

  // An attempt to open the helper’s IPC objects
  connector.connect();

  if (!connector.connected)
  {
    // Application isn’t running, start it now
    RunApplication("CKAgentNXE.exe");

    while (!connector.connected)
    {
      Sleep(10);
      connector.connect();
    }
  }

  // Some IPC dance involving a mutex, shared memory and events
  connector.sendData(2, virtualKey, scanCode, flags, extraInfo);
}
```

nxKey 애플리케이션이 사용자 권한으로 실행되므로  대부분 셋업에서  `CKAgentNXE.exe`를 실행하게 될 것이다. 그리고 도우미 애플리케이션이 명령어 코드 2를 받으면 [SendInput()](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput)를 호출한다.


왜 이런 동작을 하는지 이해하는데는 시간이 좀 필요했다. nxKey나 `CKAgentNXE.exe` 나 동일한 권한으로 실행되고 있는데 왜 직접`SendInput()`를 호출하지 않는가? 왜 이렇게 우회하는게 필요한가?


`CKAgentNXE.exe` 이 IPC 오브젝트의 보안 디스크립터에 integrity level이 Low인 프로세스에 접근하게 설정하는 것을 보게 되었다. 또한 설치 프로그램이 레지스트리에 다음 항목을 등록해 `CKAgentNXE.exe`의 권한을 자동으로 높힌다: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\Low Rights\ElevationPolicy`
그것을 보면서 깨닫게 되었다: 이 모든 것은 인터넷 익스플로러의 샌드박스 (sandbox) 때문이구나.

TouchEn이 인터넷 익스플로러 상 액티브 X로 실행될 때 integrity level은 Low이다. 이렇게 샌드박스에 갖혀있으면 `SendInput()`를 호출하는 것은 불가능하다. 이 제한을 우회하기 위해 인터넷 익스플로러 샌드박스에서 `CKAgentNXE.exe` 을 실행하는 것을 허락하고 자동으로 승급을 한다. 도우미 애플리케이션이 실행되면 샌드박스 내의 액티브X 컨트롤로 연결이 가능하고 뭔가 해달라고 요청이 가능하다. 예를 들면 `SendInput()`호출과 같이.


인터넷 익스플로러 밖에서는 이런 방식이 말도 안되지만 TouchEn nxKey는 작업을 `CKAgentNXE.exe`에게 위임한다. 이것은 보안에 부정적인 결과를 가져온다.


integrity level이 Low인  상태로 실행되는 악성 소프트웨어가 있다고 치자. 아마도 브라우저의 취약점을 이용해서 설치가 됐을 것이지만 샌드박스에 갖혀있다. 이것이 무엇을 할 수 있을까? `CKAgentNXE.exe`이 실행되기만을 기다리면 제약을 벗어날 수 있다.


내 PoC 애플리케이션이 `CKAgentNXE.exe`에게 가짜 키보드 입력을 생성하도록 요청했다. 윈도우키 그리고 C, M, D 그리고 엔터키. 이것으로 Middle integrity level(디폴트 값)로 동작하는 커멘트라인 프롬트가 열렸다. 진짜 악성 프로그램이면 임의 명력을 쳐서 샌드박스 외부에서 코드가 실행이 가능하다.


실제로 악성 소프트웨어가 그렇게 쉽게 보이는 방법을 쓰지는 않을 것이다. `CKAgentNXE.exe`는 커맨드 코드 5를 받기도 하면 이것으로 임의의 DLL을 프로세스내에 로딩한다. 그것이 시스템을 감염시키기에는 더 좋은 방법이다, 그렇게 생각히지 않는가?


적어도 이번에는 필수 설치 보안 애플리케이션 하나가 좀 유용하게 동작해서 위험을 감지했다:


![AhnLab Safe Transaction 애플리케이션이 C:\Temp\test.exe 다음에 감염되었다고 경고한다 Malware/Win.RealProtect-LS.C5210489](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/antivirus_warning.png)


악성 소프트웨어 작성자는 아마도 어떤 방법으로 이 경고가 발생하는지 알아내서 우회방법을 찾을 수 있을 것 이다. 아니면 웹 소켓 연결을 시작해 `CKAgentNXE.exe`이 실제 은행 웹사이트가  동작하 듯 안랩 애플리케이션이 시작하지 않도록 할지도 모른다. 하지만 왜 그렇게 번거롭게 할 필요가 있을까? 경고 메시지일 뿐이고 공격을 능동적으로 막지는 않는다. 사용자가 악성 소프트웨어 제거 버튼을 누르면 때는 이미 늦었다. 공격은 이미 성공했다.


## 드라이버의 키로깅 기능을 직접 접근하기


위에서 언급한 것처럼 TouchEn nxKey 애플리케이션 (드라이버에서 받는 키보드 입력을 암호화하는 녀석)은 
사용자 권한으로 실행이 된다. 높은 권한으로 실행되는 것이 아니므로 특별한 권한이 있지 않다. 그렇다면 드라이버의 기능은 어떻게 제한할까?


정답은 물론 하지 않는다 이다. 시스템 상 어떤 임의의 애플리케이션도 이 기능을 사용할 수 있다. 단지 nxKey가 드라이버와 어떻게 통신하는지만 알면 된다. 여러분이 궁금해 하겠지만 그 통신 프로토콜은 아주 복잡하지는 않다.


무슨 생각으로 이렇게 만든지 잘 모르겠다. `TKAppm.dll`은 드라이버 통신을 하는 라이브러리로 Themida를 통해서 난독화(obfuscation)를 한다. Themida를 만든 업체는 다음과 같이 약속한다:

> Themida® 는 SecureEngine® 보호기술을 사용하며 최상위 레벨로 실행될 때 이전에 보지 못한 보호 기법을 사용해서 고난위도 소프트웨어 크래킹으로부터 보호한다.


Maybe nxKey developers thought that this would offer sufficient protection against reverse engineering. Yet connecting a debugger at runtime allows saving already decrypted `TKAppm.dll` memory and load the result into Ghidra for analysis.


어쩌면 nxKey 개발자들은 이것으로 충분히 리버스 엔지니리어링 막을 수 있다고 생각했을지 모르겠다. 하지만 런타임시 디버거를 연결하면 이미 해독된 (decrypted) `TKAppm.dll` 메모리를 저장할 수 있으면 그 결과를 분석을 위해 Ghidra에서 로딩할 수 있다.


![Message box titled TouchEn nxKey. The text says: Debugging Program is detected. Please Close Debugging Program and try again. TouchEn nxKey will not work with subsequent key. (If system is virtual PC, try real PC.)](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/debugging.png)


미안하지만 너무 늦었다. 내가 필요한 건 이미 가져갔다. 안전모드 부팅시 애플리케이션이 동작을 거부하는 것도 소용이 없다.


어찌됐든 난 아주 작은 (70라인 미만) 애플리케이션을 만들어 드라이버에 연결할 수 있고 이것으로 시스템상의 모든 키보드 입력을 가로챌 수 있다. 


가장 좋은 점: 이 키로거는 nxKey 애플리케이션과 잘 연동이 된다. nxKey는 키보드 입력을 받아 암호화 하고 암호화된 데이터로 웹사이트로 보낸다. 그리고 내가 만든 작은 키로거도 동일한 키보드 입력을 평범한 텍스트 (clear text)로 받는다.


### Side-note: 드라이버가 죽다 (crash)


커널 드라이버 개발 시 알아야 할 점이 하나 있다.:드라이버가 죽으면 시스템 전체가 죽는다. 그래서 드라이버 오류가 발생하지 않도록 추가적인 확인이 필요하다.


nxKey가 사용하는 드라이버에 오류가 발생할 수 있을까? 아주 자세히 살펴보지는 않았지만 우연히 그럴 수 있다는 것을 발견했다. 애플리케이션은  [DeviceIoControl()](https://learn.microsoft.com/en-us/windows/win32/api/ioapiset/nf-ioapiset-deviceiocontrol) 를 사용해서 입력 버퍼의 포인터를 달라고 요청한다. 드라이버는 이 포인터 생성을 위해 [MmMapLockedPagesSpecifyCache()](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-mmmaplockedpagesspecifycache)를 호출한다.


그렇다. 입력 버퍼는 시스템상 모든 애플리케이션이 볼 수 있다. 하지만 그것이 문제의 핵심은 아니다. 만약 애플리케이션이 포인터를 달라고 재요청하면 어떻게 될까? 드라이버는 단순히 `MmMapLockedPagesSpecifyCache()` 를 한 번 더 호출한다.


이 것을 루프로 20초 정도 시도하니 가상 메모리가 다 소진되어 버려 `MmMapLockedPagesSpecifyCache()` 이 NULL을 리턴한다. 드라이버는 리턴 값을 확인하지 않고 죽어버린다. 빵~ 이제 OS가 자동으로 다시 리부팅을 한다.


이 문제를 가지고 취약점으로 이용해 공격하는 것은 가능해 보이지는 않는다 (첨언: 나는 바이너리  공격에 관해서는 전문가가 아니다) 하지만 쾌쾌한 문제점이긴 하다.


## 과연 문제가 수정될까?


내가 일반적으로는 취약점을 공개할 땐 이미 문제가 수정된 후이다. 유감스럽게 이번은 그렇지 못하다. 내가 알 수 있는 범위에서는 한 문제도 수정이 되지 않았다. 업체가 언제 이 문제들을 고칠지도 모른다. 그리고 사용자에게 어떻게 업데이트를 전달할 지도 모른다. 특히 은행에서 가장 최신 버전에세 3 버전 이전 버전을 아직 배포하고 있다. 이것을 기억하라: 자동 업데이트 기능이 없다.


문제를 보고하는 것도 복잡했다. 라온시큐어가 보안 전문기업이지만 서울 전화번호 하나를 제외하고는 보안문제관련 공개된 연락처가 없다. 한국에 직접 전화해서 거기서 누가 영어를 하는지 물어볼 생각은 없다.


운이 좋게도, KrCERT를 외국인이 사용할 수 있도록 [취약점 보고 양식]((https://www.krcert.or.kr/krcert/contact/vulnerability.do))을 제공하고 있다. 이 폼은 자주 에러가 나서 모든 내용을 처음부터 다시 입력하도록 하기도 하며 어떤 보고는 아무 이유없이 웹 파이어월에 잡히기도 한다. 그래도 보안 연락처를 찾는 것은 내가 할 필요가 없는 다른 사람 일이다.


내가 찾은 취약점 모두를 2022년 10월 4일에 KrCERT에 보고하였다. 그 외에도 라온시큐어 임원들에 직접 연락시도를 했지만 응답을 받지 못했다. 그래도 KrCERT에서 내 보고 내용을 라온시큐어에 전달했다고 대략 2주 후에 연락받았다. 그리고 라온시큐어에서 나에게 연락하기 위해서 내 이메일 주소를 물어봤다고 전해들었지만 연락을 받은 적이 없다.


이것이 전부이다. 취약점 보고 후 90일 동안 비공개를 유지하는 데드라인은 이미 한 주전에 지났다. TouchEn nxKey 1.0.0.78은 2022년 10월 4일에 에 릴리즈가 되었고 같은 날에 보안 취약점에 대해서 보고하였다. 이 글을 쓰는 시점에서는 여전히 가장 최신 버전이며 여기 기술된 모든 취약점들이 여전히 존재한다. 

**업데이트** (2023-01-10): 라온시큐어에서는 한국매체에 취약점을 수정하였다고 주장하였고 사용자에게 업데이트를 배포할 것이라고 언급하였다. 나는 현재 이 사실에 대해서는 확인할 수 없다. 그 회사의 다운로드 서버에서는 여전히 TouchEn nxKey 1.0.0.78을 배포하고 있다.


### Side-note: 정보 누수 (information leak)

그들이 뭔가 수정을 하고 있다는 것을 내가 어떻게 알 수 있을까? 이전에 한 번도 일어난 적이 없는 일 덕분에 가능했다. 내 PoC를 90일 데드라인 이전에 실수로 공개했다 (이것은 취약점에 기반으로 거의 완성된 공격 프로그램에 접근 가능하다는 뜻이다).


이전에는 보고서에 파일을 직접 첨부했었다. 보안 소프트웨어 때문에 첨부파일이 삭제되거나 파괴되는 일이 종종 일어났다. 그 대신에 문제를 데모하는 데 필요한 파일은 내 서버에 업로드 한다. 내 서버로의 링크는 항상 동작한다. 추가적인 혜택: 나에게 연락하지 않는 업체들이라고 해도 PoC 파일을 접근한 것은 알 수 있다. 즉, 내 보고서가 누구에게 전달이 됐는지 확인 가능하다.


며칠전에 TouchEn nxKeyr관련 파일의 접근 로그를 확인했다. 즉시 구글봇을 볼 수 있었다. 이 파일들이 구글 검색엔진에 색인되어 버렸다.


나는 무작위로 생성된 폴더 이름을 사용하기 때문에 그냥 알아 낼 수 없다. 나는 링크는 업체들과만 공유한다. 그렇다면 업체가 공격 소프트웨어에 링크를 어딘가 공개적으로 올렸을 것이다.


사실 바로 그런 일이 발생했다. 개발 서버를 발견했는데 공개적으로 볼 수 있었고 구글 검색엔진에서 인덱스를 하였다. 아마도 이 서버에서 내 PoC 관련 링크가 존재한 것으로 보인다. 이것을 발견했을 때는 업체에서 복사해서 수정한 버전으로 호스팅하고 있었다.


구글봇의 첫번째 요청은 2022년 10월 17일이다. 그렇다면 90일간 제한 기간이 끝나기 2달 전부터 이미 구글 검색으로 취약점이 검색 가능했다고 봐야할 것이다. 이 파일들은 여러번 접근을 했었고 제품 개발자 만이 접근을 한 것인지는 알기 어렵다.


이 문제를 보고 한 후 개발 서버는 공개 인터넷에서 바로 사라졌다. 하지만 민감한 보안 정보를 이렇게 경솔히 다루는 것은 전에 본 적이 없다.


## nxKey에 적용된 개념이 제대로 동작할까?

TouchEn nxKey관련 어려 취약점들을 살펴보았다. 키로거를 방지하기위해 완벽한 키로킹 툴을 개발했지만 그것에 접근을 제한하는 것은 실패했다. 하지만 아이디어는 괜찮다, 그렇지 않은가? 만약 제대로 만들어졌다면 유용한 보안 도구가 될지도 모른다.


그렇다면 궁금한 것은, 지금 막고자하는 키로거는 어떤 레벨로 실행이 되는가? 내가 보기로는 4가지 옵션밖에 없다:


1.  브라우저 내부에서. 온라인 뱅킹 페이지에 악성 자바스크립트 코드가 실행되면 암호를 캡쳐하려고 한다. 그 코드는 웹페이지가 nxKey를 활성화 하는 것을 쉽게 막을 수 있다.
2.  시스템 내부에서 사용자 권한으로. 이 권한으로 충분히 같은 사용자 권한으로 실행되고 있는 `CrossExService.exe`를 죽일 수 있다.
3.  시스템 내부에서 관리자 권한으로. 이 권한으로 충분히 nxKey 드라이버를 언로딩 하고 트로전 버전(trojanized copy)으로 바꿔칠 수 있다.
4.  하드웨어 내에서. 게임 끝. 어떤 소프트웨어 해법을 시도하려고 한다면 행운을 빌어 줄 뿐이다.


nxKey가 어떤 보안 기능을 제공하든 공격자가 nxKey와 그 기능에 대해서 모르고 있는 것에 의존하고 있다. 일반적인 공격은 막을 수 있을지 모르겠지만 한국의 은행이나 정부기관를 목표로 하는 공격을 막는데는 그다지 효과적이지 않을 것 같다.

위의 4개의 레벨 중에서 2번은 _어쩌면_ 고칠 수 있는 가능성이 있어 보인다. `CrossEXService.exe` 가 관리자 권한으로 실행되도록 만들 수 있다.  그렇게 되면 악성 소프트웨어가 이 프로세스를 조작려는 시도를 막을 수 있다. 하지만 이것은 악성 소프트웨어가 사용자의 브라우저에 접근 할 수 없다는 가정하에 유효하다.


현재 제품의 동작방식이 다른 레벨에서 동작하는 악성 소프트웨어를 효과적으로 막을 수 없어 보인다.

------
역자 추가 - 원글에 대한 댓글/토론이 있는 웹사이트:
- 원글 Hacker News: https://www.reddit.com/r/korea/comments/107bpsm/touchen_nxkey_the_keylogging_antikeylogger/
- 원글 Reddit: https://www.reddit.com/r/korea/comments/107bpsm/touchen_nxkey_the_keylogging_antikeylogger/
- 원글 긱뉴스: https://news.hada.io/topic?id=8211

- 루리웹 원문 요약: https://bbs.ruliweb.com/community/board/300143/read/59978508
- 개드립 다른 한국어 번역: https://www.dogdrip.net/456327682
- PGR21: https://www.pgr21.com/freedom/97663, https://www.pgr21.com/freedom/97695
- 클리앙: https://www.clien.net/service/board/park/17839993

- 번역글 긱뉴스: https://news.hada.io/topic?id=8274

관련 기사:
- 이뉴스투데이: http://www.enewstoday.co.kr/news/articleView.html?idxno=1630290
- 매일경제: https://www.mk.co.kr/news/it/10599751
- 연합뉴스: https://www.yna.co.kr/view/AKR20230110073700017
