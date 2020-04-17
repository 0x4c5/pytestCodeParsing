# ǰ��
###### ���˽���pluggy֮�����ǻ���Ҫ���˽�Щ֪ʶ��Ϊ��������߼���׼��
##### ����׾�����д����λָ����
###### _������ҵ����¶����а����������������Ľ���ָ����Star������õ�壬�������㣬��ʤ�м���_
</br>
</br>
</br>

# pytest-pluggy����hook�����߼�

### ǰ������˲���hook�ĵ����߼������ǻ��и�hook_executeû���ϣ������������ķ���pm.hook.calculate(a=2, b=3)��ִ�й���
### ÿ�����ǵ���pm.hook.xxx(**kwargs)��ʱ��ʵ�����ǵ�����_HookCaller�����__call__����
```python
    def __call__(self, *args, **kwargs):
        if args:
            raise TypeError("hook calling supports only keyword arguments")
        assert not self.is_historic()
        if self.spec and self.spec.argnames:
            notincall = (
                set(self.spec.argnames) - set(["__multicall__"]) - set(kwargs.keys())
            )
            if notincall:
                warnings.warn(
                    "Argument(s) {} which are declared in the hookspec "
                    "can not be found in this hook call".format(tuple(notincall)),
                    stacklevel=2,
                )
        return self._hookexec(self, self.get_hookimpls(), kwargs)
```
  * **��`__call__`�Ĵ�����Կ��������߼������һ�е�`self._hookexec`�����ǿ��Է�������_HookCaller��һ������**

<br/><br/>

#### ����˳��`self._hookexec`�����ҵ�_HookCaller�Ĺ��캯��
```
    def __init__(self, name, hook_execute, specmodule_or_class=None, spec_opts=None):
        self.name = name
        self._wrappers = []
        self._nonwrappers = []
        self._hookexec = hook_execute
```
  * **���Է��֣���������ǹ���`_HookCaller`����ʱ����ĵķ����������������ң�����`hook_execute`�Ǵ����ﴫ������**

<br/><br/>

#### ���Ƿ���`hook_execute`�Ǵ�PluginManager���register������ʵ����_HookCallerʱ���ݵ�
```python
    if hook is None:
        hook = _HookCaller(name, self._hookexec)
```

<br/><br/>

#### ����PluginManager���һ���������ҵ��÷���������ֻ��һ����װ������������
```python
    def _hookexec(self, hook, methods, kwargs):
        # called from all hookcaller instances.
        # enable_tracing will set its own wrapping function at self._inner_hookexec
        return self._inner_hookexec(hook, methods, kwargs)
```

<br/><br/>

#### ���`hook_execute`��ʵ��hook.multicall������Ҳ����multicall�����ķ�װ
```python
  self._inner_hookexec = lambda hook, methods, kwargs: hook.multicall(
      methods,
      kwargs,
      firstresult=hook.spec.opts.get("firstresult") if hook.spec else False,
      )
```

<br/><br/>

#### �����ң�û�뵽�ص���`_HookCaller`���`self.multicall`
```python
  class _HookCaller(object):
      def __init__(self, name, hook_execute, specmodule_or_class=None, spec_opts=None):
          self.name = name
          self._wrappers = []
          self._nonwrappers = []
          self._hookexec = hook_execute
          self.argnames = None
          self.kwargnames = None
          self.multicall = _multicall
```

<br/><br/>

#### ����ҵ���`_HookCaller`���`_multicall`�������ֶο�һ��
```python
  def _multicall(hook_impls, caller_kwargs, firstresult=False):
      """Execute a call into multiple python functions/methods and return the
      result(s).

      ``caller_kwargs`` comes from _HookCaller.__call__().
      """
      __tracebackhide__ = True
      results = []
      excinfo = None
      try:  # run impl and wrapper setup functions in a loop
          teardowns = []
          try:
              for hook_impl in reversed(hook_impls):
                  try:
                      args = [caller_kwargs[argname] for argname in hook_impl.argnames]
                  except KeyError:
                      for argname in hook_impl.argnames:
                          if argname not in caller_kwargs:
                              raise HookCallError(
                                  "hook call must provide argument %r" % (argname,)
                              )
```
  * **��תhook_impls,pluginִ�д�listĩβ��ʼ����Ҳ��Ϊʲô��ע���plugin��ִ�е�ԭ��**
  * **�����������hook_implsʹ�õĲ���û����hookspec��Ԥ�ȶ���Ļ����׳�HookCallError**

<br/><br/>

```python
  if hook_impl.hookwrapper:
      try:
          gen = hook_impl.function(*args)
          next(gen)  # first yield
          teardowns.append(gen)
      except StopIteration:
          _raise_wrapfail(gen, "did not yield")
  else:
       res = hook_impl.function(*args)
       if res is not None:
           results.append(res)
           if firstresult:  # halt further impl calls
               break
```
  * **hookwrapper**
    * **�������pluginʱhookwrapper����ΪTrueʱ,����ִ��plugin��yield֮ǰ�Ĵ��룬������pluginִ����ż���ִ��yield����ĵĲ��֡�**
    * **`gen = hook_impl.function(*args)`ִ��plugin function��yieldǰ�Ĳ��֣�Ȼ��ͣ��**
    * **`next(gen)`������plugin function��yield����Ĳ���**
    * **��gen�õ���generator�ӵ�teardown�У����ں�����callback**
  * **nonwrapper**
    * **ֱ�ӵ���plugin function**
    * **��ִ�н�����浽result��**
   
<br/><br/>

```python
    finally:
        if firstresult:  # first result hooks return a single value
            outcome = _Result(results[0] if results else None, excinfo)
        else:
            outcome = _Result(results, excinfo)

        # run all wrapper post-yield blocks
        for gen in reversed(teardowns):
            try:
                gen.send(outcome)
                _raise_wrapfail(gen, "has second yield")
            except StopIteration:
                pass

        return outcome.get_result()
```
  * **��һ��hook������pluginʵ�ֶ�ִ��������е�ִ�н����Ҫ��_Result���װ**
  * **����Դ���ǰ�����teardowns�е�generator��������ִ����Щ��ִ����һ��ġ�hookwrapper**
  * **����nowrapper�Ľ�����ݸ�hookwrapper�ĺ�벿��**
  * **���ﲻ������һ��hookwrapperʹ������yield���ᵼ�����ⲿ�׳��쳣��ֹ�߼� `_raise_wrapfail(gen, "has second yield")`->`RuntimeError`**
    * **������ϸ����**
    * **��gen.send(outcome)��������ݻ�hookwrapper��hookwrapper���������ִ�У�û��yield��ʱ��ᰴִ����plugin���߼���**
    * **���ٴ�����yield��ʱ�򣬻��ٴ����������ص�λ�þ����������������ִ��`_raise_wrapfail(gen, "has second yield")`���׳������ˡ�**
  * **���outcome�Ľ�����ظ����÷�`hook_execute`->`self._hookexec`->`pm.hook.xxx(**kwargs)`**

<br/><br/>  

### �����ٿ���_Result�Ĵ��룬�ȿ����캯�������ǰ�ִ�н����ִ���쳣��Ϣ��װ����_Result
```python
  class _Result(object):
      def __init__(self, result, excinfo):
          self._result = result
          self._excinfo = excinfo
```

<br/><br/>

### ������Ҫ����get_result()
```python
  def get_result(self):
      """Get the result(s) for this hook call.

      If the hook was marked as a ``firstresult`` only a single value
      will be returned otherwise a list of results.
        """
      __tracebackhide__ = True
      if self._excinfo is None:
          return self._result
      else:
          ex = self._excinfo
          if _py3:
              raise ex[1].with_traceback(ex[2])
          _reraise(*ex)  # noqa
```
  * **�ж�`hook call`�������з��쳣��**
  * **���쳣����·���`hook call`��ִ�н����**
  * **���쳣����£��׳������쳣��`BaseException`�Ļ��ݽ��**
  
<br/><br/><br/> 

## �ܽ�
#### �϶�������ְ���һ��������pluggyԴ�����ڰ����ˣ���ǰ��˵��������Դ�������Լ�д�Ķ�����������������������
