<details>
    <summary>UFW가 뭔가요?</summary>
  
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

이 외의 설정을 하려면, 모듈 설치가 필요하다.
> apt install libpam-pwquality

`etc/security/pwquality.conf`에서 설명에 따라 적절히 바꿔준다.

- - -
</details>
