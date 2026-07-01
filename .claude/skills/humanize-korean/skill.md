# humanize-korean

Korean AI writing humanizer - removes AI-tell patterns while preserving all content.

## TRIGGER

Invoke when the user says any of:
- "AI티 없애줘" / "AI 문체 제거" / "GPT 문체 빼줘"
- "자연스럽게 고쳐줘" (in context of AI-generated text)
- `/humanize [text or file]`
- "인간처럼 써줘" / "기계 같은 느낌 없애줘"

## OPERATING MODES

**Fast Mode** (default, 5,000자 이하):
단일 에이전트가 탐지·윤문·자기검증을 수행. 약 3분. 도구 호출 3회 제한.

**Strict Mode** (8,000자 초과 또는 --strict 플래그):
5단계 파이프라인: detector → rewriter → 병렬 감사(내용충실도 + 자연스러움) → orchestrator(롤백 가능).

## STEP-BY-STEP INSTRUCTIONS

### 1. 모드 결정
- 입력 글자 수 계산
- --strict 플래그 또는 8,000자 초과 → Strict Mode
- 그 외 → Fast Mode

### 2. Fast Mode 실행
단일 monolith 패스:
1. **탐지** - references/ai-tell-taxonomy.md의 패턴 스캔
2. **윤문** - references/rewriting-playbook.md 처방에 따라 수술적 수정
3. **자기검증** - 내용 충실도 및 변경률 확인 (목표 5-30%)

결과물: _workspace/YYYY-MM-DD-N/final.md

### 3. Strict Mode 실행
5개 순차 에이전트:
1. ai-tell-detector - JSON 출력 (patterns, severity_weighted_score, ai_tell_density)
2. korean-style-rewriter - 탐지 JSON 읽어 플래그된 span만 수정
3. content-fidelity-auditor - 의미 불변성 검증
4. naturalness-reviewer - 잔존 AI 흔적 및 과도한 편집 확인
5. orchestrator - fidelity 실패 또는 변경률 50% 초과 시 롤백; 아니면 최종 출력 조립

결과 파일 (_workspace/YYYY-MM-DD-N/):
- detection.json
- rewrite.md
- fidelity-audit.json
- naturalness-review.json
- final.md

### 4. 품질 등급
- **A등급**: S1 패턴 없음, S2 2개 이하, 70%+ 개선
- **B등급**: S1 패턴 없음, S2 4개 이하, 50%+ 개선
- **C등급**: S1 1-2개 잔존 또는 과도한 편집 → 2차 윤문 권장
- **D등급**: S1 3개+ 잔존 또는 심각한 과도 편집 → 인간 검토 권장

## 4대 불변 원칙

1. **의미 불변성** - 사실·수치·고유명사·직접인용: 100% 불변
2. **증거 기반 편집** - 탐지된 span만 수정; 깨끗한 구간은 손대지 않음
3. **장르 유지** - 칼럼은 칼럼으로, 구어체는 구어체로
4. **과도 편집 방지** - 변경률 30%에서 경고; 50%에서 중단

## 수정 금지 목록

- 숫자, 단위, 날짜
- 고유명사, 제품/모델명
- 직접인용 (따옴표 안 내용)
- 법령·규정
- 전문 용어 (불가피한 경우)

## 출력 형식

항상 제공:
1. 윤문된 텍스트
2. 요약: 제거된 패턴, 등급, 변경률 %
3. Strict Mode: _workspace/ 결과 파일 링크

## 심각도 기준

- **S1 (결정적)**: 1회 등장 = AI 신호. 모두 제거.
- **S2 (강함)**: 3회+ 등장 시 제거; 1-2회는 허용.
- **S3 (약함)**: 다른 패턴과 중첩될 때만 문제.

## 참고 문서

- 패턴 분류: references/ai-tell-taxonomy.md
- 윤문 처방집: references/rewriting-playbook.md
- 학술 근거: references/scholarship.md
