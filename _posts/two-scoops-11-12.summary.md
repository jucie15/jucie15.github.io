# 11. 장고 폼의 기초

> 어떤 데이터든 입력 데이터라고 한다면 장고 폼을 이용하여 유효성 검사를 해야 한다는 것이다.



## 11.1 장고 폼을 이용하여 모든 입력 데이터에 대한 유효성 검사하기

- 장고 폼은 파이썬 딕셔너리의 유효성을 검사하는 데 최상의 도구다.
- POST가 포함된 HTTP요청을 받아 유효성을 검사하는 데에만 이용할 필요는 없다!(Ex. CSV파일 등)

### TIP: Raising ValidationError

- 에러 원인을 명확히 하기 위해 code파라미터를 이용하라

```python
# Good 
ValidationError ( _ ( 'Invalid value' ),  code = 'invalid' )

# Bad
ValidationError ( _ ( '잘못된 값' ))
```

- 메세지내에 변수를 강요하지마라(placeholder를 이용하고, 'param' argument를 사용하자)

```python
# Good
ValidationError(
    _('Invalid value: %(value)s'),
    params={'value': '42'},
)

# Bad
ValidationError(_('Invalid value: %s') % value)
```

- 메시지내 변수 이용시 위치기반 형식 대신 키워드 매핑형식을 이용하자

```python
# Good
ValidationError(
    _('Invalid value: %(value)s'),
    params={'value': '42'},
)

# Bad
ValidationError(
    _('Invalid value: %s'),
    params=('42',),
)
```

- 번역을 위해 메시지에 gettext를 꼭 이용하자

```python
# Good
ValidationError(_('Invalid value'))

# Bad
ValidationError('Invalid value')
```

- 최종 적용

```python
raise ValidationError(
    _('Invalid value: %(value)s'),
    code='invalid',
    params={'value': '42'},
)
```

- 권장하진 않지만 clean()함수가 실행되고 어떤에러가 발생할지 명확히 알고있다면 간결하게 사용해도 상관은없다.

```python
ValidationError(_('Invalid value: %s') % value)
```



## 11.2 HTML 폼에서 POST메서드 이용하기

- 데이터를 변경하는 모든 HTML 폼은  POST 메서드를 이용한다.(예외 - 검색 폼)



## 11.3 데이터를 변경하는  HTTP 폼은 언제나 CSRF 보안을 이용해야 한다.

### 11.3.1 AJAX를 통해 데이터 추가하기

- 절대 AJAX 뷰를 CSRF 예외 처리하지 말 것!!
- 자세한 방법은 17.6 AJAX와 CSRF 토큰에서



## 11.4 장고의 폼 인스턴스 속성을 추가하는 방법 이해하기

- 때때로 장고 폼의 `clean()`, `clean_FOO()`, `save()` 메서드에 추가로 폼 인스턴스 속성이 필요할 때가 있다.

- 폼 예시
```python
    from django import forms

    from .models import Taster

    class TasterForm(forms.ModelForm):

      class Meta:
        model = Taster

      def __init__(self, *args, **kwargs):
        # user 속성 폼에 추가하기
        self.user = kwargs.pop('user') # super()를 호출하기 이전에 self.user가 추가 
		    super(TasterForm, self).__init__(*args, **kwargs)
```
- 뷰 예시
```python
    from django.views.generic import UpdateView
    
    from braces.views import LoginRequiredMixin
    
    from .forms import TasterForm
    from .models import Taster
    
    class TasterUpdateView(LoginRequiredMixin, UpdateView):
      model = Taster
      form_class = TasterForm
      success_url = "/someplace/"
    
      def get_form_kwargs(self):
        """키워드 인자들로 폼을 추가하는 메서드"""
        # 폼의 #kwargs 가져오기
        kwargs = super(TasterUpdateView, self).get_form_kwargs()
        # kwargs의 user_id 업데이트
        kwargs['user'] = self.request.user
        return kwargs
```


## 11.5 폼이 유효성을 검사하는 방법 알아두기

- `form_is_valid()` 가 호출될 때의 순서
    1. 폼이 데이터를 받으면 `form.is_valid()`는 `form.full_clean()` 메서드를 호출한다.
    2. `form.full_clean()`은 폼 필드들과 각각의 필드 유효성을 하나하나 검사하면서 다음과 같은 과정을 수행한다.
        1. 필드에 들어온 데이터에 대해 `to_python()`을 이용하여 파이썬 형식으로 변환하거나 변환할 때 문제가 생기면 `ValidationError`를 일으킨다.
        2. 커스텀 유효성 검사기(validator)를 포함한 각 필드에 특별한 유효성을 검사한다. 문제가 있을 때 `ValidationError`를 일으킨다.
        3. 폼에 `clean_<field>()` 메서드가 있으면 이를 실행한다.
    3. `form.full_clean()`이 `form.clean()` 메서드를 실행한다.
    4. `ModelForm` 인스턴스의 경우 `form.post_clean()`이 다음 작업을 한다.
        1. `form.is_valid()`가 `True`나 `False`로 설정되어 있는 것과 관계없이 `ModelForm`의 데이터를 모델 인스턴스로 설정한다.
        2. 모델의 `clean()` 메서드를 호출한다. 참고로 ORM을 통해 모델 인스턴스를 저장할 때는 모델의 `clean()` 메서드가 호출되지는 않는다.

### 11.5.1 모델 폼 데이터는 폼에 먼저 저장된 이후 모델 인스턴스에 저장된다.

- 폼 데이터가 폼 인스턴스로 변하는 것은 버그가 아닌 의도된 동작이다.
- `ModelForm` 에서 폼 데이터는 두 가지 각기 다른 단계를 통해 저장된다.
    1. 폼 데이터가 폼 인스턴스에 저장된다.
    2. 폼 데이터가 모델 인스턴스에 저장된다.

**`form.save()` 메서드에 의해 적용되기 전까지는 ModelForm이 모델 인스턴스로 저장되지 않기 때문에 위 과정 자체를 장점으로 이용할 수 있다.**

- Ex. 폼 입력 실패에 대해 자세한 사항이 필요할 때 폼 데이터와 모델 인스턴스의 변화를 따로 확인 가능하다.
```python
    # core/models.py
    from django.db import models
    
    class ModelFormFailureHistory(models.Model):
      form_data = models.TextField()
      model_data = models.TextField()
    
    
    # flavors/views.py
    import json
    
    from django.contrib import messages
    from django.core import serializers
    from core.models import ModelFormFailureHistory
    
    class FlavorActionMixin(object):
    
      @property
      def success_msg(self):
        return NotImplemented
    
      def form_valid(self, form):
        messages.info(self.request, self.success_msg)
        return super(FlavorActionMixin, self).form_valid(form)
    
      def form_invalid(self, form):
        """실패 내역을 확인하기 위해 유효성 검사에 실패한 폼과 모델을 저장한다"""
        form_data = json.dumps(form.cleaned_data)
        
        model_data = serializers.serialize("json", [form.instance])[1:-1]
        
        ModelFormFailureHistory.objects.create(
          form_data=form_data,
          model_data=model_data
        )
        return super(FlavorActionMixin, self).form_invalid(form)
```



## 11.6 Form.add_error()를 이용하여 폼에 에러 추가하기

- `Form.add_error()` 메서드를 사용하여 `Form.clean()` 을 더 간소화 할 수 있게 되었다.
```python
from django import forms

class IceCreamReviewForm(forms.Form):
  # tester 폼의 나머지 부분이 이곳에 위치
  ...
  def clean(self):
    cleaned_data = super(TasterForm, self).clean()
    flavor = cleaned_data.get("flavor")
    age = cleaned_data.get("age")

    if flavor == 'coffee' and age < 3:
      # 나중에 보여줄 에러들을 기록
      msg = u"Coffee Ice Cream is not for Babies."
      self.add_error('flavor', msg)
      self.add_error('age', msg)
      # 항상 처리된 데이터 전체를 반환한다
      return cleaned_data
```


## 요약

일단 폼을 작성하기 시작했다면 **코드의 명료성과 테스트**를 염두에 두자. 

폼은 장고 프로젝트에서 **주된 유효성 검사 도구**이며, 불의의 데이터 충돌에 대한 **중요한 방어 수단**이기도 하다.





# 12. 폼 패턴들

> 장고 개발자라면 반드시 알아야 할 다섯가지 폼 패턴을 알아보자



## 12.1 패턴 1: 간단한 모델폼과 기본 유효성 검사기

- `FOO모델`을 제네릭 뷰에서 이용하도록 한다.
- 해당 뷰에서 `FOO모델`에 기반을 둔 `ModelForm`을 자동 생성한다.
- 생성된 `ModelForm` 이 `FOO모델` 의 기본 필드 유효성 검사기를 이용하게 된다.



### 12.2 패턴 2: 모델폼에서 커스텀 폼 필드 유효성 검사기 이용하기

> Q. 모든 앱의 타이틀 필드가 'Tasty'로 시작되어야 한다면!?!?
A. 커스텀 필드 유효성 검사기로 처리할 수 있다.

1. 두 가지 다른 디저트 모델이 있다고 가정(Milkshake, Flavor)한다(두 모델에는 공통적으로 title 필드 존재) 
2. 수정 가능한 모든 모델 타이틀에 대한 유효성을 검사하기 위해 `[validator.py](http://validator.py)` 모듈 제작

   ```python
    # core/validators.py
    from django.core.exceptions import ValidationError
    
    def validate_tasty(value):
    	"""단어가 'Tasty'로 시작하지 않으면 ValidationError를 일으킨다."""
    	if not value.startswith("Tasty"):
    		msg = "Must start with Tasty"
 		raise ValidationError(msg)
   ```

    ### TIP: 유효성 검사기에 대한 테스트는 특별히 더 주의하자!!
   
    - 유효성 검사기는 데이터 충돌을 방지하는 중요한 기능을 한다. 따라서 테스트는 아주 특별한 경우를 포함한 모든 상황을 주의 깊게 진행해야 한다.
3. `core/model.py` 모듈을 만들고 프로젝트 전반에 이용될 `추상화 모델(TastyTitleAbstractModel)`을 추가한다.

  ```python
    #core/models.py
    from django.db import models
    from .validators import validate_tasty
    
    class TastyTitleAbstractModel(models.Model):
    	title = models.CharField(max_length=255, validators=[validate_tasty])
        
    	class Meta:
    		abstract = True
  ```

4. `Flavor` 모델에 `TastyTitleAbstractModel` 을 부모클래스로 지정

   ```python
    # flavor/models.py
    from django.db import models
    from core.models import TastyTitleAbstractModel
    
    class Flavor(TastyTitleAbstractModel):
        slug = models.SlugField()
   ```

> Q. 단지 폼에만 validate_tasty()를 이용하고 싶을때는?
Q. 타이틀 말고 다른 필드에 이를 적용하고 싶을 때는 어떻게 할 것인가?
A. 커스텀 필드 유효성 검사기를 이용하는 커스텀 폼을 작성하자!

1. 커스텀 폼 작성

   ```python
    # flavors/forms.py
    from django import forms
    
    from core.validators import vlidate_tasty
    from .models import Flavor
    
    class FlavorForm(forms.ModelForm):
        def __init__(self, *args, **kwargs):
            super(CakeForm, self).__init__(*args, **kwargs)
            self.fields["title"].validators.append(validate_tasty)
            self.fields["slug"].validators.append(validate_tasty)
    
        class Meta:
            model = Flavor
   ```

2. 커스텀 폼을 커스텀 뷰에 추가
```python
        # flavors/views.py
        from django.contrib import messages
        from django.views.generic import CreateView, UpdateView, DetailView
        
        from braces.views import LoginRequiredMixin
        
        from .models import Flavor
        from .forms import FlavorForm
        
        class FlavorActionMixin(object):
            model = Flavor
            fields = ('title', 'slug')
        
            @property
            def success_msg(self):
                return NotImplemented
        
            def form_valid(self, form):
                messages.info(self.request, self.success_msg)
                return super(FlavorActionMixin, self).form_valid(form)
   
        
        class FlavorCreateView(LoginRequiredMixin, FlavorActionMixin, CreateView):
            success_msg = 'created'
            form_class = FlavorForm
   
        
        class FlavorUpdateView(LoginRequiredMixin, UpdateView):
        		success_msg = 'updated'
            form_class = FlavorForm
```

**유효성 검사기를 사용하는 방법의 장점은 `validate_tasty()`의 코드를 변경하지 않고 이용할 수 있다는 것!**



## 12.3 패턴 3: 유효성 검사의 클린 메서드 오버라이딩하기

- **커스텀 로직으로 `clean()` 또는 `clean_<field_name>()` 메서드를 오버라이딩 할 최적의 경우**
    - 다중 필드에 대한 유효성 검사
    - 이미 유효성 검사가 끝난 데이터베이스의 데이터가 포함된 유효성 검사
- **어째서 유효성 검사에 또 한 번의 유효성 검사를 거치는가?**
    - `clean()` 메서드는 두 개 혹은 그 이상의 필드들에 대해 서로 간의 유효성 검사가 가능하다.
    - 이미 유효성 검사를 일부 마친 데이터에 대해 불필요한 데이터베이스 연동을 줄일 수 있다.



## 12.4 패턴 4: 폼 필드 해킹하기(두 개의 CBV, 두 개의 폼, 한 개의 모델)

- 나중에 입력할 데이터를 위해 **빈 필드를** 포함한 레코드를 생성하는 경우(Ex.각 상점을 빠르게 시스템에 저장한 후 전화번호나 설명등은 나중에 입력하는 경우)
- 폼을 이용하여 생성 시 `title`만 입력하고, `phone`은 나중에 입력하게 만들어보자
- `title`은 필수 입력 필드이지만, `phone` 은 입력하지 않아도된다.
```python
        # stores/models.py
        from django.db import models
        
        class IceCreamStore(models.Model):
        	title = models.CharField(max_length=100)
        	phone = models.CharField(max_length=20, blank=True)
```
- **(나쁜예제!!) `phone` 필드의 수정 폼을 오버라이드 하는 방법은 코드의 중복이 많다!!**
```python
        # stores/forms.py
        from django import forms
        
        from .models import IceCreamStore
        
        class IceCreamStoreUpdateForm(forms.ModelForm):
        	# 따라하지 말 것!!! 모델 필드를 반복해서 이용함('반복되는 일 하지 않기'규칙에 어긋남)
        	phone = forms.CharField(required=True)
        
        	class Meta:
        		model = IceCreamStore
```
- **(좋은 예제) `ModelForm` 의 `__init__()` 메서드에서 새로운 속성을 적용**
```python
        # stores/forms.py
        # self.fields라는 유사 딕셔너리 객체에서 phone을 호출
        from django import forms
        
        from .models import IceCreamStore
        
        class IceCreamStoreUpdateForm(forms.ModelForm):
        	
        	class Meta:
        		model = IceCreamStore
        	
        	def __init__(self, *args, **kwargs):
        		# 필드 오버로드가 이루어지기 이전에
        		# 원래의 __init__ 메서드 호출
        		super(IceCreamStoreUpdateForm, self).__init__(*args, **kwargs)
        	
        		self.fields['phone'].required = True
```
- 장고 폼도 결국 클래스!! 상속을 이용하여 코드를 줄이자
```python
# stores/forms.py
from django import forms

from .models import IceCreamStore

class IceCreamStoreCreateForm(forms.ModelForm):

  class Meta:
    model = IceCreamStore
    fields = ("title, )
        
              
class IceCreamStoreUpdateForm(IceCreamStoreCreateForm):
        	
	class Meta:
		model = IceCreamStore
		fields = ('title', 'phone', )
        	
    def __init__(self, *args, **kwargs):
    	super(IceCreamStoreUpdateForm, self).__init__(*args, **kwargs)
        	
      self.fields['phone'].required = True
```
### TIP: `Meta Class` 에서 `fields`를 이용하는 것은 좋지만, `exclude` 는 이용하지 말자!! 어떤 필드를 포함하는지 명확하게 명시해주는 것이 좋다.

- 폼 클래스를 이용하여 뷰를 만들어보자
```python
# stores/views.py
from django.views.generic import CreateView, UpdateView

from .forms import IceCreamStoreCreateForm
from .forms import IceCreamStoreUpdateForm
from .models import IceCreamStore

class IceCreamCreateView(CreateView):
  model = IceCreamStore
  form_class = IceCreamStoreCreateForm

class IceCreamUpdateView(UpdateView):
  model = IceCreamStore
  form_class = IceCreamStoreUpdateForm
```
## 12.5 패턴 5: 재사용 가능한 검색 믹스인 뷰

- 뷰에 간단한 검색 믹스인을 만들어보자!
- 믹스인 추가
```python    
        # core/views.py
        class TitleSearchMixin(object):
        
        def get_queryset(self):
        	# 부모의 get_queryset으로부터 queryset 가져오기
        	queryset = super(TitleSearchMixin, self).get_queryset()
        
        	# q라는 GET파라미터 가져오기
        	q = self.request.GET.get("q")
        	if q:
        		# 필터된 쿼리셋 반환
        		return queryset.filtert(title__icontains=q)
        	# q가 지정되지 않았으면 그냥 queryset 반환
        	return queryset
```
- 뷰에 믹스인 추가
```python    
        # add to flavors/views.py
        from django.views.generic import ListView
        
        from core.views import TitleSearchMixin
        from .models import Flavor
        
        class FlavorListView(TitleSearchMixin, ListView):
        	model = Flavor
```
- HTML에서 호출 시
```html    
        <!-- stores/store_list.html 템플릿 안의 폼 -->
    
        <form action="" method="GET">
        	<input type="text" name="q" />
        	<button type="submit">search</button>
    </form>
```
- **믹스인 코드를 재사용하는 건 좋은 방법이지만 단일 클래스에서 너무 많은 믹스인을 이용하면 코드 유지보수가 매우 어려줘지므로 코드는 단순하게 유지해야한다!**

## 12.6 요약

- `ModelForm`과 클래스 기반 뷰, 기본 유효성 검사기부터 커스텀 유효성 검사기 사용
- 클린 메서드들을 오버라이딩해보기
- 두 개의 뷰에 해당하는 폼을 하나의 모델에 연동하기
- 각 다른 앱에 같은 폼을 이용하기 위한 믹스인 생성하기