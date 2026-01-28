# docker-study

### Docker 관련 유용한 정보


* Container 빠져나올 때는, Ctrl + P + Q
* Docker Official Image 이용 시엔 되도록 alpine-(slim) 이미지 권장 (경량)
* Image 처음 pull 받으면, 다음 명령 실행해서 Expose Port 를 확인

```
docker image history [image name]
```