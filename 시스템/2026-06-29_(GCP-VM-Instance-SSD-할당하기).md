# GCP VM Instance에 SSD 디스크 할당하기
- **카테고리**: #시스템 #GoogleCloud
- **태그**: #시스템 #GoogleCloud #SSD #디스크 #마운트 #Linux
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-21_GCP  VM Instance에 SSD할당하기]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. SSD 할당 절차]]
- [[#3. 연관 개념]]

## 1. 핵심 요약
GCP Console에서 VM에 SSD 디스크를 추가한 뒤, Linux에서 ext4 포맷 → 마운트 → `/etc/fstab` 등록하는 절차. Oracle DB용 데이터 디렉토리(`/ssd/oradata`) 구성 예시 포함. 모든 명령은 **root**로 수행.

## 2. SSD 할당 절차

### Step 1. GCP Console에서 디스크 추가
`VM Instance` → 인스턴스 편집 → `새 디스크 추가` → 유형: SSD, 용량: 60GB 설정 후 저장

### Step 2. 디바이스명 확인
```bash
lsblk
# /dev/sdb 등으로 신규 디스크 확인
```

### Step 3. ext4 포맷
```bash
mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb
```

### Step 4. 마운트 디렉토리 생성
```bash
mkdir -p /ssd/oradata
```

### Step 5. 마운트
```bash
mount -o discard,defaults /dev/sdb /ssd/oradata
```

### Step 6. 권한 부여 및 쓰기 확인
```bash
chmod a+w /ssd/oradata

# Oracle DB에서 해당 경로에 Tablespace datafile 생성 가능 여부 확인
```

```sql
CREATE TABLESPACE test_st
  DATAFILE '/ssd/oradata/test01.dbf' SIZE 20M;
```

### Step 7. `/etc/fstab` 등록 (부팅 시 자동 마운트)
```bash
# 기존 fstab 백업
cp /etc/fstab /etc/fstab.bkup

# UUID로 fstab에 등록
echo UUID=`blkid -s UUID -o value /dev/sdb` /ssd/oradata ext4 discard,defaults,nofail 0 2 | tee -a /etc/fstab
```

### Step 8. 재기동 확인
재기동 후 `/ssd/oradata`가 정상 마운트되는지 확인.

## 3. 연관 개념
