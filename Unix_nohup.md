- 사용자가 터미널을 종료해서 프로세스는 여전히 동작하도록 하는 방법.
- No Hang up의 의미.

```
$ nohup run &
```

- nohup.out 파일이 생성되는데 이는 프로세스가 동작하며 발생하는 표준출력이 저장되는 파일. 즉, 로그이다.
- 이 로그를 다른 경로에 쓰기 위해서는?

```
$ nohup run > logs/log.log 2>&1 &
```

---

- 참고

> http://theeye.pe.kr/archives/695

> https://syuda.tistory.com/72

---

- 추가 학습 필요.
  - unix 표준 입출력 제어.
