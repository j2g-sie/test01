*docker 사용이 많아 질 것 같아서, Portainer부터 설치 하는 것이 좋을 것 같다고 생각 하였습니다.* 

## **Portainer 란?**

**Portainer는 컨테이너 배포 및 관리 작업을 간소화하는 오픈소스 플랫폼**입니다. Docker와 같은 컨테이너 환경을 쉽게 다룰 수 있도록 설계되었으며, 직관적인 웹 UI를 통해 컨테이너, 이미지, 네트워크, 볼륨을 손쉽게 관리할 수 있습니다. 이 가이드는 Portainer Community Edition(CE) 2.21.5를 설치하고 설정하는 방법을 다룹니다. 흔히 Docker Desktop과 비교되기도 합니다 

공식 웹사이트: https://www.portainer.io/

설치 문서: https://docs.portainer.io/start/install-ce/server/docker/linux

초기 셋업: https://docs.portainer.io/start/install-ce/server/setup

**Docker Desktop과의 주요 차이점:**

1. **설치 및 환경 요구 사항:** Docker Desktop은 주로 Windows 및 MacOS에서 사용되며 Docker 엔진과 함께 가상화 환경을 제공합니다. 반면 Portainer는 모든 Docker 환경에서 작동하며 가볍게 설치할 수 있습니다. Docker Desktop은 네이티브 애플리케이션(Native App)으로 운영체제와 밀접하게 통합된 데 반해, Portainer는 웹 서비스(Web Service)로 작동하여 브라우저를 통해 어디서든 접근 가능합니다. 이 차이는 사용 환경과 목적에 따라 각기 다른 장점을 제공합니다.
2. **UI 및 기능:** Docker Desktop은 Docker 전용 관리 기능을 제공하지만 Portainer는 Swarm, Kubernetes와 같은 다양한 컨테이너 오케스트레이션 도구를 지원하며 더 광범위한 관리 기능을 제공합니다.
3. **리소스 사용:** Portainer는 Docker Desktop에 비해 더 적은 리소스를 사용하므로 서버 환경에서 더 적합합니다.

## 1. Docker Compose를 사용하는 이유

Docker Compose를 사용하면 여러 컨테이너와 그 설정을 코드로 정의하고 쉽게 관리할 수 있습니다. 이 방식은 명령어를 통한 설치보다 구조화된 설정을 제공하며, 구성 변경 및 배포 과정을 간소화합니다. 또한, 반복 적인 작업을 자동화하고 팀 협업 시 설정을 공유하기 용이합니다. 혼선을 가져올 수 있다고 생각되어 docker ... 직접 명령은 설명하지 않았습니다.

## 2. Docker Compose를 사용한 Portainer 구성

Docker Compose를 사용하기 전에, Portainer 데이터를 저장할 볼륨을 먼저 생성해야 합니다. 이는 데이터가 컨테이너가 삭제되거나 재 배포되더라도 유지되도록 하기 위함입니다:

```
sudo docker volume create portainer_data
```

아래 명령을 수행 하여, 결과 확인이 가능 합니다. 

```bash
docker volume ls
DRIVER    VOLUME NAME
local     portainer_data
```

위 명령을 실행한 후 아래의 내용을 통해 Portainer를 구성할 수 있습니다:

아래는 Portainer를 구성하기 위한 `docker-compose.yml` 파일입니다:

```yaml
services:
  portainer:
    image: portainer/portainer-ce:2.21.5
    container_name: portainer
    restart: always
    ports:
      - "8000:8000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
    external: true

```

Docker Compose를 사용하여 Portainer를 배포하려면:

```bash
sudo docker compose up -d

```

> 참고: 과거에는 docker-compose 명령을 사용했지만, 현재는 Docker의 최신 버전에서 docker compose로 변경되었습니다. 이는 Docker CLI의 일부로 통합되어 설치와 관리가 더욱 간소화되었습니다. 더하여, "docker-compose"는 python 명령 이였습니다.
> 

> 참고: docker-compose.yml 파일의 가장 위에 있던 version 속성은 이제 사용되지 않으므로 경고를 피하려면 생략해야 합니다. version이 있다면, Docker Compose를 실행할 때 다음과 같은 경고 메시지가 나타날 수 있습니다:
> 
> 
> ```
> WARN[0000] /home/brian/work/server_info/portainer/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
> ```
> 
> ---
> 

## 3. 실행 결과

```bash
docker compose up -d
[+] Running 1/1
 ✔ portainer Pulled                                                                                                                                                                    2.3s
[+] Running 2/2
 ✔ Network portainer_default          Created                                                                                                                                          0.3s
 ✔ Container portainer                Started                                                                                                                                        1.7s
```

## 3. 방화벽 규칙 설정

Portainer에 접근할 수 있도록 필요한 포트(8000번과 9443번)를 방화벽에서 열어야 합니다:

```bash
sudo ufw allow 9443
sudo ufw allow 8000

sudo ufw status

```

방화벽 상태의 예상 출력:

```
Status: active

To                         Action      From
--                         ------      ----
9443                       ALLOW       Anywhere
8000                       ALLOW       Anywhere
9443 (v6)                  ALLOW       Anywhere (v6)
8000 (v6)                  ALLOW       Anywhere (v6)

```

---

## 4. Portainer 접속

컨테이너가 실행 중인 경우 다음 URL을 통해 브라우저에서 Portainer에 접속할 수 있습니다:

```
https://<your-server-ip>:9443
```

첫 접속 시 관리자 계정을 생성하고 환경을 구성해야 합니다.

https://docs.portainer.io/start/install-ce/server/setup  참고 하세요.

## 5. Portainer 컨테이너 셸 접근 관련

Portainer 컨테이너에 접근해야 하는 경우, 기본적으로 셸이 포함되어 있지 않습니다. 문제 해결을 위해 다음 명령을 사용할 수 있습니다:

```bash
docker logs portainer

```

https://www.reddit.com/r/portainer/comments/1egpam7/how_to_bash_into_the_portainers_container/ 여기 보니 없다고 하는 것 같습니다. 

# 기타 내용

## 글쓴이의 실제 동작 화면

제가 실행 해서, potainer 만 docker cotainer로 동작 중인 화면입니다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b5940867-545a-450e-a034-d06b95eee697/9f973300-c7ba-4ff8-ba96-3024089954c2/image.png)

## sudo 제거

매번 sudo 명령이 불편 하여, 아래 명령을 수행하였습니다. 

```bash
sudo usermod -aG docker $USER
```
