# Born2beRoot - This document is a System Administration related exercise.

### This project aims to introduce you to the wonderful world of virtualization.
You will create your first machine in VirtualBox (or UTM if you can’t use VirtualBox)
under specific instructions. Then, at the end of this project, you will be able to set up
your own operating system while implementing strict rules.

*Version : 1*

---

### Mandatory 

<details>
<summary>AppArmor 확인하기</summary>

* AppArmor is a Mandatory Access Control (MAC) system which is a kernel (LSM) enhancement to confine programs to a limited set of resources. AppArmor's security model is to bind access control attributes to programs rather than to users. AppArmor confinement is provided via profiles loaded into the kernel, typically on boot. (https://wiki.ubuntu.com/AppArmor)
* 요약하자면 프로그램이 사용할 수 있는 자원을 제한할 수 있도록 도와주는 프로그램이다.
* AppArmor profiles can be in one of two modes: enforcement and complain. Profiles loaded in enforcement mode will result in enforcement of the policy defined in the profile as well as reporting policy violation attempts (either via syslog or auditd). Profiles in complain mode will not enforce policy but instead report policy violation attempts.
* enforcement 모드에서는 정책을 강요(강제)하고, 정책 위반 시도를 기록한다. complain 모드에서는 정책을 강요(강제)하지는 않지만, 정책 위반 시도는 기록한다.

`aa-status` 명령으로 apparmor의 상태를 확인할 수 있다.

- - -
</details>

<details>
<summary>UFW 설정하기</summary>

* The Uncomplicated Firewall (ufw) is a frontend for iptables and is particularly well-suited for host-based firewalls. ufw provides a framework for managing netfilter, as well as a command-line interface for manipulating the firewall. ufw aims to provide an easy to use interface for people unfamiliar with firewall concepts, while at the same time simplifies complicated iptables commands to help an administrator who knows what he or she is doing. (https://wiki.ubuntu.com/UncomplicatedFirewall)
* 요약하자면 방화벽에 익숙하지 않은 사람이더라도 방화벽을 쉽게 사용할 수 있도록 만들어진 것이 ufw라는 것이다.

> apt install ufw

위 명령어를 통해 ufw를 설치할 수 있다.

`ufw status`를 통해 지금 ufw의 상태를 확인할 수 있는데, 설치 직후에는 꺼져있는 상태이므로 `ufw enable`을 통해 켤 수 있다.
<img width="725" alt="image" src="https://user-images.githubusercontent.com/67845112/185955602-8615a361-7ed7-4411-8707-6922ba7d318c.png">

과제가 요구하는 목표는 ssh를 이용하기 위한 '4242' 포트만 남겨두고 다른 모든 포트를 막는 것이다.

기본 설정은 들어오는 모든 포트에 대해 막혀있는 상태이므로, `ufw allow (port)`를 이용해 '4242'포트만 열 수 있다.

> ufw status verbose
<img width="860" alt="image" src="https://user-images.githubusercontent.com/67845112/185955396-e530904b-ef3d-498c-93a6-d41b8e2df146.png">

추가 명령어들은 `ufw help`로 확인하자.

- - -
</details>

<details>
<summary>새 user를 만들고 group은 어떻게 등록하나</summary>
  
user를 만들거나 없애는 법은 간단하다.
> useradd $(new username </br> userdel $(target username)
 
user를 만들었으면, 비밀번호를 만들어주어야 한다. 해당 명령으로 비밀번호 변경도 가능하다.
> passwd $(target username)
 
`useradd` 명령만으로는 홈 디렉토리를 만들어주지 않기에 `-m` 옵션을 추가하여 같이 만들 수 있다.
그렇지만 `adduser` 명령을 이용하면 유저를 생성함과 동시에 홈 디렉토리와 비밀번호를 같이 만들 수 있다.
 
group 생성은 명령어 한 줄로 할 수 있다.
> groupadd $(new groupname)

누군가를 어떠한 그룹에 추가하고 싶으면 다음 명령을 사용한다.
> usermod -aG $(groupname) $(username)

`-a` 옵션은 append의 약자고, `-G` 옵션은 여러 그릅을 한 번에 추가할 수 있게 해주며, `-g` 옵션은 주어진 그룹을 유저의 primary 그룹으로 만들어준다.
primary 그룹은 유저가 로그인 했을 때 주어지는 공간을 관장하며, 그 외의 secondary group들은 해당 그룹에 유저가 접근하여 읽고 쓸 수 있도록 해 준다.

어떤 유저의 특정한 그룹만 제거하고 싶으면 다음 명령을 사용한다.
> gpasswd -d $(username) $(groupname)
  
각 사용자의 그룹은 `groups $(username)`으로 확인할 수 있다.

- - -
</details>

<details>
<summary>hostname은 어떻게 바꾸나</summary>

* 호스트명(hostname)은 네트워크에 연결된 장치(컴퓨터, 파일 서버, 복사기, 케이블 모뎀 등)들에게 부여되는 고유한 이름이다. https://ko.wikipedia.org/wiki/%ED%98%B8%EC%8A%A4%ED%8A%B8%EB%AA%85)
  
<img width="229" alt="image" src="https://user-images.githubusercontent.com/67845112/186026201-2154903d-ec0f-4cf9-9f06-c3834a45a9a7.png">
로그인 하면, (user)@(hostname) 형식으로 된 문구를 볼 수 있다.

hostname은 `hostnamectl set-hostname $(new hostname)` 으로 변경할 수 있다.
변경 후 재시작하면 적용되며, `hostname` 명령으로 확인할 수도 있다.
  
<img width="362" alt="image" src="https://user-images.githubusercontent.com/67845112/186026514-01b75259-be72-4731-9465-2f5adbd99e6c.png">

- - -
</details>
  
<details>
<summary>SSH 포트 설정하기</summary>
  
* 시큐어 셸(Secure SHell, SSH)은 네트워크 상의 다른 컴퓨터에 로그인하거나 원격 시스템에서 명령을 실행하고 다른 시스템으로 파일을 복사할 수 있도록 해 주는 응용 프로그램 또는 그 프로토콜을 가리킨다. (https://ko.wikipedia.org/wiki/%EC%8B%9C%ED%81%90%EC%96%B4_%EC%85%B8)
  
ufw를 설정할 때, 4242 포트를 열었던 것을 기억할 것이다.
그러나 ssh의 기본 포트는 22이므로, 4242로 접속할 수 있도록 바꿔줄 필요가 있다.

`vi /etc/ssh/sshd_config` 으로 ssh 설정파일을 불러올 수 있다.
열자마자 주석 처리된 포트 설정부분이 보이는데, 4242로 바꿔주자.

<img width="432" alt="image" src="https://user-images.githubusercontent.com/67845112/185984825-aa0ff852-2482-43d2-b084-b4b84f6b08f1.png">

- - -
</details>


<details>
<summary>비밀번호 정책 만들기</summary>

과제가 요구하는 정책은 다음과 같다.

* Your password has to expire every 30 days.
* The minimum number of days allowed before the modification of a password will
be set to 2.
* The user has to receive a warning message 7 days before their password expires.
* Your password must be at least 10 characters long. It must contain an uppercase
letter and a number. Also, it must not contain more than 3 consecutive identical
characters.
* The password must not include the name of the user.
* The following rule does not apply to the root password: The password must have
at least 7 characters that are not part of the former password.
* Of course, your root password has to comply with this policy.

먼저, 비밀번호가 만료되는 기한과, 비밀번호를 바꿀 수 있는 최소 날짜, 비밀번호 만료 전 경고일자는 쉽게 설정할 수 있다.
`/etc/login.defs` 파일에서 PASS_MAX_DAYS, PASS_MIN_DAYS, PASS_WARN_AGE를 변경하면 된다.
<img width="1052" alt="image" src="https://user-images.githubusercontent.com/67845112/185969315-a48ee903-04c2-4e51-a4ba-4b63b8fbc1fc.png">

그러나, `/etc/login.defs`를 수정했을 때는, 기존의 유저들에게는 해당 설정이 적용되지 않는다.
이 문제는 `chage` 명령을 이용해 수동으로 적용함으로써 해결할 수 있다. 
>  chage -m 2 -M 30 -W 7 $(username)

이 외의 설정을 하려면, 별도의 모듈 설치가 필요하다.
> apt install libpam-pwquality

`/etc/security/pwquality.conf`에서 설명에 따라 적절히 바꿔준다.

- - -
</details>

<details>
<summary>sudo 설정하기</summary>

과제의 요구사항은 다음과 같다.

* Authentication using sudo has to be limited to 3 attempts in the event of an incorrect password.
* A custom message of your choice has to be displayed if an error due to a wrong
password occurs when using sudo.
* Each action using sudo has to be archived, both inputs and outputs. The log file
has to be saved in the /var/log/sudo/ folder.
* The TTY mode has to be enabled for security reasons.
* For security reasons too, the paths that can be used by sudo must be restricted.
Example:
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

`visudo` 를 이용해 sudoers 파일을 수정할 수 있다.
이 파일에서 sudo 설정을 할 수 있는데, `man sudoers`를 통해 옵션들을 살펴볼 수 있다.
적절히 참고해서 바꿔주자.

<img width="1218" alt="image" src="https://user-images.githubusercontent.com/67845112/185983155-04de4c4e-261a-4a69-bd9e-122438b35caf.png">

로그파일을 살펴보다보면, stdin/out, ttyin/out 파일이 꺠져보이는 문제가 있는데, `gzip -d`를 이용해서 정상적으로 볼 수 있다.
- - -
</details>


<details>
<summary>monitoring.sh 만들기</summary>
먼저 과제에서 요구하는 출력을 살펴보자.
  
* The architecture of your operating system and its kernel version.
* The number of physical processors.
* The number of virtual processors.
* The current available RAM on your server and its utilization rate as a percentage.
* The current available memory on your server and its utilization rate as a percentage.
* The current utilization rate of your processors as a percentage.
* The date and time of the last reboot.
* Whether LVM is active or not.
* The number of active connections.
* The number of users using the server.
* The IPv4 address of your server and its MAC (Media Access Control) address.
* The number of commands executed with the sudo program.

  
1. 아키텍쳐와 운영체제는 `uname -a`로 가져올 수 있다.
2. cpu와 관련된 정보는 `lscpu`에서 확인할 수 있는데, physical processor는 이 항목의 Socket(s), virtual processor는 Socket(s) * Core(s) per socket이다.
 (https://www.ibm.com/docs/en/power8?topic=processors-virtual)
3. 사용 가능한 메모리, 디스크와, 사용중인 메모리, 디스크는 각각 `free`와 `df`를 통해 확인할 수 있다.
4. 사용중인 cpu의 점유율은 `mpstat`을 통해 확인할 수 있다. (sysstat)
5. 마지막 부팅 시간은 `who -b`를 통해 확인할 수 있다.
6. 파티션 정보는 `lsblk`를 이용해 볼 수 있는데, 여기서 LVM 파티션이 있는지 확인할 수 있다.
7. ssh가 연결된 개수는 `ss`에서 확인해 볼 수 있다.
8. 서버를 사용중인 유저의 수는 `who`에서 확인할 수 있다.
9. 서버의 IPv4 주소는 `hostname -I`에서 볼 수 있으며, MAC주소는 `ip link`로 확인할 수 있다.
10. sudo를 이용해 실행된 명령들은 `journalctl`의 로그를 통해 확인해 볼 수 있다.
  
이제 스크립트를 직접 작성해보자.
```bash
#!/bin/bash

echo -n "#Architecture : "
uname -a

echo -n "#CPU physical : "
sockets=$(lscpu | grep Socket | awk '{print $2}')
echo $sockets

echo -n "#vCPU : "
cores=$(lscpu | grep Core | awk '{print $4}')
printf "%d" $(( $sockets * $cores ))
echo

echo -n "#Memory Usage: "
free -m | grep Mem | awk '{printf "%d/%dMB (%.2f%%)", $3, $2, $3 * 100 / $2}'
echo

echo -n "#Disk Usage: "
using_disk=$(df -mP | grep -v ^File | awk '{sum1 += $3} END {print sum1}')
total_disk=$(( $(df -mP | grep -v ^File | awk '{sum2 += $4} END {print sum2}') + $using_disk ))
printf "%d/%dMB (%d%%)\n" $using_disk $total_disk $(( $using_disk * 100 / $total_disk))

echo -n "#CPU load: "
mpstat | tail -1 | awk '{printf "%.2f", 100-$13}'
echo "%"

echo -n "#Last boot: "
who -b | awk '{print $3 " " $4}'

echo -n "#LVM use: "
if [ $(lsblk | grep lvm | wc -l) == 0 ]
then echo "no"
else echo "yes"
fi

echo -n "#Connections TCP : "
ss | grep tcp | wc -l | tr -d '\n'
echo " ESTABLISHED"

echo -n "#User log: "
who | wc -l

echo -n "#Network: IP "
hostname -I | tr -d '\n'
echo -n "("
# ifconfig | grep ether | awk '{print $2}' | tr -d '\n'
ip link | grep link/ether | awk '{print $2}' | tr -d '\n'
echo ")"

echo -n "#Sudo : "
journalctl | grep USER=root | wc -l | tr -d '\n'
echo " cmd"
```
  
이제 작성한 스크립트를 매 10분마다 실행되도록 설정해야 한다.
특정 시간마다 프로그램이 실행되도록 도와주는 `cron`을 활용할 것이다.
`crontab -e`를 통해 설정파일에 들어가서, 양식에 맞게 입력한다.

<img width="583" alt="image" src="https://user-images.githubusercontent.com/67845112/186017040-ae694a6b-8b8b-401a-8d27-ff0e0bfc2d82.png">
  
<img width="1089" alt="image" src="https://user-images.githubusercontent.com/67845112/186019520-c78eeac8-e762-4d88-b49b-af3b8dfa91d0.png">
  
- - -
</details>

### Bonus

<details>
    <summary>wordpress를 구축해보자</summary>
  
* Set up a functional WordPress website with the following services: lighttpd, MariaDB, and PHP.

일단 과제에서 요구하는 서비스들을 모두 설치해주자.
  
> apt install lighttpd mariadb-server php php-fpm php-mysql
  
* lighttpd (pronounced /lighty/) is a secure, fast, compliant, and very flexible web server that has been optimized for high-performance environments. (https://www.lighttpd.net/)
* MariaDB Server is one of the most popular open source relational databases. It’s made by the original developers of MySQL and guaranteed to stay open source. (https://mariadb.org/)
* A popular general-purpose scripting language that is especially suited to web development.
Fast, flexible and pragmatic, PHP powers everything from your blog to the most popular websites in the world. (https://php.net/)
  
wordpress가 php로 쓰여진 사이트 제작 도구이기에, php를 설치하고, lighttpd에서 사이트 서버를 돌리며, mariadb로 데이터를 관리하려는 것 같다.
php-fpm은 서버와 프로그램을 연결해주는 CGI(Common Gateway Interface)의 일종이라고 한다. 빠른 cgi라는 의미로 fastcgi라고도 불리는 것 같다.

일단 php-fpm을 이용해서 lighttpd와 프로그램을 연결할 수 있도록 몇 가지 설정이 필요하다.
> vi /etc/php/$(php verson)/fpm/pool.d/www.conf

<img width="450" alt="image" src="https://user-images.githubusercontent.com/67845112/186032900-fbc35e1f-e7c6-4582-88cd-2bee0550f63d.png">

> vi /etc/lighttpd/conf-available/15-fastcgi-php.conf
<img width="610" alt="image" src="https://user-images.githubusercontent.com/67845112/186033084-cda1f210-f8c4-4b31-be1c-1dadc9798360.png">

> lighty-enable-mod fastcgi </br> lighty-enable-mod fastcgi-php

하라는대로 `service lighttpd force-reload`를 해 준다.

이제 데이터베이스를 설정해보자.
`mysql` 을 통해 mariadb에 로그인 할 수 있다.

> CREATE DATABASE dbname; </br> GRANT ALL PRIVILEGES on dbname.* TO 'username'@'localhost' IDENTIFIED BY 'password'; </br> FLUSH PRIVILEGES;
</br> EXIT;

워드프레스에 사용할 데이터베이스를 생성하고, 권한을 부여했다.

워드프레스를 설치할 모든 준비가 끝났다.

> cd /var/www/html </br> wget https://wordpress.org/latest.tar.gz </br> tar -xvzf latest.tar.gz </br> cd wordpress </br> mv wp-config-sample.php wp-config.php </br> vi wp-config.php

리눅스의 웹 기본 폴더인 `/var/ww/html`로 가서, 워드프레스를 다운받는다.

원하는 설정을 하기 위해, 주어진 샘플파일을 이용할거다.

<img width="963" alt="image" src="https://user-images.githubusercontent.com/67845112/186035763-5192b660-03aa-4887-96a7-424106b099f0.png">

만들어뒀던 데이터베이스를 이용한다.

lighttpd의 설정을 따로 건드리지 않았으므로, 포트는 그대로 80을 이용한다.
> ufw allow 80

virtualbox에서 포트포워딩을 하고나서, hostip/wordpress에 접속하면 반가운 화면을 볼 수 있다.
<img width="618" alt="image" src="https://user-images.githubusercontent.com/67845112/186038841-84ee99cc-52f6-459c-9668-7daf4fb5d888.png">

<img width="1819" alt="image" src="https://user-images.githubusercontent.com/67845112/186038955-b6e991bc-8cd3-4318-8fc3-80c87b73e246.png">



- - -
</details>

<img width="707" alt="image" src="https://user-images.githubusercontent.com/67845112/188308267-bca6ecea-9f98-45db-9825-5f01d59f5be1.png">

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fr4pidstart%2Fft_born2beroot&count_bg=%23000000&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
