# GitHub 업로드 가이드 (MedQA-KR/EN)

이 디렉토리는 GitHub에 그대로 push할 수 있도록 정리되어 있습니다.
세 가지 업로드 방법 중 편한 것을 선택하세요.

---

## 방법 1: 명령줄(CLI) — 추천 (33MB 파일을 안정적으로 푸시)

### 사전 준비
- Git 설치 확인: `git --version`
- GitHub 계정 + Personal Access Token (PAT) 또는 SSH key 설정
  - PAT: https://github.com/settings/tokens (scope에 `repo` 체크)
  - 또는 SSH: https://docs.github.com/en/authentication/connecting-to-github-with-ssh

### 절차

```bash
# 1) GitHub에서 새 빈 repository 생성 (웹 UI에서)
#    이름: MedQA-KR-EN  (또는 원하는 이름)
#    Description: A Physician-Validated Bilingual Medical QA Dataset
#    Public 선택, README/LICENSE는 추가하지 마세요 (이미 포함되어 있음)

# 2) 압축 해제한 폴더로 이동
cd MedQA-KR-EN

# 3) 로컬 git 초기화
git init -b main

# 4) 모든 파일 스테이징
git add .
git status   # 추가된 파일 확인

# 5) 첫 커밋
git commit -m "Initial release of MedQA-KR/EN v1.0"

# 6) 원격 저장소 연결 (URL을 본인 계정에 맞게 변경)
git remote add origin https://github.com/<your-username>/MedQA-KR-EN.git

# 7) Push
git push -u origin main
```

PAT를 쓰면 push 시 username과 token (비밀번호 자리에 token) 입력하면 됩니다.

---

## 방법 2: GitHub Desktop (GUI)

1. https://desktop.github.com 설치
2. **File → Add Local Repository** → MedQA-KR-EN 폴더 선택
3. "create a repository" 클릭 → 이름 확인 후 **Create Repository**
4. **Publish repository** 클릭 (Public 체크, "Keep code private" 체크 해제)
5. 자동으로 GitHub에 업로드됨

---

## 방법 3: 웹 UI 드래그&드롭 (소규모/간편)

> ⚠️ 33MB CSV는 웹 업로드에서 다소 느릴 수 있음. CLI 방식 권장.

1. GitHub에서 새 repository 생성 (Public)
2. "uploading an existing file" 링크 클릭
3. 폴더 안의 모든 파일을 끌어다 놓기
4. 커밋 메시지 입력 후 **Commit changes**

---

## 업로드 후 권장 추가 설정

### 1. About 섹션 채우기
Repository 메인 페이지 우측 ⚙️ 아이콘 → 다음 정보 입력:
- Description: `A Physician-Validated Bilingual Medical QA Dataset for Licensing Examination Education`
- Website: `https://www.aihub.or.kr/aihubdata/data/view.do?dataSetSn=71874`
- Topics: `medical-dataset`, `medical-qa`, `medical-education`, `bilingual-dataset`, `kmle`, `usmle`, `llm-training`

### 2. License 자동 인식 확인
`LICENSE` 파일이 있으니 GitHub가 자동으로 "CC0 1.0" 배지를 표시합니다.

### 3. Release 만들기
- **Releases → Create a new release**
- Tag: `v1.0.0`
- Title: `MedQA-KR/EN v1.0.0`
- Description: README의 "Demonstrated utility" 표 등 핵심 내용 요약
- 이렇게 하면 DOI를 받기 위해 Zenodo에 연결할 수 있습니다 (아래 참조)

### 4. Zenodo 연결로 DOI 발급 (강력 추천)
Scientific Data submission에서 영구적인 dataset DOI를 요구할 수 있습니다.

1. https://zenodo.org/login에서 GitHub 계정으로 로그인
2. **GitHub** 탭에서 MedQA-KR-EN repository 토글을 ON
3. GitHub에서 Release를 만들면 (위 3번) Zenodo가 자동으로 archive하고 **DOI 발급**
4. 이 DOI를 논문 Data Availability 섹션과 CITATION.cff에 추가

### 5. 데이터셋 인지도 확대
- HuggingFace Datasets에도 미러: https://huggingface.co/new-dataset
  - `dataset_card.md`가 그대로 README로 동작합니다
  - 자동으로 데이터 미리보기, 검색 기능 제공
- Papers with Code dataset page 등록

---

## 파일 크기 관련 주의사항

- GitHub 단일 파일 제한: 100MB (push 거부됨)
- GitHub 권장 repo 크기: 1GB 미만
- 현재 데이터셋 총 크기: **약 74MB** (CSV 33MB + JSONL 두 개 약 41MB)
- ✅ Git LFS **불필요**. 일반 git push로 충분.

만약 향후 추가 데이터로 100MB를 넘는 단일 파일이 생기면:
```bash
git lfs install
git lfs track "data/*.csv"
git add .gitattributes
```

---

## 문제 해결

### "remote: Permission denied (publickey)"
SSH 인증 실패. PAT로 HTTPS 방식 사용하거나 SSH key 등록 확인.

### "fatal: refusing to merge unrelated histories"
GitHub에서 README/LICENSE를 만들지 마라고 했는데 만들었을 때 발생.
```bash
git pull origin main --allow-unrelated-histories
git push -u origin main
```

### Push가 매우 느림
CSV 파일 때문일 수 있음. 한 번만 업로드되니 인내심을 가지세요. 정 안 되면:
```bash
git config http.postBuffer 524288000   # 500MB buffer
git push
```

### 한글 파일명/경로 문제
이 repository에는 한글 파일명이 없도록 정리했으므로 문제 없음.

---

## 체크리스트

업로드 전 마지막 확인:

- [ ] README.md, UPLOAD_GUIDE.md의 `<your-username>`, `<your-org>` 자리표시자를 실제 GitHub 계정으로 모두 교체
- [ ] CITATION.cff의 ORCID는 실제 값으로 교체 (없으면 그 줄 삭제)
- [ ] dataset_statistics.json 내용 확인
- [ ] LICENSE 파일이 CC0임을 확인 (실제로 CC0 배포 권한이 있는지 마지막 확인)

업로드 후:

- [ ] Public 설정 확인
- [ ] About 섹션 채우기
- [ ] Topics 추가
- [ ] v1.0.0 Release 생성
- [ ] Zenodo 연결로 DOI 발급
- [ ] 논문 제출 시 DOI를 Data Availability 섹션에 기재
