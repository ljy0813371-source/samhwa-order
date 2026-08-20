# 자료 자동 취합기 - GitHub + Vercel 배포판

로컬 실행판(`merge_app.zip`)과 달리, 이 버전은 **깃허브에 push하면 자동으로 웹사이트에 반영**되도록 만든 서버리스(Vercel) 배포용입니다.

## 로컬판과의 차이점 (중요)

| | 로컬판 (merge_app.zip) | Vercel판 (이 폴더) |
|---|---|---|
| 실행 방식 | 팀 PC에서 `python app.py` | 깃허브 push → Vercel 자동 배포 |
| 접속 | 사내망 IP로만 | 인터넷 어디서나 URL로 |
| 데이터 위치 | 회사 내부에만 존재 | **Vercel(미국 등 외부 클라우드) 서버를 거쳐감** |
| 다운로드 방식 | 병합 → 목록에서 다운로드 클릭 | 병합 버튼 누르면 **바로 다운로드** (서버리스라 파일을 서버에 저장해두지 않음) |
| 업로드 용량 제한 | 사실상 제한 없음 | **요청당 약 4.5MB** (Vercel Hobby 플랜 기준) |

**⚠️ 다시 한번: 재고/생산 데이터처럼 사외비 수준의 자료라면, 외부 클라우드를 거치는 이 방식이 회사 보안 정책에 맞는지 먼저 확인하시는 게 안전합니다.**

## 배포 순서

### 1. 깃허브에 코드 올리기

```bash
cd merge_app_vercel
git init
git add .
git commit -m "자료 취합기 초기 버전"
```

GitHub에서 새 저장소(비공개 추천)를 만든 뒤:

```bash
git remote add origin https://github.com/<계정명>/<저장소명>.git
git branch -M main
git push -u origin main
```

### 2. Vercel과 연동

1. https://vercel.com 접속 → GitHub 계정으로 로그인
2. "Add New... → Project" 클릭
3. 방금 만든 저장소 선택 → Import
4. Framework Preset은 자동으로 "Other"로 잡힐 텐데, 그대로 두고 **Deploy** 클릭
5. 1~2분 후 `https://<프로젝트명>.vercel.app` 주소가 발급됩니다

### 3. 이후 업데이트 방법

코드를 수정하고 나서:
```bash
git add .
git commit -m "수정 내용"
git push
```
push만 하면 Vercel이 알아서 재배포합니다. 별도 명령 필요 없습니다.

## 로컬에서 미리 테스트하는 법

배포 전에 내 PC에서 먼저 확인하고 싶다면:

```bash
cd merge_app_vercel
pip install -r requirements.txt
python -c "from api.index import app; app.run(port=5050)"
```
브라우저에서 `http://127.0.0.1:5050` 접속.

또는 Vercel CLI를 쓰면 실제 서버리스 환경과 거의 동일하게 로컬에서 테스트 가능합니다:
```bash
npm i -g vercel
vercel dev
```

## 파일 구성

```
merge_app_vercel/
├── vercel.json          # Vercel 빌드/라우팅 설정
├── requirements.txt      # 파이썬 패키지 목록
└── api/
    ├── index.py           # Flask 앱 전체 (화면 + 병합 로직 한 파일에 포함)
    ├── merge_excel.py
    ├── merge_word.py
    └── merge_ppt.py
```

로컬판과 달리 화면(HTML/CSS/JS)이 `index.py` 안에 문자열로 들어있습니다. 서버리스 환경에서 정적 파일 경로 문제를 피하기 위한 것으로, 기능상 차이는 없습니다.

## 만약 업로드 용량 제한(4.5MB)에 자주 걸린다면

- Vercel Pro 플랜(유료)으로 올리면 제한이 완화됩니다
- 또는 서버리스가 아닌 **상시 실행 방식** 호스팅(Render, Railway 등)으로 바꾸면 이 제한 자체가 없어집니다. 이 경우 원래 로컬판(`app.py` 방식, 2단계 업로드/다운로드 구조)을 그대로 배포하면 됩니다 — 필요하시면 그 배포판도 만들어드릴 수 있습니다.
