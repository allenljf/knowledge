#Activity
* Activity是四大元件之一，它提供一個介面讓使用者點選和各種滑動操作
* Activity生命週期
	* onCreate:初始，onStart:可見但沒顯示在前台，onResume:可互動

	* 主流程: onCreate -> onStart -> onResume -> running -> onPause -> onStop -> onDestroy

	* 返回頁面: onStop -> onRestart -> onStart -> onResume

	* 記憶體不足: onStop -> killed -> onCreate

	* 鎖屏(home鍵到桌面) onPause -> onStop -> 亮屏(從桌面回來) -> onRestart -> onStart -> onResume

	* Activity 1中途呼叫了另一個Activity 2時 onPause(1) -> onCreate(2) -> onStart(2) - onResume(2) -> onStop(1) 先暫停第一個，等第二個完成啟動時才stop第一個

	* 把剛剛的Activity 1再叫回來 onPause(2) -> onRestart(1) -> onStart(1) -> onResume(1) -> onStop(2) -> onDestroy(2)
----
* Activity 的啟動模式

	* standard(預設)，每次啟動創建一個，同一個Activity可以多存在多個

	* SingleTop，當最上方Activity又啟動自己一次，該intent會透過onNewIntent()被遞送到已存在的相同的Activity的實例，ex聊天室點推播復用聊天室UI，登入頁按兩次不會產生兩個實例，耗時操作從桌面回來

	* SingleTask，如果Activity不存在就開一個新的作為root activity，如果已存在就透過onNewIntent()將intent導到已存在的activity實例，並清除該activity stack之前的頁面，ex首頁

	* SingleInstance，只存在一個activity實例，ex整個手機系統只有一個打電話實例

	* clearTaskOnLaunch 該activity位於root，離開task後，則清除activity之外的所有activity

	* finishOnTaskLaunch Activity只要一離開task就清除
---
Activity的傳值

* 方法1 
val intent = Intent(this, ActivityB::class.java).apply {
    putExtra(EXTRA_MESSAGE, message)
}
startActivity(intent)

* 收值1
```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val message = intent.getStringExtra(EXTRA_MESSAGE)
    val textView = findViewById<TextView>(R.id.textView).apply {
        text = message
    }
}
```

* 方法2
```
var intent: Intent = Intent(this, ActivityB().javaClass) 
intent.putExtra("text", "傳值") 
startActivity(intent)
```

* 收值2
```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val bundle = this.intent.extras
        homeTitle.text = bundle?.get("text").toString()
    }
```
---
從Activity B返回時傳值給Acitvity A
Activity A 
```
var intent = Intent(this, ActivityB::class.java)
startActivityForResult(intent, 1)
```

```
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    when(resultCode) {
        1 -> if(resultCode == Acitvity.ReSULT_OK) {
             val returnValue = data?.getStringExtra("key")
        }
    }
}
```
Activity A新方法
```
registerForActivityResult(ActivityResultContracts.StartActivityForResult() {
    val data = it.data
    val resultCode = it.resultCode
}.launch(Intent(context, ActivityB::class.java))
```

Activity B
```
intent.putExtra("key", "value")
setResult(Acitvity.ReSULT_OK, intent)
finish()
```
---
* 螢幕旋轉儲存與恢復資料

	* onStop -> onSaveInstanceState -> bundle -> onStart -> onRestoreInstanceState
---
#Fragment
* Fragment生命週期
* 被建立 onAttach -> onCreate -> onCreateView -> onActivityCreated 可見 onStart -> onResume -> onPause -> onStop -> onDestoryView -> onDestroy -> onDetach

* Fragment從背景回來 onDestoryView -> onCreateView

* onAttach 當Fragment與Activity發生關聯時呼叫

* onCreateView 首次繪製頁面時呼叫

* onActivityCreated 可與Activity互動
---
Fragment與View的差別
Fragment可以管理生命週期，可組裝不同的View，但較耗資源

---
Fragment切換

```
val newFragment = MainFragment()
        val transaction = supportFragmentManager.beginTransaction()
        transaction.replace(R.id.fragment, newFragment)
        transaction.addToBackStack(null)
        transaction.commit()
```
避免切換new instance
```
if (!to.isAdded()) {
    // 隱藏當前的fragment，add下一個到Activity中
    transaction.hide(from).add(R.id.content_frame, to).commit()
} else {
    // 隱藏當前的fragment，顯示下一個
    transaction.hide(from).show(to).commit()
}
```
---
ViewPager 與 Fragment 關聯在一起
* FragmentPagerAdapter
當 Fragment 不被使用者看見時，此 Adapter 會執行 FragmentTransation 的 Detach（純粹銷毀畫面），如果要讓使用者看見時，就會執行 FragmentTransation 的 Attach。Fragment 不會從 FragmentManager 中移除，記憶體會消耗較大
* FragmentStatePagerAdapter
FragmentStatePagerAdapter 所用的機制是 add / remove
當使用者往後滑動時，前面的 Fragment 會被回收掉，只保留 當前的 Fragment 與 前後的 Fragment。
* FragmentStateAdapter 用於ViewPager2(此元件也可用RecyclerView.Adapter)

---

Fragment 傳遞資料
使用activityViewModels讓fragment之間與其宿主activity之間共享數據
```
class MainActivity : AppCompatActivity() {
    private val viewModel: ItemViewModel by viewModels()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.selectedItem.observe(this, Observer { item ->
            // Perform an action with the latest item data
        })
    }
}
```

```
class FragmentA : Fragment() {
    private val viewModel: ItemViewModel by activityViewModels()
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        viewModel.filteredList.observe(viewLifecycleOwner, Observer { list ->
            // Update the list UI
        }
    }
}
```

```
class FragmentB : Fragment() {
    private val viewModel: ItemViewModel by activityViewModels()
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        viewModel.filters.observe(viewLifecycleOwner, Observer { set ->
            // Update the selected filters UI
        }
    }
}
```
父 Fragment 與子 Fragment 共享數據 requireParentFragment
```
Class ListFragment: Fragment() {
    private val viewModel: ListViewModel by viewModels()
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        viewModel.filteredList.observe(viewLifecycleOwner, Observer { list ->
            // Update the list UI
        }
    }
}
```

```
class ChildFragment: Fragment() {
    private val viewModel: ListViewModel by viewModels({requireParentFragment()})
    ...
}
```

數據從Fragment B 傳回Fragment A 
Fragment A 
```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setFragmentResultListener("requestKey") { requestKey, bundle ->
        val result = bundle.getString("bundleKey")
        // Do something with the result
    }
}
```
Fragment B
```
setFragmentResult("requestKey", bundleOf("bundleKey" to result))
```

數據從子Fragment傳回父Fragment
父
```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    childFragmentManager.setFragmentResultListener("requestKey") { key, bundle ->
        val result = bundle.getString("bundleKey")
        // Do something with the result
    }
}
```
子
```
setFragmentResult("requestKey", bundleOf("bundleKey" to result))
```

宿主Activity接收Fragment結果
```
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        supportFragmentManager
                .setFragmentResultListener("requestKey", this) { requestKey, bundle ->
            val result = bundle.getString("bundleKey")
            // Do something with the result
        }
    }
}
```
---
#Service
BroadcastReceiver 的靜態動態註冊及區別
* 動態，用程式碼註冊，可自由控制註冊與取消，生命週期隨著當前頁面
* 靜態，要在AndroidManifest設定，app關閉還是可收到廣播
---
Content Provider將一些資料儲存在系統裡，然後可以讓其他 App 可以來讀取，App 可以透過ContentResolver去取出這些資料，比方說一些相關的聯絡人或是通話紀錄軟體

---
Service 是可以在背景中長時間執行操作的應用程式元件，沒有UI，執行在main thread，ex播放音樂，檔案輸入輸出，網路存取
* startedService
透過 startService( ) 啟動的 service 與呼叫者沒有關係，即使呼叫者關閉了， service 仍然會執行。想停止 service 要呼叫 stopService() ，系統會呼叫 onDestory( ) 停止 service 。
可用於音樂播放，檔案下載
LifeCycle : onCreate( ) ➞ onStartCommand( ) ➞ onDestroy( )

* bindService
透過 bindService( ) 啟動的 service 與呼叫者綁定，只要呼叫者關閉(destroy)， service 就終止。可用於顯示音樂秒數或下載進度畫面
LifeCycle : onCreate( ) ➞ onBind( ) ➞onUnbind( ) ➞ onDestroy( )

* IntentService
內有一個工作線程來處理耗時操作，這裏面的任務就會自動由新的執行緒來處理，每一個耗時操作會以工作隊列的方式，排進一個 queue 裡，用 onHandleIntent 去處理，最後會自動 onDestroy

---
#View
Custom View Component
* 製作Layout XML 可將root容器標籤設為merge消除重複嵌套優化效能
* 客製化屬性attr.xml定義需要的屬性
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="NumberSelect">
        <attr name="str_value" format="string  " />
        <attr name="int_value" format="integer" />
        <attr name="bool_value" format="boolean" />
        <attr name="color_value" format="color" />
        <attr name="dim_value" format="dimension" />
    </declare-styleable>
</resources>
```
* 建立Class繼承例如LinearLayout，在init的時候取得xml的設定值
```
private fun init(context: Context, attrs: AttributeSet?) {
        View.inflate(context, R.layout.number_select, this)
        this.str_value = 0
        if (attrs != null) {
            val attributes = context.theme.obtainStyledAttributes(
                attrs,
                R.styleable.NumberSelect,
                0, 0
            )
            //從Layout上 取得預設值
            this.str_value = attributes.getInt(R.styleable.NumberSelect_max_value,      this.maxValue)
```
---
#Thread
ANR(Application Not Responding) 主線程里做了太多耗時的操作系統會給用戶一個彈出提示(程式無響應)，讓用戶手動選擇繼續等待還是強制關閉此APP
主執行緒超過5秒沒有反應
broadcast receiver 10秒內沒執行完
service 各生命週期20秒內沒執行完

解決方法
避免在主線程做大量耗時操作，網路，計算，檔案存取，可用async task, thread, rxJava, coroutines等切換線程處理
避免載入大圖片
避免在onCreate, onResume做過多事情
broadCastReceiver可在onReceivce啟動service做事
善用MVP, MVVM架構

* Coroutines是輕量級的線程，去除了Callback的寫法讓非同步可以像同步程式一樣。它不會阻塞 thread，而且還可以被取消。Coroutine core 會幫你管理 thread 的數量，讓你不需要自行管理，這也可以避免不小心建立過多的 thread。
* Scope 可以操作它旗下所有的 Coroutines ，但反過來說，假設我 cancel 了 Scope ，它底下所有的 Coroutines 也都取消了
* Job 就是 Coroutines，更確切來說，Job 指的是 單一個 Coroutines 的生命週期
	* Dispatchers.Main：就是 Main thread 包裝，Android 需要操作到 UI thread 通常會用它
	* Dispatchers.Default：就是預設，會開另一個 Thread，通常不會跑在 Main thread 上
	* Dispatchers.IO：基於 Default 去加強的 Dispatchers ，它跟 Default 最本質的區別在於 ，它開的 thread 數量會比較多
* 最典型的特色，允許 method 被暫停( suspended)執行之後再回復(resumed)執行，而暫停執行的 method 狀態允許被保留，復原後再以暫停時的狀態繼續執行。在 main thread 執行到 function A 需要等 IO thread 耗時處理的結果，那我先暫停 function A， 協調讓出 main thread 讓 main thread 去執行其他的事情，等到 IO thread 的耗時處理結束後得到結果後再回復 function A 繼續執行讓程式碼更簡單更直覺，Kotlin 裡頭使用任何標記 suspend 的 method(後面會提)都會在 Scope 裡面，這樣才可以控制 Coroutines 的行進與存活與否。

* Handler
android中系统不允許在非Main Thread更新UI。當我們在非主線程做了耗時操作後，需要去更新UI的時候，我們就需要使用Handler來執行更新操作

1. Handler、Looper和MessageQueue是組成「消息傳遞機制」的重要原素。
2. Handler是一個類，其負責把消息物件「加到」消息隊列(其又可稱為訊息佇列MessageQueue，為採用先進先出法的隧道)裡。
3. Looper負責不斷地將消息物件從消息隊列中「取出」來。而如果消息隊列中無消息物件的話，則Looper會造成阻塞，即等待的狀態。
運行步驟：
1. Handler把消息物件放到消息隊列之中。
2. Looper不斷地於消息隊列的頭部向外取出消息物件。
3. Looper將會找到與消息物件對應的Handler物件。
4. Looper將會調用Handler物件中的「handleMessage(Message msg)」方法去處理該消息物件。

* HandlerThread
線程中建立一個Looper迴圈器，讓Looper輪詢訊息佇列，當有耗時任務進入佇列時，則不需要開啟新執行緒，在原有的執行緒中執行耗時任務即可，否則執行緒阻塞
	* HandlerThread優點是非同步不會堵塞，減少對效能的消耗
	* HandlerThread缺點是不能同時繼續進行多工處理，需要等待進行處理，處理效率較低
	* HandlerThread與執行緒池不同，HandlerThread是一個序列佇列，背後只有一個執行緒

```
var handlerThread: HandlerThread = HandlerThread("HandlerThread")
handlerThread.start()

var handlerThreadHandler = Handler(handlerThread.looper)
var handler=Handler()
downloadBtn.setOnClickListener {
   textView.text="Download..."
    handlerThreadHandler.post {
        Thread.sleep(10000)
       handler.post {
           textView.text="Download success"
       }
    }
}
```

```
// create and start new Handler Thread
val handlerThread = HandlerThread("MyBackgroundTask")
handlerThread.start()

// create new instance of Handler passing in the Looper from handlerThread above
val handler = Handler(handlerThread.looper)
val runnable = Runnable {
     // put your code that must run outside
}
// here we have to execute the runnable code
handler.post(runnable)
```

---
String 字串常量
StringBuffer 字串變數（執行緒安全）
StringBuilder 字串變數（非執行緒安全）

對 String 型別進行改變的時候其實都等同於生成了一個新的 String 物件
StringBuffer 類則結果就不一樣了，每次結果都會對 StringBuffer 物件本身進行操作，而不是生成新的物件，再改變物件引用
會在方方法上加synchronized關鍵字

String s1 
String s2
s1 + s2 慢於 StringBuffer

StringBuilder 用在字串緩衝區被單個執行緒使用的時候，大多情況速度較快

---
Value type 實值型別變數會儲存資料的值，如數值的1、3.51、10、149….等。指派一個實值型別變數給其他實值型別變數，會複製所包含的值。Java語言的基本資料型別(Primitive Types)如byte、short、int、long、float、double、boolean和char為實值型別。資料不會同步修改

Reference type 參考型別變數是儲存資料的參考(資料所在位址)。參考型別變數的指派會複製物件的參考，但不會複製物件的值。除了Java基本資料型別之外其他型別均為參考型別(Reference Types)，如類別(Class)、介面(Interface)、字串(String)、矩陣(Array)等。其中一個值修改，資料會同步修改

---
#Memory
一個已經廢棄的物件因為一些原因無法被回收，導致它一直占用記憶體造成浪費，即稱為記憶體洩漏。當一個物件持有一個生命週期比它短的物件的引用，就有可能引起記憶體洩漏。

常見記憶體洩漏
* 靜態集合類引起記憶體洩露 static HashMap引用的Object無法回收
* 監聽器 addXXXListener()，沒有再釋放物件時remove listener
* 單例模式引起的記憶體洩漏。因為單例擁有靜態特性，其生命週期跟整個app一樣長，在實例化一個需要context的單例對象時，若傳入activity的context，會使該單例一直持有該activity的引用，導致activity即便退出了也無法被回收，產生記憶體洩漏。
* Handler 延遲發送，或請求網路連線，途中Activity關閉，執行緒尚未完成，但持有Handler的引用，Handler又持有Activity的引用，導致該Activity無法回收直到請求結束

避免Memory Leak，有以下幾種做法：
* 使用適當大小的圖片
* 降低static變數的使用
* 要使用Context時，盡量使用ApplicationContext
* 各種網路連線使用try連線，finally釋放連線
* 當生命週期結束時，將可能洩露物件或行為停止，移除或設為空

LeakCanary是個很好的工具，只要在專案中implement它，你在操作APP時有遇到Memory Leak的狀況，它就會跳出通知(基本的原理就是 Activity 跑到 onDestroy 的時候，去檢測 Activity 是否真的被 GC)，並告訴妳在哪邊發生了Memory Leak。

StrongReference(強引用)，不主動回收
SoftReference(軟引用)，快out of memory才回收
WeakReference(弱引用)，gc時優先回收

使用WeakReference宣告變數，但記得做好被回收後的Error handling

---

#Design pattern
單例模式 Singleton
某个类，创建时需要消耗很多资源，即new出这个类的代价很大；或者是这个类占用很多内存，如果创建太多这个类实例会导致内存占用太多。

volatile
* 保证了不同线程对这个变量进行操作时的可见性即一个线程修改了某个变量的值，这新值对其他线程来是立即可见的。
* 禁止进行指令重排序。

```
public class Singleton{
    private volatile static Singleton instance;
    
    private Singleton(){};
    public static Singleton getInstance(){
        if(instance==null){
            sychronized(Singleton.class){
                if(instance==null)
                    instance=new Singleton();
            }
        }
        return instatnce;
    }
}
```

Builder模式
有很多变量之间依赖的关系，不按順序set最後才build再按順序執行

```
public class MyBuilder{
    private int id;
    private String num;
    public MyData build(){
        MyData d=new MyData();
        d.setId(id);
        d.setNum(num);
        return t;
    }
    public MyBuilder setId(int id){
        this.id=id;
        return this;
    }
    public MyBuilder setNum(String num){
        this.num=num;
        return this;
    }

}

public class Test{
    public static void  main(String[] args){
        MyData d=new MyBuilder().setId(10).setNum("hc").build();
    }
}
```

工廠模式
有一個父類別基類，由建構子傳入不同的參數決定new什麼子類

```
public abstract class Product{
    public abstract void method();
} 

public class ConcreteProductA extends Prodect{
    public void method(){
        System.out.println("我是产品A!");
    }
}

public class ConcreteProductB extends Prodect{
    public void method(){
        System.out.println("我是产品B!");
    }
}

public  abstract class Factory{
    public abstract Product createProduct();
}

public class MyFactory extends Factory{

    public Product createProduct(){
        return new ConcreteProductA();
    }
}
```

抽象工廠模式
抽象父類別基類，不實作方法，可作為不同特性子類的群組分類

```
public abstract class AbstractProductA{
    public abstract void method();
}
public abstract class AbstractProdectB{
    public abstract void method();
}

public class ConcreteProductA1 extends AbstractProductA{
    public void method(){
        System.out.println("具体产品A1的方法！");
    }
}
public class ConcreteProductA2 extends AbstractProductA{
    public void method(){
        System.out.println("具体产品A2的方法！");
    }
}
public class ConcreteProductB1 extends AbstractProductB{
    public void method(){
        System.out.println("具体产品B1的方法！");
    }
}
public class ConcreteProductB2 extends AbstractProductB{
    public void method(){
        System.out.println("具体产品B2的方法！");
    }
}

public abstract class AbstractFactory{
    public abstract AbstractProductA createProductA();

    public abstract AbstractProductB createProductB();
}

public  class ConcreteFactory1 extends AbstractFactory{
    public  AbstractProductA createProductA(){
        return new ConcreteProductA1();
    }

    public  AbstractProductB createProductB(){
        return new ConcreteProductB1();
    }
}

public  class ConcreteFactory2 extends AbstractFactory{
    public  AbstractProductA createProductA(){
        return new ConcreteProductA2();
    }

    public  AbstractProductB createProductB(){
        return new ConcreteProductB2();
    }
}
```

Apapter模式

RecyclerView每一個item中，把資料轉換為顯示邏輯

觀察者模式
RecyclerView當資料變化時，用notifyDataSetChanged()通知更新
RxJava Observable資料變化時，通知Oberver更新資料

裝飾模式
自己創一個新類，給一個基類額外添加方法

策略模式
演算法

---

#架構

架構

MVC
Android中对MVC的应用很经典，我们的布局文件如main.xml就是对应View层，
本地的数据库数据或者是网络下载的数据就是对应Model层，而Activity对应Controller层。
三者相互依賴，一但更新了其中一者，另外兩者也必須跟著修改
隨著不斷的開發和添加功能，Controller 的代碼會越來越臃腫
難以進行單元測試

MVP
和 MVC 不同的是，Model 層拿到數據後，並不直接傳給 View 更新，而是交還給 Presenter，Presenter 再把數據交給 View，並更新畫面。
三者相互依賴變成都只依賴 Presenter
View 只負責收到使用者回饋後，呼叫 Presenter 拿取數據，並在接收到數據的時候，更新畫面。
Model 被動的接收到 Presenter 命令，拿取資料，並回傳給 Presenter。
Presenter 透過介面與 View 和 Model 溝通，是 View 和 Model 的唯一連結窗口。
方便進行單元測試　
由於 Presenter 對 View 是透過介面進行操作，在對 Presenter 進行不依賴 UI 環境的單元測試的時候，可以 Mock 一個 View 對象，單元測試的時候就可以完整的測試 Presenter 業務邏輯的正確性。

MVVM
透過觀察者模式將 View 和 Model 巧妙地連接在一起，
一旦 Model 的數據發生變化，觀察者 View 就能夠感應到這個更動，並把數據更新到 UI 畫面上，ViewModel 甚至不需要持有 View 的引用，更方便進行單元測試。

省去了 MVP 中用來連接彼此的介面，Model 層數據更新後也不必透過介面 callback 給 view，因為 View 會透過 observe 感知數據的變動並更新畫面。

搭配 DataBinding、LiveData 等框架使用
MVP 需要 Mock 一個 View 對象才能進行測試，由於 ViewModel 不需持有 View 的引用，更方便進行單元測試。

---
#事件傳遞

觸控事件的型別 ACTION_DOWN，ACTION_MOVE，ACTION_UP
從外向內傳遞 Activity->ViewGroup->ViewGroup...->View 
從內向外消費 View->ViewGroup...->ViewGroup->Activity

返回true，表示事件被消耗掉。不需要分發下去。消費了事件，後續的事件會繼續傳遞到這裡。並且這裡會認為事件被View消費掉了，所以上層的VIewGroup不會收到要處理onTouchEvent的事件。
返回false，表示事件被廢棄掉。後續的其他事件不會被傳遞到這裡。而且可以理解為，我這個View不想處理這個事件，所以ViewGroup會觸發onTouchEvent事件。
dispatchTouchEvent 和onTouchEvent，返回true，代表由自己處理這個事情。阻斷後面的流程。
dispatchTouchEvent 和onTouchEvent，返回false，表示不由自己來處理這個事件，流程交由上一層級來處理。

---


