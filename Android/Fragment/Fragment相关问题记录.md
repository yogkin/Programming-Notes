---
## 1 Fragment 的 onActivityResult 的问题

多层嵌套Fragment， onActivityResult 问题这样解决。 在最新的`Support v4-23.2`中已经对此问题进行修复。

---
## 2 Fragment 对 Menu 菜单操作

- 首先在Fragment的onCreate方法中设置      setHasOptionsMenu(true);

然后实现下面方法，需要注意的是对于菜单的处理，Activity的优先级高于Fragment

```
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        inflater.inflate(R.menu.menu_frag_rep, menu);
        super.onCreateOptionsMenu(menu, inflater);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        Toast.makeText(getActivity(), "id = " + item.getTitle(), Toast.LENGTH_SHORT).show();
        return super.onOptionsItemSelected(item);
    }
```

---
## 3 Fragement通讯

- 对于Fragement与Activity通讯，建议使用接口，而不是getActivity强转，因为这样fragment的复用性太差，代码也有耦合。
- 对于Fragement间的通讯，可是使用Fragmen的`setTargetFragment`方法，或者通过Activity实现。

```
//https://stackoverflow.com/questions/44977682/how-to-set-target-fragment
public class DialogFragmentB extends DialogFragment{ 
   ... 
   public Dialog onCreateDialog (Bundle b){
   // here i create two dialogs, first dialogA and then it calls dialogB 
   // finally dialogB has to return his datas and datas from dialogA in fragmentA 
      ... 
      public void onClick(View v){
         ... 
         FragmentManager fm = getFragmentManager(); 
         // these two code's lines resolved my headache -------------- 
         Fragment ft = fm.findFragmentByTag(FragmentA.FRAGMENT_A_TAG); 
         ft.setTargetFragment(ft, FragmentA.CODE_REQUEST); 
         // ---------------------------------------------------------- 
         dialogB.dismiss(); 
         sendResult(Activity.RESULT_OK, myData);
      } 
      return dialogB; 
   } 
   // And here myData goes in onActivityResult in fragmentA 
    private void sendResult(int resultCode, MyData myData){
        if(getTargetFragment() == null){ 
            return; 
        } 
        Intent intent = new Intent();
        intent.putExtra(EXTRA_DATE_2, myData);
        getTargetFragment().onActivityResult(getTargetRequestCode(), resultCode, intent);
    } 
} 
```

---
## 4 Fragment事务应该立即执行还是加入到事物队列

popBackStackImmediate还是popBackStack？

- 如果Activity中存在多个Fragment，当需要返回的1一个Fragmetn后立即加入新的Fragment。建议使用popBackStackImmediate，不然后面的加入操作会影响前面的出栈操作，可能造成重影。
- 另外注意，新添加的commitNow()方法有个限制，无法把当前提交的 Fragment 添加到堆