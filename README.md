# RoomBaslc2
演示视频

<video src="https://yi-sheep.github.io/RoomBasic2/Res/mp4/1.mp4"  autoplay loop muted>浏览器不支持播放该视频</video>

[无法播放点击](https://yi-sheep.github.io/RoomBasic2/Res/mp4/1.mp4)

Room 是在 SQLite 的基础上提供了一个抽象层，让用户能够在充分利用 SQLite 的强大功能的同时，获享更强健的数据库访问机制。
[官方文档](https://developer.android.google.cn/jetpack/androidx/releases/room?hl=zh_cn)

从上一个教程继续，为了避免太乱，我将上一个教程结束的项目从新复制出了一个，这一期还是从上一期结束哪里开始，只不过我将之前的无用代码删了，具体可以去看看源码，要是你还没看前面的教程，请移步[RoomBaslc](https://github.com/yi-sheep/RoomBasic)先看了再回来看这一期。

### 开始
这一期我们使用recyclerView将数据库的数据显示出来，首先是要改变一下之前的布局界面。

<img src="https://yi-sheep.github.io/RoomBasic2/Res/image/1.png"/>

我去掉了之前的ScrollView及其子控件,使用recyclerView代替了,还加入了一个switch，用来切换适配器。

然后创建两个itemView的布局，一个是普通的，一个是卡片。
先来看看普通的：

<img src="https://yi-sheep.github.io/RoomBasic2/Res/image/2.png"/>

这个就是一个ConsteaintLayout的根布局里面是几个控件，自己按照想法布局进行这里使用我的参考。
卡片：

<img src="https://yi-sheep.github.io/RoomBasic2/Res/image/3.png"/>

根布局是一个卡片布局了，然后里面再放了一个ConsteaintLayout的布局方便我们使用拖拽的方式，进行布局。

这些工作都完成了，就该创建一个适配器了。

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
    List<Word> mWords = new ArrayList<>(); // 用于获取数据
    boolean useCardView; // 用于判断用户选择使用卡片item还是默认item

    /**
     * 传入数据对象
     * @param words
     */
    public void setWords(List<Word> words) {
        mWords = words;
    }

    /**
     * 在实例化当前适配器的时候传入一个布尔值
     * true表示使用卡片item
     * false表示使用默认item
     * @param useCardView
     */
    public MyAdapter(boolean useCardView) {
        this.useCardView = useCardView;
    }

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        // 实现父类RecyclerView.Adapter中的抽象方法，在这个方法中初始化item，然后返回这个itemView
        LayoutInflater inflater = LayoutInflater.from(parent.getContext());
        View itemView;
        // 这里判断用户想要使用哪种item
        if (useCardView) {
            itemView = inflater.inflate(R.layout.item_c_layout, parent, false);
        } else {
            itemView = inflater.inflate(R.layout.item_layout, parent, false);
        }
        return new ViewHolder(itemView);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        // 这也是实现的父类的抽象方法，在这个方法中，做数据的绑定
        Word word = mWords.get(position); // 通过position(当前是第几itemView)获取到word中的对应数据
        holder.textViewNumber.setText(String.valueOf(position + 1)); // 设置布局中显示序号的textView,因为position是从0开始的，所以需要加1
        holder.textViewEnglish.setText(word.getWord()); // 设置布局中显示单词的textView,通过获取到的word中对应数据对象获取到单词
        holder.textViewChinese.setText(word.getChineseMeaning()); // 设置显示中文意思的textView
        // 给item设置点击事件
        holder.itemView.setOnClickListener(v -> {
            // 这里定义一个URI，使用的百度翻译的地址，通过分析前面一部分是固定的，想要翻译的单词跟在后面
            // 比如要翻译hello,uri就是 https://fanyi.baidu.com/#en/zh/hello
            Uri uri = Uri.parse("https://fanyi.baidu.com/#en/zh/" + holder.textViewEnglish.getText());
            Intent intent = new Intent(Intent.ACTION_VIEW); // 定义一个隐式意图
            intent.setData(uri); // 传递URI
            holder.itemView.getContext().startActivity(intent); // 启动意图
        });
    }

    @Override
    public int getItemCount() {
        // 实现父类的抽象方法，这个方法返回数据的长度
        return mWords.size();
    }

    /**
     * 这是自定义的ViewHolder
     * 用来获取itemView中的控件
     * 方便在适配器中直接对控件做操作
     * 必须要有一个构造函数并接受一个View
     */
    static public class ViewHolder extends RecyclerView.ViewHolder {
        TextView textViewNumber,textViewEnglish, textViewChinese;
        public ViewHolder(View itemView) {
            super(itemView);
            textViewNumber = itemView.findViewById(R.id.textViewNumber);
            textViewEnglish = itemView.findViewById(R.id.textViewEnglish);
            textViewChinese = itemView.findViewById(R.id.textViewChinese);
        }
    }
}
```

代码中的注释也很多，就不多解释了。注意控件id，有可能你的设置的id和我的不一样。

然后在MainActivity中修改代码，将不用的都删掉，特别是我们在改activity_main.xml的时候将滚动布局删掉了，这时候MainActivity中就会有些报错的地方，删掉进行。

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    WordViewModel mViewModel; // ViewModel
    RecyclerView mRecyclerView;
    MyAdapter mAdapter,mAdapter2; // 定义两个适配器变量
    Switch mSwitch;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mAdapter = new MyAdapter(true); // 初始化第一个适配器，使用的是卡片item
        mAdapter2 = new MyAdapter(false); // 初始化第二个适配器，使用的是默认item

        mRecyclerView = findViewById(R.id.recyclerView);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this)); // 给recyclerView设置布局样式
        mRecyclerView.setAdapter(mAdapter); // 设置视频器，这里我们先设置成卡片item的适配器
        mSwitch = findViewById(R.id.switch1);
        mSwitch.setChecked(true); // 因为我们设置的默认适配器是卡片item的所以需要改一下，自己领悟一下这里
        // Switch的点击事件
        mSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
            // 判断当前switch处于开状态还是关状态
            if (isChecked) {
                // 开状态,设置卡片item适配器
                mRecyclerView.setAdapter(mAdapter);
            } else {
                // 关状态,设置默认item适配器
                mRecyclerView.setAdapter(mAdapter2);
            }
        });

        mViewModel = ViewModelProviders.of(this).get(WordViewModel.class);
        mViewModel.getQueryAllWordLive().observe(this,words -> {
            // 在这里liveData监听到数据发生变化后
            // 将两个适配器的数据都有设置好，再通过调用notifyDataSetChanged()方法告诉recyclerView数据发生了变化你需要刷新界面
            mAdapter.setWords(words);
            mAdapter2.setWords(words);
            mAdapter.notifyDataSetChanged();
            mAdapter2.notifyDataSetChanged();
        });
        findViewById(R.id.button).setOnClickListener(this);
        findViewById(R.id.button2).setOnClickListener(this);
        findViewById(R.id.button3).setOnClickListener(this);
        findViewById(R.id.button4).setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                Word word1 = new Word("Hello!","你好！"); // 创建两个对象传入不同的数据
                Word word2 = new Word("Word.","世界。");
                mViewModel.insertWord(word1,word2);
                break;
            case R.id.button2:
                Word word3 = new Word("Hi!","你好！"); // 创建一个对象，传入要更新成的数据
                word3.setId(1); // 设置id，根据id修改数据，要修改那一个数据就设置那一个id
                mViewModel.updateWord(word3);
                break;
            case R.id.button3:
                mViewModel.deleteAllWord();
                break;
            case R.id.button4:
                Word word4 = new Word(); // 这创建一个空对象就可以了
                word4.setId(2); // 根据id删除
                mViewModel.deleteWord(word4);
                break;
        }
    }
}
```

现在就可以运行了，看看有没有问题。

下面我们来设置一个itemView的点击效果
在默认的那个item布局的根布局中加入，下面三句。

```xml
android:background="?attr/selectableItemBackground" // 设置背景，这个就是点击后有效果
android:clickable="true"
android:focusable="true"
```

在使用了卡片布局的那个xml中的ConsteaintLayout加入下面的。

```xml
android:background="?attr/selectableItemBackground"
android:focusable="true"
```

---

感谢B站大佬longway777的[视频教程](https://www.bilibili.com/video/BV1w4411k7iY)

如果侵权，请联系qq:1766816333
立即删除

---