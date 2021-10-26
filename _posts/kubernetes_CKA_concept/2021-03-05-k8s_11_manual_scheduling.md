---
layout: single
title: "[CKA Concept] 11. Manual Scheduling"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Manual Scheduling에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Manual Scheduling
- Manual Scheduling을 번역하면 **수동으로하는 스케줄링**을 뜻한다.
- 오브젝트 구성파일에는 원래 nodeName이라는 항목을 따로 넣어주지 않지만 내부적으로 스케줄러가 Pod 생성시 노드를 배정할 때 nodeName이라는 항목에 적절한 node를 넣어준다.

ex)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  nodeName: node01
...
...
```

- **만약 Scheduler가 없거나 동작하지않는다면?** <br>-> Pod을 생성할경우에 Pod의 상태가 **Pending(보류)** 상태로 머물러있을 것이다.
- 이러한 경우 사용자가 임의로 구성파일에 **nodeName 항목을 추가**하여 노드를 배정해주면 정상적으로 Pod이 생성된다.
- 하지만 이러한 노드배정은 Pod을 생성할 때만 가능하며, 생성되어있는 Pod의 Node를 수정할 수 는 없다.
- 생성된 Pod의 Node를 수정하고 싶다면 Binding 오브젝트를 생성 한뒤 Pod Binding API에 POST 요청을 보냄

<br><br>


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests