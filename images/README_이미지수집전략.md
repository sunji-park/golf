# 골프장 이미지 자료 정리 전략

## 1. 폴더 구조 (권장)

```
images/
├── korea/
│   ├── 잭니클라우스_골프클럽_코리아/
│   │   ├── logo.png            ← 로고 (투명 배경 PNG)
│   │   ├── clubhouse.jpg       ← 클럽하우스 외관
│   │   ├── overview.jpg        ← 코스 전체 조감도
│   │   ├── signature.jpg       ← 시그니처 홀
│   │   ├── aerial.jpg          ← 항공 사진
│   │   ├── hole01.jpg ~ hole18.jpg   ← 각 홀별 사진
│   │   ├── amenity_restaurant.jpg
│   │   ├── amenity_driving.jpg
│   │   ├── map.png             ← 코스 레이아웃 맵
│   │   └── _meta.json          ← 이미지 출처·저작권 메타데이터
│   ├── 클럽나인브릿지/
│   ├── 해슬리나인브릿지/
│   └── ... (골프장별 폴더)
├── japan/
│   ├── Hirono_Golf_Club/
│   ├── Kawana_Hotel_Golf_Course/
│   └── ...
└── _samples/                    ← 샘플/테스트용
```

**폴더명 규칙**: 한글·영문 모두 허용. 공백은 `_` 로, 특수문자 `\/:*?"<>|` 는 제거.
현재 상위 30개 한국·일본 골프장 폴더가 미리 생성되어 있습니다.

## 2. 이미지 수집 방법 (4가지 옵션 비교)

### 옵션 A. 공식 홈페이지 직접 크롤링 ⭐ 권장 (1순위)
- **장점**: 고해상도 원본, 저작권 안전(약관 준수 시), 일관된 브랜드 이미지
- **단점**: 사이트마다 구조 다름, robots.txt 확인 필수
- **방법**:
  - `golfcourse_database.xlsx` T열(공식 홈페이지)의 URL을 순회
  - `/gallery/`, `/course/`, `/about/` 등 서브 페이지에서 `<img>`, `og:image`, hero slider 추출
  - **반드시 준수**: robots.txt, 요청 간 1~3초 지연(rate limit), User-Agent 명시

### 옵션 B. 공식 SNS/유튜브 스크래핑
- **인스타그램/페이스북 공식 계정**: 최신 필드 상태, 이벤트, 계절감 있는 사진
- **유튜브 홈 커버 이미지**: `https://www.youtube.com/channel/ID` 에서 배너/썸네일
- 수집 도구: `instaloader` (인스타), `yt-dlp --write-thumbnail` (유튜브)

### 옵션 C. 예약 플랫폼 공개 이미지
- **XGolf, 김캐디, 티업, 골프존카운티, GolfZon, Smoothround** — 제휴 업체 이미지
- **일본**: GDO (golfdigest.co.jp), 樂天GORA (gora.golf.rakuten.co.jp)
- 주의: 재배포 시 라이선스 확인

### 옵션 D. Google/Naver 이미지 검색 API
- **Google Custom Search JSON API** (일 100건 무료)
  - 쿼리: `"골프장명" site:officialdomain.com filetype:jpg`
- **Naver 검색 API** (한국 골프장에 유리, 하루 25,000건 무료)
- 단점: 저작권 검증을 개별로 해야 함

## 3. 메타데이터(`_meta.json`) 권장 스키마

폴더마다 이미지의 출처·라이선스 추적을 위해 다음 JSON 저장:

```json
{
  "course_id": "KR-001",
  "course_name": "잭니클라우스 골프클럽 코리아",
  "last_updated": "2026-04-21",
  "images": [
    {
      "filename": "clubhouse.jpg",
      "source_url": "https://www.jacknicklausgolfclubkorea.com/img/clubhouse.jpg",
      "source_type": "official_website",
      "license": "official_use_only",
      "width": 1920,
      "height": 1080,
      "collected_at": "2026-04-21T10:30:00Z"
    }
  ]
}
```

## 4. 자동 다운로드 스크립트 (샘플)

`crawl_images.py` 템플릿이 `images/` 폴더 루트에 생성되어 있습니다.
기본 흐름:

1. `golfcourse_database.xlsx` 읽기 → 각 골프장의 이름·홈페이지 URL 리스트 생성
2. 홈페이지에서 `<img>` 태그 + `og:image` 메타태그 수집
3. 해상도 1024×768 이상만 필터링 (로고 제외)
4. 파일명 정리 규칙 적용 (hole01.jpg, clubhouse.jpg 등)
5. `_meta.json` 기록 + 실패 로그(`_errors.log`) 저장

**실행 전 체크리스트**:
- [ ] 각 사이트의 robots.txt 확인
- [ ] 상업적 이용 여부 결정 (내부 DB vs 공개 서비스)
- [ ] 연락처 명시된 User-Agent 설정 (ex: `GolfNowBot/1.0 (contact@domain.com)`)
- [ ] 요청 간 최소 2초 지연
- [ ] 1차는 50개 샘플로 테스트 후 스케일업

## 5. 이미지 품질 기준 (권장)

| 용도            | 최소 해상도     | 파일 크기      | 포맷           |
|----------------|----------------|----------------|----------------|
| 썸네일(리스트) | 400×300        | ≤50KB          | JPEG, WebP    |
| 상세페이지     | 1200×800       | ≤300KB         | JPEG(progressive) |
| 히어로/배너    | 1920×1080      | ≤800KB         | JPEG, WebP    |
| 로고           | 512×512 투명   | ≤100KB         | PNG, SVG      |
| 코스맵         | 원본 유지      | ≤2MB           | PNG, PDF      |

원본 확보 후 **3단계 리사이징 파이프라인** (imagemagick 또는 sharp.js)으로 자동 생성 권장:
- `*_thumb.jpg`, `*_md.jpg`, `*_hero.jpg`

## 6. 저작권·법적 고려사항

- **한국**: 공표된 저작물의 인용은 공정한 관행 내에서 가능 (저작권법 제28조). 상업적 DB 제공 시 각 골프장과 이미지 이용 계약 필요
- **일본**: 著作権法 제32조 — 인용 허용되나 출처 표시 필수
- **권장 접근**: (1) 먼저 공공데이터포털 API 활용, (2) 주요 100개소는 직접 연락해 제휴 이미지 받기, (3) 나머지는 공개 플레이스홀더 + 사용자 업로드 허용

## 7. 빠른 시작 (3단계)

```bash
# 1. 필수 패키지 설치
pip install requests beautifulsoup4 pillow openpyxl

# 2. 크롤러 실행 (50개 샘플)
python images/crawl_images.py --limit 50 --sheet korea

# 3. 결과 확인
ls images/korea/*/
```

---
_최종 업데이트: 2026-04-21_
