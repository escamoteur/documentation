# Invoking Commands

## InvokeCommand

At times it can be convenient to execute a command in response to some observable that isn't perhaps tied to a user interaction. For example, a feature that automatically saves the current document by executing a `SaveCommand` every 5 minutes. The `InvokeCommand` extension makes it easy to achieve this:

```cs
var interval = TimeSpan.FromMinutes(5);
Observable
    .Timer(interval, interval)
    .InvokeCommand(this.ViewModel, x => x.SaveCommand);
```

> **Hint** `InvokeCommand` respects the command's executability. That is, if the command's `CanExecute` method returns `false`, `InvokeCommand` will not execute the command when the source observable ticks.

## Execute

The other way to execute a command from other contexts than an observable is calling the command's `Execute` method with the option to pass parameters to the command see [Command Parameters](command-parameters.md)


>**Warning**
>If you are calling `Execute` or `InvokeCommand `on a asynchronous `ReactiveCommand` you have to do this from within an asynchronous context. Otherwise your command will be executed synchronously with the consequence that `IsExecuting` of the command doesn't emit any values and any subscriptions of that command won't be called.