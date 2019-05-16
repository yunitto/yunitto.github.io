---
title: Newline Character
date: "2019-05-16"
template: "post"
draft: false
slug: "/posts/newline/"
category: "UNIX"
tags:
- "UNIX"
- "newline"
description: Newline? EOL? Line Feed? Line Break?
---
## Newline, EOL(End-Of-Line), Line Feed, Line Break
Newline/EOL/Line Feed/Line Break은 모두 같은 말이다. line의 끝과 새 line의 시작을 나타내는데, OS마다 다른 캐릭터를 쓴다.

- `\n (ASCII 0x0A)` Line Feed(LF) - Unix and Unix-like systems(Linux, macOS, FreeBSD)
- `\r (ASCII 0x0D)` Carriage Return(CR)
- `\r\n (ASCII 0x0D0A)` CRLF - Microsoft Windows

이처럼 Newline 캐릭터가 여러 종류가 있으니 텍스트 데이터를 정제할 때 빼먹지 말고 꼼꼼히 처리해주어야한다.

## 라인과 파일 끝에 Newline character을 넣어야 하는 이유
라인과 파일 끝을 제대로 인식하기 위해서다. POSIX에서는 Line을 다음과 같이 [정의한다.](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html#tag_03_206)

```
A sequence of zero or more non- <newline> characters plus a terminating <newline> character.
```

git을 사용할 때 파일 끝에 newline을 추가해주지 않으면 git diff시 `\ No newline at end of file` 문구가 뜬다. 이걸 무시하고 커밋하면 git이 파일의 마지막 줄을 인식하지 못했기 때문에, 이후 같은 파일을 수정할 때 기존의 마지막 줄이 git diff에 포함된다. (꽤 성가시다!)

유닉스 계열에서 줄 수를 셀 때도 new line이 없으면 마지막 줄을 인식하지 못한다.
```
$ echo "hello" > text.txt
$ xxd text.txt
00000000: 6865 6c6c 6f0a                           hello
$ wc -l text.txt
       1 text.txt

# -n option: Do not output the trailing newline.
$ echo -n "hello" > text.txt 
$ xxd text.txt
00000000: 6865 6c6c 6f                             hello
$ wc -l text.txt
       0 text.txt
```