--------------------2015-11-30 17:25:37 ----------------------------
#### 1. 改变物理back键逻辑 ####
	@Override  
    public boolean onKeyDown(int keyCode, KeyEvent event)  
    {  
        if (keyCode == KeyEvent.KEYCODE_BACK )  
        {  
            // 创建退出对话框  
            AlertDialog isExit = new AlertDialog.Builder(this).create();  
            // 设置对话框标题  
            isExit.setTitle("系统提示");  
            // 设置对话框消息  
            isExit.setMessage("确定要退出吗");  
            // 添加选择按钮并注册监听  
            isExit.setButton("确定", lisImageSwitchertener);  
            isExit.setButton2("取消", listener);  
            // 显示对话框  
            isExit.show();  
        }  
        return false;  
    }  
#### 2. Activity和Fragment的生命周期 ####

1. Activity
 
	[http://blog.csdn.net/android_tutor/article/details/5772285](http://blog.csdn.net/android_tutor/article/details/5772285)
![](http://i.imgur.com/37kMWwb.png)

2. Fragment

	[http://www.cnblogs.com/smyhvae/p/3983234.html](http://www.cnblogs.com/smyhvae/p/3983234.html "Fragment生命周期")
![](http://i.imgur.com/uYL4lMq.png)
![](http://i.imgur.com/MYY2z70.png)


#### 3. Android所有权限列表 ####
	[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1105/510.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1105/510.html)
#### 4. Android 判断网络类型（以广播形式） ####

	class NetWorkReceiver extends BroadcastReceiver{

	    @Override
	    public void onReceive(Context context, Intent intent) {
	        ConnectivityManager connectivityManager = (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
	        NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
	        if(networkInfo!=null&&networkInfo.isConnected()) {
				String type = networkInfo.getTypeName(); 
				if (type.equalsIgnoreCase("WIFI")) {  
	                mNetWorkType = NETWORKTYPE_WIFI;  
	            } else if (type.equalsIgnoreCase("MOBILE")) {  
	                String proxyHost = android.net.Proxy.getDefaultHost();
					mNetWorkType = TextUtils.isEmpty(proxyHost)  
                        ? (isFastMobileNetwork(context) ? NETWORKTYPE_3G : NETWORKTYPE_2G)  
                        : NETWORKTYPE_WAP;  
	        }else {
	            Toast.makeText(context, "您的网络不可用，请检察网络", Toast.LENGTH_SHORT).show();
	        }
	    }

	}

	/**判断移动网络类型*/
	﻿﻿﻿﻿private static boolean isFastMobileNetwork(Context context) {
		Context.TELEPHONY_SERVICE);
	    switch (telephonyManager.getNetworkType()) {
	        case TelephonyManager.NETWORK_TYPE_1xRTT:
	            return false; // ~ 50-100 kbps
	        case TelephonyManager.NETWORK_TYPE_CDMA:
	            return false; // ~ 14-64 kbps
	        case TelephonyManager.NETWORK_TYPE_EDGE:
	            return false; // ~ 50-100 kbps
	        case TelephonyManager.NETWORK_TYPE_EVDO_0:
	            return true; // ~ 400-1000 kbps
	        case TelephonyManager.NETWORK_TYPE_EVDO_A:
	            return true; // ~ 600-1400 kbps
	        case TelephonyManager.NETWORK_TYPE_GPRS:
	            return false; // ~ 100 kbps
	        case TelephonyManager.NETWORK_TYPE_HSDPA:
	            return true; // ~ 2-14 Mbps
	        case TelephonyManager.NETWORK_TYPE_HSPA:
	            return true; // ~ 700-1700 kbps
	        case TelephonyManager.NETWORK_TYPE_HSUPA:
	            return true; // ~ 1-23 Mbps
	        case TelephonyManager.NETWORK_TYPE_UMTS:
	            return true; // ~ 400-7000 kbps
	        case TelephonyManager.NETWORK_TYPE_EHRPD:
	            return true; // ~ 1-2 Mbps
	        case TelephonyManager.NETWORK_TYPE_EVDO_B:
	            return true; // ~ 5 Mbps
	        case TelephonyManager.NETWORK_TYPE_HSPAP:
	            return true; // ~ 10-20 Mbps
	        case TelephonyManager.NETWORK_TYPE_IDEN:
	            return false; // ~25 kbps
	        case TelephonyManager.NETWORK_TYPE_LTE:
	            return true; // ~ 10+ Mbps
	        case TelephonyManager.NETWORK_TYPE_UNKNOWN:
	            return false;
	        default:
	            return false;
	        }
    }
记得申明网络权限 “android.permission.ACCESS_NETWORK_STATE”

#### 5. 利用广播实现开机启动 ####
（1）静态注册广播：

	<receiver android:name="XXXReceiver">
		<intent-filter>
			<action android:name="android.intent.action.BOOT_COMPLETED"/>
		</intent-filter>
	</receiver>

（2）自定义广播接收者：

	public class BootCompleteReceiver extends BroadcastReceiver{
		 @Override
    	public void onReceive(Context context, Intent intent) {
			//TODO 开机启动的逻辑

		}
	}
#### 6. activity管理栈 ####
	/**
	 * Activity管理器
	 **/
	public class ActivityCollector{
	    public static List<Activity> activities = new ArrayList<>();
	    public static void addActivity(Activity activity){
	        activities.add(activity);
	    }
	    public static void removeActivity(Activity activity){
	        activities.remove(activity);
	    }
	    public static void finishAll(){
	        for(Activity activity: activities){
	            if(!activities.isFinishing()){
	                activity.finish();
	            }
	        }
	    }
	}

	/**
	 *BaseActivity
	 */
	public class BaseActivity extends Activity{
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        ActivityCollector.addActivity(this);
	    }
	
	    @Override
	    protected void onDestroy() {
	        super.onDestroy();
	        ActivityCollector.removeActivity(this);
	    }
	}

#### 7.完全退出应用 ####
方法一、自己维护一个Activity任务栈
用集合保存所有Activity实例，退出应用时，遍历集合，逐个消灭

	MyApplication.java

	import java.util.LinkedList;
	import java.util.List;
	import android.app.Activity;
	import android.app.AlertDialog;
	import android.app.Application;
	import android.content.DialogInterface;
	import android.content.Intent;
	
	public class MyApplication extends Application {
		private List<Activity> mList = new LinkedList<Activity>();
		private static MyApplication instance;
		
		private MyApplication() {
		}
		
		public synchronized static MyApplication getInstance() {
			if (instance == null) {
			  instance = new MyApplication();
			}
			return instance;
		}
		
		// add Activity
		public void addActivity(Activity activity) {
		  if(mList.contains(activity) return;
		  mList.add(activity);
		}
		// 遍历集合，逐一finish
		// 最好从Activity栈底开始干掉
		public void exit() {
			try {
			  for (Activity activity : mList) {
			    if (activity != null)
			      activity.finish();
			  }
			} catch (Exception e) {}
		}
		
		@Override
		public void onLowMemory() {
			  super.onLowMemory();
			  System.gc();
			}
	}

写一个BaseActivity当作所有activity 的父类，然后初始化的时候执行如下动作：

	public void onCreate(Bundle savedInstanceState) {
	  super.onCreate(savedInstanceState);
	  //把所有activity加入activity管理栈
	  MyApplication.getInstance().addActivity(this);
	}

	


在需要退出程序的时候，调用：

	MyApplication.getInstance().exit();


方法二、使用广播

编写一个BaseActivity，此为所有Activity的基类，其内部注册一个退出Action的广播接收者，故而其所有的子Activity都会注册这个接收者。
当收到退出广播时，各个子Activity调用自己的finish()，结束一生。

	BaseActivity.java
	
	public abstract class BaseActivity extends Activity {
		protected static final String ACTION_EXIT = "net.yrom.action.EXIT";
		// 写一个广播的内部类，当收到动作时，结束activity
		private BroadcastReceiver exitReceiver = new BroadcastReceiver() {
		@Override
		public void onReceive(Context context, Intent intent) {
		  ((Activity)context)finish(); // 子Activity结束今生
		  }
		};
		
		@Override
		public void onResume() {
		  super.onResume();
		  // 在基类activity中注册广播
		  IntentFilter filter = new IntentFilter(ACTION_EXIT);
		  registerReceiver(exitReceiver, filter);
		}
		// 子Activity调用这个方法来退出整个应用
		public void close() {
		  Intent intent = new Intent(ACTION_EXIT);
		  sendBroadcast(intent);// 该函数用于发送广播
		  finish();
		}
		@Override
		protected void onDestroy() {
		  super.onDestroy();
		  unregisterReceiver(exitReceiver);
		
		}
	}





------------------------2015-12-01 22:47:28 ------------------------------

#### 1. 多点登录强制下线（单点登录SSO） ####

#### 2. Android的几种dialog ####
 后面加了。。。

#### 3. Android NetWorkInfo类的isConnected()与isAvailable()两个方法 ####
	public class MainActivity extends Activity
	{
	    /** Called when the activity is first created. */
	    @Override
	    public void onCreate(Bundle savedInstanceState)
	    {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	 
	        this.registerReceiver(mBroadcastReceiver, new IntentFilter(
	                ConnectivityManager.CONNECTIVITY_ACTION));
	    }

		private BroadcastReceiver mBroadcastReceiver = new BroadcastReceiver()
	    {
	        @Override
	        public void onReceive(Context context, Intent intent)
	        {
	            Bundle bundle = intent.getExtras();
	            NetworkInfo aNetworkInfo = (NetworkInfo) bundle
	                    .get(ConnectivityManager.EXTRA_NETWORK_INFO);
	 
	            if (aNetworkInfo.isConnected())
	            {
	                System.out.println("connecte");
	            } else
	            {
	                System.out.println("not connect");
	            }
	 
	            if (aNetworkInfo.isAvailable())
	            {
	                System.out.println("available");
	            } else
	            {
	                System.out.println("not available");
	            }
	            System.out.println("--------------------");
	        }
	   	 };
	}

	各个状态如下：
	
	1，显示连接已保存，但标题栏没有，即没有实质连接上，       输出为：not connect， available
	
	2，显示连接已保存，标题栏也有已连接上的图标，            输出为：connect， available
	
	3，选择不保存后                                       输出为：not connect， available
	
	4，选择连接，在正在获取IP地址时                        输出为：not connect， not available
	
	5，连接上后                                          输出为：connect， available

#### 4. BroadcastReceiver生命周期 ####

 （1）每次广播到来时 , 会重新创建 BroadcastReceiver 对象 , 并且调用 onReceive() 方法 , 执行完以后 , 该对象即被销毁 . 当 onReceive() 方法在 10 秒内没有执行完毕，就会导致ANR。如果需要执行长任务，那么就有必要使用Service。千万不要使用新线程，这是很危险的事情。因为有可能线程没有执行完，BroadcastReceiver就挂了。BroadcastReceiver 会堵塞主线程。唯有 onReceive() 结束，主线程才得以继续进行。

（2）android:priority表示优先级，最大优先级为integer 的最大值2147483647，“最高”只是限于静态注册。
在，没有限制优先级的情况下，动态注册比静态注册权限高。

（3）通过静态方式注册的广播接收器，不需要手动注销，该对象的实例在onReceive被调用之后就会在任意时间内被销毁

注意：BroadcastReceiver的生命周期只有10秒，不要在OnReceive方法内执行任何耗时操作，若要执行耗时操作可以通过发送Intent给Service操作，切记不能去开子线程，由于BroadcastReceiver只有10秒的生命周期，当宿主线程挂了，那么子线程也自动销毁了。

------------------------------------2015-12-02 11:01:34 ------------------------------------

#### 1. PendingIntent相关 ####
介绍：PendingIntent就是Intent的延迟包装类

作用：在当前的Activity不立即使用此Intent进行处理，而将此Intent封装后传递给其他的Activity程序，而其他的Activity程序在需要使用此Intent时才进行操作

PendingIntent Flag：

	FLAG_CANCEL_CURRENT：重新生成一个新的PendingIntent对象
	FLAG_NO_CREATE：当PendingIntent不存在，返回null（不创建）
	FLAG_ONE_SHOT：用于表示该PendingIntent只能被使用一次
	FLAG_UPDATE_CURRENT：用于更新已存在的PendingIntent携带的Intent中的信息
	FLAG_IMMUTABLE：不可变的PendingIntent

#### 2. 集合遍历相关 ####

List集合遍历方法:
	
	/**遍历list集合*/ 
	public void traversingList(List<String> list){  
		        //方法一：通过下标遍历  
		        for (int i = 0; i < list.size(); i++) {  
		            System.out.println(list.get(i));  
		        }  
		        //方法二：Iterator迭代器遍历  
		        Iterator<String> itr = list.iterator();  
		        while(itr.hasNext()){  
		            String str = itr.next();  
		            System.out.println(str);  
		        }  
		    }  	

Set集合遍历方法：

	/**遍历Set集合*/  
    public void traversingSet(Set<String> set){  
        //方法一：Iterator迭代器遍历  
        Iterator<String> itr = set.iterator();  
        while(itr.hasNext()){  
            String str = itr.next();  
            System.out.println(str);  
        }  
          
        //方法二：通过增强型for循环遍历  
        //注：Set集合中不存在下标，因此无法通过下标遍历，对于Java编译器而言，方法一和方法二是等价的  
        for (String str : set) {  
            System.out.println(str);  
        }  
    }  

Map集合遍历方法：

	/**遍历Map集合*/  
    public void traversingMap(Map<String,String> map){  
        //方法一：通过Entry遍历<迭代Entry>  
        for(Entry<String, String> entry : map.entrySet()) {  
            System.out.println(entry.getKey()+"："+entry.getValue());  
        }  
        //方法二：通过Set集合遍历<迭代Set>  
        for(String key: map.keySet()){  
            System.out.println(key + "：" + map.get(key));  
        }  
    }  

#### 3. Dialog相关 ####
（1）多按钮对话框

	/**
     * 多按钮对话框
     */
    private void dialog1() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("确定对话框");
        builder.setMessage("测试对话框");
        builder.setPositiveButton("好评", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        });
        builder.setNegativeButton("差评", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        });
        builder.setNeutralButton("点赞", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        });
        builder.setIcon(R.drawable.ic_launcher);
        builder.show();
    }

（2）单选对话框

	/**
     * 单选对话框
     */
    private void dialog2() {
        String items[] = {"item1", "item2", "item3", "item4", "item5"};
        AlertDialog.Builder builder = new AlertDialog.Builder(this);

        int position = 0;//默认单选的位置

        builder.setTitle("单选对话框").setSingleChoiceItems(items, position, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                //TODO 实现自己的需求
            }
        }).show();
    }

（3）多选对话框

	/**
     * 多选对话框
     */
    private void dialog3() {
        //默认多选的状态
        boolean b[] = {false, false, true, false, false};
        String items[] = {"item1", "item2", "item3", "item4", "item5"};
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("多选对话框").setMultiChoiceItems(items, b, new DialogInterface.OnMultiChoiceClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which, boolean isChecked) {
                //TODO 实现自己的需求
            }
        }).show();
    }

（4）列表对话框

	/**
     * 列表对话框
     */
    private void dialog4() {
        String items[] = {"item1", "item2", "item3", "item4", "item5", "item6", "item7", "item8"};
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("列表对话框").setItems(items, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                //TODO 实现自己的需求
            }
        }).show();
    }
 
（5）添加自定义布局对话框

	/**
     * 添加自定义布局对话框
     */
    private void dialog5() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setIcon(R.drawable.ic_launcher).setTitle("添加布局对话框").
                setPositiveButton("yes", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {

                    }
                }).setNegativeButton("no", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        }).setMessage("这个内容根据自己需求，可要可不要");
        View view = LayoutInflater.from(this).inflate(R.layout.items, null);
        AlertDialog dialog = builder.create();
        dialog.setView(view);
        dialog.show();
    }

（6）PopupWindow类型的对话框

	private void dialog9() {
        View view = LayoutInflater.from(this).inflate(R.layout.dialog, null);
        //实例化pw对话框并且设置布局，大小，是否能获得焦点，第三个参数为true可以获得焦点，一般设置为true即可
        final PopupWindow pw = new PopupWindow(view, WindowManager.LayoutParams.MATCH_PARENT,
                WindowManager.LayoutParams.WRAP_CONTENT, true);
        //对话框是否获得焦点，这是为true时，点击pw以外的地方不响应，否则点击响应。
        pw.setFocusable(true);
        //原本是点击pw之外窗口消失，但是实验发现无效，只有设置了setBackgroundDrawable时才有效。
		//        pw.setOutsideTouchable(true);
        //设置pw对话框的动画效果
        pw.setAnimationStyle(R.style.Animation1);
        //这个很重要，设置pw对话框背景为全透明，只有设置这个，点击pw对话框以外内容时，对话框消失，并且对话框能响应back还回键。
        pw.setBackgroundDrawable(new ColorDrawable(0x00000000));
        //设置对话框的位置偏移量
        int x = 0;
        int y = getStatusBarHeight() + getActionBarHeight();
        //相对于父控件显示对话框
        pw.showAtLocation(parentView, Gravity.TOP, x, y);

		//        showAsDropDown(View anchor)：相对某个控件的位置（正左下方），无偏移
		
		//        showAsDropDown(View anchor, int xoff, int yoff)：相对某个控件的位置，有偏移
		
		//        showAtLocation(View parent, int gravity, int x, int y)：相对于父控件的位置（例如正中央Gravity.CENTER，下方Gravity.BOTTOM等），可以设置偏移或无偏移

        //pw对话框设置半透明背景。原理：pw显示时，改变整个窗口的透明度为0.7，当pw消失时，透明度为1
        final WindowManager.LayoutParams params = MainActivity.this.getWindow().getAttributes();
        params.alpha = 0.7f;
        MainActivity.this.getWindow().setAttributes(params);

        view.findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                isExit = true;
                pw.dismiss();
                params.alpha = 1f;
                MainActivity.this.getWindow().setAttributes(params);
            }
        });

        //pw对话框消失监听事件
        pw.setOnDismissListener(new PopupWindow.OnDismissListener() {
            @Override
            public void onDismiss() {
                params.alpha = 1f;
                MainActivity.this.getWindow().setAttributes(params);
            }
        });

    }
1.PopupWindow焦点问题： 
setFocusable(boolean b); true时对话框之外不可点击，反之，系统默认设置为true，为防止意外，一般setFocusable(true).

2.PopupWindow不响应Back返回键问题： 
解决这个问题只需设置对话框背景透明pw.setBackgroundDrawable(new ColorDrawable(0x00000000));设置完之后点击对话框之外的地方pw对话框同样关闭。值得注意：设置pw.setOutsideTouchable(false)无效。

3.PopupWindow背景问题： 
我这里使用的方法是：pw对话框弹出时，改变当前Window窗口透明度 alpha属性，当对话框关闭时将当前窗口透明度修改回来即可。具体可参考上面代码实现。

#### 4. Android中Popupwindow和Dialog的区别 ####

（1）Popupwindow在显示之前一定要设置宽高，Dialog无此限制。

（2）Popupwindow默认不会响应物理键盘的back，除非显示设置了popup.setFocusable(true);而在点击back的时候，Dialog会消失。

（3）Popupwindow不会给页面其他的部分添加蒙层，而Dialog会。

（4）Popupwindow没有标题，Dialog默认有标题，可以通过dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);取消标题

（5）二者显示的时候都要设置Gravity。如果不设置，Dialog默认是Gravity.CENTER。

（6）二者都有默认的背景，都可以通过setBackgroundDrawable(new ColorDrawable(android.R.color.transparent));去掉。

最本质的差别就是：AlertDialog是非阻塞式对话框：AlertDialog弹出时，后台还可以做事情；而PopupWindow是阻塞式对话框：PopupWindow弹出时，程序会等待，在PopupWindow退出前，程序一直等待，只有当我们调用了dismiss方法的后，PopupWindow退出，程序才会向下执行。
这两种区别的表现是：AlertDialog弹出时，背景是黑色的，但是当我们点击背景，AlertDialog会消失，证明程序不仅响应AlertDialog的操作，还响应其他操作，其他程序没有被阻塞，这说明了AlertDialog是非阻塞式对话框；PopupWindow弹出时，背景没有什么变化，但是当我们点击背景的时候，程序没有响应，只允许我们操作PopupWindow，其他操作被阻塞。


#### 5. LayoutInflater 相关 ####
（1）LayoutInflater的作用：把xml类型的布局转化成相应的View对象

（2）getViewById和getLayoutInflater().inflate的区别：

LayoutInflater是用来找res/layout/下的xml布局文件，并且实例化；
而findViewById()是找xml布局文件下的具体widget控件(如Button、TextView等)。对于一个没有被载入或者想要动态载入的界面，都需要使用LayoutInflater.inflate()
来载入；对于一个已经载入的界面，就可以使用Activiyt.findViewById()方法来获得其中的界面元素。

（3）获得 LayoutInflater 实例的三种方式：

1>.LayoutInflater inflater = getLayoutInflater(); //调用Activity的getLayoutInflater()

2>.LayoutInflater localinflater =(LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

3>.LayoutInflater inflater = LayoutInflater.from(context);

注意：这三种方式最终本质是都是调用的Context.getSystemService()。

我们每次创建的activity.xml文件时，系统会自动在这个xml的最外层嵌套一层FrameLayout，作用是为了使xml的最外层布局的layout_width和layout_height起作用。因为这两个属性有效的前提是必须有父级容器。

---------------------2015-12-03 10:16:43 ---------------------------

#### 1.ListView的setSelection()和setSelectionFromTop() ####

setSelectionFromTop()的作用是设置ListView选中的位置，同时在Y轴设置一个偏移量（padding值），setSelection()让ListView定位到指定Item的位置，这两个方法的区别是 setSelection()内部就是调用了setSelectionFromTop()，只不过是Y轴的偏移量是0而已。

#### 2.屏蔽返回键，home键以及其他实体按键 ####

屏蔽返回键:

	public boolean onKeyDown(int keyCode, KeyEvent event) {
	
	    switch (keyCode) {
	        case KeyEvent.KEYCODE_BACK:
	        return true;
	    }
	    return super.onKeyDown(keyCode, event);
	}

屏蔽home键:

	public void onAttachedToWindow() {
	    this.getWindow().setType(WindowManager.LayoutParams.TYPE_KEYGUARD);
	    super.onAttachedToWindow();
	}

屏蔽其他实体按键:

	switch (keyCode) {
	    case KeyEvent.KEYCODE_HOME:
	        return true;
	    case KeyEvent.KEYCODE_CALL:
	        return true;
	    case KeyEvent.KEYCODE_SYM:
	        return true;
	    case KeyEvent.KEYCODE_VOLUME_DOWN:
	        return true;
	    case KeyEvent.KEYCODE_VOLUME_UP:
	        return true;
	    case KeyEvent.KEYCODE_STAR:
        	return true;
	}


#### 3.onActivityResult() 和onResume()的调用顺序问题 ####
onActivityResult()发生在onResume()之前.

调用完一个存在的activity之后，onActivityResult将会返回以下数据：你调用时发出的requestCode、被调用activity的结果标志resultCode（如RESULT_OK）和其他的额外数据。我们期望的都是得到RESULT_OK，表示调用成功，但是当被调用activity什么也没返回，或者调用过程中发生崩溃时，resultCode的值会为RESULT_CANCELED，重新回到调用activity时会马上执行onActivityResult方法，然后才是onResume()方法。

#### 4.自定义menu ####

	@Override  
	/** 
	* 创建MENU 
	*/  
	public boolean onCreateOptionsMenu(Menu menu) {  
		menu.add("menu");// 必须创建一项  
		return super.onCreateOptionsMenu(menu);  
	}  
	  
	@Override  
	/** 
	* 拦截MENU 
	*/  
	public boolean onMenuOpened(int featureId, Menu menu) {  
		if (menuDialog == null) {  
		    menuDialog = new AlertDialog.Builder(this).setView(menuView).show();  
		} else {  
		    menuDialog.show();  
		}  
		return false;// 返回为true 则显示系统menu  
	}  
	  
	menuGrid = (GridView) menuView.findViewById(R.id.gridview);  
	menuGrid.setAdapter(getMenuAdapter(menu_name_array, menu_image_array));  
	/** 监听menu选项 **/  
	menuGrid.setOnItemClickListener(new OnItemClickListener() {  
	public void onItemClick(AdapterView<?> arg0, View arg1, int arg2,  
	long arg3) {  
	switch (arg2) {  
		case ITEM_SEARCH:// 搜索  
		  
		break;  
		case ITEM_FILE_MANAGER:// 文件管理  
		  
		break;  
		case ITEM_DOWN_MANAGER:// 下载管理  
		  
		break;  
		case ITEM_FULLSCREEN:// 全屏  
		  
		break;  
		case ITEM_MORE:// 翻页  
		if (isMore) {  
			menuGrid.setAdapter(getMenuAdapter(menu_name_array2,  
			menu_image_array2));  
			isMore = false;  
		} else {// 首页  
			menuGrid.setAdapter(getMenuAdapter(menu_name_array,  
			menu_image_array));  
			isMore = true;  
		}  
			menuGrid.invalidate();// 更新menu  
			menuGrid.setSelection(ITEM_MORE);  
			break;  
		}  
	 }  
	}); 

-------------------2015-12-04 10:13:51 --------------------------------

#### 1.相机调用时，拍出来照片是反正的，加入下面这句话 ####

 android:configChanges ="keyboardHidden|orientation|screenSize">
 

---------------------------------------------2015-12-07 09:55:24 -----------------------------------

#### 1.Notification通知栏####
 创建一条普通的Notification

	NotificationCompat.Builder mBuilder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
	NotificationManager mNotificationManager = （NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
	// mId可以用来更新这个Notification
	mNotificationManager.notify(mId, mBuilder.build());

 为notification设置宽视图样式

	NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(this)
	    .setSmallIcon(R.drawable.notification_icon)
	    .setContentTitle("Event tracker")
	    .setContentText("Events received")
	NotificationCompat.InboxStyle inboxStyle = new NotificationCompat.InboxStyle();
	String[] events = new String[6];
	// Sets a title for the Inbox style big view
	inboxStyle.setBigContentTitle("Event tracker details:");
	...
	// Moves events into the big view
	for (int i=0; i < events.length; i++) {
	    inboxStyle.addLine(events[i]);
	}
	// Moves the big view style object into the notification object.
	mBuilder.setStyle(inBoxStyle);
	...
	// Issue the notification here.

#### 2. Notification的flag####

如何使自己的Notification像Android QQ一样能出现在 “正在运行的”栏目下面：
设置Notification.flags = Notification.FLAG_ONGOING_EVENT;

当用户点击 Clear 之后，能够清除该通知：
设置 n.flags 为 Notification.FLAG_AUTO_CANCEL

如果要以Intent启动一个Activity时，一定要设置：
intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP|Intent.FLAG_ACTIVITY_NEW_TASK);

	Intent.FLAG_ACTIVITY_CLEAR_TOP ：如果在当前Task中，有要启动的Activity，那么把该Acitivity之前的所有Activity都关掉，并把此Activity置前以避免创建Activity的实例。
	
	Intent.FLAG_ACTIVITY_NEW_TASK ：系统会检查当前所有已创建的Task中是否有该要启动的Activity的Task，若有，则在该Task上创建Activity，若没有则新建具有该Activity属性的Task，并在该新建的Task上创建Activity。

#### 3.改变 Notification 在“正在运行的”栏目下面的布局 ####

创建 RemoteViews 并赋给 Notification.contentView ，再把 PendingIntent 赋给 Notification.contentIntent。

	PendingIntent contentIntent = PendingIntent.getActivity(
    arg0.getContext(),
    R.string.app_name,
    i,
    PendingIntent.FLAG_UPDATE_CURRENT);
	RemoteViews rv = new RemoteViews(Main.this.getPackageName(), R.layout.notification_view);
	rv.setImageViewResource(R.id.image, R.drawable.chat);
	rv.setTextViewText(R.id.text, "Hello,there,I'm john.");
	n.contentView = rv;
	n.contentIntent = contentIntent;
	         
	nm.notify(R.string.app_name, n);

注意，如果使用了contentView，那么便不要使用Notification.setLatestEventInfo。如果setLatestEventInfo在赋给 Notification.contentView 的代码之后，那么contentView的效果将被覆盖，显示的便是 setLatestEventInfo 的效果；如果 setLatestEventInfo 在 Notification.contentView 的代码之前，那么显示的便是 Notification.contentView 的效果，也就是说不管你想要setLatestEventInfo 或 contentView 的自定义效果，请保证始终只有一句设置代码，因为在最后一句绑定的时候，之前的设置contentView或setLatestEventInfo的代码都是完全没有必要的。

#### 4.单例模式 ####

单例模式介绍：单例模式属于设计模式中的 “创建型模式”（控制对象创建过程）。

单例模式的特点：使某个class在整个虚拟机中只能存在一个实例。

单例模式的实现：

	第一种，基本的延迟创建方法：

		package com.example.hello;
		public class Singleton {
		    private static Singleton instance;
		    
		    private Singleton() { }
		    
		    public static Singleton getInstance () {
		        if (instance == null) {
		            instance = new Singleton();
		        }
		        return instance;
		    }
		    
		    public void sayHelloToWorld () {
		        System.out.print("Hello, world");
		    }
		}
		注意的点：
			（1）构造函数被声明为private。
			（2）想要访问这个类，只能通过Singleton.getInstance()来完成。
		这种方式的缺点：
			线程A首现调用了Singleton.getInstance()，它发现instance为null，于是它去创建对象，然后给instance赋值，但是在创建对象或者赋值完成之前的某个时刻，线程B也调用了Singleton.getInstance()去创建对象，因为A线程的调用过程并没有结束，所以此时线程B也会成功创建对象，这样就不能保证单例了，所以使用的时候一定要注意访问线程问题。

	第二种，同步的延迟创建方法

		package com.example.hello;
		public class Singleton {
		    private static Singleton instance;
		    
		    private Singleton() { }
		    
		    public static synchronized Singleton getInstance () {
		        if (instance == null) {
		            instance = new Singleton();
		        }
		        return instance;
		    }
		    
		    public synchronized void sayHelloToWorld () {
		        System.out.print("Hello, world");
		    }
		}
		和第一种方法的唯一区别就是在每个方法前面都加了synchronized关键字，
		弥补了第一种方法所造成的线程不安全的缺点。（推荐使用）
		

	第三种，双重检查加锁方法

		package com.example.hello;
		public class Singleton {
		    private static volatile Singleton instance;
		    private Singleton() {
		    }
		    public static Singleton getInstance() {
		        if (instance == null) {
		            synchronized (Singleton.class) {
		                if (instance == null) {
		                    instance = new Singleton();
		                }
		            }
		        }
		        return instance;
		    }
		    public synchronized void sayHelloToWorld() {
		        System.out.print("Hello, world");
		    }
		}
		这种方法相对于前一种方法而言，可以稍微降低一点加锁的开销。
		注意的位置：
			（1）instance是要被声明为volatile的。（如果不声明为volatile，则可能会出现可见性问题）【用volatile修饰的变量，线程在每次使用变量的时候，都会读取变量修改后的最新值】
		

-----------------------------2015-12-08 09:33:14 ------------------------------------------------

#### 1.fragment和activity的联系 ####

（1）如果Activity是暂停状态，其中所有的Fragment都是暂停状态；如果Activity是stopped状态，这个Activity中所有的Fragment都不能被启动；如果Activity被销毁，那么它其中的所有Fragment都会被销毁。

（2）如果继承自ListFragment，onCreateView()默认的实现会返回一个ListView，所以不用自己实现。

（3）Fragment的返回栈由Activity管理；而Activity的返回栈由系统管理。


----------------------------------2015-12-09 09:53:50 ---------------------------------


#### 1.给TextView中的文字加上链接 ####
项目上需要给TextView中特定格式文本加上”超链接“，让用户可以点击链接后，可以开启关心该uri的Activity。
如给”abc12345“加上链接，使其点击后，开启Activity，该Activity意图过滤器关心的uri scheme为”abc://”。
利用正则表达式，匹配出”abc12345“。
利用Linkify类给文字加上链接到”abc://abc12345”:

		public void addLink(TextView text){
		    String localDesc = description;
		    text.setText(localDesc);
		    Linkify.addLinks(text, Pattern.compile("(ac\\d{5,})", Pattern.CASE_INSENSITIVE),"abc://");
		}
这样用户点击后，内部会startActivity(intent); 
该intent的data为 “abc://abc12345”，这样注册了该scheme IntentFilter的Activity就会被开启了。
当然，也可以给符合http协议的url自动加上超链接：

	Pattern http = Pattern.compile("(http://(?:[a-z0-9.-]+[.][a-z]{2,}+(?::[0-9]+)?)(?:/\\S*)?)",Pattern.CASE_INSENSITIVE);
    Linkify.addLinks(text,http,"http://");

用户点击后，开启的Intent 包含data为 “http://…”，这样就会开启浏览器，浏览该网址了。

#### 2.给TextView加上个“展开/收起”的功能 ####

给TextView限制maxLine为4行，当getLineCount() >=4时，显示“展开”按钮。

	mTextView.setText("large text");
	mTextView.post(new Runnable() {
	
	    @Override
	    public void run() {
	
	    int lineCount = mTextView.getLineCount();
	    if(lineCount >=4)
	        showDetailButton();
	    }
	});

#### 3.BaseActivity的写法 ####

	/**
	 * 应用程序Activity的基类
	 * 
	 * @author kymjs
	 * @version 1.0
	 * @created 2013-11-24
	 */
	public abstract class BaseActivity extends Activity implements
	        OnClickListener {
	    private static final int ACTIVITY_RESUME = 0;
	    private static final int ACTIVITY_STOP = 1;
	    private static final int ACTIVITY_PAUSE = 2;
	    private static final int ACTIVITY_DESTROY = 3;
	 
	    public int activityState;
	 
	    // 是否允许全屏
	    private boolean mAllowFullScreen = true;
	 
	    public abstract void initWidget();
	 
	    public abstract void widgetClick(View v);
	 
	    public void setAllowFullScreen(boolean allowFullScreen) {
	        this.mAllowFullScreen = allowFullScreen;
	    }
	 
	    @Override
	    public void onClick(View v) {
	        widgetClick(v);
	    }
	 
	    /***************************************************************************
	     * 
	     * 打印Activity生命周期
	     * 
	     ***************************************************************************/
	 
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        AppLog.debug(this.getClass() + "---------onCreat ");
	        // 竖屏锁定
	        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
	        if (mAllowFullScreen) {
	            requestWindowFeature(Window.FEATURE_NO_TITLE); // 取消标题
	        }
	        AppManager.getAppManager().addActivity(this);
	        initWidget();
	    }
	 
	    @Override
	    protected void onStart() {
	        super.onStart();
	        AppLog.state(this.getClass(), "---------onStart ");
	    }
	 
	    @Override
	    protected void onResume() {
	        super.onResume();
	        activityState = ACTIVITY_RESUME;
	        AppLog.state(this.getClass(), "---------onResume ");
	    }
	 
	    @Override
	    protected void onStop() {
	        super.onResume();
	        activityState = ACTIVITY_STOP;
	        AppLog.state(this.getClass(), "---------onStop ");
	    }
	 
	    @Override
	    protected void onPause() {
	        super.onPause();
	        activityState = ACTIVITY_PAUSE;
	        AppLog.state(this.getClass(), "---------onPause ");
	    }
	 
	    @Override
	    protected void onRestart() {
	        super.onRestart();
	        AppLog.state(this.getClass(), "---------onRestart ");
	    }
	 
	    @Override
	    protected void onDestroy() {
	        super.onDestroy();
	        activityState = ACTIVITY_DESTROY;
	        AppLog.state(this.getClass(), "---------onDestroy ");
	        AppManager.getAppManager().finishActivity(this);
	    }
	}

定义一个初始化Activity控件的抽象方法initWidget()；

像findviewbyid()这类代码就可以写在这里，不会影响代码结构了。这里需要提一点的是：setContentView（）方法一定要写在initWidget()里，而不能再写到oncreate里面了，看代码可以知道，initwidget方法是存在于super()中的，而如果再写到oncreate里，就相当于先调用了findview再去调用setcontentView，这样肯定会报空指针异常。

requestWindowFeature(Window.FEATURE_NO_TITLE); // 取消标题

对于这段代码，如果你要使用系统的ActionBar的时候，一点要记得调用setAllowFullScreen，设置为false，否则BaseActivity自动取消了ActionBar你又去使用，肯定也会出异常。

还有一点：Baseactivity已经实现了OnClickListener，所以子类无需再次实现，控件可以直接在initWidget里面setonclicklistener(this);然后在widgetClick(View v)中设置监听事件即可。


#### 4.android应用框架搭建------AppManager ####

在我们开发应用的时候，经常会有很多很多的activity，这时候，我们就需要一个activity栈来帮忙管理activity的finish和start。

	/**
	 * 应用程序Activity管理类：用于Activity管理和应用程序退出
	 * @author kymjs
	 * @version 1.0
	 */
	public class AppManager {
	    private static Stack<BaseActivity> activityStack;
	    private static AppManager instance;
	 
	    private AppManager() {
	    }
	 
	    /**
	     * 单实例 , UI无需考虑多线程同步问题
	     */
	    public static AppManager getAppManager() {
	        if (instance == null) {
	            instance = new AppManager();
	        }
	        return instance;
	    }
	 
	    /**
	     * 添加Activity到栈
	     */
	    public void addActivity(BaseActivity activity) {
	        if (activityStack == null) {
	            activityStack = new Stack<BaseActivity>();
	        }
	        activityStack.add(activity);
	    }
	 
	    /**
	     * 获取当前Activity（栈顶Activity）
	     */
	    public BaseActivity currentActivity() {
	        if (activityStack == null || activityStack.isEmpty()) {
	            return null;
	        }
	        BaseActivity activity = activityStack.lastElement();
	        return activity;
	    }
	 
	    /**
	     * 获取当前Activity（栈顶Activity） 没有找到则返回null
	     */
	    public BaseActivity findActivity(Class<?> cls) {
	        BaseActivity activity = null;
	        for (BaseActivity aty : activityStack) {
	            if (aty.getClass().equals(cls)) {
	                activity = aty;
	                break;
	            }
	        }
	        return activity;
	    }
	 
	    /**
	     * 结束当前Activity（栈顶Activity）
	     */
	    public void finishActivity() {
	        BaseActivity activity = activityStack.lastElement();
	        finishActivity(activity);
	    }
	 
	    /**
	     * 结束指定的Activity(重载)
	     */
	    public void finishActivity(Activity activity) {
	        if (activity != null) {
	            activityStack.remove(activity);
	            activity.finish();
	            activity = null;
	        }
	    }
	 
	    /**
	     * 结束指定的Activity(重载)
	     */
	    public void finishActivity(Class<?> cls) {
	        for (BaseActivity activity : activityStack) {
	            if (activity.getClass().equals(cls)) {
	                finishActivity(activity);
	            }
	        }
	    }
	 
	    /**
	     * 关闭除了指定activity以外的全部activity 如果cls不存在于栈中，则栈全部清空
	     * 
	     * @param cls
	     */
	    public void finishOthersActivity(Class<?> cls) {
	        for (BaseActivity activity : activityStack) {
	            if (!(activity.getClass().equals(cls))) {
	                finishActivity(activity);
	            }
	        }
	    }
	 
	    /**
	     * 结束所有Activity
	     */
	    public void finishAllActivity() {
	        for (int i = 0, size = activityStack.size(); i < size; i++) {
	            if (null != activityStack.get(i)) {
	                activityStack.get(i).finish();
	            }
	        }
	        activityStack.clear();
	    }
	 
	    /**
	     * 应用程序退出
	     */
	    public void AppExit(Context context) {
	        try {
	            finishAllActivity();
	            ActivityManager activityMgr = (ActivityManager) context
	                    .getSystemService(Context.ACTIVITY_SERVICE);
	            activityMgr.killBackgroundProcesses(context.getPackageName());
	            System.exit(0);
	        } catch (Exception e) {
	            System.exit(0);
	        }
	    }
	}

 我们可以这样定义一个Toast

	public static showMessage(String msg){
	    Toast.makeText(AppManager.getAppManager().currentActivity(), msg, Toast.LENGTH_SHORT).show();
	}

#### 5.android应用框架搭建------工具类（FileUtils） ####

	import java.io.File;
	import java.io.IOException;

	import android.os.Environment;
	 
	public class FileUtils {
	    /**
	     * 检测SD卡是否存在
	     */
	    public static boolean checkSDcard() {
	        return Environment.MEDIA_MOUNTED.equals(Environment
	                .getExternalStorageState());
	    }
	 
	    /**
	     * 获取文件保存点
	     */
	    public static File getSaveFile(String fileNmae) {
	        File file = null;
	        try {
	            file = new File(Environment.getExternalStorageDirectory()
	                    .getCanonicalFile() + "/" + fileNmae);
	        } catch (IOException e) {
	        }
	        return file;
	    }
	 
	    /**
	     * 从指定文件夹获取文件
	     */
	    public static File getSaveFile(String folder, String fileNmae) {
	        File file = new File(getSavePath(folder), fileNmae);
	        return file;
	    }
	 
	    /**
	     * 获取文件保存路径
	     */
	    public static String getSavePath(String folder) {
	        return Environment.getExternalStorageDirectory() + "/" + folder;
	    }
	}

#### 6.android应用框架搭建------工具类（StringUtils） ####

	import java.io.UnsupportedEncodingException;
	import java.security.MessageDigest;
	import java.security.NoSuchAlgorithmException;
	import java.text.ParseException;
	import java.text.SimpleDateFormat;
	import java.util.Date;
	import java.util.regex.Pattern;
	 
	import android.app.Activity;
	import android.content.Context;
	import android.content.pm.ApplicationInfo;
	import android.content.pm.PackageManager;
	import android.content.pm.PackageManager.NameNotFoundException;
	import android.os.Bundle;
	import android.telephony.TelephonyManager;
	 
	/**
	 * 字符串操作工具包
	 */
	public class StringUtils {
	    private final static Pattern emailer = Pattern
	            .compile("\\w+([-+.]\\w+)*@\\w+([-.]\\w+)*\\.\\w+([-.]\\w+)*");
	    private final static Pattern phone = Pattern
	            .compile("^((13[0-9])|170|(15[^4,\\D])|(18[0,5-9]))\\d{8}$");
	 
	    private final static ThreadLocal<SimpleDateFormat> dateFormater = new ThreadLocal<SimpleDateFormat>() {
	        @Override
	        protected SimpleDateFormat initialValue() {
	            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	        }
	    };
	 
	    private final static ThreadLocal<SimpleDateFormat> dateFormater2 = new ThreadLocal<SimpleDateFormat>() {
	        @Override
	        protected SimpleDateFormat initialValue() {
	            return new SimpleDateFormat("yyyy-MM-dd");
	        }
	    };
	 
	    /**
	     * 返回当前系统时间
	     */
	    public static String getDataTime(String format) {
	        SimpleDateFormat df = new SimpleDateFormat(format);
	        return df.format(new Date());
	    }
	 
	    /**
	     * 返回当前系统时间
	     */
	    public static String getDataTime() {
	        return getDataTime("HH:mm");
	    }
	 
	    /**
	     * 毫秒值转换为mm:ss
	     * 
	     * @author kymjs
	     * @param ms
	     */
	    public static String timeFormat(int ms) {
	        StringBuilder time = new StringBuilder();
	        time.delete(0, time.length());
	        ms /= 1000;
	        int s = ms % 60;
	        int min = ms / 60;
	        if (min < 10) {
	            time.append(0);
	        }
	        time.append(min).append(":");
	        if (s < 10) {
	            time.append(0);
	        }
	        time.append(s);
	        return time.toString();
	    }
	 
	    /**
	     * 将字符串转位日期类型
	     * 
	     * @return
	     */
	    public static Date toDate(String sdate) {
	        try {
	            return dateFormater.get().parse(sdate);
	        } catch (ParseException e) {
	            return null;
	        }
	    }
	 
	    /**
	     * 判断给定字符串时间是否为今日
	     * 
	     * @param sdate
	     * @return boolean
	     */
	    public static boolean isToday(String sdate) {
	        boolean b = false;
	        Date time = toDate(sdate);
	        Date today = new Date();
	        if (time != null) {
	            String nowDate = dateFormater2.get().format(today);
	            String timeDate = dateFormater2.get().format(time);
	            if (nowDate.equals(timeDate)) {
	                b = true;
	            }
	        }
	        return b;
	    }
	 
	    /**
	     * 判断给定字符串是否空白串 空白串是指由空格、制表符、回车符、换行符组成的字符串 若输入字符串为null或空字符串，返回true
	     */
	    public static boolean isEmpty(String input) {
	        if (input == null || "".equals(input))
	            return true;
	 
	        for (int i = 0; i < input.length(); i++) {
	            char c = input.charAt(i);
	            if (c != ' ' && c != '\t' && c != '\r' && c != '\n') {
	                return false;
	            }
	        }
	        return true;
	    }
	 
	    /**
	     * 判断是不是一个合法的电子邮件地址
	     */
	    public static boolean isEmail(String email) {
	        if (email == null || email.trim().length() == 0)
	            return false;
	        return emailer.matcher(email).matches();
	    }
	 
	    /**
	     * 判断是不是一个合法的手机号码
	     */
	    public static boolean isPhone(String phoneNum) {
	        if (phoneNum == null || phoneNum.trim().length() == 0)
	            return false;
	        return phone.matcher(phoneNum).matches();
	    }
	 
	    /**
	     * 字符串转整数
	     * 
	     * @param str
	     * @param defValue
	     * @return
	     */
	    public static int toInt(String str, int defValue) {
	        try {
	            return Integer.parseInt(str);
	        } catch (Exception e) {
	        }
	        return defValue;
	    }
	 
	    /**
	     * 对象转整
	     * 
	     * @param obj
	     * @return 转换异常返回 0
	     */
	    public static int toInt(Object obj) {
	        if (obj == null)
	            return 0;
	        return toInt(obj.toString(), 0);
	    }
	 
	    /**
	     * String转long
	     * 
	     * @param obj
	     * @return 转换异常返回 0
	     */
	    public static long toLong(String obj) {
	        try {
	            return Long.parseLong(obj);
	        } catch (Exception e) {
	        }
	        return 0;
	    }
	 
	    /**
	     * String转double
	     * 
	     * @param obj
	     * @return 转换异常返回 0
	     */
	    public static double toDouble(String obj) {
	        try {
	            return Double.parseDouble(obj);
	        } catch (Exception e) {
	        }
	        return 0D;
	    }
	 
	    /**
	     * 字符串转布尔
	     * 
	     * @param b
	     * @return 转换异常返回 false
	     */
	    public static boolean toBool(String b) {
	        try {
	            return Boolean.parseBoolean(b);
	        } catch (Exception e) {
	        }
	        return false;
	    }
	 
	    /**
	     * 判断一个字符串是不是数字
	     */
	    public static boolean isNumber(String str) {
	        try {
	            Integer.parseInt(str);
	        } catch (Exception e) {
	            return false;
	        }
	        return true;
	    }
	 
	    /**
	     * 获取AppKey
	     */
	    public static String getMetaValue(Context context, String metaKey) {
	        Bundle metaData = null;
	        String apiKey = null;
	        if (context == null || metaKey == null) {
	            return null;
	        }
	        try {
	            ApplicationInfo ai = context.getPackageManager()
	                    .getApplicationInfo(context.getPackageName(),
	                            PackageManager.GET_META_DATA);
	            if (null != ai) {
	                metaData = ai.metaData;
	            }
	            if (null != metaData) {
	                apiKey = metaData.getString(metaKey);
	            }
	        } catch (NameNotFoundException e) {
	 
	        }
	        return apiKey;
	    }
	 
	    /**
	     * 获取手机IMEI码
	     */
	    public static String getPhoneIMEI(Activity aty) {
	        TelephonyManager tm = (TelephonyManager) aty
	                .getSystemService(Activity.TELEPHONY_SERVICE);
	        return tm.getDeviceId();
	    }
	 
	    /**
	     * MD5加密
	     */
	    public static String md5(String string) {
	        byte[] hash;
	        try {
	            hash = MessageDigest.getInstance("MD5").digest(
	                    string.getBytes("UTF-8"));
	        } catch (NoSuchAlgorithmException e) {
	            throw new RuntimeException("Huh, MD5 should be supported?", e);
	        } catch (UnsupportedEncodingException e) {
	            throw new RuntimeException("Huh, UTF-8 should be supported?", e);
	        }
	 
	        StringBuilder hex = new StringBuilder(hash.length * 2);
	        for (byte b : hash) {
	            if ((b & 0xFF) < 0x10)
	                hex.append("0");
	            hex.append(Integer.toHexString(b & 0xFF));
	        }
	        return hex.toString();
	    }
	 
	    /**
	     * KJ加密
	     */
	    public static String KJencrypt(String str) {
	        char[] cstr = str.toCharArray();
	        StringBuilder hex = new StringBuilder();
	        for (char c : cstr) {
	            hex.append((char) (c + 5));
	        }
	        return hex.toString();
	    }
	 
	    /**
	     * KJ解密
	     */
	    public static String KJdecipher(String str) {
	        char[] cstr = str.toCharArray();
	        StringBuilder hex = new StringBuilder();
	        for (char c : cstr) {
	            hex.append((char) (c - 5));
	        }
	        return hex.toString();
	    }
	}

#### 7.软键盘弹出后屏幕适配 ####

需要对适配的Activity设置属性：windowSoftInputMode

属性介绍如下：

	adjustPan：（平移模式）整个软键盘把屏幕内容挤上去，键盘不会覆盖页面任何内容
	adjustResize：（压缩模式）压扁当前页面，给键盘留出合适的空间

#### 8. EditText的焦点和键盘的关系####
软键盘其实是一个Dialog。InputMethodService为我们的输入法创建了一个Dialog，并且对某些参数进行了设置，使之能够在底部或者全屏显示。当我们点击输入框时，系统会对当前的主窗口进行调整，以便留出相应的空间来显示该Dialog在底部，或者全屏。

关闭editText自动弹出键盘：

新方法：
在manifest.xml中对应activity添加：
	android:windowSoftInputMode="stateHidden"

第一种办法：阻断焦点

	在editText父控件下添加：
	android:focusable="true"   
	android:focusableInTouchMode="true"

第二种办法：直接关闭输入法

	private void closeInputMethod() {
	    InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
	    boolean isOpen = imm.isActive();
	    if (isOpen) {
	        // imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS);//没有显示则显示
	        imm.hideSoftInputFromWindow(mobile_topup_num.getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);
	    }
	}

	if语句里面的几个参数根据需要设置：
	1、方法一(如果输入法在窗口上已经显示，则隐藏，反之则显示)
	InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE); 
	imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS); 
	  
	2、方法二（view为接受软键盘输入的视图，SHOW_FORCED表示强制显示）
	InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE); 
	imm.showSoftInput(view,InputMethodManager.SHOW_FORCED); 
	[java] view plaincopy
	imm.hideSoftInputFromWindow(view.getWindowToken(), 0); //强制隐藏键盘 
	 
	3、调用隐藏系统默认的输入法
	((InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE)).hideSoftInputFromWindow(WidgetSearchActivity.this.getCurrentFocus().getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);  (WidgetSearchActivity是当前的Activity) 
	 
	4、获取输入法打开的状态
	InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE); 
	boolean isOpen=imm.isActive();//isOpen若返回true，则表示输入法打开 


EditText失去焦点时关闭软键盘：

	/** 
     * 某个特定view获得焦点时，关闭软键盘 
     * @param context 
     *            view所在activity 
     * @param view 
     *            当前activity中获取焦点的view 
     */  
    public void closeKeyboardForCommonAct(Context context, View view) {  
        InputMethodManager imm = (InputMethodManager) ((Activity) context)  
                .getSystemService(Context.INPUT_METHOD_SERVICE);  
        if (((Activity) context).getCurrentFocus().getWindowToken() != null) {  
            imm.hideSoftInputFromInputMethod(view.getWindowToken(),  
                    InputMethodManager.HIDE_NOT_ALWAYS);  
        }  
    }  

	// 点击空白处 软键盘消失  
	@Override  
	public boolean onTouchEvent(MotionEvent event) {  
	    InputMethodManager imm = (InputMethodManager) getSystemService(INPUT_METHOD_SERVICE);  
	    return imm.hideSoftInputFromWindow(this.getCurrentFocus().getWindowToken(), 0);  
	} 

#### 9.自定义软键盘的Enter键 ####
把EditText的Ime Options属性设置成不同的值，Enter键上可以显示不同的文字或图案

	actionNone : 回车键，按下后光标到下一行
	actionGo ： Go，
	actionSearch ： 一个放大镜
	actionSend ： Send
	actionNext ： Next
	actionDone ： Done，隐藏软键盘，即使不是最后一个文本输入框

#### 10. Activity与Service通信的方式 ####

（1）如果Activity与Service在同进程内，那么直接使用Binder绑定的方法进行通信;
（2）如果Activity与Service在不同进程中，但Activity只需要向Service发送信息而不需要从Service中获取信息，那么使用Handler/Messenger的方法进行通信会比较方便;
（3）如果Activity需要从另一进程中的Service获取信息，那么必须使用AIDL的方法进行通信。

#### 11.Android 开启和关闭wifi ####
需要申请的权限：

	android.permission.ACCESS_WIFI_STATE 
	android.permission.CHANGE_WIFI_STATE 
	android.permission.WAKE_LOCK 

	//获取WifiManager
	wifiManager = (WifiManager) this.getSystemService(Context.WIFI_SERVICE);  
	//开启、关闭wifi
	if (wifiManager.isWifiEnabled()) {  
		wifiManager.setWifiEnabled(false);  
	} else {  
		wifiManager.setWifiEnabled(true);  
	}


#### 12.activity的启动模式 ####

（1）standard：不管有没有已存在的实例，都生成新的实例。
	5.0之前：总是在调用者的Task生成新的实例。
	![](http://i.imgur.com/hyIbi4t.png)
	5.0之后：如果跨应用之间启动Activity，会创建一个新的Task，新生成的Activity就会放入刚创建的Task中。
	![](http://i.imgur.com/Ou9GD3U.jpg)

	使用场景：standard这种启动模式适合于撰写邮件Activity或者社交网络消息发布Activity。

（2）singleTop：在栈结构中寻找是否有实例正位于栈顶，如果有则不再生成新的，而是使用当前的这个Activity实例，并调用这个实例的onNewIntent方法。（不能保证只有一个实例）

	![](http://i.imgur.com/i8XHcN5.png)
	
	使用场景：关于singleTop一个典型的使用场景就是搜索功能。假设有一个搜索框，每次搜索查询都会将我们引导至SearchActivity查看结果，为了更好的交互体验，我们在结果页顶部也放置这样的搜索框。

（3）singleTask：如果有对应的Activity实例，则使此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象显示。

（4）singleInstance：启用一个新的栈结构，将Acitvity放置于这个新的栈结构中，并保证不再有其他Activity实例进入。

[http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)
		
---------------------------------------------------------2015-12-10 14:24:14 --------------------------------------------

#### 1. Android activity的setResult()在什么时候调用 ####

activity返回result是在被finish的时候，也就是说调用setResult()方法必须在finish()之前。
如果在如下方法中调用setResult()也有可能不会返回成功： onPause()， onStop()， onDestroy(),
因为这些方法调用不一定是在finish之前的，当然在onCreate()就调用setResult肯定是在finish之前的。


#### 2.收集奔溃日志 ####

[http://droidyue.com/blog/2015/12/06/practise-about-crash-in-android/?droid_refer=ninki_posts](http://droidyue.com/blog/2015/12/06/practise-about-crash-in-android/?droid_refer=ninki_posts)

#### 3.ANR出现的原因及解决办法 ####

出现场景：

	（1）主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
	
	（2）主线程中存在耗时的计算
	
	（3）主线程中错误的操作，比如Thread.wait或者Thread.sleep等
	
	（4）应用在5秒内未响应用户的输入事件（如按键或者触摸）
	
	（5）BroadcastReceiver未在10秒内完成相关的处理

如何避免：

基本的思路就是将IO操作在工作线程来处理，减少其他耗时操作和错误操作。

	（1）使用AsyncTask处理耗时IO操作。
	
	（2）使用Thread或者HandlerThread时，调用Process.setThreadPriority
		(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，
		因为默认Thread的优先级和主线程相同。
	
	（3）使用Handler处理工作线程结果，而不是使用Thread.wait()或者Thread.sleep()来阻塞主线程。
	
	（4）Activity的onCreate和onResume回调中尽量避免耗时的代码。
	
	（5）BroadcastReceiver中onReceive代码也要尽量减少耗时，建议使用IntentService处理。

如何定位：

	如果开发机器上出现问题，我们可以通过查看/data/anr/traces.txt即可，最新的ANR信息在最开始部分。
	我们从stacktrace中即可找到出问题的具体行数。

#### 4.为Android程序申请权限注意 ####
因为有些权限隐式地需要feature,即当你显示使用uses-permission,会默认地为程序加入uses-feature. 
而Android以及Google Play判断是否可以安装和现实的依据是,设备包含的system features是否完全包含程序申请的全部features. 只有在全部满足了程序需要的feature的设备上才可以展示并安装.

解决问题:如何加权限,不减少支持设备：

如果你增加的权限并且及引入的feature不是必须使用的,可以显示地将该feature设置为不需要.继续上面的例子.在manifest中加入

	<uses-feature android:name="android.hardware.camera" android:required="false" />

--------------------------2015-12-14 11:27:08 --------------------------------

#### 1.onTouch()和 onClick() ####

（1） onTouch()先于onCLick()执行。

		onTouch()返回false，表示事件没有被消费，会继续传下去

		onTouch()返回true，表示事件被消费，到这里截止

（2） onTouch()执行后会把事件传递给onClick()。

#### 2.onTouch和onTouchEvent有什么区别，又该如何使用？ ####
这两个方法都是在View的dispatchTouchEvent中调用的，onTouch优先于onTouchEvent执行。如果在onTouch方法中通过返回true将事件消费掉，onTouchEvent将不会再执行。

另外需要注意的是，onTouch能够得到执行需要两个前提条件，第一mOnTouchListener的值不能为空，第二当前点击的控件必须是enable的。因此如果你有一个控件是非enable的，那么给它注册onTouch事件将永远得不到执行。对于这一类控件，如果我们想要监听它的touch事件，就必须通过在该控件中重写onTouchEvent方法来实现。

![](http://i.imgur.com/DPch8Hi.jpg)

![](http://i.imgur.com/PSKrVMz.jpg)


#### 3.为什么图片轮播器里的图片使用Button而不用ImageView？ ####

在图片轮播器里使用Button，主要就是因为Button是可点击的，而ImageView是不可点击的。如果想要使用ImageView，可以有两种改法。第一，在ImageView的onTouch方法里返回true，这样可以保证ACTION_DOWN之后的其它action都能得到执行，才能实现图片滚动的效果。第二，在布局文件里面给ImageView增加一个android:clickable="true"的属性，这样ImageView变成可点击的之后，即使在onTouch里返回了false，ACTION_DOWN之后的其它action也是可以得到执行的。

#### 4.Android事件分发机制 ####
1. Android事件分发是先传递到ViewGroup，再由ViewGroup传递到View的。

2. 在ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouchEvent方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。

3. 子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。

![](http://i.imgur.com/WSwjp9x.png)

#### 5.组合模式 ####

组合：将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。


-------------------------------------------2015-12-15 11:57:17 ------------------------------------------

#### 1.Git使用全流程 ####

一、还没有远程库

	远程服务器：已GitHub为例，其他远程服务器类似，只需要更改相应的库地址就行
		（1）在github上创建库
		（2）复制库地址
		（3）定位到本地服务器目录，在要上传的项目根目录下右键打开git bash
		（4）git init

![](http://i.imgur.com/mnZ98Ln.png)

		（5）git remote add origin 刚刚复制的地址

![](http://i.imgur.com/iYoqtgJ.png)
 
		（6）git add . 提交工程内所有的文件到本地库

![](http://i.imgur.com/MMfSZpa.png)

		（7）git commit -m "提交描述信息" 

![](http://i.imgur.com/I4A9nYB.png)

		（8）git push -u origin master（由于远程库是空的，第一次推送master分支时，加上了-u参数）

![](http://i.imgur.com/d3v1KM4.png)

		（10）刷新github，发现已经提交了代码。

![](http://i.imgur.com/60gLTVP.png)

		（11）当修改或增加新的文件时，先上传到本地服务器上，然后拉取远程库代码处理冲突，最后再上传到远程库。命令如下：
			git pull --rebase origin master 
			（如果有新增文件，先add 
				git add 文件1.xxx 文件2.xxx ... ...）
			git commit -m "提交描述信息"
			git push orign master
	

二、已经有远程库

	步骤一：
		git init
		git remote add origin 远程库地址
		git clone 远程库地址
	步骤二：
		其他操作与上面的第（6）一样

如果需要删除git本地库，那么把项目目录下的.git文件夹删除即可。如果需要重新使用，再次git init。

#### 2.比较 GET 与 POST ####

![](http://i.imgur.com/R79549D.png)



---------------------------------------------------2015-12-17 09:33:20 ----------------------------

#### 1.View的绘制流程  ####

经历的三个主要方法：

	onMeasure()、onLayout()、onDraw()

	onMeasure()：用于测量视图的大小。
	onLayout()：用于给视图进行布局，也就是确定视图的位置。
 	onDraw()：真正地开始对视图进行绘制。ViewRoot中的代码会继续执行并创建出一个Canvas对象，然后调用View的draw()方法来执行具体的绘制工作。


#### 2.i++和++i 的区别 ####

根本区别是语义上的区别，
i++返回值是i，++i的返回值是i+1，在for循环中他们的结果是一样的，
如果没有用到返回值的话，区别在于效率。

- 若i是内置的数值类型，两者效率完全一样


- 若i是一些自定义的类，如iterator，++i的效率  > = i++的效率（++i得到左值，i++得到右值，由于++i是得到的是左值，直接原地操作，不需要要临时变量保存结果，效率高点）

![](http://i.imgur.com/oyvQCK6.png)

#### 3。AIDL的作用 ####
需要在不同进程间实现通信，就需要用到AIDL（Android Interface Definition Language）技术去完成。

--------------------------2015-12-18 09:29:59 -------------------------------------------
#### 1.adb常用命令合辑 ####
1、adb常用命令



- 列出所有连接的android设备。

		adb devices 
- 以下命令都是对单个devices而言，如果存在多个devices的话，下面的命令都需要将adb变为

		adb -s deviceId
- 启动adb服务，如果已经启动，不重复启动

		adb start-server
- 停止adb服务

		adb kill-server
- 安装应用
 
		adb install package名字或者*.apk
- 卸载应用, -k表示保留应用数据和缓存

		adb uninstall <-k> package
- 以root运行

		adb root
- 进入devices命令行模式，进入命令行模式，就是linux命令行了
 
		adb shell
- 进入devices命令行模式，并运行命令command

		adb shell command
- adb命令启动程序
	
		adb shell am start -n <package>/<package>.<activity>
- adb命令启动程序 Debug模式

		adb shell am start -D -n <package>/<package>.<activity>
- 将本地的文件传送到device上，如安装系统apk, adb push a.apk /system/app/
 
		adb push <local> <remote>
- 将device上的文件拉到本地，如将某个系统应用复制到d盘, adb pull /system/app/a.apk d:\\

		adb pull <remote> <local>
- 挂载devices，对devices拥有写权限
 
		adb remount
- 重启设备

		adb reboot
- 以刷机模式重启

		adb reboot -recovery


#### 2.Android常用目录讲解 ####
应用常用目录列表：

- /data/data/package_name/ 应用的数据目录，包括cache、databases、lib、shared_prefs，分别存放cache、数据库、lib、SharedPreferences数据

- /data/system/dropbox 存放系统fc，应用fc，应用ANR，系统启动日志、日志备份等。如：	system_app_anr@1367921168510.txt表示某个时间点anr日志，
	system_app_crash@1368011664687.txt为某个时间点fc日志。

- 可以使用adb pull拷贝数据到本地，
adb pull /data/data/cn.trinea.android.demo/databases/androiddemo d:\\表示拷贝数据库到d盘
adb pull /data/system/dropbox/ d:\\systemNotes表示将若有日志拷贝到到d盘



----------
2015-12-31 10:46:47 

#### 1.Android为什么要设计出Bundle而不是直接使用HashMap来进行数据传递？ ####
Bundle内部是由ArrayMap实现的。

- ArrayMap的内部实现是两个数组，
	一个int数组是存储对象数据对应下标，一个对象数组保存key和value，
	内部使用二分法对key进行排序，
	所以在添加、删除、查找数据的时候，都会使用二分法查找，
	只适合于小数据量操作，如果在数据量比较大的情况下，那么它的性能将退化。

- 而HashMap内部则是数组+链表结构，
	所以在数据量较少的时候，HashMap的Entry Array比ArrayMap占用更多的内存。

因为使用Bundle的场景大多数为小数据量，我没见过在两个Activity之间传递10个以上数据的场景，所以相比之下，在这种情况下使用ArrayMap保存数据，在操作速度和内存占用上都具有优势，因此使用Bundle来传递数据，可以保证更快的速度和更少的内存占用。

另外一个原因，则是在Android中如果使用Intent来携带数据的话，需要数据是基本类型或者是可序列化类型，HashMap使用Serializable进行序列化，而Bundle则是使用Parcelable进行序列化。而在Android平台中，更推荐使用Parcelable实现序列化，虽然写法复杂，但是开销更小，所以为了更加快速的进行数据的序列化和反序列化，系统封装了Bundle类，方便我们进行数据的传输。



----------
2016-01-04 10:30:31 


#### 1.获得当前屏幕显示的Activity ####

	ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);  
	ComponentName cn = am.getRunningTasks(1).get(0).topActivity;  
	Log.d("", "pkg:"+cn.getPackageName());  
	Log.d("", "cls:"+cn.getClassName()); 

#### 2. Activity、Window、View的关系####

##### 一、View和ViewGroup #####

	Android系统中的所有UI类都是建立在View和ViewGroup这两个类的基础上的。

	所有View的子类成为”Widget”，所有ViewGroup的子类成为”Layout”。

	View和ViewGroup之间采用了组合设计模式，可以使得“部分-整体”同等对待。

	ViewGroup作为布局容器类的最上层，布局容器里面又可以有View和ViewGroup。
![](http://i.imgur.com/WEMQFff.png)

##### 二、LayoutInflater，LayoutInflater.inflate() #####

	LayoutInflater是一个用来实例化XML布局文件为View对象的类

    LayoutInflater.infalte(R.layout.test,null)用来从指定的XML资源中填充一个新的View


##### 三、Activity、Window、View之间的关系 #####

	当我们运行程序的时候，有一个setContentView()方法，Activity其实不是显示视图（直观上感觉是它），实际上Activity调用了PhoneWindow的setContentView()方法，

	然后加载视图，将视图放到这个Window上，而Activity其实构造的时候初始化的是Window（PhoneWindow），Activity其实是个控制单元，即可视的人机交互界面。

![](http://i.imgur.com/ml4eEMF.png)

	运行程序后，Activity会调用PhoneWindow的setContentView()来生成一个Window，

	而此时的setContentView就是那个最底层的View。

	然后通过LayoutInflater.infalte()方法加载布局生成View对象并通过addView()方法添加到Window上。

	（一层一层的叠加到 Window上，一个Activity构造的时候只能初始化一个Window(PhoneWindow) ）

### 结论：Activity其实不是显示视图，View才是真正的显示视图 ###


----------
2016-01-05 10:28:42 
#### 1.ViewStub的一些特点 ####

1. ViewStub只能Inflate一次，之后ViewStub对象会被置为空。按句话说，某个被ViewStub指定的布局被Inflate后，就不会够再通过ViewStub来控制它了。
2. ViewStub只能用来Inflate一个布局文件，而不是某个具体的View，当然也可以把View写在某个布局文件中。

		基于以上特点，可以使用ViewStub的场景如下：

	- 在程序的运行期间，某个布局在Inflate后，就不会有变化，除非重新启动。
	- 想要控制显示与隐藏的是一个布局文件，而非某个View。所以ViewStub没办法控制布局文件中指定的控件。


#### 2.View的绘制原理 ####

先来上张图，看看View的组成：

![](http://i.imgur.com/lUGmc5q.png)

官方介绍：

	View的绘制由三个过程组成，测量过程、布局过程和绘制过程。
		
		-测量过程：这个过程是由onMeasure（int，int）完成，
			onMeasure方法的作用：从上到下遍历视图树，也就是layout文件，在遍历过程中，每个视图都会向下层
       			  传递尺寸和规格，遍历结束后（onMeasure方法完成），每个视图都保存了自己的尺寸和规格。 
		- 布局过程：这个过程是由onLayout（int，int，int，int）完成，
			onLayout方法的作用：从上到下遍历视图树，在遍历过程中每个父视图通过测量过程的结果定位所有子视图的位置信息。
		- 绘制过程：这个过程是由onDraw()完成，
			onDraw方法的作用：绘制视图，简化顺序是：
				第一步. 绘制视图的背景 	
				第二步. 对视图的内容进行绘制，调用了onDraw()方法，具体过程由子类去实现  
				第三步. 对当前视图的所有子视图进行绘制（如果没有子类，就不需要去绘制了）
				第四步. 对视图的滚动条进行绘制（所有view默认都是有进度条的，只是一些并没有显示）

		 	重点在第二步，所有自定义控件都需要实现onDraw()方法，



#### 3.res/raw和assets的不同点 ####
1.res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。

2.res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹

3.assets文件夹时工程默认创建的，但是res/raw需要手动创建




----------
2016-01-06 16:52:50 

#### 1.View.isShown() ####

这个方法只有当这个view本身以及它的父view和祖先view（如果有）都是visible的时候，才返回true。

#### 2.TextView和EditText都有setError（） ####

用户输入时，错误信息利用EditText.setError("")方法省时省力！
清除错误信息时，同样是调用setError方法，此时参数为null即可。

#### 3.android:descendantFocusability讲解 ####

如果遇到listView的item中包含很多需要获取焦点的控件的时候，需要在item的跟布局下加入

		android:descendantFocusability=”blocksDescendants”

属性，这样子widget就不会抢夺ViewGroup的焦点了。另外，descendantFocusability的几个属性见下图：

![](http://i.imgur.com/4ym8LQw.png)

#### 4.android:scaleType (ImageView)属性讲解 ####

定义在 ImageView 中怎么缩放/剪裁图片，ImageView.ScaleType共八种：

![](http://i.imgur.com/1XJKGgd.png)

#### 5.android:imeOptions（EditText）属性讲解 ####
EditText中使用对键盘的action键（一般是右下角）定制，并且通过监听键盘执行相应的操作，但是imeOptions并 不仅仅局限于此，还有很对对键盘控制的属性，比方在横屏的时候，控制键盘不全屏显示等。

![](http://i.imgur.com/QBYxG5V.png)

----------
2016-01-07 16:52:50 

#### 1.在代码中设置颜色的方式 ####

	TextView tText=(TextView) findViewById(R.id.textv_name);  
	//第1种：  
	tText.setTextColor(android.graphics.Color.RED);//系统自带的颜色类  
	  
	// 第2种：  
	tText.setTextColor(0xffff00ff);//0xffff00ff是int类型的数据，分组一下0x|ff|ff00ff，0x是代表颜色整数的标记，ff是表示透明度，ff00ff表示颜色，注意：这里ffff00ff必须是8个的颜色表示，不接受ff00ff这种6个的颜色表示。  
	   
	//第3种：  
	tText.setTextColor(android.graphics.Color.parseColor("#87CEFA")) ; //还是利用Color类；  
	  
	  
	//第4种：  
	tText.setTextColor(this.getResources().getColor(R.color.red));  
	  
	  
	/*通过获得资源文件进行设置。根据不同的情况R.color.red也可以是R.string.red或者R.drawable.red， 
	 * 当然前提是需要在相应的配置文件里做相应的配置，如(xml 标签): 
	 *       
	 * <color name="red">#FF0000</color> 
        <drawable name="red">#FF0000</drawable> 
        <string name="red">#FF0000</string>*/  


#### 2.代码里设置view ####

1. 添加一个LinearLayout   layout；参数设置为：layout_width:wrap_content;
   layout_height:wrap_content;
		

	LinearLayout layout = new LinearLayout(this); // 线性布局方式  
	layout.setOrientation(LinearLayout.HORIZONTAL); //  
	layout.setBackgroundColor(0xff00ffff);  
	LinearLayout.LayoutParams LP_MM = new LinearLayout.LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);    
	layout.setLayoutParams(LP_MM);  

2.  添加一个ImageView；参数设置成layout_width:50;layout_height:50;

	ImageView imageView= new ImageView(this);  
	imageView.setBackgroundResource(R.drawable.maomao);  
	LinearLayout.LayoutParams PARA = new LinearLayout.LayoutParams(50,50);  
	imageView.setLayoutParams(PARA);  
	layout.addView(imageView);  

3.  添加一个TextView；参数设置成layout_width:wrap_content;layout_height:wrap_content;

	TextView tv = new TextView(this); // 普通聊天对话  
	tv.setText("我和猫猫是新添加的");  
	tv.setBackgroundColor(Color.GRAY);  
	LinearLayout.LayoutParams LP_WW = new LinearLayout.LayoutParams(  
    LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);  
	tv.setLayoutParams(LP_WW);  
	layout.addView(tv);  

4.  设置margin（只有layout可以）

	LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT);  
	lp.setMargins(10, 20, 30, 40);  
	imageView.setLayoutParams(lp);  

5.  动态设置padding(所有view都可以用)

	ImageView imageView = new ImageView(this);  
	imageView.setPadding(5,5,5,5);// （←，↑，→，↓） 


----------
2016-02-15 18:17:40 

#### 1.Android 设置DrawableRight和DrawableLeft 点击事件 ####

重写EditText的onTouchEvent，捕捉到点击事件，然后判断用户点击的位置是否点击到图标即可.

http://wiyi.org/android_drawablelistener.html


----------
2016-02-18 11:50:07 

#### 1.android layout布局属性 ####

第一类：属性值 true或者 false

           android:layout_centerHrizontal 水平居中
	　　    android:layout_centerVertical 垂直居中
	　　    android:layout_centerInparent 相对于父元素完全居中
	　　    android:layout_alignParentBottom 贴紧父元素的下边缘
	　　    android:layout_alignParentLeft 贴紧父元素的左边缘
	　　    android:layout_alignParentRight 贴紧父元素的右边缘
	　　    android:layout_alignParentTop 贴紧父元素的上边缘
	　　    android:layout_alignWithParentIfMissing 如果对应的兄弟元素找不到的话就以父元素做参照物

           android:layout_alignParentStart紧贴父元素结束位置开始

           android:layout_alignParentEnd紧贴父元素结束位置结束

           android:animateLayoutChanges布局改变时是否有动画效果

           android:clipChildren定义子布局是否一定要在限定的区域内

           android:clipToPadding定义布局间是否有间距

           android:animationCache定义子布局也有动画效果

           android:alwaysDrawnWithCache定义子布局是否应用绘图的高速缓存

           android:addStatesFromChildren定义布局是否应用子布局的背景

           android:splitMotionEvents定义布局是否传递touch事件到子布局

           android:focusableInTouchMode定义是否可以通过touch获取到焦点

           android:isScrollContainer定义布局是否作为一个滚动容器 可以调整整个窗体

           android:fadeScrollbars滚动条自动隐藏

           android:fitsSystemWindows设置布局调整时是否考虑系统窗口(如状态栏)

           android:visibility定义布局是否可见

           android:requiresFadingEdge定义滚动时边缘是否褪色

           android:clickable定义是否可点击

           android:longClickable定义是否可长点击

           android:saveEnabled设置是否在窗口冻结时（如旋转屏幕）保存View的数据

           android:filterTouchesWhenObscured所在窗口被其它可见窗口遮住时,是否过滤触摸事件

           android:keepScreenOn设置屏幕常亮

           android:duplicateParentState是否从父容器中获取绘图状态(光标,按下等)

           android:soundEffectsEnabled点击或触摸是否有声音效果

           android:hapticFeedbackEnabled设置触感反馈

第二类：属性值必须为id的引用名“@id/id-name”

         android:layout_alignBaseline 本元素的文本与父元素文本对齐

	　　  android:layout_below 在某元素的下方
	　　  android:layout_above 在某元素的的上方
	　　  android:layout_toLeftOf 在某元素的左边
	　　  android:layout_toRightOf 在某元素的右边

         android:layout_toStartOf本元素从某个元素开始

         android:layout_toEndOf本元素在某个元素结束

	　　  android:layout_alignTop 本元素的上边缘和某元素的的上边缘对齐
	　　  android:layout_alignLeft 本元素的左边缘和某元素的的左边缘对齐
	　　  android:layout_alignBottom 本元素的下边缘和某元素的的下边缘对齐
	　　  android:layout_alignRight 本元素的右边缘和某元素的的右边缘对齐

         android:layout_alignStart本元素与开始的父元素对齐

         android:layout_alignEnd本元素与结束的父元素对齐

         android:ignoreGravity 指定元素不受重力的影响

         android:layoutAnimation定义布局显示时候的动画

         android:id 为布局添加ID方便查找

         android:tag为布局添加tag方便查找与类似

         android:scrollbarThumbHorizontal设置水平滚动条的drawable。

         android:scrollbarThumbVertical设置垂直滚动条的drawable

         android:scrollbarTrackHorizontal设置水平滚动条背景（轨迹）的色drawable

         android:scrollbarTrackVertical设置垂直滚动条背景（轨迹）的色drawable

         android:scrollbarAlwaysDrawHorizontalTrack 设置水平滚动条是否含有轨道

         android:scrollbarAlwaysDrawVerticalTrack 设置垂直滚动条是否含有轨道

         android:nextFocusLeft 设置左边指定视图获得下一个焦点

         android:nextFocusRight设置右边指定视图获得下一个焦点

         android:nextFocusUp设置上边指定视图获得下一个焦点

         android:nextFocusDown设置下边指定视图获得下一个焦点

         android:nextFocusForward设置指定视图获得下一个焦点

         android:contentDescription 说明

         android:OnClick 点击时从上下文中调用指定的方法
         
第三类：属性值为具体的像素值，如30dip，40px,50dp

        android:layout_width定义本元素的宽度

        android:layout_height定义本元素的高度

        android:layout_margin 本元素离上下左右间的距离

	　　 android:layout_marginBottom 离某元素底边缘的距离
	　　 android:layout_marginLeft 离某元素左边缘的距离
	　　 android:layout_marginRight 离某元素右边缘的距离
	　　 android:layout_marginTop 离某元素上边缘的距离

        android:layout_marginStart本元素里开始的位置的距离

        android:layout_marginEnd本元素里结束位置的距离

        android:scrollX水平初始滚动偏移

        android:scrollY垂直初始滚动偏移

        android:background本元素的背景

        android:padding指定布局与子布局的间距

        android:paddingLeft指定布局左边与子布局的间距

        android:paddingTop指定布局上边与子布局的间距

        android:paddingRight指定布局右边与子布局的间距

        android:paddingBottom指定布局下边与子布局的间距

        android:paddingStart指定布局左边与子布局的间距与android:paddingLeft相同

        android:paddingEnd指定布局右边与子布局的间距与android:paddingRight相同

        android:fadingEdgeLength 设置边框渐变的长度

        android:minHeight最小高度

        android:minWidth最小宽度

        android:translationX 水平方向的移动距离

        android:translationY垂直方向的移动距离

        android:transformPivotX相对于一点的水平方向偏转量

        android:transformPivotY相对于一点的垂直方向偏转量

第四类：属性值问Android内置值的

        android:gravity控件布局方式

        android:layout_gravity布局方式

        android:persistentDrawingCachehua定义绘图的高速缓存的持久性   

        android:descendantFocusability控制子布局焦点获取方式 常用于listView的item中包含多个控件 点击无效

        android:scrollbars设置滚动条的状态

        android:scrollbarStyle设置滚动条的样式

        android:fitsSystemWindows设置布局调整时是否考虑系统窗口(如状态栏)

        android:scrollbarFadeDuration设置滚动条淡入淡出时间

        android:scrollbarDefaultDelayBeforeFade设置滚动条N毫秒后开始淡化，以毫秒为单位。

        android:scrollbarSize设置滚动调大小

        android:fadingEdge 设置拉滚动条时 ,边框渐变的放向

        android:drawingCacheQuality设置绘图时半透明质量

        android:OverScrollMode滑动到边界时样式

        android:alpha设置透明度

        android:rotation旋转度数

        android:rotationX水平旋转度数

        android:rotationY垂直旋转度数

        android:scaleX设置X轴缩放

        android:scaleY设置Y轴缩放

        android:verticalScrollbarPosition摄者垂直滚动条的位置

        android:layerType设定支持

        android:layoutDirection定义布局图纸的方向

        android:textDirection定义文字方向

        android:textAlignment文字对齐方式

        android:importantForAccessibility设置可达性的重要行

        android:labelFor添加标签

============================================================================================
	为了更精确地控制应用程序在UI上的文字书写顺序（从左到右，或者从右到左），Android 4.2 引入了如下的API：
	
	android:layoutDirection —该属性设置组件的布局排列方向
	
	android:textDirection — 该属性设置组件的文字排列方向
	
	android:textAlignment — 该属性设置文字的对齐方式
	
	getLayoutDirectionFromLocale() —该方法用于获取指定地区的惯用布局方式
	
	在使用从右到左的排列方式时，你甚至创建自定义的布局方式，可绘制对象，以及其他资源。仅仅是简单地使用资源匹配器“ldrtl”对你的资源进行一下标识，你就可以把资源定义为“从右到左方向的资源”。在调试和优化从右到左的布局方面，HierarchyViewer目前支持对start/end属性，布局方向，文字方向，文字对齐方式等所有信息的层次化显示。



----------
2016-02-23 14:54:44 


#### 1.ImageView和button不同，它是默认不可点击的 ####

#### 2. 各种常用view与layout的继承关系####

![](http://i.imgur.com/55BfPuA.png)




----------
2016-02-29 16:55:55 

#### 1.MVP架构讲解 ####

MVP（Model View Presenter）
相比MVC，可以让Model和View完全解耦

	- View 对应于Activity\Fragment，负责View的绘制以及与用户交互
	- Model 依然是业务逻辑和实体模型,用于数据的增删改查等，也包括一些数据对象 
	- Presenter 是View跟Model的“中间人”，接收View的请求后，从Model获取数据交给View,完成View于Model间的交互

MVC到MVP的变化过程：

![](http://i.imgur.com/JRIWCed.png)

转变为：

![](http://i.imgur.com/4TWM6OR.png)

----------
2016-03-02 09:29:11 

#### 1.正则表达 ####

使用正则进行验证的方式：

	private Pattern pattern;
    private Matcher matcher;	

	private static final String *_PATTERN = "正则表达规则";
    public boolean validate(final String username){//需要匹配的内容
		pattern = Pattern.compile(*_PATTERN);
        matcher = pattern.matcher(username);
        return matcher.matches();
    }

常见的正则表达式：

- 校验数字的表达式
	
	1 数字：^[0-9]*$
	
	2 n位的数字：^\d{n}$
	
	3 至少n位的数字：^\d{n,}$
	
	4 m-n位的数字：^\d{m,n}$
	
	5 零和非零开头的数字：^(0|[1-9][0-9]*)$
	
	6 非零开头的最多带两位小数的数字：^([1-9][0-9]*)+(.[0-9]{1,2})?$
	
	7 带1-2位小数的正数或负数：^(-)?\d+(.\d{1,2})?$
	
	8 正数、负数、和小数：^(-|+)?\d+(.\d+)?$
	
	9 有两位小数的正实数：^[0-9]+(.[0-9]{2})?$
	
	10 有1~3位小数的正实数：^[0-9]+(.[0-9]{1,3})?$
	
	11 非零的正整数：^[1-9]\d$ 或 ^([1-9][0-9]){1,3}$ 或 ^+?[1-9][0-9]*$
	
	12 非零的负整数：^-[1-9][]0-9"$ 或 ^-[1-9]\d$
	
	13 非负整数：^\d+$ 或 ^[1-9]\d*|0$
	
	14 非正整数：^-[1-9]\d*|0$ 或 ^((-\d+)|(0+))$
	
	15 非负浮点数：^\d+(.\d+)?$ 或 ^[1-9]\d.\d|0.\d[1-9]\d|0?.0+|0$
	
	16 非正浮点数：^((-\d+(.\d+)?)|(0+(.0+)?))$ 或 ^(-([1-9]\d.\d|0.\d[1-9]\d))|0?.0+|0$
	
	17 正浮点数：^[1-9]\d.\d|0.\d[1-9]\d$ 或 ^(([0-9]+.[0-9][1-9][0-9])|([0-9][1-9][0-9].[0-9]+)|([0-9][1-9][0-9]))$
	
	18 负浮点数：^-([1-9]\d.\d|0.\d[1-9]\d)$ 或 ^(-(([0-9]+.[0-9][1-9][0-9])|([0-9][1-9][0-9].[0-9]+)|([0-9][1-9][0-9])))$
	
	19 浮点数：^(-?\d+)(.\d+)?$ 或 ^-?([1-9]\d.\d|0.\d[1-9]\d|0?.0+|0)$

- 校验字符的表达式

	1 汉字：^[\u4e00-\u9fa5]{0,}$

	2 英文和数字：^[A-Za-z0-9]+$ 或 ^[A-Za-z0-9]{4,40}$
	
	3 长度为3-20的所有字符：^.{3,20}$
	
	4 由26个英文字母组成的字符串：^[A-Za-z]+$
	
	5 由26个大写英文字母组成的字符串：^[A-Z]+$
	
	6 由26个小写英文字母组成的字符串：^[a-z]+$
	
	7 由数字和26个英文字母组成的字符串：^[A-Za-z0-9]+$
	
	8 由数字、26个英文字母或者下划线组成的字符串：^\w+$ 或 ^\w{3,20}$
	
	9 中文、英文、数字包括下划线：^[\u4E00-\u9FA5A-Za-z0-9_]+$
	
	10 中文、英文、数字但不包括下划线等符号：^[\u4E00-\u9FA5A-Za-z0-9]+$ 或 ^[\u4E00-\u9FA5A-Za-z0-9]{2,20}$
	
	11 可以输入含有^%&',;=?$\"等字符：[^%&',;=?$\x22]+
	
	12 禁止输入含有~的字符：[^~\x22]+

- 特殊需求表达式

	1 Email地址：^\w+([-+.]\w+)@\w+([-.]\w+).\w+([-.]\w+)*$

	2 域名：[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(/.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+/.?
	
	3 InternetURL：[a-zA-z]+://[^\s] 或 ^http://([\w-]+\.)+[\w-]+(/[\w-./?%&=])?$
	
	4 手机号码：^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$
	
	5 电话号码("XXX-XXXXXXX"、"XXXX-XXXXXXXX"、"XXX-XXXXXXX"、"XXX-XXXXXXXX"、"XXXXXXX"和"XXXXXXXX)：^((\d{3,4}-)|\d{3.4}-)?\d{7,8}$
	
	6 国内电话号码(0511-4405222、021-87888822)：\d{3}-\d{8}|\d{4}-\d{7}
	
	7 身份证号(15位、18位数字)：^\d{15}|\d{18}$
	
	8 短身份证号码(数字、字母x结尾)：^([0-9]){7,18}(x|X)?$ 或 ^\d{8,18}|[0-9x]{8,18}|[0-9X]{8,18}?$
	
	9 帐号是否合法(字母开头，允许5-16字节，允许字母数字下划线)：^[a-zA-Z][a-zA-Z0-9_]{4,15}$
	
	10 密码(以字母开头，长度在6~18之间，只能包含字母、数字和下划线)：^[a-zA-Z]\w{5,17}$
	
	11 强密码(必须包含大小写字母和数字的组合，不能使用特殊字符，长度在8-10之间)：^(?=.\d)(?=.[a-z])(?=.*[A-Z]).{8,10}$
	
	12 日期格式：^\d{4}-\d{1,2}-\d{1,2}
	
	13 一年的12个月(01～09和1～12)：^(0?[1-9]|1[0-2])$
	
	14 一个月的31天(01～09和1～31)：^((0?[1-9])|((1|2)[0-9])|30|31)$
	
	15 钱的输入格式：
	
	16 1.有四种钱的表示形式我们可以接受:"10000.00" 和 "10,000.00", 和没有 "分" 的 "10000" 和 "10,000"：^[1-9][0-9]*$
	
	17 2.这表示任意一个不以0开头的数字,但是,这也意味着一个字符"0"不通过,所以我们采用下面的形式：^(0|[1-9][0-9]*)$
	
	18 3.一个0或者一个不以0开头的数字.我们还可以允许开头有一个负号：^(0|-?[1-9][0-9]*)$
	
	19 4.这表示一个0或者一个可能为负的开头不为0的数字.让用户以0开头好了.把负号的也去掉,因为钱总不能是负的吧.下面我们要加的是说明可能的小数部分：^[0-9]+(.[0-9]+)?$
	
	20 5.必须说明的是,小数点后面至少应该有1位数,所以"10."是不通过的,但是 "10" 和 "10.2" 是通过的：^[0-9]+(.[0-9]{2})?$
	
	21 6.这样我们规定小数点后面必须有两位,如果你认为太苛刻了,可以这样：^[0-9]+(.[0-9]{1,2})?$
	
	22 7.这样就允许用户只写一位小数.下面我们该考虑数字中的逗号了,我们可以这样：^[0-9]{1,3}(,[0-9]{3})*(.[0-9]{1,2})?$
	
	23 8.1到3个数字,后面跟着任意个 逗号+3个数字,逗号成为可选,而不是必须：^([0-9]+|[0-9]{1,3}(,[0-9]{3})*)(.[0-9]{1,2})?$
	
	24 备注：这就是最终结果了,别忘了"+"可以用"*"替代如果你觉得空字符串也可以接受的话(奇怪,为什么?)最后,别忘了在用函数时去掉去掉那个反斜杠,一般的错误都在这里
	
	25 xml文件：^([a-zA-Z]+-?)+[a-zA-Z0-9]+\.[x|X][m|M][l|L]$
	
	26 中文字符的正则表达式：[\u4e00-\u9fa5]
	
	27 双字节字符：[^\x00-\xff] (包括汉字在内，可以用来计算字符串的长度(一个双字节字符长度计2，ASCII字符计1))
	
	28 空白行的正则表达式：\n\s*\r (可以用来删除空白行)
	
	29 HTML标记的正则表达式：<(\S?)[^>]>.?</\1>|<.? /> (网上流传的版本太糟糕，上面这个也仅仅能部分，对于复杂的嵌套标记依旧无能为力)
	
	30 首尾空白字符的正则表达式：^\s|\s$或(^\s)|(\s$) (可以用来删除行首行尾的空白字符(包括空格、制表符、换页符等等)，非常有用的表达式)
	
	31 腾讯QQ号：[1-9][0-9]{4,} (腾讯QQ号从10000开始)
	
	32 中国邮政编码：[1-9]\d{5}(?!\d) (中国邮政编码为6位数字)
	
	33 IP地址：\d+.\d+.\d+.\d+ (提取IP地址时有用)
	
	34 IP地址：((?:(?:25[0-5]|2[0-4]\d|[01]?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d?\d))


----------
2016-03-04 16:35:44 

#### 1.EditText 不弹出输入法总结 ####

- 方法一：

	在AndroidMainfest.xml中选择相应的activity，设置windowSoftInputMode属性为adjustUnspecified|stateHidden

- 方法二：

	让EditText失去焦点，使用EditText的clearFocus方法（获得焦点的方法是editText.requestFocus() ）

- 方法三：

	强制隐藏Android输入法窗口
	EditText edit=(EditText)findViewById(R.id.edit); 
	InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE); 
	imm.hideSoftInputFromWindow(edit.getWindowToken(),0);  

----------
2016-03-11 10:23:27 

### Android动画专题  ###

#### 简单的动画 ####




1.代码实现alpha  

	Animation anim = new AlphaAnimation(0f, 1.0f);  
    anim.setDuration(2000);  
    anim.setRepeatCount(Animation.INFINITE);  
    anim.setRepeatMode(Animation.RESTART);  
XML 实现 alpha ：

	<?xml version="1.0" encoding="utf-8"?>  
	<set xmlns:android="http://schemas.android.com/apk/res/android" >  
	      
	    <alpha  
	        android:duration="2000"  
	        android:fromAlpha="0"  
	        android:repeatCount="3"//动画重复的次数  
	        android:repeatMode="reverse"//动画重复的模式  reverse：倒序，即从最后一帧播放动画；  restart：顺序，即从头开始播放动画  
	        android:toAlpha="1.0" />  
	  
	</set> 


2.代码实现scale

	Animation anim1 = new ScaleAnimation(0.5f, 1.0f, 0.5f, 1.0f, 0.5f, 0.5f);  
    anim1.setDuration(2000);  
    anim1.setRepeatCount(3);  
    anim1.setRepeatMode(Animation.REVERSE);  
    anim1.setInterpolator(this, interpolator.accelerate_decelerate);  
    anim1.setFillAfter(true); 

XML实现scale
	
	<?xml version="1.0" encoding="utf-8"?>  
	<set xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:fillAfter="true"  
	    android:interpolator="@android:anim/decelerate_interpolator" >  
	 <!-- 画面停留在最后一帧 -->  
	    >  
	  
	    <scale  
	        android:duration="2000"  
	        android:fromXScale="0.5"  
	        android:fromYScale="0.5"  
	        android:pivotX="50.0%" >  
	 <!-- 动画相对于空间X坐标开始的位置50。0%表示从空间的正中央开始缩放动画 -->  
	        android:pivotY="50.0%"  
	        android:repeatCount="3"  
	        android:repeatMode="reverse"  
	        android:toXScale="1.0"  
	        android:toYScale="1.0"  
	         />  
	  
	    </scale>  
	</set> 

3.代码实现 translate

	Animation animTranlate = new TranslateAnimation(0, 0, 0, 400);  
    animTranlate.setDuration(2000);  
    animTranlate.setFillAfter(true);  
    animTranlate.setInterpolator(this, interpolator.bounce);  
    text.startAnimation(animTranlate);  

XML实现 translate  

	<?xml version="1.0" encoding="utf-8"?>  
	<set xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:interpolator="@android:anim/decelerate_interpolator"  
	    android:fillAfter="true" >  
	  
	    <translate  
	        android:duration="2000"  
	        android:fromXDelta="0"  
	        android:fromYDelta="0"  
	        android:toXDelta="0"  
	        android:toYDelta="1080" />  
	  
	</set> 

4.代码实现rotate

		Animation animRotate = new RotateAnimation(0f, 360f,  
                Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,  
                0.5f);  
        animRotate.setDuration(2000);  
        animRotate.setInterpolator(this, interpolator.bounce);  
        animRotate.setFillAfter(true);  
		animRotate.setStartOffset(2000);//动画执行前停留时间  
        text.startAnimation(animRotate);  

XML实现rotate 

	<?xml version="1.0" encoding="utf-8"?>  
	<set xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:fillAfter="true"  
	    android:interpolator="@android:anim/accelerate_decelerate_interpolator" >  
	  
	    <rotate  
	        android:duration="2000"  
	        android:fromDegrees="0"  
	        android:pivotX="50%"  
	        android:pivotY="50%"  
        android:toDegrees="180" //负数就是逆时针，正数顺时针。  
	/>  
	  
	</set> 

组合动画的实现

	<?xml version="1.0" encoding="utf-8"?>  
	<set xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:duration="2000"  
	    android:fillAfter="true"  
	    android:interpolator="@android:anim/accelerate_decelerate_interpolator" >  
	  
	    <alpha  
	        android:fromAlpha="0"  
	        android:toAlpha="1.0" />  
	  
	    <scale  
	        android:fromXScale="0"  
	        android:fromYScale="0"  
	        android:pivotX="50%"  
	        android:pivotY="50%"  
	        android:toXScale="1.0"  
	        android:toYScale="1.0" />  
	  
	</set>

代码实现组合动画

	Animation anim1 = new ScaleAnimation(0.5f, 1.0f, 0.5f, 1.0f, 0.5f, 0.5f);  
    anim1.setDuration(2000);  
    anim1.setRepeatCount(3);  
    anim1.setRepeatMode(Animation.REVERSE);  
    anim1.setInterpolator(this, interpolator.accelerate_decelerate);  
    anim1.setFillAfter(true);  
    // text.setAnimation(anim1);  

    Animation animTranlate = new TranslateAnimation(0, 0, 0, 400);  
    animTranlate.setDuration(2000);  
    animTranlate.setFillAfter(true);  
    animTranlate.setInterpolator(this, interpolator.bounce);  
    // text.startAnimation(animTranlate);  
    Animation animRotate = new RotateAnimation(0f, 360f,  
            Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,  
            0.5f);  

    animRotate.setDuration(2000);  
    animRotate.setInterpolator(this, interpolator.bounce);  
    animRotate.setFillAfter(true);  
    // text.startAnimation(animRotate);  

    AnimationSet animationSet = new AnimationSet(true);  
    animationSet.addAnimation(animRotate);  
    animationSet.addAnimation(animTranlate);  
    animationSet.addAnimation(anim1);  
    text.startAnimation(animationSet); 


----------
2016-03-24 09:34:41 

#### TextView中文字通过SpannableString设置属性 ####

	//创建一个 SpannableString对象      
	SpannableString msp = new SpannableString("字体测试字体大小一半两倍前景色背景色正常粗体斜体粗斜体下划线删除线x1x2电话邮件网站短信彩信地图X轴综合");     
          
	  //设置字体(default,default-bold,monospace,serif,sans-serif)    
	  msp.setSpan(new TypefaceSpan("monospace"), 0, 2, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
	  msp.setSpan(new TypefaceSpan("serif"), 2, 4, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
	      
	  //设置字体大小（绝对值,单位：像素）     
	  msp.setSpan(new AbsoluteSizeSpan(20), 4, 6, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
	  msp.setSpan(new AbsoluteSizeSpan(20,true), 6, 8, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //第二个参数boolean dip，如果为true，表示前面的字体大小单位为dip，否则为像素，同上。    
	      
	  //设置字体大小（相对值,单位：像素） 参数表示为默认字体大小的多少倍    
	  msp.setSpan(new RelativeSizeSpan(0.5f), 8, 10, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //0.5f表示默认字体大小的一半    
	  msp.setSpan(new RelativeSizeSpan(2.0f), 10, 12, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //2.0f表示默认字体大小的两倍    
	      
	  //设置字体前景色    
	  msp.setSpan(new ForegroundColorSpan(Color.MAGENTA), 12, 15, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //设置前景色为洋红色    
	      
	  //设置字体背景色    
	  msp.setSpan(new BackgroundColorSpan(Color.CYAN), 15, 18, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //设置背景色为青色    
	   
	  //设置字体样式正常，粗体，斜体，粗斜体    
	  msp.setSpan(new StyleSpan(android.graphics.Typeface.NORMAL), 18, 20, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //正常    
	  msp.setSpan(new StyleSpan(android.graphics.Typeface.BOLD), 20, 22, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //粗体    
	  msp.setSpan(new StyleSpan(android.graphics.Typeface.ITALIC), 22, 24, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //斜体    
	  msp.setSpan(new StyleSpan(android.graphics.Typeface.BOLD_ITALIC), 24, 27, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  //粗斜体    
	      
	  //设置下划线    
	  msp.setSpan(new UnderlineSpan(), 27, 30, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
	      
	  //设置删除线    
	  msp.setSpan(new StrikethroughSpan(), 30, 33, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
	      
	  //设置上下标    
	  msp.setSpan(new SubscriptSpan(), 34, 35, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //下标       
	  msp.setSpan(new SuperscriptSpan(), 36, 37, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);   //上标                
	      
	  //超级链接（需要添加setMovementMethod方法附加响应）    
	  msp.setSpan(new URLSpan("tel:4155551212"), 37, 39, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //电话       
	  msp.setSpan(new URLSpan("mailto:webmaster@google.com"), 39, 41, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //邮件       
	  msp.setSpan(new URLSpan("http://www.baidu.com"), 41, 43, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //网络       
	  msp.setSpan(new URLSpan("sms:4155551212"), 43, 45, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //短信   使用sms:或者smsto:    
	  msp.setSpan(new URLSpan("mms:4155551212"), 45, 47, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //彩信   使用mms:或者mmsto:    
	  msp.setSpan(new URLSpan("geo:38.899533,-77.036476"), 47, 49, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);     //地图       
	      
	  //设置字体大小（相对值,单位：像素） 参数表示为默认字体宽度的多少倍    
	  msp.setSpan(new ScaleXSpan(2.0f), 49, 51, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE); //2.0f表示默认字体宽度的两倍，即X轴方向放大为默认字体的两倍，而高度不变    
	        
	  //设置项目符号    
	  msp.setSpan(new BulletSpan(android.text.style.BulletSpan.STANDARD_GAP_WIDTH,Color.GREEN), 0 ,53, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE); //第一个参数表示项目符号占用的宽度，第二个参数为项目符号的颜色    
	  tv_textView.setText(msp);    
	  tv_textView.setMovementMethod(LinkMovementMethod.getInstance());   


----------
2016-04-21 10:51:21 
### 如何正确使用Context ###
首现上一张图：

![](http://i.imgur.com/UU1PwiA.jpg)

数字解释：

	 数字1：启动Activity在这些类中是可以的，但是需要创建一个新的task。一般情况不推荐。

     数字2：在这些类中去layout inflate是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。

     数字3：在receiver为null时允许，在4.2或以上的版本中，用于获取黏性广播的当前值。（可以无视）

  注：ContentProvider、BroadcastReceiver之所以在上述表格中，是因为在其内部方法中都有一个context用于使用。

什么情况下用使用什么context：
- 使用的周期是否在activity周期内，如果超出，必须用application；常见的情景包括：AsyncTask，Thread，第三方库初始化等等。有些情景，只能用activity：比如，对话框，各种View，需要startActivity的等。
- 凡是跟UI相关的，都应该使用 Activity做为Context来处理；其他的一些操作，Service,Activity,Application等实例都可以。但是，对持有Activity的对象要做到心中有数，保证它能随着生命周期的销毁而被回收，慎用static关键字。

另外，在一个app中，context满足以下公式：
  总Context实例个数 = Service个数 + Activity个数 + 1（Application对应的Context实例）


----------
2016-04-29 17:21:04 
#### Handler相关 ####

- new Handler()的时候，默认情况下Handler会绑定当前代码执行的线程，我们在主线程中实例化了uiHandler，所以uiHandler就自动绑定了主线程，即UI线程。一个Looper也就对应了一个MessageQueue。


#### 在子线程中更新UI的办法汇总 ####

1. handler.sendMessage（）。

2. handler的post操作（在子线程中使用handler的post可以这样写↓）

	public class MainActivity extends Activity {  
  
    	private Handler handler;  
  
	    @Override  
	    protected void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        setContentView(R.layout.activity_main);  
	        handler = new Handler();  
	        new Thread(new Runnable() {  
	            @Override  
	            public void run() {  
	                handler.post(new Runnable() {  
	                    @Override  
	                    public void run() {  
	                        // 在这里进行UI操作  
	                    }  
	                });  
	            }  
	        }).start();  
	    }  
	}  

3. Activity.runOnUiThread()【原理：如果当前线程不是ui线程，就调用handler.post()】

4. view 的post()。【原理：调用handler.post()】

----------
2016-05-04 14:29:01 
#### volley过程简图  ####

![](http://i.imgur.com/0VGzG1S.jpg)



----------
2016-05-05 15:30:04 
#### getWidth()方法和getMeasureWidth()方法的区别 ####
1.getMeasureWidth()方法在measure()过程结束后就可以获取到了，
而getWidth()方法要在layout()过程结束后才能获取到。
2.getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，
而getWidth()方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

----------
2016-05-11 14:38:03 
#### 1.getContext()、getApplication()、getApplicationContext()、getActivity()的区别 ####
(1).getContext():获取到当前对象的上下文。 
(2).getApplication():获得Application的对象 
(3).getApplicationContext():获得应用程序的上下文。有且仅有一个相同的对象。生命周期随着应用程序的摧毁而销毁。就像是社会，所有的都发生在这个社会上，仅且只有一个社会。每个Activity都有自己的上下文，而整个应用只有一个上下文 
(4)getActivity():获得Fragment依附的Activity对象。Fragment里边的getActivity()不推荐使用原因如下：这个方法会返回当前Fragment所附加的Activity，当Fragment生命周期结束并销毁时（host==null），getActivity()返回的是null，所以在使用时要注意判断null或者捕获空指针异常。所以只要判断getActivity()为空，就可以不再执行下面的代码，这完全不影响业务的使用。

----------
2016-05-30 11:02:04 

#### 提高绘制中的性能技巧 ####
绘制原理简述：Android系统每隔16ms发出VSYNC信号，触发对UI进行渲染，那么整个过程如果保证在16ms以内就能达到一个流畅的画面。如果系统发出VSYNC信号，而此时无法进行渲染，还在做别的操作，那么就会导致丢帧的现象。（16ms就是60fps，低于这个数值，人眼就可以感知变化过程）。

检测方式：

- 通过Hierarchy Viewer去检测渲染效率，去除不必要的嵌套
- 通过Show GPU Overdraw去检测Overdraw，最终可以通过移除不必要的背景以及使用canvas.clipRect解决大多数问题。

技巧简述：
### 检测Overdraw ###
- Overdraw 的处理方案一：移除不必要的background
	- 重复的bg，尤其是颜色，优化的时候要去掉。
	- 每个Activity的DecorView都会有默认的Background，如果我们有自己的Background，那么去掉这些默认的Background。方法如下：
	
		getWindow().setBackgroundDrawable(null);

- Overdraw 的处理方案二：clipRect的妙用

	自定义View时，如果出现View的叠加现象，那么去除最底层不被用户看到的View部分也会优化View的绘制，避免过度重复绘制。

	过度绘制的案例：
	![](http://i.imgur.com/6oNhm4S.png)
	左边显示的时效果图，右边显示的是开启Show Override GPU之后的效果，可以看到，卡片叠加处明显的过度渲染了。除了最后一张需要完整的绘制，其他的都只需要绘制显示的部分就可以了。具体操作可参考[http://blog.csdn.net/lmj623565791/article/details/45556391](http://blog.csdn.net/lmj623565791/article/details/45556391 "这里")。

### 通过Hierarchy Viewer去检测渲染效率 ###
工具效果图：
![](http://i.imgur.com/BUiMk3g.png)

图中红、黄、绿三个颜色分别代表measure 、layout、draw的绘制速度，红色相对其他两个状态来说速度是最慢的，绿色最佳。根据工具给出的线框图可以一定程度上优化layout的布局层次，从而减少不必要的布局，起到优化界面渲染速度的效果。

----------
2016-06-01 09:54:17 

### 使用SharedPreference需要注意的地方 ###
#### commit()和apply()的区别 ####
	返回值区别：apply()没有返回值，而commit()返回boolean表明修改是否提交成功。

	操作效率区别：apply()是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘,而commit()是同步的提交到硬件磁盘。

	使用场景：如果对提交的结果不关心的话，建议使用apply()，如果需要确保提交成功且有后续操作的话，还是需要用commit()。

### 隐式Intent的使用场景 ###

1. 启动闹钟：

	public void createAlarm(String message, int hour, int minutes) {
	    Intent intent = new Intent(AlarmClock.ACTION_SET_ALARM)
	            .putExtra(AlarmClock.EXTRA_MESSAGE, message)
	            .putExtra(AlarmClock.EXTRA_HOUR, hour)
	            .putExtra(AlarmClock.EXTRA_MINUTES, minutes);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

	参数说明：

		- EXTRA_HOUR：设定小时
		 
		- EXTRA_MINUTES：设定分钟
		 
		- EXTRA_MESSAGE：设定自定义消息
		 
		- EXTRA_DAYS：设定每周的周几重复响铃。该字段的值应定义成ArrayList，每个元素是Calendar类中的静态字段（如Calendar.Monday）。对于那些一次性响铃，无需设置这个extra。
		 
		- EXTRA_RINGTONE：设定闹钟铃声。该字段的值应设定成一个scheme为 content: 的URI类型，该URI指向了一个音频文件的地址；您也可以将值设定为 VALUE_RINGTONE_SILENT，这表示将闹钟设为静音。
		 
		- EXTRA_VIBRATE：设定是否响铃的同时震动。该字段的值为boolean类型。
		 
		- EXTRA_SKIP_UI：设定启动系统闹钟应用程序时，是否跳过UI界面，值为boolean类型，若为true，表示不显示设定闹钟的dialog对话框，而直接进入设置闹钟的activity。

	闹钟权限：<uses-permission android:name="com.android.alarm.permission.SET_ALARM" />

	intent-filter配置：
	<activity ...>
	    <intent-filter>
	        <action android:name="android.intent.action.SET_ALARM" />
	        <category android:name="android.intent.category.DEFAULT" />
	    </intent-filter>
	</activity>


2. 启动计时器（适用于Android4.4以上）
	
	public void startTimer(String message, int seconds) {
	    Intent intent = new Intent(AlarmClock.ACTION_SET_TIMER)
	            .putExtra(AlarmClock.EXTRA_MESSAGE, message)
	            .putExtra(AlarmClock.EXTRA_LENGTH, seconds)
	            .putExtra(AlarmClock.EXTRA_SKIP_UI, true);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

	参数说明：

		- EXTRA_LENGTH：设置倒计时的秒数
		- 
		- EXTRA_MESSAGE：设置到时消息
		- 
		- EXTRA_SKIP_UI：设定启动系统计时器应用程序时，是否跳过UI界面，值为boolean类型，若为true，表示不显示设定计时器的设置dialog对话框，而直接进入计时器的activity。

	权限设置：<uses-permission android:name="com.android.alarm.permission.SET_ALARM" />
	
	intent-filter配置：
	<activity ...>
	    <intent-filter>
	        <action android:name="android.intent.action.SET_TIMER" />
	        <category android:name="android.intent.category.DEFAULT" />
	    </intent-filter>
	</activity>

3. 相机

	static final int REQUEST_IMAGE_CAPTURE = 1;
	static final Uri mLocationForPhotos;
	
	public void capturePhoto(String targetFilename) {
	    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
	    intent.putExtra(MediaStore.EXTRA_OUTPUT,
	            Uri.withAppendedPath(mLocationForPhotos, targetFilename));
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivityForResult(intent, REQUEST_IMAGE_CAPTURE);
	    }
	}
	
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
	        Bitmap thumbnail = data.getParcelable("data");
	        // Do other work with full size photo saved in mLocationForPhotos
	        ...
	    }
	}

	intent-filter配置：
	<activity ...>
	    <intent-filter>
	        <action android:name="android.media.action.IMAGE_CAPTURE" />
	        <category android:name="android.intent.category.DEFAULT" />
	    </intent-filter>
	</activity>

4. 通讯录类别（如果只是通过Contacts Provider读取联系人信息，则无需申请READ_CONTACTS权限。）

#### 选择联系人 ####
	static final int REQUEST_SELECT_CONTACT = 1;

	public void selectContact() {
	    Intent intent = new Intent(Intent.ACTION_PICK);
	    intent.setType(ContactsContract.Contacts.CONTENT_TYPE);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivityForResult(intent, REQUEST_SELECT_CONTACT);
	    }
	}
	
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	    if (requestCode == REQUEST_SELECT_CONTACT && resultCode == RESULT_OK) {
	        Uri contactUri = data.getData();
	        // Do something with the selected contact at contactUri
	        ...
	    }
	}
	
#### 获得具体联系人信息 ####

	static final int REQUEST_SELECT_PHONE_NUMBER = 1;
	
	public void selectContact() {
	    // Start an activity for the user to pick a phone number from contacts
	    Intent intent = new Intent(Intent.ACTION_PICK);
	    intent.setType(CommonDataKinds.Phone.CONTENT_TYPE);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivityForResult(intent, REQUEST_SELECT_PHONE_NUMBER);
	    }
	}
	
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	    if (requestCode == REQUEST_SELECT_PHONE_NUMBER && resultCode == RESULT_OK) {
	        // Get the URI and query the content provider for the phone number
	        Uri contactUri = data.getData();
	        String[] projection = new String[]{CommonDataKinds.Phone.NUMBER};
	        Cursor cursor = getContentResolver().query(contactUri, projection,
	                null, null, null);
	        // If the cursor returned is valid, get the phone number
	        if (cursor != null && cursor.moveToFirst()) {
	            int numberIndex = cursor.getColumnIndex(CommonDataKinds.Phone.NUMBER);
	            String number = cursor.getString(numberIndex);
	            // Do something with the phone number
	            ...
	        }
	    }
	}

	参数：

		- CommonDataKinds.Phone.CONTENT_TYPE，表示获取联系人号码
		- 
		- CommonDataKinds.Email.CONTENT_TYPE，表示获取联系人Email
		- 
		- CommonDataKinds.StructuredPostal.CONTENT_TYPE，表示获取联系人的邮寄地址
		- 
		- 通过ContactsContract.CommonDataKinds中的其他字段，还可以获取更多联系人信息。

#### 查看联系人 ####

	public void viewContact(Uri contactUri) {
	    Intent intent = new Intent(Intent.ACTION_VIEW, contactUri);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

	参数：
		Action：ACTION_VIEW
		
		Data URI Scheme：content:<URI>，该Uri指向了查看的联系人地址，有两种方法可获得该Uri：
		
		通过上一小节的ACTION_PICK，获得返回的Uri。此方式无需申请权限。
		直接检索通讯录联系人（ Retrieving a List of Contacts）。此方式需要READ_CONTACTS权限。
		
		MIME Type：空

#### 编辑联系人信息 ####

	public void editContact(Uri contactUri, String email) {
	    Intent intent = new Intent(Intent.ACTION_EDIT);
	    intent.setData(contactUri);
	    intent.putExtra(Intents.Insert.EMAIL, email);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

	参数:

	Action：ACTION_EDIT

	Data URI Scheme：content:<URI>，获取该Uri的方式和权限与上一小节的获取方式一致。
	
	MIME Type：由Uri决定。
	
	Extras：ContactsContract.Intents.Insert


#### 新增联系人 ####

	public void insertContact(String name, String email) {
	    Intent intent = new Intent(Intent.ACTION_INSERT);
	    intent.setType(Contacts.CONTENT_TYPE);
	    intent.putExtra(Intents.Insert.NAME, name);
	    intent.putExtra(Intents.Insert.EMAIL, email);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

5. 拨打电话


	public void dialPhoneNumber(String phoneNumber) {
	    Intent intent = new Intent(Intent.ACTION_DIAL);
	    intent.setData(Uri.parse("tel:" + phoneNumber));
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

	需要权限：
	<uses-permission android:name="android.permission.CALL_PHONE" />

6. 打开某个设置选项

	public void openWifiSettings() {
	    Intent intent = new Intent(Intent.ACTION_WIFI_SETTINGS);
	    if (intent.resolveActivity(getPackageManager()) != null) {
	        startActivity(intent);
	    }
	}

	参数说明：

	Action：

		ACTION_SETTINGS
		
		ACTION_WIRELESS_SETTINGS
		
		ACTION_AIRPLANE_MODE_SETTINGS
		
		ACTION_WIFI_SETTINGS
		
		ACTION_APN_SETTINGS
		
		ACTION_BLUETOOTH_SETTINGS
		
		ACTION_DATE_SETTINGS
		
		ACTION_LOCALE_SETTINGS
		
		ACTION_INPUT_METHOD_SETTINGS
		
		ACTION_DISPLAY_SETTINGS
		
		ACTION_SECURITY_SETTINGS
		
		ACTION_LOCATION_SOURCE_SETTINGS
		
		ACTION_INTERNAL_STORAGE_SETTINGS
		
		ACTION_MEMORY_CARD_SETTINGS

7. 打开网页

		public void openWebPage(String url) {
		    Uri webpage = Uri.parse(url);
		    Intent intent = new Intent(Intent.ACTION_VIEW, webpage);
		    if (intent.resolveActivity(getPackageManager()) != null) {
		        startActivity(intent);
		    }
		}

	intent-filter：

		<activity ...>
		    <intent-filter>
		        <action android:name="android.intent.action.VIEW" />
		        <!-- Include the host attribute if you want your app to respond
		             only to URLs with your app's domain. -->
		        <data android:scheme="http" android:host="www.example.com" />
		        <category android:name="android.intent.category.DEFAULT" />
		        <!-- The BROWSABLE category is required to get links from web pages. -->
		        <category android:name="android.intent.category.BROWSABLE" />
		    </intent-filter>
		</activity>


参考地址：[http://blog.csdn.net/vanpersie_9987/article/details/51244558#rd](http://blog.csdn.net/vanpersie_9987/article/details/51244558#rd "隐式intent的强大之处")



### service需要注意的地方 ###

1.service运行在主线程中，只要进程存在，service就可以存在，对activity的依赖性很小。


----------
2016-08-01 10:22:26 
### ActivityThread管理着什么 ###

![activityThread功能说明](http://mmbiz.qpic.cn/mmbiz_png/v1LbPPWiaSt6oQKdJkbPZlRMPvGpHzv8L7v1JbHMljoGlEOMNJ2Ke8QMP77jgc3M2bqHSUxx77JIhEBu2iaq0j9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



----------
2016-08-09 16:04:04 

### SimpleAdapter作为 ListView的适配器，行布局中支持哪些组件 ###

根据SimpleAdapter的源码可以知道，使用SimpleAdapter作为适配器是，会按照如下顺序判断View:
 
1、该view是否实现checkable接口
 
2、该view是否是TextView
 
3、该view是否是ImageView
 
如果以上三种类型都不是，就会抛出IllegalStateExeception

----------
2016-08-10 16:50:26 
### Android里面为什么要设计出Bundle而不是直接用Map结构 ###

- Bundle内部是由ArrayMap实现的，ArrayMap的内部实现是两个数组，一个int数组是存储对象数据对应下标，一个对象数组保存key和value，内部使用二分法对key进行排序，所以在添加、删除、查找数据的时候，都会使用二分法查找，只适合于小数据量操作，如果在数据量比较大的情况下，那么它的性能将退化。而HashMap内部则是数组+链表结构，所以在数据量较少的时候，HashMap的Entry Array比ArrayMap占用更多的内存。因为使用Bundle的场景大多数为小数据量，我没见过在两个Activity之间传递10个以上数据的场景，所以相比之下，在这种情况下使用ArrayMap保存数据，在操作速度和内存占用上都具有优势，因此使用Bundle来传递数据，可以保证更快的速度和更少的内存占用。

- 另外一个原因，则是在Android中如果使用Intent来携带数据的话，需要数据是基本类型或者是可序列化类型，HashMap使用Serializable进行序列化，而Bundle则是使用Parcelable进行序列化。而在Android平台中，更推荐使用Parcelable实现序列化，虽然写法复杂，但是开销更小，所以为了更加快速的进行数据的序列化和反序列化，系统封装了Bundle类，方便我们进行数据的传输。

----------

----------

----------


### 特别篇一：一些经验的分享 ###
1. 在Android依赖库中使用switch-case语句访问资源ID时会报如下图所示的错误：
	![](http://i.imgur.com/HBZaWv8.jpg)

	错误原因：Android library中生成的R.java中的资源ID不是常数。

	解决办法：通过if-else-if条件语句来引用资源ID。

		![](http://i.imgur.com/Ha8tzXO.jpg)

2. 不能在Activity没有完全显示时显示PopupWindow和Dialog，如果需要，请保证在activity加载完成后在再show dialog。

3. 在多进程之间不要用SharedPreferences共享数据，虽然可以（MODE_MULTI_PROCESS），但极不稳定，如果需要，请使用ContentProvider 。

4. 

5. 当前Activity的onPause方法执行结束后才会执行下一个Activity的onCreate方法，所以在onPause方法中不适合做耗时较长的工作，这会影响到页面之间的跳转效率；

6. 使用Android的透明主题一定要谨慎，透明主题会导致很多问题，比如：如果新的Activity采用了透明主题，那么当前Activity的onStop方法不会被调用；在设置为透明主题的Activity界面按Home键时，可能会导致刷屏不干净的问题；进入主题为透明主题的界面会有明显的延时感；

7. 不要在非UI线程中初始化ViewStub，否则会返回null。
 
8. 不要通过Bundle传递大块的数据，否则会报TransactionTooLargeException异常，最大的数据量限制为1MB

9. 尽量不要使用An
10. imationDrawable，它在初始化的时候就将所有图片加载到内存中，特别占内存，并且还不能释放，释放之后下次进入再次加载时会报错；

10. 9patch图不能通过tinypng压缩，不然会有问题；

11. 使用Adapter的时候，如果你使用了ViewHolder做缓存，在getView的方法中无论这项的每个视图是否需要设置属性(比如TextView设置的属性可能为null，item的某一个按钮的背景为透明、某一项的颜色为透明等)，都需要为每一项的所有视图设置属性（textview的属性为空也需要设置setText(“”)，背景透明也需要设置），否则在滑动的过程中会出现内容的显示错乱。

12. 使用Toast时，建议定义一个全局的Toast对象，这样可以避免连续显示Toast时不能取消上一次Toast消息的情况（如果你有连续弹出Toast的情况，避免使用Toast.makeText）；

13. View的面积越大绘制的时间就越长，透明通道对View的绘制速度影响很大；

14. 不要通过Msg传递大的对象，会导致内存问题；

15. TextView（往往 TextView 派生子类同样适用）调用 setText 方法设置一个 int 型的数据，千万要将该值转为 String，否则在某些设备中它会默认去查询 R 文件中定义的资源（部分设备可能会报空指针异常）。 

16. 是好在每个Activity上加入

17. 
		<activity android:configChanges="orientation|keyboardHidden"> 

18. 少用Thread，而多使用AsyncTask。

19. 主线程只做UI控制和Frameworks回调相关的事。附属线程只做费时的后台操作。交互只通过Handler。这样就可以避免大量的线程问题。 

20. asset下使用第三方字体 xx.ttf 必须都为小写 。

21. 不要在Application对象中缓存数据化，这有可能会导致你的程序崩掉。请使用Intent在各组件之间传递数据，抑或是将数据存储在磁盘中,然后在需要的时候取出来。


[https://github.com/futurice/android-best-practices/blob/master/translations/Chinese/README.cn.md](https://github.com/futurice/android-best-practices/blob/master/translations/Chinese/README.cn.md)



### 特别篇二：一些代码重构的经验 ###

#### No.1：重复代码的 提炼 ####

	优点：总代码量大大减少，维护方便，代码条理更加清晰易读。
	原则：寻找代码当中完成某项子功能的重复代码，找到以后请毫不犹豫将它移动到合适的方法当中，并存放在合适的类当中。
实例：

	![](http://i.imgur.com/Sbqpw37.jpg)

#### No.2：冗长方法的 分割 ####

	优点：代码条理更加清晰易读。
	原则：分割一个大方法时，大部分都是针对其中的一些子功能分割
	注意点：需要给每一个子功能起一个恰到好处的方法名（很重要）
实例：

![](http://i.imgur.com/TNSR6Ft.jpg)

#### No.3-1：嵌套条件分支的优化（1） ####

	优点：代码条理更加清晰易读。（卫语句）
	原则：将不满足某些条件的情况放在方法前面，并及时跳出方法，以免对后面的判断造成影响。
实例：

![](http://i.imgur.com/X4jXthn.jpg)

#### No.3-2：嵌套条件分支的优化（1） ####
当没办法使用上面那个技巧的时候，使用下面这个

实例：
	
![](http://i.imgur.com/O9J6xZ8.jpg)

#### No.4：去掉一次性的 临时变量 ####

	优点：减少代码量，代码条理更加清晰易读。
	过程：
		第一步： 
			找出所有的一次性变量
		第二步：
			去掉他们！

实例：

![](http://i.imgur.com/7qDUEKd.jpg)

#### No.5：消除过长参数列表 ####

	优点：减轻代码阅读难度，减少思维发散，代码条理更加清晰易读。
	原则：将这些参数封装成一个对象传递给方法，从而去除过长的参数列表。

实例：

![](http://i.imgur.com/mWTwfXE.jpg)

#### No.6：提取类或继承体系中的常量 ####

	优点：便于维护，是代码结构更灵活
	原则：为了消除一些魔数或者是字符串常量等等。

实例：

![](http://i.imgur.com/stkqxud.jpg)

#### No.7：让类提供应该提供的方法 ####

	优点：大幅度减少困难的重复代码。
	原则：让这个类做它该做的事情，而不应该让我们替它做。

实例：

![](http://i.imgur.com/RpMVMrQ.jpg)
![](http://i.imgur.com/jWBcBKJ.jpg)


#### No.8：拆分冗长的类 ####

	尽可能的简化一个类...

#### No.9：提取继承体系中重复的属性与方法到父类 ####



----------


2016-06-12 14:39:24 
### 自定义View 主题 ###

#### Configuration类 ####

	Configuration config = getResources().getConfiguration();
	//获取国家码
	int countryCode = config.mcc;
	//获取网络码
	int networkCode = config.mnc;
 	//判断横竖屏
	if(config.orientation==Configuration.ORIENTATION_PORTRAIT){
		//竖屏
	}else{
		//横屏
	}

#### ViewConfiguration类 ####

	ViewConfiguration viewConfiguration = ViewConfiguration.get(context);

	//获取touchSlop（最小活动距离，当滑动距离大于这个值时才判定滑动动作）
	int touchSlop = viewConfiguration.getScaledTouchSlop();

	//获取滑动速度的最小值最大值
	int minVeloctity = viewConfiguration.getScaledMinimumFlingVelocity();
	int maxVeloctity = viewConfiguration.getScaledMaximumFlingVelocity();

	//判断物理按键是否存在	
	boolean isHavePermanentMenuKey = viewConfiguration.hasPermanentMenuKey();

	//双击间隔时间，在该时间内是双击还是单击
    int doubleTapTimeout = ViewConfiguration.getDoubleTapTimeout();

	//长按状态需要的时间
	int longPressTimeout = ViewConfiguration.getLongPressTimeout();

	//重复按键时间
	int keyRepeatTimeout = ViewConfiguration.getKeyRepeatTimeout();

#### GestureDetector类 (简化touch操作)####
#### VelocityTracker类 (跟踪触摸事件的速率)####
#### ViewDragHelper类 （View拖拽相关操作）####
#### Scroller类 ####

- 	scrollTo()和scrollBy()的关系

	scrollBy()最终调用了scrollTo()，scrollBy()是scrollTo()的封装。

-   scroll的本质
  
    scroll只是滑动View的内容。


	//	
    //
	//
	//

### MeasureSpec解析 ###

由mode和size组成，表示父View对子View宽高的要求，用一个32位整型数据表示，最高的两位表示mode，剩下的30位表示大小。子view的MeasureSpec是由自己的size和父view的MeasureSpec共同决定的。


	//获取mode
	int specMode = MeasureSpec.getMode(measureSpec);
	//获取size
	int specSize = MeasureSpec.getSize(measureSpec);
	//生成MeasureSpec
	int meaasureSpec = MeasureSpec.makeMeasureSpex(size,mode);

	三种mode：
		MeasureSpec.EXACTLY（精确模式） 使用子view的具体值，这个数值为 1<<30  
		MeasureSpec.AT_MOST  使用父view的具体值（match_parent）,这个数值为 2<<30
		MeasureSpec.UNSPECIFIED  父容器不对子view做出大小限制。不常用。这个数值为 0<<30

在View类中的getChildMeasureSpec()方法里去确定子view的MeasureSpec。
![](http://i.imgur.com/XlLgHwt.png)

### onMeasure()源码流程 ###

（1）在onMeasure中调用setMeasuredDimension()设置View的宽高。

（2）在setMeasuredDimension()中调用getDefaultSize()获取View的宽高。

（3）在getDefaultSize()中调用getSuggestedMinimumWidth()或getSuggestedMinimumHight()获取到View的宽高的最小值。


##### getSuggestedMinimumWidth()/getSuggestedMinimumHight() #####

return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());

根据背景判断，如果背景为空，则返回宽/高的最小值（这个值可以在xml里设置，如果没有设置，默认为0）；

##### getDefaultSize() #####

	public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }

以上过程表明在Measure阶段，view的宽高是由MeasureSpec的size决定。

##### setMeasuredDimension() #####

把获取到的view 的宽高设置到view中。


### onLayout()源码流程 ###

setFrame() ----->  确定子view在父view中的位置。
layout方法是view用与确定自己在它父view的位置。
ViewGroup的onLayout()是用来确定子view的位置。

问题：getMeasuredWidth()和getWidth()的区别
答： 在onMeasure()执行完毕后通过getMeasuredWidth()就可以获取到相应的值，而getWidth()想要获取到相应的值必须要在onLayout()后才能获取到。除此之外，两者的计算方式也不一样，getWidth()= 右坐标 - 左坐标
getMeasuredWidth() = onMeasure()中的setMeasuredDimension（width, height）的相应值。

**onLayout(l,t,r,b)的四个参数所代表的含义：**
![](http://i.imgur.com/scUs6Xn.png)


### onDraw()源码流程 ###

draw的步骤：
1.绘制background

（2.如果必要的话，保存canvas的状态）

3.绘制view的内容（核心）

4.绘制子view

(5.如果有必要的话，绘制视图滑动时边框的渐变效果）

6.绘制scrollbar