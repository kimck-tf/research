# 본 하네스 작업 컨텍스트

## 본 논문 (대상)

- **논문**: *Developing of an Geometry-aware AI model for Predicting Stress Distribution in Aluminum Wheels under Impact* (JSAE 2026, Changgon Kim et al.)
- **위치**: `../260502/ref_paper/[JSAE] Manuscript_ENG_260406(submit).pdf`
- **백본**: MeshGraphNets (Pfaff, ICLR 2021), N=20 layers, H=64, B=2
- **기법**: Modified L₂ loss(+10%) + Random edge augmentation(+3%)
- **데이터**: 85 휠, 580 샘플, 200~400k 노드 (spoke 영역만, 평균 2.4%)
- **성능**: Top-20% MAPE 7.0%, KLD 0.005, R²(Top 100%) 0.712

## 선행 세션 (260502)

- **산출물**: `../260502/최종_연구동향_및_향후방향_보고서.md` (773줄, 60건 reference)
- **갭 분석**: 5축, 27건 갭, 우선순위 Top 3 (시계열 P1, Multi-scale P2, UQ P2)
- **8개 연구방향 D1~D8** — 본 도메인 불일치 문제는 **D1 (SOTA Baseline ablation)** 의 자연스러운 확장이며, **D5 (Multi-scale GNN)** 와도 부분 연관

## 본 하네스의 위치

- 본 하네스는 260502 보고서가 다루지 못한 **노드/요소 도메인 불일치 문제** 에 집중
- 학술 동향만이 아닌 **Hyundai 사내 실무 적용 가능성** 까지 평가
- 6개월 내 D1과 병행 실행 가능한 구체적 구현 로드맵 산출
- 후속 publication 1건 후보 (예: "FEM-Aware Decoding for MeshGraphNet Stress Prediction")

## 사용자(Changgon Kim) 프로파일

- 현대자동차 책임연구원, 본 논문 제1저자
- FEM(Abaqus) + GNN 양쪽에 깊은 전문성
- 실무 효용성 + 학술 정당성을 동시에 요구
- 한국어 응답 선호

## 본 하네스 산출물의 형식 요구사항

- 한국어 학술·실무 혼합 톤
- 보고서 분량: 15~25페이지 추정 (260502 대비 압축)
- 본 논문 데이터 (580 샘플, 200~400k 노드) 에 적용 가능한 구체적 권장사항
- 최종 추천 1개 + 단계적 구현 로드맵 (1/3/6개월 마일스톤)
