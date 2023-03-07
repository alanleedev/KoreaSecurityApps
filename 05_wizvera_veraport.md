# 베라포트: 제대로 작동하지 않는 한국의 애플리케이션 관리 소프트웨어


* :kr: *번역상태*: 1차 검토/수정 중 (2023-03-06)

```
이 글은 저자의 허락을 받아 영문으로 된 원문을 한국어로 번역한 글입니다.
번역 오류 및 잘못된 내용에 대한 수정은 [참여방법](https://github.com/alanleedev/KoreaSecurityApps/blob/main/CONTRIBUTION.md)을 읽은 후 Pull Request를 부탁드려요.
```

## 번역 관련 정보

- 아래 글 번역에 참여한 이:
  - [@alanleedev](https://github.com/alanleedev)

- 원본 문서: [Veraport: Inside Korea’s dysfunctional application management](https://palant.info/2023/03/06/veraport-inside-koreas-dysfunctional-application-management/) (2023-03-06)
- 번역 문서 최초 작성일: 2023-03-06

---



이전에 이야기한 바와 같이 한국의 은행 웹사이트는 [소위보안 애플리케이션이라 불리는 것들을 설치를 요구](https://palant.info/2023/01/02/south-koreas-online-security-dead-end/)한다. 
동시에 [TouchEn nxKey](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/)나 [IPinside](https://palant.info/2023/01/25/ipinside-koreas-mandatory-spyware/)와 같은 애플리케이션이 자동 업데이트 기능이 부족하다는 것도 보았다.
만약 보안 문제가 발생하더라도 사용자에게 시의 적적하게 업데이트를 전달하는 것이 거의 불가능하다.

앞에 예는 단지 애플리케이션 두 개일 뿐이다.
한국의 은행 웹사이트는 보통 다섯 개 정도의 애플리케이션을 요구하며 웹사이트가 다르면 사용되는 애플리케이션도 다르다. 
설치하고 업데이트해야하는 애플리케이션이 꽤 된다.

다행히도 위즈베라 (Wizvera)에서 만든 베라포트 (Veraport) 애플리케이션이 이런 일을 처리한다.
특정 웹사이트를 사용하기 위해 필요한 모든 애플리케이션을 자동으로 설치해준다.
만약 필요할 경우 업데이트도 설치한다.

![Laptop with Veraport logo on the left, three web servers on the right. First server is labeled “Initiating server,” the arrow going from it to the laptop says “Get policy from banking.example.” Next web server is labeled “Policy server,” the arrow pointing from the laptop to it says “Installation policy?” and the arrow back “Install app.exe from download.example.” The final web server is labeled “Download server” and an arrow points to it from the laptop saying “Give me app.exe.”](https://palant.info/temp/sZ5dD1oX9vE5aP9a/veraport.png)


만약 이것의 권한이 막강하다는 생각이 든다면 실제 그렇기 때문이다.
베라포트는 이미 [북한 해커들의 공격에 사용되었다](https://threatpost.com/hacked-software-south-korea-supply-chain-attack/161257/)고 뉴스에 나온 적이 있다.

그 당시에는 모두가 공격에 뚫린 웹서버에 원인을 돌렸다.
난 베라포트가 어떻게 동작하는지 좀 더 자세히 살펴본 후 이런 결론을 내렸다: 여기서 사용하는 접근방법은 본질적으로 위험하다.

베라포트 3.8.6.5(2월 28일에 릴리즈 됨)을 보면 기존에 신고된 보안 문제들이 해결된 것으로 보인다.
하지만 사용자들이 해당 버전으로 업데이트하는 것은 오랜 시간이 걸릴 것이다.
또한 베라포트의 고객들이 임의의 소프트웨어를 배포할 수 있게 허락하는 것은 여전히 본질적인 위험이다. 

#### 목차

-   [발견한 내용 요약](#발견한-내용-요약)
-   [은행 웹사이트가 애플리케이션을 배포하는 방법](#은행-웹사이트가-애플리케이션을-배포하는-방법)
-   [위즈베라 베라포트 동작방식](#위즈베라-베라포트-동작방식)
-   [악성 정책으로부터 보호](#악성-정책으로부터-보호)
-   [보안 내 구멍](#보안-내-구멍)
    -   [전송 중 데이터 보호 부족](#전송-중-데이터-보호-부족)
    -   [너무 넓게 설정된 allowedDomains](#너무-넓게-설정된-allowedDomains)
    -   [서명키는 누가 가지고 있나?](#서명키는-누가-가지고-있나)
    -   [인증기관](#인증기관)
-   [구멍을 결합한 익스플로잇 (exploit)](#구멍을-결합한-익스플로잇-exploit)
    -   [악성 웹사이트에서 기존 정책 파일 사용](#악성-웹사이트에서-기존-정책-파일-사용)
    -   [악성 바이너리의 실행](#악성-바이너리의-실행)
    -   [시각적 단서 제거](#시각적-단서-제거)
-   [정보 유출: 로컬 애플리케이션](#정보-유출-로컬-애플리케이션)
-   [웹서버 취약점](#웹서버-취약점)
    -   [HTTP 응답 분할 (Response Splitting)](#http-응답-분할-response-splitting)
    -   [서비스 작업자를 통한 영구 XSS](#서비스-작업자를-통한-영구-XSS)
-   [문제 신고하기](#문제-신고하기)
-   [무엇이 고쳐졌나](#무엇이-고쳐졌나)
-   [남은 문제들](#남은-문제들)

## 발견한 내용 요약

베라포트는 어떤 애플리케이션이 어디로부터 설치할지 정하는 정책파일을 서명한다.
여기서 사용된 암호화는 정상적이지만, 접근방법은 몇 가지 문제가 있다:

- 서명 유효성 검사에 여전히 사용되는 루트 인증서 중 하나는 MD5 해싱과 1024비트 RAS키를 사용한다. 이런 인증서는 지원이 중단된지 10년이 넘었다.
- 다운로드를 위한 연결에 HTTPS를 강제하지 않는다. HTTPS가 사용될 때도 서버 인증서를 검증하지 않는다.
- 다운로드한 파일의 무결성을 올바르게 검증하지 않는다. 애플리케이션 서명 유효성 검사는 쉽게 우회할 수 있고 해시 기반 검증이 가능하지만 사용하지 않는다.
- 무결성 검증을 쉽게 우회할 수 없더라도 베라포트는 손상된 바이너리를 사용할 지 여부를 사용자에게 맡긴다.
- 사용자의 선택이나 눈에 보이는 단서가 없이 애플리케이션을 다운로드하고 설치하는 동작을 시작할 수 있다.
- 여전히 개별 웹사이트(예: 온라인 뱅킹)에서 소프트웨어 배포에 대해서 책임지며 종종 이미 알려진 보안 문제가 있는 오래된 애플리케이션을 제공한다.
- 공개적으로 유출된 서명인증서나 악의적인 정책파일을 철회할 수 있는 장치가 없다.

그 외에도 `https://127.0.0.1:16106`에서 동작하는 베라포트의 로컬 웹서버에는 영구적 [Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) 을 비롯한 여러 보안취약접이 존재한다.
이것을 통해 요청하는 어떤 웹사이트에든 사용자 기기에서 실행 중인 프로세스의 전체 노출한다.
보안 애플리케이션의 경우 애플리케이션 버전도 노출한다.

마지막으로 베라포트는 공개된 보안 취약점이 존재하는 오래된 오픈소스  라이브러리 여럿을 사용해서 만들어졌다.
예를 들면, OpenSSL 1.0.2j (2016년에 릴리즈)을 웹서버 및 서명 무결성 검증을 위해 사용한다.
OpenSSL 취약점은 특히 잘 문서화되어 있다. 이 버전의 경우 [최소 3개의 알려진 높은 심각도 취약점과 13개의 알려진 중간 심각도 취약점](https://www.openssl.org/news/vulnerabilities-1.0.2.html)이 있다.

로컬 웹서버는 몽구스 5.5 (2014년에 릴리즈)를 사용한다.
웹사이트로 받는 JSON 데이터의 파싱은 JsonCpp 0.5.0 (2010년 릴리즈)을 사용한다.
그렇다, 거의 13년 전 버전이다.
현재 버전은 JsonCpp 1.9.5으로 여러 보안문제들이 향상되었다.

## 은행 웹사이트가 애플리케이션을 배포하는 방법

한국의 은행에서 사용하는 로그인 웹사이트는 여러 개의 소위 보안 애플리케이션에서 제공하는 자바스크립트 코드를 실행한다. 
각 SDK는 애플리케이션이 사용자의 컴퓨터에 존재하는지 먼저 확인한다.
만약 존재하지 않을 경우 보통은 사용자를 다운로드 페이지로 리다이렉트(redirect)한다

![Screenshot of a page titled “Install Security Program.” Below it the text “To access and use services on Busan Bank website, please install the security programs. If your installation is completed, please click Home Page to move to the main page. Click [Download Integrated Installation Program] to start automatica installation. In case of an error message, please click 'Save' and run the downloaded the application.” Below that text the page suggests downloading “Integrated installation (Veraport)” and five individual applications.](https://palant.info/temp/sZ5dD1oX9vE5aP9a/applications.png)

이것은 소프트웨어 공급업체의 다운로드 페이지가 아니라 은행의 웹페이지이다.
설치가 필수인 여러 애플리케이션 목록을 보여주며 사용자가 다운로드 받기를 기대한다.
보통 은행의 웹서버가 다운로드 서버 역할도 한다.

따라서 각 은행이 배포하는 애플리케이션의 버전이 다른 며 때론 최신 버전보다 몇 년이 뒤쳐진다는 것이 놀랍지도 않다.
또한 오래되어 사용하지 않는 설치 페이지를 찾는 것도 어렵지 않다.
이 페이지에서 애플리케이션을 다운로드 하는 것은 보통 여전히 동작하지만 10년 전버전 일 수 있다.

예를 들면, 몇 주 전까지만 해도 한국씨티은행 웹페이지에서 2020년 버전인 TouchEn nxKey 1.0.0.75(그 당신 최신 버전은 1.0.0.78이었다)를 배포하고 있었다.
하지만 실수로 잘못된 다운로드페이지로 갈 경우 2015년 버전인 TouchEn nxKey 1.0.0.5를 다운로드 받을 것이다.

부산은행의 경우 리눅스나 맥OS 사용자를 위한 소프트웨어 패키지가 있다고 하지만 실제 다운로드 받을 수 없거나 윈도우 소프트웨어를 다운로드 한다.
다운로드 가능한 리눅스 패키지의 경우 2015년 것이며 최신 웹브라우저에서 지원하지 않는 [NPAPI](https://en.wikipedia.org/wiki/NPAPI)에 의존한다.

분명히 사용자들이 이러한 복잡한 문제를 다루기를 기대할 수 없다.
그래서 은행들이 보통 "통합 설치"를 제공한다.
이것은 사용자들이 위즈베라 베라포트 애플리케이션을 다운로드하여 그것이 모든 픽요한 작업을 하도록 하는 것을 뜻한다.


## 위즈베라 베라포트 동작방식

베라포트가 각 애플리케이션의 최신 버전을 어디서 가져올지 언제 업데이트를 할지 안다고 기대했다면 실제는 그렇지 않다.
대신 애플리케이션 설치를 자동화 할 뿐이다.
그것은 사용자들이 하는 것을 정확히 그대로 한다: (보통 은행의 서버에서) 각 애플리케이션을 다운로드하고 설치 프로그램을 실행한다.

![Laptop with Veraport logo on the left, three web servers on the right. First server is labeled “Initiating server,” the arrow going from it to the laptop says “Get policy from banking.example.” Next web server is labeled “Policy server,” the arrow pointing from the laptop to it says “Installation policy?” and the arrow back “Install app.exe from download.example.” The final web server is labeled “Download server” and an arrow points to it from the laptop saying “Give me app.exe.”](https://palant.info/temp/sZ5dD1oX9vE5aP9a/veraport.png)

은행 웹사이트가 `https://127.0.0.1:16106`에서 동작하는 베라포트의 로컬 웹서버에 연결해서 [JSONP 요청](https://en.wikipedia.org/wiki/JSONP)을 보낸다.
`getAxInfo`와 같은 명령어를 사용해 어떤 (일반적으로 자체) 웹사이트에서 설치 정책을 다운로드한다:

```js
send_command("getAxInfo", {
  "configure": {
    "domain": "http://banking.example/",
    "axinfourl": "http://banking.example/wizvera/plinfo.html",
    "type": "normal",
    "language": "eng",
    "browser": "Chrome/106.0.0.0",
    "forceinstall": "TouchEnNxKeyNo",
  }
});
```

이것을 통해 베라포트가 `http://banking.example/wizvera/plinfo.html`에서 정책파일을 다운로드해서 검증한다.
이것은 아래와 유사한 XML파일 형태이다:

```xml
<pluginInstallInfo>
  <version>2.2</version>
  <createDate>2022/02/18 17:04:35</createDate>
  <allowDomains>www.citibank.co.kr;*.citibank.co.kr;cbolrnd.apac.nsroot.net</allowDomains>
  <allowContexts/>
  <object type="Must">
    <objectName>TouchEnNxKeyNo</objectName>
    <displayName>TouchEnNxKey-multi64</displayName>
    <objectVersion>1.0.0.73</objectVersion>
    <signVerify>confirm</signVerify>
    <hashCheck>ignore</hashCheck>
    <browserType>Mozilla</browserType>
    <objectMIMEType>
      file:%ProgramFiles(x86)%\RaonSecure\TouchEn nxKey\TKMain.dll
    </objectMIMEType>
    <downloadURL>
      /3rdParty/raon/TouchEn/nxKey/nxKey/module/
      TouchEn_nxKey_Installer_32bit_MLWS_nonadmin.exe /silence
    </downloadURL>
    <backupURL>
      /3rdParty/raon/TouchEn/nxKey/nxKey/module/
      TouchEn_nxKey_Installer_32bit_MLWS_nonadmin.exe
    </backupURL>
    <systemType>64</systemType>
    <javascriptURL/>
    <objectHash/>
    <extInfo/>
    <policyInfo/>
    <description/>
  </object>
  <object type="Must">
    …
  </object>
  …
</pluginInstallInfo>
```

여기서 TouchEn nxKey을 필수 애플리케이션으로 표시한다.
베라포트는 `objectMIMEType` 항목을 통해서 애플리케이션이 이미 설치되었는지 인식한다.
설치가 안되었을 경우 `downloadURL` 와 `backupURL` 항목을 사용해 설치파일을 다운로드한다.

이 데이터기 처리되면 웹사이트에서 베라포트의 사용자 인터페이스를 열 수 있다:
```js
send_command("show");
```

다음에 수행되는 작업은 이전에 전달된 `type` 설정 매개변수에 의해 결정된다.
"관리(manage)"모드에서는 사용자가 어떤 애플리케이션을 설치할지 선택할 수 있게 해준다.
이미 설치된 애플리케이션을 삭제하는 것도 이론적으로는 가능하지만 내가 시도했을 때 동작하지 않았다.
"일반(normal)" 같은 모드에서는 필수로 간주되는 애플리케이션을 다운로드를 시작하고 설치한다.


## 악성 정책으로부터 보호

베라포트는 시작 서버에 재한을 두지 않는다.
어떤 웹사이트든 로컬 웹서버와 통시할 수 있으므로 어떤 웹사이트든 소프트웨어 설치를 시작할 수 있다.
악성 웹사이트가 이 기능을 악용해 악성 소프트웨어(malware)를 설치하는 것을 무엇이 막는가?

이번에는 (오직) 난독화만 사용하지 않았다.
정책파일은암호화된 서명을 거쳐야 한다.
서명자는 베라포트 애플리케이션 내에 하드코딩된 2개의 인증기관으로부터 검증이 되야한다.
이론상으로는 베라포트의 고객만이 정책파일에 서명할 수 있고 악성 웹사이트는 유효한 정책을 약용하는 것밖에 할 수 없다.

추가적으로 베라포트는 정책파일을 호스팅할 수 있는 웹사이트를 제한한다.
정책파일내에 `allowedDomains` 항목을 사용한다.
내가 알 수 있는 한, 웹주소 구문분석(parsing)은 제대로 동작하고 우회를 허용하지 않는다.

정책파일의 `downloadURL` 와 `backupURL` 에 상대경로가 존재한다면 (매우 일반적임), 정책파일 위치를 기준으로 상대적으로 처리한다.
원칙적으로는 이러한 보안 정책이 결합되어 합법적인 정책을 악용하더라도 신뢰할 수 없는 위치에서 다운로드를 시작할 수 없다고 한다.

## 보안 내 구멍

위의 조치는 기본적인 보호를 제공하지만 악의적인 행위자가 남용할 수 있는 구멍이 많다.

### 전송 중 데이터 보호 부족

베라포트는 일반적으로 HTTPS 연결을 강제하지 않는다.
소프트웨어 다운로드를 포함한 모든 연결은 암호화되지 않은 HTTP를 사용할 수 있다.
실제로 AhnLab Safe Transaction 다운로드에 대해 암호화되지 않은 HTTP 이외의 다른 것을 사용하는 정책을 본 적이 없다.
신뢰할 수 없는 네트워크에 연결되면 다운로드가 악성 소프트웨어(malware)로 대체될 수 있다.

상대 경로를 통해 다운로드되는 애플리케이션과 다를점이 없다.
악의적인 웹 사이트는 정책 파일을 (쉽게) 조작할 수 없지만 암호화되지 않은 HTTP 연결을 통해 정책 다운로드를 시작할 수 있다.
상대 경로로 표시된 모든 다운로드는 암호화되지 않은 HTTP 연결을 통해 다운로드된다.

그렇다고 HTTPS 연결을 일관되게 사용한다고 해서 문제가 완전히 해결되지 않는다.
베라포트를 테스트 했을 때 HTTPS 연결에 대해서도 서버의 신원를 확인하지 않았다.
따라서 `https://download.example`에서 애플리케이션 다운로드가 시작되더라도 신뢰할 수 없는 네트워크에서 악성 서버가 `download.example`로 가장하면 베라포트는 이를 수락한다.

### 너무 넓게 설정된 allowedDomains

위즈베라는 고객에게 서명 키를 제공하고 고객이 직접 정책 파일에 서명하는 것으로 보인다.
이러한 고객에게 특히 'allowedDomains'와 관련하여 보안적으로 안전한 설정을 선택하는 방법에 대한 가이드라인이 제공되는지 모르겠다.

위의 Citibank 예를 보더라도 `*.citibank.co.kr`은 citibank.co.kr의 모든 하위 도메인이 정책 파일을 호스팅할 수 있음을 의미한다.
상대 다운로드 경로와 관련하여 각 하위 도메인은 악성 소프트웨어(malware)를 배포할 가능성이 있다.
그러한 하위 도메인이 많이 있을 것으로 추정하고 [subdomain takeover](https://developer.mozilla.org/en-US/docs/Web/Security/Subdomain_takeovers)를 통해 그 중 일부는 탈취당했을 수 있다.
다른 옵션은 직접 웹사이트를 해킹하는 것으로 [2020년에 이미 발생한](https://threatpost.com/hacked-software-south-korea-supply-chain-attack/161257/) 것으로 보인다.

정책 파일 몇 개만 살펴봤는데 회사 도메인에 대한 포괄적인 화이트리스트(blanket whitelist)가 모든 파일에 존재한다.
하나의 파일은 더 큰 문제를 안고 있다: 여러 허용되는 IP 범위를 목록에 포함한다.
왜 한국 회사가 Abbott 연구소와 미국 국방부에 속한 IP주소 범위를 이 목록에 넣었는지 모르겠지만 그렇게 했다.
127.0.0.1 또한 목록에 포함되어 있다.

### 서명키는 누가 가지고 있나?

당연히 위즈베라 고객은 어떤 정책 파일이든 서명할 수 있다.
따라서 그들이 `example.com` 웹사이트에서 `malicious.exe`를 설치하도록 허용하려는 경우 막을 방법이 없다.
그들은 유효한 서명 인증서만 필요하며 이 인증서의 사용은 그들의 웹사이트로 제한되지 않는다.
Wizvera 웹사이트에는 많은 고객이 나열되어 있다.

![A large list of company logos including Citibank in a section titled “Customers and Partners” in Korean.](https://palant.info/temp/sZ5dD1oX9vE5aP9a/customers.png)

이 모든 고객이 자신에게 부여된 막강한 권한을 인식하고 서명 인증서의 개인 키를 매우 안전한 곳에 보관하기를 바란다.
이 키가 잘못된 손에 넘어가면 악의적인 정책에 서명하는 데 악용될 수 있다.

개인 키가 유출되는 방법에는 여러 가지가 있다.
자주 발생하는 일이지만 회사가 해킹당할 수 있다.
보안이 부족한 소스 코드 저장소 등을 통해 데이터를 유출할 수 있다.
불만을 품은 직원이 뇌물을 받거나 직접 악의적인 사람에게 키를 판매할 수 있다.
아니면 다국적 기업의 경우 어떤 정부 회사에 키를 넘겨달라고 요청할 수 있다.

그런 일이 발생하면 위즈베라는 남용을 방지하기 위해 아무것도 할 수 없다.
서명 인증서가 손상된 것으로 알려진 경우에도 애플리케이션에는 인증서 철회할 장치가 없다.
유효한 서명이 있는 한 알려진 악성 정책을 차단하는 장치도 없다.
손상을 제한하는 유일한 방법은 새로운 베라포트 버전을 배포하는 것이다.
자동 업데이트 기능 없이는 프로세스로는 몇 년이 걸린다.

### 인증기관

이것은 합법적인 서명 인증서의 악용이다.
그러나 누군가가 자신의 서명 인증서를 만들 수 있다면 어떨까?

베라포트가 두 개의 인증 기관을 허릭힌 덕분에 완전히 비현실적인 것은 아니다.

|  Authority name  |         Validity         | Key size | Signature algorithm |
|:----------------:|:------------------------:|:--------:|:-------------------:|
| axmserver        | 2008-11-16 to 2028-11-11 | 1024     | MD5                 |
| VERAPORT Root CA | 2020-07-21 to 2050-07-14 | 2048     | SHA256              |

두 인증 기관 중 오래된 인증 기관이 2020년까지 독점적으로 사용된 것으로 보인다.
1024비트 키와 MD5 서명은 이미 사용이 중단된지 오래이며 브라우저에서는 이러한 인증기관의 단계적인 제거를 [이미 10년전](https://wiki.mozilla.org/CA:MD5and1024)에 시작했다.
그러나 베라포트는 지금 2023년에도 이 인증 기관을 허용하고 있다.

내 지식으로는 아직 아무도 1024비트 RSA 키를 성공적으로 소인수분해 하지 못했다.
누구도 주어진 MD5 서명으로 충돌을 생성하는 데 성공하지 못했다.
그러나 이 두 가지 시나리오는 충분히 실현가능해 보여 이미 10년 이상 전에 브라우저에서 예방 조치를 취했다.

시대가 지난 기술관련해 말인데 [Microsoft의 인증 기관에 대한 요구 사항](https://learn.microsoft.com/en-us/previous-versions//cc751157(v=technet.10)#a-root-requirements)은 다음과 같이 말한다:

> 루트 인증서는 배포 신청일로부터 25년 이내에 만료되어야 한다.

이 요구 사항이 필요한 이유는 인증 기관이 오래될수록 기반 기술이 더 낡아 보안이 뚫릴 가능성이 높기 때문이다.
따라서 베라포트는 새로운 인증 기관이 갖는 30년 수명에 대해서 재고해야 할 수 있다.

## 구멍을 결합한 익스플로잇 (exploit)

악성 웹사이트에서 성공적인 베라포트 익스플로잇(exploit)을 실행할 수 있다.
사용자가 그 곳을 방문하면 사용자 모르게 악성 애플리케이션 설치를 시작한다.
내 개념 증명으로 그것을 보여준다.

### 악성 웹사이트에서 기존 정책 파일 사용

앞에서 언급했듯이 정책 파일은 특정 도메인에서 호스팅해야 한다.
이 제한은 쉽게 우회할 수 없지만 은행 웹사이트를 해킹할 필요도 없다.
대신 네트워크 연결을 신뢰할 수 없는 상황을 고려했다, 예를 들면 개방된 와이파이.

다른 사람의 WiFi에 연결했을 때 그들 소유의 웹 페이지로 연결될 수 있다.
[종속 포털](https://en.wikipedia.org/wiki/Captive_portal)처럼 보일 수 있지만 실제로는 베라포트를 악용하려고 시도한다, 예를 들어 hanacard.co.kr의 기존 서명된 정책 파일을 사용한다.

그리고 그들이 네트워크를 제어하기 때문에 예를 들어 www.hanacard.co.kr이  IP 주소 192.0.2.1을 가지고 있다고 컴퓨터에 알릴 수 있다.
위에서 언급했듯이 베라포트는 192.0.2.1이 실제로 www.hanacard.co.kr이 아니라는 사실을 인식하지 못한다.

따라서 악성 웹사이트는 베라포트를 통해 자동 설치를 시작할 수 있다.

![“Wizvera updater”창. “DelphinoG3 를 설치중…”](https://palant.info/temp/sZ5dD1oX9vE5aP9a/veraport_installing.png)

### 악성 바이너리의 실행

앞에서 언급했듯이 상대 다운로드 경로는 정책 파일을 기준으로 처리된다.
따라서 "DelphinoG3" 애플리케이션은 정책 파일과 마찬가지로 "www.hanacard.co.kr"에서 다운로드되고 있다. 하지만 실제로는 공격자가 제어하는 서버에서 다운로드 된다.

하지만 악성 애플리케이션은 적어도 즉시 설치되지 않는다:

![A message box saying: It’s wrong signature for DelfinoG3 [0x800B0100,0x800B0100], Disregards this error and continues a progress?](https://palant.info/temp/sZ5dD1oX9vE5aP9a/wrong_signature.png)

이러한 암호와 같은 메시지를 사용하면 사용자가 "확인"을 클릭하고 악성 애플리케이션을 실행할 가능성이 높다.
그렇기 때문에 보안에 민감한 결정을 사용자에게 맡겨서는 안된다. 그런데 여기선 어떤 서명을 의미할까?

정책 파일에는 다운로드에 대해 해시 기반 검증을 수행하는 옵션이 있다.
그러나 내가 확인한 모든 웹사이트에서 이 기능은 사용되지 않는다:

```xml
<hashCheck>ignore</hashCheck>
```

이것은 일반적인 코드 서명에 관한 것이다.
그리고 코드 서명이된 악성 소프트웨어에 대해서는 [들어본 적이 있다](https://www.makeuseof.com/tag/what-is-code-signed-malware/).

하지만 서명이 유효하지 않아도 된다!
자체 서명된 실행 파일을 서버에 업로드하려고 했다.
그리고 베라포트는 이것이 아무 문제 제기 없이 실행되도록 허용했다!

![Console window titled “delfino-g3.exe”, on top of it a message box saying “Hi.”](https://palant.info/temp/sZ5dD1oX9vE5aP9a/running.png)

내가 만든 최소한의 "악성" 애플리케이션이 그다지 인상적이지 않다는 것을 안다.
하지만 실제 악성 소프트웨어는 여기서 시스템 깊숙이 파고들 것이다.
어쩌면 파일 암호화를 시작하거나, 사용자을 염탐하거나, 사용자의 돈을 전부 털기 위해 "단순히" 다음 은행 거래를 기다릴 지도 모른다.

이 애플리케이션은 이제 관리자 권한으로 실행되며 시스템의 모든 항목을 변경할 수 있다는 것을 기억하라. 
사용자는 권한 상승 프롬프트(애플리케이션의 유효하지 않은 서명에 대해 경고하는 메시지)를 수락할 필요조차 없었다 – 베라포트 자체가 각 개별 설치 프로그램에 대한 권한 상승 프롬프트를 표시하지 않기 위해 권한상승된 상태로 실행된다.

### 시각적 단서 제거

분명히, 사용자는 갑자기 나타난 베라포트 설치 화면을 보면 걱정할 수도 있다.
운 좋게도 공격자는 해당 설치 화면을 마음대로 설정할 수 있다.

예를 들어 악성 웹 사이트는 다음과 같은 설정을 전달할 수 있다:

```js
  send_command("getAxInfo", {
    "configure": {
      "type": "normaldesc",
      "addinfourl": "http://malicious.example/addinfo.json",
      …
    }
  });
```

`addinfo.json` 파일은 다운로드에 대한 이름과 설명을 임의로 변경할 수 있어서 사용자가 의심을 갖지 않도록 한다:

![Screenshot of the Veraport window listing a number of applications with names like “Not TouchEn nxKey” and “Not IPinside LWS.” Description is always “Important security update, urgent!”](https://palant.info/temp/sZ5dD1oX9vE5aP9a/customized.png)

그러나 텍스트를 조작할 필요조차 없다.
`bgurl` 설정 매개변수는 창 전체의 배경 이미지를 설정한다.
이 이미지의 크기가 잘못된 경우 어떻게 될까?  베라포트 창은 그 크기에 따라 크기가 조정된다.
그리고 그것이 1x1 픽셀 이미지라면?
그럼 보이지 않는 창이된다.
임무 완료: 시각적 단서 없음.

## 정보 유출: 로컬 애플리케이션

베라포트 서버에서 지원하는 흥미로운 명령 중 하나는 'checkProcess' 이다: 애플리케이션 이름을 넘겨주면 현재 실행 중인 경우 프로세스에 대한 정보를 반환한다.
애플리케이션 이름으로 `*`를 넘겨주면 어떻게 될까?

```js
let processes = send_command("checkProcess", "*");
```

사소한 웹페이지의 출력결과:

![The processes running on your computer, followed by a list of process names and identifiers, e.g. 184 msedgewebview2.exe](https://palant.info/temp/sZ5dD1oX9vE5aP9a/installed_applications1.png)

[IPinside를 통한 복잡한 검색](https://palant.info/2023/01/25/ipinside-koreas-mandatory-spyware/#information-about-running-processes)보다 사용자가 어떤 애플리케이션을 실행하고 있는지 확실히 알 수 있는 더 편리한 방법이다.

보안 애플리케이션의 경우 `getPreDownInfo` 명령이 추가 정보를 제공한다.
정책 파일을 처리하고 이미 설치된 응용 프로그램을 확인한다.
여러 웹사이트에서 정책 파일을 가져옴으로써 수십 개의 서로 다른 애플리케이션을 확인하는 개념 증명을 만들었다:

![Following security applications have been found: DelfinoG3-multi64 3.6.8.4, ASTx-multi64 1.4.0.155, TouchEnNxKey-multi64 1.0.0.78, UriI3GM-multi64 3.0.0.17, ASTx-multi64 2.5.0.206, INISAFEWebEX-multi64 3.3.2.36, MAGIC-PKI 22.0.8811.0](https://palant.info/temp/sZ5dD1oX9vE5aP9a/installed_applications2.png)

이 접근 방법으로 버전 번호를 확보하면 악의적인 웹 사이트가 이상적인 공격을 준비할 수 있다: 사용자가 설치한 오래된 보안 소프트웨어를 찾고 악용할 취약점을 선택한다.

## 웹서버 취약점

### HTTP 응답 분할 (Response Splitting)

앞에서 본 것처럼 'https://127.0.0.1:16106' 에서 실행되는 베라포트의 로컬 웹 서버는 여러 명령에 응답한다.
그러나 리디렉션 엔드포인트도 있다: `https://127.0.0.1:16106/redirect?url=https://example.com/`은 `https://example.com/`으로 리디렉션한다.
아니, 이게 무슨 용도인지 모르겠다.

이 엔드포인트를 테스트하면 리디렉션 주소의 유효성 검사가 수행되지 않는 것으로 보인다.
베라포트는 주소에서 줄 바꿈 문자도 기꺼이 허용하므로 [HTTP 응답 분할](https://owasp.org/www-community/attacks/HTTP_Response_Splitting)이 가능하다. 이것은 HTTP 응답을 생성하는 모든 라이브러리들이 헤더 이름이나 값에서 줄바꿈 문자를 금지한 이후로 거의 사라진 취약성 종류이다. 
하지만 Veraport는 그러한 라이브러리를 사용하지 않는다.

따라서 `https://127.0.0.1:16106/redirect?url=https://example.com/%0ACookie:%20a=b`요청으로 다음과 같은 응답을 얻는다:

```
HTTP/1.1 302 Found
Location: https://example.com/
Cookie: a=b
```

HTTP 헤더의 성공적인 인젝션으로 127.0.0.1에 쿠키를 설정하였다 . 그리고 `Location` 헤더를 유효하지 않게 만들어 리디렉션을 방지하고 대신 임의의 콘텐츠를 제공할 수도 있다. `https://127.0.0.1:16106/redirect?url=%0AContent-Type:%20text/html%0A%0A%3Cscript%3Ealert(document.domain)%3C/script%3E` 요청으로 다음 응답을 얻는다:
```
HTTP/1.1 302 Found
Location: 
Content-Type: text/html

<script>alert(document.domain)</script>
```

구글 크롬은 실제로 127.0.0.1 환경에서 이 스크립트를 실행하므로 여기에 반사형(reflected) [교차 사이트 스크립팅(XSS) 취약점](https://en.wikipedia.org/wiki/Cross-site_scripting)이 사용되었다.
반면에 모질라 파이어폭스는 이러한 공격으로부터 보호한다 – 302 응답에 포함된 콘텐츠는 렌더링되지 않는다.

### 서비스 작업자를 통한 영구 XSS

127.0.0.1에서 반사형 XSS는 잠재적인 공격자에게 그다지 유용하지 않다.
그렇다면 이를 영구적인 XSS로 바꿀 수 있을까?
[서비스 워커](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)를 설치하면 가능하다.

서비스 작업자 설치에 대한 장애물은 의도적으로 매우 높게 설정된다. 한 번 보자:

- HTTPS를 사용해야 한다.
- 코드는 대상 범위 내에서 실행되어야 한다.
- 서비스 워커 파일은 JavaScript MIME 형식을 사용해야 한다.
- 서비스 워커 파일은 대상 범위 내에 있어야 한다.

베라포트는 어떤 이유로 로컬 웹 서버에 HTTPS를 사용하며, 해당 환경에서 코드를 실행하는 방법을 이미 찾았다.
따라서 처음 두 조건이 충족된다.
다른 두 가지에 관해서는 HTTP 응답 분할을 사용히야 유효한 MIME 유형을 가진 JavaScript 코드를 얻을 수 있을 것이다. 그러나 더 간단한 방법이 있다.

기억나는가? 베라포트 서버는 [JSONP](https://en.wikipedia.org/wiki/JSONP)를 통해 통신힌다. 
따라서 `https://127.0.0.1:16106/?data={}&callback=hi` 요청은 다음과 같은 자바스크립트 파일을 생성한다:
```js
hi({"res": 1});
```

JSONP의 사용은 권장되지 않으며 매우 오랫동안 그래왔다.
그러나 _만약_ 애플리케이션이 JSONP를 사용해야 한다면 콜백 이름을 검증하는 것이 좋다, 예를 들면 영문 및 숫자만 허용한다.

알아맞춰 보아라: 베라포트는그러한 유효성 검사를 수행하지 않는다.
악의적인 콜백 이름이 JavaScript 데이터에 임의의 코드를 삽입할 수 있음을 의미힌다.
예를 들어 `https://127.0.0.1:16106/?data={}&callback=alert(document.domain)//`으로 다음 자바스크립트 코드를 얻을 수 있다:

```js
alert(document.domain)//hi({"res": 1});
```

그리고 서비스 워커로 등록할 수 있는 임의의 코드가 포함된 JavaScript 파일이 있다
개념 증명에서 서비스 워커는 `https://127.0.0.1:16106/`에 대한 요청을 처리한다.
그런 다음 피싱(phishing) 페이지를 제공한다.

![Screenshot of the browser window titled “Citibank Internet Banking” and showing 127.0.0.1:16106 as page address. The page is a login page titled “Welcome. Please Sign On. More secure local sign-in! Brought you by Veraport.”](https://palant.info/temp/sZ5dD1oX9vE5aP9a/localhost.png)

어떤 경로로 접근했던 관계없이 `https://127.0.0.1:16106/`에 표시되는 내용이다.
서비스 워커는 등록이 취소되거나 다른 서비스 워커로 교체될 때까지 요청을 유지하고 처리하며 브라우저가 다시 시작되고 브라우저 데이터가 삭제되더라도 유지된다.

## 문제 신고하기

내가 찾은 이슈들을 총 6개의 신고서로 정리했다.
다른 한국 보안 애플리케이션과 마찬가지로 [KrCERT 취약성 보고서 양식](https://www.krcert.or.kr/krcert/contact/vulnerability.do)을 통해 이 신고서를 제출했다.

이 양식은 일반적으로 신뢰할 수 없고 종종 오류 메시지를 생성하지만 이번에는 가장 중요한 두 신고서를 곧바로 거부했다. 
신고서 텍스트의 무언가가 웹 애플리케이션 방화벽을 작동했다.

텍스트를 수정하려고 했지만 소용이 없었다.
심지어 이 보고서에만 존재하지만 이전 보고서에는 없는 단어 목록을 수집했지만 여전히 운이 없었다.
결국 2022년 12월 3일 양식을 사용하여 4개의 신고서를 보내고 나머지 2개는 이메일로 신고했다.

이틀 후 이메일을 통해 문제를 이메일로 제출하라는 응답을 받았고 즉시 처리했다.
답장에서 이전 신고서가 여러 번 수신되었음을 언급했다.
취약점 제출 양식에 오류가 발생할 때마다 실제로 신고서를 데이터베이스에 추가하지만 단지 이메일 알림 전송에 실패할 뿐이다.

2023년 1월 5일에 KrCERT는 취약성 양식을 통해 제출된 최소 4개의 신고서를 위즈베라에 전달했을을 통보하였다. 이메일로 제출된 신고서에 관해서는 한동안 연락을 받지 못해 위즈베라가 해당 보고서를 받았는지에 대한 확신이 없었다.

그러나 이것이 내가 KrCERT로부터 들은 마지막이 아니었다.
2월 6일에 그들로부터 버그 바운티 프로그램에 나를 초대하는 요청하지도 않은 이메일을 받았다.

![Screenshot of a Korea-language email from vuln_notice@krcert.or.kr. The “To” field contains a list of censored email addresses.](https://palant.info/temp/sZ5dD1oX9vE5aP9a/spam.png)

그렇다, 모든 수신자의 이메일 주소가 "받는 사람" 필드에 나열되었다.
그들은 이메일을 통해 보안 연구원 740명의 이메일 주소를 유출했다.

2시간 후에 온 사과 이메일에 따르면 그들은 실제로 두 번째 이메일에서도 같은 실수를 저질러 총 1,490명이 영향을 받았다.
이 이메일은 또한 완화 조치를 제안했는데 KrCERT가 속한 정부 기관인 KISA에 사건을 보고하는 것이다.

## 무엇이 고쳐졌나

I did not receive any communication from Wizvera, but I accidentally stumbled upon their download server. And this server had a text file with a full change history. That’s how I learned that Veraport 3.8.6.4 is out:

```
2864,3864 (2023-01-26) 취약점 수정적용
```

Veraport 3.8.6.4를 테스트 했을 때 일부 문제를 해결했지만 다른 문제는 해결하지 못했다.
특히, 악성 애플리케이션 설치는 여전히 최소한의 변경으로 작동했다 - HTTPS 연결이 아닌 HTTP를 통해 정책을 다운로드하기만 하면 된다.

그래서 2월 22일에 KrCERT에 내 의견을 위즈베라에 전달할 것을 요청하는 이메일을 보냈고, 그들은 같은 날 전달해 주었다.
변경 내역에 따르면 그 결과로 Veraport 3.8.6.5에 다양한 추가 변경 사항이 구현되어 2월 28일에 출시되었다.

전체적으로 직접 악용될 수 있는 모든 문제가 해결된 것으로 보인다.
특히 이제 HTTPS 연결에 대해 서버 신원의 유효성을 검사하고 있다.
또한 Veraport 3.8.6.5는 HTTP 다운로드를 HTTPS로 자동 업그레이드를 통해 신뢰할 수 없는 네트워크가 더 이상 설치를 조작할 수 없다.

창 크기는 더 이상 배경 이미지에 의해 결정되지 않으므로 애플리케이션 창을 더 이상 이런 식으로 숨길 수 없다. 
Veraport 3.8.6.5를 사용하면 웹사이트도 더 이상 애플리케이션에 대한 설명을 변경할 수 없습니다.

리디렉션 엔드포인트는 베라포트의 로컬 서버에서 제거되었으며 JSONP 엔드포인트는 이제 콜백 이름을 허용된 문자 집합으로 제한합니다.

OpenSSL은  베라포트 3.8.6.4에서 버전 1.0.2u로, Veraport 3.8.6.5에서 버전 1.1.1t로 업데이트되었다.
후자는 실제로 최신버전이며 알려진 취약점이 없다.

changelog에 따르면 JsonCpp 0.10.7이 현재 사용되고 있다.
이 버전은 2016년에 출시되었지만 애플리케이션이 Visual Studio 2008로 컴파일되는 한 최신 버전을 사용하는 것은 불가능하다.

베라포트 3.8.6.5는 [TLS 보안에 대한 내 블로그 게시물](https://palant.info/2023/02/06/weakening-tls-protection-south-korean-style/)에 언급된 문제도 해결했다.
현재 설치 중에 인증 기관이 생성되고 있다.
그 외에도 응용 프로그램은 TLS 없이 포트 16116에서 통신할 수 있다.

흥미롭게도 'checkProcess'를 악용 해 실행 중인 프로세스를 나열하는 문제도 2021년에 이미 알려진 해결된 문제라는 것도 알게 되었다:

```
2860, 3860 (2021-10-29)
- checkProcess 가 아무런 결과를 리턴하지 않도록 수정
```

나를 위한 변론: 베라포트를 테스트했을 때 최신 버전이 무엇인지 알 수 있는 방법이 없었다.
위즈베라가 보안 문제를 수정하는 릴리스에 대해 고객에게 알린 지 한 달 후인 2023년 3월 1일에도 10개의 웹 사이트 중 3개(Google 검색 결과의 위치에 따름)에서만이 베라포트의 최신 버전을 다운로드할 수 있었다.
이는 기존 사용자가 업데이트했다는 의미가 아니라 사용자가 베라포트를 다시 설치하기로 결정한 경우 이 버전을 받는다는 의미이다. 그러나 이 것만 봐도 웹 사이트 10 개 중 7 개는 몇 년 뒤쳐져있다.

|       Website       | Veraport version | Release year |
|:-------------------:|:----------------:|:------------:|
| busanbank.co.kr     | 3.8.6.0          | 2021         |
| citibank.co.kr      | 3.8.6.4          | 2023         |
| hanacard.co.kr      | 3.8.6.1          | 2021         |
| hi.co.kr            | 3.8.6.0          | 2021         |
| ibk.co.kr           | 3.8.5.1          | 2020         |
| ibki.co.kr          | 3.8.6.4          | 2023         |
| koreasmail.co.kr    | 2.5.6.1          | 2013         |
| ksfc.co.kr          | 3.8.6.1          | 2021         |
| lina.co.kr          | 3.8.5.1          | 2020         |
| yuantasavings.co.kr | 3.8.6.4          | 2023         |

## 남은 문제들

애플리케이션 서명 유효성 검사는 베라포트 3.8.6.4에서 여전히 손상되어있다.
베라포트 3.8.6.5에서도 마찬가지일 것으로 추정되지만 확인이 복잡하다.
이제 연결 무결성을 신뢰할 수 있으므로 더 이상 중요한 문제가 아나다.

`checkProcess`는 더 이상 사용할 수 없지만 `getPreDownInfo` 명령은 베라포트 최신 버전에서 계속 접근할 수 있다.
따라서 어떤 웹사이트든 어떤 보안 응용 프로그램이 설치되어 있는지 계속 확인할 수 있다.
버전 번호만 검열되어 더 이상 사용할 수 없다.

Veraport 3.8.6.5도 여전히 8년 된 mongoose 5.5 라이브러리를 로컬 웹 서버에 사용하는 것으로 보인다.
이 라이브러리는 업그레이드되지 않았다.

물론 개념적인 문제는 해결되지 않았으며 해결하기가 훨씬 더 복잡하다.
베라포트 고객은 여전히 임의의 애플리케이션을 강제로 설치할 수 있는 권한이 있으며, 오래된 악성 소프트웨어를 포함한다.
그리고 그들은 자신의 웹사이트로 제한되지 않으며 모든 웹사이트에 대한 정책 파일에 서명할 수 있다.

베라포트 고객의 손상된 서명 인증서는 여전히 철회할 수 없으며 알려진 악의적인 정책도 철회할 수 없다.
마지막으로 오래된 루트 인증서(1024비트, MD5)가 애플리케이션에 여전히 존재한다.
