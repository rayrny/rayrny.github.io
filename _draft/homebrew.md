m1에 homebrew를 설치해서 사용하려면 설치한 뒤에도 환경 변수를 설정해 주어야 한다.
아래의 명령어 입력을 통해 환경 변수를 설정해 줄 수 있으며, 이후에 `vi ~/.zshrc` 를 입력하면 `/opt/homebrew/bin/brew` 이 새롭게 생긴 것을 확인할 수 있다.

```bash
$ echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/<USER_ID>/.zprofile
$ eval "$(/opt/homebrew/bin/brew shellenv)"
```

근데 코드 커밋 후 터미널을 끈 뒤에 VS 코드를 다시 켜니...

<img width="514" alt="스크린샷 2021-12-13 오후 10 00 24" src="https://user-images.githubusercontent.com/48341341/147333027-8f3d48a0-da84-4d34-8bf8-988f4d0a0b34.png">

왜 못찾는거야..! 심지어 `vi ~/.zshrc` 입력하면 또 path는 잘 지정되어 있다..
이유는 못찾았지만 brew만 다시 설치하니 이전에 설치했던 패키지들을 다시 사용할 수 있게 되었다.

<img width="694" alt="스크린샷 2021-12-13 오후 10 01 59" src="https://user-images.githubusercontent.com/48341341/147333560-077e8dd3-58e6-4131-b710-4ea5fce59a15.png">



그리고 오늘은 서버를 도커로 켜봤다! 과연 결과는?!

<img width="975" alt="스크린샷 2021-12-13 오후 10 04 10" src="https://user-images.githubusercontent.com/48341341/147333576-7aaf69d6-077c-48e1-92f4-c5669852e2f0.png">

서버 키고 ```yarn start```를 호기롭게 입력했으나, 결과는.. ㅠ

<img width="975" alt="스크린샷 2021-12-13 오후 10 05 37" src="https://user-images.githubusercontent.com/48341341/147333626-0d82755d-1720-4053-903f-aea5c0c7e027.png">


다시 ```yarn install``` 부터 다시 시도하고, 리눅스니까 ```echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p``` 해주고 다시 시작하니 잘된다!!

<hr>

처음 설치할 때 뭔가를 잘못했다보다. 이후 부터는 위와 같은 현상이 발생하지 않고 ```yarn start``` 만 해도 바로 접속이 된다.
