# 환경

- OS: Debian 13 (Trixie)
- GPU: NVIDIA Tesla V100 SXM2 16GB x 2
- 독점 소프트웨어 허용(`nvidia-driver`)

```bash
Linux <host name> 6.12.69+deb13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.69-1 (2026-02-08) x86_64 GNU/Linux
```

# 사전 준비

Debian은 기본적으로 자유 소프트웨어만을 제공합니다. NVIDIA 드라이버와 같은 독점 소프트웨어를 설치하기 위해서는 저장소를 따로 추가해야합니다.

OS 설치 과정에서 non-free software 저장소 설정을 하거나, 기존 환경에서는 `/etc/apt/sources.list`를 수정할 수 있습니다.

## 저장소 추가

`non-free`와 `non-free-firmware` 구성 요소를 추가합니다.

# 의존성 패키지 목록
| 패키지 명                       | 설명                                                                   |
| --------------------------- | -------------------------------------------------------------------- |
| `linux-headers-$(uname -r)` | 현재 사용 중인 커널에 맞는 헤더 파일로, 드라이버 모듈을 커널에 빌드할 때 필요                        |
| `build-essential`           | C/C++ 컴파일러(gcc) 세트로, 드라이버 소스 코드를 빌드하기 위해 필요한 기초 도구                   |
| `dkms`                      | Dynamic Kernel Module Support. 커널 업데이트 시 자동으로 드라이버 모듈을 빌드해서 연결해주는 도구 |
# 설치 패키지 목록

| 패키지 명                 | 설명                                                              |
| --------------------- | --------------------------------------------------------------- |
| `nvidia-driver`       | GPU 하드웨어와 OS를 연결하는 드라이버                                         |
| `firmware-nvidia-gsp` | 최신 NVIDIA GPU에 포함된 GSP(GPU System Processor)용 펌웨어로 드라이버 로드 시 필요 |
| `nvidia-smi`          | NVDIA System Management Interface. GPU 상태 모니터링 및 설정을 위한 CLI 도구  |
# 드라이버 설치

## Nouveau 드라이버 비활성화

OS 설치 시 포함된 `nouveau` 오픈소스 GPU 드라이버가 커널을 점유하고 있으면, `nvidia-driver`와 서로 경쟁하게 되서 드라이버가 로드되지 않을 수 있습니다.

### `nvidia-driver` 패키지 설치 시 확인해볼 사항
- 설치 마무리 과정에서 `update-initramfs` 관련 메시지 표시 확인
	- `sudo update-initramfs -u`를 실행해서 초기 부팅 이미지에서 nouveau 를 수동으로 제거할 수 있습니다.
- 설치 완료 후, `/etc/modprobe.d/nvidia-blacklists-nouveau.conf` 파일 생성을 확인하고, `blacklist nouveau`가 기록됐는지 확인
	- 직접 파일을 생성하고 기록할 수 있습니다.

패키지 설치 및 재부팅 이후 `nvidia-smi` 명령으로 설치된 드라이버 버전 및 인식된 하드웨어를 모니터링할 수 있습니다.

# 드라이버 설치 과정 트러블슈팅
r
필요한 패키지를 모두 설치 및 재부팅 후 `nvidia-smi` 명령에서 문제가 발생할 수 있습니다.
```bash
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
```

## 장치 인식 확인

PCI 장치 스캔 결과에서 GPU 하드웨어가 인식되는지 확인해볼 수 있습니다.
```bash
lspci | grep -i nvidia
```

기대 출력
```bash
04:00.0 3D controller: NVIDIA Corporation GV100GL [Tesla V100 SXM2 16GB] (rev a1)  
05:00.0 3D controller: NVIDIA Corporation GV100GL [Tesla V100 SXM2 16GB] (rev a1)
```
## Secure Boot

시스템이 Secure Boot가 활성화돼있다면, 서명되지 않은 독점 소프트웨어의 로드가 차단될 수 있습니다.

- `mokutil --sb-state` 명령으로 현재 Secure Boot 옵션이 활성화됐는지 확인
- 시스템 재부팅 후 BIOS 설정으로 진입
	- Secure Boot 옵션을 비활성화
	- 저장 후 재부팅하여 `nvidia-smi` 명령 실행

## 드라이버 빌드 확인

드라이버 패키지 설치 후 `dkms`에 의해 커널 버전에 맞춰서 빌드되지 않을 수 있습니다.

DKMS 상태 확인
```bash
sudo dkms status
```

기대 출력
```bash
nvidia-current/550.163.01, 6.12.69+deb13-amd64, x86_64: installed
```

목록이 없거나 에러가 발생한다면 강제 빌드를 시도해볼 수 있습니다.
```bash
sudo dpkg-reconfigure nvidia-kernel-dkms
```

트러블 슈팅 과정 상에서 출력 결과는 드라이버가 추가만 된 상태였습니다.
```bash
nvidia-current/550.163.01: added
```

## 커널 모듈 로드 확인

드라이버가 커널에 로드됐는지 확인해볼 수 있습니다.

명령 실행
```bash
lsmod | grep nvidia
```

기대 출력
```
nvidia_uvm           4915200  0  
nvidia              60702720  1 nvidia_uvm  
drm                   774144  12 gpu_sched,drm_kms_helper,drm_exec,drm_suballoc_helper,drm_display_helper,nvidia,drm_buddy,amdgpu,drm_ttm_helper,ttm,amdxcp
```

출력 결과가 없다면 드라이버가 로드되지 않은 것, 반대로 결과가 있다면 통신에 문제가 있다고 볼 수 있습니다.

아무 출력 결과가 없다면 수동으로 로드를 시도해볼 수 있습니다.
```bash
sudo modprobe nvidia
```

수동으로 로드하는 과정에서도 문제가 발생할 수 있습니다.

에러 전문
```bash
modprobe: FATAL: Module nvidia-current not found in directory /lib/modules/6.12.69+deb13-amd64  
modprobe: ERROR: Error running install command 'modprobe -i nvidia-current ' for module nvidia: retcode 1  
modprobe: ERROR: could not insert 'nvidia': Invalid argument
```

`dkms`가 현재 사용 중인 커널용으로 드라이버 빌드를 완료하지 못했고, 모듈을 찾을 수 없었습니다.
```bash
modprobe: FATAL: Module nvidia-current not found in directory 
```

`nvidia` 모듈 로드 시 `nvidia-current`도 같이 로드하는 체인에서, `nvidia-current` 로드가 실패했으므로, 체인 전체가 중단됐습니다.
```bash
modprobe: ERROR: Error running install command 'modprobe -i nvidia-current ' for module nvidia: retcode 1  
```

최종적으로 `nvidia` 모듈을 커널에 로드하는 과정이 실패했다는 결과 보고가 있습니다.
```bash
modprobe: ERROR: could not insert 'nvidia': Invalid argument
```

### 드라이버 빌드 시도

`dkms status` 출력 결과 상 드라이버 버전으로 빌드를 시도할 수 있습니다.
```bash
sudo dkms build -m nvidia-current -v 550.163.01
```

빌드 완료 후 커널에 설치합니다.
```bash
sudo dkms install -m nvidia-current -v 550.163.01
```

설치 이후, `dkms status` 출력 결과에서 드라이버가 `installed` 상태로 표시된다면, 로드를 시도할 수 있습니다.
```bash
sudo modprobe nvidia
```

재부팅 없이 `nvidia-smi` 명령을 실행해볼 수 있습니다.

# GPU 최적화

`nvidia-smi`의 `Persistence Mode`를 활성화하면, 드라이버가 메모리에 상주하여 응답 속도를 향상시킬 수 있습니다.
GPU 클럭이 전압 대비 비선형적 증가 추이를 보이는 장비 특성 상 최대 전력 소모량을 제한하여 전성비를 개선할 수 있습니다.


`nvidia-smi`를 통해 `Persistence Mode`를 활성화하고 최대 전력 소모량을 제한하는 `systemd` 서비스를 작성합니다. 여기서 V100 SXM2 16GB의 기본 TDP는 300W이고, 최대 전력 소모량을 200W로 제한하고 있습니다.
```ini title="/etc/systemd/system/nvidia-monitor.service"
[Unit]
Description=NVIDIA GPU Persistence and Power Limit Setup
After=syslog.target network.target

[Service]
Type=oneshot
RemainAfterExit=yes
# Persistence Mode 활성화
ExecStart=/usr/bin/nvidia-smi -pm 1
# Power Limit 설정
ExecStart=/usr/bin/nvidia-smi -pl 200

[Install]
WantedBy=multi-user.target
```

서비스를 활성화하고 실행할 수 있습니다.
```bash
sudo systemctl daemon-reload
sudo systemctl enable nvidia-monitor.service
sudo systemctl start nvidia-monitor.service
```
