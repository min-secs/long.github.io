---
title: ListView结合xUtils仿微信实现分页
date: 2021-2-19 17:12:42
tags: 
categories: Android日记

---

>前言:最近涉及到和QQ打交道,定义所有的好友一共只能有300条消息,如果一次性从数据库读取300条或者更多,界面会有细微的卡顿.所以考虑了下分页,第一次进来只显示20条(仿微信),当用户滑到第一条后,如果数据库有消息,则再加载20条.

##步骤-问把大象关冰箱,总共分几步?
###1.自定义absListview.scrollListerner
核心的东西是监听ListView的scrollListerner,网上扒了一个挺不错的,大家用的时候实现这个scrollListerner,完善自己的逻辑即可
```java
public class MyOnScrollListener implements OnScrollListener {
 private int totalItemCount;
      //ListView最后的item项
	  private int lastItem;
      //listview第一项
	  private int firstItem;
	    //用于判断当前是否在加载
	  private boolean isLoading;
	    //底部加载更多布局
	  private View footer;
	    //接口回调的实例
	  private OnloadDataListener listener;
	 
	  //数据
	  private List<MsgBean> data;
	  Handler handler = new Handler();
     
	 
	  public MyOnScrollListener(View footer, List<MsgBean> data) {
	    this.footer = footer;
	    this.data = data;
	  }
	  //设置接口回调的实例
	  public void setOnLoadDataListener(OnloadDataListener listener) {
	    this.listener = listener;
	  }
	   /**
	   * 滑动状态变化
	   *
	   * @param view
	   * @param scrollState 1 SCROLL_STATE_TOUCH_SCROLL是拖动  2 SCROLL_STATE_FLING是惯性滑动 0SCROLL_STATE_IDLE是停止 , 只有当在不同状态间切换的时候才会执行
	   */
	  @Override
	   public void onScrollStateChanged(AbsListView view, int scrollState) {
	    //如果数据没有加载，并且滑动状态是停止的,并且滚到了第一个item,可在此做下拉更新或者上拉更新的判断
	    if (!isLoading && firstItem == 0 && scrollState == SCROLL_STATE_IDLE) {
	      //显示加载更多
	      footer.setVisibility(View.VISIBLE);
	    
	      //模拟一个延迟两秒的刷新功能
	       handler.postDelayed(new Runnable() {
	        @Override
	        public void run() {
	          if (listener != null) {
	            //开始加载更多数据
	            loadMoreData();
	            //回调设置ListView的数据
	            listener.onLoadData(data);
	             //加载完成后操作什么
	            loadComplete();
	          }
	        }
	      }, 2000);
	    }
	  }
	   /**
	   * 当加载数据完成后，设置加载标志为false表示没有加载数据了
	   * 并且设置底部加载更多为隐藏
	   */
	  private void loadComplete() {
	    isLoading = false;
	    footer.setVisibility(View.GONE);
	 
	  }
	    /**
	   * 开始加载更多新数据，这里每次只更新三条数据
	   */
	  private void loadMoreData() {
	    isLoading = true;
	    MsgBean msg = null;
	    for (int i = 0; i < 3; i++) {
	      msg = new MsgBean();
	     msg .setRemark("Liming"+i);
	     msg .setMsgID(i);
	      data.add(stu);
	    }
	  }
	    /**
	   * 监听可见界面的情况
	   *
	   * @param view       ListView
	   * @param firstVisibleItem 第一个可见的 item 的索引
	   * @param visibleItemCount 可以显示的 item的条数
	   * @param totalItemCount  总共有多少个 item
	   */
	  @Override
	   public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
	    //实现下拉加载
	    lastItem = firstVisibleItem + visibleItemCount;
           //实现上拉加载
	    firstItem = firstVisibleItem;
	     //总listView的item个数
	    this.totalItemCount = totalItemCount;
	  }
	  //回调接口
	  public interface OnloadDataListener {
	    void onLoadData(List<MsgBean> data);
	  }
	 }
```
###2.实现此接口
```java
public class ListPageActivity extends Activity implements MyOnScrollListener.OnloadDataListener  {
 @Override
	  protected void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_list_page);
	
	    //显示到ListView上
	    showListView(data);
 //自定义的滚动监听事件
	    MyOnScrollListener onScrollListener = new MyOnScrollListener(header, data);
	    //设置接口回调
	    onScrollListener.setOnLoadDataListener(this);
	    //设置ListView的滚动监听事件
	    mListView.setOnScrollListener(onScrollListener);

  @Override
	  public void onLoadData(List<MsgBean> data) {
	    //加载数据完成后，展示数据到ListView
	    showListView(data);
	  }
}
```
showListView里面无疑是普通的更新adapter的工作
那么我们如何借助xutils的数据库进行分类呢?
###3.利用xutils数据库操作进行分页处理
首先,我们理一下思路,上面我们已经实现了上拉的回调,在此回调中把新来的数据加载到adapter即可.
//下文db是Dbmanager的实例,可参考[xutils3用法](http://www.jianshu.com/p/95b6c4d7b7ce)

    /**
     * 当前屏幕显示的消息数量
     */
    private int MAX_MSG_NUMBER = 20;
```java
private List<MsgBean> getDataFromDb() {

	 List<?> dbSize = db.selector(MsgBean.class).where(WhereBuilder.b("id", "=", 400)).findAll();//记得捕获null指针和DbException异常
//如果数据库比我们显示的页数小,则不偏移,否则,偏移到我们需要显示的位置
		if (dbSize.size() < MAX_MSG_NUMBER) {
			indexOffset = 0;
		} else {
			indexOffset = dbSize.size() - MAX_MSG_NUMBER;
		}
	 
	 List<MsgBean> datas = db.selector(MsgBean.class).where(WhereBuilder.b("id", "=", 400)).limit(MAX_MSG_NUMBER)
					.offset(indexOffset).findAll();
		return datas;
	}
```

>这里解释一下          

    db.selector(MsgBean.class).where(WhereBuilder.b("id", "=", 400)).limit(MAX_MSG_NUMBER).offset(indexOffset).findAll();是我们实现分页的关键
.limit是我们定义的分页大小
.offset偏移量,我们数据库的大小是不变的,如果不定义偏移量,那么我们定义的分页大小每次只从0取到19.假设数据库中有21条数据,那么我们需要从1取到20,而不是0到19,所以偏移1.

然后我们在loadMoreData中

    MAX_MSG_NUMBER += MAX_MSG_NUMBER;
    getDataFromDb();
将大小自加,即完成加载更多的功能,在onLoadData(List<MsgBean> data)中加载数据即可.
***
后面贴上我对xutils数据库操作的封装,还有很多不完善之处
```java
/**
 * 数据库 xutils用法
 * @author 青楼爱小生
 */
public class DbUtil {
	private static final String TAG = DbUtil.class.getName();
	private static DbUtil dbUtil;
	private DbManager db;
	private DbUtil(){
		db = x.getDb(MyApplication.getInstance().daoConfig);
	}
	public static DbUtil getInstance(){
		if(dbUtil == null){
			synchronized (DbUtil.class) {
				if(dbUtil == null){
					dbUtil = new DbUtil();
				}
			}
		}
		return dbUtil;
	}
	/**
	 * 增加数据
	 * @param list
	 * @throws DbException 
	 */
	public void addMsgList(List<MsgBean> list) {
		try {
			db.saveOrUpdate(list);
		} catch (DbException e) {
			e.printStackTrace();
			LogHelper.e(TAG, e.getMessage());
		}
	}
	/**
	 * 增加一条数据
	 * @param node
	 * @throws DbException
	 */
	public void addMsgToDb(MsgBean node) {
		try {
			db.saveOrUpdate(node);
		} catch (DbException e) {
			e.printStackTrace();
			LogHelper.e(TAG, e.getMessage());
		}
	}
	/**
	 * 删除表中所有数据
	 * @param cls  创建的表的映射
	 * @throws DbException 
	 */
	public void deleteAll(Class cls) {
		try {
			db.delete(cls);
		} catch (DbException e) {
			LogHelper.e(TAG, e.getMessage());
			e.printStackTrace();
		}
	}
	/**
	 * 删除第一条数据
	 * @param cls
	 */
	@SuppressWarnings("unchecked")
	public void deleteFirst(Class cls){
		try {
			db.delete(db.findFirst(cls));
		} catch (DbException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	/**
	 * 查询表中所有数据
	 * @throws DbException 
	 */
	@SuppressWarnings("unchecked")
	public List<?> findAll(Class cls) {
		try {
			return db.findAll(cls) == null ? Collections.emptyList() : db.findAll(cls);
		} catch (DbException e) {
		e.printStackTrace();
			LogHelper.e(TAG, e.getMessage());
			return Collections.emptyList();
		}
	}
	/**
	 * //添加查询条件进行查询
		List<ChildInfo> all = db.selector(ChildInfo.class).where("id",">",2).and("id","<",4).findAll();
	 * @return 搜索指定条件的数据
	 */
	 @SuppressWarnings("unchecked")
	public List<?> findDataByWhere(Class cls,WhereBuilder format){
		try {
			return db.selector(cls).where(format).findAll()== null ?
					Collections.emptyList() :db.selector(cls).where(format).findAll();
		} catch (DbException e) {
		LogHelper.e(TAG, e.getMessage());
			e.printStackTrace();
			return Collections.emptyList();
		}
	
	}
	/**
	 * 添加查询条件进行查询
	 * @param cls  表映射
	 * @param str  select语句
	 * @param format  where语句
	 * @return List<DbModel> DbModel  key为数据库列名 value为值
	 * eg:(Selector.from(Parent.class)  
                                   .where("id" ,"<", 54)  
                                   .and(WhereBuilder.b("age", ">", 20).or("age", " < ", 30))  
                                   .orderBy("id")  
                                   .limit(pageSize)    .offset(pageSize * pageIndex));  
	 * 
	 * 
	 * 
	 */
	 @SuppressWarnings("unchecked")
	public Selector<?> findDataBySelector(Class cls,WhereBuilder format){
		try {
			return  db.selector(cls).where(format);
		} catch (DbException e) {
		// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}
		
}
```

