# 프로젝트 개요
- **pocketmon-api**를 활용
- **jq**를 활용해 json을 파싱하여 **포켓몬 gif**, **이름**을 불러와서 **웹페이지 제작**

---

# `/pocketmon/pocketmon.sh` 동작 개요

이 스크립트는 **포켓몬 API(PokeAPI)** 를 활용하여,
배열에 정의된 포켓몬 중 무작위로 하나를 선택해 **뒤모습 GIF와 한국어 이름**을 가져와 웹페이지(`/var/www/html/pocketmon.html`)를 자동 생성하는 기능
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
SP뒷모습 GIF 확인 가능 🎉
![포켓몬 뒷모습](파오리_고화질2.gif)

---
