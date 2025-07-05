---
layout: post
title: "리눅스 환경에서 단말의 메모리 사용량 분석"
description: "리눅스 환경에서 단말의 메모리 사용량을 확인하고 분석하는 방법에 대해 알아본다."
author: "hjs1212"
categories:
    - linux
tags:
    - memory
    - troubleshooting
date: 2025-07-05
---

# 개요
리눅스 환경에서 개발을 하다가 갑자기 editor나 terminal이 느려지거나 ssh 접속이 정상적으로 되지 않는 현상이 발생  
local에 접속하여 확인 결과 사용 메모리가 부족한 상황이었으며 이를 해결하기 위해 확인하였던 사항들을 정리


# 확인 사항
## free
```bash
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.8G        5.2G        146M         64M        2.4G        2.2G
Swap:          5.1G           0        5.1G
```
현재 단말의 메모리 사용량을 확인하는 명령어이며 `-h` 옵션을 통해 사람이 읽기 쉬운 형태로 출력
각 필드의 의미는 다음과 같음
- `total`: 총 메모리 용량
- `used`: 사용 중인 메모리 용량
- `free`: idle 메모리 용량
- `shared`: 여러 프로세스가 공유하는 메모리 용량
- `buff/cache`: 버퍼와 캐시로 사용 중인 메모리 용량
- `available`: 현재 프로세스가 사용할 수 있는 메모리 용량  

total은 used + free + buff/cache로 구성되어 있으며, used는 shared를 포함하며, Available은 free + 일부 buff/cache로 구성  
따라서 성능 이슈가 발생한 경우 `used`를 통하여 실제 사용 중인 메모리 확인 및 `available`을 통하여 현재 사용 가능한 메모리를 확인

## MemInfo
```bash
$ cat /proc/meminfo
MemTotal:        8011788 kB
MemFree:          148612 kB
MemAvailable:    2268568 kB
Buffers:            2668 kB
Cached:          2270092 kB
SwapCached:            0 kB
Active:            91388 kB
Inactive:        2260644 kB
Active(anon):      13300 kB
Inactive(anon):   132084 kB
Active(file):      78088 kB
Inactive(file):  2128560 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       5242876 kB
SwapFree:        5242876 kB
Dirty:               388 kB
Writeback:             0 kB
AnonPages:         79368 kB
Mapped:            32296 kB
Shmem:             66064 kB
Slab:             199524 kB
SReclaimable:     164488 kB
SUnreclaim:        35036 kB
KernelStack:        2944 kB
PageTables:         4704 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     9248768 kB
Committed_AS:     357396 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      178616 kB
VmallocChunk:   34359547900 kB
HardwareCorrupted:     0 kB
AnonHugePages:     16384 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:       98240 kB
DirectMap2M:     7241728 kB
DirectMap1G:     3145728 kB
```
단말의 메모리 사용 내용을 기록하고 았눈 파일, `/proc/`에 존재하며 `free` 멸령의 출력 내용 또한 해당 파일을 읽어옴
각 필드의 의미는 다음과 같음
- `MemTotal`: 총 메모리 용량
- `MemFree`: 현재 사용하지 않는 메모리 용량
- `MemAvailable`: 현재 프로세스가 사용할 수 있는 메모리 용량
- `Buffers`: 버퍼로 사용 중인 메모리 용량
- `Cached`: 캐시로 사용 중인 메모리 용량
- `SwapTotal`: 총 스왑 메모리 용량
- `SwapFree`: 현재 사용하지 않는 스왑 메모리 용량
- `Active`: 현재 사용 중인 메모리 용량
- `Inactive`: 현재 사용하지 않는 메모리 용량
- `Active(anon)`: 현재 사용 중인 익명 메모리 용량
- `Inactive(anon)`: 현재 사용하지 않는 익명 메모리 용량
- `Active(file)`: 현재 사용 중인 파일 메모리 용량
- `Inactive(file)`: 현재 사용하지 않는 파일 메모리 용량
- `Unevictable`: 스왑할 수 없는 메모리 용량
- `Mlocked`: mlock()로 잠긴 메모리 용량
- `SwapCached`: 스왑된 후 다시 메모리로 복원된 페이지
- `Dirty`: 디스크에 쓰기 대기 중인 메모리 용량
- `Writeback`: 디스크에 쓰기 중인 메모리 용량
- `AnonPages`: 익명 페이지로 사용 중인 메모리 용량
- `Mapped`: 파일에 매핑된 메모리 용량
- `Shmem`: 공유 메모리로 사용 중인 메모리 용량
- `Slab`: 커널 내부에서 사용하는 메모리 용량
- `SReclaimable`: 재사용 가능한 슬랩 메모리 용량
- `SUnreclaim`: 재사용할 수 없는 슬랩 메모리 용량
- `KernelStack`: 커널 스택으로 사용 중인 메모리 용량
- `PageTables`: 페이지 테이블로 사용 중인 메모리 용량
- `NFS_Unstable`: NFS에서 불안정한 상태의 메모리
- `Bounce`: 바운스 버퍼로 사용 중인 메모리 용량
- `WritebackTmp`: 임시로 쓰기 대기 중인 메모리 용
- `CommitLimit`: 현재 시스템에서 사용할 수 있는 최대 메모리 용량
- `Committed_AS`: 현재 프로세스가 할당한 메모리 용량
- `VmallocTotal`: 가상 메모리의 총 용량
- `VmallocUsed`: 현재 사용 중인 가상 메모리 용량
- `VmallocChunk`: 가상 메모리에서 사용 가능한 최대 연속 블록
- `HardwareCorrupted`: 하드웨어 오류로 인해 손상된 메모리 용
- `AnonHugePages`: 익명 거대 페이지로 사용 중인 메모리 용량
- `HugePages_Total`: 전체 거대 페이지 수
- `HugePages_Free`: 현재 사용 가능한 거대 페이지 수
- `HugePages_Rsvd`: 예약된 거대 페이지 수
- `HugePages_Surp`: 초과 거대 페이지 수
- `Hugepagesize`: 거대 페이지의 크기
- `DirectMap4k`: 4KB 페이지로 매핑된 메모리 용
- `DirectMap2M`: 2MB 페이지로 매핑된 메모리 용
- `DirectMap1G`: 1GB 페이지로 매핑된 메모리 용

`MemTotal`은 free의 `total`과 동일하며, `MemFree`는 free의 `free`, `MemAvailable`은 free의 `available`과 동일하며 `Buffers`와 `Cached`는 free의 `buff/cache`와 동일  
`slab`은 커널 내부에서 사용하는 메모리로 `slabtop` 명령어를 통해 확인 가능

## top
```bash
$ top
top - 05:33:17 up 3 min,  0 users,  load average: 1.17, 0.52, 0.20
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7837.6 total,   7311.3 free,    281.8 used,    244.5 buff/cache
MiB Swap:   1024.0 total,   1024.0 free,      0.0 used.   7407.9 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    1 root      20   0    4136   2888   2632 S   0.0   0.0   0:00.01 bash
   11 root      20   0    6728   2608   2224 R   0.0   0.0   0:00.00 top
```
현재 동작 중인 프로세스의 상태 및 자원 사용량을 확인하는 명령어
- VIRT: 프로세스가 사용하는 가상 메모리의 크기
- RES: 프로세스가 실제로 사용하는 물리적 메모리의 크기
- SHR: 프로세스가 다른 프로세스와 공유하는 메모리의 크기

VIRT는 Virtual Memory의 약자로 프로세스가 사용하는 전체 가상 메모리의 크기를 나타내며, RES는 Resident Set Size의 약자로 프로세스가 실제로 사용하는 물리적 메모리의 크기를 SHR는 Shared Memory의 약자로 프로세스가 다른 프로세스와 공유하는 메모리의 크기를 나타냄  
VIRT의 경우 일반적인 상황에서도 비교적 큰 값을 가지며 그 내용자체가 문제가 되지는 않음  
RES는 실제 사용하는 크기를 나타내기에 해당 값이 크다면 문제가 발생 할 여지가 있음  
SHR의 경우 큰 값을 가질 수록 다른 프로세스와 메모리를 공유하고 있다는 의미로, 일반적으로 큰 값이 문제가 되지는 않음

## slabtop
```bash
$ slaptop
 Active / Total Objects (% used)    : 665941 / 668820 (99.6%)
 Active / Total Slabs (% used)      : 10857 / 10857 (100.0%)
 Active / Total Caches (% used)     : 64 / 91 (70.3%)
 Active / Total Size (% used)       : 76077.65K / 76767.28K (99.1%)
 Minimum / Average / Maximum Object : 0.01K / 0.11K / 8.00K

  OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME
155136 155136 100%    0.03K   1212      128      4848K kmalloc-32
122230 122230 100%    0.02K    719      170      2876K fsnotify_event_holder
 77824  77824 100%    0.01K    152      512       608K kmalloc-8
 59840  59503  99%    0.06K    935       64      3740K kmalloc-64
 49182  49182 100%    0.19K   2342       21      9368K dentry
 47616  47616 100%    0.02K    186      256       744K kmalloc-16
 43299  43280  99%    0.08K    849       51      3396K selinux_inode_security
 22644  22644 100%    0.11K    629       36      2516K sysfs_dir_cache
 20528  20528 100%    1.00K   1283       16     20528K xfs_inode
 18954  18954 100%    0.58K    702       27     11232K inode_cache
  5432   5432 100%    0.07K     97       56       388K Acpi-ParseExt
  4158   4029  96%    0.21K    231       18       924K vm_area_struct
  3978   3978 100%    0.04K     39      102       156K Acpi-Namespace
  3968   2460  61%    0.06K     62       64       248K anon_vma
  3948   3948 100%    0.09K     94       42       376K kmalloc-96
  3888   3600  92%    0.25K    243       16       972K kmalloc-256
  2752   2752 100%    0.12K     86       32       344K kmalloc-128
  2730   2730 100%    0.10K     70       39       280K buffer_head
  2625   2326  88%    0.19K    125       21       500K kmalloc-192
  2424   2338  96%    0.64K    101       24      1616K proc_inode_cache
  2380   2380 100%    0.05K     28       85       112K shared_policy_node
  1960   1960 100%    0.57K     70       28      1120K radix_tree_node
  1533   1533 100%    0.38K     73       21       584K blkdev_requests
  1504   1504 100%    1.00K     94       16      1504K kmalloc-1024
   888    888 100%    0.66K     37       24       592K shmem_inode_cache
   728    728 100%    0.15K     28       26       112K xfs_ili
   672    657  97%    0.50K     42       16       336K kmalloc-512
   646    646 100%    0.81K     34       19       544K task_xstate
   624    577  92%    2.00K     39       16      1248K kmalloc-2048
   420    322  76%    1.12K     15       28       480K signal_cache
   363    310  85%    2.84K     33       11      1056K task_struct
   292    292 100%    0.05K      4       73        16K ip_fib_trie
   285    285 100%    2.06K     19       15       608K sighand_cache
   275    275 100%    0.62K     11       25       176K sock_inode_cache
   264    264 100%    4.00K     33        8      1056K kmalloc-4096
   256    256 100%    0.06K      4       64        16K kmem_cache_node
   204    204 100%    0.94K     12       17       192K RAW
   195    195 100%    2.06K     13       15       416K idr_layer_cache
   170    170 100%    0.12K      5       34        20K fsnotify_event
   162    162 100%    0.44K      9       18        72K scsi_cmd_cache
   156    156 100%    0.10K      4       39        16K blkdev_ioc
   128    128 100%    0.25K      8       16        32K kmem_cache
   120    120 100%    1.56K      6       20       192K mm_struct
   112    112 100%    1.12K      4       28       128K UDPv6
   104    104 100%    0.30K      4       26        32K nf_conntrack_ffffffff819a09c0
   100    100 100%    0.62K      4       25        64K files_cache
   ```
slab 메모리 사용량을 확인하는 명령어로, 이때 slab memory는 kernel 내부에서 사용하는 메모리를 의미   
해당 명령은 root 권한으로 실행되어야 하며, 각 필드의 의미는 다음과 같음
- `OBJS`: 현재 slab에 할당된 객체의 수
- `ACTIVE`: 현재 slab에서 사용 중인 객체의 수
- `USE`: 사용률
- `OBJ SIZE`: 객체의 크기
- `SLABS`: slab의 수
- `OBJ/SLAB`: slab당 객체의 수
- `CACHE SIZE`: slab의 크기
- `NAME`: slab의 이름
- `Active / Total Objects (% used)`: 현재 slab에서 사용 중인 객체와 전체 객체의 비율
- `Active / Total Slabs (% used)`: 현재 slab에서 사용 중인 slab과 전체 slab의 비율
- `Active / Total Caches (% used)`: 현재 slab에서 사용 중인 캐시와 전체 캐시의 비율
- `Active / Total Size (% used)`: 현재 slab에서 사용 중인 크기와 전체 크기의 비율
- `Minimum / Average / Maximum Object`: slab에서 사용되는 객체의 최소, 평균, 최대 크기  

해당 명령어 사용 시 `Objects`나 `Active`의 수가 지나치게 크거나 `Size`가 지나치게 크다면 kernel 내부 memory 사용 과정에 문제가 있을 수 있음

# 시나리오
## Available 메모리 부족
해당 시나리오의 경우 `buff/cache`가 증가하는 경우라면 큰 문제는 아닐 수 있지만, `used`가 증가하는 경우라면 문제가 발생할 수 있음   
`used`가 증가하고 있는 경우라고 판단되는 경우 해당 증가량이 어디서 이뤄지고 있는 지 확인이 필요  
`/proc/meminfo`를 확인하여 slab memory와 관련 된 영역(`slab`, `SReclaimable`, `SUnreclaim`)이 비정상적으로 크거나 지속적으로 증가하고 있다면 kernel 내부 문제가, user space와 관련된 영역(`Buffers`, `Cached`, `Active`, `Inactive`)가 비정상적으로 크거나 지속적으로 증가하고 있다면 동작 중인 프로세스가 원인일 가능성이 있음

### user space에서 문제가 발생하는 경우
해당 경우에는 대부분 개발자의 실수가 원인이 되기에 어느 프로세스에서 메모리를 많이 사용하고 있는지 확인이 필요  
`top` 명령어과 `shift + m`을 통해 메모리 사용량 기준으로 정렬하여 어느 프로세스가 메모리를 많이 사용하고 있는지 확인  
프로세스가 특정되었다면 프로세스 메모리 사용 분석 기법을 통해 해당 프로세스가 메모리를 많이 사용하고 있는 이유를 분석

### kernel 내부에서 문제가 발생하는 경우
해당 경우에는 개발자의 실수도 있을 수 있지만, 대부분 kernel 내부에서 문제가 발생하는 경우도 존재  
`dentry`나 `inode`가 많은 메모리를 사용하고 있다면 파일시스템에 문제를, `kmalloc`이 많이 사용하고 있다면 커널 내부의 버그를 의심 할 수 있음  

## swap 사용량 증가
충분히 사용가능한 메모리가 있음에도 불구하고 swap의 사용량이 지속적으로 증가한다면 memory leak이 발생하고 있는 것일 수 있음  
해당 경우에는 `top` 명령어를 통해 RES가 지속적으로 증가하고 있는 프로세스를 확인하고, 해당 프로세스의 메모리 사용 분석 기법을 통해 메모리 누수가 발생하고 있는지 확인  