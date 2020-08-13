# 2020-08-pattern

Model
 - Entity, sqlite, room등

View
 - xml, Activity, Fragment


1. MVC
	- View로부터 입력 받거나 특정 이벤트 발생시 Model, View변경 할 수 있음.
	- View 내부에서 Controller역할을 수행.(Business logic을 View에서 가지고 있음)
	- 장점: 모델과 뷰는 분리, 모델은 어디에도 종속되지 않음. 직관성 있고 개발이 빠름
	- 단점: 유닛테스트가 어렵고, 코드가 길어지고 재사용 및 유지보수가 점차 어려워진다.

~~~ java
public class MainActivity extends AppCompatActivity implements MainViewHolder.HolderClickListener {
    
    RecyclerView recyclerView;
    LinearLayoutManager linearLayoutManager;
    MainAdapter adapter;
    Database database = Database.getInstance();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        recyclerView = findViewById(R.id.recycler_view);
        linearLayoutManager = new LinearLayoutManager(this);
        recyclerView.setLayoutManager(linearLayoutManager);
        adapter = new MainAdapter(this);
        recyclerView.setAdapter(adapter);
        adapter.setItems(database.getPersonList());
        database.setOnDatabaseListener(new Database.DatabaseListener(){
            @Override
            public void onChanged() {
                adapter.setItems(database.getPersonList());
            }
        });
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        menu.add("Add");
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        database.add(new Person(System.currentTimeMillis(), String.format("New Charles %d", new Random().nextInt(1000))));
        return super.onOptionsItemSelected(item);

    }

    @Override
    public void onDeleteClick(Person person) {
        database.remove(person);
    }
}
~~~
    
2. MVP
	- Pesent 개념을 두어 View에서 Business logic을 분리
	- 장점: View와 Model을 의존성이 없고 유닛테스트가 수월해짐
	- 단점: View와 Present간의 의존성이 높고 기능이 추가될때마다 Present가 거대해질 수 있음

~~~ java
public class MainActivity extends AppCompatActivity implements MainContract.View, MainViewHolder.HolderClickListener {
    
    RecyclerView recyclerView;
    LinearLayoutManager linearLayoutManager;
    MainAdapter adapter;
    MainContract.Presenter presenter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        recyclerView = findViewById(R.id.recycler_view);
        linearLayoutManager = new LinearLayoutManager(this);
        adapter = new MainAdapter(this);
        recyclerView.setLayoutManager(linearLayoutManager);
        recyclerView.setAdapter(adapter);
        presenter = new MainPresenter(Database.getInstance(), this);
        presenter.load();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        menu.add("Add");
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        presenter.addPerson(new Person(System.currentTimeMillis(), String.format("New Charles %d", new Random().nextInt(1000))));
        return super.onOptionsItemSelected(item);

    }

    @Override
    public void showPersonList(List<Person> personList) {
        adapter.setItems(personList);
    }

    @Override
    public void notifyDataChanged() {
        adapter.notifyDataSetChanged();
    }

    @Override
    public void onDeleteClick(Person person) {
        presenter.removePerson(person);
    }
}
~~~

3. MVVM
	- ViewModel은 Model을 wrapping, View에 필요한 Observable data준비. 
	- 장점: 뷰 의존성이 없음으로 유닛테스트 용이
	- 단점: 유지관리, XML에 코드를 추가할 수 있어 로직을 넣는경우 관리의 어려움, 이를 방지하기 위해 binding 표현식에서 뷰모델에서 값을 직접 가져오도록 하고 값을 계산하거나 파생하지 않는 방식으로 하자

~~~ java
public class MainActivity extends AppCompatActivity implements MainViewHolder.HolderClickListener {
   
    MainViewModel viewModel;
    LinearLayoutManager linearLayoutManager;
    MainAdapter adapter;
    ActivityMainBinding binding;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        linearLayoutManager = new LinearLayoutManager(this);
        adapter = new MainAdapter(this);
        binding.recyclerView.setLayoutManager(linearLayoutManager);
        binding.recyclerView.setAdapter(adapter);

        viewModel = new MainViewModel(Database.getInstance());
        binding.setViewModel(viewModel);

        viewModel.load();

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        menu.add("Add");
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        viewModel.addPerson(new Person(System.currentTimeMillis(), String.format("New Charles %d", new Random().nextInt(1000))));
        return super.onOptionsItemSelected(item);

    }

    @Override
    public void onDeleteClick(Person person) {
        viewModel.removePerson(person);
    }
}
~~~

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <variable
            name="viewModel"
            type="com.charlezz.mvvmsample.viewmodel.MainViewModel" />
    </data>
    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="0dp"
            android:layout_height="0dp"
            app:items="@{viewModel.items}"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
~~~
