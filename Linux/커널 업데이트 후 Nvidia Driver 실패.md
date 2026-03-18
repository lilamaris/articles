Debian 13에서 패키지 업그레이드 목록에 리눅스 커널이 있었나보다.

`6.12.69+deb13-amd64` 버전에서 `6.12.73+deb13-amd64` 버전으로 업데이트 됐다.

그 이후 nvidia-smi 가 드라이버/라이브러리 간 통신 실패를 보고했고,

이를 해결하고자 업데이트 전 스냅샷으로 롤백을 했는데, Emergence Boot 모드로 진입했다. Grub  부트 옵션에서 요구하는 리눅스 커널 버전 `6.12.73+deb13-amd64`이 해당 스냅샷에 존재하지 않은 것이 원인이라고 생각한다.

이 경우에는 어떻게 할 수 있을까? 이전 버전의 커널로 모듈들을 전부 다시 빌드해서 롤백을 성공시키거나,  아니면 Grub 부트 옵션에서 이전 스냅샷의 리눅스 커널 버전을 명시해서 부트를 시도해볼 수 있을 것 같다.

하지만 귀찮아서 나는 그냥 새 커널 버전에 맞춰서 문제가 있는 모듈들만 수정하기로 했다.

nvidia-smi 는 현재 커널 버전, 설치된 nvidia-driver 버전, dkms가 성공적으로 커널 버전에 맞춰서 빌드한 모듈이 있어야 동작할 수 있다. 커널 업데이트 후 버전이 아래와 같이 버전 불일치 상황을 확인할 수 있다.

```bash
uname -r  
> 6.12.73+deb13-amd64

sudo dkms status    
> nvidia/580.126.20, 6.12.69+deb13-amd64, x86_64: installed
```

```bash
sudo apt update
sudo apt install --reinstall nvidia-driver nvidia-kernel-dkms nvidia-smi
sudo apt install --reinstall linux-headers-$(uname -r)
```

```bash
sudo dkms autoinstall  
Sign command: /lib/modules/6.12.73+deb13-amd64/build/scripts/sign-file  
Signing key: /var/lib/dkms/mok.key  
Public certificate (MOK): /var/lib/dkms/mok.pub  
  
Autoinstall of module nvidia/580.126.20 for kernel 6.12.73+deb13-amd64 (x86_64)  
Building module(s).......... done.  
Signing module /var/lib/dkms/nvidia/580.126.20/build/nvidia.ko  
Signing module /var/lib/dkms/nvidia/580.126.20/build/nvidia-modeset.ko  
Signing module /var/lib/dkms/nvidia/580.126.20/build/nvidia-drm.ko  
Signing module /var/lib/dkms/nvidia/580.126.20/build/nvidia-uvm.ko  
Signing module /var/lib/dkms/nvidia/580.126.20/build/nvidia-peermem.ko  
Installing /lib/modules/6.12.73+deb13-amd64/updates/dkms/nvidia.ko.xz  
Installing /lib/modules/6.12.73+deb13-amd64/updates/dkms/nvidia-modeset.ko.xz  
Installing /lib/modules/6.12.73+deb13-amd64/updates/dkms/nvidia-drm.ko.xz  
Installing /lib/modules/6.12.73+deb13-amd64/updates/dkms/nvidia-uvm.ko.xz  
Installing /lib/modules/6.12.73+deb13-amd64/updates/dkms/nvidia-peermem.ko.xz  
Running depmod.... done.  
  
Autoinstall on 6.12.73+deb13-amd64 succeeded for module(s) nvidia.
```

```bash
sudo update-initramfs -u -k all  
update-initramfs: Generating /boot/initrd.img-6.12.73+deb13-amd64  
update-initramfs: Generating /boot/initrd.img-6.12.69+deb13-amd64  
update-initramfs: Generating /boot/initrd.img-6.12.43+deb13-amd64
```

