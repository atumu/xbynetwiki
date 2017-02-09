title: 使用apache_poi读取excel文件 

#  使用Apache POI读取Excel文件 
 Apache的POI组件是Java操作Microsoft Office办公套件的强大API，其中**对Word，Excel和PowperPoint都有支持**，当然使用较多的还是Excel，因为Word和PowerPoint用程序动态操作的应用较少。那么本文就结合POI来介绍一下操作Excel的方法。 

**Office 2007的文件结构完全不同于2003，所以对于两个版本的Office组件，POI有不同的处理API，分开使用即可。**首先来说几个Excel的基本概念。对于一个Excel文件，这称为一个工作簿（Workbook），打开Excel之后，在下方会有sheet1/2/3这样的选项卡，点击可以切换到不同的sheet中，这个sheet称作工作表。每个工作表就是我们编辑的区域，这是一张二维表，阿拉伯数字控制行数，从1开始，而程序中还是0，类似数组和集合。字母控制列数，从A开始，Z以后是两个字母控制。对于每一行，我们称为Row，列就是Column，行列可以确定唯一的一个元素，那么就是单元格，称为Cell。 

POI组件可以方便的操纵这些元素，但初次接触POI可能会有畏惧心理，因为要对每个单元格进行设置，那么不管是用数组还是集合，从工作簿，工作表，行下来的代码量都不会小，这是不能避免的，但是按照这个处理顺序走，就一定可以得到结果。 
有了这些基础的概念之后，我们就可以操作Excel了。先来看一下所需的依赖，因为涉及到2007，就要额外加一些依赖。 
![](/data/dokuwiki/javaweb/pasted/20151020-102926.png)
POI的API 与实际使用中的Excel很类似，可以说是POI把Excel中的workbook、sheet、cell等对象化了，在实际使用中极易理解。但由于Apache POI在存在已有不短时间，至少在excel2007之前就已经出现，造成同样一套Api并不能同时读取(写入)xls和xlsx两种类型的Excel文件。但poi对excel有一个很好的抽象(ss包下的Workbook、Sheet、Cell等类)，可以一定程度上忽略xls/xlsx的处理细节，针对其通用部分进行处理，但如果需要对各自有理强的支持，还是建议使用相应的API。
 Apache POI 各个包功能描述：
        HSSF － 提供读写Microsoft Excel XLS格式档案的功能。 
        XSSF － 提供读写Microsoft Excel OOXML XLSX格式档案的功能。 
        HWPF － 提供读写Microsoft Word DOC格式档案的功能。 
        HSLF － 提供读写Microsoft PowerPoint格式档案的功能。 
        HDGF － 提供读Microsoft Visio格式档案的功能。 
        HPBF － 提供读Microsoft Publisher格式档案的功能。 
        HSMF － 提供读Microsoft Outlook格式档案的功能。
        
关于Apache POI一些重要的地方:
1)Apache POI包含适合Excel97-2007(.xls文件)的HSSF实现.
2)Apache POI XSSF实现用来处理Excel2007文件(.xlsx).
3)Apache POI HSSF和XSSF提供了读/写/修改Excel表格的机制.
4)Apache POI提供了XSSF的一个扩展SXSSF用来处理非常大的Excel工作单元.SXSSF API需要更少的内存,因此当处理非常大的电子表格同时堆内存又有限时,很合适使用.
5)有两种模式可供选择--事件模式和用户模式.事件模式要求更少的内存,因为用tokens来读取Excel并处理.用户模式更加面向对象并且容易使用,因此在我们的示例中使用用户   模式.
6)Apache POI为额外的Excel特性提供了强大支持,例如处理公式,创建单元格样式--颜色,边框,字体,头部,脚部,数据验证,图像,超链接等. 
Maven依赖：
```

<dependency>  
    <groupId>org.apache.poi</groupId>  
    <artifactId>poi</artifactId>  
    <version>3.10-FINAL</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.poi</groupId>  
    <artifactId>poi-ooxml</artifactId>  
    <version>3.10-FINAL</version>  
</dependency>

```
建议参考文章：http://sarin.iteye.com/blog/845035

##  读取Excel文件 
下面从读取Excel开始，首先建立一个Excel 2003以下版本的xls文件。设定几列来看。来存储学生信息的Excel表如下： 
![](/data/dokuwiki/javaweb/pasted/20151020-102954.png)
这里的姓名，性别和班级是文本值，而年龄和成绩是数字值，这在设计对象和处理时要注意区分。那么可以如下设计这个对象
```

public class Student {  
    private String name;  
    private String gender;  
    private int age;  
    private String sclass;  
    private int score;  
    public Student() {  
        super();  
    }  
    public Student(String name, String gender, int age, String sclass, int score) {  
        super();  
        this.name = name;  
        this.gender = gender;  
        this.age = age;  
        this.sclass = sclass;  
        this.score = score;  
    }  
//省略了getter和setter方法  
    @Override  
    public String toString() {  
        return "Student [age=" + age + ", gender=" + gender + ", name=" + name  
                + ", sclass=" + sclass + ", score=" + score + "]";  
    }  
}  


```
** 提供一个有参数的构造方法，用于生成对象写入Excel文档。**这个对象就能刻画Excel文件中的数据了，下面就是写程序将Excel文件加载并处理，然后将内容读出，**读取顺序是工作簿->工作表->行->单元格**。这样一分析就很简单了。我们定义两个Excel文件，内容相同，只是版本不同，**分2003和2007来处理。** 
创建工作簿时可以接收一个输入流对象，那么输入流对象可以从文件对象来生成，这样就可以继续进行了。**取出工作表，取出行，遍历单元格，数据就拿到了**。代码如下：
```

/** 
 * POI读取Excel示例，分2003和2007 
 *  
 * @author Nanlei 
 *  
 */  
public class ReadExcel {  
    private static String xls2003 = "C:\\student.xls";  
    private static String xlsx2007 = "C:\\student.xlsx";  
    /** 
     * 读取Excel2003的示例方法 
     *  
     * @param filePath 
     * @return 
     */  
private static List<Student> readFromXLS2003(String filePath) {  
        File excelFile = null;// Excel文件对象  
        InputStream is = null;// 输入流对象  
        String cellStr = null;// 单元格，最终按字符串处理  
        List<Student> studentList = new ArrayList<Student>();// 返回封装数据的List  
        Student student = null;// 每一个学生信息对象  
try {  
            excelFile = new File(filePath);  
            is = new FileInputStream(excelFile);// 获取文件输入流  
            HSSFWorkbook workbook2003 = new HSSFWorkbook(is);// 创建Excel2003文件对象  
            HSSFSheet sheet = workbook2003.getSheetAt(0);// 取出第一个工作表，索引是0  
            // 开始循环遍历行，表头不处理，从1开始  
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {  
                student = new Student();// 实例化Student对象  
                HSSFRow row = sheet.getRow(i);// 获取行对象  
                if (row == null) {// 如果为空，不处理  
                    continue;  
                }  
// 循环遍历单元格  
                for (int j = 0; j < row.getLastCellNum(); j++) {  
                    HSSFCell cell = row.getCell(j);// 获取单元格对象  
                    if (cell == null) {// 单元格为空设置cellStr为空串  
                        cellStr = "";  
                    } else if (cell.getCellType() == HSSFCell.CELL_TYPE_BOOLEAN) {// 对布尔值的处理  
                        cellStr = String.valueOf(cell.getBooleanCellValue());  
                    } else if (cell.getCellType() == HSSFCell.CELL_TYPE_NUMERIC) {// 对数字值的处理  
                        cellStr = cell.getNumericCellValue() + "";  
                    } else {// 其余按照字符串处理  
                        cellStr = cell.getStringCellValue();  
                    }  
// 下面按照数据出现位置封装到bean中  
                    if (j == 0) {  
                        student.setName(cellStr);  
                    } else if (j == 1) {  
                        student.setGender(cellStr);  
                    } else if (j == 2) {  
                        student.setAge(new Double(cellStr).intValue());  
                    } else if (j == 3) {  
                        student.setSclass(cellStr);  
                    } else {  
                        student.setScore(new Double(cellStr).intValue());  
                    }  
                }  
                studentList.add(student);// 数据装入List  
            }  
} catch (IOException e) {  
            e.printStackTrace();  
        } finally {// 关闭文件流  
            if (is != null) {  
                try {  
                    is.close();  
                } catch (IOException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
        return studentList;  
    }  
/** 
     * 主函数 
     *  
     * @param args 
     */  
    public static void main(String[] args) {  
        long start = System.currentTimeMillis();  
        List<Student> list = readFromXLS2003(xls2003);  
        for (Student student : list) {  
            System.out.println(student);  
        }  
        long end = System.currentTimeMillis();  
        System.out.println((end - start) + " ms done!");  
    }  
}  

```
做几点说明，如果不处理表头，那么就从准备处理的行开始，而整个sheet对行的索引是从0开始的，而Excel中是1，这点和数组/集合类似。对于单元格中的数字，默认按double类型处理，所以只能字符串转double，再取出int值。最后执行主函数，得到如下内容： 

这样就拿到对象的List了，之后要持久到数据库或者直接做业务逻辑就随心所欲了。下面来看2007的处理，处理流程和2003是类似的，区别就是使用的对象，2003中对象是HSSF*格式的，而2007是XSSF*格式的。方法如下：
```

public static List<Student> readFromXLSX2007(String filePath) {  
        File excelFile = null;// Excel文件对象  
        InputStream is = null;// 输入流对象  
        String cellStr = null;// 单元格，最终按字符串处理  
        List<Student> studentList = new ArrayList<Student>();// 返回封装数据的List  
        Student student = null;// 每一个学生信息对象  
        try {  
            excelFile = new File(filePath);  
            is = new FileInputStream(excelFile);// 获取文件输入流  
            XSSFWorkbook workbook2007 = new XSSFWorkbook(is);// 创建Excel2003文件对象  
            XSSFSheet sheet = workbook2007.getSheetAt(0);// 取出第一个工作表，索引是0  
            // 开始循环遍历行，表头不处理，从1开始  
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {  
                student = new Student();// 实例化Student对象  
                XSSFRow row = sheet.getRow(i);// 获取行对象  
                if (row == null) {// 如果为空，不处理  
                    continue;  
                }  
                // 循环遍历单元格  
                for (int j = 0; j < row.getLastCellNum(); j++) {  
                    XSSFCell cell = row.getCell(j);// 获取单元格对象  
                    if (cell == null) {// 单元格为空设置cellStr为空串  
                        cellStr = "";  
                    } else if (cell.getCellType() == HSSFCell.CELL_TYPE_BOOLEAN) {// 对布尔值的处理  
                        cellStr = String.valueOf(cell.getBooleanCellValue());  
                    } else if (cell.getCellType() == HSSFCell.CELL_TYPE_NUMERIC) {// 对数字值的处理  
                        cellStr = cell.getNumericCellValue() + "";  
                    } else {// 其余按照字符串处理  
                        cellStr = cell.getStringCellValue();  
                    }  
                    // 下面按照数据出现位置封装到bean中  
                    if (j == 0) {  
                        student.setName(cellStr);  
                    } else if (j == 1) {  
                        student.setGender(cellStr);  
                    } else if (j == 2) {  
                        student.setAge(new Double(cellStr).intValue());  
                    } else if (j == 3) {  
                        student.setSclass(cellStr);  
                    } else {  
                        student.setScore(new Double(cellStr).intValue());  
                    }  
                }  
                studentList.add(student);// 数据装入List  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        } finally {// 关闭文件流  
            if (is != null) {  
                try {  
                    is.close();  
                } catch (IOException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
        return studentList;  
    }  

```
##  写入Excel 
 下面来做简单的文件写入，也就是准备输入写入Excel文件，为了演示，直接创建对象，而实际应用中数据可以是来自数据库的。写入文件就是文件解析的逆过程。**但POI的组件不是从单元格开始创建文件的，还是从工作簿开始创建，进而创建工作表，行和单元格，最终将整个工作簿写入文件，完成操作。我们来看具体写法**

```

/** 
 * 生成Excel示例，2003和2007 
 *  
 * @author Nanlei 
 *  
 */  
public class GenerateExcel {  
    private static String xls2003 = "C:\\student.xls";  
    private static String xlsx2007 = "C:\\student.xlsx";  
    private static List<Student> studentList = null;  
    private static Student[] students = new Student[4];  
    /** 
     * 静态块初始化数据 
     */  
    static {  
        studentList = new ArrayList<Student>();  
        students[0] = new Student("张三", "男", 23, "一班", 94);  
        students[1] = new Student("李四", "女", 20, "一班", 92);  
        students[2] = new Student("王五", "男", 21, "一班", 87);  
        students[3] = new Student("赵六", "女", 22, "一班", 83);  
        studentList.addAll(Arrays.asList(students));  
    }  
    /** 
     * 创建2003文件的方法 
     *  
     * @param filePath 
     */  
    public static void generateExcel2003(String filePath) {  
        // 先创建工作簿对象  
        HSSFWorkbook workbook2003 = new HSSFWorkbook();  
        // 创建工作表对象并命名  
        HSSFSheet sheet = workbook2003.createSheet("学生信息统计表");  
        // 遍历集合对象创建行和单元格  
        for (int i = 0; i < studentList.size(); i++) {  
            // 取出Student对象  
            Student student = studentList.get(i);  
            // 创建行  
            HSSFRow row = sheet.createRow(i);  
            // 开始创建单元格并赋值  
            HSSFCell nameCell = row.createCell(0);  
            nameCell.setCellValue(student.getName());  
            HSSFCell genderCell = row.createCell(1);  
            genderCell.setCellValue(student.getGender());  
            HSSFCell ageCell = row.createCell(2);  
            ageCell.setCellValue(student.getAge());  
            HSSFCell sclassCell = row.createCell(3);  
            sclassCell.setCellValue(student.getSclass());  
            HSSFCell scoreCell = row.createCell(4);  
            scoreCell.setCellValue(student.getScore());  
        }  
        // 生成文件  
        File file = new File(filePath);  
        FileOutputStream fos = null;  
        try {  
            fos = new FileOutputStream(file);  
            workbook2003.write(fos);  
        } catch (Exception e) {  
            e.printStackTrace();  
        } finally {  
            if (fos != null) {  
                try {  
                    fos.close();  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
    }  
    /** 
     * 主函数 
     *  
     * @param args 
     */  
    public static void main(String[] args) {  
        long start = System.currentTimeMillis();  
        generateExcel2003(xls2003);  
        long end = System.currentTimeMillis();  
        System.out.println((end - start) + " ms done!");  
    }  
}  

```
这样就生成了2003版Excel文件，只是最简单的操作，并没有涉及到单元格格式等操作，而2007的方法就是改改对象的名称，很简单，这里不再贴出了。

进阶请看：http://sarin.iteye.com/blog/846679 http://sarin.iteye.com/blog/853418 http://sarin.iteye.com/blog/859163
下面给出一个简单的实现Demo：
```

import static net.yeah.likun_zhang.util.Debug.printf;
 
import java.io.FileInputStream;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;
 
import net.yeah.likun_zhang.util.Debug;
 
import org.apache.commons.lang3.StringUtils;
import org.apache.poi.hssf.usermodel.HSSFDateUtil;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
 
/**
 * 读取Excel 97~2003 xls格式 /2007~ xlsx格式
 * @author      ZhangLiKun
 * @mail        likun_zhang@yeah.net
 * @date        2013-5-11
 */
public class ExcelReader {
 
    /**
     * 创建工作簿对象
     * @param filePath
     * @return
     * @throws IOException
     * @date    2013-5-11
     */
    public static final Workbook createWb(String filePath) throws IOException {
        if(StringUtils.isBlank(filePath)) {
            throw new IllegalArgumentException("参数错误!!!") ;
        }
        if(filePath.trim().toLowerCase().endsWith("xls")) {
            return new HSSFWorkbook(new FileInputStream(filePath)) ;
        } else if(filePath.trim().toLowerCase().endsWith("xlsx")) {
            return new XSSFWorkbook(new FileInputStream(filePath)) ;
        } else {
            throw new IllegalArgumentException("不支持除：xls/xlsx以外的文件格式!!!") ;
        }
    }
     
    public static final Sheet getSheet(Workbook wb ,String sheetName) {
        return wb.getSheet(sheetName) ;
    }
     
    public static final Sheet getSheet(Workbook wb ,int index) {
        return wb.getSheetAt(index) ;
    }
     
    public static final List<Object[]> listFromSheet(Sheet sheet) {
         
        int rowTotal = sheet.getPhysicalNumberOfRows() ;
        Debug.printf("{}共有{}行记录！" ,sheet.getSheetName() ,rowTotal) ;
         
        List<Object[]> list = new ArrayList<Object[]>() ;
        for(int r = sheet.getFirstRowNum() ; r <= sheet.getLastRowNum() ; r ++) {
            Row row = sheet.getRow(r) ;
            if(row == null)continue ;
            // 不能用row.getPhysicalNumberOfCells()，可能会有空cell导致索引溢出
            // 使用row.getLastCellNum()至少可以保证索引不溢出，但会有很多Null值，如果使用集合的话，就不说了
            Object[] cells = new Object[row.getLastCellNum()] ;
            for(int c = row.getFirstCellNum() ; c <= row.getLastCellNum() ; c++) {
                Cell cell = row.getCell(c) ;
                if(cell == null)continue ;
                cells[c] = getValueFromCell(cell) ;
            }
            list.add(cells) ;
        }
         
        return list ;
    }
     
     
    /**
     * 获取单元格内文本信息
     * @param cell
     * @return
     * @date    2013-5-8
     */
    public static final String getValueFromCell(Cell cell) {
        if(cell == null) {
            printf("Cell is null !!!") ;
            return null ;
        }
        String value = null ;
        switch(cell.getCellType()) {
            case Cell.CELL_TYPE_NUMERIC :   // 数字
                if(HSSFDateUtil.isCellDateFormatted(cell)) {        // 如果是日期类型
                    value = new SimpleDateFormat(DatePattern.LOCALE_ZH_DATE.getValue()).format(cell.getDateCellValue()) ;
                } else  value = String.valueOf(cell.getNumericCellValue()) ;
                break ;
            case Cell.CELL_TYPE_STRING:     // 字符串
                value = cell.getStringCellValue() ;
                break ;
            case Cell.CELL_TYPE_FORMULA:    // 公式
                // 用数字方式获取公式结果，根据值判断是否为日期类型
                double numericValue = cell.getNumericCellValue() ;
                if(HSSFDateUtil.isValidExcelDate(numericValue)) {   // 如果是日期类型
                    value = new SimpleDateFormat(DatePattern.LOCALE_ZH_DATE.getValue()).format(cell.getDateCellValue()) ;
                } else  value = String.valueOf(numericValue) ;
                break ;
            case Cell.CELL_TYPE_BLANK:              // 空白
                value = ExcelConstants.EMPTY_CELL_VALUE ;
                break ;
            case Cell.CELL_TYPE_BOOLEAN:            // Boolean
                value = String.valueOf(cell.getBooleanCellValue()) ;
                break ;
            case Cell.CELL_TYPE_ERROR:              // Error，返回错误码
                value = String.valueOf(cell.getErrorCellValue()) ;
                break ;
            default:value = StringUtils.EMPTY ;break ;
        }
        // 使用[]记录坐标
        return value + "["+cell.getRowIndex()+","+cell.getColumnIndex()+"]" ;
    }  
     
}
execl操作工具类示列：
<code java>
import org.apache.poi.ss.usermodel.Cell;  
import org.apache.poi.ss.usermodel.CellStyle;  
import org.apache.poi.ss.usermodel.Row;  
import org.apache.poi.ss.usermodel.Sheet;  
import org.apache.poi.ss.usermodel.Workbook;  
import org.apache.poi.ss.usermodel.WorkbookFactory;  
  
/** 
 * Excel工具类 
 *  
 * <pre> 
 * 基于Apache的POI类库 
 * </pre> 
 *  
 * @author 陈峰 
 */  
public class POIExcelMakerUtil {  
  
    private File excelFile;  
  
    private InputStream fileInStream;  
  
    private Workbook workBook;  
  
    public POIExcelMakerUtil(File file) throws Exception {  
        this.excelFile = file;  
        this.fileInStream = new FileInputStream(this.excelFile);  
        this.workBook = WorkbookFactory.create(this.fileInStream);  
    }  
  
    /** 
     * 写入一组值 
     *  
     * @param sheetNum 
     *            写入的sheet的编号 
     * @param fillRow 
     *            是写入行还是写入列 
     * @param startRowNum 
     *            开始行号 
     * @param startColumnNum 
     *            开始列号 
     * @param contents 
     *            写入的内容数组 
     * @throws Exception 
     */  
    public void writeArrayToExcel(int sheetNum, boolean fillRow,  
            int startRowNum, int startColumnNum, Object[] contents)  
            throws Exception {  
        Sheet sheet = this.workBook.getSheetAt(sheetNum);  
        writeArrayToExcel(sheet, fillRow, startRowNum, startColumnNum, contents);  
    }  
  
    /** 
     * 写入一组值 
     *  
     * @param sheetNum 
     *            写入的sheet的名称 
     * @param fillRow 
     *            是写入行还是写入列 
     * @param startRowNum 
     *            开始行号 
     * @param startColumnNum 
     *            开始列号 
     * @param contents 
     *            写入的内容数组 
     * @throws Exception 
     */  
    public void writeArrayToExcel(String sheetName, boolean fillRow,  
            int startRowNum, int startColumnNum, Object[] contents)  
            throws Exception {  
        Sheet sheet = this.workBook.getSheet(sheetName);  
        writeArrayToExcel(sheet, fillRow, startRowNum, startColumnNum, contents);  
    }  
  
    private void writeArrayToExcel(Sheet sheet, boolean fillRow,  
            int startRowNum, int startColumnNum, Object[] contents)  
            throws Exception {  
        for (int i = 0, length = contents.length; i < length; i++) {  
            int rowNum;  
            int columnNum;  
            // 以行为单位写入  
            if (fillRow) {  
                rowNum = startRowNum;  
                columnNum = startColumnNum + i;  
            }  
            // 　以列为单位写入  
            else {  
                rowNum = startRowNum + i;  
                columnNum = startColumnNum;  
            }  
            this.writeToCell(sheet, rowNum, columnNum,  
                    convertString(contents[i]));  
        }  
    }  
  
    /** 
     * 向一个单元格写入值 
     *  
     * @param sheetNum 
     *            sheet的编号 
     * @param rowNum 
     *            行号 
     * @param columnNum 
     *            列号 
     * @param value 
     *            写入的值 
     * @throws Exception 
     */  
    public void writeToExcel(int sheetNum, int rowNum, int columnNum,  
            Object value) throws Exception {  
        Sheet sheet = this.workBook.getSheetAt(sheetNum);  
        this.writeToCell(sheet, rowNum, columnNum, value);  
    }  
  
    /** 
     * 向一个单元格写入值 
     *  
     * @param sheetName 
     *            sheet的名称 
     * @param columnRowNum 
     *            单元格的位置 
     * @param value 
     *            写入的值 
     * @throws Exception 
     */  
    public void writeToExcel(String sheetName, int rowNum, int columnNum,  
            Object value) throws Exception {  
        Sheet sheet = this.workBook.getSheet(sheetName);  
        this.writeToCell(sheet, rowNum, columnNum, value);  
    }  
  
    /** 
     * 向一个单元格写入值 
     *  
     * @param sheetNum 
     *            sheet的编号 
     * @param columnRowNum 
     *            单元格的位置 
     * @param value 
     *            写入的值 
     * @throws Exception 
     */  
    public void writeToExcel(int sheetNum, String columnRowNum, Object value)  
            throws Exception {  
        Sheet sheet = this.workBook.getSheetAt(sheetNum);  
        this.writeToCell(sheet, columnRowNum, value);  
    }  
  
    /** 
     * 向一个单元格写入值 
     *  
     * @param sheetNum 
     *            sheet的名称 
     * @param columnRowNum 
     *            单元格的位置 
     * @param value 
     *            写入的值 
     * @throws Exception 
     */  
    public void writeToExcel(String sheetName, String columnRowNum, Object value)  
            throws Exception {  
        Sheet sheet = this.workBook.getSheet(sheetName);  
        this.writeToCell(sheet, columnRowNum, value);  
    }  
  
    private void writeToCell(Sheet sheet, String columnRowNum, Object value)  
            throws Exception {  
        int[] rowNumColumnNum = convertToRowNumColumnNum(columnRowNum);  
        int rowNum = rowNumColumnNum[0];  
        int columnNum = rowNumColumnNum[1];  
        this.writeToCell(sheet, rowNum, columnNum, value);  
    }  
  
    /** 
     * 将单元格的行列位置转换为行号和列号 
     *  
     * @param columnRowNum 
     *            行列位置 
     * @return 长度为2的数组，第1位为行号，第2位为列号 
     */  
    private static int[] convertToRowNumColumnNum(String columnRowNum) {  
        columnRowNum = columnRowNum.toUpperCase();  
        char[] chars = columnRowNum.toCharArray();  
        int rowNum = 0;  
        int columnNum = 0;  
        for (char c : chars) {  
            if ((c >= 'A' && c <= 'Z')) {  
                columnNum = columnNum * 26 + ((int) c - 64);  
            } else {  
                rowNum = rowNum * 10 + new Integer(c + "");  
            }  
        }  
        return new int[] { rowNum - 1, columnNum - 1 };  
    }  
  
    private void writeToCell(Sheet sheet, int rowNum, int columnNum,  
            Object value) throws Exception {  
        Row row = sheet.getRow(rowNum);  
        Cell cell = row.getCell(columnNum);  
        if (cell == null) {  
            cell = row.createCell(columnNum);  
        }  
        cell.setCellValue(convertString(value));  
    }  
  
    /** 
     * 读取一个单元格的值 
     *  
     * @param sheetName 
     *            sheet的名称 
     * @param columnRowNum 
     *            单元格的位置 
     * @return 
     * @throws Exception 
     */  
    public Object readCellValue(String sheetName, String columnRowNum)  
            throws Exception {  
        Sheet sheet = this.workBook.getSheet(sheetName);  
        int[] rowNumColumnNum = convertToRowNumColumnNum(columnRowNum);  
        int rowNum = rowNumColumnNum[0];  
        int columnNum = rowNumColumnNum[1];  
        Row row = sheet.getRow(rowNum);  
        if (row != null) {  
            Cell cell = row.getCell(columnNum);  
            if (cell != null) {  
                return getCellValue(cell);  
            }  
        }  
        return null;  
    }  
  
    /** 
     * 获取单元格中的值 
     *  
     * @param cell 单元格 
     * @return 
     */  
    private static Object getCellValue(Cell cell) {  
        int type = cell.getCellType();  
        switch (type) {  
        case Cell.CELL_TYPE_STRING:  
            return (Object) cell.getStringCellValue();  
        case Cell.CELL_TYPE_NUMERIC:  
            Double value = cell.getNumericCellValue();  
            return (Object) (value.intValue());  
        case Cell.CELL_TYPE_BOOLEAN:  
            return (Object) cell.getBooleanCellValue();  
        case Cell.CELL_TYPE_FORMULA:  
            return (Object) cell.getArrayFormulaRange().formatAsString();  
        case Cell.CELL_TYPE_BLANK:  
            return (Object) "";  
        default:  
            return null;  
        }  
    }  
  
    /** 
     * 插入一行并参照与上一行相同的格式 
     *  
     * @param sheetNum 
     *            sheet的编号 
     * @param rowNum 
     *            插入行的位置 
     * @throws Exception 
     */  
    public void insertRowWithFormat(int sheetNum, int rowNum) throws Exception {  
        Sheet sheet = this.workBook.getSheetAt(sheetNum);  
        insertRowWithFormat(sheet, rowNum);  
    }  
  
    /** 
     * 插入一行并参照与上一行相同的格式 
     *  
     * @param sheetName 
     *            sheet的名称 
     * @param rowNum 
     *            插入行的位置 
     * @throws Exception 
     */  
    public void insertRowWithFormat(String sheetName, int rowNum)  
            throws Exception {  
        Sheet sheet = this.workBook.getSheet(sheetName);  
        insertRowWithFormat(sheet, rowNum);  
    }  
  
    private void insertRowWithFormat(Sheet sheet, int rowNum) throws Exception {  
        sheet.shiftRows(rowNum, rowNum + 1, 1);  
        Row newRow = sheet.createRow(rowNum);  
        Row oldRow = sheet.getRow(rowNum - 1);  
        for (int i = oldRow.getFirstCellNum(); i < oldRow.getLastCellNum(); i++) {  
            Cell oldCell = oldRow.getCell(i);  
            if (oldCell != null) {  
                CellStyle cellStyle = oldCell.getCellStyle();  
                newRow.createCell(i).setCellStyle(cellStyle);  
            }  
        }  
    }  
  
    /** 
     * 重命名一个sheet 
     *  
     * @param sheetNum 
     *            sheet的编号 
     * @param newName 
     *            新的名称 
     */  
    public void renameSheet(int sheetNum, String newName) {  
        this.workBook.setSheetName(sheetNum, newName);  
    }  
  
    /** 
     * 重命名一个sheet 
     *  
     * @param oldName 
     *            旧的名称 
     * @param newName 
     *            新的名称 
     */  
    public void renameSheet(String oldName, String newName) {  
        int sheetNum = this.workBook.getSheetIndex(oldName);  
        this.renameSheet(sheetNum, newName);  
    }  
  
    /** 
     * 删除一个sheet 
     *  
     * @param sheetName 
     *            sheet的名称 
     */  
    public void removeSheet(String sheetName) {  
        this.workBook.removeSheetAt(this.workBook.getSheetIndex(sheetName));  
    }  
  
    /** 
     * 写入Excel文件并关闭 
     */  
    public void writeAndClose() {  
        if (this.workBook != null) {  
            try {  
                FileOutputStream fileOutStream = new FileOutputStream(  
                        this.excelFile);  
                this.workBook.write(fileOutStream);  
                if (fileOutStream != null) {  
                    fileOutStream.close();  
                }  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
        if (this.fileInStream != null) {  
            try {  
                this.fileInStream.close();  
            } catch (Exception e) {  
            }  
        }  
    }  
  
    private static String convertString(Object value) {  
        if (value == null) {  
            return "";  
        } else {  
            return value.toString();  
        }  
    }  
  
}  

```
</code>
参考：http://www.open-open.com/lib/view/open1368451501266.html
http://sarin.iteye.com/blog/845035
http://www.journaldev.com/2562/java-readwrite-excel-file-using-apache-poi-api
http://yunzhu.iteye.com/blog/1836696