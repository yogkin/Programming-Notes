## 密码可见

- [在密码输入框中添加显示明文功能](http://www.wangchenlong.org/2016/03/21/1603/211-show-password-reveal/)
- [HideReturnsTransformationMethod](http://developer.android.com/reference/android/text/method/HideReturnsTransformationMethod.html)
- [PasswordTransformationMethod](http://developer.android.com/reference/android/text/method/PasswordTransformationMethod.html)


方式1：

```java
    if (isChecked) {
        // 显示密码
        password_edit.setInputType(InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD);
        }
    else {
        // 隐藏密码
        password_edit.setInputType(InputType.TYPE_CLASS_TEXT | InputType.TYPE_TEXT_VARIATION_PASSWORD);
```

方式2：

```java
    public class MainActivity extends Activity {
        private Button mSwitchButton;
        private EditText mPasswordEditText;
        private boolean isHidden=true;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main);
            init();
        }
        private void init(){
            mSwitchButton=(Button) findViewById(R.id.button);
            mPasswordEditText=(EditText) findViewById(R.id.editText);
            mSwitchButton.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    if (isHidden) {
                        //设置EditText文本为可见的
                        mPasswordEditText.setTransformationMethod(HideReturnsTransformationMethod.getInstance());
                    } else {
                        //设置EditText文本为隐藏的
                        mPasswordEditText.setTransformationMethod(PasswordTransformationMethod.getInstance());
                    }
                    isHidden = !isHidden;
                    mPasswordEditText.postInvalidate();
                    //切换后将EditText光标置于末尾
                    CharSequence charSequence = mPasswordEditText.getText();
                    if (charSequence instanceof Spannable) {
                        Spannable spanText = (Spannable) charSequence;
                        Selection.setSelection(spanText, charSequence.length());
                    }
     
                }
            });
        }
         
    }
```

## 输入类型

```
    editText.setInputType(InputType.TYPE_CLASS_NUMBER); //输入类型  
    editText.setFilters(new InputFilter[]{new InputFilter.LengthFilter(6)}); //最大输入长度  
    editText.setTransformationMethod(PasswordTransformationMethod.getInstance()); //设置为密码输入框
```

## 弹出软件盘修改回车键的 Action

```
EditTxt.setImeOptions
```

## 修改光标颜色

1：使用反射

```java
     public static void tintCursorDrawable(EditText editText, int color) {
            try {
                Field fCursorDrawableRes = TextView.class.getDeclaredField("mCursorDrawableRes");
                fCursorDrawableRes.setAccessible(true);
                int mCursorDrawableRes = fCursorDrawableRes.getInt(editText);
                Field fEditor = TextView.class.getDeclaredField("mEditor");
                fEditor.setAccessible(true);
                Object editor = fEditor.get(editText);
                Class<?> clazz = editor.getClass();
                Field fCursorDrawable = clazz.getDeclaredField("mCursorDrawable");
                fCursorDrawable.setAccessible(true);
    
                if (mCursorDrawableRes <= 0) {
                    return;
                }
    
                Drawable cursorDrawable = editText.getContext().getResources().getDrawable(mCursorDrawableRes);
                if (cursorDrawable == null) {
                    return;
                }
    
                Drawable tintDrawable  = tintDrawable(cursorDrawable, ColorStateList.valueOf(color));
                Drawable[] drawables = new Drawable[] {tintDrawable, tintDrawable};
                fCursorDrawable.set(editor, drawables);
            } catch (Throwable ignored) {
            }
        }
```


2.使用xml属性(3.0后)

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <shape xmlns:android="http://schemas.android.com/apk/res/android" >
        <size android:width="3dp"  />
        <solid android:color="#0Fddbb"  />
    </shape>

      <EditText
                android:textCursorDrawable="@drawable/edit_curtor"
                android:id="@+id/et2"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
```

## EditText焦点错乱问题

当界面中存在多个EditText时，可能出现一下问题。

你会发现当点击第一个`EditText`时，第二个`EditText`会有光标闪以下，或者点击第二个或者之后的`EditText`,第一个`EditText`会有光标闪一下。通过Log你会发现从第二次点击`EditText`起，每次点击`EditText`都会先触发一次失去焦点，再触发一次获取焦点。由此可以推断当存在多个`EditText`时，一个`EditText`失去焦点会触发另一个获取焦点。

如何避免另一个EditText获取焦点?

解决这个问题的做法是在同一个XML中将一个不会有响应的控件(如`TextView`)设置如下属性：

```xml
android:focusable="true"
android:focusableInTouchMode="true"
```

问题完美解决。



