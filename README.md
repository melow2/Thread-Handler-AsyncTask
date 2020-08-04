# Thread
CPU는 연산을 하고 Memory는 연산을 할 수 있게 cpu로 옮겨준다. CPU가 동시간대 여러 작업을 하는 것을 MutiThread라고 한다. 
하지만 실제로 동시간대 작업을 하는 것이 아니며, switching 하는 시간이 빨라서 보통 동 시간대로 작업을 한다고 인식한다. 
주로 메인 스레드에서 시간이 길어지는 작업을 thread로 처리한다.

#
## 가장 간단한 구현 방법.
**1. Thread 객체의 run 메서드 구현.**
```
NewThread newThread = new NewThread();
newThread.setDaemon(true);              // 응용프로그램 종료 시, 함께 종료.
newThread.start();                      // run() 메소드 호출
class NewThread extends Thread {
    @Override
    public void run(){
            ....
    }
}
```
**2. Runnable 객체의 run 메서드 구현.**
```
SecondRunnable runnable = new SecondRunnable();
Thread newThread = new Thread(runnable);
newThread.setDaemon(true);
newThread.start();
class SecondRunnable implement Runnable {
    @Override
    public void run(){
            ....
    }
}
```
#
# Handler
스레드 간에 통신 매체이며,스레드에서 작업한 결과물로 메인스레드와의 통신이 필요한 경우 사용한다.
Handler의 경우에는 non-static 클래스이므로 외부 클래스의 레퍼런스를 가지고 있다.
따라서 앱이 종료되더라도 GC의 대상이 되지 않을 수 있으므로, 메모리 릭이 발생할 위험이 있다.
즉 Activity가 종료되더라도 GC가 되지 않고, Message가 Message queue에 남아 memory leak의 원인이 된다는 것이다.
**메모리 릭을 피하기 위한 Handler 코드는 아래와 같다.**

#
## WeakHandler
메인 스레드(UI 스레드)와 서브 스레드간에 메세지를 주고받는 예제.
서브 스레드에서는 view reference를 직접 참조할 수 없다.그래서 Handler를 통해
UI Thread로 신호(message)를 보내 통신할 수 있다. 
```
class MainActivity : AppCompatActivity() {

    lateinit var mBinding: ActivityMainBinding

    companion object {
        private var mainNum: Int = 0
        private var secondNum: Int = 0
        private lateinit var mHandler: MyHandler
        class MyHandler(val activity: MainActivity) : Handler(Looper.getMainLooper()) {
            private var mWeakActivity: WeakReference<MainActivity> = WeakReference(activity)
            override fun handleMessage(msg: Message) {
                super.handleMessage(msg)
                var _activity = mWeakActivity.get()
                if (_activity != null && msg.what==0) {
                    _activity.mBinding.tvSecond.text = secondNum.toString()
                }
            }
        }

    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        mHandler = MyHandler(this)
        mBinding.apply {
            btnStart.setOnClickListener {
                startNum();
            }
        }
    }

    private fun startNum() {
        mainNum++;
        val newThread = NewThread();
        newThread.isDaemon = true
        newThread.start()
        mBinding.tvMain.text = mainNum.toString()
    }

    class NewThread() : Thread() {
        override fun run() {
            while (true) {
                secondNum++;
                try {
                    sleep(600)
                } catch (e: Exception) { }
                val msg = Message.obtain() // 기존의 메세지 객채를 재활용. 메모리 낭비를 방지.
                msg.what = 0
                msg.arg1 = 0
                msg.arg2 = 0
                msg.obj = null
                mHandler.sendMessage(msg)
                mHandler.sendEmptyMessage(0); // 정보가 없을 경우. 신호만 보냄.
            }
        }
    }
}
```
#
## runOnUiThread 
Handler를 통하지 않고, UI 스레드의 작업을 진행 할 수 있다.
```
runOnUiThread{
// UI 스레드 작업.
}
```


