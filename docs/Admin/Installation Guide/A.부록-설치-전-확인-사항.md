# A.부록: 설치 전 확인 사항

### 사용자 계정의 리소스 한계 값 확인

OS 명령어인 “ulimit”으로 사용자 계정에 설정된 리소스 한계값을 확인 또는 변경할 수 있다.

- File Size  
:	프로세스가 생성 가능한 파일의 최대 크기

- Data segment size  
:    프로세스가 사용 가능한 논리적 메모리의 최대 크기(VSZ측면)

- Max memory size  
:    프로세스가 사용 가능한 물리적 메모리의 최대 크기(RSS측면)

- Open files (descriptor)  
:    프로세스가 동시에 접근 가능한 파일 및 소켓의 최대 개수

- Stack size
:    최대 스택 사이즈

- Virtual memory  
:    프로세스가 사용 가능한 가상 메모리의 최대 크기

유닉스 시스템은 사용자 계정의 리소스 한계값들을 "unlimited"로 설정할 것을 권장한다. 이 때 core file size는 unlimited로 설정하지 않도록 한다. 만일 Altibase 서버가 비정상 종료하여 코어를 덤프할 경우 메모리 데이터베이스를 모두 core 파일로 저장하기 때문에 unlimited로 설정하면 디스크 부족이 발생할 수 있다. Altibase 클라이언트 제품은 Stack size가 최소 70KB 이상이어야 한다.

### OS별 커널 파라미터 설정

OS별로 제공되는 유틸리티를 사용해서 시스템 커널 파라미터 값을 확인 또는 변경할 수 있다.

시스템 커널 파라미터는 크게 다음과 같이 분류된다.

- Semaphore  
:    IPC접속을 위한 세마포어 설정

- File-cache  
:    운영체제의 File-cache 확보로 인한 메모리 부족을 방지하기 위한 설정

- 기타 설정

#### HP-UX

##### 설정 방법

HP 11.31 이상: /usr/sbin/kctune 유틸리티를 사용하여 설정

##### 권장 값

<table>
	<tbody>
		<tr>
			<th>분류</th>
			<th>커널 항목</th>
			<th>권장 값(bytes)</th>
		</tr>
		<tr>
			<td rowspan="3">공유 메모리</td>
			<td>shmmax</td>
			<td>2G+1</td>
		</tr>
		<tr>
			<td>shmmin</td>
			<td>500</td>
		</tr>
		<tr>
			<td>shmseg</td>
			<td>200</td>
		</tr>
		<tr>
			<td rowspan="9">세마포어</td>
			<td>semmns</td>
			<td>8192</td>
		</tr>
		<tr>
			<td>semmns</td>
			<td>8192</td>
		</tr>
		<tr>
			<td>semmni</td>
			<td>5029</td>
		</tr>
		<tr>
			<td>semmsl</td>
			<td>2000</td>
		</tr>
		<tr>
			<td>semmap</td>
			<td>5024</td>
		</tr>
		<tr>
			<td>semmnu</td>
			<td>1024</td>
		</tr>
		<tr>
			<td>semopm</td>
			<td>512</td>
		</tr>
		<tr>
			<td>semume</td>
			<td>512</td>
		</tr>
		<tr>
			<td>semvmx</td>
			<td>32767</td>
		</tr>
		<tr>
			<td rowspan="2">파일 캐쉬(HP의 권고안은 8G이하에서 20%, 8G이상에서 10%이다)</td>
			<td>dbc_min_pct</td>
			<td>5%</td>
		</tr>
		<tr>
			<td>dbc_max_pct</td>
			<td>5~20%</td>
		</tr>
		<tr>
			<td rowspan="6">그 외</td>
			<td>maxdsiz</td>
			<td>2GB</td>
		</tr>
		<tr>
			<td>maxdsiz_64bit</td>
			<td>Altibase가 사용할 것으로 예측하는 메모리 용량</td>
		</tr>
		<tr>
			<td>max_thread_proc</td>
			<td>600 이상</td>
		</tr>
		<tr>
			<td>maxfiles</td>
			<td>2048 이상</td>
		</tr>
		<tr>
			<td>nproc</td>
			<td>6142</td>
		</tr>
		<tr>
			<td>maxusers</td>
			<td>124</td>
		</tr>
	</tbody>	
</table>


#### AIX

##### 설정 방법

/usr/bin/smit 유틸리티를 사용하여 설정한다.

##### 권장 값

공유 메모리 및 세마포어는 HP-UX와 동일하게 설정한다.

###### File-Cache의 설정

AIX의 경우 File-Cache의 설정 정책에 따라 시스템의 여유 메모리(Free Memory)가 있음에도 불구하고 파일 시스템이 응용 프로그램의 Heap영역에서 메모리를 Swap-out시킨 후 이를 File-Cache로 사용하는 경우가 발생할 수 있다. (이를 Steal 이라고 한다)

AIX 5.2이상 버전에서는 이러한 steal현상을 방지하기 위해 다음과 같은 커널 파라미터를 설정할 수 있다.

```bash
minperm =  5%
lru_file_repage = 0 (AIX 5.2 ML4 이상에서만 설정 가능)
strict_maxclient = 0
```

###### Posix AIO의 설정

Posix AIO는 AIX가 제공하는 디스크 처리 성능 개선 항목으로 수동으로 활성화를 해야 한다. 그러나, AIX6.1부터는 이 항목이 기본으로 활성화되어 있다.

활성화하는 방법은 다음과 같다.

-   smit에서 "Configure Defined Asynchronous I/O"를 Available로 변경한다.

-   smit에서 "STATE to be configured at system restarted" 를 Available로 변경한다.

위의 설정을 하지 않을 경우 AIX에서 Altibase의 설치 및 구동이 불가능하다.

#### LINUX

##### 설정 방법

/proc/sys/kernel 경로에 sem, shmmax, shmmni, swapiness 등의 파일에 설정한다. RedHat 7.2 이상에서는 /etc/systemd/loginid.conf에서 RemoveIPC 설정값을 확인한다.

##### 권장 값

공유 메모리 및 세마포어는 HP-UX와 동일하게 설정한다.

단, 리눅스 커널 버전이 2.5 이상이 아닐 경우 IPC접속을 사용하는 세션이 갑자기 단절되는 현상이 발생할 수 있다.

서버 부팅 시 자동으로 커널 파라미터가 설정되게 하려면, /etc/rc.d/rc.local 파일 내에 아래의 항목을 추가하라.

```bash
/etc/rc.d/rc.local 파일 내에 아래의 항목을 추가하라.
echo 2147483648 > /proc/sys/kernel/shmmax
echo 4096 > /proc/sys/kernel/shmmni
echo 200 32000 512 5029 > /proc/sys/kernel/sem
echo 5 > /proc/sys/vm/swappiness
```

###### RemoveIPC 설정

RedHat 7.2 이상의 버전에서는 RemoveIPC 설정값을 ‘no’로 설정하는 것을 권장한다(기본값은 ‘yes’). RemoveIPC가 ‘yes’로 설정되면 세마포어가 부족하여 비정상종료가 발생할 수 있기 때문이다.

설정값을 변경하려면 /etc/systemd/logind.conf에서 RemoveIPC=no로 설정한 후 OS를 재시작해야한다.

### THP 설정 확인 및 비활성화 방법

THP(Transparent Huge Pages)는 메모리 페이지의 크기를 증가시킴으로써, TLB(Translation Lookaside Buffer)를 조회하는 비용을 줄이기 위한 목적으로 리눅스에서 제공하는 메모리 관리 시스템이다. 하지만 원래 의도와 달리 메모리 할당 지연 및 단편화를 유발하여 오히려 시스템 성능이 저하되는 경우가 많다.

Altibase를 사용하기 위해 THP 옵션을 비활성화(never)로 해야 한다.

#### THP 설정 확인

THP에서 설정할 수 있는 옵션은 always, madvise, never 3가지이다. [ ] 로 둘러싸인 것이 현재 적용된 옵션이다. 각각의 의미는 아래와 같다.

-   madvise: madvise() 함수를 통해 THP 사용을 명시적으로 요청한 프로세스에만 THP가 활성화되는 옵션이다.
    
-   always: 모든 프로세스에 항상 THP가 적용되게 된다.

-   never: madvise() 함수 요청과 관계없이 모든 프로세스에서 THP가 비활성화되는 것을 의미한다.

THP 설정 확인 방법은 아래와 같다.

1. 아래 명령을 실행한다

   ```bash
   $ cat /sys/kernel/mm/transparent_hugepage/enabled
   ```

2. 레드햇 리눅스에서는 아래 명령을 실행한다

   ```bash
   $ cat /sys/kernel/mm/redhat_transparent_hugepage/enabled
   ```

3. 아래와 같은 결과가 화면에 출력된다.

   ```bash
   $ [always] madvise never
   ```

#### THP 비활성화 방법

Altibase의 운영을 위해서 THP 옵션을 never로 설정할 것을 권고한다.

1. root 계정으로 /etc/grub.conf의 kernel boot 라인 끝에 `transparent_hugepage=never`를 아래처럼 추가한다.
   
	```bash
	kernel /vmlinuz-2.6.32-220.el6.x86_64 ro root=UUID=067b9803-90ca-4875-a018-ff043adde1ed rd_NO_LUKS LANG=ko_KR.UTF-8 rd_NO_MD quiet rhgb crashkernel=128M  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_LVM rd_NO_DM transparent_hugepage=never
	```
   
2. 시스템을 재시작한다.

3. THP 옵션이 never 인지 확인한다.

### 디스크 구성 상태 확인

ALTIBASE의 디스크 I/O는 기본적으로 리두 로그 파일과 데이터 파일에서 발생한다. 디스크 I/O에 따른 성능 저하를 감소시키기 위해 물리적으로 분리된 디스크 영역에 리두 로그 파일과 데이터 파일이 분산되도록 구성할 것을 권장한다.

### OS Patch

#### Linux
glibc에서 malloc/free 등이 race condition으로 인해 deadlock이 발생할수 있는 버그가 있어, 해당 버그가 반영된 패치 이상으로 패치해야 한다. 따라서, glibc-2.12-1.166.el6_7.1 이상으로 glibc 패치를 권고한다. [참고](https://bugzilla.redhat.com/show_bug.cgi?id=1244002)

#### AIX

AIX에서 Altibase를 사용할 경우 메모리가 증가하는 현상(heapmin library bug)이
발생한다. 이를 방지하기 위해서는 [IBM Support Potal](http://www-01.ibm.com/support/docview.wss?uid=swg21110831)에서
해당 버전의 C/C++ compilers를 패치해야 한다.
