# Handling uncaught ReactiveUI Exceptions

jcmm33 [12:00 PM] 
Not wishing to state the obvious, but for ReactiveObject derived items then unless you specify a ThrownException handler its channeled through RxApp.DefaultExceptionHandler, which you can change.

It won’t trap all Reactive Extensions exceptions (there is a different way to do that) but for ReactiveObject based ones i.e. those with a ThrownException property it channels through RxApp.DefaultExceptionHandler if no subscription for ThrownException is present

```
public class MyRxHandler : IObserver<Exception>
{
    public void OnNext(Exception value)
    {
        throw value;
    }

    public void OnError(Exception error)
    {
        throw error;
    }

    public void OnCompleted()
    {
        throw new NotImplementedException();
    }
}

RxApp.DefaultExceptionHandler = new MyRxHandler();
