# Fragment状态丢失问题

---
## 1 问题

在使用Fragment的时可能会遇到这样的问题：

```
    java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
    at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1341)
    at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1352)
    at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:595)
    at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:574)
```

意思是说不能在Activity的onSaveInstanceState方法执行之后去commit一个FragmentTransaction，原因是**Fragment的状态保存依托于其所关联的Activity，而Activity就是通过onSaveInstanceState方法来保存状态的，所以在onSaveInstanceState方法执行之后去去commit一个FragmentTransaction的话有可能造成Fragment的状态无法保存，所以Android直接抛出了异常。 **



---
## 2 异常何时产生

现象：

- 相对老的设备上这个异常可能会抛出的没有那么频繁
- 如果你的应用使用了support library的话，可能会比使用官方框架更容易崩溃一点

**在Honeycomb这个Android版本中，Activity的生命周期做了相当的调整**

- 在Honeycomb之前的版本中，Activity在pause以前都是不能被杀掉的，所以onSaveInstanceState() 就会在onPause()之前被立刻调用
- 从Honeycomb开始，Activity变成了在stop之前不能被杀掉。所以自然onSaveInstanceState()就会变成在onStop()之前被立刻调用

||Honeycomb之前 |Honeycomb之后 |
| --- | --- | --- |
| 在`onPause()`后Activity能否被杀死 | NO | NO |
| 在`onStop()`后Activity能否被杀死 | YES | NO |
| `onSaveInstanceState(Bundle)` 保证在哪个方法前调用 | `onPause()` | `onStop()` |

support library需要根据系统版本的不同, 来调整自己的行为：

|| Honeycomb之前 | Honeycomb之后|
| --- | --- | --- |
| `commit()` before `onPause()` | OK | OK |
| `commit()` between `onPause()` and `onStop()` | STATE LOSS | OK |
| `commit()` after `onStop()` | EXCEPTION | EXCEPTION |





---
## 3 如何避免异常

**谨慎的在Activity的生命周期方法中调用transaction的commit方法**

- 一般在onCreate中去commit transaction
- FragmentActivity#onResumeFragments()或者Activity#onPostResume()中去commit也不会导致状态丢失。
- 避免异步回调后commit transaction
- 在onActivityResult方法后commit transaction需要特别注意,[参考](http://stackoverflow.com/questions/16265733/failure-delivering-result-onactivityforresult)
- 不得已的情况下，使用commitAllowingStateLoss()


在onActivityResult方法后commit transaction最好使用下面方式。

```
    private boolean mReturningWithResult = false;
     
    @Override 
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        mReturningWithResult = true;
    } 
     
    @Override 
    protected void onPostResume() { 
        super.onPostResume(); 
        if (mReturningWithResult) {
            // Commit your transactions here. 
        } 
        // Reset the boolean flag back to false for next time. 
        mReturningWithResult = false;
    } 
```

---
## 4 参考

- [fragment-transactions-and-activity-state-loss译文](http://jaredlam.github.io/blog/2015/12/23/android-fragment-transactions-and-activity-state-loss-yi/)
- [原文](http://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html)
