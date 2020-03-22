---
layout: single
permalink: /django/gistalk/1
title: "gistalk 강의/수업 선택 forms.py (1)"
date: 2019-12-04
categories: django gistalk
author_profile: true
comment: true
---

gistalk의 [강의평가하기][gistalk-tas-evaluate]에서 강의를 선택하다 보면 **"강의/수업 년도"**와 **"강의/수업 학기"**가 위에서 선택된 과목에 따라서 특정되지 않고 2010년부터 2019년까지 모든 학기에 대해서 선택을 할 수 있도록 되어있다는 것을 알 수 있다.

이런 방식은 불분명한 강의/수업 년도 및 학기를 선택할 수 있게 하기 때문에 학기를 특정해주도록 하면 좋겠다.

현재 강의/수업 년도 및 학기를 입력 받는 방식을 forms.py에서 찾아볼 수 있었다. 여기서 몇 가지 수정 사항을 찾을 수 있었다.

1. 강의/수업 년도와 강의/수업 학기를 따로 입력받지 말고, 같이 입력 받자.
2. db에 접근해서 각 강의 별 개강 여부를 확인하여 띄어주도록 하자.

자세한 개선 방법에 관해서는 더 자세히 공부를 해봐야 알 것 같고, 이번에는 지금 작성된 코드 일부를 보고 알아본 내용을 정리하겠다.

## Form fields

django에서 입력을 받기 위해서는 forms.py에서 Form클래스를 사용하는데, fields를 이용해서 무엇을 입력 받을 지를 결정한다.
field를 정의할 때, 각 field들이 어떤 특성을 가지고 있는지 알려주어야 하는데 그 특성들 중에는 다음과 같은 것들이 있다.
이와 관련해서는 [django doc][django-docs-forms-field]에서 확인할 수 있다.

## label

label은 각 field들이 입력을 받으려고 interface에 출력될 때 나타나는 일종의 **이름**이다.

## widget

widget은 어떤 형식으로 입력을 받을 건지를 의미한다.
html의 input type과 비슷한 것 같다.
이와 관련해서는 [django docs][django-docs-forms-widget]에서 확인할 수 있다.

공부한 내용은 더 많지만 일단 여기서 정리하기로 하고 일단 블로그 포스트 글씨 크기를 줄이는 법을 먼저 알아봐야겠다.

[django-docs-forms-field]: https://docs.djangoproject.com/en/3.0/ref/forms/fields/
[django-docs-forms-widget]: https://docs.djangoproject.com/en/3.0/ref/forms/widgets/
[gistalk-tas-evaluate]: http://gistalk.net/tas/evaluate/
