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


