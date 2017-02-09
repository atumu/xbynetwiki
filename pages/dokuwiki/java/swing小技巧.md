title: swing小技巧 

#  Swing技巧与自适应布局 
##  LookAndFeel 
推荐采用此大牛开发的：http://www.cnblogs.com/jb2011/archive/2012/09/12/2681579.html
```

<!-- 这是LookAndFeel的依赖 -->
	<dependency>
			<groupId>org.jb2011</groupId>
			<artifactId>beautyeye_lnf</artifactId>
			<version>3.5</version>
		</dependency>

```
```

		try {
			UIManager.put("RootPane.setupButtonVisible", false);
			BeautyEyeLNFHelper.launchBeautyEyeLNF();
		} catch (Exception e) {
		}
	 	SwingUtilities.updateComponentTreeUI(mainFrame);

```
##  常规窗体技巧 

```

	 	JFrame mainFrame=new MainFrame();
	 	mainFrame.setVisible(true);
	 	mainFrame.pack();//延迟计算大小，这样才能保证界面布局正常。
		
//	 是启动窗口最大化 mainFrame.setExtendedState(JFrame.MAXIMIZED_BOTH);
	 	//是窗口居中显示
	 	mainFrame.setLocationRelativeTo(null);
	 	//禁用窗口最大化
	 	 mainFrame.setResizable(false);

```
```

/**
	 * 初始化窗体关闭策略
	 */
	private void initWindowClose(){
		setDefaultCloseOperation(DO_NOTHING_ON_CLOSE);
		addWindowListener(new WindowAdapter() {
			@Override
			public void windowClosing(WindowEvent e) {
				// TODO Auto-generated method stub
				int a = JOptionPane.showConfirmDialog(null, "确定关闭吗？", "温馨提示",
					      JOptionPane.YES_NO_OPTION);
			    if (a == 0) {  		 
				    	System.exit(0);  //关闭
					}
				}
		});
	}

```
##  Swing JLabel显示html 
```

JLabel label=new JLabel("<html><font color='red'>参数不合法，请重新输入</font>");注意必须在前面加上<html>否则不识别。

```
##  关于Swing自适应布局 
Swing中的` BorderLayout `能够自适应窗体大小的改变来改变布局。并重新计算组件大小重新排列。
所以如果要想Swing窗体能够自适应的话，需要随时使用该布局。
时推荐一个非常灵活的支持在BorderLayout中自适应的布局组件` MigLayout `:http://www.miglayout.com/
关于MigLayout可参考文档：
http://www.miglayout.com/QuickStart.pdf
http://www.hakkaku.net/articles/20090810-515
http://www.cnblogs.com/waising/p/3858539.html
**MigLayout中fill与grow,growx,growy可以与BorderLayout组合实现自适应**

**MigLayout**
1. 初始化：
  * MigLayout l = new MigLayout();
  * MigLayout l = new MigLayout("","","");三个字符串参数
主要使用的是以上两种构造函数，第一种无参的就不用介绍了，主要是第二种。
第一个参数可以使用以下语句：
  * ` wrap ` + 数字：指定默认在第几个组件后进行换行，如： wrap 2 表示在第二个组件后进行分行。
  * ` insets ` + 数字：指定默认与边界的距离，有两种方式：insets 10 表示四边均为10，insets 1 2 3 4 分别指示顶部、左侧、下部、右侧距离

第二个参数：指定横向的单元格的各个属性。每个单元格的属性用[]括起来。如[][][]表示一行有三个单元格。各单元格可以定制的属性如下：
  * ` grow `: x 方向按上一级的宽度进行延伸。**注意如果此处不添加grow，那么在添加组件的时候使用` growx `会没有效果。** ` grow和fill用于自适应 `
  * ` 40! `: 表明该列的单元格宽度固定为40
  * ` 10:30:40 `：表明该列的单元格宽度最小为10、最佳为30、最大为40
  * ` ::40 `：表明该单元格最大值为４０。其它也可以是:30: 或者：30::等。
  * ` center/right/left `： 指定该组件在水平方向的对齐方式
同时，也可以指定各个单元格之间的间隔：` []30[][] `说明第一个和第二个单元格之间相隔30

第三个参数：指定纵向的单元格的各个属性。
  * ` grow `: 指定纵向是否进行延伸
  * ` ::: ` ：指定高度属性，与第二个参数意义一致。
  * ` top/bottom/center `：指定在垂直方向的对齐方式。

**2. 添加组件：**
panel.add(button,"");
""里面使用各个参数，经常使用的有以下参数：
  * ` growx `: 在水平方向延伸
  * ` growy `：在垂直方向延伸
  * ` span `：占用本行的所有单元格
  * span 2: 占用横向的两个单元格
  * span 2 3: 占用横向两个、纵向三个单元格
  * ` wrap `: 添加本组件后进行分行 
  * ` gapleft/gapright/gaptop/gapbottom `: 指定四周的间隔
  * ` split ` 2: 将该单元格分成两个单元格
  * ` h ::: ` ：指定高度属性，如h 10:20:30 或者h 10! 或者h ::20或者h :20:或者h 20::等。
  * ` w ::: ` ：指定宽度属性
```

	<!-- 这是LookAndFeel的依赖 -->
	<dependency>
			<groupId>org.jb2011</groupId>
			<artifactId>beautyeye_lnf</artifactId>
			<version>3.5</version>
		</dependency>
	<!-- 这是migLayout的依赖 -->
  		<dependency>
			<groupId>com.miglayout</groupId>
			<artifactId>miglayout-swing</artifactId>
			<version>5.0</version>
		</dependency>

```
示例：
```

public class MainFrame extends JFrame {
	private static final double VERSIONCODE=1.0;
	private static final String VERSION="-----Version:"+VERSIONCODE;
	
	private JPanel contentPanel;
	/**
	 * 启动Frame
	 */
	public static void startFrame(){
		try {
			UIManager.put("RootPane.setupButtonVisible", false);
			BeautyEyeLNFHelper.launchBeautyEyeLNF();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	 	JFrame mainFrame=new MainFrame();
	 	mainFrame.setVisible(true);
	 	mainFrame.pack();
//	 	mainFrame.setExtendedState(JFrame.MAXIMIZED_BOTH);
	 	//是窗口居中显示
	 	mainFrame.setLocationRelativeTo(null);
	 	//禁用窗口最大化
	 	 mainFrame.setResizable(false);
	 	SwingUtilities.updateComponentTreeUI(mainFrame);
		
	}
	
	public MainFrame(){
		super();
		init();
		
	}
	private  void init(){
		initWindowClose();
		//初始化窗体框架，如标题，图标，字体等等。
		initFrame();
		//初始化窗体布局内容
		initContent();
	}
	/**
	 * 初始化窗体布局内容
	 */
	private void initContent(){
		//上，选择输出目录
		JPanel topPanel=new JPanel(new MigLayout("wrap","100[120px,fill,right]5[grow,left]20[]100","30[]10"));
		contentPanel.add(topPanel,BorderLayout.NORTH);
		JLabel outPutDirLabel=new JLabel("请选择输出文件目录:");
		final JTextField outPutDirField=new JTextField(50);
		String baseDir=System.getProperty("user.home");
		System.out.println(baseDir);
		JButton fileChooseBtn=GuiUtils.fastFileChooser(this, null, baseDir, JFileChooser.DIRECTORIES_ONLY, new CallBack(){

			public void callback(Object selectedDir) {
				// TODO Auto-generated method stub
				if(selectedDir!=null){
					outPutDirField.setText(selectedDir.toString());
				}
			}
			
		});
		topPanel.add(outPutDirLabel);
		topPanel.add(outPutDirField,"growx");
		topPanel.add(fileChooseBtn);
		
		
		//中，主体内容
		JPanel centerBg=new JPanel(new BorderLayout());
		contentPanel.add(centerBg, BorderLayout.CENTER);
		
		JPanel centerPanel=new JPanel(new MigLayout("wrap","100[60px,fill,right]5[grow,left]100","20[]20[]20[]20[]30"));
		centerBg.add(centerPanel,BorderLayout.CENTER);
		
		
		JLabel nameLabel=new JLabel("姓名");
		JTextField nameField=new JTextField(40);
		centerPanel.add(nameLabel);
		centerPanel.add(nameField,"growx");
		
		JLabel compLabel=new JLabel("公司");
		JTextField compField=new JTextField(40);
		centerPanel.add(compLabel);
		centerPanel.add(compField,"growx");
		
		JLabel timeLabel=new JLabel("有效期");
		JTextField timeField=new JTextField(40);
		centerPanel.add(timeLabel);
		centerPanel.add(timeField,"growx");
		
		//下面按钮布局
		JPanel bottomPanel=new JPanel(new MigLayout());
		centerPanel.add(bottomPanel,"cell 1 3,right");
		JButton genBtn=new JButton("生成");
		bottomPanel.add(genBtn);

	}
	/**
	 * 初始化窗体框架，如标题，图标，字体等等。
	 * 初始化容器面板。
	 */
	private void initFrame(){
		setTitle("Sword Plus Licence生成器"+VERSION);
		contentPanel = new JPanel();
//		contentPanel.setLayout(new MigLayout("wrap","[grow]",""));
		contentPanel.setLayout(new BorderLayout());
		setContentPane(contentPanel);
	}
	/**
	 * 初始化窗体关闭策略
	 */
	private void initWindowClose(){
		setDefaultCloseOperation(DO_NOTHING_ON_CLOSE);
		addWindowListener(new WindowAdapter() {
			@Override
			public void windowClosing(WindowEvent e) {
				// TODO Auto-generated method stub
				int a = JOptionPane.showConfirmDialog(null, "确定关闭吗？", "温馨提示",
					      JOptionPane.YES_NO_OPTION);
			    if (a == 0) {  		 
				    	System.exit(0);  //关闭
					}
				}
		});
	}
}

```
一些配置参数的记录
```

migLayout
### =
Created Thursday 27 August 2015

LayoutConstraints
-----------------
wrap
panel.add(comp3, "wrap") // Wrap to next row
panel.add(comp4)

MigLayout layout = new MigLayout("wrap 3");

panel.add(comp3, "wrap 15")

span
panel.add(comp1)
panel.add(comp2, "span 2") // The component will span two cells.
panel.add(comp3, "wrap") // Wrap to next row
panel.add(comp4, "span") // Span without "count" means span whole row.


panel.add(comp1);
panel.add(comp2, "span 2 2"); // The component will span 2x2 cells.
panel.add(comp3, "wrap"); // Wrap to next row
panel.add(comp4);
panel.add(comp5, "wrap"); // Note that it "jumps over" the occupied cells.

split
panel.add(comp1);
panel.add(comp2, "split 2"); // Split the cell in two
panel.add(comp3); // Will be in same cell as previous
panel.add(comp4, "wrap"); // Wrap to next row
panel.add(comp5);

根据行列坐标布局：
panel.add(comp1, "cell 0 0"); // "cell column row"
panel.add(comp2, "cell 1 0");
panel.add(comp3, "cell 2 0");
panel.add(comp4, "cell 0 1");

panel.add(comp1, "cell 0 0");
panel.add(comp2, "cell 1 0 2 1"); // "cell column row width height"
panel.add(comp3, "cell 3 0");
panel.add(comp4, "cell 0 1 4 1");

指定Gap
-----
Grid Gaps:
MigLayout layout = new MigLayout(
"", // Layout Constraints
"[][]20[]", // Column constraints
"[]20[]"); // Row constraints

Component Gaps
panel.add(comp1)
panel.add(comp2, "gapleft 30") //"gapbefore" and "gaptop".
panel.add(comp3, "wrap") // Wrap to next row
panel.add(comp4)

Component Sizes
The sizes are specified in theform: "min:preferred:max" (E.g. "10:20:40").
panel.add(comp, "width 10:20:40");
panel.add(comp, "height ::40"); // Same as "hmax 40".
panel.add(comp, "w 40!"); // w is short for width.

Panel Insets
MigLayout layout = new MigLayout("insets 10");
MigLayout layout = new MigLayout("insets 0 10 10 20"); // T, L, B, R.

Component Alignment
panel.add(comp, "align left");

Growing and Shrinking Components Depending on Available Space
-------------------------------------------------------------
panel.add(comp, "growx") // Grow horizontally. Same as "growx 100"
panel.add(comp, "growy") // Grow vertically. Same as "growy 100"
panel.add(comp, "grow") // Grow both. Same as "grow 100 100"
panel.add(comp, "shrink 0") // Will not shrink.

```