# 建立灵活动态的UI

> 编写:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/basics/fragments/fragment-ui.html>

如果你的APP设计成要支持范围广泛的屏幕尺寸时，在可利用的屏幕空间内，你可以通过在不同的布局配置中重用你的fragment来优化你的用户体验。

比如，一个手机设备可能适合一次只有一个fragment的单面板用户交互。相反，在更大屏幕尺寸的平板电脑上，你可能更想要两个fragment并排在一起，用来向用户展示更多信息。

![fragments-screen-mock](fragments-screen-mock.png)

**Figure 1.** 两个fragments，在同一个activity不同屏幕尺寸中用不同的配置来展示。在大屏幕上，两个fragment被并排放置，但是在手机上，一次只放置一个fragment，所以在用户导航中，两个fragment必须进行替换。

[FragmentManager](http://developer.android.com/reference/android/support/v4/app/FragmentManager.html)类提供了方法，让你在activity运行时能够对fragment进行添加，移除，替换，来实现动态的用户体验。

## 在activity运行时添加fragment

除了像[上节课](creating.html)介绍的用`<fragment>`标签在activity的布局文件中定义fragment的方法外，你还可以在activity运行时动态添加fragment。如果你打算在activity的生命周期内替换fragment，这是必须的。

为了执行fragment的增加或者移除操作，你必须用 [FragmentManager](http://developer.android.com/reference/android/support/v4/app/FragmentManager.html) 创建一个[FragmentTransaction](http://developer.android.com/intl/zh-cn/reference/android/support/v4/app/FragmentTransaction.html)对象, FragmentTransaction提供了用来增加、移除、替换以及其它一些操作的APIs。

如果你的activity允许移除或者替换fragment，那么你应该在activity的<a href="http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)">onCreate()</a>方法中添加初始的fragment(s).

运用fragment（特别是那些你在运行时添加的）的一个很重要的规则就是在布局中你必须有一个容器[View](http://developer.android.com/reference/android/view/View.html)，fragment的layout将会放在这个view里面。

下面的这个布局是[上节课](creating.html)介绍的布局的另一个版本，只不过这次的布局一次只显示一个fragment。为了从一个布局替换为另外一个布局，activity的布局包含了一个空的 [FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html)作为fragment的容器。

注意文件名与上节课的布局一样，但是文件目录没有`large`标识， 所以下面这个布局将被用在比large小的屏幕上，因为较小的屏幕无法同时放置两个fragments。

res/layout/news_articles.xml:
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

在你的activity里面，用Support Library APIs调用 [getSupportFragmentManager()](http://developer.android.com/intl/zh-cn/reference/android/support/v4/app/FragmentActivity.html#getSupportFragmentManager%28%29)方法获取[FragmentManager](http://developer.android.com/reference/android/support/v4/app/FragmentManager.html) 对象。然后调用 <a href="http://developer.android.com/reference/android/support/v4/app/FragmentManager.html#beginTransaction()">beginTransaction()</a> 方法创建一个[FragmentTransaction](http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html)对象，并调用<a href="http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#add(android.support.v4.app.Fragment,%20java.lang.String)">add()</a>方法添加一个fragment。

你可以使用同一个 [FragmentTransaction](http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html)进行多次fragment事务。当你完成这些变化操作的时候，必须调用<a href="http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#commit()">commit()</a>方法。

例如，如何添加一个fragment到之前的layout中

```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);

        // Check that the activity is using the layout version with
        // the fragment_container FrameLayout
        if (findViewById(R.id.fragment_container) != null) {

            // However, if we're being restored from a previous state,
            // then we don't need to do anything and should return or else
            // we could end up with overlapping fragments.
            if (savedInstanceState != null) {
                return;
            }

            // Create a new Fragment to be placed in the activity layout
            HeadlinesFragment firstFragment = new HeadlinesFragment();

            // In case this activity was started with special instructions from an
            // Intent, pass the Intent's extras to the fragment as arguments
            firstFragment.setArguments(getIntent().getExtras());

            // Add the fragment to the 'fragment_container' FrameLayout
            getSupportFragmentManager().beginTransaction()
                    .add(R.id.fragment_container, firstFragment).commit();
        }
    }
}
```

因为fragment是在activity运行时被添加进来的（不是在XML布局中用`<fragment>`定义的），activity 可以移除这个fragment或者用另外一个来替换它。

## Fragment替换

替换fragment的过程与添加过程类似，只需要将<a href="http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#add(android.support.v4.app.Fragment,%20java.lang.String)">add()</a>方法替换为 <a href="http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#replace(int,%20android.support.v4.app.Fragment)">replace()</a>方法。

记住当你执行fragment事务的时候，例如移除或者替换，你经常要适当地让用户可以向后导航与"撤销"这次改变。为了让用户向后导航fragment事务，你必须在[FragmentTransaction](http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html)提交前调用<a href="http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#addToBackStack(java.lang.String)">addToBackStack()</a>方法。

> **Note：**当你移除或者替换一个fragment并把它放入返回栈中时，被移除的fragment的生命周期是stopped(不是destoryed).当用户返回重新恢复这个fragment,它的生命周期是restarts。如果你没把fragment放入返回栈中，那么当他被移除或者替换时，它的生命周期是destoryed。

下面是替换一个fragment的例子

```java
// Create fragment and give it an argument specifying the article it should show
ArticleFragment newFragment = new ArticleFragment();
Bundle args = new Bundle();
args.putInt(ArticleFragment.ARG_POSITION, position);
newFragment.setArguments(args);

FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack so the user can navigate back
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```

<a href="http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#addToBackStack(java.lang.String)">addToBackStack()</a>方法提供了一个可选的String参数为事务指定了一个唯一的名字。这个名字不是必须的，除非你打算用[FragmentManager.BackStackEntry](http://developer.android.com/reference/android/support/v4/app/FragmentManager.BackStackEntry.html) APIs来进行一些高级的fragments操作。
