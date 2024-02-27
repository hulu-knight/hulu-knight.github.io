---
layout: post
title: android 隐藏底部导航栏
tags: java
author: hulu-knight
date: 2024-02-27 11:24 +0800
---



android 隐藏底部导航栏的工具类，及一些踩的坑

## 工具类

```java
public class NavigationUtils {

    public static void setNavBarVisibility(@NonNull final Activity activity, boolean isVisible) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT) return;
        if (activity == null || activity.isFinishing() || activity.isDestroyed()){
            return;
        }
        setNavBarVisibility(activity, activity.getWindow(), isVisible);

    }

    public static void setNavBarVisibility(Context context, Window window, boolean isVisible) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT || window == null) return;
        final ViewGroup decorView = (ViewGroup) window.getDecorView();
        for (int i = 0, count = decorView.getChildCount(); i < count; i++) {
            final View child = decorView.getChildAt(i);
            final int id = child.getId();
            if (id != View.NO_ID) {
                String resourceEntryName = getResNameById(context, id);
                if ("navigationBarBackground".equals(resourceEntryName)) {
                    child.setVisibility(isVisible ? View.VISIBLE : View.INVISIBLE);
                }
            }
        }
        final int uiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;
        if (isVisible) {
            decorView.setSystemUiVisibility(decorView.getSystemUiVisibility() & ~uiOptions);
        } else {
            decorView.setSystemUiVisibility(decorView.getSystemUiVisibility() | uiOptions);
        }
    }

    /**
     * 在 popupwindow show之后调用, 务必设置 setFocusable(false)
     */
    public static void setNavBarVisibility(Context context, PopupWindow popupWindow, boolean isVisible) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT || popupWindow == null) return;
        final ViewGroup decorView = (ViewGroup) popupWindow.getContentView();
        for (int i = 0, count = decorView.getChildCount(); i < count; i++) {
            final View child = decorView.getChildAt(i);
            final int id = child.getId();
            if (id != View.NO_ID) {
                String resourceEntryName = getResNameById(context, id);
                if ("navigationBarBackground".equals(resourceEntryName)) {
                    child.setVisibility(isVisible ? View.VISIBLE : View.INVISIBLE);
                }
            }
        }
        final int uiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;
        if (isVisible) {
            decorView.setSystemUiVisibility(decorView.getSystemUiVisibility() & ~uiOptions);
        } else {
            decorView.setSystemUiVisibility(uiOptions);
        }
        popupWindow.setFocusable(true);
        popupWindow.update();
    }

    public static boolean isNavBarVisible(Activity activity) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT ) return false;
        if (activity == null || activity.isFinishing() || activity.isDestroyed()){
            return false;
        }
        Window window = activity.getWindow();
        boolean isVisible = false;
        ViewGroup decorView = (ViewGroup) window.getDecorView();
        for (int i = 0, count = decorView.getChildCount(); i < count; i++) {
            final View child = decorView.getChildAt(i);
            final int id = child.getId();
            if (id != View.NO_ID) {
                String resourceEntryName = getResNameById(activity, id);
                if ("navigationBarBackground".equals(resourceEntryName)
                        && child.getVisibility() == View.VISIBLE) {
                    isVisible = true;
                    break;
                }
            }
        }
        if (isVisible) {
            // 对于三星手机，android10以下非OneUI2的版本，比如 s8，note8 等设备上，
            // 导航栏显示存在bug："当用户隐藏导航栏时显示输入法的时候导航栏会跟随显示"，会导致隐藏输入法之后判断错误
            // 这个问题在 OneUI 2 & android 10 版本已修复
            if (isSamsung()
                    && Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1
                    && Build.VERSION.SDK_INT < Build.VERSION_CODES.Q) {
                try {
                    return Settings.Global.getInt(activity.getContentResolver(), "navigationbar_hide_bar_enabled") == 0;
                } catch (Exception ignore) {
                }
            }

            int visibility = decorView.getSystemUiVisibility();
            isVisible = (visibility & View.SYSTEM_UI_FLAG_HIDE_NAVIGATION) == 0;
        }

        return isVisible;
    }

    private static Boolean isSamsung() {
        String manufacturer = Build.MANUFACTURER;
        if ("samsung".equalsIgnoreCase(manufacturer)) {
            return true;
        }
        return false;

    }

    private static String getResNameById(Context context, int id) {
        try {
            return context.getResources().getResourceEntryName(id);
        } catch (Exception ignore) {
            return "";
        }
    }
}

```

## 调用时机

### 1. activity

在``onResume()``中调用

```java
@Override
public void onResume() {
    super.onResume();
    NavigationUtils.setNavBarVisibility(this, false);
}
```

### 2. dialog

- 自定义dialog中：在``onstart()``调用

```java
@Override
protected void onStart() {
    super.onStart();
    NavigationUtils.setNavBarVisibility(mContext, false);
}
```

- 普通dialog：在``dialog.show()``方法之后调用

```java
mDialog.show();
NavigationUtils.setNavBarVisibility(mContext, mDialog.getWindow(), false);
```

### 3. PopupWindow

- 普通popupWindow：在``popupWindow.showAsDropDown()``或者``popupWindow.showAtLocation()``之后调用，注意在调用之前确保设置``popupWindow.setFocusable(true);``为``true``

```java
mPopupWindow.setFocusable(true);
mPopupWindow.showAsDropDown(mView, 0, 0);
NavigationUtils.setNavBarVisibility(mContext, mPopupWindow, false);
```

- 自定义PopupWindow：重写``showAtLocation()``或者``showAsDropDown()``

```kotlin
class NavigationPopupWindow @JvmOverloads constructor(
    private val mView: View,
    private val mWidth: Int,
    private val mHeight: Int,
    private val isShowNavigation: Boolean,
    private val mIsOutsideTouchable: Boolean = true,
    ) : PopupWindow(mView, mWidth, mHeight, mIsOutsideTouchable) {

    override fun showAsDropDown(anchor: View?, xoff: Int, yoff: Int, gravity: Int) {
        this.isFocusable = isShowNavigation  // 强制设置focus为false
        super.showAsDropDown(anchor, xoff, yoff, gravity)
        NavigationUtils.setNavBarVisibility(mView.context, this, isShowNavigation)
    }

    override fun showAtLocation(parent: View?, gravity: Int, x: Int, y: Int) {
        this.isFocusable = isShowNavigation
        super.showAtLocation(parent, gravity, x, y)
        NavigationUtils.setNavBarVisibility(mView.context, this, isShowNavigation)

    }
}
```

​			然后调用

```java
NavigationPopupWindow popupWindow = new NavigationPopupWindow(viewDialog, ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT, false);
popupWindow.showAsDropDown(mView, 0, 0);
```

## 踩坑

当我们都设置上了隐藏导航栏之后，启动app，打开一个activity，弹出一个dialog，这都隐藏了导航栏，但是当关闭dialog之后，会发现activity的导航栏又出现了。这就要回到activity的生命周期来看了

``activity onCreate()`` - ``activity onStart()`` - ``activity onResume()`` - 启动弹窗 - ``dialog onStart() ``- 关闭弹窗 - ``dialog onDismiss() ``- ``activity onPause()`` - ``activity onStop() ``- ``activity onDestory()``

而设置隐藏导航栏的代码分别在 ``activity onResume()``，``dialog onStart() ``中，而dialog的关闭导致activity重新获得焦点，但是却没有走``onResume()``自然会展示导航栏。

所以我们只需要重写 activity的``onWindowFocusChanged()``

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        NavigationUtils.setNavBarVisibility(this, false);
    }
}
```

在检测到焦点改变的时候，隐藏导航栏即可。

## 总结

可能有人问如果没有dialog的引用，该如何隐藏导航栏？通常这种情况出现在项目的库文件中开放方法直接展示的dialog，我没有找到很好的解决方法，以前可以通过获取window顶层的dialog，来获取到dialog的引用，但是后面被谷歌废弃了，写在这里仅供记录。
