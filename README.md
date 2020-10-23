이 내용은 Android Studio 2020년 10월 23일 기준 최신버전을 사용하고있습니다.
=============


Gradle
-------------
```
  implementation 'com.google.android.material:material:1.2.1'
  implementation 'androidx.viewpager2:viewpager2:1.0.0'
```
Gradle에 위에 2줄을 추가해주고 Sync해줍니다!
  
  
 layout
-------------
  Tablayout(이하 Tl) Viewpager2(이하 vp2) 를 사용할 activity 생성하거나 mainactivity에 넣으시구요 
  Fragment layout을 생성해줍니다. (여러개 사용할만큼) [생성할때는 blank로 전역으로 생성하고 체크박스 해제없이 합니다 {이때 target source set은 main입니다.}]
  
 
 Global varial (Data Class)
------------- 
// 전역변수로 사용하는이유 (개발하기 편함)
Fragment로 값을 보내기위해 전역 변수를 사용합니다 혹은 전역 data class 변수를 사용합니다.

저는 전역 참조의 클래스명은 global로 했고  변수명은 dataclass_send(data class)입니다
※JSON TO KOTLIN 플러그인을 이용해서 GSON 파싱으로 데이터 클래스를 생성해서 썻습니다

global.kt
```
class global {
    companion object {
        lateinit var Senddata: dataclass_send
    }
}
```
senddata_detail.kt
```
data class senddata_detail(
    val name: String,
    val code: String,
    val url: String
)
```
dataclass_send.kt
```
data class dataclass_send(
    val cameraList: List<senddata_detail>,
)
```
 
 Adapter
-------------
```
class FragmentPc(fa : FragmentActivity): FragmentStateAdapter(fa) { 
    override fun getItemCount(): Int {
        return 3 + global.Senddata.nameing.size // 변동하는 탭 갯수를위해
    }

    override fun createFragment(position: Int): Fragment {
        var fragment:Fragment
            when(position) {
                0 -> {
                    fragment = fragment1() // 0번 탭에 표시될 fragment
                }
                1 -> {
                    fragment = fragment2()// 1번 탭에 표시될 fragment
                }
                2 -> {
                    fragment = fragment3()// 2번 탭에 표시될 fragment
                }
                else -> { // n번 탭에 표시될 fragment
                    fragment = fragmentvarials() 
                    val bd:Bundle = Bundle()  // 값을 보내기위해 bundle을 준비
                    bd.putString("param1",global.dataclass_send.senddata_detail[position - 3].name) // activity 와 같습니다 값을 보냅니다.
                    fragment.arguments = bd // activity와 fragment의 데이터전달 차이점 은 arguments로 보낸다는점 activity는 intent로 put데이터를 넣죠? 
                }
            }

        return fragment
    }
}
```

 Fragment (값을 받아서 사용하는 예제만)
-------------
```
private const val ARG_PARAM1 = "param1" // 값을 받을 parameter 명 전역선언 혹은 getstring의 문자열로 적어주면됨!

class fragmentvarials: Fragment() {
  private var param1: String? = null // Fragment내부에서 사용하기위해 변수 선언
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        arguments?.let {
            param1 = it.getString(ARG_PARAM1) // activity와 비슷합니다. 
        }
    }
    override fun onPause() { // Fragment가 비활성화될경우 값 출력
        println("i'm pause ${param1}")
        super.onPause()
    }
    
    override fun onResume() { // Fragment가 활성화될경우 값 출력
        println(param1)
        super.onResume()
    }
}
```

 MainActivity(예제이니 Main에)
-------------
```
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        init()
    }
    private fun init() {
           Viewpager2.adapter = FragmentPc(this@MainActivity)
           TabLayoutMediator(PCedit_Tablayout, PCedit_Viewpager2) { tab, position ->
                    when (position) {
                        0 -> {
                            tab.text = "1번"
                        }
                        1 -> {
                            tab.text = "2번"
                        }
                        2 -> {
                            tab.text = "3번"
                        }
                        else -> {
                            val Camera_Current = global.dataclass_send.senddata_detail[position-3]
                            tab.text = Camera_Current.name // 변수의 내용에 따라 바뀌는 탭의 문자열을 위해
                        }
                    }
           }.attach()
            PCedit_Viewpager2.registerOnPageChangeCallback(object: ViewPager2.OnPageChangeCallback() { 
            //tablayout의 addOnTabSelectedListener은 맨처음 실행되는 Fragment를 모름니다.
            // 그래서 Viewpager2의 registerOnPageChangeCallback를 통해 맨 처음 실행되는 Fragment의 position을 알수있습니다.
                override fun onPageSelected(position: Int) {

                    when (position) {

                    }
                    super.onPageSelected(position)
                }
            })
            PCedit_Tablayout.addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener{
                override fun onTabReselected(tab: TabLayout.Tab?) {

                }

                override fun onTabUnselected(tab: TabLayout.Tab?) {

                }

                override fun onTabSelected(tab: TabLayout.Tab?) {

                }

            })
    }
    
}
```
끝!

이거때문에 어제 야근하고 오늘 아침 깨달음을 알았다.........

어제 하루동안 시도해본 횟수 약 10번? 

어떻게 Viewpager2에서 Fragment로 값을보내는거야!! 하고 찾다가

폭풍 구글링으로 결국 찾아냈다. 망할 Activity랑 비슷했다니!!!! bundle은 정말 처음봤다..

※처음에 이구성은 Recycler로 한개의 Activity에 layout옵션의 Visible= "Gone"으로 처리하고있다가

너무 코드가 복잡해지고 길어지고 해서 Fragment로 처리를해봤다. 

결과 - 아주깔끔  매우만족


~~풀스택은 역시 재밌어 ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ~~




