# 📡 DACON: RAID - API 레퍼런스

**Base URL**: `https://your-project.repl.co/api`  
**버전**: 1.0  
**인증**: Replit Auth Headers

---

## 📋 목차

1. [인증](#1-인증)
2. [사용자 관리](#2-사용자-관리)
3. [대회 관리](#3-대회-관리)
4. [원정대 관리](#4-원정대-관리)
5. [지원 관리](#5-지원-관리)
6. [평가 관리](#6-평가-관리)
7. [에러 코드](#7-에러-코드)

---

## 1. 인증

### 인증 방식
Replit Auth를 사용하며, 모든 요청에 다음 헤더를 포함해야 합니다:

```http
X-Replit-User-Id: {user_id}
X-Replit-User-Name: {user_name}
X-Replit-User-Email: {user_email}
```

### 1.1 현재 사용자 정보 조회

```http
GET /auth/me
```

**Headers:**
```
X-Replit-User-Id: required
X-Replit-User-Name: required
X-Replit-User-Email: required
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "replitUserId": "string",
  "email": "user@example.com",
  "displayName": "홍길동",
  "photoURL": "https://...",
  "bio": "데이터 분석가입니다.",
  "roles": ["데이터 분석가", "ML 엔지니어"],
  "specializations": ["CV", "NLP"],
  "skills": ["Python", "PyTorch", "TensorFlow"],
  "githubUrl": "https://github.com/user",
  "kaggleUrl": "https://kaggle.com/user",
  "blogUrl": "https://blog.com",
  "trustCommitment": 4.5,
  "trustContribution": 4.2,
  "trustCommunication": 4.8,
  "trustCollaboration": 4.6,
  "totalReviews": 15,
  "createdAt": "2025-01-01T00:00:00.000Z",
  "updatedAt": "2025-01-10T12:00:00.000Z"
}
```

**Errors:**
- `401 Unauthorized`: 인증 헤더 없음
- `404 Not Found`: 사용자를 찾을 수 없음

### 1.2 사용자 등록/업데이트

```http
POST /auth/register
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required
X-Replit-User-Name: required
X-Replit-User-Email: required
```

**Request Body:**
```json
{
  "displayName": "홍길동",
  "photoURL": "https://...",
  "roles": ["데이터 분석가"],
  "specializations": ["CV", "NLP"],
  "skills": ["Python", "PyTorch"],
  "bio": "데이터 분석가입니다.",
  "githubUrl": "https://github.com/user",
  "kaggleUrl": "https://kaggle.com/user",
  "blogUrl": "https://blog.com"
}
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "replitUserId": "string",
  "email": "user@example.com",
  "displayName": "홍길동",
  // ... (1.1과 동일한 구조)
}
```

**Errors:**
- `400 Bad Request`: 필수 필드 누락
- `401 Unauthorized`: 인증 실패
- `500 Internal Server Error`: 서버 오류

---

## 2. 사용자 관리

### 2.1 특정 사용자 조회

```http
GET /users/:userId
```

**Parameters:**
- `userId` (path, required): 사용자 ID

**Response (200 OK):**
```json
{
  "id": "uuid",
  "displayName": "홍길동",
  "photoURL": "https://...",
  "bio": "데이터 분석가입니다.",
  "roles": ["데이터 분석가"],
  "specializations": ["CV", "NLP"],
  "skills": ["Python", "PyTorch"],
  "portfolioLinks": {
    "github": "https://github.com/user",
    "kaggle": "https://kaggle.com/user",
    "blog": "https://blog.com"
  },
  "trustScore": {
    "commitment": 4.5,
    "contribution": 4.2,
    "communication": 4.8,
    "collaboration": 4.6,
    "totalReviews": 15
  },
  "createdAt": "2025-01-01T00:00:00.000Z",
  "updatedAt": "2025-01-10T12:00:00.000Z"
}
```

**Errors:**
- `404 Not Found`: 사용자를 찾을 수 없음

### 2.2 사용자 프로필 수정

```http
PATCH /users/:userId
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required (본인만 수정 가능)
```

**Request Body:**
```json
{
  "displayName": "홍길동2",
  "bio": "업데이트된 소개",
  "roles": ["데이터 분석가", "ML 엔지니어"],
  "skills": ["Python", "PyTorch", "TensorFlow"]
}
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "displayName": "홍길동2",
  // ... (업데이트된 전체 프로필)
}
```

**Errors:**
- `403 Forbidden`: 본인 프로필이 아님
- `404 Not Found`: 사용자를 찾을 수 없음

---

## 3. 대회 관리

### 3.1 대회 목록 조회

```http
GET /competitions
```

**Query Parameters:**
- `isActive` (boolean, optional): 활성 대회만 조회 (default: true)
- `type` (string, optional): 대회 유형 필터 ("Tabular", "CV", "NLP", "RL")
- `page` (number, optional): 페이지 번호 (default: 1)
- `limit` (number, optional): 페이지당 항목 수 (default: 20)

**Example:**
```
GET /competitions?isActive=true&type=CV&page=1&limit=10
```

**Response (200 OK):**
```json
{
  "competitions": [
    {
      "id": "uuid",
      "daconId": "dacon-123",
      "name": "이미지 분류 대회",
      "type": "CV",
      "startDate": "2025-01-01T00:00:00.000Z",
      "endDate": "2025-03-31T23:59:59.000Z",
      "isActive": true,
      "createdAt": "2024-12-01T00:00:00.000Z",
      "updatedAt": "2024-12-01T00:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25,
    "totalPages": 3
  }
}
```

### 3.2 특정 대회 조회

```http
GET /competitions/:id
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "daconId": "dacon-123",
  "name": "이미지 분류 대회",
  "type": "CV",
  "startDate": "2025-01-01T00:00:00.000Z",
  "endDate": "2025-03-31T23:59:59.000Z",
  "isActive": true,
  "createdAt": "2024-12-01T00:00:00.000Z",
  "updatedAt": "2024-12-01T00:00:00.000Z"
}
```

**Errors:**
- `404 Not Found`: 대회를 찾을 수 없음

---

## 4. 원정대 관리

### 4.1 원정대 목록 조회

```http
GET /raids
```

**Query Parameters:**
- `status` (string, optional): 상태 필터 ("recruiting", "full", "active", "completed")
- `competitionId` (string, optional): 대회 ID로 필터
- `goal` (string, optional): 목표 필터 ("학습 중심", "상위권 목표", "프로덕트 완성")
- `sortBy` (string, optional): 정렬 기준 (default: "createdAt")
- `order` (string, optional): 정렬 순서 ("asc", "desc", default: "desc")
- `page` (number, optional): 페이지 번호 (default: 1)
- `limit` (number, optional): 페이지당 항목 수 (default: 20)

**Example:**
```
GET /raids?status=recruiting&goal=상위권 목표&sortBy=createdAt&order=desc&page=1&limit=10
```

**Response (200 OK):**
```json
{
  "raids": [
    {
      "id": "uuid",
      "name": "상위권 진입 원정대",
      "description": "상위 10% 진입을 목표로...",
      "status": "recruiting",
      "goal": "상위권 목표",
      "expectedHoursPerWeek": "10시간 이상",
      "collaborationMethod": "디스코드",
      "creator": {
        "id": "uuid",
        "displayName": "홍길동",
        "photoURL": "https://..."
      },
      "competition": {
        "id": "uuid",
        "name": "이미지 분류 대회",
        "type": "CV"
      },
      "slots": [
        {
          "id": "uuid",
          "role": "데이터 분석가",
          "level": "중급",
          "count": 2,
          "filled": 1
        }
      ],
      "members": [
        {
          "userId": "uuid",
          "role": "데이터 분석가",
          "user": {
            "id": "uuid",
            "displayName": "김철수",
            "photoURL": "https://..."
          }
        }
      ],
      "_count": {
        "applications": 5
      },
      "createdAt": "2025-01-01T00:00:00.000Z",
      "updatedAt": "2025-01-10T12:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 45,
    "totalPages": 5
  }
}
```

### 4.2 특정 원정대 상세 조회

```http
GET /raids/:id
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "name": "상위권 진입 원정대",
  "description": "상위 10% 진입을 목표로...",
  "status": "recruiting",
  "goal": "상위권 목표",
  "expectedHoursPerWeek": "10시간 이상",
  "collaborationMethod": "디스코드",
  "creator": {
    "id": "uuid",
    "displayName": "홍길동",
    "photoURL": "https://...",
    "trustScore": {
      "commitment": 4.5,
      "contribution": 4.2,
      "communication": 4.8,
      "collaboration": 4.6,
      "totalReviews": 15
    }
  },
  "competition": {
    "id": "uuid",
    "name": "이미지 분류 대회",
    "type": "CV",
    "endDate": "2025-03-31T23:59:59.000Z"
  },
  "slots": [
    {
      "id": "uuid",
      "role": "데이터 분석가",
      "level": "중급",
      "count": 2,
      "filled": 1
    },
    {
      "id": "uuid",
      "role": "ML 엔지니어",
      "level": "고급",
      "count": 1,
      "filled": 0
    }
  ],
  "members": [
    {
      "userId": "uuid",
      "role": "데이터 분석가",
      "slotId": "uuid",
      "joinedAt": "2025-01-02T00:00:00.000Z",
      "user": {
        "id": "uuid",
        "displayName": "김철수",
        "photoURL": "https://...",
        "trustScore": {
          "commitment": 4.7,
          "contribution": 4.5,
          "communication": 4.6,
          "collaboration": 4.8,
          "totalReviews": 10
        }
      }
    }
  ],
  "applications": [
    {
      "id": "uuid",
      "applicantId": "uuid",
      "appliedRole": "ML 엔지니어",
      "status": "pending",
      "fitScore": 85.5,
      "message": "ML 엔지니어로 지원합니다...",
      "appliedAt": "2025-01-05T10:00:00.000Z",
      "applicant": {
        "id": "uuid",
        "displayName": "이영희",
        "photoURL": "https://...",
        "roles": ["ML 엔지니어"],
        "trustScore": {
          "commitment": 4.6,
          "contribution": 4.4,
          "communication": 4.7,
          "collaboration": 4.5,
          "totalReviews": 8
        }
      }
    }
  ],
  "createdAt": "2025-01-01T00:00:00.000Z",
  "updatedAt": "2025-01-10T12:00:00.000Z"
}
```

**Errors:**
- `404 Not Found`: 원정대를 찾을 수 없음

### 4.3 원정대 생성

```http
POST /raids
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required
```

**Request Body:**
```json
{
  "name": "상위권 진입 원정대",
  "description": "상위 10% 진입을 목표로 하는 원정대입니다. 주 2-3회 온라인 회의를 통해 진행 상황을 공유하고, 디스코드로 실시간 소통합니다.",
  "competitionId": "uuid",
  "goal": "상위권 목표",
  "expectedHoursPerWeek": "10시간 이상",
  "collaborationMethod": "디스코드",
  "slots": [
    {
      "role": "데이터 분석가",
      "level": "중급",
      "count": 2
    },
    {
      "role": "ML 엔지니어",
      "level": "고급",
      "count": 1
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "id": "uuid",
  "name": "상위권 진입 원정대",
  "description": "상위 10% 진입을 목표로...",
  "status": "recruiting",
  "goal": "상위권 목표",
  "expectedHoursPerWeek": "10시간 이상",
  "collaborationMethod": "디스코드",
  "createdBy": "uuid",
  "competitionId": "uuid",
  "creator": {
    "id": "uuid",
    "displayName": "홍길동",
    "photoURL": "https://..."
  },
  "competition": {
    "id": "uuid",
    "name": "이미지 분류 대회",
    "type": "CV"
  },
  "slots": [
    {
      "id": "uuid",
      "role": "데이터 분석가",
      "level": "중급",
      "count": 2,
      "filled": 0
    },
    {
      "id": "uuid",
      "role": "ML 엔지니어",
      "level": "고급",
      "count": 1,
      "filled": 0
    }
  ],
  "createdAt": "2025-01-10T15:30:00.000Z",
  "updatedAt": "2025-01-10T15:30:00.000Z"
}
```

**Errors:**
- `400 Bad Request`: 필수 필드 누락 또는 유효하지 않은 값
- `401 Unauthorized`: 인증 실패
- `404 Not Found`: 대회를 찾을 수 없음

### 4.4 원정대 수정

```http
PATCH /raids/:id
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required (원정대 리더만 수정 가능)
```

**Request Body:**
```json
{
  "name": "수정된 원정대 이름",
  "description": "수정된 설명",
  "status": "active",
  "collaborationMethod": "슬랙"
}
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "name": "수정된 원정대 이름",
  // ... (업데이트된 전체 원정대 정보)
}
```

**Errors:**
- `403 Forbidden`: 원정대 리더가 아님
- `404 Not Found`: 원정대를 찾을 수 없음

### 4.5 원정대 삭제

```http
DELETE /raids/:id
```

**Headers:**
```
X-Replit-User-Id: required (원정대 리더만 삭제 가능)
```

**Response (200 OK):**
```json
{
  "message": "Raid deleted successfully"
}
```

**Errors:**
- `403 Forbidden`: 원정대 리더가 아님
- `404 Not Found`: 원정대를 찾을 수 없음

---

## 5. 지원 관리

### 5.1 원정대에 지원하기

```http
POST /applications
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required
```

**Request Body:**
```json
{
  "raidId": "uuid",
  "appliedRole": "ML 엔지니어",
  "appliedSlotId": "uuid",
  "message": "ML 엔지니어로 지원합니다. PyTorch와 TensorFlow를 활용한 CV 모델 구축 경험이 3년 이상 있으며, 특히 object detection과 segmentation 분야에 강점이 있습니다. 주 15시간 이상 투자 가능하며, 디스코드로 실시간 소통 가능합니다."
}
```

**Response (201 Created):**
```json
{
  "id": "uuid",
  "applicantId": "uuid",
  "raidId": "uuid",
  "appliedRole": "ML 엔지니어",
  "appliedSlotId": "uuid",
  "message": "ML 엔지니어로 지원합니다...",
  "status": "pending",
  "fitScore": 85.5,
  "appliedAt": "2025-01-10T16:00:00.000Z",
  "applicant": {
    "id": "uuid",
    "displayName": "이영희",
    "photoURL": "https://...",
    "roles": ["ML 엔지니어"],
    "skills": ["Python", "PyTorch", "TensorFlow"],
    "trustScore": {
      "commitment": 4.6,
      "contribution": 4.4,
      "communication": 4.7,
      "collaboration": 4.5,
      "totalReviews": 8
    }
  },
  "raid": {
    "id": "uuid",
    "name": "상위권 진입 원정대",
    "creator": {
      "id": "uuid",
      "displayName": "홍길동"
    },
    "competition": {
      "id": "uuid",
      "name": "이미지 분류 대회"
    }
  }
}
```

**Errors:**
- `400 Bad Request`: 필수 필드 누락 또는 이미 지원한 원정대
- `401 Unauthorized`: 인증 실패
- `404 Not Found`: 원정대 또는 사용자를 찾을 수 없음

### 5.2 특정 원정대의 지원자 목록 조회 (리더 전용)

```http
GET /applications/raid/:raidId
```

**Headers:**
```
X-Replit-User-Id: required (원정대 리더만 조회 가능)
```

**Response (200 OK):**
```json
[
  {
    "id": "uuid",
    "applicantId": "uuid",
    "appliedRole": "ML 엔지니어",
    "appliedSlotId": "uuid",
    "message": "ML 엔지니어로 지원합니다...",
    "status": "pending",
    "fitScore": 85.5,
    "appliedAt": "2025-01-10T16:00:00.000Z",
    "applicant": {
      "id": "uuid",
      "displayName": "이영희",
      "photoURL": "https://...",
      "bio": "ML 엔지니어입니다.",
      "roles": ["ML 엔지니어"],
      "specializations": ["CV", "Object Detection"],
      "skills": ["Python", "PyTorch", "TensorFlow"],
      "portfolioLinks": {
        "github": "https://github.com/user",
        "kaggle": "https://kaggle.com/user"
      },
      "trustScore": {
        "commitment": 4.6,
        "contribution": 4.4,
        "communication": 4.7,
        "collaboration": 4.5,
        "totalReviews": 8
      }
    }
  },
  {
    "id": "uuid",
    "applicantId": "uuid",
    "appliedRole": "데이터 분석가",
    "appliedSlotId": "uuid",
    "message": "데이터 분석가로 지원합니다...",
    "status": "pending",
    "fitScore": 78.2,
    "appliedAt": "2025-01-09T14:30:00.000Z",
    "applicant": {
      "id": "uuid",
      "displayName": "박민수",
      "photoURL": "https://...",
      // ... (applicant 정보)
    }
  }
]
```

**Errors:**
- `403 Forbidden`: 원정대 리더가 아님
- `404 Not Found`: 원정대를 찾을 수 없음

### 5.3 지원 수락/거절

```http
PATCH /applications/:id
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required (원정대 리더만 가능)
```

**Request Body:**
```json
{
  "status": "accepted"  // "accepted" 또는 "rejected"
}
```

**Response (200 OK):**
```json
{
  "id": "uuid",
  "applicantId": "uuid",
  "raidId": "uuid",
  "appliedRole": "ML 엔지니어",
  "appliedSlotId": "uuid",
  "message": "ML 엔지니어로 지원합니다...",
  "status": "accepted",
  "fitScore": 85.5,
  "appliedAt": "2025-01-10T16:00:00.000Z",
  "respondedAt": "2025-01-11T09:00:00.000Z",
  "applicant": {
    "id": "uuid",
    "displayName": "이영희",
    // ... (applicant 정보)
  },
  "raid": {
    "id": "uuid",
    "name": "상위권 진입 원정대",
    // ... (raid 정보)
  }
}
```

**참고:**
- `status: "accepted"` 시 자동으로:
  1. `RaidMember` 생성
  2. `RaidSlot.filled` 증가
  3. 모든 슬롯이 찼을 경우 `Raid.status` → `"full"`로 변경

**Errors:**
- `400 Bad Request`: 유효하지 않은 상태 값 또는 슬롯이 가득 참
- `403 Forbidden`: 원정대 리더가 아님
- `404 Not Found`: 지원을 찾을 수 없음

### 5.4 내 지원 현황 조회

```http
GET /applications/my-applications
```

**Headers:**
```
X-Replit-User-Id: required
```

**Response (200 OK):**
```json
[
  {
    "id": "uuid",
    "raidId": "uuid",
    "appliedRole": "ML 엔지니어",
    "appliedSlotId": "uuid",
    "message": "ML 엔지니어로 지원합니다...",
    "status": "pending",
    "fitScore": 85.5,
    "appliedAt": "2025-01-10T16:00:00.000Z",
    "raid": {
      "id": "uuid",
      "name": "상위권 진입 원정대",
      "status": "recruiting",
      "creator": {
        "id": "uuid",
        "displayName": "홍길동",
        "photoURL": "https://..."
      },
      "competition": {
        "id": "uuid",
        "name": "이미지 분류 대회",
        "type": "CV",
        "endDate": "2025-03-31T23:59:59.000Z"
      }
    }
  },
  {
    "id": "uuid",
    "raidId": "uuid",
    "appliedRole": "데이터 분석가",
    "appliedSlotId": "uuid",
    "message": "데이터 분석가로 지원합니다...",
    "status": "accepted",
    "fitScore": 92.0,
    "appliedAt": "2025-01-05T10:00:00.000Z",
    "respondedAt": "2025-01-06T14:30:00.000Z",
    "raid": {
      "id": "uuid",
      "name": "학습 중심 원정대",
      "status": "active",
      // ... (raid 정보)
    }
  }
]
```

---

## 6. 평가 관리

### 6.1 평가 제출

```http
POST /reviews
```

**Headers:**
```
Content-Type: application/json
X-Replit-User-Id: required
```

**Request Body:**
```json
{
  "raidId": "uuid",
  "revieweeId": "uuid",
  "commitment": 5,
  "contribution": 4,
  "communication": 5,
  "collaboration": 4,
  "feedback": "적극적으로 참여해주셔서 감사합니다. 특히 모델 실험 결과를 상세히 공유해주셔서 팀 전체에 큰 도움이 되었습니다."
}
```

**참고:**
- 평가 항목은 1-5점 척도
- `feedback`은 선택 사항

**Response (201 Created):**
```json
{
  "id": "uuid",
  "raidId": "uuid",
  "reviewerId": "uuid",
  "revieweeId": "uuid",
  "commitment": 5,
  "contribution": 4,
  "communication": 5,
  "collaboration": 4,
  "feedback": "적극적으로 참여해주셔서...",
  "createdAt": "2025-03-31T10:00:00.000Z",
  "reviewer": {
    "id": "uuid",
    "displayName": "홍길동",
    "photoURL": "https://..."
  },
  "reviewee": {
    "id": "uuid",
    "displayName": "이영희",
    "photoURL": "https://..."
  },
  "raid": {
    "id": "uuid",
    "name": "상위권 진입 원정대",
    "competition": {
      "name": "이미지 분류 대회"
    }
  }
}
```

**Errors:**
- `400 Bad Request`: 필수 필드 누락, 평가 범위 초과, 또는 이미 평가함
- `401 Unauthorized`: 인증 실패
- `404 Not Found`: 원정대 또는 사용자를 찾을 수 없음

### 6.2 특정 사용자가 받은 평가 조회

```http
GET /reviews/user/:userId
```

**Query Parameters:**
- `page` (number, optional): 페이지 번호 (default: 1)
- `limit` (number, optional): 페이지당 항목 수 (default: 20)

**Response (200 OK):**
```json
{
  "reviews": [
    {
      "id": "uuid",
      "raidId": "uuid",
      "commitment": 5,
      "contribution": 4,
      "communication": 5,
      "collaboration": 4,
      "feedback": "적극적으로 참여해주셔서...",
      "createdAt": "2025-03-31T10:00:00.000Z",
      "reviewer": {
        "id": "uuid",
        "displayName": "홍길동",
        "photoURL": "https://..."
      },
      "raid": {
        "id": "uuid",
        "name": "상위권 진입 원정대",
        "competition": {
          "name": "이미지 분류 대회"
        }
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 8,
    "totalPages": 1
  },
  "averageScores": {
    "commitment": 4.6,
    "contribution": 4.4,
    "communication": 4.7,
    "collaboration": 4.5,
    "overall": 4.55
  }
}
```

### 6.3 특정 원정대의 평가 조회

```http
GET /reviews/raid/:raidId
```

**Response (200 OK):**
```json
[
  {
    "id": "uuid",
    "commitment": 5,
    "contribution": 4,
    "communication": 5,
    "collaboration": 4,
    "feedback": "적극적으로 참여해주셔서...",
    "createdAt": "2025-03-31T10:00:00.000Z",
    "reviewer": {
      "id": "uuid",
      "displayName": "홍길동",
      "photoURL": "https://..."
    },
    "reviewee": {
      "id": "uuid",
      "displayName": "이영희",
      "photoURL": "https://..."
    }
  }
]
```

---

## 7. 에러 코드

### HTTP 상태 코드

| 코드 | 설명 | 예시 |
|------|------|------|
| 200 | OK | 요청 성공 |
| 201 | Created | 리소스 생성 성공 |
| 400 | Bad Request | 잘못된 요청 (필수 필드 누락, 유효하지 않은 값) |
| 401 | Unauthorized | 인증 실패 또는 인증 헤더 없음 |
| 403 | Forbidden | 권한 없음 (예: 다른 사용자의 리소스 수정 시도) |
| 404 | Not Found | 리소스를 찾을 수 없음 |
| 409 | Conflict | 중복 리소스 (예: 이미 지원한 원정대) |
| 429 | Too Many Requests | Rate Limit 초과 |
| 500 | Internal Server Error | 서버 내부 오류 |

### 에러 응답 형식

```json
{
  "error": "Error message",
  "message": "Detailed error description (development only)",
  "statusCode": 400
}
```

### 일반적인 에러 메시지

#### 인증 관련
```json
{
  "error": "Authentication required",
  "statusCode": 401
}
```

```json
{
  "error": "Invalid authentication",
  "statusCode": 401
}
```

#### 권한 관련
```json
{
  "error": "Not authorized to update this raid",
  "statusCode": 403
}
```

```json
{
  "error": "Not authorized to view applications",
  "statusCode": 403
}
```

#### 리소스 관련
```json
{
  "error": "Raid not found",
  "statusCode": 404
}
```

```json
{
  "error": "User not found",
  "statusCode": 404
}
```

#### 유효성 검증
```json
{
  "error": "Missing required fields",
  "statusCode": 400
}
```

```json
{
  "error": "Already applied to this raid",
  "statusCode": 400
}
```

```json
{
  "error": "Slot is already full",
  "statusCode": 400
}
```

#### Rate Limiting
```json
{
  "error": "Too many requests from this IP, please try again later.",
  "statusCode": 429
}
```

---

## 8. Rate Limiting

### 제한 사항
- **윈도우**: 15분
- **최대 요청 수**: 100 요청
- **적용 범위**: `/api/*` 모든 엔드포인트

### Rate Limit 헤더
응답에 다음 헤더가 포함됩니다:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1609459200
```

---

## 9. 변경 이력

### v1.0 (2025-11-01)
- 초기 API 버전
- 사용자, 대회, 원정대, 지원, 평가 관리 기능

---

## 10. 지원

- 📧 기술 지원: dev@dacon-raid.com
- 📖 전체 문서: [개발 가이드](./DACON_RAID_Replit_개발가이드.md)
- 💬 커뮤니티: [Replit Discord](https://replit.com/discord)
