[[>Computer network]]
`tc -V` : 버전 확인

만약 "**eth0**으로 들어오고 나가는 패킷의 50%를 드랍시키고 싶다"면 다음과 같이 명령어를 실행하면 됩니다!

```
sudo tc qdisc add dev eth0 root netem loss 50%
```

>debugging: `RTNETLINK answers: Operation not permitted`
>
	Docker containers do not have full privileges by default. Try adding this to the `docker run` command:
	`sudo docker run -it --cap-add=NET_ADMIN debian`

