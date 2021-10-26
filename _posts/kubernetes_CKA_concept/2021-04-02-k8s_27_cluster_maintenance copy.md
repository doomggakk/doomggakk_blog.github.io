---
layout: single
title: "[CKA Concept] 27. Cluster Maintenance"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Cluster Maintenance에 대해 정리한다. 
toc: true
toc_sticky: true
---

## OS Upgrade
- 쿠버네티스에서 OS에 관련된 업데이트를 해야하고 어느 Node가 다운되어야 할때 그대로 진행시켜버리면 해당 Node안에 있던 App들을 접속이 불가능해진다.
- 그리고 나서 Node가 다시 정상으로 돌아왔을 때 App이 정상작동하게 된다. App이 기능하지 않는 시간이 존재하게된다.
- 이런 경우에 **drain명령**을 사용하여 해당Node의 App들을 안전하게 다른 Node에 임시로 옮겨두는 기능을 수행한다.
- Node가 다시 복구되면 uncordon 명령을 통해 해당 Node에 다시 Scheduling 되도록 한다.
- 하지만 이러한 경우 이전에 있던 App들이 다시 해당 Node에 생성되지는 않는다. 새로생긴 Pod만 정상적으로 생성이 된다.

<br>

- ```drain``` : node안에 있는 app들을 다른 node에 옮기고 schedule 되지 않도록 함 -> maintenance를 진행함

- ```uncordon``` : 이제 노드가 사용가능해졌으니 가져다 쓰라는 의미를 부여한다. 

- ```cordon``` : kubectl cordon은 지정된 노드에 더이상 포드들이 스케쥴링되서 실행되지 않도록 합니다.
해당 노드에서 생성된 Pod에는 SchedulingDisabled Status가 추가된다. 

## Version
- Version관리는 아래의 규칙을 따른다.

![](/assets/img/kubernetes/27_version_1.png)
[출처]Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- 첫째자리(major) : 
- 둘째자리(minor) : 주로 1달에 한번 기능변경 및 추가 된 사항 배포
- 셋째자리(patch) : Bug를 고친다거나 할 때 수시로 배포

<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 초기화 컨테이너](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)
- [#25 Cluster Maintenance (drain, uncordon) - 작성자 ijoos](https://blog.naver.com/ijoos/222167125237)