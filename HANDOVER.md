# 이어서 작업하기

다른 PC나 새 대화에서 이 프로젝트를 이어갈 때 **이 파일 하나만 읽으면** 됩니다.
Claude Code 에게는 이렇게 말해주세요 — *"snap-reference 프로젝트 이어서 작업. HANDOVER.md 읽어줘."*

---

## 1. 이게 무슨 프로젝트인가

친구끼리 **스냅사진 레퍼런스**를 6개 컨셉으로 나눠 모으고, 서로 실시간으로 확인하는 웹 보드.
빌드 도구 없는 **단일 파일 웹앱**(`index.html`)이고, 데이터는 Supabase에 있습니다.

## 2. 주소

| 용도 | 주소 |
|---|---|
| **앱 (우리용)** | https://snap-reference-omega.vercel.app |
| **보기 전용 (제3자)** | https://snap-reference-omega.vercel.app/?view |
| 테스트 모드 | https://snap-reference-omega.vercel.app/?test |
| 기획 문서 | https://snap-reference-omega.vercel.app/docs.html |
| 코드 저장소 | https://github.com/ri0ms/snap-reference |

## 3. 세 서비스가 맡은 일

```
브라우저
  │ ① 주소 입력
  ▼
[ Vercel ]   ──▶ index.html 을 내려줌 (화면·기능)
  │ ② 페이지가 스스로 연결
  ▼
[ Supabase ] ──▶ 사진·댓글을 읽고 씀
```

- **GitHub** `ri0ms/snap-reference` — 코드 보관. 여기에 올리면 Vercel이 **자동 배포**합니다
- **Vercel** — 배포. 주소는 고정이고, 새 커밋마다 같은 주소의 내용만 갈아끼워집니다
- **Supabase** 프로젝트 `noqhumgrvthbyamszdzf` — 사진 파일 + 데이터

> Supabase URL과 공개 키는 `index.html` 상단 설정 블록에 들어 있습니다. 따로 적어둘 필요 없습니다.
> `sb_secret_...` 로 시작하는 키는 **절대 코드에 넣지 마세요.**

## 4. 개발 환경 — 알아둬야 할 제약

| 항목 | 상태 |
|---|---|
| **git** | 이 PC에 **설치 안 됨.** 파일은 GitHub 웹에서 드래그로 업로드 |
| 업로드용 폴더 | `바탕화면\snap-reference-업로드` — 올릴 파일만 담아둔 사본 |
| 작업 폴더 | `바탕화면\기타` — 원본 |
| 로컬 확인 서버 | `.claude/serve.ps1` + `launch.json` (Windows 전용, 상대 경로라 어디서든 동작) |

**수정 → 반영 순서**: 작업 폴더에서 고침 → 업로드 폴더로 복사 → GitHub 웹에 드래그 → 1분 뒤 자동 배포.

## 5. 확정된 디자인 결정

자세한 내용은 `Reference-DESIGN.md`. 핵심만 적으면:

| 항목 | 값 |
|---|---|
| 배경 | `#F2F2F0` (오프화이트, **다크 모드 없음** — 사진 톤 판단 기준선 고정) |
| 강조색 | `#0D7370` 틸 그린 — 제목과 누를 수 있는 것에만 |
| 영문 제목 | DM Serif Display 400 |
| 카테고리 이름 (탭·카드) | Parisienne 필기체, **대소문자 혼용** (대문자만 쓰면 안 읽힘) |
| 한글·본문 | Pretendard |
| 사진 그리드 | 저스티파이드(행 높이 통일), 모바일은 2열 고정 |
| 사진 보기 | 3단계 — 목록 클릭 → 크게 보기 → (사진 한 번 더) 전체 화면 → (또 한 번) 2배 확대. 뒤로 가기가 한 겹씩 닫습니다 |
| 사진 넘기기 | `‹ ›` · `←` `→` · **좌우 스와이프** (50px 이상 · 0.7초 이내 · 가로가 세로보다 클 때만. 2배 확대 중에는 밀어보기라 넘기지 않음) |
| 사람 구분 | 사진 오른쪽 위 동그라미. **혜림 `#D63A2F`(빨강) · 은서 `#2F8F5B`(초록)** 고정 |

**하지 말아야 할 것** — 강조색을 `--ink-60`(#6B6B68)보다 훨씬 어둡게 바꾸지 마세요. 탭에서 "색이 바뀐 것"으로 안 읽히고 "진해진 것"으로 보입니다. 짙은 남색·자주·올리브가 전부 여기서 실패했습니다.

## 6. Supabase 구조 (복구가 필요할 때)

```sql
create table photos (
  id text primary key, cat text not null, folder text, name text,
  ratio real not null, path text not null, uploader text not null,
  created_at timestamptz default now()
);
create table folders (
  id text primary key, cat text not null, name text not null,
  created_at timestamptz default now()
);
create table comments (
  id bigserial primary key,
  photo_id text references photos(id) on delete cascade,
  who text not null, body text not null,
  created_at timestamptz default now()
);
create table covers (
  cat text primary key,
  photo_id text references photos(id) on delete set null
);

alter table photos   enable row level security;
alter table folders  enable row level security;
alter table comments enable row level security;
alter table covers   enable row level security;
create policy "open" on photos   for all using (true) with check (true);
create policy "open" on folders  for all using (true) with check (true);
create policy "open" on comments for all using (true) with check (true);
create policy "open" on covers   for all using (true) with check (true);

alter publication supabase_realtime add table photos;
alter publication supabase_realtime add table comments;
alter publication supabase_realtime add table covers;
```

**저장소**: `photos` 버킷 (Public). 정책 3개가 **모두** 있어야 합니다.

```sql
create policy "up"  on storage.objects for insert to anon with check (bucket_id = 'photos');
create policy "del" on storage.objects for delete to anon using (bucket_id = 'photos');
create policy "sel" on storage.objects for select to anon using (bucket_id = 'photos');
```

> `sel` 정책이 빠지면 **삭제가 조용히 실패**합니다. 파일을 찾을 수 없어서 지우지 못하는데 에러도 안 납니다. 실제로 한 번 겪었습니다.

## 7. 테스트 규칙 — 꼭 지켜주세요

보드는 **실제로 쓰이는 중**입니다. 사용자: **혜림 · 은서** (2026-09-18 기준 사진 8장).

- 테스트는 **`?test`** 로 접속합니다 → 업로더 이름이 `__test__` 로 고정되고, 저장된 닉네임을 건드리지 않습니다
- 정리는 **`__test__` 로 표시된 것만**. 사진마다 **파일이 2개**(`<id>.jpg` 원본 + `<id>_t.jpg` 목록용 사본)이니 둘 다 지워야 합니다:
  ```js
  const mine = await sb.from('photos').select('id,path').like('uploader','__test__%');
  const paths = [];
  mine.data.forEach(p => { paths.push(p.path); paths.push(p.id + '_t.jpg'); });
  await sb.storage.from('photos').remove(paths);
  await sb.from('photos').delete().like('uploader','__test__%');
  await sb.from('comments').delete().like('who','__test__%');
  ```
- **절대 하지 말 것**: `delete().neq('id','')` 같은 전체 삭제, `storage.list()` 결과 전부 삭제
- 지우기 전에 "지울 것 / 남길 것"을 먼저 뽑아 확인하세요

> Supabase 무료 플랜은 **백업을 내려받을 수 없습니다.** 지우면 복구가 안 됩니다.

## 8. 남은 제약과 다음 단계 후보

| 항목 | 현재 | 비고 |
|---|---|---|
| 쓸 수 있는 사람 | **`혜림` · `은서` 두 이름만** (`index.html` 의 `ALLOWED`) | 다른 이름은 첫 화면에서 막힘. **화면에서만 막는 방식** |
| 권한 | 위 제한도 `?view` 도 우회 가능 | 서버로 막으려면 Supabase Auth 필요 |
| 링크 붙여넣기 (OG 이미지 추출) | **미구현** | 다른 사이트를 읽으려면 서버(프록시) 필요 |
| 사진 저장 | 사진 1장 = 파일 2개. **원본** `<id>.jpg` (긴 변 2400px / q88) + **목록용 사본** `<id>_t.jpg` (긴 변 800px / q82) | 목록은 사본을, 크게 보기는 원본을 씁니다. 사본이 없으면 원본으로 자동 대체 |
| 줄이는 위치 | **Worker(별도 스레드)** — 한 번 디코딩해 두 사본을 함께 만듭니다 | 메인 스레드에서 하면 12MP 한 장에 화면이 140ms 멈춥니다. Worker 를 못 쓰는 브라우저는 예전 방식으로 내려갑니다 |
| HEIC | PC 크롬에서 못 읽음 | 아이폰 사파리에서는 정상 |
| 무료 플랜 정지 | 7일 무활동 시 Supabase가 쉬어감 | 대시보드에서 버튼 한 번으로 복구 |

## 9. 다른 PC에서 시작하는 순서

1. GitHub → **Code → Download ZIP** → 압축 풀기 (항상 최신)
2. 백업해둔 **`.claude` 폴더**를 그 안에 넣기 (없으면 로컬 확인만 못 함)
3. 그 폴더에서 Claude Code 열기
4. *"snap-reference 프로젝트 이어서 작업. HANDOVER.md 읽어줘."*

Supabase·Vercel·GitHub은 **로그인만 되면** 그대로 이어집니다.

---

*최종 갱신: 2026-09-18*
