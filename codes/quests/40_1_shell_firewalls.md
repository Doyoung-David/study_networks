### **🧪 문제 1: 특정 IP 차단 상태 확인 후 차단 설정**

#### **✅ 실행 예시**

$ sudo ./problem1.sh

\[INFO\] 현재 rich rule 목록에 192.168.0.100 차단 룰이 존재하지 않습니다.

\[INFO\] 차단 룰을 추가합니다...

success

또는

$ sudo ./problem1.sh

\[INFO\] 192.168.0.100은 이미 차단되어 있습니다.

\[SKIP\] 추가 작업을 수행하지 않습니다.
```bash
ip_check=$(firewall-cmd --list-rich-rules | grep "$1")

if [ ! $ip_check ]; then
        echo "No such IP in the rules. Adding $1 in the rules now..."
        sudo firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address=$1 reject"
        sudo firewall-cmd --reload
else
        echo "$1 is already blocked."
fi
```

```bash
[doyoung@192.168.0.46 ~/Downloads]$ . problem1.sh 192.168.0.43
^[[O^[[I
No such IP in the rules. Adding 192.168.0.43 in the rules now...
[sudo] password for doyoung: 
Sorry, try again.
[sudo] password for doyoung: 
success
[doyoung@192.168.0.46 ~/Downloads]$ firewall-cmd --list-rich-rules
^[[O^[[Irule family="ipv4" source address="192.168.0.43" reject
rule family="ipv4" source address="192.168.0.34" reject

# ping 결과
[seungjae@192.168.0.43 ~]$ ping 192.168.0.46
PING 192.168.0.46 (192.168.0.46) 56(84) bytes of data.
From 192.168.0.46 icmp_seq=1 Destination Port Unreachable
```
---

### **🔒 문제 2: 포트 8080이 열려 있다면 닫기**

#### **✅ 실행 예시**

$ sudo ./problem2.sh

\[INFO\] 포트 8080/tcp 이 열려 있습니다. 제거합니다...

success

또는

$ sudo ./problem2.sh

\[INFO\] 포트 8080/tcp 이 열려 있지 않습니다. 아무 작업도 수행하지 않습니다.

```bash
port_var=$(sudo firewall-cmd --list-ports | grep "$1")
if [ $port_var ]; then 
        echo "$1 is opened now. Closing it now..."
        sudo firewall-cmd --permanent --remove-port=$1
        sudo firewall-cmd --reload
else
        echo "$1 is already closed."
fi
```
```bash
[doyoung@192.168.0.46 ~/Downloads]$ sudo firewall-cmd --list-ports

[doyoung@192.168.0.46 ~/Downloads]$ . problem2.sh 8080/tcp
8080/tcp is already closed.
```