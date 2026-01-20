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

## 주의사항

1. 원본 파일은 항상 `.bak` 확장자로 백업합니다
2. UTF-8 임시 파일은 `.utf8` 확장자를 사용합니다
3. `iconv` 명령어는 Windows에서 Git Bash 또는 WSL을 통해 사용 가능합니다
4. 파일 인코딩이 확실하지 않은 경우 `file 파일명` 명령으로 먼저 확인합니다
