---
layout: post
title: Django 좋아요/싫어요 버튼 구현(ajax 이용)
---



### 구현 기능

- 각 공약 별 좋아요/싫어요 버튼 추가

- 각 유저는 각 공약 당 좋아요/싫어요 중 한개만 선택 가능

  ​

### LikeOrDisLike 관계 테이블 추가

- 좋아요/싫어요를 위해 N:M 관계테이블에 like와 dislike 필드 추가 
- 추후 pledge 모델에서 through통해 관계 생성
- models.py

```python
class LikeOrDislike(models.Model):
    # 좋아요/싫어요를 위한 N:M 유저와 공약간의 관계모델
    user = models.ForeignKey(settings.AUTH_USER_MODEL, related_name='like_dislike') # 유저와 1:N 관계 설정
    pledge = models.ForeignKey(Pledge) # 공약과 1:N 모델 설정
    like = models.BooleanField(default=False) # 좋아요
    dislike = models.BooleanField(default=False) # 싫어요

    def __str__(self):
        return '{}공약의 좋아요 {},싫어요 {}'.format(self.pledge, self.like, self.dislike)
```



### Pledge모델에 like_dislike 필드  추가

- 유저는 여러개의 공약에 좋아요/싫어요를 할 수 있고, 공약은 여러 유저로부터 좋아요/싫어요를 받을 수 있다.
- User 모델과 N:M 관계 설정
- 참고문서 : [Many-to-many relationships](https://docs.djangoproject.com/es/1.10/topics/db/examples/many_to_many/)
- 참고문서 : [property Decorator](https://www.programiz.com/python-programming/property)
- 참고 문서 : [related_name](http://stackoverflow.com/questions/2642613/what-is-related-name-used-for-in-django)
- models.py 수정

```Python
class Pledge(models.Model):
    # 공약 모델
    congressman = models.ForeignKey(CongressMan) # 국회의원 모델과 1toN 관계 설정
    title = models.CharField(max_length=32) # 공약 이름
    status = models.IntegerField(default=0) # 공약 상태
    like_dislike = models.ManyToManyField(settings.AUTH_USER_MODEL, through='LikeOrDislike') # 좋아요/싫어요 모델 LikeOrDisLike 모델을 통해 User와 N:M 관계 설정
    # event_status = models.BooleanField(default=False) # 공약 상태변경 이벤트 활성화 상태
    description = models.TextField(max_length=1024) # 공약에 대한 추가 설명
    created_at = models.DateTimeField(auto_now_add=True) # 공약 날짜

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('pledge:pledge_detail', args = [self.pk])

    @property
    def get_total_like(self):
        # 해당 공약의 좋아요 갯수 카운트 @property 장식자를 통해 템플릿에서 쉽게 접근하게 한다.
        return LikeOrDislike.objects.filter(pledge_id=self.id, like=True).count()

    @property
    def get_total_dislike(self):
        # 해당 공약의 싫어요 갯수 카운트 @property 장식자를 통해 템플릿에서 쉽게 접근하게 한다.
        return LikeOrDislike.objects.filter(pledge_id=self.id, dislike=True).count()
```



### view에 pledge_like, dislike 메소드 추가

- 참고문서 : [view decorators](https://docs.djangoproject.com/en/1.10/topics/http/decorators/)

- ManyToManyField에서 through 속성 이용시 add, create, set을 지원하지 않는다.

- 추가 삭제 시 LikeOrDislike모델로 직접 접근

- LikeOrDislike.objects.filter(pledge=pledge, user=user).delete() # 인스턴스 삭제

- LikeOrDislike.objects.create(
  ​                user=user,
  ​                pledge=pledge,
  ​                like=True,
  ​                dislike=False,
  ​                ) # 인스턴스 생성

- views.py

  - #### 좋아요 버튼 클릭시

```Python
def pledge_like(request, pledge_pk):
    # 공약 좋아요 버튼 클릭시
    if request.is_ajax():
        # ajax 요청일 경우
        user = request.user # 요청한 유저
        pledge = get_object_or_404(Pledge, pk=pledge_pk) # 해당 공약 인스턴스 생성

        if user.like_dislike.filter(pledge_id=pledge.id).exists():
            # 좋아요/싫어요 중 하나라도 눌렀을 경우능
            # 둘 중 한개만 선택 가능
            like_instance = user.like_dislike.get(pledge_id=pledge.id) # 좋아요/싫어요 인스턴스 생성
            if like_instance.like == True:
                # 눌려있는 버튼이 좋아요일 경우
                LikeOrDislike.objects.filter(pledge=pledge, user=user).delete() # 인스턴스 삭제
            else:
                # 싫어요일 경우 좋아요로 바꾼 후 저장
                like_instance.like = True
                like_instance.dislike = False
                like_instance.save()
        else:
            # 버튼이 하나도 안눌려있을 경우 새롭게 생성
            LikeOrDislike.objects.create(
                user=user,
                pledge=pledge,
                like=True,
                dislike=False,
                )

        context = {} # 좋아요 개수와 싫어요 개수를 response에 담아 보낸다
        context['like_count'] = pledge.get_total_like
        context['dislike_count'] = pledge.get_total_dislike
    else:
        # ajax 요청이 아닐 경우
        context = {'status': 'fail'}

    # dic 형식을 json 형식으로 바꾸어 전달한다.
    return HttpResponse(json.dumps(context), content_type='application/json')
```

- #### 	싫어요 버튼 클릭 시	

```python
def pledge_dislike(request, pledge_pk):
    # 공약 싫어요 버튼 클릭 시
    if request.is_ajax():
        # ajax 요청일 경우
        user = request.user # 요청한 유저
        pledge = get_object_or_404(Pledge, pk=pledge_pk) # 해당 공약 인스턴스 생성

        if user.like_dislike.filter(pledge_id=pledge.id).exists():
            # 좋아요/싫어요 중 하나라도 눌렀을 경우
            # 둘 중 한개만 선택 가능
            like_instance = user.like_dislike.get(pledge_id=pledge.id) # 좋아요/싫어요 인스턴스 생성
            if like_instance.dislike == True:
                # 눌려있는 버튼이 싫어요일 경우
                LikeOrDislike.objects.filter(pledge=pledge, user=user).delete() # 인스턴스 삭제
            else:
                # 좋아요일 경우 싫어요로 바꾼 후 저장
                like_instance.like = False
                like_instance.dislike = True
                like_instance.save()
        else:
            # 버튼이 하나도 안눌려있을 경우 새롭게 생성
            LikeOrDislike.objects.create(
                user=user,
                pledge=pledge,
                like=False,
                dislike=True,
                )
        context = {} # 좋아요 개수와 싫어요 개수를 response에 담아 보낸다
        context['like_count'] = pledge.get_total_like
        context['dislike_count'] = pledge.get_total_dislike
    else:
        # ajax 요청이 아닐 경우
        context = {'status': 'fail'}

    # dic 형식을 json 형식으로 바꾸어 전달한다.
    return HttpResponse(json.dumps(context), content_type='application/json')
```



### url  패턴 추가

- urls.py

```python
urlpatterns = [
	url(r'^(?P<pledge_pk>\d+)/pledge_like/$', views.pledge_like, name='pledge_like'),
	url(r'^(?P<pledge_pk>\d+)/pledge_dislike/$', views.pledge_dislike, name='pledge_dislike'),
]
```



### 템플릿에 버튼 및 ajax 코드 추가

- 임시로 버튼 2개 생성 후 ajax 통신 코드 추가
- Pledge_detail.html
- javascript ajax 코드 부분

```html
<script type="text/javascript">
// 좋아요 버튼 처리
// 버튼 클릭 > ajax통신 (like url로 전달) > views의 like 메소드에서 리턴하는 값 전달받기 > 성공시 콜백 호출
$('.like').click(function(){
  var pk = $(this).attr('name') // 클릭한 요소의 attribute 중 name의 값을 가져온다.
  $.ajax({
      url: "{% url 'pledge:pledge_like' pledge.pk %}", // 통신할 url을 지정한다.
      data: {'csrfmiddlewaretoken': '{{ csrf_token }}'}, // 서버로 데이터를 전송할 때 이 옵션을 사용한다.
      dataType: "json", // 서버측에서 전송한 데이터를 어떤 형식의 데이터로서 해석할 것인가를 지정한다. 없으면 알아서 판단한다.

      success: function(response){
        // 요청이 성공했을 경우 좋아요/싫어요 개수 레이블 업데이트
        $('#like_count'+ pk).html("count : "+ response.like_count);
        $('#dislike_count'+ pk).html("count : "+ response.dislike_count);
      },
      error:function(error){
        // 요청이 실패했을 경우
        alert(error)
      }
  });
})

// 싫어요 버튼 처리
// 버튼 클릭 > ajax통신 (dislike url로 전달) > views의 dislike 메소드에서 리턴하는 값 전달받기 > 성공시 콜백 호출
$('.dislike').click(function(){
  var pk = $(this).attr('name') // 클릭한 요소의 attribute 중 name의 값을 가져온다.
  $.ajax({
      url: "{% url 'pledge:pledge_dislike' pledge.pk %}", // 통신할 url을 지정한다.
      data: {'csrfmiddlewaretoken': '{{ csrf_token }}'}, // 서버로 데이터를 전송할 때 이 옵션을 사용한다.
      dataType: "json", // 서버측에서 전송한 데이터를 어떤 형식의 데이터로서 해석할 것인가를 지정한다. 없으면 알아서 판단한다.

      success: function(response){
        // 요청이 성공했을 경우 좋아요/싫어요 개수 레이블 업데이트
        $('#like_count'+ pk).html("count : "+ response.like_count);
        $('#dislike_count'+ pk).html("count : "+ response.dislike_count);
      },
      error:function(error){
        // 요청이 실패했을 경우
        alert(error)
      }
  });
})
</script>
```

-  html 버튼 추가 부분

```html
<input type="button" class="like" name="{{ pledge.pk }}" value="Like"> <!-- 좋아요 버튼 -->
<p id="like_count{{ pledge.pk }}">count : {{ pledge.get_total_like }}</p> <!-- 좋아요 개수 표시 -->

<input type="button" class="dislike" name="{{ pledge.pk }}" value="Dislike"> <!-- 싫어요 버튼 -->
<p id="dislike_count{{ pledge.pk }}">count : {{ pledge.get_total_dislike }}</p> <!-- 싫어요 개수 표시 -->
```

