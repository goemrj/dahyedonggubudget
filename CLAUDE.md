# 🌿 Our Budget - Claude Code Project Guide

> Our money, Our future.

이 문서는 Claude Code가 Our Budget 프로젝트를 개발할 때 항상 참고하는 프로젝트 가이드이다.

---

# 프로젝트 개요

프로젝트명: Our Budget

GitHub Repository:
https://github.com/goemrj/dahyedonggubudget

배포:
GitHub Pages

현재 구조:
단일 index.html (HTML + CSS + JavaScript)

빌드 과정 없음.

main 브랜치에 push하면 GitHub Pages가 자동 배포된다.

---

# 프로젝트 목표

이 프로젝트는 단순한 가계부가 아니다.

다혜와 동구가 앞으로 오랫동안 함께 사용할 개인 자산관리 서비스이다.

개발 시 항상

안정성

사용성

유지보수성

을 최우선으로 생각한다.

최신 기술을 적용하는 것보다

기존 기능을 깨지 않는 것이 더 중요하다.

---

# 프로젝트 철학

항상 아래 원칙을 따른다.

- 기록보다 습관을.
- 기능보다 사용성을.
- 화려함보다 편안함을.
- 새로운 기술보다 안정성을.
- 혼자가 아닌 함께 사용하는 서비스를 만든다.

---

# 사용자 성향

프로젝트 오너는 개발자가 아니다.

따라서

복잡한 UI보다

직관적인 UI를 선호한다.

디자인 성향

- Toss 스타일
- Apple 스타일
- Notion 스타일

선호

- 화이트
- 네이비
- 미니멀
- 여백
- 오래 봐도 질리지 않는 디자인

비선호

- 과한 색상
- 유치한 디자인
- 핑크 테마
- 화려한 애니메이션

---

# 절대 하지 말 것

절대로

기존 기능을 임의 삭제하지 않는다.

사용자가 요청하지 않은 리팩터링을 하지 않는다.

Firestore 구조를 변경하지 않는다.

store 구조를 변경하지 않는다.

데이터를 자동 삭제하지 않는다.

사용자가 허락하기 전

commit

push

branch 변경

을 하지 않는다.

---

# 작업 순서

모든 작업은 아래 순서를 따른다.

1.

요청 분석

↓

2.

기존 코드 확인

↓

3.

변경 계획 설명

↓

4.

사용자 승인

↓

5.

구현

↓

6.

테스트

↓

7.

변경 파일 안내

↓

8.

commit / push
(사용자가 허락한 경우만)

---

# 기술 스택

Vanilla JavaScript

HTML

CSS

Firebase

Firestore (compat SDK)

GitHub Pages

빌드 없음

프레임워크 없음

---

# 프로젝트 구조

현재 프로젝트는

index.html

하나로 구성된다.

가능한 기존 구조를 유지한다.

파일 분리는 사용자가 명시적으로 요청하기 전까지 하지 않는다.

---

# 데이터 구조

store 객체 하나가 프로젝트의 모든 데이터를 가진다.

주요 컬렉션

transactions

fixedExpenses

savings

incomes

assets

assetSnapshots

goal

---

# 저장 방식

모든 변경은

save()

를 통해 이루어진다.

save()

↓

undo 기록

↓

_rev 증가

↓

debounce

↓

writeStoreNow()

↓

Firestore 저장

---

# Firestore

실시간 동기화 사용.

echo 문제는

_rev

lastLocalRev

비교 방식으로 처리한다.

JSON 문자열 비교 방식은 사용하지 않는다.

---

# Undo

Undo는

항상 Firestore 연결 여부와 관계없이

먼저 기록한다.

Undo 기능을 깨뜨리는 수정은 금지한다.

---

# UI 원칙

새 UI를 만들기 전에

기존 UI를 재사용한다.

새 버튼을 만들기 전에

기존 버튼을 확인한다.

새 Picker를 만들기 전에

기존 Picker를 확인한다.

새 DatePicker를 만들지 않는다.

기존 openDatePicker()를 사용한다.

---

# 디자인 원칙

Card 기반

적당한 Radius

심플한 Shadow

White Background

Primary Color

현재 프로젝트에서 사용하는 네이비를 유지한다.

사용자가 요청하지 않는 한

전체 테마를 변경하지 않는다.

---

# 모바일

viewport

width=680

을 유지한다.

사용자가 요청하기 전까지

width=device-width

로 변경하지 않는다.

---

# Git 규칙

작업 시작 전

git status

확인.

작업 후

변경 파일 안내.

commit

push

는

항상 사용자 승인 후.

---

# 코드 작성 원칙

새 함수를 만들기 전에

기존 함수를 재사용한다.

새 CSS를 만들기 전에

기존 CSS를 확인한다.

중복 코드를 만들지 않는다.

함수 이름을 임의 변경하지 않는다.

---

# 문서 규칙

새 기능을 만들면

필요 시

문서도 함께 수정한다.

---

# 향후 목표

Version 1

현재 웹 서비스 완성

Version 2

자산관리 고도화

Version 3

Android

iPhone

Capacitor

Version 4

AI 자동 분류

OCR

소비 분석

---

# 보류 기능

휴대폰 결제 자동 등록

(Android 전환 후 검토)

OCR

---

# Claude에게

이 프로젝트는

실제 사용 중인 프로젝트이다.

항상

안전한 수정

최소 변경

기존 패턴 유지

를 최우선으로 생각한다.

추측해서 구현하지 않는다.

모르는 부분은

반드시 사용자에게 먼저 질문한다.

사용자가 허락하지 않은

대규모 리팩터링은 하지 않는다.

항상

기존 기능이 정상 동작하는 것을

가장 중요하게 생각한다.

---

End of Document
