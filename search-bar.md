# Search Bar



<div align="left" data-full-width="false">

<figure><img src=".gitbook/assets/search_bar.JPG" alt="" width="283"><figcaption><p>Android Search Bar</p></figcaption></figure>

 

<figure><img src=".gitbook/assets/search_demo.gif" alt="" width="300"><figcaption><p>Search Bar Demo</p></figcaption></figure>

</div>

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- NestedScrollingChild goes here (NestedScrollView, RecyclerView, etc.). -->
    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <!-- Screen content goes here. -->

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/searchRv"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
            tools:listitem="@layout/product_row_item" />
    </androidx.core.widget.NestedScrollView>

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appBarLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <com.google.android.material.search.SearchBar
            android:id="@+id/search_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Search Product" />
    </com.google.android.material.appbar.AppBarLayout>

    <com.google.android.material.search.SearchView
        android:id="@+id/searchView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:hint="Search Product"
        app:layout_anchor="@id/search_bar">
        <!-- Search suggestions/results go here (ScrollView, RecyclerView, etc.). -->


    </com.google.android.material.search.SearchView>
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

***

{% embed url="https://github.com/hamzaazman/DummyArch" %}
