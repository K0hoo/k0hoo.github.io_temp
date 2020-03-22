---
layout: single
permalink: /django/gistalk/3
title: "Custom Command를 통한 데이터 세팅(seed_qusetion, seed_course)"
date: 2020-03-23
categories: gistalk
author_profile: true
comments: true
---

## Custom Command

django의 commands는 manage.py에 의해 실행된다. manege.py를 이용하는 [커스텀 command][django-docs-management-command]를 만들 수 있다.

```python
rom django.core.management.base import BaseCommand, CommandError
from polls.models import Question as Poll

class Command(BaseCommand):
    help = 'Closes the specified poll for voting'

    def add_arguments(self, parser):
        parser.add_argument('poll_ids', nargs='+', type=int)

    def handle(self, *args, **options):
        for poll_id in options['poll_ids']:
            try:
                poll = Poll.objects.get(pk=poll_id)
            except Poll.DoesNotExist:
                raise CommandError('Poll "%s" does not exist' % poll_id)

            poll.opened = False
            poll.save()

            self.stdout.write(self.style.SUCCESS('Successfully closed poll "%s"' % poll_id))
```

- add_argument: argument를 추가한다.
- handle: 코맨트를 구현하는 함수다.

커스텀 코맨드를 통해 2가지 코맨드를 만들었다.

## seed_qustion

question을 만드는 command이다. question의 구성은 gistalk1.0과 같도록 했다.
question에 대한 정보를 가지고 있는 csv파일이 필요하다.
question은 course와는 다르게 한 번 세팅하면 끝이니 자세한 설명(CSV파일 세팅)은 생략하고 절차만 설명하겠다.

### 절차

1. Course Group(Combination)을 만든다.
2. Question Set을 만든다.
3. Course Group과 Question Set을 연결시켜준다. (Question Set의 ManyToManyField)
4. CSV파일을 읽어서 Question을 만들고 Question을 Question Set과 연결시켜준다.

## seed_course

course를 만드는 command이다. 사실 course보다는 Lesson을 만드는데 포커싱이 되어있고, Lesson을 만들 때 참조할 Course가 없으면 Course를 만든다. 이미 만들어진 Course가 있으면 Course를 만들지는 않는다.

seed_course도 CSV파일이 필요하다. CSV파일을 만드는 것은 [이 포스트][tas-course-import]를 많이 참고했다.(아마 비밀번호가 필요하다.)

zeus에서 강의를 로드하면 network response에 select.do라는 파일(?)이 나오는데 로드된 강의들에 대한 정보를 가지고 있다.

![zeus-course-list](/assets/images/post/django/2020-03-22/zeus-course-text.JPG)

[위의 링크][tas-course-import]에서 이 text를 어떻게 다룰지에 다루고 있지만 워드와 엑셀이 없는지라(ㅎㅎ;) 새로운 방식을 찾아보았다.

### Course CSV파일 만드는 과정 예시) 2020년도 1학기

**준비물**

- [libre office calc][libre-office]
- [libre office writer][libre-office]
- [notepad++][notepad]
- window 메모장

**한 학기마다 따로따로 해야한다. 여러 학기를 한 번에 할 수 없다.**

1. text들 사이에 있는 빨간 점들이 어떤 character인지는 잘 모르겠지만 libre office writer에 옮겨보면 *#*으로 보인다.

   > 이 텍스트는 과목 리스트를 2번 반복해서 가지고 있어서(데이터 구조는 다르다.) 반복되는 지점을 찾아서 아래 텍스트를 취해주고 위쪽은 버려도 된다.
   > 위쪽은 강사목록을 원하는 방식대로 취하기가 쉽지 않다.
   > 아래 사진 처럼 dataset의 인덱스가 있는 부분을 기점으로 아래를 취하면 된다.

   ![libre-writer-index](/assets/images/post/django/2020-03-22/libre-writer-index.JPG)

2. 텍스트를 notepad++로 옮긴다. 왜인지는 잘 모르겠는데, 다른 텍스트 에디터들에서는 개행문자(\n, \t)가 인식되지 않았다.(아마 인코딩 문제인 것 같긴 하다.) 그런데 notepad++는 개행문자에 대한 인식이 아주 잘 된다.

3. 줄 바꿈을 없애준다. (모두 선택(ctrl + a) 후 줄바꿈 해제(ctrl + j)) 바꾸기(ctrl + h)를 통해, **#2020**을 **\n2020**로 바꿔주고(줄 바꿈이 생기고 첫 번째 줄 이후의 모든 줄의 첫머리가 2020이어야 한다.) **#**을 **\t**로 바꿔준다.

   > 텍스트를 잘 살펴보면 #을 기점으로 데이터 항목들이 바뀌는 것을 알 수 있다. 이 텍스트를 CSV로 전환할 수 있도록 하는 것이 우리의 목표인데, CSV파일로 바꾸기 위해서는 어디서 행을 바꾸고 어디서 열을 바꿀지를 정하는 것이 중요하다. CSV로 바꿀 때, 줄바꿈(\n)은 열을 탭(\t)은 행바꿈을 해준다.

   ![notepad-change](/assets/images/post/django/2020-03-22/notepad-change.JPG)

4. 파일을 저장하고 libre office calc로 열어보자. 문자집합은 **utf-8**로 구분 기호는 탭이 켜져 있으면 된다. 다시보니 기타에 #을 입력하면 됐을 것 같다.

   ![notepad-change](/assets/images/post/django/2020-03-22/libre-office-calc.JPG)

5. 필요없어 보이는 행은 없애고, 행은 다음과 같은 순서로 맞추어준고, 1열은 index를 넣어주는 용도로 쓴다.(1열에 데이터를 넣으면 그 데이터는 db에 들어가지 않는다.)

   - A: 년도
     _2019, 2020_
   - B: 학기
     > spring, summer, fall, winter 네 가지 중에 하나를 넣어야 한다.
   - C: 과목코드
     > BS2102
   - D 분반 1, 2
     > 숫자로 분반을 넣는다. 1, 2
   - E 전공
     > Major모델에 저장되어 있는 전공이면 그 전공과 연결시켜주고 없으면 새로 만들 것이다.
   - F 과목명(한글)
     > 분자 세포학, 세포 생물학
   - G 과목명(영어)
     > Molecular Biology, Cell Biology
   - H 필수/선택
     > 필수과목인지 선택과목인지 구분
   - I 교과/논문연구
     > 교과과목인지 논문연구 과목인지 구분(보통 학사논문연구만 논문연구다.)
   - J 교수 리스트 "김철수[00001] 박준희[00002]"
     > 이름[번호]순으로 되어 있어야 하며, 교수들 사이에는 공백이 있어야 한다.
   - K 학사/석사/박사
     > 학사, 석사, 또는 박사가 있어야 하며, 공통 과목인 경우에는 공백이 들어간다.
   - L 강의 3
   - M 실험 0
   - N 학점 3
     > 셋 다 숫자가 들어가야 하며 강의/실험/학점을 나타낸다.
     >
     > 각자 다른 행에 들어가야 하며 이를 위해 /를 \t으로 바꿔주어야 한다.
   - O 장소 생명과학부(BT포함)(326)
     > 비어있어도 된다.
   - P 시간 화10:30~12:00 목10:30~12:00
     > 비어있어도 되며, 요일과 시간 사이에 공백이 있으면 안 된다. '월 '을 '월'로 고치듯 모든 요일 사이에 공백을 없앤다.
     > 2020 1학기의 경우 All 00:00~00:00라는 데이터가 있었는데, 공백으로 만들어야 한다.(MOOC과목)
   - Q 수강정원
     > 0인 경우, 수강 제한 없음.
   - R 수강인원

   그리고 CSV파일로 저장한다. 이 과정에서 오류가 많이 생길 것 같은데 문제가 있으면 댓글을 남겨주세요.

6. SCV파일을 tas_data 폴더에 넣고 커맨드 창에 다음 커맨드를 입력한다.
   ```
   python manage.py seed_course --year 년도 --season 학기
   python manage.py seed_course --year 2020 --season spring
   ```

**함수 절차**

CSV파일을 array로 전환한 다음, 루프를 돌려서 한 lesson씩 추가한다.

1. 대응되는 Course가 없으면(과목 코드와 한글 과목명으로 매칭)새로 Course를 추가한다.

2. 대응되는 Major가 없으면 Major를 새로 만들어서 (찾은 혹은 만든)Course에 연결한다.

3. 대응되는 Instructor가 없으면(번호로 매칭) 새로 Instructor를 추가한다.

4. 대응되는 TimeStampModel가 없으면(요일, 시작 시간, 끝나는 시간으로 매칭) 새로 TimeStampModel을 추가한다.

5. 강사들이 많은 경우는 강사들 명수만큼 루프를 돌리며 Lesson을 만든다.(강사가 없는 강의의 경우 하나만 만든다.)

6. db세팅이 끝나면 다음과 같은 안내문이 나온다. Semester가 등록이 안 되어 있었다면 새로 첫 줄이 다르게 나온다.

   ```
   A semester(year: 2020, season: speing) is already exist
   All lectures included in the file were held in 2020 spring.
   ```

## PS

변경이 생기면 다시 새로운 포스트 링크를 걸어두겠습니다.

[django-docs-management-command]: https://docs.djangoproject.com/en/3.0/howto/custom-management-commands/
[tas-course-import]: https://bangseogs.tistory.com/45
[libre-office]: https://www.libreoffice.org/download/download/
[notepad]: https://notepad-plus-plus.org/downloads/
