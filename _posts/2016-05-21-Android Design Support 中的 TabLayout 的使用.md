---
layout: post
title: "Android Design Support 中的 TabLayout 的使用"
keywords: 
description: ""
category: 
tags: 
---

<!--markdown-->Android Material Design控件之一 TabLayout ，在很多app上都有应用。使用起来非常的方便和美观。  

 ![Screenshot_2016-05-21-20-04-54_com.demo.liangddyy.demo_design](http://539go.com/usr/uploads/2016/05-21/Screenshot_2016-05-21-20-04-54_com.demo.liangddyy.demo_design.png) ![Screenshot_2016-05-21-20-04-58_com.demo.liangddyy.demo_design](http://539go.com/usr/uploads/2016/05-21/Screenshot_2016-05-21-20-04-58_com.demo.liangddyy.demo_design.png)  
  
> 添加依赖
  
```  
compile 'com.android.support:design:23.3.0'  
```  
  
> 示例：  
  
添加主布局文件  
  
MainActivity.xml  
  
```  
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="vertical">  
    <LinearLayout  
        android:orientation="vertical"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent">  
        <android.support.design.widget.TabLayout  
            android:id="@+id/tl_main"  
            android:layout_width="match_parent"  
            android:layout_height="wrap_content">  
        </android.support.design.widget.TabLayout>  
        <android.support.v4.view.ViewPager  
            android:id="@+id/vp_content"  
            android:layout_width="wrap_content"  
            android:layout_height="0dp"  
            android:layout_weight="1">  
        </android.support.v4.view.ViewPager>  
    </LinearLayout>  
</LinearLayout>  
```  
  
适配类  
  
MyPagerAdapter  
  
```  
/**  
 * Created by leon0001 on 2016/5/21.  
 */  
class MyPagerAdapter extends FragmentPagerAdapter {  
    List<Fragment> list;    //填充到ViewPager的fragment列表  
    List<String> title;     //tab中的title文字列表  
    public MyPagerAdapter(FragmentManager fm, List<Fragment> list, List<String>title) {  
        super(fm);  
        this.list = list;  
        this.title = title;  
    }  
    //获取所对应的fragment  
    @Override  
    public Fragment getItem(int position) {  
        return list.get(position);  
    }  
    //FragmentPager的个数  
    @Override  
    public int getCount() {  
        return list.size();  
    }  
    @Override  
    public CharSequence getPageTitle(int position) {  
        return title.get(position);  
    }  
}  
```  
  
创建BaseFragment类，留空，暂时没什么用，只是方便拓展一些。  
  
BaseFragment  
  
```  
public class BaseFragment extends Fragment{  
}  
```  
  
创建三个Fragment类  
  
```  
public class ContentFragment extends BaseFragment {  
    @Nullable  
    @Override  
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_content, container, false);  
        return view;  
    }  
}  
  
public class MusicFragment extends BaseFragment {  
    @Nullable  
    @Override  
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_music, container, false);  
        return view;  
    }  
}  
  
public class OtherFragment extends BaseFragment {  
    @Nullable  
    @Override  
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_other, container, false);  
        return view;  
    }  
}  
  
```  
  
创建对应的三个 Fragment布局文件，如：  
  
fragment_content.xml  
  
```  
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent">  
    <TextView  
        android:text="个性推荐"  
        android:textSize="40sp"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"/>  
</LinearLayout>  
```  
  
其余两个 fragment_other、fragment_music 布局文件类似  
  
最后一步，在MainActivity中初始化。  
  
```  
    private void initFragment(){  
        mViewPager = (ViewPager) findViewById(R.id.vp_content);  
        mTabLayout = (TabLayout) findViewById(R.id.tl_main);  
  
        BaseFragment fragment1 = new ContentFragment();  
        BaseFragment fragment2 = new MusicFragment();  
        BaseFragment fragment3 = new OtherFragment();  
  
        mViewList.add(fragment1);  
        mViewList.add(fragment2);  
        mViewList.add(fragment3);  
  
        mTitleList.add("个性推荐");  
        mTitleList.add("音乐");  
        mTitleList.add("其他");  
  
        mTabLayout.setTabMode(TabLayout.MODE_FIXED);  
        mTabLayout.addTab(mTabLayout.newTab());  
        mTabLayout.addTab(mTabLayout.newTab());  
        mTabLayout.addTab(mTabLayout.newTab());  
  
        MyPagerAdapter myPagerAdapter = new MyPagerAdapter(getSupportFragmentManager()  
                ,mViewList  
                ,mTitleList);  
        mViewPager.setAdapter(myPagerAdapter);  
        mTabLayout.setupWithViewPager(mViewPager);  
        //mTabLayout.setTabsFromPagerAdapter(myPagerAdapter);  
    }  
```  
  
最后在 Oncreate() 中执行 initFragment();  
  
```  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
  
        initFragment();  
    }  
```  
  
  
  
  
