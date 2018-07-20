Sir, I have used onSaveInstanceState() and onRestoreInstanceState() method to handle screen rotation. But App is crashing on rotation.
Will you please help me fix this issue?
=
MainActivity.java
-
```java
package com.inc.mountzoft.popularmoviesstage_2;

import android.content.Context;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Bundle;
import android.os.Parcelable;
import android.support.annotation.NonNull;
import android.support.design.widget.BottomNavigationView;
import android.support.v7.app.ActionBar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.RecyclerView;
import android.util.Log;
import android.view.MenuItem;
import android.view.View;
import android.widget.Toast;

import java.util.List;

import butterknife.BindView;
import butterknife.ButterKnife;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

public class MainActivity extends AppCompatActivity {

    public static boolean insertData;
    public static boolean popular = true;
    public static boolean topRated;

    public static String API_KEY;

    public static List<Movie> movies;

    @BindView(R.id.navigation)
    BottomNavigationView navigation;

    @BindView(R.id.recycler_view)
    RecyclerView mRecyclerView;
    GridAutofitLayoutManager layoutManager;

    private final String KEY_RECYCLER_STATE = "recycler_state";
    private Parcelable recyclerViewState;

    boolean firstTime = true;

    private ActionBar toolbar;

    ApiInterface apiService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        toolbar = getSupportActionBar();
        navigation.setOnNavigationItemSelectedListener(mOnNavigationItemSelectedListener);
        toolbar.setTitle("Most Popular");

        API_KEY = BuildConfig.MY_MOVIE_DB_API_KEY;

        apiService =
                ApiClient.getClient().create(ApiInterface.class);
        if(isNetworkAvailable()) {
            if(firstTime) {
                firstTime = false;
                loadPopularMovies();
            }
        }

        if(MoviesRecyclerViewAdapter.fav) {
            toolbar.setTitle(R.string.favorites);
            loadFavoriteMovies();
            mRecyclerView.setVisibility(View.VISIBLE);
        }
    }


    protected void onSaveInstanceState(Bundle state) {
        super.onSaveInstanceState(state);

        // Save list state
        recyclerViewState = layoutManager.onSaveInstanceState();
        state.putParcelable(KEY_RECYCLER_STATE, recyclerViewState);
    }

    protected void onRestoreInstanceState(Bundle state) {
        super.onRestoreInstanceState(state);

        // Retrieve list state and list/item positions
        if(state != null)
            recyclerViewState = state.getParcelable(KEY_RECYCLER_STATE);
    }

    @Override
    public void onResume() {
        super.onResume();
        Log.d("reached", "on_resume");

        if (recyclerViewState != null) {
            layoutManager.onRestoreInstanceState(recyclerViewState);
            Log.d("reached", "on_resume inside if");
        }
    }


    private BottomNavigationView.OnNavigationItemSelectedListener mOnNavigationItemSelectedListener
            = new BottomNavigationView.OnNavigationItemSelectedListener() {

        @Override
        public boolean onNavigationItemSelected(@NonNull MenuItem item) {
            switch (item.getItemId()) {
                case R.id.navigation_most_popular:

                    popular = true;
                    MoviesRecyclerViewAdapter.fav = false;
                    topRated = false;

                    toolbar.setTitle(R.string.most_popular);
                    if(isNetworkAvailable()) {
                        loadPopularMovies();
                    }else {
                        mRecyclerView.setVisibility(View.GONE);
                    }
                    return true;
                case R.id.navigation_top_rated:

                    topRated = true;
                    MoviesRecyclerViewAdapter.fav = false;
                    popular = false;

                    toolbar.setTitle(R.string.top_rated);
                    if(isNetworkAvailable()) {
                        loadTopRatedMovies();
                    }else {
                        mRecyclerView.setVisibility(View.GONE);
                    }
                    return true;
                    //==============================================================================
                case R.id.navigation_favorites:
                    MovieDetails.del = false;
                    MainActivity.insertData = false;

                    MoviesRecyclerViewAdapter.fav = true;
                    popular = false;
                    topRated = false;

                    RoomDatabaseInitializer.populateAsync(RoomMovieDatabase.getAppDatabase(getApplicationContext()));
                    toolbar.setTitle(R.string.favorites);
                    mRecyclerView.setVisibility(View.VISIBLE);
                    loadFavoriteMovies();
                    return true;
                    //==============================================================================
            }
            return false;
        }
    };
//==================================================================================================
    public void loadFavoriteMovies(){

        if(isNetworkAvailable()){

            insertData = false;
            RoomDatabaseInitializer.populateAsync(RoomMovieDatabase.getAppDatabase(this));

            Call<MovieResponse> callForMostFavoriteMovies = apiService.getPopularMovies(API_KEY);
            callForMostFavoriteMovies.enqueue(new Callback<MovieResponse>() {
                @Override
                public void onResponse(Call<MovieResponse>call, Response<MovieResponse> response) {
                    movies = response.body().getResults();
                    layoutManager = new GridAutofitLayoutManager(getApplicationContext(), 200);
                    mRecyclerView.setLayoutManager(layoutManager);
                    mRecyclerView.setAdapter(new MoviesRecyclerViewAdapter(getApplicationContext(), true, true));
                }

                @Override
                public void onFailure(Call<MovieResponse>call, Throwable t) {}
            });
        }else {

            insertData = false;
            RoomDatabaseInitializer.populateAsync(RoomMovieDatabase.getAppDatabase(this));

            layoutManager = new GridAutofitLayoutManager(getApplicationContext(), 200);
            mRecyclerView.setLayoutManager(layoutManager);
            mRecyclerView.setAdapter(new MoviesRecyclerViewAdapter(getApplicationContext(), true, false));
        }
    }
//==================================================================================================
public void loadPopularMovies(){
    Call<MovieResponse> callForMostPopularMovies = apiService.getPopularMovies(API_KEY);
    callForMostPopularMovies.enqueue(new Callback<MovieResponse>() {
        @Override
        public void onResponse(Call<MovieResponse> call, Response<MovieResponse> response) {
            movies = response.body().getResults();
            layoutManager = new GridAutofitLayoutManager(getApplicationContext(), 200);
            mRecyclerView.setLayoutManager(layoutManager);
            mRecyclerView.setAdapter(new MoviesRecyclerViewAdapter(getApplicationContext(), movies, false, true));
        }

        @Override
        public void onFailure(Call<MovieResponse> call, Throwable t) {}
    });
}
//==================================================================================================
public void loadTopRatedMovies(){
    Call<MovieResponse> callForTopRatedMovies = apiService.getTopRatedMovies(API_KEY);
    callForTopRatedMovies.enqueue(new Callback<MovieResponse>() {
        @Override
        public void onResponse(Call<MovieResponse> call, Response<MovieResponse> response) {
            movies = response.body().getResults();
            layoutManager = new GridAutofitLayoutManager(getApplicationContext(), 200);
            mRecyclerView.setLayoutManager(layoutManager);
            mRecyclerView.setAdapter(new MoviesRecyclerViewAdapter(getApplicationContext(), movies, false, true));
        }

        @Override
        public void onFailure(Call<MovieResponse> call, Throwable t) {}
    });
}
//==================================================================================================
    public boolean isNetworkAvailable() {
        ConnectivityManager connectivityManager
                = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
        return activeNetworkInfo != null && activeNetworkInfo.isConnected();
    }
}

```

This is what logcat saying while rotating
-
`
07-20 22:11:38.071 18995-18995/com.inc.mountzoft.popularmoviesstage_2 D/reached: on_resume
07-20 22:11:38.071 18995-18995/com.inc.mountzoft.popularmoviesstage_2 D/AndroidRuntime: Shutting down VM
07-20 22:11:38.073 18995-18995/com.inc.mountzoft.popularmoviesstage_2 E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.inc.mountzoft.popularmoviesstage_2, PID: 18995
    java.lang.RuntimeException: Unable to resume activity {com.inc.mountzoft.popularmoviesstage_2/com.inc.mountzoft.popularmoviesstage_2.MainActivity}: java.lang.NullPointerException: Attempt to invoke virtual method 'void com.inc.mountzoft.popularmoviesstage_2.GridAutofitLayoutManager.onRestoreInstanceState(android.os.Parcelable)' on a null object reference
        at android.app.ActivityThread.performResumeActivity(ActivityThread.java:3249)
        at android.app.ActivityThread.handleResumeActivity(ActivityThread.java:3280)
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2623)
        at android.app.ActivityThread.handleRelaunchActivity(ActivityThread.java:4226)
        at android.app.ActivityThread.access$900(ActivityThread.java:183)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1481)
        at android.os.Handler.dispatchMessage(Handler.java:111)
        at android.os.Looper.loop(Looper.java:194)
        at android.app.ActivityThread.main(ActivityThread.java:5651)
        at java.lang.reflect.Method.invoke(Native Method)
        at java.lang.reflect.Method.invoke(Method.java:372)
        at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:959)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:754)
     Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'void com.inc.mountzoft.popularmoviesstage_2.GridAutofitLayoutManager.onRestoreInstanceState(android.os.Parcelable)' on a null object reference
        at com.inc.mountzoft.popularmoviesstage_2.MainActivity.onResume(MainActivity.java:103)
        at android.app.Instrumentation.callActivityOnResume(Instrumentation.java:1267)
        at android.app.Activity.performResume(Activity.java:6205)
        at android.app.ActivityThread.performResumeActivity(ActivityThread.java:3234)
        at android.app.ActivityThread.handleResumeActivity(ActivityThread.java:3280) 
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2623) 
        at android.app.ActivityThread.handleRelaunchActivity(ActivityThread.java:4226) 
        at android.app.ActivityThread.access$900(ActivityThread.java:183) 
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1481) 
        at android.os.Handler.dispatchMessage(Handler.java:111) 
        at android.os.Looper.loop(Looper.java:194) 
        at android.app.ActivityThread.main(ActivityThread.java:5651) 
        at java.lang.reflect.Method.invoke(Native Method) 
        at java.lang.reflect.Method.invoke(Method.java:372) 
        at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:959) 
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:754) 
`

Project Link
-
https://github.com/Jithin-Jude/PopularMoviesStage_2
