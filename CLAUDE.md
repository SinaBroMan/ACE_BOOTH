# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 언어 및 커뮤니케이션 규칙

- **기본 응답 언어**: 한국어
- **코드 주석**: 한국어로 작성
- **커밋 메시지**: 한국어로 작성
- **문서화**: 한국어로 작성
- **변수명/함수명**: 영어 (코드 표준 준수)

## 프로젝트 개요

**PlayDay**는 AI 도구 체험을 재미있게 소개하는 단일 HTML 페이지 애플리케이션입니다.

- **파일**: `playday_booth.txt` (2850줄, ~895KB의 단일 HTML 파일)
- **빌드 프로세스**: 없음 (순수 HTML/CSS/JavaScript)
- **브라우저 호환성**: 모던 브라우저 (Canvas, localStorage, fetch 필요)

## 아키텍처

### 주요 섹션 (3단계)

1. **Intro 오버레이** (라인 93-1500)
   - 셔터 애니메이션으로 시작
   - AI 경험 선택 (정리형/감성형/구현형)
   - "건너뛰기" 옵션 포함

2. **Runner 게임 섹션** (라인 1500-2000)
   - Canvas 기반 2D 게임
   - 플레이어 점프 + 장애물 회피
   - 유혹도(temptation) 시스템: 점수에 따라 증가 (18% → 96%)
   - 게임 오버 시 최고 점수 저장

3. **Booth 그리드 섹션** (라인 2024-2041)
   - 3개의 AI 도구 카드 표시
   - 각 카드 클릭 시 상세 정보 모달 표시

### 상태 관리

**localStorage 키**: `playday_ai_profile_v2`

```javascript
{
  experience: null | '정리형' | '감성형' | '구현형',
  highScore: number,           // runner 게임 최고 점수
  plays: number,               // 게임 플레이 횟수
  aiClicks: number,            // AI 도구 클릭 횟수
  firstInterest: null | string, // 첫 관심 부스 유형
  lastTemptation: number,      // 마지막 유혹도 (18-96)
  lastChoiceTimeMs: number     // 마지막 선택 시간
}
```

### Booth 데이터 구조

각 booth 객체 (라인 2080-2147):

```javascript
{
  id: string,              // "notebooklm-yt", "suno-exit-song", "gpt-dashboard"
  level: string,           // "1단계", "2단계", "3단계"
  stageNumber: string,     // "01", "02", "03"
  theme: string,           // "stage1", "stage2", "stage3" (색상 테마)
  title: string,           // 부스 제목
  tool: string,            // 사용 도구명
  chip: string,            // 부스 태그 (예: "3분 영상 구조화")
  tagline: string,         // 한 줄 설명
  description: string,     // 상세 설명
  guideSteps: string[],    // 4-5개의 단계별 가이드
  examples: string[],      // 2-3개의 프롬프트 예시
  tip: string,             // 팁/주의사항
  link: string             // 외부 도구 링크
}
```

## 핵심 기능 및 구현

### Runner 게임

**게임 루프** (라인 2400-2401):
- 60fps 기반 requestAnimationFrame
- 플레이어: 점프(vy = -13.8) + 중력 가속도
- 장애물: 자동 생성, 선택된 AI 경험에 따라 다른 크기/속도

**유혹도 시스템** (라인 2447-2474):
- 점수 기반으로 단계별 증가
- 40, 90, 150, 220, 320점 이상일 때마다 상승
- UI와 텍스트 업데이트 함께 실행

**Canvas 렌더링** (라인 2476-2550):
- `drawBackground()`: 그리드 배경, 지표선, 타워
- `drawPlayer()`: 플레이어 사각형 (그림자 효과 포함)
- `drawObstacles()`: 장애물 라벨 표시
- `roundRect()`: 둥근 사각형 유틸 (Canvas 기본 함수 없음)

### Modal 시스템

**Modal 구성** (라인 2045-2077):
- 배경 클릭 또는 닫기 버튼으로 닫힘
- ESC 키 지원
- 마지막 포커스 요소로 복원
- 동적 색상 테마 적용 (booth.theme별)

**상호작용**:
- 부스 카드 클릭/Enter/Space → 모달 열기
- "체험하러 가기" 버튼 → 새 창에서 tool 링크 열기

## 주요 변수 및 함수

- `boothData`: 3개 부스의 전체 데이터 배열
- `state`: 현재 사용자 상태 (localStorage에서 로드)
- `game`: Runner 게임 상태 객체 (canvas 좌표, 점수, 게임 루프)
- `trackEvent()`: 이벤트 로깅 함수 (console.log 기반)
- `saveState()`, `loadState()`: localStorage 관리
- `openModal()`, `closeModal()`: 모달 제어

## 색상 시스템

CSS 변수 사용 (라인 8-32):

- **배경**: `--bg`, `--bg-2`, `--bg-3` (진한 파랑)
- **텍스트**: `--text` (밝은 파랑), `--muted` (회색)
- **상태**: `--good` (초록), `--warn` (노랑), `--danger` (분홍)
- **스테이지 별**: `--stage-1a/1b`, `--stage-2a/2b`, `--stage-3a/3b` (그래디언트 쌍)

## 콘텐츠 수정 가이드

### Booth 추가/수정
1. `boothData` 배열에 새 객체 추가
2. `id`, `stageNumber`, `theme` 고유하게 설정
3. `guideSteps` 및 `examples`는 배열 형식 필수
4. `link`에 실제 외부 도구 URL 입력

### Runner 게임 난이도 조정
- 장애물 속도: `game.speed` (라인 2345 근처)
- 생성 빈도: `spawnRate` 수정
- 점프 높이: `game.player.vy = -13.8` 변경
- 중력값: `game.gravity` 조정

### 색상 테마 수정
- CSS 변수(`:root`) 변경
- `stage1`, `stage2`, `stage3` 클래스별 배경색 조정

## 성능 고려사항

- Canvas 렌더링: 매 프레임 전체 재그리기 (최적화 가능)
- localStorage: 비동기 아님 (자동 동기)
- 이벤트 리스너: 모달 열기/닫기, 키보드 입력, 부스 카드 클릭
- 애니메이션: CSS + Canvas 혼합 사용

## 접근성

- **Modal**: `role="dialog"`, `aria-modal="true"`, 레이블 aria 속성
- **부스 카드**: `aria-label` 포함
- **폼**: 모든 버튼에 `type="button"` 명시
- **포커스 관리**: 모달 열고 닫을 때 포커스 복원

## 브라우저 API 사용

- `localStorage`: 상태 저장/로드
- `Canvas 2D Context`: Runner 게임 렌더링
- `Intersection Observer**: 부스 카드 리빌 애니메이션
- `window.open()`: 도구 링크 새 창 열기
- `requestAnimationFrame`: 게임 루프

## 빌드 및 배포

**현재 방식**: 그대로 브라우저에서 `playday_booth.txt` 열기
- HTML 파일 자체가 완전한 애플리케이션
- 프로세싱, 번들링, 빌드 단계 불필요
- 변경사항 저장 후 브라우저 새로고침만 하면 적용

## 문제 해결

- **localStorage 초기화**: 개발자 도구 → Application → Clear All
- **게임 루프 멈춤**: Canvas 크기 재계산 필요 (윈도우 리사이즈 이벤트)
- **모달 포커스 트래프**: ESC 키 + 배경 클릭 처리 확인
