# CLAUDE.md — 실험실 장비 배치 시뮬레이터

## 프로젝트 개요
Three.js 기반 3D 실험실 장비 배치 시뮬레이터. 단일 `index.html` 파일로 구성. GitHub Pages로 배포.

- **배포 URL**: https://chihun-lee.github.io/equipment-placement/
- **레포**: https://github.com/Chihun-Lee/equipment-placement
- **현재 버전**: v2 (SAVE_VERSION = 2)
- **제작자**: chihunlee@kims.re.kr

## 기술 스택
- Three.js 0.160.0 (unpkg CDN, ES modules)
- GLTFLoader (GLB 3D 모델 로드)
- 순수 HTML/JS — 빌드 도구 없음, 서버 사이드 없음
- 저장: JSON 파일 + localStorage

## 핵심 구조

### 데이터 모델
```
spaces[] — 방/공간 목록 (id, name, x, z, w, d, h, floorColor, wallColor)
placed[] — 배치된 객체 (type: equipment|door|barrier|person|robot)
LIBRARY[] — 장비 라이브러리 (40종)
PRIMITIVES{} — 프리미티브 빌더 (20종, 장비명 자동 매칭)
```

### 장비 3D 표현 우선순위
1. GLB 3D 모델 (사용자 업로드)
2. 프리미티브 조합 (장비명 자동 매칭)
3. 사진 텍스처 (배경제거 + alphaTest 박스)
4. 단색 박스 (폴백)

### Y축 좌표 규칙 (중요!)
- **프리미티브** (`usePrimitive=true`): y=0 기준, 위로 h만큼 빌드
- **박스** (`usePrimitive=false`): y=h/2 (중심 기준)
- 드래그, 업데이트, 로드 시 이 구분을 반드시 유지

### Three.js 주의사항
- `Object.assign(mesh, {position: ...})` 사용 금지 → `.position.set()` 사용
- 프리미티브 빌더는 `pm()` 헬퍼 사용
- 프리미티브/person/robot은 투명 `pickBox`로 레이캐스트 피킹

## 저장 포맷 (JSON v2)
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
- v1 → v2 마이그레이션: `migrateData()` 함수에서 room → spaces[0] 변환
- 향후 v3: `if (data.version === 2) { ... data.version = 3; }` 패턴

## 실행 환경

- **Anaconda 사용 금지** — miniforge(conda-forge)만 사용
- 프론트엔드 프로젝트라 클러스터 불필요, 맥북에서 직접 실행

## 알림 시스템 (cluster-notify)

배포/빌드 결과 Telegram 알림:
```bash
~/Code/cluster-notify/notify.sh equiment_placement started "GitHub Pages 배포 시작"
~/Code/cluster-notify/notify.sh equiment_placement training_complete "배포 완료"
~/Code/cluster-notify/notify.sh equiment_placement error "배포 실패"
```

## 배포
```bash
# 수정 후 배포
git add index.html
git commit -m "설명"
git push
# 1~2분 후 GitHub Pages에 자동 반영
```

## 기능 목록 (v2)
- 장비 라이브러리 40종 + 커스텀 추가
- 프리미티브 자동 3D (SEM, 3D프린터, 전기로, 현미경, 후드, 테이블, 캐비닛, 냉장고, 글로브박스, 인장시험기, XRD, EDM, PC, 모니터, 보관함, 레이저장비, 스캐너, 압연기, 의자, 경도기)
- 문 (벽 스냅, 경첩 좌/우, 열림 안/밖, swing arc)
- 방호벽 (투명, 자유배치)
- 사람 형상, 로봇팔 (리치 반경)
- 다중 공간 (인접 방)
- 테이블탑 자동 스냅
- 줄자, 충돌 감지 (빨간 경고)
- 저장/불러오기, 스크린샷 (단일 + 4방향)
- GLB 모델/사진 업로드
- 카메라 프리셋, 배경색, 치수 표시
- 색상 팔레트, 키보드 단축키
