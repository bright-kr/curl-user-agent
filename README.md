# cURL에서 User Agent 설정 및 변경 방법

[![Bright Data Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 Web스크레이핑 작업을 개선하기 위해 cURL에서 User-Agent 헤더를 구성하고 수정하는 과정을 안내합니다:

- [User Agents and Their Significance](#user-agents-and-their-significance)
- [The Standard cURL User Agent](#the-standard-curl-user-agent)
- [Methods to Configure the cURL User Agent Header](#methods-to-configure-the-curl-user-agent-header)
  - [Directly Setting a Custom User Agent](#directly-setting-a-custom-user-agent)
  - [Configuring a Custom User Agent Header](#configuring-a-custom-user-agent-header)
- [Creating a User Agent Rotation System in cURL](#creating-a-user-agent-rotation-system-in-curl)
  - [Implementation in Bash](#implementation-in-bash)
  - [Implementation in PowerShell](#implementation-in-powershell)
- [Final Thoughts](#final-thoughts)

## User Agents and Their Significance

User agent는 웹 브라우저, 요청를 생성하는 애플리케이션, HTTP 클라이언트가 요청의 소스 소프트웨어를 공개하기 위해 [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) HTTP 헤더에 포함하는 텍스트 문자열을 의미합니다. 이 식별자에는 일반적으로 브라우저/애플리케이션 유형, 운영체제, 그리고 추가 사양에 대한 정보가 포함됩니다.

다음은 Chrome 브라우저의 일반적인 user agent 문자열 예시입니다:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
```

이 헤더 정보는 요청가 정상적인 브라우저에서 발생했는지, 혹은 다른 소프트웨어 도구에서 발생했는지를 판단하는 데 중요한 역할을 합니다. 자동화된 스크레이핑 도구는 종종 일관성이 없거나 범용적인 user agent 문자열을 사용하여 자동화된 성격을 드러냅니다. 따라서 user agent 헤더는 안티봇 기술이 실제 사용자와 자동화 스크립트를 구분하는 데 도움을 줍니다.

## The Standard cURL User Agent

대부분의 HTTP 클라이언트와 마찬가지로 [cURL](https://brightdata.co.kr/blog/web-data/what-is-curl)은 HTTP 요청를 전송할 때 User-Agent 헤더를 자동으로 포함합니다. 기본 cURL user agent는 다음 형식을 따릅니다:

```
curl/X.Y.Z 
```

여기서 X.Y.Z는 설치된 cURL 버전을 의미합니다.

이를 확인하려면, 요청자가 전송한 User-Agent 헤더를 반환하는 [httpbin.io](https://httpbin.io/)의 /user-agent 엔드포인트로 테스트 요청를 보내면 됩니다:

```sh
curl "https://httpbin.io/user-agent"
```

> **Note**:
> Windows 사용자는 curl 대신 curl.exe를 사용해야 합니다. 이는 PowerShell에서 curl이 [Invoke-WebRequest](https://brightdata.co.kr/blog/how-tos/powershell-invoke-webrequest-with-proxy)의 별칭(alias)으로 동작하는 반면, curl.exe는 실제 cURL Windows 실행 파일을 가리키기 때문입니다.

응답는 다음과 유사하게 표시됩니다:

```json
{
  "user-agent": "curl/8.4.0"
}
```

보시다시피 cURL은 자신의 버전(curl/8.4.0)을 user agent로 설정합니다. 이는 요청가 cURL에서 비롯되었음을 명확히 식별하기 때문에 문제가 됩니다. 콘텐츠를 보호하도록 설계된 웹사이트 보호 메커니즘은 이러한 요청를 비인간(non-human)으로 쉽게 플래그 처리하여 차단할 수 있습니다.

따라서 cURL user agent 헤더를 커스터마이징하는 것이 중요합니다.

## How to Configure the cURL User Agent Header

cURL에서 user agent를 수정하는 방법은 두 가지 접근 방식이 있습니다.

### Directly Setting a Custom User Agent

cURL은 이를 위한 전용 옵션을 제공합니다. 구체적으로, [\-A or –user-agent](https://curl.se/docs/manpage.html#-A) 플래그를 사용하면 cURL 요청에서 User-Agent 헤더의 내용을 정의할 수 있습니다.

이 옵션으로 cURL user agent 헤더를 설정하는 문법은 다음과 같습니다:

```sh
curl -A|--user-agent "<user-agent_string>" "<url>"
```

예시는 다음과 같습니다:

```sh
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

위 명령에 대한 출력은 다음과 같습니다:

```json
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
}
```

위 명령은 다음의 긴 형태와 동일합니다:

```sh
curl --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

일반적으로 권장되지는 않지만, `-A`에 빈 문자열을 전달하여 User-Agent 헤더를 완전히 제거할 수도 있습니다. 이를 확인하려면, 수신 リクエ스트의 모든 헤더를 표시하는 httpbin.io의 [/headers](https://httpbin.io/headers) 엔드포인트에 접근하면 됩니다:

```sh
curl -A "" "https://httpbin.io/headers"
```

결과는 다음과 같이 표시됩니다:

```json
{
  "headers": {
    "Accept": [
      "*/*"
    ],
    "Host": [
      "httpbin.io"
    ]
  }
}
```

예상대로 User-Agent 헤더가 나타나지 않습니다.

User-Agent 헤더는 유지하되 값만 비워두고 싶다면, `-A`에 공백 한 칸을 제공하면 됩니다:

```sh
curl -A " " "https://httpbin.io/headers"
```

이번에는 다음을 받게 됩니다:

```json
{
  "headers": {
    "Accept": [
      "*/*"
    ],
    "Host": [
      "httpbin.io"
    ],
    "User-Agent": [
      ""
    ]
  }
}
```

이 경우 User-Agent 헤더는 존재하지만, 값은 비어 있습니다.

### Configuring a Custom User Agent Header

근본적으로 User-Agent는 표준 HTTP 헤더로 동작합니다. 즉, [\-H or –header](https://curl.se/docs/manpage.html#-H) [option](https://curl.se/docs/manpage.html#-H)을 사용하여 cURL에서 다른 HTTP 헤더와 동일하게 구성할 수 있습니다:

```sh
curl -H|--header "User-Agent: <user-agent_string>" "<url>"
```

더 자세한 설명은 [how to send HTTP headers with cURL](https://brightdata.co.kr/blog/how-tos/http-headers-with-curl) 튜토리얼을 확인해 보시기 바랍니다.

다음은 `-H` 옵션의 데모입니다:

```sh
curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

> **Note**:
> [HTTP headers are designed to be case-insensitive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) 하므로, "User-Agent"와 "user-agent"는 동일하게 취급됩니다.

출력은 다음과 같습니다:

```json
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
}
```

이 cURL 명령은 다음과 동일하게 동작합니다:

```sh
curl --header "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

특수한 시나리오를 위해, 다음 두 가지 대체 헤더 형식도 고려할 수 있습니다:

- `"User-Agent:"`는 User-Agent 헤더를 완전히 제거합니다
- `"User-Agent: "`는 User-Agent 헤더를 포함하되 값은 비워 둡니다

## Creating a User Agent Rotation System in cURL

cURL로 대규모 자동 요청를 수행할 때는 단일 User-Agent 값을 사용하는 것만으로는 충분하지 않을 수 있습니다. 이는 [anti-bot systems](https://brightdata.co.kr/webinar/bot-detection)이 모든 유입 연결을 추적하기 때문입니다. 동일한 IP 주소에서 동일한 헤더를 가진 수많은 요청가 감지되면, 제한을 적용할 가능성이 큽니다.

핵심은 요청의 출처 IP를 무작위화하는 것입니다. user agent를 로테이션하면 다양한 브라우저에서 온 요청처럼 보이게 하여, 일시적 제한이나 차단이 트리거될 위험을 줄일 수 있습니다.

cURL user agent 로테이션을 구현하는 데에는 세 단계가 있습니다:

1. **Gather user agents**: 다양한 브라우저(서로 다른 버전 및 디바이스 포함)에서 수집한 실제 user agent 문자열 컬렉션을 구성합니다.
2. **Create rotation logic**: 컬렉션에서 user agent를 무작위로 선택하는 코드를 개발합니다.
3. **Randomize your requests**: 선택된 user agent를 각 cURL 요청에 적용합니다.

이 구현은 Unix 기반 시스템과 Windows 간에 약간의 코드 차이가 필요합니다. 두 가지 접근 방식을 모두 살펴보겠습니다!

### Implementation in Bash

먼저 [User Agent String.com](https://useragentstring.com/pages/useragentstring.php) 같은 리소스에서 정상적인 user agent를 수집하고 배열에 저장합니다:

```
user_agents=( 
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" 
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0" 
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0" 
) 
```

다음으로 [RANDOM](https://tldp.org/LDP/abs/html/randomvar.html) 변수를 사용하여 이 목록에서 무작위로 선택하는 함수를 만듭니다:

```sh
get_random_user_agent() { 
    local count=${#user_agents[@]} 
    local index=$((RANDOM % count)) 
    echo "${user_agents[$index]}" 
} 
```

이제 함수를 호출하여 무작위 user agent를 가져오고, 이를 cURL 요청에 포함합니다:

```sh
user_agent=$(get_random_user_agent) 

curl -A "$user_agent" "https://httpbin.io/user-agent"
```

> **Note**:
> 대상 URL은 특정 요구사항에 맞게 조정하시기 바랍니다.

이 요소들을 결합하면 다음과 같은 완전한 bash 스크립트가 됩니다:

```sh
#!/bin/bash 

user_agents=( 
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" 
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0" 
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0" 
) 

get_random_user_agent() { 
    local count=${#user_agents[@]} 
    local index=$((RANDOM % count)) 
    echo "${user_agents[$index]}" 
} 

user_agent=$(get_random_user_agent) 

curl -A "$user_agent" "https://httpbin.io/user-agent"
```

이 스크립트를 여러 번 실행하면 매번 다른 user agent가 나타나는 것을 확인할 수 있습니다—목표 달성입니다!

Output:

```ssh
arturk@maint-xubuntu20:~/Support/Sandbox/curl-user-agent$ ./random_ua.sh 
{
  "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"
}
arturk@maint-xubuntu20:~/Support/Sandbox/curl-user-agent$ ./random_ua.sh 
{
  "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"
}
arturk@maint-xubuntu20:~/Support/Sandbox/curl-user-agent$ ./random_ua.sh 
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"
}
```

### Implementation in PowerShell

먼저 [WhatIsMyBrowser.com](https://www.whatismybrowser.com/) 같은 소스에서 실제 user agent를 수집합니다. 그런 다음 이를 [array variable](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays?view=powershell-7.4)에 저장합니다:

```sh
$user_agents = @(

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"

    # Add more user agents here

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

)
```

다음으로 컬렉션에서 user agent를 무작위로 선택하는 함수를 개발합니다:

```sh
function Get-RandomUserAgent {

    # Count the total user agents available

    $count = $user_agents.Count

    # Generate a random index within the valid range

    $index = Get-Random -Maximum $count

    # Return the randomly chosen user agent

    return $user_agents[$index]

}
```

마지막으로 이 함수를 호출하여 무작위 user agent를 얻고, 이를 cURL リクエ스트에 적용합니다:

```sh
# get the random user agent

$user_agent = Get-RandomUserAgent

# make an HTTP request to the given URL 

# using the random user agent

curl.exe -A "$user_agent" "https://httpbin.io/user-agent"
```

모든 내용을 합치면 다음과 같은 완전한 PowerShell 스크립트가 됩니다:

```sh
# list of user agents 

$user_agents = @( 

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" 

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0" 

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0" 

) 

function Get-RandomUserAgent { 

    # number of user agents in the list 

    $count = $user_agents.Count 

    # generate a random number from 0 to $count 

    $index = Get-Random -Maximum $count 

    # return the randomly extracted user agent 

    return $user_agents[$index] 

} 

# get a random user agent 

$user_agent = Get-RandomUserAgent 

# make an HTTP request to the given URL 

# using the random user agent 

curl.exe -A "$user_agent" "https://httpbin.io/user-agent"
```

이를 .ps1 파일로 저장하고 여러 번 실행하십시오. 실행할 때마다 서로 다른 user agent 문자열이 표시됩니다.

Output:

```ssh
PS C:\Users\user\Desktop\marketing\curl-user-agent> .\1.ps1
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
}
PS C:\Users\user\Desktop\marketing\curl-user-agent> .\1.ps1
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
}
PS C:\Users\user\Desktop\marketing\curl-user-agent> .\1.ps1
{
  "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"
}
```

## Final Thoughts

cURL에서 User-Agent 헤더를 커스터마이징하면, 요청가 일반 브라우저에서 온 것처럼 보이게 하여 기본적인 안티봇 시스템을 속이는 데 도움이 될 수 있습니다. 그러나 고도화된 안티봇 기술은 여전히 이러한 요청를 식별하고 차단할 수 있습니다.

속도 제한 같은 제한을 극복하려면 [integrating proxies with cURL](https://brightdata.co.kr/blog/proxy-101/curl-with-proxies)을 고려해 보시기 바랍니다. 더욱 강력한 솔루션이 필요하다면, cURL 또는 기타 HTTP 클라이언트를 사용한 자동 웹 リクエ스트를 위해 설계된 종합적인 차세대 스크레이핑 API인 [Scraper API](https://brightdata.co.kr/products/web-scraper)를 살펴보시기 바랍니다.

오늘 무료 체험에 등록하고 차이를 경험해 보십시오!