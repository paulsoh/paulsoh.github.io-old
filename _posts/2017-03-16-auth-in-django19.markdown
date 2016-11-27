---
layout: post
title:  "[DJANGO 1.9] 웹 요청(WEB REQUEST)에서의 인증(AUTHENTICATION)"
date:   2016-03-16
---

<p class="intro">
왜 알아야하는가?
</p>

웹 프로젝트를 진행하다보면 필연적으로 마주하게 되는 고민 중 하나는:

<blockquote>
로그인한 유저와 로그인하지 않은 유저간의 웹 페이지에 대한 접근 가능 여부를 다르게 적용할수는 없을까?
</blockquote>

간단한 게시판을 봐도 게시글의 제목들이 나와있는 목록은 누구에게나 보이지만 각 게시물의 상세 내용들은 로그인을 한 사용자에 대해서만 접근을 허락하는 것이 일반적이다.

Django에서는 간단한 방법으로 유저의 상태(Authenticate이 되었는지 여부)에 따라서 뷰에 대한 접근성을 편리하게 조정할 수 있다.

다음은 Django 1.9 Documentation 중 이 부분에 대한 설명을 정리해 본 것이다.

장고에서 제공하는 기본 User 모델을 사용한다고 가정하겠다. (Django에서는 회원 가입 및 로그인/로그아웃의 기능도 매우 쉽게 구현할 수 있도록 기능들을 제공하지만 이 부분에 대한 설명은 별도 범위이기 때문에 이 글에서는 다루지 않도록 한다.)

우선 가장 간단하게 구현하는 방법은 다음과 같다.

{% highlight python %}
from django.conf import settings
from django.shortcuts import redirect

def my_view(request):
    if not request.user.is_authenticated():
        return redirect('%s?next=%s' % (settings.LOGIN_URL, request.path))
    # ...
{% endhighlight %}

원리는 간단하다. request.user.is_authenticated(): 의 boolean 값을 보고 뷰 내에서 분기 처리를 하는 것이다.

저렇게 될 수 있는 이유는 로그인 과정에서 user가 authenticate이 되었으면, 해당 세션(session) 내에서는 request.user에 대한 값이 계속 유지가 되게 된다. request에 대한 세션이 종료되지 않는 이상 웹 프로젝트 내의 어떤 뷰를 보더라도 항상 request.user.is_authenticated():는 true 값을 갖게 되고 그렇기 때문에 위의 코드와 같이 분기처리를 하면 되는 것이다.

가장 간단한 방법이다. 이 방법을 사용하기 위해서는 모든 뷰에 들어가서 분기 처리를 해주어야 한다.

이것 보단 나은 방법은 있겠지…

{% highlight python %}
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...
{% endhighlight %}

위에서는 decorator를 이용하여 로그인이 된 상태에서만 뷰에 대한 접근을 허용하는 것이다. 위의 경우보다는 덜 직관적이지만 훨씬 더 간단하게 구현될 수 있다는 것을 볼 수 있다. 뿐만 아니라
@login_required(redirect_field_name='my_redirect_field'

@login_required(login_url='/accounts/login/')
와 같이 로그인이 안된 상태로 해당 뷰에 접근을 했을 때 뷰를 redirect를 해줄 수 있다. 이 경우에는 로그인을 할 수 있는 뷰로 포워딩을 해주는 것이 일반적이다.

이 방법 정도라면 상당히 괜찮은 방법이라는 생각이 든다. 뷰 함수 내부를 건드리지 않고 단순히 decorator만으로 wrapping을 해주는 것이기 때문에 뭔가 기존의 코드와 충돌이 나는 것이 거의 없을 것이라고 생각된다. 하지만 위의 방법도 한계(?)가 있는데 그것은 바로

Function based view에서만 사용할 수 있다는 것이다.

Django에서는 그렇기 때문에 Class based view에서도 위와 같은 구현을 할 수 있도록 믹스인을 제공한다. 아래의 예를 보자.

{% highlight python %}
from django.contrib.auth.mixins import LoginRequiredMixin

class MyView(LoginRequiredMixin, View):
    login_url = '/login/'
    redirect_field_name = 'redirect_to'
{% endhighlight %}

뷰에 단순히 LoginRequiredMixin 를 넣어주게 되면 그 뷰는 위의 경우와 마찬가지로 로그인이 되었을 때에만 접근이 가능하게 된다. 그리고 해당 믹스인을 사용할 때에는 login_url와 redirect_field_name이라는 파라미터값도 설정을 할 수 있게 된다. (이것은 위와 마찬가지)

이상으로 Django에서 Function based view와 Class based view로 구현한 뷰(view)들의 인증 방법에 대한 설명을 마치도록 하겠다. Django에서는 늘 겪는 고민이지만 FBV와 CBV의 갈림길에서 모든 경우에 대한 방법을 숙지함으로써 robust한 개발자가 될 수 있도록 하자.

추가 내용 – CBV에서 method_decorator를 쓰는 방법 (Django documentation에는 나와있지 않지만 이 방법으로도 가능하다.)


{% highlight python %}
from django.contrib.auth.mixins import LoginRequiredMixin

from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator

@method_decorator(login_required, name='dispatch')
class MyView(View):
  ...
{% endhighlight %}
