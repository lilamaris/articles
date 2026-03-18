K8S 에 사설 도커 레지스트리를 올렸는데, 이미지를 만들고, 레지스트리에 Push 할 때마다 Payload too large 413 에러를 받았다. 

클러스터 서버는 앞 단에 Cloudflare 를 두고 있고, 내 도메인에 와일드카드로 proxied 설정을 해놓은 상태인데, 이 경우 Cloudflare에서 413 에러를 응답하는 것을 확인했다.

registry.<도메인> 에 대해서 proxy 설정을 끈 Record를 따로 만들고 해결.

