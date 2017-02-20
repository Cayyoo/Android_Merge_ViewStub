# Merge、ViewStub等UI性能优化
 * 1、ViewStub
 * 2、merge
 * 3、include
 * 4、Lint等Analyze工具

使用要点：
 ```java
  /**
 * Android.view.ViewStub，延迟加载，根据需求显示或不显示某个布局。
 * ViewStub是一个轻量级的View，它是一个看不见的，不占布局位置，占用资源非常小的控件。
 * 可以为ViewStub指定一个布局，在Inflate布局的时候，只有ViewStub会被初始化，
 * 然后当ViewStub被设置为可见的时候，或是调用了ViewStub.inflate()的时候，ViewStub所指向的布局就会被Inflate和实例化，然后ViewStub的布局属性都会传给它所指向的布局。
 * 这样，就可以使用ViewStub来方便的在运行时，要还是不要显示某个布局。
 *
 * <ViewStub />是一个不可见的，大小为0的View。
 *
 * 注：ViewStub目前还不支持 <merge /> 标签。
 */
 
 
 /**
 * ViewStub的一些特点：
 * 1. ViewStub只能Inflate一次，之后ViewStub对象会被置为空。按句话说，某个被ViewStub指定的布局被Inflate后，就不会够再通过ViewStub来控制它了。
 * 2. ViewStub只能用来Inflate一个布局文件，而不是某个具体的View，当然也可以把View写在某个布局文件中。
 *
 * 1.当布局文件inflate时，ViewStub控件虽然也占据内存，但是相相比于其他控件，ViewStub所占内存很小；
 * 2.布局文件inflate时，ViewStub主要是作为一个“占位符”的性质，放置于view tree中，且ViewStub本身是不可见的。
 *   ViewStub中有一个layout属性，指向ViewStub本身可能被替换掉的布局文件，在一定时机时，通过viewStub.inflate()完成此过程；
 * 3.ViewStub本身是不可见的，对ViewStub setVisibility(..)与其他控件不一样，ViewStub的setVisibility 成View.VISIBLE或INVISIBLE如果是首次使用，
 *   都会自动inflate其指向的布局文件，并替换ViewStub本身，再次使用则是相当于对其指向的布局文件设置可见性。
 */
 
 
 
 /**
 * 使用ViewStub的情况有：
 * 1. 在程序的运行期间，某个布局在Inflate后，就不会有变化，除非重新启动。
 *    因为ViewStub只能Inflate一次，之后会被置空，所以无法指望后面接着使用ViewStub来控制布局。
 *    所以当需要在运行时不止一次的显示和隐藏某个布局，那么ViewStub是做不到的。
 *    这时就只能使用View的可见性来控制了。
 * 2. 想要控制显示与隐藏的是一个布局文件，而非某个View。
 *    因为设置给ViewStub的只能是某个布局文件的Id，所以无法让它来控制某个View。
 *    所以，如果想要控制某个View(如Button或TextView)的显示与隐藏，或者想要在运行时不断的显示与隐藏某个布局或View，只能使用View的可见性来控制。
 */
 
 
 
 /**
 * 使用ViewStub注意事项：
 * 某些布局属性要加在ViewStub而不是实际的布局上面，才会起作用，
 * 比如上面用的android:layout_margin*系列属性，如果加在TextView上面，则不会起作用，需要放在它的ViewStub上面才会起作用。
 * 而ViewStub的属性在inflate()后会都传给相应的布局。
 */
 
 
 
 /**
 * <merge />
 * 用于减少视图层级
 * 使用merge时优化UI层级时，视图的根节点必须是FrameLayout，否则无效（使用其他根节点布局，如LinearLayout时，层级优化无效，本例未验证使用LinearLayout是否无效，暂时使用FrameLayout作为根节点）。
 * merge可使多余的FrameLayout节点被合并在一起，或者可以理解为将merge标签中的子集直接加到Activity的FrameLayout根节点下。
 * include和merge是配合使用的，不是一个互斥的或者说是平级的关系。
 *
 *（1）merge只能用在布局XML文件的根元素
 *（2）使用merge来inflate一个布局时，必须指定一个ViewGroup作为其父元素，并且要设置inflate的attachToRoot参数为true。
 *    （参照inflate(int, ViewGroup, boolean)）
 *（3）不能在ViewStub中使用merge标签。最直观的一个原因就是ViewStub的inflate方法中根本没有attachToRoot的设置
 *    ViewStub标签中导入的layout布局，其根节点不能使用merge标签。
 *
 * <merge />只可以作为xml layout的根节点。
 * 当需要扩充的xml layout本身是由merge作为根节点的话，需要将被导入的xml layout置于 viewGroup中，同时需要设置attachToRoot为True。
 *
 * 当应用Include从外部导入xml结构时，可以将被导入的xml用merge作为根节点表示，
 * 这样当被嵌入父级结构中后可以很好的将它所包含的子集融合到父级结构中，而不会出现冗余的节点。
 *
 *
 * 1、merge布局 和FrameLayout类似,相同的效果。不同的是 merge布局只能被<include>标签包含，或者Activity.setContentView所使用。
 *   当LayoutInflater遇到能被其他layout用<include>包含进去，并不再另外生成ViewGroup容器，本元素也特别有用这个标签时，它会跳过它，
 *   并将<merge />内的元素添加到<merge />的父元素里。Activity能直接使用的原因是Activity的父元素是FrameLayout。
 * 2、 merge 能被其他layout用<include>包含进去，并不再另外生成ViewGroup容器。就是说,会减少一层layout到达优化layout的目的
 *
 * 1、merge必须放在布局文件的根节点上。
 * 2、merge并不是一个ViewGroup，也不是一个View，它相当于声明了一些视图，等待被添加。
 * 3、merge标签被添加到A容器下，那么merge下的所有视图将被添加到A容器下。
 * 4、因为merge标签并不是View，所以在通过LayoutInflate.inflate方法渲染的时候， 第二个参数必须指定一个父容器，且第三个参数必须为true，也就是必须为merge下的视图指定一个父亲节点。
 * 5、如果Activity的布局文件根节点是FrameLayout，可以替换为merge标签，这样，执行setContentView之后，会减少一层FrameLayout节点。
 * 6、自定义View的时候，根节点如果需要设置成LinearLayout，建议让自定义的View节点设置成merge，然后自定义View
 * 7、因为merge不是View，所以对merge标签设置的所有属性都是无效的。
 */
 
 
 
 /**
 * Lint
 *
 * 布局文件中通过使用 Lint 来查找可能的布局优化。Lint 现在已经代替了 Layoutopt，而且更有效率，下面是 Lint 的一些规则。
 * 1、尽量使用复合图片 ，一个线性布局中如果包含一个 ImageView 和一个 TextView，那么你可以使用复合图片来替换。
 * 2、去掉不需要的根节点 ，如果一个 FrameLayout 是整个布局的根节点，并且他没有提供背景，留白之类的东西，
 *    那么我们可以使用 merge 标签来让他变得更有效。
 * 3、减少布局中的枝叶 ，如果一个布局没有子 View 或者背景，那么他可以被移除掉（况且他本身就是不可见的）来让布局更有效。
 * 4、减少父母层级 ，如果一个布局没有兄弟，并且他不是 ScrollView 或者根 View，并且也没有背景，那么他就可以直接被移除掉，
 *    他的孩子可以直接被移到他父母的层级下。
 * 5、避免过深的布局层级 ，多次嵌套的布局文件不利于性能。你可以考虑通过相对布局或者网格布局来提升性能，默认的最深布局深度是10。
 */
 
 ```
 

