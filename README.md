# Docker Study

### Docker 관련 유용한 정보


* Container 빠져나올 때는, Ctrl + P + Q
* Docker Official Image 이용 시엔 되도록 alpine-(slim) 이미지 권장 (경량)
* Image 처음 pull 받으면, 다음 명령 실행해서 Expose Port 를 확인
```
docker image history [image name]
```

* Image .tar 파일로 생성 (여러 Layer로 되어 있기에 tar 로 압축)
```
docker save [image name] > [image name].tar
```

* Image .tar 파일을 이미지로 Load
```
docker load < [image name].tar
```

* Docker Private Registry 사용
```
# registry 이미지 Pull
docker pull registry

# 컨테이너 실행
docker run -d \
-p 8000:5000 \
--restart=always \
--name local-registry \
registry

# docker registry 주소 등록
sudo vi /etc/init.d/docker
DOCKER_OPTS="$DOCKER_OPTS --insecure-registry localhost:8000"

sudo vi /etc/docker/daemon.json
{
  "insecure-registries" : ["localhost:8000"]
}

# docker restart
sudo systemctl restart docker

# registry 에 이미지 Push
docker tag [image name] localhost:8000/[image name]
docker push localhost:8000/[image name]

# registry 에서 이미지 Pull
docker pull localhost:8000/[image name]

# registry repository 확인
curl -X GET http://localhost:8000/v2/_catalog

# registry image tag 확인
curl -X GET http://localhost:8000/v2/[image name]/tags/list
```

* Docker Container Life Cycle
```
# docker run (create, start)
1. docker create -> image의 snapshot을 /var/lib/docker 영역에 생성
2. docker start -> 읽고 쓰기가 가능한 Process 영역 container layer 를 생성하여 동적 컨테이너를 구성
3. docker stop -> container layer 삭제
4. docker rm -> image snapshot 삭제
```

* Docker Log 관리
```
/var/lib/docker/containers/[container id]/[container id]-json.log 로 쌓임

vi /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "30m",
    "max-file": "5"
  }
}
```

* Container 수정 후 Commit
```
Container 변경 사항 commit 하여 이미지 새로 생성

docker commit [container id] [image name]
```

* Docker Network
```
1. Docker가 자동으로 docker0 라는 기본 브릿지 네트워크를 생성
2. 각 Container 는 내부적으로 eth0(이더넷, Mac Address, IP 생성) 가 생성되고, 이는 veth(virtual ethernet) 으로 docker0 와 연결됨
-> 그래서 같은 docker0 에 붙어있기에 컨테이너끼리 서로 IP 통신 가능
-> 단, 컨테이너는 “IP로” 상대를 지정하지만, docker0(브릿지)는 그 패킷을 “MAC만 보고” 스위칭 함
3. 외부로의 네트워크 통신은 Host 가 라우팅
4. 서로 다른 Host 에서 동작하는 컨테이너 간 통신은 Overay Network 이용
```

* 사용자 정의 네트워크
```
# 사용자 정의 네트워크 생성
docker network create \
    --driver bridge \
    --subnet 172.16.31.0/24 \
    -- ip-range 172.16.31.100/26 \
    --gateway 172.16.31.1 \
    [network name]

# 컨테이너 실행 시 사용자 정의 네트워크 사용
docker run -d --network [network name] [image name]
```

* 서로 다른 네트워크 연결 (bridge(gateway) 가 다름)
```
docker network connect [network name] [container id]
```

* Docker Network DNS (컨테이너 명, ip 외에 다른 도메인으로 접근)
```
사용자 정의 네트워크에서만 가능 (docker0 사용 X docker DNS 가 없기에)

docker run --network [network name] --network-alias [dns name] [image name]
```

* Container CPU 제어
```
Docker 는 host 와 커널을 공유하기에(cgroup 사용) lscpu 등으로 확인했을 때, core 수가 변하거나 하진 않음

-- cpu-shares : 컨테이너가 사용할 수 있는 CPU 사용 시간에 대한 가중치를 상대적으로 설정
-- cpuset-cpus : 사용 가능한 CPU 코어를 지정
-- cpus : 컨테이너의 CPU 상한 설정

docker run -d --cpu-shares 512 --cpuset-cpus 0,1 [image name]
```

* Container Memory 제어
```
Docker 는 host 와 커널을 공유하기에(cgroup 사용) free 등으로 확인했을 때, 메모리 양이 변하거나 하진 않음
--memory : 컨테이너가 사용하는 최대 메모리 사용량 제한
--memory-reservation : (soft limit) HostOS 의 메모리 가용율이 떨어지는 경우 활성화 되어 최소한의 메모리 사용 보장
--memory-swap : 컨테이너가 사용할 수 있는 swap memory 사용량 제한

docker run -d --memory 100m --memory-reservation 50m [image name]
```

* Container Disk 제어
```
--blkio-weight, --blkio-weight-device : Block I/O 할당량 제한
-- device-read-bps, --device-write-bps : 초당 Block throughput 을 제한
-- device-read-iops, --device-write-iops : 초당 Block I/O 횟수를 제한

docker run -d --blkio-weight 100 \
    --device-read-bps /dev/sda:100k \ 
    --device-write-bps /dev/sda:100k \ 
    --device-read-iops /dev/sda:1000 \ 
    --device-write-iops /dev/sda:1000 [image name]
```

* Container Volume 설정
```
1. volume 생성 후 사용 (/var/lib/docker 에 저장)
docker volume create [volume name]
docker run -d -v [volume name]:/data [image name]

2. bind volume
docker run -d -v [host path]:[container path]:(ro|rw) [image name]

3. tmpfs volume 
docker run -d --tmpfs /tmp [image name]
```

* Host 시간 설정과 Container 시간 설정 동기화
```
docker run --rm -v /etc/localtime:/etc/localtime:ro [image name]

-> Dockerfile 에서 timezone 추가하는 걸 권장

FROM ubuntu:18.04
RUN apt-get update && apt-get install -y tzdata
ENV TZ=Asia/Seoul
```

* Docker Container Layer(Overlay Storage) 사용량 제한

```
우선, Storage 사용량 제한을 하려면 파일 시스템이 xfs 로 지정되어야 하고(디렉토리 별 디스크 제한하기 위해), 
추가 기능으로 pquota(project quota) 가 설정 되어야 함

docker run -d --storage-opt size=1G [image name]

vi /etc/default/grub
10 GRUB_CMDLINE_LINUX="quiet splash rootflags=pquota"

vi /etc/fstab
UUID=... /var/lib/docker xfs defaults,pquota 0 0

# overlay Storage 전체 제한

vi /etc/docker/daemon.json
{
  "storage-opts": [
    "overlay2.size=10G"
  ]
}

# Volume 을 제한하려면, Host 에서 스토리지 자체를 제한된 크기로 만들어서 마운트해야 함
```

* Dockerfile 경량화

```
1. 컨테이너 이미지에서 불필요한 바이너리를 모두 제거하여 이미지 크기를 경량화 한다.
--no-install-recommends <package>, .dockerignore 사용 등도 사용

apt clean autoclean && \
apt autoremove -y && \
rm -rfv /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

```
2. Docker 에서 제공하는 최소 기본 이미지인 alpine 혹은 slim, scratch(Go Lang 등) 을 사용한다.
```

```
3. multi-stage build 를 사용하여 최종 이미지의 크기를 최소화한다.
첫번째 stage(빌드도구)에서 생성된 생성파일을 두번째 stage(배포이미지)에 제공

FROM builder_image AS builder
# Build process

FROM scratch
# Final image
COPY --from=builder /app/main /
```

```
4. Layer 수를 최소화
FROM, LABEL, RUN, COPY, ADD, ENV 등의 명령은 Docker build 시 Layer 를 만들기 때문에 '\' 으로 명령어 한 번으로 처리하는 것을 권장

ex) 
잘못된 예
LABEL purpose="test"
LABEL version="1.0"
LABEL description="test"

올바른 예
LABEL purpose="test" \
      version="1.0" \
      description="test"
```

```
5. 보안을 위해 root 사용자가 아닌 일반 사용자 생성 후 사용
RUN useradd -ms /bin/bash [user name]
USER [user name]
```

