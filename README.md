#WeatherFromGL
#ver_1.0.0

#Jetpack开发组件工具集
A1.ViewModel:1专门存放与界面相关的数据，只要是界面能看到的数据，它的相关变量都应该存放在viewModel中。
            2viewModel的生命周期和Activity不同，它可以保证手机屏幕发生旋转的时候不会重新被创建，只有当Activity退出的时候才会跟着Activity一起销毁。
            
            
1.class MainViewModel: ViewModel() {
    var count = 0
}
2.Activity中获取viewModel
 viewModel = ViewModelProviders.of(this).get(MainViewModel::class.java)  
 var toString = viewModel.count.toString()
 
 (####不可以直接去创建viewModel的实例，一定要通过ViewModelProviders来获取viewModel的实例，因为viewmodel有独立的生命周期，且生命周期长于Activity,如果在onCreate()获取viewModel的实例，那么每次onCreate()执行的时候，viewModel都会创建一个新的实例，这样当手机屏幕旋转的时候，就无法保 留里面的数据了*******)

A2 向viewModel中传递数据

1.class MainViewModel4(countReserved:Int):ViewModel() {
   var counter = countReserved
   
}

//向MainViewModel4的构造方法中传递数据，需要用到ViewModelProvider.Factory

2.class MyViewModel4Factory(private val countReserved: Int) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return MainViewModel4(countReserved) as T
            
    }
    
}

3.Activity中获取viewModel

    lateinit var viewModel4: MainViewModel4
    lateinit var sp: SharedPreferences
    
        sp = getPreferences(Context.MODE_PRIVATE)

        val counterReserved = sp.getInt("count_reserved", 0)

        viewModel4 = ViewModelProviders.of(this, MyViewModel4Factory(counterReserved))
        .get(MainViewModel4::class.java)

        button_add.setOnClickListener {
            viewModel4.counter++
            refreshCount()
        }
        button_clear.setOnClickListener {
            viewModel4.counter = 0
            refreshCount()
        }
        refreshCount()
    }

    fun refreshCount() {
        textView.text = viewModel4.counter.toString()
    }

    override fun onPause() {
        super.onPause()
        sp.edit().putInt("count_reserved", viewModel4.counter).apply()
    }


A3
Lifecycles 感知 activity fragment 的生命周期



1.class MyObserver : LifecycleObserver {    

    //activityStart()在activity的onStart()触发时调用
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun activityStart() {

    }

    //activityStart()在activity的onStop触发时调用
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun activityStop(){
        
    }

}
2.
当activity的生命周期发生变化时去通知MyObserver,Activity Fragment 本身就是一个lifecycleOwner的实例
observer = MyObserver()
lifecycle.addObserver(observer)//MyObserver 就能自动感知到Activity的生命周期了


只要activity 是继承 AppCompatActivity的，或者Fragment是继承自andoidx.fragment.app.Fragment的，那么它们本身就是一个LifecycleOwner的实例
主动获知当前的生命周期的状态

class MyObserver2(val lifecycle:Lifecycle):LifecycleObserver{

}

 observer2  = MyObserver2(lifecycle)
 var currentState = lifecycle.currentState
 lifecycle.addObserver(observer2)



object KS{

    private var keyStore: KeyStore = KeyStore.getInstance("AndroidKeyStore").also { it.load(null) }
   
    init {
       getKeyPair()
    }

    private fun getKeyPair() {
        val kpg = KeyPairGenerator.getInstance(  KeyProperties.KEY_ALGORITHM_RSA, "AndroidKeyStore"  )
        kpg.initialize(
            KeyGenParameterSpec.Builder(
                "ALIAS",
                KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
            )
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_RSA_PKCS1)
                .setDigests(
                    KeyProperties.DIGEST_SHA256, KeyProperties.DIGEST_SHA512
                ).build()
        )
        kpg.generateKeyPair()
    }

    private fun containsAlias(): Boolean {
        if (TextUtils.isEmpty("ALIAS")) {
            return false
        }
        var contains = false
        try {
            contains = keyStore.containsAlias("ALIAS")
        } catch (e: java.lang.Exception) {
            e.printStackTrace()
        }
        return contains
    }

    fun encode(content: String?): ByteArray? {
        if (content == null) {
            return null
        }
        try {
            val publicKey = keyStore.getCertificate("ALIAS").publicKey
            val cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")
            cipher.init(Cipher.ENCRYPT_MODE, publicKey)
            return cipher.doFinal(content.toByteArray())
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return null
    }

    fun decode(data: ByteArray?): String? {
        if (data == null) {
            return null
        }
        try {
            val entry = keyStore.getEntry("ALIAS", null)
            val privateKeyEntry = entry as KeyStore.PrivateKeyEntry
            val cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")
            cipher.init(Cipher.DECRYPT_MODE, privateKeyEntry.privateKey)
            return String(cipher.doFinal(data))
        } catch (e:Exception) {
            e.printStackTrace()
        }
        return null
    }}
