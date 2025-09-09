# pocketmon_jq
- pocketmon-api를 활용하여 jq를 활용해 json을 파싱하여 포켓몬 gif, 이름을 불러와서 웹페이지 제작

---

# `/pocketmon/pocketmon.sh` 동작 개요

이 스크립트는 **포켓몬 API(PokeAPI)** 를 활용하여,
배열에 정의된 포켓몬 중 무작위로 하나를 선택해 **뒤모습 GIF와 한국어 이름**을 가져와 웹페이지(`/var/www/html/pocketmon.html`)를 자동 생성하는 기능

---

## 주요 기능 정리

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

* `pokeapi.co`에서 선택된 포켓몬의 **뒷모습 애니메이션 GIF URL**을 가져옴.

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

### 5. 실행 결과

* 실행 후 `/var/www/html/pocketmon.html` 파일이 생성됨.
* 브라우저에서 `http://<서버IP>/pocketmon.html` 접속 시 결과 확인 가능.
* 실행할 때마다 랜덤 포켓몬이 나오므로, 페이지를 다시 생성하면 새로운 포켓몬 등장.

---

## 사용 예시

```bash
cd /pocketmon
sh pocketmon.sh
```

출력:

```
웹페이지가 /var/www/html/pocketmon.html 에 생성되었습니다.
http://<서버IP>/pocketmon.html 로 접속하세요.
```

브라우저에서 접속 → 랜덤 포켓몬의 한국어 이름과 뒤모습 GIF 확인 가능 🎉


---

👉 제가 보기엔 학습용/재미용 스크립트로 딱 좋은데요! 원하시면 제가 **좀 더 확장해서** 랜덤 대신 **버튼 클릭 시 새 포켓몬 출력되게 하는 웹페이지 버전**도 정리해드릴까요?
