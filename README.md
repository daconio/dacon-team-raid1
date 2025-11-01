# 🚀 DACON: RAID - 팀 빌딩 시스템

<div align="center">

![Status](https://img.shields.io/badge/status-in%20development-yellow)
![Platform](https://img.shields.io/badge/platform-Replit-blue)
![License](https://img.shields.io/badge/license-MIT-green)

**데이콘 플랫폼 사용자를 위한 가이드 마켓플레이스 기반 팀 빌딩 시스템**

[빠른 시작](#-빠른-시작) • [문서](#-문서) • [기능](#-주요-기능) • [기술 스택](#-기술-스택) • [로드맵](#-개발-로드맵)

</div>

---

## 📖 프로젝트 소개

DACON: RAID는 데이콘 대회 참가자들이 **효과적으로 팀을 구성**하고 **신뢰 기반으로 협업**할 수 있도록 지원하는 플랫폼입니다.

### 🎯 핵심 가치

- **🎮 사용자 주체성**: WoW의 파티 찾기처럼 사용자가 직접 팀을 구성
- **📊 데이터 기반 투명성**: 캐글의 검증 시스템처럼 역량과 적합도를 투명하게 표시
- **🛡️ 신뢰 시스템**: 상호 평가를 통한 신뢰 점수로 '잠수', '무임승차' 방지
- **🤝 다학제적 협업**: 데이터 분석가뿐만 아니라 기획자, 디자이너, 개발자도 참여

### 💡 주요 문제 해결

| 기존 문제 | RAID 솔루션 |
|----------|------------|
| 비공식 채널로 팀 구성 (오픈채팅, 포럼) | 공식 플랫폼으로 체계적인 팀 매칭 |
| 팀원의 실력/기여도 파악 어려움 | 검증 가능한 프로필 + 적합도 점수 |
| 잠수, 무임승차 등 부정적 경험 | 신뢰 점수 시스템으로 투명한 평가 |
| 특정 역할(데이터 분석가)만 수요 집중 | 다양한 역할군 지원 (기획, 디자인, 개발) |

---

## 🚀 빠른 시작

### 사전 요구사항

- ✅ [Replit 계정](https://replit.com) (무료)
- ✅ [Neon PostgreSQL 계정](https://neon.tech) (무료)
- ✅ Node.js 기본 지식

### 5분 설치 가이드

```bash
# 1. Replit에서 프로젝트 생성
# "Create Repl" → "Node.js" 템플릿 선택 → 프로젝트명: "dacon-raid"

# 2. 의존성 설치
npm install

# 3. 환경 변수 설정 (Replit Secrets)
# DATABASE_URL=your_neon_postgresql_url

# 4. 데이터베이스 마이그레이션
cd server
npx prisma migrate dev --name init
npx prisma generate

# 5. 개발 서버 실행
cd ..
npm run dev
```

🎉 **완료!** 브라우저에서 프로젝트가 실행됩니다.

자세한 가이드는 [빠른 시작 가이드](./DACON_RAID_빠른시작가이드.md)를 참조하세요.

---

## 📚 문서

### 개발자 가이드
- 📘 [**Replit 개발 가이드**](./DACON_RAID_Replit_개발가이드.md) - 전체 개발 프로세스 (118 페이지)
- 🚀 [**빠른 시작 가이드**](./DACON_RAID_빠른시작가이드.md) - 5분 안에 실행하기
- 📡 [**API 레퍼런스**](./DACON_RAID_API_레퍼런스.md) - REST API 문서
- 🛡️ [**관리자 대시보드 가이드**](./DACON_RAID_관리자_대시보드_가이드.md) - 완전한 관리자 시스템 구현
- 🎨 [**컬러 시스템 가이드**](./DACON_RAID_컬러시스템_가이드.md) - 새로운 디자인 시스템 (밝은 테마)

### 기획 문서
- 📋 [**PRD 문서**](./DACON_RAID_PRD.md) - 제품 요구사항 정의서

---

## ✨ 주요 기능

### Phase 1: MVP (2.5개월) ✅

#### 1️⃣ 사용자 프로필 시스템
- ✅ 다면적 역량 표시 (역할, 전문 분야, 기술 스택)
- ✅ 포트폴리오 링크 연동 (GitHub, Kaggle, Blog)
- ✅ 신뢰 점수 표시 (4가지 평가 항목)

#### 2️⃣ 원정대(팀) 생성 및 모집
- ✅ 역할 슬롯 시스템 (역할별 필요 인원 정의)
- ✅ 목표 및 시간 투자 명시
- ✅ 협업 방식 설정 (디스코드, 슬랙, 오프라인 등)

#### 3️⃣ 지원 및 매칭 시스템
- ✅ 적합도 점수 자동 계산 (0-100점)
- ✅ 리더 대시보드 (지원자 관리)
- ✅ 수락/거절 및 알림 시스템

#### 4️⃣ 신뢰 시스템
- ✅ 팀 활동 후 상호 평가 (4가지 항목)
- ✅ 신뢰 점수 누적 및 프로필 반영
- ✅ 미평가 시 페널티

### Phase 2: 고도화 (6개월) 🔄

- 🔄 ML 기반 적합도 알고리즘
- 🔄 통합 팀 채팅 (WebSocket)
- 🔄 파일 공유 및 작업 트래킹 (칸반 보드)
- 🔄 뱃지 시스템 및 명예의 전당

### Phase 3: 생태계 확장 (12개월) 📅

- 📅 크로스 플랫폼 통합 (캐글, 다른 플랫폼)
- 📅 커뮤니티 기능 (포럼, Q&A)
- 📅 프로덕트 쇼케이스
- 📅 모바일 앱 (React Native)

---

## 🛠️ 기술 스택

### Frontend
```
React 19 + TypeScript
Vite (빌드 도구)
Tailwind CSS (스타일링)
Lucide Icons (아이콘)
React Router (라우팅)
```

### Backend
```
Express.js (REST API)
Node.js 18+
Prisma (ORM)
Replit Auth (인증)
```

### Database
```
Neon PostgreSQL (서버리스)
무료 티어: 0.5GB 스토리지, 무제한 쿼리
```

### 개발 도구
```
Replit (IDE + 호스팅 + 협업)
GitHub (버전 관리)
Prisma Studio (데이터베이스 GUI)
```

### 배포
```
Replit Deployments
자동 스케일링
Zero Configuration
```

---

## 📁 프로젝트 구조

```
dacon-raid/
├── client/                     # 프론트엔드
│   ├── src/
│   │   ├── components/        # UI 컴포넌트
│   │   │   ├── ui/           # 기본 UI (Button, Card)
│   │   │   ├── Header.tsx
│   │   │   └── RaidCard.tsx
│   │   ├── contexts/         # React Context
│   │   │   ├── AuthContext.tsx
│   │   │   └── DataContext.tsx
│   │   ├── pages/            # 페이지 컴포넌트
│   │   │   ├── HomePage.tsx
│   │   │   ├── RaidDetailPage.tsx
│   │   │   ├── ProfilePage.tsx
│   │   │   ├── CreateRaidPage.tsx
│   │   │   └── DashboardPage.tsx
│   │   ├── types.ts          # TypeScript 타입
│   │   ├── constants.ts      # 상수 정의
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── index.css
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── server/                    # 백엔드
│   ├── src/
│   │   ├── routes/           # API 라우트
│   │   ├── controllers/      # 비즈니스 로직
│   │   ├── middleware/       # Express 미들웨어
│   │   ├── utils/            # 유틸리티 함수
│   │   └── index.ts          # Express 서버
│   ├── prisma/
│   │   └── schema.prisma     # Prisma 스키마
│   ├── package.json
│   └── tsconfig.json
├── .replit                    # Replit 설정
├── replit.nix                 # Nix 환경 설정
└── README.md
```

---

## 🗺️ 개발 로드맵

### ✅ Completed

- [x] 프로젝트 구조 설계
- [x] 데이터베이스 스키마 설계
- [x] API 엔드포인트 설계
- [x] 개발 가이드 작성

### 🔄 In Progress (Week 1-10)

- [ ] **Week 1-2**: 프로젝트 설정 및 기반 구축
  - [ ] Replit 프로젝트 생성
  - [ ] 프론트엔드/백엔드 환경 설정
  - [ ] Neon PostgreSQL 연결
  - [ ] Replit Auth 설정

- [ ] **Week 3-4**: 사용자 프로필 시스템
  - [ ] 사용자 등록/로그인
  - [ ] 프로필 생성 및 수정 UI
  - [ ] 포트폴리오 링크 관리

- [ ] **Week 5-6**: 원정대 기능
  - [ ] 원정대 생성 폼
  - [ ] 원정대 목록 페이지
  - [ ] 원정대 상세 페이지

- [ ] **Week 7-8**: 지원 및 매칭 시스템
  - [ ] 지원하기 기능
  - [ ] 적합도 점수 계산
  - [ ] 리더 대시보드

- [ ] **Week 9-10**: 평가 시스템 및 테스트
  - [ ] 평가 양식 UI
  - [ ] 신뢰 점수 계산
  - [ ] 종단 간 테스트

### 📅 Upcoming (Week 11+)

- [ ] Phase 2: 협업 도구 및 고급 기능
- [ ] Phase 3: 생태계 확장

---

## 🎯 비즈니스 목표

| 지표 | 목표 | 현재 |
|------|------|------|
| 대회 참가율 증가 | +30% | - |
| 팀 대회 완주율 | 60% 이상 | - |
| DAU/MAU 비율 | 45% 이상 | - |
| 팀 매칭 성공률 | 75% 이상 | - |
| MVP 출시 기간 | 2.5개월 | 진행 중 |

---

## 💰 비용 절감 효과

### Replit 사용 시 (권장)
```
✅ IDE: 무료
✅ 호스팅: $0-7/월 (종량제)
✅ 데이터베이스 (Neon): 무료
✅ 협업 도구: 무료
✅ 총 개발 비용: ~$42,150 (린 팀 기준)
```

### 전통적 방식 대비
```
❌ IDE: 로컬 환경 설정
❌ 호스팅: AWS/GCP ~$50-200/월
❌ 데이터베이스: Firebase ~$100-300/월
❌ 협업 도구: GitHub Teams ~$4-21/월
❌ 총 개발 비용: ~$120,000
```

**절감 효과**: **65% 감소** 💰

---

## 👥 타깃 사용자

### 1️⃣ 성장형 신입 (24세, 김데이터)
- **목표**: 실전 경험 축적, 포트폴리오 구축
- **니즈**: 명확한 역할, 학습 기회, 초보자 친화적 팀

### 2️⃣ 실력자 중급 (28세, 박분석)
- **목표**: 상위권 입상, 강력한 팀 구성
- **니즈**: 검증된 실력자, 명확한 커밋먼트

### 3️⃣ 기획자 (30세, 이프로덕트)
- **목표**: 기획/전략 역할로 기여
- **니즈**: 다학제적 팀, 역할 명확화

### 4️⃣ 멀티플레이어 (32세, 최개발)
- **목표**: End-to-end 프로토타입 구현
- **니즈**: 명확한 개발 범위, 협업 도구

---

## 🤝 기여 가이드

### 개발 환경 설정
```bash
# 1. 저장소 클론
git clone https://github.com/your-org/dacon-raid.git
cd dacon-raid

# 2. 의존성 설치
npm install

# 3. 환경 변수 설정
cp .env.example .env
# .env 파일에 DATABASE_URL 등 설정

# 4. 데이터베이스 마이그레이션
cd server
npx prisma migrate dev
npx prisma generate

# 5. 개발 서버 실행
cd ..
npm run dev
```

### 브랜치 전략
```
main          - 프로덕션 배포 브랜치
develop       - 개발 통합 브랜치
feature/*     - 새 기능 개발
bugfix/*      - 버그 수정
hotfix/*      - 긴급 수정
```

### 커밋 메시지 규칙
```
feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 포맷팅
refactor: 코드 리팩토링
test: 테스트 추가/수정
chore: 빌드/설정 변경
```

---

## 🐛 문제 해결

### 자주 묻는 질문

#### Q: 데이터베이스 연결 오류
```bash
# DATABASE_URL 확인
echo $DATABASE_URL

# Prisma Client 재생성
cd server
npx prisma generate
npx prisma migrate dev
```

#### Q: npm install 실패
```bash
# 캐시 정리 후 재설치
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

#### Q: 포트 충돌
```bash
# 프로세스 확인 및 종료
lsof -ti:3000 | xargs kill -9
lsof -ti:5173 | xargs kill -9
```

더 많은 문제 해결 방법은 [개발 가이드 - 문제 해결](./DACON_RAID_Replit_개발가이드.md#10-문제-해결-가이드)을 참조하세요.

---

## 📞 연락처

- **제품 문의**: product@dacon-raid.com
- **기술 지원**: dev@dacon-raid.com
- **일반 문의**: hello@dacon-raid.com
- **Discord**: [DACON RAID Community](https://discord.gg/dacon-raid) (예정)

---

## 📄 라이선스

MIT License - 자유롭게 사용, 수정, 배포 가능합니다.

---

## 🙏 감사의 글

- **Replit** - 빠르고 효율적인 개발 환경 제공
- **Neon** - 서버리스 PostgreSQL 무료 티어
- **Prisma** - 타입 안전한 ORM
- **DACON 커뮤니티** - 피드백 및 아이디어 제공

---

<div align="center">

**Made with ❤️ by DACON RAID Team**

[⬆ Back to top](#-dacon-raid---팀-빌딩-시스템)

</div>
