#  FragmentDialog

DialogFragment 继承于 Fragemnt，比普通的 Dialog 多了很多好处，比如当屏幕旋转时，一般的 dialog 会丢失状态，而 DialogFragmetn 则不会

---
## 1 FragmentDialog 的使用

使用 AppCompatDialogFragement 需要实现以下方法

```
      public class Dialog extends AppCompatDialogFragment {
    
        private EditText mNameEt;
        private EditText mPwdEt;
        private Button mSubmitBtnl;

        /**
            这里可以设置Dialog的一些属性
        */
        public Dialog() {
            setStyle(DialogFragment.STYLE_NO_TITLE, R.style.full);
        }
    
        /**
        用于提供dialog的内容布局
        */
        @Nullable
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
            return inflater.inflate(R.layout.dialog_fragment, container, false);
        }
    
        @Override
        public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
            super.onViewCreated(view, savedInstanceState);
            mNameEt = (EditText) getView().findViewById(R.id.id_dialog_fragment_name_et);
            mPwdEt = (EditText) getView().findViewById(R.id.id_dialog_fragment_pwd_et);
            mSubmitBtnl = (Button) getView().findViewById(R.id.id_dialog_submit_btn);
        }
    
        /**
        用于提供一个dialog，可以在这里设置dialog的属性
        */
        @Override
        public android.app.Dialog onCreateDialog(Bundle savedInstanceState) {
            AppCompatDialog dialog = new AppCompatDialog(getActivity(), getTheme());
            Window dialogWindow = dialog.getWindow();
            WindowManager.LayoutParams lp = dialogWindow.getAttributes();
            dialogWindow.setGravity(Gravity.CENTER);
            lp.width = WindowManager.LayoutParams.MATCH_PARENT;
            lp.height = WindowManager.LayoutParams.MATCH_PARENT;
            dialogWindow.setWindowAnimations(R.style.dialog_anim);
            dialogWindow.setAttributes(lp);
            return dialog;
        }
```

onCreateDialog用于返回一个Dialog，默认已经实现，而onCreateView中返回的视图是作为dialog的contentView。

需要注意的地方：

1. window的属性设置使用`supportRequestWindowFeature()`。
2. theme最好继承`AlertDialog.AppCompat`之类的。

---
## 2 常用属性

```
             /* 
             * 获取的窗口对象及参数对象以修改对话框的布局设置,
             * 可以直接调用getWindow(),表示获得这个Activity的Window
             * 对象,这样这可以以同样的方式改变这个Activity的属性.
             */
            Window dialogWindow = dialog.getWindow();
            WindowManager.LayoutParams lp = dialogWindow.getAttributes();
            dialogWindow.setGravity(Gravity.LEFT | Gravity.TOP);
            /*
             * lp.x与lp.y表示相对于原始位置的偏移.
             * 当参数值包含Gravity.LEFT时,对话框出现在左边,所以lp.x就表示相对左边的偏移,负值忽略.
             * 当参数值包含Gravity.RIGHT时,对话框出现在右边,所以lp.x就表示相对右边的偏移,负值忽略.
             * 当参数值包含Gravity.TOP时,对话框出现在上边,所以lp.y就表示相对上边的偏移,负值忽略.
             * 当参数值包含Gravity.BOTTOM时,对话框出现在下边,所以lp.y就表示相对下边的偏移,负值忽略.
             * 当参数值包含Gravity.CENTER_HORIZONTAL时
             * ,对话框水平居中,所以lp.x就表示在水平居中的位置移动lp.x像素,正值向右移动,负值向左移动.
             * 当参数值包含Gravity.CENTER_VERTICAL时
             * ,对话框垂直居中,所以lp.y就表示在垂直居中的位置移动lp.y像素,正值向右移动,负值向左移动.
             * gravity的默认值为Gravity.CENTER,即Gravity.CENTER_HORIZONTAL |
             * Gravity.CENTER_VERTICAL.
             * 
             * 本来setGravity的参数值为Gravity.LEFT | Gravity.TOP时对话框应出现在程序的左上角,但在
             * 我手机上测试时发现距左边与上边都有一小段距离,而且垂直坐标把程序标题栏也计算在内了,
             * Gravity.LEFT, Gravity.TOP, Gravity.BOTTOM与Gravity.RIGHT都是如此,据边界有一小段距离
             */
            lp.x = 100; // 新位置X坐标
            lp.y = 100; // 新位置Y坐标
            lp.width = 300; // 宽度
            lp.height = 300; // 高度
            lp.alpha = 0.7f; // 透明度
            // 当Window的Attributes改变时系统会调用此函数,可以直接调用以应用上面对窗口参数的更改,也可以用setAttributes
            // dialog.onWindowAttributesChanged(lp);
            dialogWindow.setAttributes(lp);
```

---
## 3 DialogFragment与Fragment的通讯

方式：

    1： dialog.setTargetFragment(ContentFragment.this, REQUEST_EVALUATE);
    2： 手动 onActivityResult

案例：一个Fragment启动一个DialogFragment，在 DialogFragment操作一些数据，然后返回给 Fragment；在Fragment中我们这样写，

```
    EvaluateDialog dialog = new EvaluateDialog();  
    //注意setTargetFragment  
    dialog.setTargetFragment(ContentFragment.this, REQUEST_EVALUATE);  
    dialog.show(getFragmentManager(), EVALUATE_DIALOG);  

    //接收返回回来的数据  
    @Override  
    public void onActivityResult(int requestCode, int resultCode, Intent data)  
    {  
        super.onActivityResult(requestCode, resultCode, data);  
        
        if (requestCode == REQUEST_EVALUATE)  
        {  
            String evaluate = data  
                    .getStringExtra(EvaluateDialog.RESPONSE_EVALUATE);  
            Toast.makeText(getActivity(), evaluate, Toast.LENGTH_SHORT).show();  
            Intent intent = new Intent();  
            intent.putExtra(RESPONSE, evaluate);  
            getActivity().setResult(Activity.REQUEST_OK, intent);  
        }  
    
    }  
```

在 DialogFragment 中可以直接setResult，直接调用原来Fragment的onActivityResult的方法

```
    // 设置返回数据  
    protected void setResult(int which){  
        // 判断是否设置了targetFragment  
        if (getTargetFragment() == null)  
            return;  
        
        Intent intent = new Intent();  
        intent.putExtra(RESPONSE_EVALUATE, mEvaluteVals[which]);  
        getTargetFragment().onActivityResult(ContentFragment.REQUEST_EVALUATE,  
                Activity.RESULT_OK, intent);  
        }  
    }  
```