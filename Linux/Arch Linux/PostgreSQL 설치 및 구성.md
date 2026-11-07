# 설치

`postgresql` 패키지는 `Extra` 레포지토리에 있다

```bash
sudo pacman -S postgresql
```

설치 후 권한 상승 프로그램을 사용해서 *postgres* 사용자로 전환

```bash
sudo -iu postgres
```

> 현재 접속한 사용자가 `/etc/sudoers` 에 있어야한다.

PostgreSQL를 사용하기 전에 반드시 데이터베이스 클러스터가 초기화되야한다.

```bash
[postgres]$ initdb -D /var/lib/postgres/data
```
```bash
[postgres]$ initdb ---locale=C.UTF-8 --encoding=UTF8 -D /var/lib/postgres/data --data-checksums
```

`postgresql.service` enable 및 start

```bash
sudo systemctl enable postgresql.service
sudo systemctl start postgresql.service
```

