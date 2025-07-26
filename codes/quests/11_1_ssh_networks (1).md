# **Network Shell Script 실습 문제**

## **문제 환경 설정**

실습을 위해 다음 파일들을 생성하세요:

\# 1\. 네트워크 로그 파일 생성
```bash
cat > network.log << 'EOF'
2024-01-15 10:30:25 192.168.1.100 CONNECT success
2024-01-15 10:30:30 192.168.1.101 CONNECT failed
2024-01-15 10:31:15 192.168.1.102 CONNECT success
2024-01-15 10:31:20 192.168.1.100 DISCONNECT success
2024-01-15 10:32:10 192.168.1.103 CONNECT success
2024-01-15 10:32:15 192.168.1.101 CONNECT success
2024-01-15 10:33:25 192.168.1.104 CONNECT failed
2024-01-15 10:33:30 192.168.1.102 DISCONNECT success
EOF
```

\# 3\. 접속 통계 파일 생성
```bash
cat > connections.txt << 'EOF'
192.168.1.100 5
192.168.1.101 12
192.168.1.102 8
192.168.1.103 3
192.168.1.104 15
192.168.1.105 7
EOF
```

---

## **문제 1: 네트워크 연결 상태 분석기**

**요구사항:**

* `network.log` 파일을 분석하여 연결 성공/실패 통계를 출력하는 스크립트 작성  
* 전체 연결 시도 수, 성공 수, 실패 수를 계산  
* 성공률을 백분율로 표시 (소수점 제거)

**출력 형태:**

\=== 네트워크 연결 분석 결과 \===

전체 연결 시도: X건

성공: Y건

실패: Z건

성공률: W%

**제한사항:**

* if문과 변수만 사용  
* grep, wc, cut 명령어 활용  
* 파일명은 스크립트 실행 시 첫 번째 인자로 받기
```bash
#스크립트
total_attempt=$(wc -l network.log | cut -d" " -f1)
success=$(cat network.log | grep "success" | wc -l | cut -d" " -f1)
failed=$(cat network.log | grep "failed" | wc -l | cut -d" " -f1)

echo "total: $total_attempt"
echo "success: $success"
echo "failed: $failed"

echo $((100 * $success / $total_attempt))% 
```
```bash
#출력결과
[reyne@192.168.0.52 ~/Downloads/doyoung]$ . network.sh
total: 8
success: 6
failed: 2
75%
```
---

## **문제 2: IP 주소별 접속 빈도 상위 리스트**

**요구사항:**

* `network.log`에서 IP 주소별 접속 횟수를 계산  
* 접속 횟수 기준으로 내림차순 정렬하여 상위 3개만 출력  
* 각 IP의 첫 접속 시간도 함께 표시

**출력 형태:**

\=== 접속 빈도 TOP 3 \===

1위: 192.168.1.XXX (X회) \- 첫 접속: 10:XX:XX

2위: 192.168.1.XXX (X회) \- 첫 접속: 10:XX:XX  

3위: 192.168.1.XXX (X회) \- 첫 접속: 10:XX:XX

**제한사항:**

* if문과 변수만 사용  
* cut, sort, uniq, grep 명령어 활용  
* head나 tail로 결과 제한
```bash
#스크립트
top3=$(cat network.log | cut -d" " -f3 | sort -n | uniq -c | head -n 3)
#frequency
first_fre=$(echo $top3 | cut -d" " -f1 | head -n 1)
second_fre=$(echo $top3 | cut -d" " -f1 | head -n 2 | tail -n 1)
third_fre=$(echo $top3 | cut -d" " -f1 | tail -n 1)

#IP
first=$(echo $top3 | cut -d" " -f2)
second=$(echo $top3 |  cut -d" " -f4)
third=$(echo $top3 |  cut -d" " -f6)

#first connection
first_time=$(grep "$first" ./network.log | head -n 1 | cut -d" " -f2)
second_time=$(grep "$second" ./network.log | head -n 1 | cut -d" " -f2)
third_time=$(grep "$third" ./network.log | head -n 1 | cut -d" " -f2)

#output
echo "$first $first_fre $first_time"
echo "$second $second_fre $second_time"
echo "$third $third_fre $third_time"
```
```bash
#출력 결과
[reyne@192.168.0.52 ~/Downloads/doyoung]$ . network2.sh 
192.168.1.100 2 10:30:25
192.168.1.100 2 10:30:25
192.168.1.100 2 10:30:25
```
---

## **문제 3: 서버 상태 점검 스크립트**

**요구사항:**

* `servers.sh` 실행해 각 서버에 대해 ping 테스트 실행  
* 응답 있는 서버와 없는 서버를 구분하여 출력  
* 응답 시간이 100ms 이상인 서버는 "느림" 표시

**입력 형태:**

	**\~$ [servers.sh](http://servers.sh) 123.92.0.12**

**출력 형태:**

\=== 서버 상태 점검 결과 \===

\[정상\] web01 (**123.92.0.12**) \- 응답시간: XXms

OR

\=== 서버 상태 점검 결과 \===

\[오프라인\] db01 (**123.92.0.11**) \- 응답없음

...

**제한사항:**

* if문과 변수만 사용  
* cut, ping 명령어 활용  
* ping은 1회만 실행 (`ping -c 1`)

```bash
# 스크립트 내용
speed=$(ping -c 1 $1 | head -n 2 | cut -d" " -f 8 | cut -d"=" -f 2 | cut -d" " -f 1)

if [ $speed != "0" ]; then
        echo "$1 $speed m/s"
else
        echo "$1 doesn't response."
fi
```
```bash
# 출력 결과
[reyne@192.168.0.52 ~/Downloads/doyoung]$ . servers.sh www.google.com
www.google.com 
37.5 m/s
[reyne@192.168.0.52 ~/Downloads/doyoung]$ . servers.sh 192.168.1.52
-bash: [: !=: unary operator expected
192.168.1.52 doesn't response.
[reyne@192.168.0.52 ~/Downloads/doyoung]$ 
```
---

## **문제 4: 네트워크 트래픽 임계값 모니터링**

**요구사항:**

* `connections.txt`에서 접속 수가 10 이상인 IP를 "높음", 5-9는 "보통", 4 이하는 "낮음"으로 분류  
* 각 분류별로 개수 계산하여 출력  
* "높음" 분류의 IP들만 별도로 나열

**출력 형태:**

\=== 트래픽 분석 결과 \===

높음(10회 이상): X개

보통(5-9회): Y개  

낮음(4회 이하): Z개

\[주의 필요 IP 목록\]

192.168.1.XXX (XX회)

192.168.1.XXX (XX회)

**제한사항:**

* if문과 변수만 사용  
* cut, sort 명령어 활용  
* 숫자 비교를 위한 조건문 사용
```bash
# 스크립트
Traffic=$(cat connections.txt | cut -d" " -f2 | sort -n)
High=0
Moderate=0
Low=0

for c in $Traffic; do
        if [ $c -ge "10" ]; then
                High=$(($High+1))
                grep "$c" connections.txt | cut -d" " -f1 >> notice 
        elif [ $c -le "4" ]; then
                Low=$(($Low+1))
        else
                Moderate=$(($Moderate+1))
        fi
done

echo $High
echo $Moderate
echo $Low
cat notice
```
```bash
# 출력 결과
2
3
1
192.168.1.101
192.168.1.104
```
---

## **문제 5: 현재 시스템 네트워크 정보 수집기**

**요구사항:**

* 현재 시스템의 IP 주소, 기본 게이트웨이, 활성 인터페이스 개수를 출력  
* 인터넷 연결 상태 확인 (8.8.8.8로 ping 테스트)  
* 모든 정보를 보기 좋게 정리하여 출력

**출력 형태:**

\=== 시스템 네트워크 정보 \===

내부 IP: 192.168.1.XXX

기본 게이트웨이: 192.168.1.X

활성 인터페이스: X개

인터넷 연결: 정상/차단

**제한사항:**

* if문과 변수만 사용  
* ip, hostname, ping, grep, wc 명령어 활용  
* 각 정보를 변수에 저장 후 출력

---

## **실행 방법**

각 문제의 스크립트를 작성한 후 다음과 같이 실행:

\# 실행 권한 부여

chmod \+x script\_name.sh

\# 스크립트 실행

./script\_name.sh \[인자\]

## **주의사항**

* 모든 스크립트는 `#!/bin/bash`로 시작  
* 변수 선언 시 공백 없이 작성: `var=value`  
* if문 조건 확인 시 `[ ]` 사용  
* 명령어 결과를 변수에 저장할 때 `$(command)` 사용  
* 파일 존재 여부 확인: `[ -f filename ]`

