---
layout: post
title: "Shimcache 정리 보고서"
date: 2026-09-18
last_modified_at: 2026-09-18
categories: study
tags: ["Blog", "study", "code", "linux", "정보보안"]
---

# Shimcache 정리 보고서

작성자 | 김우리 | oy699111@gmail.com

과제 내용 | Shimcache 에 대해 아주 자세하게, 아주 많은 시간을 들여서, 적어도 누가 이걸 물어보면 모르는게 없을 만큼 공부해서 "보고서"로 제출 (7/12)

과제 기한 | 2024.07.09 - 2024.07.12 23:59

## [목차]

1. 서론
2. 본론
    1. Amcache vs Shimcache
    2. 도구
    3. 디지털 포렌식 가치
    4. 구조
    5. SDB
    6. 파일
    7. 단점
3. 결론

## 서론

Shimcache는 윈도우 운영체제 아티팩트로, 모든 실행 프로그램 파일의 경로, 크기, 마지막 수정시간, 마지막 접근 시간 등 정보를 저장하고 있다. Shimcache는 Windows 호환성 shim 레이어에 의해 생성되는 캐시로, Prefech와 비슷하지만 하지만, 프리패치는 한정적이라 사라질 가능성이 있다. 반대로 사라진 정보가 Shimcache에서는 보여질 수 있는 가능성이 있다. 프로그램 간 호환성을 제어하는 캐시 역할을 한다. 응용 프로그램 간 호환성을 제어하고 트러블 슈팅문제해결을 위해 만든 파일이다. 악성코드 실행 시 호환성 문제가 발생하기에 침해분석에 활용된다. 프리패치와 비슷한 응용프로그램의 실행 정보가 저장된다. 특정 malware가 실행된 시스템을 식별할 수 있다. Shimcache 정보를 통해 어떤 응용프로그램이 호환성 모드에서 실행되었는지, 어떤 레지스트리 설정이 변경되었는지 등을 파악할 수 있다는 장점이 있는데 이는 시스템의 호환성 문제를 해결하거나, 보안 이슈를 찾는 데 매우 도움이 될 수 있다. 

AppCompatCache의 위치이다. (Shimcache) 

<img src="/assets/img/shimcache_1.png" alt="shimcache_1">


## 본론

1. Amcache vs Shimcache

가장 보편적으로 비교되는 두 가지가 AmCache 과 Shimcache이다. 둘은 비슷하면서도 다른데, 각자의 장점과 단점이 명확하고 차이점이 분명하다. 애플리케이션 호환성 캐시(Shimcache)와 AmCache는 실행된 애플리케이션과 설치된 프로그램에 대한 정보를 저장하는 유사한 기능을 제공하지만 몇 가지 차이점이 있다. Shimcache는 이전 버전의 Windows에서 실행되도록 설계된 프로그램에 대한 이전 버전과의 호환성을 제공하도록 설계된 반면 AmCache는 특정 버전의 OS에 국한되지 않는다. Shimcache는 레지스트리에 저장되는 반면 AmCache는 레지스트리나 BCF 파일에 저장할 수 있다. Shimcache는 프로세스 실행 플래그와 같은 프로그램 호환성과 관련된 보다 구체적인 데이터도 저장하는 반면 AmCache는 장치 및 드라이버 정보, 바로 가기 정보, 인벤토리 데이터를 포함한 보다 광범위한 정보를 저장한다. 둘은 Windows 애플리케이션 호환성 인프라 내의 아티팩트다.

둘의 위치도 다른데, ShimCache의 위치는

```bash
HKEY_LOCAL_MACHINE\
SYSTEM\
CurrentControlSet\
Control\
Session Manager\
AppCompatCache\
AppCompatCache
```

AmCache 의 위치는, BCF 파일이나 레지스트리 하이브 파일 둘 중 하나로 저장된다. 

```bash
\Windows\
AppCompat\
Programs
```

결론적으로 Shimcache는, 운영 체제와 애플이케이션의 호환성에 대한 정보를 조장하기 위해 설계 된 아티팩트이다. 또한, 이전 버전의 윈도우에서 실행할 수 있도록 설계되었고, 레지스트리에 저장이 되며 프로그램 호환성과 관련된 것들을 구체적인 데이터로 저장한다는 것이다. 그리고, 각 애플리케이션이 shim되었는지 여부에 관계없이 검사된다는 것이 특징이다. Shimcache 증거가 저장하는 것들은 명확하게 나와있지 않지만, 간추려 보면 파일 전체 경로, 파일 크기, $Standard_Information(SI) 마지막 수정 시간, Shimcache 마지막 업데이트 시간, 프로세스 실행 플래그 등과 같은 다양한 파일 메타데이터를 저장한다. 또한, 데이터 보존 기간이 제한되어 있고, 다양한 시스템 이벤트나 도구를 통해 지워질 수 있다는 단점이 있다.

Amcache는 나중에 설치된 애플리케이션과 그것을 사용함에 있어서 더 자세한 인벤토리를 제공하기 위해 만들어진 아티팩트로, Shimcache와 목적부터 차이가 있다. 그리고 데이터 보존 기간이 더 길다. 애플리케이션 사용에 대한 보다 포괄적인 과거 기록을 제공할 수 있어서 꽤 과거의 내용을 파악해야 한다면 AmCache를 선택하는 것이 좋다. 

1. 도구

분석할 수 있는 도구로는 여러가지가 있으며, 각각의 장단점이 명확하다. 하지만 이 보고서에서는 Eric Zimmerman의 AppCompatCacheParser를 중점으로 본다.  AppCompatCacheParser은 SYSTEM 레지스트리 하이브가 아닌 레지스트리에 저장된 AppCompatCache 데이터를 파싱한다. 이 도구는 Shimcache뿐만 아니라 Amcache의 데이터를 파싱할 수 있다. 파일 경로, 마지막 수정 시간, 실행 플래그와 같은 정보를 추출하고 CSV, XML, JSON을 포함한 다양한 형식으로 데이터를 출력할 수 있으므로 효과적이다. 

```bash
AppCompatCacheParser.exe -f C:\Windows\System32\ config \SYSTEM --csv E:\AppCompatCache
```

위 코드로 경로를 파악하여 찾아 가면, PowerShell을 사용해 Amcache와 Shimcache를 모두 수집할 수 있다.

```bash

# 출력 파일을 저장할 경로를 지정한다.
$outputPath="C:\Output"
```

```bash

# Github 저장소에서 AmcacheParser를 다운로드한다.
Invoke-WebRequest -Uri"https://ericzimmerman.github.io/Downloas/AmcacheParser.zip"-OutFile"C:\path\to\AmcacheParser.zip"
```

```bash

# Amcache를 수집합니다.
$amcache= &"C:\Path\To\AmcacheParser.exe"-f"C:\Windows\AppCompat\Programs\Amcach.hve"-csv -o"$outputPath\Amcache.csv"
```

```bash

# Shimcache를 수집한다.
$shimcache= Get-ItemProperty"HKLM:\SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatCace\AppCompatCache"| Export-Csv"$outputPath\Shimcache.csv"-NoTypeInformation
```

이 도구는 이름에서 알 수 있듯이 ArtiFast ShimCache Artifact Parser는 전적으로 ShimCache의 아티팩트를 분석하는 것에 초점이 맞춰져 있다. 사건을 만들고 조사를 위한 증거를 추가한 후, 아티팩트 선택 단계에서ShimCache Artifact를 선택하여 분석 및 조사에 사용할 수 있다. 조사자가 shimcache의 내용을 추출하고 분석하여 시스템 활동에 대한 자세한 보기를 제공할 수 있다는 장점이 있고, Windows 7 이상을 실행하는 시스템에서만 사용할 수 있으며, 이전 버전의 Windows를 실행하는 시스템에서 조사자는 prefetch 폴더와 같은 다른 정보 소스를 사용하여 유사한 데이터를 수집할 수 있다. 

위의 도구 밖에도 Microsoft Sysinternals 도구인 sdelete, 오픈소스 도구인 shimcacheparser 등 Shimcache를 분석할 수 있는 여러 도구가 있다.  

1. 디지털 포렌식 가치

ShimCache에서 볼 수 있는 정보는 실행 파일 이름, 파일 경로, 마지막 수정 날짜, 시간 등을 기록하여 볼 수 있다.  이러한 항목을 분석하면 실행 파일이 시스템에서 실행되었는지 여부를 식별할 수 있다. 로컬 드라이브 외에도 이동식 미디어와 UNC 경로의 실행 파일도 ShimCache에 저장된다. 그리고, 시스템이 재부팅되거나 종료될 때 하드 드라이브에 기록된다. 이 기능은 안티 포렌식(레지스트리 항목의 데이터 삭제)을 복잡하게 만들어 귀찮게 할 수 있다. ShimCache를 분석하면 특히 맬웨어 사고 분석 중에 귀중한 정보를 제공받을 수 있다. 

조금 더 자세하게 살펴보면, Shimcache는 디지털 포렌식 및 사고 대응의 관점에서 중요하게 사용된다. 예시로 활동 타임라인을 작성할 수 있는데, 조사관은 shimcache를 조사하여 어떤 애플리케이션이 언제 실행되었는지 확인하고 시스템 활동 타임라인을 작성할 수 있다. 두 번째로, 시스템이 수정된 대로 정보를 제공받을 수 있다. Shimcache는 애플리케이션 실행의 결과로 시스템에 가해진 수정에 대한 정보를 준다. 이는 공격자 또는 맬웨어의 동작을 이해하고 어떤 파일이 생성 또는 수정되었는지, 어떤 레지스트리 키에 액세스했는지 보여주는 데 유용하다. 악의적 또는 비정상적인 활동을 감지해 활동을 식별하는데 도움이 된다. Shimcache에서 알려지지 않았거나 의심스러운 애플리케이션이 발견되면 맬웨어 또는 공격 시도가 있음을 나타낼 수 있다. 

1. 구조

크기는 1024개 항목으로 되어 있다. ShimCache에 입력된 최신 파일은 64비트 타임스탬프와 함께 항목 상단에 나열되며, 레지스트리의 ShimCache 항목은 파일, 경로 및 타임스탬프 목록이다.

아래 이미지는, Shimcache의 구조이다. 

![shimcache_2](/assets/img/shimcache_2.png)

Windows 시스템에서 애플리케이션이 실행되면 운영 체제는 shimcache에 실행을 기록한다. 애플리케이션 이름, 실행 파일 경로, 실행 시간, 레지스트리 변경, 새 파일 생성 등의 애플리케이션 실행으로 인해 시스템에 발생한 모든 수정 사항도 기록한다. 이 정보는 캐시 항목이라고 알려진 일련의 레코드로 shimcache에 저장되며, 각 캐시 항목에는 애플리케이션 이름, 실행 파일 경로 및 실행 시간이 포함된다. shimcache에는 실행 파일의 체크섬이 포함되어 있어 운영 체제가 파일의 무결성을 확인할 수 있다. ****

1. SDB

Shim Database의 약어이며, 운영체제의 버전이 업그레이드 될 경우 신규 생성 또는 삭제되는 DLL의 모음인 API로 인해 프로그램 간 호환성 문제가 발생하고 이를 응용 프로그램 호환성 데이터베이스(Application Compatibility Database)구조를 이용하여 해결한다. 

1. 파일

응용 어플리케이션 실행 시 운영체제의 상이한 버전으로 인한 호환성 문제를 해결하기 위한 함수 등을 말하는데, kernel32.dll의 내부 함수인 BasepCheckBadApp 함수가 대표적이다. 해당 함수가 호출되면 프로그램이 프로그램 별 호환성 문제를 해결하기 위해서 SDB의 내용을 참고 하는데, 이보다 더빠른 문제를 해결하기 위해 캐시 데이터를 선행 참조하게 되는데 이를 심캐시(ShimCache) 라고 한다.

ShimCache는 **`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache\AppCompatCahe`** 위치에서 값을 확인해 볼 수 있다.

그 밖에도 여러 파일의 위치를 볼 수 있는데, 

ShimCache 아티팩트 소스 파일의 위치

```java
C:\Windows\System32\config\SYSTEM
```

Windows 7, 8 및 10에서 레지스트리 키

```java
**HKLM\SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatCache\AppCompatCache.**
```

Windows XP에서 레지스트리 키

```java
**SYSTEM\CurrentControlSet\Control\SessionManager\앱 호환성**
```

등의 위치로 이동했을 때 해당 위치에 알맞는 파일을 볼 수 있다. 

1. 단점

완벽한듯 보이는 ShimCache 아티팩트에도 단점이 존재한다. 첫 번째로 정보는 메모리에 보관되고 시스템이 종료될 때만 레지스트리에 기록되는데, 이는 라이브 응답을 수행할 때 이 증거 소스를 가져오는 능력에 영향을 미친다. 이러한 제한 사항을 해결하기 위해 메모리에서 ShimCache를 읽는 Volatility 플러그인을 작성했다. Fred House, Claudiu Teodorescu, Andrew Davis등 이 플러그인들은 32비트 및 64비트 아키텍처에서 Windows XP SP2부터 Windows 2012 R2까지 지원하며, 2015년 Volatility 플러그인 콘테스트에서 우승했다.

![shimcache_3](/assets/img/shimcache_3.png)

ShimCacheMem 플러그인과 함께 Volatility를 분석된 시스템의 메모리에 사용하는 방법을 보여 준다. 메모리에서 직접 ShimCache를 보거나 시스템 종료 후 레지스트리를 쿼리하여 이 경우 Prefetch 아티팩트에서 발견된 증거를 확인할 수 있다. 그리고 Windows Server 시스템에서는 기본적으로 Prefetch가 비활성화되어 있기 때문에 ShimCache가 더 가치 있는 아티팩트가 될 수 있다.

PreFetch가 더 안전한 것은 사실이지만, Win 환경에서는 ShimCache가 더 가치있게 사용이 된다. 모든 Windows 운영 체제에서 이 아티팩트를 사용할 수 있다는 점을 감안할 때, ShimCache에서 얻은 정보는 매우 다양하다. 이 경우의 결과로서, ShimCache는 시스템에서 실행되는 regedit.exe 및 rundll32.exe에 대한 Prefetch 결과를 지원할 수 있었다.

## 결론

Shimcache는 디지털 포렌식 및 사고 대응 조사를 위한 귀중한 정보 소스로 사용되며, 시스템 활동의 타임라인과 시스템에 대한 수정 사항에 대한 정보를 제공하며 조사자가 악의적이거나 비정상적인 활동을 식별하는 데 도움이 될 수 있다.sdelete 및 shimcacheparser와 같은 도구를 사용하여 조사자는 shimcache의 내용을 추출하고 분석할 수 있고 이를 실제 상황에 사용할 수 있다. 그로 인해 시스템에서 공격자 또는 맬웨어의 행동을 더 깊이 이해할 수 있게 되며, 디지털 포렌식 전문가로서 shimcache를 활용하면 진실을 밝히고 복잡한 사이버 보안 사고를 이해하는 데 큰 도움이 될 수 있다. Windows XP에서 ShimCache는 최대 96개 항목을 유지하지만 Windows 7 이하에서는 ShimCache가 최대 1024개 항목을 유지할 수 있다. ShimCache Parser를 사용하면 해당 내용을 구문 분석하고 볼 수 있다는 장점이 있다. 또한, Windows 환경에서 Prefetch 아티팩트보다 ShimCache 아티팩트의 가치가 더 높다는 것에서 중요하게 작용한다. 

---

FTK, HxD 둘 다 안보여서 레지스트리 편집기로 진행!

HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache\AppCompatCache

![shimcache_4](/assets/img/shimcache_4.png)

의 경로로 ShimCache 를 분석하기 위해 들어가 보았지만 없어서 분석을 할 수 없었다…

![shimcache_5](/assets/img/shimcache_5.png)

정상적으로 분석이 가능한 것에서는 이렇게 값이 뜬다.