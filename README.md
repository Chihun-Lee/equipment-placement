# 실험실 장비 배치 시뮬레이터 (Lab Equipment Placement Simulator)

Three.js 기반의 웹 3D 실험실 장비 배치 시뮬레이터입니다. 브라우저에서 바로 실행되며, 장비 배치·공간 구성·충돌 감지·AI 3D 형상 생성 등을 지원합니다.

- **배포 URL**: https://chihun-lee.github.io/equipment-placement/
- **레포**: https://github.com/Chihun-Lee/equipment-placement
- **모바일 버전(IKEA 가로 레이아웃)**: https://chihun-lee.github.io/equipment-placement/m1.html
- **현재 버전**: v2.5
- **제작**: chihunlee@kims.re.kr

---

## 주요 기능

### 장비 라이브러리
- 40여 종의 기본 장비 프리셋 (SEM, 3D프린터, 전기로, 후드, 테이블, 캐비닛, 냉장고, 글로브박스, 인장시험기, XRD, EDM, PC, 모니터 등)
- 사용자 정의 장비 추가 (크기·색상·카테고리 자유 설정)

### 3D 형상 생성 우선순위
1. **GLB 3D 모델** — 사용자가 업로드한 `.glb` 파일
2. **프리미티브 조합** — 장비명이 내장 프리미티브(PRIMITIVES)와 매칭되면 자동 3D 렌더링
3. **사진 텍스처** — 사진 업로드 시 배경 제거 후 박스 면에 적용 (alphaTest)
4. **AI 3D 형상** — Anthropic Claude API를 사용해 JSON 파츠 조합으로 자동 3D 생성
5. **단색 박스** — 모든 방법이 실패했을 때의 폴백

### AI 장비 추가 위자드
- 장비 이름만 입력하면 Claude API가 세부 질문을 생성
- 질문 답변 → AI가 3D 파츠 구성(박스·실린더 조합) 생성
- 크기 미세 조정 후 라이브러리 등록 및 배치
- 설정된 bbox에 정확히 맞도록 정규화 (v2.5에서 강화)

### 공간 및 배치
- 다중 공간(방) 지원 — 인접 방 추가/삭제
- 방 크기·벽 색·바닥 색 커스터마이즈
- 문 (벽 스냅, 경첩 좌/우, 열림 방향, swing arc 시각화)
- 방호벽 (투명 배리어, 자유 배치)
- 사람 형상 · 로봇팔 (리치 반경 시각화)
- 테이블탑 장비 자동 스냅 (가장 가까운 surface 위에)

### 조작
- **드래그**: 마우스로 객체 이동 (바닥 위 XZ 평면)
- **방향키**: 선택된 객체 미세 이동 (Shift로 0.01m 단위)
- **R**: 90도 회전
- **L**: 객체 고정(잠금) 토글
- **Delete/Backspace**: 선택 객체 삭제
- **Ctrl+Z / Ctrl+Shift+Z**: 되돌리기 / 앞으로가기
- **Enter / Escape**: 선택 해제

### 기타 기능
- 줄자 (두 점 거리 측정, 장비 surface 위에서도 가능)
- 충돌 감지 (빨간색 겹침 영역 하이라이트)
- 저장/불러오기 (JSON 파일, localStorage 자동 저장)
- 스크린샷 (단일 + 4방향 동시 캡처)
- 카메라 프리셋 · 자유 카메라 모드
- 배경색/치수 표시 커스터마이즈
- 색상 팔레트 · 키보드 단축키

---

## 기술 스택

- **Three.js 0.160.0** — WebGL 3D 렌더링 (unpkg CDN, ES modules)
- **GLTFLoader** — GLB 3D 모델 로드
- **순수 HTML/JS** — 빌드 도구 · 서버 없음 (단일 `index.html`)
- **저장**: JSON 파일 + localStorage
- **AI**: Anthropic Claude API (옵션, 사용자 API 키 입력)

---

## 파일 구조

```
equiment_placement/
├── index.html       # 메인 애플리케이션 (데스크톱/일반용, v2.5)
├── m1.html          # 모바일 가로(IKEA 스타일) 레이아웃
├── manifest.json    # PWA 매니페스트
├── sw.js            # 서비스 워커 (오프라인 캐시)
├── icon-192.png     # PWA 아이콘 (192×192)
├── icon-512.png     # PWA 아이콘 (512×512)
├── CLAUDE.md        # Claude Code용 프로젝트 지침
└── README.md        # 이 파일
```

---

## 데이터 모델

```js
spaces[]   // 방/공간 목록 (id, name, x, z, w, d, h, floorColor, wallColor)
placed[]   // 배치된 객체 (type: equipment | door | barrier | person | robot)
LIBRARY[]  // 장비 라이브러리 (40+종)
PRIMITIVES // 프리미티브 빌더 (20종, 장비명 자동 매칭)
```

### 저장 포맷 (JSON v2)
```json
{
  "version": 2,
  "spaces": [...],
  "placed": [...],
  "nextId": N,
  "activeSpaceId": N,
  "nextSpaceId": N,
  "bgColor": "#hex"
}
```

- `version: 1` → `version: 2` 마이그레이션 지원 (기존 room 필드를 spaces[0]으로 변환)

### Y축 좌표 규칙
- **프리미티브/AI 3D** (`usePrimitive=true`): y=0 기준, 위로 h만큼 빌드
- **박스** (`usePrimitive=false`): y=h/2 (중심 기준)
- 드래그·업데이트·로드 시 이 구분을 반드시 유지

---

## 버전 히스토리

### v2.5 (현재)
- AI 3D 장비가 박스 크기에 정확히 맞게 보이도록 정규화 로직 개선 (2-pass 방식, geometry에 직접 baking)
- AI 프롬프트 개선: 메인 바디가 bbox의 95-100%를 채우도록 강제
- AI 파츠 캐시 버전 관리 (v2.5 업데이트 시 자동 무효화)

### v2.4
- 프리미티브/AI 형상 장비도 W/D/H 수정 시 새 크기로 완전 재빌드
- 편집 패널에 "크기 적용" 버튼 추가

### v2.3
- AI 3D 형상 크기 보정 (bbox 정규화 초기 도입)
- 저장/불러오기 시 `usePrimitive` 상태 유지
- 객체 고정(잠금) 기능 추가 (🔒 체크박스, L 키)

### v2.2
- UI 대폭 개선 + 9개 사용자 요청 반영
- 테이블탑 장비: 체크박스 토글, surface 안착 유지, 충돌 무시

### v2.1
- AI 장비 위자드 + 신규 장비 6종 + 충돌 개선
- 우측 패널 분리 + 방향키 이동 + 레이아웃 개선
- 선택 바운딩박스 하이라이트 + Ctrl+Z 되돌리기

### v2.0
- 다중 공간(방) 지원
- JSON 저장 포맷 v1 → v2 마이그레이션

---

## 배포

GitHub Pages 자동 배포:
```bash
git add index.html
git commit -m "설명"
git push
# 1~2분 후 https://chihun-lee.github.io/equipment-placement/ 반영
```

### 로컬 실행
```bash
# 단순 HTTP 서버 사용
python3 -m http.server 8000
# 또는
npx serve .
```
브라우저에서 `http://localhost:8000` 접속.

---

## 개발 메모

### Three.js 주의사항
- `Object.assign(mesh, {position: ...})` 사용 금지 → `.position.set()` 사용
- 프리미티브 빌더는 `pm()` 헬퍼 사용
- 프리미티브/person/robot은 투명 `pickBox`로 레이캐스트 피킹

### AI 3D 형상 생성 (v2.5 기준)
- 파츠는 0-1 범위 fraction으로 표현 (w/h/d의 비율)
- `buildFromAIParts()`가 2-pass로 처리:
  1. 임시 메쉬를 빌드해 실제 bbox 측정 (회전 포함 정확)
  2. 측정된 스케일로 geometry 크기를 직접 보정하여 재빌드 (scale 속성 미사용 → pickBox/label 간섭 없음)
- 원통(cylinder)은 XZ 평균 스케일(sqrt(sx·sd))로 원형 유지

### 알림 시스템 연동
배포/빌드 결과는 Telegram 알림으로 연동 가능:
```bash
~/Code/cluster-notify/notify.sh equiment_placement started "GitHub Pages 배포 시작"
~/Code/cluster-notify/notify.sh equiment_placement training_complete "배포 완료"
```

---

## 라이선스

내부 연구용 (KIMS). 문의: chihunlee@kims.re.kr
