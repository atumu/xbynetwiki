title: exp4j数学计算库 

Exp4j是一个简单易用的开源Java数学表达式计算工具，由德国Java开源爱好者Frank发起并持续进行维护，旨在提供对数学表达式的计算功能。
项目地址：http://www.objecthunter.net/exp4j/
#  安装 
```

<dependency>
    <groupId>net.objecthunter</groupId>
    <artifactId>exp4j</artifactId>
    <version>0.4.5</version>
</dependency>
  </code>
#  使用 
<code java Exp4jDemo.java>
import de.congrace.exp4j.Calculable; 
import de.congrace.exp4j.ExpressionBuilder; 
import de.congrace.exp4j.UnknownFunctionException; 
import de.congrace.exp4j.UnparsableExpressionException; 
 
/** 
 * Exp4j Demo 
 * @author William Xu 
 */ 
public class Exp4jDemo { 
    // 包含变量的数学表达式 
    private final String FUNCTION = "x/y + (x+y)*z"; 
    public Exp4jDemo() { 
    } 
    public void testFunction() { 
 
        // 构建表达式，并声明变量定义 
        ExpressionBuilder builder = new ExpressionBuilder(FUNCTION) 
                .withVariableNames("x", "y", "z"); 
         
        // 以下两种方式也可以声明变量，并直接给变量进行赋值 
        /*ExpressionBuilder.withVariable(String var,double value) 
        ExpressionBuilder.withVariables(Map<String,Double> variables)*/ 
         
        try { 
            // 生成计算对象 
            Calculable calc = builder.build(); 
            // 设置变量的值 
            calc.setVariable("x", 5); 
            calc.setVariable("y", 3); 
            calc.setVariable("z", 4); 
            // 计算结果 
            System.out.println(calc.calculate()); 
 
        } catch (UnknownFunctionException e) { 
            e.printStackTrace(); 
        } catch (UnparsableExpressionException e) { 
            e.printStackTrace(); 
        } 
    } 
    public static void main(String[] args) { 
        Exp4jDemo exp4jDemo = new Exp4jDemo(); 
        exp4jDemo.testFunction(); 
    } 
} 

```
参考：http://williamx.blog.51cto.com/3629295/1065823