# XMVP
[![GitHub release](https://img.shields.io/badge/release-1.1.1-red.svg)](https://github.com/xujiaji/XMVP/releases) [![GitHub release](https://img.shields.io/badge/bintray-1.1.1-brightgreen.svg)](https://bintray.com/xujiaji/maven/xmvp/1.1.1) 

# 中文文档: [XMVP（简洁的MVP框架）](http://www.jianshu.com/p/b60e8ef138d1)

# Update
> v1.1.1 Add a fragment of the extended V4 package in the 'io.xujiaji.xmvp.view.base.v4' package.

# Introduction
1. You need to customize the 'View', 'Presenter', 'Model' subclasses inherited from 'XContract'.
2. You need a class that implements the just-defined 'XContract.Presenter' subclass interface.
3. You need to extend the 'XBasePresenter' to make it implement the 'XContract.Presenter' subinterface defined in step 1 and add two generics: the first is the View interface and the second is the Model implementation class.
4. You need to extend 'XBaseActivity' or 'XBaseFragment' so that it implements the 'XContract.View' subinterface defined in step 1 and add the 'XBasePresenter' implementation class in step 3 to generic.

just now:

- You can call the method in step 2 directly through the presenter in the Activity or Fragment
- You can call the definition of the View interface in Presenter method, and can be called by model in step 2 Model implementation of the class method
- and the start and end methods in Presenter for the start and end of the Activity and Fragmentt lifecycle.
- through XMVP you do not care about View, Presenter, Model is how to connect, you can easily decouple the project.
- Finally you can easily build the code using the '[MVPManager](https://github.com/xujiaji/MVPManager)' plugin


# How to use?
> Just need 4 steps. 

### Add xmvp dependency into your build.gradle
```
dependencies {
    compile 'com.github.xujiaji:xmvp:1.1.1'
}
```
### step1:define a contract
You need to define a contract in contracts package, it contains a extend 'XContract.Presenter' interface and a extend 'XContract.View' interface.
> example:HomeContract

``` java
public interface HomeContract {
    interface Presenter extends XContract.Presenter{
        void loadData(Activity activity);
    }

    interface View extends XContract.View{
        void loadStart();
        void loadEnd(List<FileEntity> fileEntities);
    }

    interface Model extends XContract.Model {
        void scanFile(final Activity activity, final FileHelper.Listener<List<FileEntity>> listener);
    }
}
```

### step2:An implementation class for the Model interface.
> example:HomeModel

``` java
public class HomeModel implements HomeContract.Model {
    @Override
    public void scanFile(final Activity activity, final FileHelper.Listener<List<FileEntity>> listener) {
        new Thread() {
            @Override
            public void run() {
                final FileHelper fileHelper = new FileHelper();
                fileHelper.unzip(activity, listener);
            }
        }.start();
    }
}
```

### step3:An implementation class for the Presenter interface.
> example: HomePresenter

``` java
public class HomePresenter extends XBasePresenter<HomeContract.View, HomeModel> implements HomeContract.Presenter {

    @Override
    public void loadData(Activity activity) {
        view.loadStart();
        model.scanFile(activity, new FileHelper.Listener<List<FileEntity>>() {
            @Override
            public void success(List<FileEntity> fileEntities) {
                view.loadEnd(fileEntities);
            }
        });
    }
}
```

### step4:An implementation class for the View interface.
> example: HomeActivity

``` java
public class HomeActivity extends XBaseActivity<HomePresenter> implements HomeContract.View {
    @BindView(R.id.toolbar)
    Toolbar toolbar;
    @BindView(R.id.rvFolder)
    RecyclerView rvFolder;
    @BindView(R.id.fragParentLayout)
    FrameLayout fragParentLayout;
    private ProgressDialog dialog;
    private HomeAdapter adapter;
    @BindView(R.id.appBarLayout)
    AppBarLayout appBarLayout;

    @Override
    protected void onInit() {
        super.onInit();
        ButterKnife.bind(this);
        toolbar.setTitle(R.string.xmvp_folder);
        setSupportActionBar(toolbar);
        rvFolder.setLayoutManager(new LinearLayoutManager(this));
        adapter = new HomeAdapter(new ArrayList<FileEntity>());
        rvFolder.setAdapter(adapter);
        presenter.loadData(this);
    }

    @Override
    protected void onListener() {
        super.onListener();
        adapter.setOpenLister(new HomeAdapter.OpenListener() {
            @Override
            public void open(File file) {
                showCode(file);
            }
        });

        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                onBackPressed();
            }
        });
    }

    /**
     * 显示代码
     *
     * @param file
     */
    private void showCode(File file) {
        FragmentTransaction ft = getFragmentManager().beginTransaction();
        CodeFragment cf = CodeFragment.newInstance(file);
        ft.replace(R.id.fragParentLayout, cf);
        ft.addToBackStack(null);
        ft.commit();
        showBlack(file.getName());
//        rvFolder.setVisibility(View.GONE);
        fragParentLayout.setVisibility(View.VISIBLE);
    }

    /**
     * 显示返回按钮和标题，传入null则隐藏返回按钮
     * @param title
     */
    private void showBlack(String title) {
        if (title != null) {
            toolbar.setNavigationIcon(R.drawable.ic_navigate_before_black_24dp);
            toolbar.setTitle(title);
            appBarLayout.setExpanded(true, true);
        } else {
            toolbar.setNavigationIcon(null);
            toolbar.setTitle(R.string.xmvp_folder);
            fragParentLayout.setVisibility(View.GONE);
        }

    }


    @Override
    public void loadStart() {
        dialog = new ProgressDialog(this);
        dialog.setTitle("正在加载数据");
        dialog.setCancelable(false);
        dialog.show();
    }

    @Override
    public void loadEnd(List<FileEntity> fileEntities) {
        if (dialog != null && dialog.isShowing()) {
            dialog.dismiss();
        }
        adapter.getData().addAll(fileEntities);
        adapter.notifyDataSetChanged();
    }

    @Override
    protected int getContentId() {
        return R.layout.activity_home;
    }

    @Override
    public void onBackPressed() {
        if (fragParentLayout.getVisibility() == View.VISIBLE) {
            showBlack(null);
        }
        super.onBackPressed();
    }
}
```

#### You think this MVP too much trouble?
MVPManager helps you manage MVP code quickly.
[link MVPManager](https://github.com/xujiaji/MVPManager)

#### Home UML
![mvp uml](display/mvp.png)

# License
```
   Copyright 2016 XuJiaji

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
```
