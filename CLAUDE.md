# Heroes of Might and Magic III 한글화 프로젝트

## EUC-KR 인코딩 파일 수정 프로세스

bitmap 디렉토리의 텍스트 파일들은 EUC-KR (또는 CP949) 인코딩으로 되어 있어, 직접 수정이 어렵습니다.
다음 프로세스를 따라 수정합니다:

### 1. EUC-KR → UTF-8 변환

```bash
iconv -f EUC-KR -t UTF-8 원본파일.txt > 원본파일.txt.utf8
```

### 2. UTF-8 파일 수정

UTF-8로 변환된 파일을 Read, Edit 도구로 수정합니다.

### 3. UTF-8 → EUC-KR 변환

```bash
iconv -f UTF-8 -t EUC-KR 원본파일.txt.utf8 > 원본파일.txt.new
```

### 4. 백업 및 교체

```bash
mv 원본파일.txt 원본파일.txt.bak
mv 원본파일.txt.new 원본파일.txt
```

### 전체 예시

```bash
cd bitmap

# 변환
iconv -f EUC-KR -t UTF-8 advevent.txt > advevent.txt.utf8

# 수정 (Edit 도구 사용)

# 역변환
iconv -f UTF-8 -t EUC-KR advevent.txt.utf8 > advevent.txt.new

# 백업 및 교체
mv advevent.txt advevent.txt.bak
mv advevent.txt.new advevent.txt
```

## 수정 완료 내역

### 2026-01-20

#### advevent.txt
- 라인 1: "교관은당신이" → "교관은 당신이"
- 라인 22: "한떼의" → "한 떼의"
- 라인 25: "주변의%s들이" → "주변의 %s들이"
- 라인 25: "얻기위해" → "얻기 위해"

#### artevent.txt
- 라인 14: "그당신에" → "그 당신에"
- 라인 18: "날아다니는게" → "날아다니는 게"
- 라인 36: "한떼의" → "한 떼의"

#### SeerHut.txt
- 라인 5: "&d 레벨이 되었군" → "%d 레벨이 되었군"
- 라인 5: "받겠나?I " → "받겠나?"

## 줄바꿈 형식 (중요!)

HoMM3 텍스트 파일은 **혼합 줄바꿈 형식**을 사용합니다. 이를 지키지 않으면 게임이 크래시됩니다.

### 원본 파일의 줄바꿈 규칙

| 위치 | 줄바꿈 형식 | 바이트 |
|------|-------------|--------|
| 일반 행 끝 | CRLF | `0d 0a` |
| Description 내부 | LF만 | `0a` |

### Description 필드 구조 예시

```
"{마법책}←LF
←LF (빈 줄)
설명 텍스트"←CRLF
다음행...←CRLF
```

- Description 필드: 마지막 열의 `"{제목}\n\n설명}"` 형식
- 제목 뒤 줄바꿈과 빈 줄은 **LF만** 사용
- 행의 끝(다음 데이터 행으로 넘어갈 때)은 **CRLF** 사용

### 크래시 원인

텍스트 에디터로 파일을 열고 저장하면 모든 LF가 CRLF로 변환될 수 있습니다.
이 경우 게임이 Description 내부의 CRLF를 새로운 행으로 잘못 파싱하여 **NULL 포인터 접근 크래시**가 발생합니다.

### 줄바꿈 복구 방법

수정 후 줄바꿈이 깨진 경우, 다음 2단계로 복구합니다:

```bash
cd bitmap

# 1단계: CRLF+CRLF → CRLF+LF (빈 줄 수정)
xxd -p 파일.txt | tr -d '\n' | sed 's/0d0a0d0a/0d0a0a/g' | xxd -r -p > temp.txt
mv temp.txt 파일.txt

# 2단계: CRLF+LF → LF+LF (Description 내부 수정)
xxd -p 파일.txt | tr -d '\n' | sed 's/0d0a0a/0a0a/g' | xxd -r -p > temp.txt
mv temp.txt 파일.txt
```

### 줄바꿈 검증

```bash
# CR과 LF 개수 확인
cr=$(od -A n -t x1 파일.txt | tr ' ' '\n' | grep -c '^0d$')
lf=$(od -A n -t x1 파일.txt | tr ' ' '\n' | grep -c '^0a$')
echo "CR=$cr, LF=$lf"

# 원본과 비교하여 CR < LF 이면 정상 (혼합 형식)
# CR = LF 이면 모두 CRLF로 변환된 상태 (수정 필요)
```

## 주의사항

1. 원본 파일은 항상 `.bak` 확장자로 백업합니다
2. UTF-8 임시 파일은 `.utf8` 확장자를 사용합니다
3. `iconv` 명령어는 Windows에서 Git Bash 또는 WSL을 통해 사용 가능합니다
4. 파일 인코딩이 확실하지 않은 경우 `file 파일명` 명령으로 먼저 확인합니다
5. **텍스트 파일 수정 후 반드시 줄바꿈 형식을 검증합니다** (위 "줄바꿈 검증" 참고)
