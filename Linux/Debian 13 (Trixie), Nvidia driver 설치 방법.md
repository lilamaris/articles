K8S 클러스터에서 GPU 자원을 활용하기 위해 `nvidia/gpu-operator`를 추가하던 도중, 파드에서 `nvidia-smi` 명령을 실행하지 못하고 서버의 메모리 자원을 모두 소모하다 OOM 이 발생했습니다.

호스트 드라이버로 `nvidia-driver-550`이 설치된 환경이었고, `nvidia/gpu-operator`는 헬름으로 설치했었습니다.

초기 헬름 설치 파라미터
```bash
helm install gpu-operator nvidia/gpu-operator -n gpu-operator \
--create-namespace \
--set driver.enabled=false
```
## 드라이버 업데이트 전 시도할만한 방법
### 구성 요소 비활성화 후 재설치

```bash
helm install gpu-operator nvidia/gpu-operator -n gpu-operator \
--create-namespace \
--set driver.enabled=false \
--set dcgmExporter=false \
--set toolkit=false \
--set migManager=false \
--set nodeStatusExporter=false
```

### Secure Boot 비활성화

Secure Boot 활성화된 호스트였으나, `nvidia-driver-550` 빌드 후 dkms 키를 mokutil로 등록한 후, 호스트에서는 `nvidia-smi`로 통신이 가능한 상황이었습니다. 따라서 싱글 노드 클러스터의 같은 호스트 내 파드에서도 정상 동작을 기대했습니다.

### BIOS 업데이트
당시 사용 중이던 서버의 마더보드 `ASUS ProArt B650-CREATOR`가 펌웨어 버전 1517을 사용 중이었고, 작성 시점 최신 3513 버전 AGESA(ComboAM5 PI_Pre1.3.0.0)로 업데이트 했습니다.

## 드라이버 설치

[[Debian 13 (Trixie), NVIDIA V100 드라이버 설치 및 최적화#의존성 패키지 목록]]를 진행했다는 전제를 두고 있습니다.

### 레포지토리 추가

Debian13 Non-free 패키지 저장소에서는 550 버전의 드라이버를 제공합니다. 보다 높은 버전의 드라이버를 설치하기 위해서는 [Nvidia에서 제공하는 Debian 용 레포지토리](https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/)를 추가할 수 있습니다.

```bash
wget -O /tmp/cuda-keyring.deb \
  https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb

sudo dpkg -i /tmp/cuda-keyring.deb
sudo apt update
```

2025년 12월 말 발표된 590 드라이버부터 Volta, Pascal, Maxwell 아키텍처 또한 레거시로 되고 지원이 중단됐고, 이로 인해 이전 버전을 설치해야한다면 Debian 12 레포지토리를 추가할 수 있습니다.

```bash
wget -O /tmp/cuda-keyring.deb \
  https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb
  
sudo dpkg -i /tmp/cuda-keyring.deb
sudo apt update
```

### Pinning 패키지 설치

특정 버전의 드라이버 패키지 설치를 추적하기 위한 pinning 패키지를 확인할 수 있습니다.

```bash
apt-cache search nvidia-driver-pinning | head -n 50
apt-cache policy nvidia-driver-pinning-580
```

기대 응답
```bash
nvidia-driver-pinning-580 - APT driver pinning file for driver branch 580  
nvidia-driver-pinning-580.105.08 - APT driver pinning file for driver version 580.105.08  
nvidia-driver-pinning-570.211.01 - APT driver pinning file for driver version 570.211.01  
nvidia-driver-pinning-570 - APT driver pinning file for driver branch 570  
nvidia-driver-pinning-580.126.09 - APT driver pinning file for driver version 580.126.09  
nvidia-driver-pinning-580.126.16 - APT driver pinning file for driver version 580.126.16
```

```bash
apt-cache policy nvidia-driver-pinning-580
nvidia-driver-pinning-580:
  Installed: 580-5
  Candidate: 580-5
  Version table:
 *** 580-5 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64  Packages
        100 /var/lib/dpkg/status
     580-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64  Packages
```

Pinning 패키지를 설치해서 apt 시스템이 특정 버전의 드라이버를 추적할 수 있습니다.

```bash
sudo apt install -y nvidia-driver-pinning-580
```

## 드라이버 패키지 설치

데이터센터 카드의 경우 오픈 커널(`nvidia-open`) 패키지 호환 대상이 아니므로 독점 패키지를 설치해야합니다.

```bash
sudo apt install -y nvidia-driver nvidia-kernel-dkms
```

## APT Update 시 키 검증 실패

```bash
Policy rejected non-revocation signature (PositiveCertification) requiring second pre-image resistance because: SHA1 is not considered secure since
```

Debian 13 의 apt 에서 사용하는 sqv 검증기가 2026년 2월초부터 SHA-1 기반 서명을 거부합니다. Debian 12 CUDA 레포지토리의 키가 아직 SHA-1으로 서명되어있다면 설치가 불가능한 상황일 수 있습니다.

작성 시점 해결할 수 있는 방안은 아래와 같습니다.

1. apt 검증을 sqv 대신 gpgv를 사용하도록 바꾸기
2. 직접 .deb 파일을 받아서 설치 및 홀드하기

### gpgv 사용하기

```bash
cat <<'EOF' | sudo tee /etc/apt/apt.conf.d/99force-gpgv
APT::Key::gpgvcommand "/usr/bin/gpgv";
EOF
sudo apt update
```