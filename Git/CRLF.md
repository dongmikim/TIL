맥 윈도우 협업시 git 에서 CRLF 개행 문자 차이로 인한 문제 해결방법

- window는 기본이 CRLF (Carriage-Return, \r) (LineFeed, \n)
- mac은 LF

문제 상황: Mac, window 팀원 줄바꿈 형식이 vscode에서 다르게 표시되어 오해 발생 

원인
- window에서 vscode에서는 CRLF로 표시되나, github 올릴 때 LF로 변환되어 올라감.
  - 아마 `core.eol = lf` 이 설정이 추가된게 아닐까
  - git config 정보는 `git config --list` 명령어로 확인 가능
  - 기존에 잇는 글로벌 설정 변경: `git config --unset --global core.eol`
- vscode 표시때문에 헷갈림

