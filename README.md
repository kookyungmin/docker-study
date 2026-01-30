# docker-study

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
Docker 는 host 와 커널을 공유하기에(cgroup 사용) 디스크 양을 제한하거나 하진 못함
--blkio-weight, --blkio-weight-device : Block I/O 할당량 제한
-- device-read-bps, --device-write-bps : 초당 Block throughput 을 제한
-- device-read-iops, --device-write-iops : 초당 Block I/O 횟수를 제한

docker run -d --blkio-weight 100 \
    --device-read-bps /dev/sda:100k \ 
    --device-write-bps /dev/sda:100k \ 
    --device-read-iops /dev/sda:1000 \ 
    --device-write-iops /dev/sda:1000 [image name]
```



