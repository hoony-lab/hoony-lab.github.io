---
title: PaaS-TA - Troubleshooting: Dashboard
description:
search: true
categories:
  - paas-ta
  - troubleshooting
tags:
  - paas-ta
  - 사용자의 서버가 불안정합니다
permalink:

last_modified_at: 2021-05-27
---



## "사용자의 서버가 불안정합니다."

Dashboard 에서 애플리케이션 목록과 서비스 목록은 잘 조회되는데 **"사용자의 서버가 불안정합니다."** 라는 메시지가 뜬다.

  - Dashboard 에서 애플리케이션 탭, 서비스 탭 확인
    - 애플리케이션 상태 확인
    - 서비스에서 "대시보드" 버튼이 없어짐 (~~이러면 뭘 확인해야지~~)


## 앱이 돌아가도 완전히 정상이라고 할 수 없겠지 ?

  - IaaS Dashboard 에서 VM의 상태 확인
  - Inception 에서 instance 및 process의 상태 확인
    - `bosh instances -p`

모든 instance와 process가 `Process State : running` 임을 확인해도 에러 메세지가 뜬다. 더 깊숙히 찾아봐야 한다.


## 이제 뭐해, 로그 봐야지 !

instance나 process가 정상적인 상태가 아니라면 그 부분을 먼저 확인해 볼 수도 있겠지만, 모두 정상이라고 하니까 Dashboard가 있는 portal 관련 deployment 를 확인해야 한다.

portal 관련 deployment는 portal-ui, portal-api 2가지가 있다. ui는 api에서 데이터를 받아와서 보여주는 방식이니 api를 확인한다. (~~묻지도 따지지도 않고 당연히 api 먼저 확인~~)

`bosh -d <deployment-name> ssh <instance-name>`

```shell
# instance
cd /var/vcap/sys/log/<some-portal-api-process>
```


## "Too Many Connection"

DB 로그를 살펴보니, 로그인 시도에 관한 로그가 1초에 여러 건이 몇 수십분 동안 지속되고 있다. `stdout` 파일의 마지막 수정시간이 한시간인데, `stdout` 파일의 마지막 수정시간은 현재 시간(`date`)이다.

  - 누군가의 무차별 대입 공격(Brute Force attack) 인가?
  - 누군가의 무한한 로그인 시도 및 새로고침 인가 ?  


  - `process` 재시작
    - `monit restart <process-name>`  


## 지켜보자

![please-please-please](https://media.giphy.com/media/3orieTfp1MeFLiBQR2/giphy.gif)


.
