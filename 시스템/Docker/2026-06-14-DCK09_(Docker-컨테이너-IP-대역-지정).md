# Docker 컨테이너 IP 대역 지정 (daemon.json)

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #network #IP대역 #daemon #bridge
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Docker container의 IP 대역지정 - BigDataTeam]]

## 1. 핵심 요약

Docker 기본 IP 대역(`172.17.0.0/16`)이 사내 네트워크와 충돌할 때 `/etc/docker/daemon.json`으로 커스텀 IP 대역을 지정한다. `bip`는 docker0 브리지 IP, `default-address-pools`는 사용자 정의 네트워크 풀을 설정한다.

---

## 2. 설정 (/etc/docker/daemon.json)

```bash
cat <<EOF > /etc/docker/daemon.json
{
  "bip": "172.30.0.1/24",
  "default-address-pools": [
    { "base": "172.31.16.0/20", "size": 24 }
  ]
}
EOF

# 적용
sudo systemctl restart docker
```

## 3. 항목 설명

| 항목 | 설명 |
|------|------|
| `bip` | docker0 기본 브리지 IP (172.30.0.1 → 172.30.0.254 범위) |
| `default-address-pools.base` | 사용자 정의 네트워크 전체 풀 (172.31.16.0~172.31.31.255) |
| `default-address-pools.size` | 새 네트워크 생성 시 서브넷 크기 (/24씩 순차 할당) |

> 첫 번째 custom 네트워크 → `172.31.16.0/24`, 두 번째 → `172.31.17.0/24` …

## 4. 연관 개념

- [[2026-06-14-DCK05_(Docker-Compose-주요-명령)]]
