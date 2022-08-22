원격 데스크탑 검색후 크롬에서 직접다운받기 (명령어 x)


1. /usr/share/xsessions 에서 각자의 ubuntu.desktop 파일을 연다.(그냥 안열리면 text editor로 열면됨)

.desktop 파일에서 Exec= 행의 

gnome-session으로 시작하는 부분부터 끝까지를 복사한다.

예시

```python
Exec=env GNOME_SHELL_SESSION_MODE=ubuntu gnome-session --session=ubuntu
```

```jsx
Exec=env GNOME_SHELL_SESSION_MODE=ubuntu /usr/bin/gnome-session --systemd --session=ubuntu
```

1. chrome-remote-desktop-session을 home파일에 만든다.(.파일은 그냥보면 안보인다. 숨김 파일을 보기 show hidden file을 체크해야 보인다.)

```python
 touch .chrome-remote-desktop-session
```

1. .chrome-remote-desktop-session 파일에

exec /etc/X11/Xsession + '아까 복사한부분' 을 넣어주고 저장한다.

**('까지 넣어야함')**

```python
# 예시
Exec=env 'GNOME_SHELL_SESSION_MODE=ubuntu /usr/bin/gnome-session --systemd --session=ubuntu'
```

이러면 리눅스 컴퓨터에 다른 컴퓨터로 원격으로 들어갈수있어진다.하지만 화면이 뜨지않는 오류에 걸리게 된다. 추가적인 작업이 필요하다.

## 추가작업

1. 터미널 켜서 계정을 추가한다.

```python
sudo usermod -a -G chrome-remote-desktop 계정이름
```

안되면 스킵가능

1. 실행중인 크롬 원격 데스크톱(chrome remote desktop)을 중지한다.

```python
/opt/google/chrome-remote-desktop/chrome-remote-desktop --stop
```

1. 기존의 chrome remote desktop 설정 파일을 백업해 놓는다. 혹시 잘못되면 백업한 파일을 사용하면 된다.

```python
sudo cp /opt/google/chrome-remote-desktop/chrome-remote-desktop /opt/google/chrome-remote-desktop/chrome-remote-desktop.orig
```

1. 아래 명령어로 opt/google/chrome-remote-deskptop에 있는 파일을 열어준다.(그냥 열면 안되고 sudo로 열어야 수정가능함)

```python
sudo gedit /opt/google/chrome-remote-desktop/chrome-remote-desktop
```

1. 열어준 파일의 해당부분들을 수정한다.

현재 디스플레이 숫자를 FRIST_X_DISPLAY_NUMBER에 넣어준다. 

```jsx
echo $DISPLAY
```

```python
FIRST_X_DISPLAY_NUMBER = 2
```

다음 코드를 찾아 주석 처리 한다. 주석 처리는 문장 앞에 #을 추가하면 된다.

```python
#  while os.path.exists(X_LOCK_FILE_TEMPLATE % display):
#  display += 1
```

1. launch_session 함수를 찾아 수정한다. _launch_x_server()와 _launch_x_session()을 주석처리 해서 새로운 display가 생성되지 않게 한다. 그리고 두 줄의 코드를 추가하여 기존의 디스플레이를 사용한다. 이제 파일을 저장하고 편집 툴을 종료한다. (ubuntu 20.04면 밑에 빨간거 참고)

```python
def launch_session(self, x_args):
    self._init_child_env()
    self._setup_pulseaudio()
    self._setup_gnubby()
    #self._launch_x_server(x_args)
    #self._launch_x_session()
    display = self.get_unused_display_number()
    self.child_env["DISPLAY"] = ":%d" % display
```

**주의: 아래의 두 코드는 다른 함수에게 있다.**

laucn_session(self, x_args)에 복사해 붙여놓고

**다른함수에 있는 건 지워주면 된다.** 

```python
display = self.get_unused_display_number()
self.child_env["DISPLAY"] = ":%d" % display
```

## !!!!!!!!!Ubuntu 20.04 ver!!!!!!!!!!!!!!

```python
def launch_session(self, server_args, backoff_time):
    """Launches process required for session and records the backoff time
    for inhibitors so that process restarts are not attempted again until
    that time has passed."""
    logging.info("Setting up and launching session")
    self._init_child_env()
    self.setup_audio()
    self._setup_gnubby()
    #self._launch_server(server_args)
    #if not self._launch_pre_session():
      # If there was no pre-session script, launch the session immediately.
      #self.launch_desktop_session()
    display = self.get_unused_display_number()
    self.child_env["DISPLAY"] = ":%d" % display
    self.server_inhibitor.record_started(MINIMUM_PROCESS_LIFETIME,
                                      backoff_time)
    self.session_inhibitor.record_started(MINIMUM_PROCESS_LIFETIME,
                                     backoff_time)
```

1. chrome-remote-desktop을 다시 실행시킨다.

```python
/opt/google/chrome-remote-desktop/chrome-remote-desktop --start
```

그럼 잘되는걸 확인할 수 있다.