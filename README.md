# NFS 스토리지

- 네트워크로 파일공유
- 네트워크 파일시스템
- 서버에서 공유할 디렉토리 준비
- 클라이언트에서 마운트 방식으로 연결

```cpp
# dnf install -y nfs-utils // 첫번째로 패키지 설정

#mkdir /share  //공유할 디렉토리 생성

# echo "test nfs" > /share/test.txt 
[root@vm1 ~]# cat /share/test.txt  //디렉토리 준비 
test nfs
[root@vm1 ~]# vim /etc/nfs.conf
[root@vm1 ~]# vim /etc/exports	

exports 편집
ㄴ> #Directory      Target  Option
/share          192.168.10.100/32(rw,sync) //읽기 쓰기 동기화 옵션 추가, share 파일 등록

// # grep -v '#' /etc/exports
// /share          192.168.10.100/32(rw,sync)  //지정 확인 방법

# exportfs -r  //테이블 리로드 
# systemctl status nfs-server.service  //서비스 재시작 
○ nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
             
    //vim /etc/nfs.conf 직접수정가능
    
    
#systemctl enable --now nfs-server   //서버설정 완료
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.

**+방화벽 설정**

# dnf install -y nfs-utils   +패키지 설정

**마운트 지정**
// # mount -t nfs -o rw,sync,sec=sys SERVERIP:PATH M/P
# mount -t nfs -o rw,sync 192.168.10.10:/share /mnt  //대상의 서버의 ip주소가 포함되고 마운트 디렉토리, 방화벽도 열어야 함 만족하면 명령어로 잘 돌아갈것임

```

실습06/04

```cpp
가상머신 3대 중
하나는 NFS서버 역할 , 하나는 접근안되는 시스템 , 하나는 접근 가능한 시스템

1. NFS 서버 구성
	패키지 설치
	디렉토리 공유 ( /nfs )
	기타사용자 쓰기권한 추가
	공유 설정 시 시스템 하나만 접근 가능하게 설정 (옵션은 rw)
	

# mkdir /share2
[root@vm1 ~]# echo "test nfs" > /share2/test.txt

# ls -l /share2
total 4
-rw-r--r--. 1 root root 9 Jun  4 07:27 test.txt
# cat /share2/test.txt  //확인 
test nfs

# vim /etc/exports //공유하기 위해서는 export수정필요

수정하기
#Directory      Target  Option
/share2         192.168.10.20/32(rw,sync)   //20 번 클라이언트만 접근 가능하게 설정
                #192.168.10.0/24
                # *
~                                                                                                  ~                         

# exportfs -r

//서버에 설정 해놓고 각 클라이언트에서 마운트 

2. 각 시스템에서 마운트 시도 (하나만 가능)

//20번 서버에서 접근하면 접근됨
# mount -t nfs -o rw,sync 192.168.10.10:share2 /mnt

//100번 서버에서 접근하면 접근이 불가능함 
# mount -t nfs -o rw,sync 192.168.10.10:nfs /mnt
mount.nfs: **access denied** by server while mounting 192.168.10.10:nfs

3. 마운트 후 파일 생성 후 서버에서 확인

4. 마운트 설정을 /etc/fstab 에 설정하고 재부팅(디렉토리에 접근할때 마운트)

# umount /mnt
# vim /etc/fstab
192.168.10.10:/nfs      /mnt    nfs     defaults        0       0

# mount -a
# mount |grep /mnt
# df -h 

# reboot
# mount |grep /mnt 
```
