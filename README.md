# 이번달도 돈없다냥

소비 반성문을 60초 동안 소리 내어 읽는 웹 서비스.
프롬프터처럼 텍스트가 아래에서 위로 올라오고, 다 읽으면 그 사람을 판결하고, 내 다짐을 영수증에 남긴다.

정적 사이트라서 빌드 과정이 없다. `index.html` 하나가 전부다.

## 배포

GitHub에 올리고 Vercel에 연결하면 끝난다. 프레임워크는 Other, 빌드 명령과 출력 디렉터리는 비워두면 된다.

```
git init
git add .
git commit -m "first"
git branch -M main
git remote add origin https://github.com/<계정>/<저장소>.git
git push -u origin main
```

로컬에서 볼 때는 파일을 직접 열지 말고 서버로 띄운다. `file://`로 열면 마이크 권한이 안 열린다.

```
python3 -m http.server 8000
```

마이크는 `https` 또는 `localhost`에서만 동작한다. Vercel은 https라 그대로 된다.

## 콘텐츠 고치기

전부 `index.html` 안의 `<script>` 블록에 있다.

### 낭독 속도

분당 글자수. 이 숫자만 바꾸면 전체 체감 속도가 바뀐다.
직접 소리 내어 읽어보고 벅차거나 심심하면 여기를 조정한다.

```js
const CATS = {
  regret:{ ..., cpm:{live:310, practice:265}},
  money: { ..., cpm:{live:380, practice:325}},
  trend: { ..., cpm:{live:345, practice:295}}
};
```

`live`가 본방송(기본값), `practice`가 연습. 콘텐츠 길이는 글자수에서 자동으로 계산된다.

### 원고 추가

`CONTENT` 배열에 넣는다. `key:true`인 줄은 읽을 때 빨간색으로 강조되고, 경제·트렌드는 `keep` 문장이 완료 화면에 다시 뜬다.

```js
{
  id:'r-example',        // 고유값
  cat:'regret',          // regret | money | trend
  amt:'− 120,000원',     // 카드에 뜨는 금액
  word:'텅장',            // 배경 손글씨
  title:'제목',
  mission:'읽는 방법 안내',
  lines:[
    '보통 줄',
    {t:'강조할 줄', key:true}
  ]
}
```

`cat`이 `regret`이면 읽은 뒤 판결과 다짐 화면이 뜨고, `money`나 `trend`면 바로 완료 화면으로 간다.

### 후기와 다짐

지금은 비어 있다. 실제로 받은 것만 넣는다.

```js
const SEED_REVIEWS = {
  'r-omakase': [{text:'읽다가 내 카드값이 떠올랐다'}]
};
const SEED_VOWS = ['이번 달엔 배달 다섯 번만'];
```

## 카피 A/B 테스트

링크 뒤에 `?c=` 를 붙이면 유입 경로가 로그에 남는다. 완료 이벤트에도 같은 값이 붙어서 카피별 완독률을 볼 수 있다.

```
https://<주소>/?c=a   도전형
https://<주소>/?c=b   각성형
https://<주소>/?c=c   상황형
```

## 데이터

현재는 브라우저 메모리에만 쌓이고 새로고침하면 사라진다. 홈 맨 아래 "기록된 이벤트 보기"로 확인할 수 있다.

쌓이는 이벤트: `app_open` `filter_change` `content_open` `read_start` `pause` `resume` `tempo_downshift` `read_abandon` `read_complete` `verdict` `vow_submit` `review_submit` `mic_granted` `mic_denied`

서버로 보내려면 `logEvent` 안의 주석 처리된 부분을 살리고 엔드포인트를 붙이면 된다.

```js
function logEvent(name, data){
  const e = {t:new Date().toISOString(), name, ...data};
  LOG.push(e);
  // fetch('/api/log', {method:'POST', body:JSON.stringify(e)});
}
```

## 아직 안 된 것

- 다짐과 후기가 저장되지 않는다 (DB 필요)
- 일주일 뒤 "그때 그 다짐 지켰어요?" 알림
- 사용자가 직접 반성문을 등록하는 기능
- 등록 글 자동 검수

## 외부 의존

CDN 세 개만 쓴다. 끊겨도 시스템 폰트로 떨어지고 앱은 돌아간다.

- Pretendard (본문)
- 배민 주아 (제목)
- 감자꽃 (손글씨)
