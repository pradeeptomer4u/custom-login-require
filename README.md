# custom-login-require
custom login required function or class using session key 

from django.utils.decorators import method_decorator
from .models import CustomUser

########### Views ###################

def login_user(request, username, user_type):
    request.session['logged_in'] = True
    request.session['username'] = username
    request.session['user_type'] = user_type


def login(request):
    if request.POST:
        username = str(request.POST['username'])
        password = str(request.POST['password'])
        try:
            users_ = CustomUser.objects.get(username=username, password=password)
            user_type = str(users_.user_type)
            data = {"user_type": user_type}
            login_user(request, username,user_type)

            if request.session['user_type'] in ['admin']:
                return redirect('/home')

        except DoesNotExist:
            return render(request,'login.html', {'error':'Invalid Credentails'})
        except KeyError:
            return render(request,'login.html', {})
    elif request.session.get('logged_in') is None or not True:
        return render(request,'login.html', {})
    else:
        return render(request,'login.html', {})

############ Login Required function and class using session key ################


def login_required():
    session_key = 'logged_in'
    fail_redirect_to = '/login'

    def _login_required(view_func):
        @wraps(view_func)
        def __login_required(request, *args, **kwargs):
            try:
                session = request.session.get(session_key)
                if session is None:
                    return redirect(fail_redirect_to)
                if session is not True:
                    return redirect(fail_redirect_to)
            except Exception as e:
                return redirect(fail_redirect_to)
            else:
                return view_func(request, *args, **kwargs)
        return __login_required

    return _login_required


class LoginRequiredMixin(object):
    @method_decorator(login_required())
    def dispatch(self, request, *args, **kwargs):
        return super(LoginRequiredMixin, self).dispatch(request, *args, **kwargs)


############### using custom login require class #########################

class ListExampleView(LoginRequiredMixin,ListView):

    modal = Example
    template_name = 'example_list.html'

    def get_context_data(self, *args, **kwargs):
        context = super(ListExampleView, self).get_context_data(*args, **kwargs)
        context['kwargs'] = self.kwargs
        return context

    def get_queryset(self):
        return Example.objects.filter(id=self.kwargs['id'])




