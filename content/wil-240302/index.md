---
emoji: 📕
title: WIL 3월 2주차
date: '2024-03-15 22:00:00'
author: zzinao
tags: Javascript prototype
categories: WIL
---

한 주간 공부한 내용 정리

## 1. Javascript Prototype

### 1-1) 자바스크립트 객체의 Prototype Link

프로토타입 기반 객체지향 설계와 클래스 기반 객체지향 설계의 차이점을 공부했다.

### 1-2) Object.create()와 new Function 상속의 차이점

class가 아닌 ES5 문법으로 상속을 진행했고 프로토타입 체이닝이 어떻게 진행되는지 알아보았다.

<bold>: 참고 자료</bold>

[프로토타입 기반 언어, 자바스크립트](https://ui.toast.com/weekly-pick/ko_20160603)

[ES6 Class는 단지 prototype 상속의 문법설탕일 뿐인가?](https://roy-jung.github.io/161007_is-class-only-a-syntactic-sugar/)

[자바스크립트는 왜 프로토타입을 선택했을까](https://medium.com/@limsungmook/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%99%9C-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85%EC%9D%84-%EC%84%A0%ED%83%9D%ED%96%88%EC%9D%84%EA%B9%8C-997f985adb42)

## 2. 자바스크립트의 비동기 개념 이해

### 2-1) 실행 컨텍스트의 동작 방식

### 2-2) Microtask, MacroTask, AnimationFrame의 차이점

각 task queue의 실행 시점과 어떠한 메서드가 속해있는지를 알아보았다.

<bold>: 참고 자료</bold>

[What the heck is the event loop anyway?](https://youtu.be/8aGhZQkoFbQ)

[Jake Archibald: 루프 속 - JSConf.Asia](https://youtu.be/cCOL7MC4Pl0?si=wLSwvos2nD-iTpD6)

## 3. V8 엔진의 hidden class와 inline caching

### 3-1) V8엔진의 코드 최적화

자바스크립트의 런타임 동적성을 v8엔진이 어떻게 최적화 하는지 알아보았다.

:참고 자료

[Understanding the V8 JavaScript Engine](https://youtu.be/xckH5s3UuX4)
