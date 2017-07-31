---
layout: post
title: "Android 动态更新Viewpager+fragment完美解决方案"
keywords: 
description: ""
category: Android
tags: 
---

<!--markdown-->最近遇到个问题，一直没有找到很好的解决方案，今天终于解决了。  

Viewpager嵌套多个Fragment  
  
1. 现在我要改变fragment中的数据。  
  
   这个非常的简单，网上有很多答案都能解决。  
  
2. 改变Viewpager的数量，也就是说我要动态的增加或者删除Viewpager的页面数量。
  
   这个就非常操蛋了。  
  
查遍资料，才找到解决办法。写下来备忘。  

## 从FragmentPagerAdapter的运行机制中找解决办法 
  
通常使用FragmentPagerAdapter都会另外写个类，继承自FragmentPagerAdapter，类似下面的写法  
  
```  
    public class MyFragmentPagerAdapter extends FragmentPagerAdapter {  
        public MyFragmentPagerAdapter(FragmentManager fm) {  
            super(fm);  
        }  
        @Override  
        public Fragment getItem(int position) {  
            return NewsListFragment.newInstance(myUrls.get(position));  
        }  
        @Override  
        public int getCount() {  
            return myUrls.size();  
        }  
    }  
```  
  
以上只是个简单示例，链表正常是应该写在里面的。  
  
之前查资料看网上的答案是，复写 getItemPosition 方法，让其每次都调用getItem 方法加载新的内容，以此达到动态更新viewPager的目的。代码如下  
  
```  
@Override  
public int getItemPosition(Object object) {  
    //return super.getItemPosition(object);  
    return POSITION_NONE;  
}  
```  
  
实际上，这种方法在使用FragmentPagerAdapter 时时行不通的。FragmentPagerAdapter中有一个 `instantiateItem`方法，此方法会在 `getItem` 之前被调用。看下源码：  
  
```  
@Override  
    public Object instantiateItem(ViewGroup container, int position) {  
        if (mCurTransaction == null) {  
            mCurTransaction = mFragmentManager.beginTransaction();  
        }  
  
        final long itemId = getItemId(position);  
  
        // Do we already have this fragment?  
        String name = makeFragmentName(container.getId(), itemId);  
        Fragment fragment = mFragmentManager.findFragmentByTag(name);  
        if (fragment != null) {  
            if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);  
            mCurTransaction.attach(fragment);  
        } else {  
            fragment = getItem(position);  
            if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);  
            mCurTransaction.add(container.getId(), fragment,  
                    makeFragmentName(container.getId(), itemId));  
        }  
        if (fragment != mCurrentPrimaryItem) {  
            fragment.setMenuVisibility(false);  
            fragment.setUserVisibleHint(false);  
        }  
  
        return fragment;  
    }  
```  
  
可以看到，`getItem` 是在此方法中调用的。这里大概的意思是，每次getItem之前，都会判断有无这个fragment，如果有，就直接创建，如果没有，则调用`getItem` 得到一个fragment。  
  
之所以这么做，大概是为了运行效率吧，毕竟fragment还是很大的，每次都去重新创建，太耗性能了。所以，我们要如何来更新fragment ?  
  
网上的几种方法 大概的原因是 复写`instantiateItem` 方法，来做相关处理。但是实际使用中，当我增加或删除页面数量时，就会出问题。  
  
所以，尝试另外一种方法。上面的`instantiateItem` 方法中，保存tag时，使用的是 itemId作为标识，所以当我们改变了数据时，去改变一下 ID不就好了？  
  
```  
public class MyFragmentPagerAdapter extends FragmentPagerAdapter {  
        private long baseId = 0;  
  
        public MyFragmentPagerAdapter(FragmentManager fm) {  
            super(fm);  
        }  
  
//        @Override  
//        public int getItemPosition(Object object) {  
//            //return super.getItemPosition(object);  
//            return POSITION_NONE;  
//        }  
  
        @Override  
        public long getItemId(int position) {  
            // give an ID different from position when position has been changed  
            return baseId + position;  
        }  
  
        @Override  
        public Fragment getItem(int position) {  
            return NewsListFragment.newInstance(myUrls.get(position));  
        }  
  
        /**  
         * 更新fragment的数量之后，在调用notifyDataSetChanged之前，changeId(1) 改变id，改变tag  
         * @param n  
         */  
        public void changeId(int n) {  
            // shift the ID returned by getItemId outside the range of all previous fragments  
            baseId += getCount() + n;  
        }  
  
        @Override  
        public int getCount() {  
            return myUrls.size();  
        }  
  
        @Override  
        public CharSequence getPageTitle(int position) {  
            return myUrls.get(position).getName();  
        }  
    }  
```  
  
首先增加了个 baseId 作为标识。  
  
myUrls 只是我的一个链表数据，这个忽略掉就行。  
  
增加了一个 `changeId` 方法，用来改变 ID，重写 getItemId方法。  
  
所以此时，当我需要改变Viewpager中的fragment的数量或者数据时，先调用 `changeId` 方法，再调用`notifyDataSetChanged` ，如下：  
  
```  
mAdapter.changeId(1);  
mAdapter.notifyDataSetChanged();  
mViewpager.setAdapter(mAdapter);  
```  
  
此时，由于id不一样，所以FragmentPagerAdapter 会认为现在的 fragment和之前创建的fragment都不一样，从而调用getItem 来重新加载。从而达到 既可以改变数据，也能动态增加删除fragment的效果。  
  
参考:[http://stackoverflow.com/questions/10396321/remove-fragment-page-from-viewpager-in-android][1]  
  
  
  [1]: http://stackoverflow.com/questions/10396321/remove-fragment-page-from-viewpager-in-android  
