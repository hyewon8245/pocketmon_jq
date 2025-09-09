# 귀여운 포켓몬의 뒷모습 웹페이지

---

## 👥 Team Members
<table>
  <tr>
    <th>이정이</th>
    <th>홍혜원</th>
  </tr>
  <tr>
    <td align="center">
      <img src="https://github.com/2jeong2.png" width="120" /><br/>
      <a href="https://github.com/2jeong2">@이정이</a>
    </td>
    <td align="center">
      <img src="https://github.com/hyewon8245.png" width="120" /><br/>
      <a href="https://github.com/hyewon8245">@홍혜원</a>
    </td>
  </tr>
</table>

---

## ✅ **프로젝트 개요**

- **pocketmon-api**를 활용
- **jq**를 활용해 json을 파싱하여 **포켓몬 gif**, **이름**확인
- **nginx**를 활용하여 포켓몬이름과 포켓몬 뒷모습 gif를 볼 **웹페이지 제작**
![전체내용](최종완성.gif)

---
## 📂 **프로젝트 디렉토리 구조**
```
var/
    www/
        html/
            pocketmon.html     # pocketmon.sh 실행 결과물
            backiscute.html    # 메인페이지
pocketmon/
    pocketmon.sh               # 랜덤 포켓몬 뽑아 HTML 생성하는 스크립트
```
---

## 🔍**프로젝트 과정**

### nginx 설치

```bash
sudo apt update
sudo apt install nginx
```
---

### 🧑‍💻 `/pocketmon/pocketmon.sh` 생성
**포켓몬 API(PokeAPI)** 를 활용하여,
배열에 정의된 포켓몬 중 무작위로 하나를 선택해 **뒷모습 GIF와 한국어 이름**을 가져와 웹페이지(`/var/www/html/pocketmon.html`)를 자동 생성

### pocketmon 디렉토리 생성

```bash
mkdir /pocketmon
```
### pocketmon.sh 생성과 실행 권한 설정

```bash
cd pocketmon
vi pocketmon.sh
chmod +x pocketmon.sh
```


### `/pocketmon/pocketmon.sh` 동작 설명
<details>
<summary>포켓몬 랜덤 HTML 생성 스크립트 보기</summary>
  
```bash
#!/bin/bash

# 의존성 체크 curl과 jq가 깔려있는지 확인
for bin in curl jq; do
  command -v "$bin" >/dev/null 2>&1 || { echo "ERROR: $bin not found"; exit 1; }
done

# 포켓몬 배열(좋아하는 포켓몬 20마리)
POKEMONS=("chansey" "meowth" "lucario" "shinx" "pikachu" "magikarp" "gyarados" "turtwig" "oshawott" "farfetchd" "Pichu" "Munchlax" "Pachirisu" "Metapod" "Slowpoke" "Exeggutor" "Dratini" "Mewtwo" "Mew" "Rowlet")


# 배열 길이
LEN=${#POKEMONS[@]}

# 랜덤 인덱스 선택 
IDX=$((RANDOM % LEN))
POKEMON=${POKEMONS[$IDX]}

# API 호출해서 해당 포켓몬의 gif URL 가져오기
# API는 포켓몬에 관련된 데이터를 저장한 내용으로 jq로 파싱하여 뒷모습 gif파일을 가져오도록 만듦
URL=$(curl -s https://pokeapi.co/api/v2/pokemon/$POKEMON \
  | jq -r '.sprites.versions."generation-v"."black-white".animated.back_default')
# species URL 이름을 추출할때 여러 언어들의 이름 정보들을 가진 api 추출
SPECIES_URL=$(curl -s https://pokeapi.co/api/v2/pokemon/$POKEMON | jq -r '.species.url')

# 여러 언어 이름 중 한국어 이름 추출
KOR_NAME=$(curl -s $SPECIES_URL | jq -r '.names[] | select(.language.name=="ko") | .name')


# HTML 파일 생성
OUTFILE="/var/www/html/pocketmon.html"

cat <<EOF > $OUTFILE
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>$POKEMON Sprite</title>
    <style>
      body { font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif; margin: 40px; }
    .card { max-width: 560px; border: 1px solid #ddd; border-radius: 12px; padding: 24px; box-shadow: 0 4px 12px rgba(0,0,0,.06); }
    h1 { margin: 0 0 12px; font-size: 28px; }
    .sub { color: #666; margin-bottom: 20px; }
    .imgbox { display: flex; align-items: center; justify-content: center; min-height: 200px; background: #fafafa; border-radius: 10px; }
    .note { margin-top: 16px; color: #999; font-size: 14px; }
  </style>
  </head>
<body>
  <div class="card">
    <h1>$KOR_NAME</h1>
    <div class="sub">뒷모습 귀엽지
    <div class="imgbox">
     <img src="$URL" alt="$POKEMON">
    </div>
    <div class="note">데이터 출처: <a href="$URL" target="_blank" rel="noreferrer">PokeAPI</a></div>
  </div>
</body>
</html>
EOF

echo "웹페이지가 $OUTFILE 에 생성되었습니다."
echo "http://<서버IP>/pocketmon.html 로 접속하세요."

```
</details>

<details>
<summary>주요 기능 정리</summary>

### 1. 의존성 체크

```bash
for bin in curl jq; do
  command -v "$bin" >/dev/null 2>&1 || { echo "ERROR: $bin not found"; exit 1; }
done
```

* `curl`, `jq` 명령어가 설치되어 있는지 확인.
* 없으면 에러 메시지 출력 후 종료.

---

### 2. 포켓몬 배열 준비

```bash
POKEMONS=("chansey" "meowth" "lucario" ... "Rowlet")
```

* 좋아하는 포켓몬 20마리 목록 정의.
* `RANDOM` 값을 이용해 무작위 포켓몬을 선택.

---

### 3. API 호출

```bash
URL=$(curl -s https://pokeapi.co/api/v2/pokemon/$POKEMON \
  | jq -r '.sprites.versions."generation-v"."black-white".animated.back_default')
```

* `pokeapi.co`에서 선택된 포켓몬의 **뒤모습 애니메이션 GIF URL**을 가져옴.

```bash
SPECIES_URL=$(curl -s https://pokeapi.co/api/v2/pokemon/$POKEMON | jq -r '.species.url')
KOR_NAME=$(curl -s $SPECIES_URL | jq -r '.names[] | select(.language.name=="ko") | .name')
```

* Species API를 추가로 호출해 **여러 언어 이름 중 한국어 이름**을 추출.

---

### 4. HTML 파일 생성

```bash
OUTFILE="/var/www/html/pocketmon.html"
cat <<EOF > $OUTFILE
... (HTML 구조) ...
EOF
```

* HTML 카드 형태의 페이지를 생성.
* `<h1>` 태그에는 포켓몬의 **한글 이름** 출력.
* `<img>` 태그에는 포켓몬 **뒤모습 GIF** 삽입.
* 하단에 데이터 출처(PokeAPI) 표기.

---
</details>

---

### pocketmon.sh 실행 결과

* 실행 후 `/var/www/html/pocketmon.html` 파일이 생성됨.
* 브라우저에서 `http://<서버IP>/pocketmon.html` 접속 시 결과 확인 가능.
* 실행할 때마다 랜덤 포켓몬이 나오므로, 페이지를 다시 생성하면 새로운 포켓몬 등장.

--- 

### pocketmon.sh 실행 예시

```bash
cd /pocketmon
sh pocketmon.sh
```

출력:

```
웹페이지가 /var/www/html/pocketmon.html 에 생성되었습니다.
http://<서버IP>/pocketmon.html 로 접속하세요.
```
브라우저에서 접속 → 랜덤 포켓몬의 한국어 이름과 모습 GIF 확인 가능 🎉

![포켓몬 뒷모습](파오리_고화질2.gif)
---

## Nginx 설정 변경
pocketmon.html같이 뜨지 않고 /pocketmon 이라는 웹페이지 경로로 뜨게 하기 위함


### Nginx 경로 마운트(alias) 설정


`/etc/nginx/sites-available/default` 파일에 아래 블록을 추가 

```bash
location /pokemon/ {
	alias /var/www/html/;
	index pocketmon.html;
	try_files성



- **메인 페이지와 연결**
    
    메인페이지에서 사진 클릭 시 [http://localhost:81/pokemon/](http://localhost:81/pokemon/)로 이동
    
    메인 페이지 html 코드 [/backiscute.html]
    
    ```bash
    <!DOCTYPE html>
    <html lang="ko">
    <head>
      <meta charset="UTF-8">
      <title>귀여운 포켓몬 뒷모습 모음</title>
      <style>
        body {
          font-family: "Comic Sans MS", "Arial", sans-serif;
          background-color: #fffafc;
          margin: 0;
          padding: 0;
          text-align: center;
        }
    
        header {
          background-color: #ffb6c1;
          padding: 20px;
          box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
    
        header h1 {
          margin: 0;
          color: #fff;
          font-size: 2.5rem;
          text-shadow: 1px 1px 2px #ff69b4;
        }
    
        .gallery {
          display: grid;
          grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
          gap: 20px;
          padding: 30px;
          max-width: 1000px;
          margin: auto;
        }
    
        .gallery a {
          display: block;
          background-color: #ffe4f0;
          border-radius: 20px;
          padding: 15px;
          transition: transform 0.2s ease, box-shadow 0.2s ease;
          text-decoration: none;
        }
    
        .gallery a:hover {
          transform: scale(1.05);
          box-shadow: 0 4px 10px rgba(0,0,0,0.15);
        }
    
        .gallery img {
          width: 100%;
          height: auto;
          border-radius: 15px;
        }
    
        footer {
          margin-top: 30px;
          padding: 15px;
          background-color: #ffb6c1;
          color: white;
          font-size: 0.9rem;
        }
      </style>
    </head>
    <body>
      <header>
        <h1>✨ 귀여운 포켓몬 뒷모습 모음 ✨</h1>
      </header>
    
      <section class="gallery">
        <!-- 이미지와 연결할 HTML 경로만 바꾸면 됩니다 -->
        <a href="http://localhost:81/pokemon/">
          <img src="./IMG_0622.jpg" alt="오랑오랑과 플러피 뒷모습">
        </a>
        
        <!-- 필요한 만큼 추가 -->
      </section>
    
      <footer>
        만든이: 이정이 , 홍혜원🐾
      </footer>
    </body>
    </html>
    ```
    
    ## 메인페이지 엔진엑스에 마운트
    
    [/etc/nginx/sites-available/default]
    
    ```bash
    location /backiscute/ {
            alias /var/www/html/;
            index backiscute.html;
            try_files $uri $uri/ =404;
    }
    ```

    

