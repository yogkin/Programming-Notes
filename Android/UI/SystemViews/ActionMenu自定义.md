# 添加自定义菜单

在使用 Toolbar时，可能需要在Toolbar上添加几个MenuItem，这是笔记简单的，但是如果需要实现自定义的菜单的呢？

如下图所示：

![](index_files/1cea7b7b-1810-4be5-bff0-7b39fff2185b.png)

这是我们需要使用`actionProviderClass`了。

定义菜单如下：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <menu xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:app="http://schemas.android.com/apk/res-auto">
        
        <item
            android:id="@+id/common_menu_cart_enter"
            android:icon="@drawable/common_shopping_cart"
            android:title="@string/shopping_cart_tittle"
            app:actionProviderClass="com.mondial.fashiontech.modules.common.CartMenuItemProvider"
            app:showAsAction="always"/>
    
    </menu>
```

注意`actionProviderClass`和`showAsAction`的命名空间使用的是app，而不是android，因为我们使用的是support包中的相关类。

然后实现我们的`CartMenuItemProvider`:


```java
    public class CartMenuItemProvider extends ActionProvider {
    
        public CartMenuItemProvider(Context context) {
            super(context);
        }

        @Override
        public View onCreateActionView() {
        
            @SuppressLint("InflateParams")
            View menuItem = LayoutInflater.from(getContext()).inflate(R.layout.mall_layout_cart_count, null, false);
           int size = getContext().getResources().getDimensionPixelSize( android.support.design.R.dimen.abc_action_bar_default_height_material);
    
            ViewGroup.LayoutParams layoutParams = new ViewGroup.LayoutParams(size, size);
            menuItem.setLayoutParams(layoutParams);
            return menuItem;
        }
    
    }
```

只需要在onCreateActionView中返回我们自定的ActionView即可。
