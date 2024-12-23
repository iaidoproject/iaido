# i-Aegis Mail Send Module(Verion 1.0)
i-Aegis Mail Send Module(Verion 1.0)은 탐지 시나리오를 바탕으로 
관리자 또는 지정된 담당자에게 탐지된 이벤트의 정보 또는 처리(조치)가 완료된 정보를 메일로 발송해 주는 이메일 알림 서비스이다.

## To start using i-Aegis Mail Send Module
1. 설치 : iAegis app의 하위폴더 bin/scripts/ 안에 Module의 압축을 해제하는 것으로 완료됩니다.
    1) iAegis app의 하위폴더 bin/scripts/로 이동(/home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/)
        $ cd /home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/
    2) 모듈의 tar.gz 압축을 해제
        $ tar -zxvf mailSendModule.tar.gz
    3) 모듈의 모든 소스를 /home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/로 이동
        $ mv /home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/mailSendModule/* /home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/
2. 설정
    1) 모듈의 실행파일(eventAlarm.sh) 수정(3번째 라인의 경로 이동 명령어를 현재의 환경에 맞춰 수정)
        - 모듈의 실행파일(eventAlarm.sh)를 vi 명령어로 연다.
            $ vi /home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/eventAlarm.sh
        - 수정명령어(i)를 이용해 해당파일(eventAlarm.sh)의 3번째 라인인 경로 이동 명령어를 현재의 환경에 맞춰 수정한다.
            1 #!/bin/bash
            2
            3 cd /home/UMSECUI/splunk8/etc/apps/iAegis/bin/scripts/      -> 이부분을 cd /home/UMSECUI/splunk/etc/apps/iAegis/bin/scripts/ 로 수정
            4 $SPLUNK_HOME/bin/python3 eventAlarm.py $8
    2) 발송될 메일에 사용될 링크 수정
        - 모듈의 설정파일(setting.json)을 vi 명령어로 연다.
            $ vi /home/UMSECUI/splunk8/etc/apps/iAegis/bin/scripts/setting.json
        - 수정명령어(i)를 이용해 해당파일(setting.json)의 5번째 라인인 "summaryLink"의 value 값을 해당 링크페이지에 해당하는 url로 수정한다.
            1 {
            2     "mail" : {
            3         "mailUse" : true,
            4         "mailTitle": "i-AEGIS 탐지 이벤트",
            5         "summaryLink" : "https://10.112.3.101:8001/ko-KR/app/iAegis/stats_snr_detail",       -> 이부분을 환경에 맞는 페이지 링크로 수정
            6         "api" : {
            7             "sendUrl" : "https://openapi.samsung.net",
            8             "sendPath": "/mail/api/v2.0/mails/send?",
            9             "senderUserId" : "um.monitor",
            10             "header" : {
            11                           ...

## 소스 흐름
1. 기능 별 
    1) 모듈 실행(eventAlarm.sh -> eventAlarm.py)
    2) logging 설정 및 log파일 생성(commonPackage/logger.py -> commonPackage/settingFileRead.py -> setting.json -> commonPackage/logger.py)
    3) 모듈의 설정파일에서 설정 값 읽기(commonPackage/settingFileRead.py -> setting.json)
    4) 데이터 압축해제(inputData/unzip.py)
    5) 압축해제된 원천 데이터 변형(inputData/tramsform.py)
    6) 변형이 완료된 데이터를 grouping(inputData/grouping.py)
    7) grouping이 완료된 데이터를 메일 발송을 위한 형태의 데이터로 변형(inputData/tramsform.py)
    8) 발송될 메일의 본문 내용(html형식)으로 데이터 생성(mailTemplates/mailHtmlOutput.py)
        - html의 style이 부분 생성(mailTemplates/mailContents/htmlCommon.py의 htmlStyleFun)
        - html의 본문 컨텐트의 시작 div 태그 생성(mailTemplates/mailContents/htmlCommon.py의 htmlWrapStartFun) 
        - html의 본문 제목 컨텐트 생성(mailTemplates/mailContents/htmlCommon.py의 htmlMailTitleFun)
        - html의 본문 퇴직예정자 컨텐트 생성(mailTemplates/mailContents/htmlCommon.py의 htmlRetirementFun) 
            : 퇴직예정자가 탐지 되었을 경우에만 html에 생성됨
        - html의 본문 information 컨텐트 생성(mailTemplates/mailContents/htmlInfo.py)
        - html의 본문 Summary 컨텐트 생성(mailTemplates/mailContents/htmlSummary.py)
        - html의 본문 시나리오 조건 컨텐트 생성(mailTemplates/mailContents/htmlScenarioConditions.py)
        - html의 본문 Detection 컨텐트 생성(mailTemplates/mailContents/htmlDetection.py)
        - html의 본문 컨텐트의 종료 div 태그 생성(mailTemplates/mailContents/htmlCommon.py의 htmlWrapEndFun)
    9) api를 호출하여 메일 발송(apiCall.py)
2. 전체

    eventAlarm.sh -> eventAlarm.py -> 
    commonPackage/logger.py -> commonPackage/settingFileRead.py -> setting.json -> commonPackage/logger.py -> eventAlarm.py -> 
    commonPackage/settingFileRead.py -> setting.json -> eventAlarm.py -> 
    inputData/unzip.py -> eventAlarm.py -> 
    inputData/tramsform.py -> eventAlarm.py ->
    inputData/grouping.py -> eventAlarm.py ->
    inputData/tramsform.py -> eventAlarm.py ->
    mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlCommon.py의 htmlStyleFun -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlCommon.py의 htmlWrapStartFun -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlCommon.py의 htmlMailTitleFun -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlCommon.py의 htmlRetirementFun -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlInfo.py -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlSummary.py -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlScenarioConditions.py -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlDetection.py -> mailTemplates/mailHtmlOutput.py ->
        mailTemplates/mailContents/htmlCommon.py의 htmlWrapEndFun -> 
    mailTemplates/mailHtmlOutput.py -> eventAlarm.py -> 
    apiCall.py -> eventAlarm.py

## File & folder description of i-Aegis Mail Send Module
1. 파일&폴더 별 기능 설명(파일명의 ABC순으로 설명)
    1) [commonPackage] 폴더 : Mail Send Module의 공통 기능을 모아 놓은 pagkage
        a. [logger.py] : Mail Send Module 실행 중 발생하는 로그를 [log/log_오늘날짜.log]에 내용을 작성하는 기능
        b. [settingFileRead.py] : Mail Send Module 실행에 필요한 설정 내용이 명시되어 있는 [setting.json]파일을 읽어오는 기능
    2) [inputData] 폴더 : 원천 데이터의 압축을 풀거나 변형, 그룹화하는 기능을 모아 놓은 pagkage
        a. [cron.py] : 크론식으로 되어 있는 데이터를 한글 표현법(요구사항이 반영된 표현법)으로 변형해주는 기능
        b. [grouping.py] : 데이터를 원하는 조건에 맞게 구룹화해주는 기능
        c. [tramsform.py] : 데이터를 원하는 형태로 변형해 주는 기능
        d. [unzip.py] : 원천데이터의 압축을 풀어주는 기능(현재는 gz파일의 압축을 해제해줌)
    3) [log] 폴더 : Mail Send Module 실행 중 발생하는 log 파일을 모아놓은 폴더
        - [log/log_오늘날짜.log] : Mail Send Module 실행 중 해당 날짜의 파일에 로그내용을 기록함.
    4) [mailTemplates] 폴더 : 데이터를 html 형식을 모아 놓은 폴더
        a. [mailContens] 폴더 : html의 contens 내용을 모아 놓은 폴더
            - [htmlCommon.py] :
            - [htmlDetection.py] :
            - [htmlInfo.py] :
        b. [mailHtmlOutput.py] : [mailContens] 폴더 안에 있는 각각의 컨텐츠를 합쳐 하나의 html 형식으로 변환해주는 기능 

2. setting.json 파일 설명
    1) 

## 실행 중 오류 발생 시 확인사항
1. 
