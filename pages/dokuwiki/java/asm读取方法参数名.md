title: asm读取方法参数名 

#  运行时asm读取方法参数名 
##  方式一、存在问题:（仅供查看，存在问题） 
下面的代码用eclipse编译的class文件可以被正确解析出来，但是使用javac编译的class文件并不能正确解析，这是为什么呢？
**javac生成的class文件用这个工具得到的` 变量顺序有问题 `，是按照变量的使用顺序排序的。**那如何到得一个变量定义的顺序呢。
其实在**ASM遍历方式**中的MethodVisitor的visitLocalVariable有一个参数int ` index.它记录了方法中本地变量(localVariable)原始的顺序。通过对它排序后能够得到正确的顺序 `如下
```

    public void visitLocalVariable(String name, String desc, String signature,
            Label start, Label end, int index)

``` 
` ASM树形遍历方式中(在接下来的方式二中会介绍)LocalVariableNode中有个index属性，它记录了方法中本地变量(localVariable)原始的顺序。通过对它排序后能够得到正确的顺序。 `

方式一：以下代码存在问题，请勿使用。仅用于演示。(其实也可以修改本方式，通过在visitLocalVariable实现中附加记录index变量，然后进行重排序即可正确。)
```

import org.objectweb.asm.Label;
import org.objectweb.asm.MethodVisitor;
public class ArgNameMethodVisitor extends MethodVisitor {
    private Map<int,String> argumentNames;
    private int argLen; //变量个数
    public ReadMethodArgNameMethodVisitor(int api,List<String> argumentNames,int argLen) {
        super(api);
        this.argumentNames=argumentNames;
        this.argLen=argLen;
        
    }
  //asm遍历局部变量
    @Override
    public void visitLocalVariable(String name, String desc, String signature,
            Label start, Label end, int index) {
        if("this".equals(name)) { //如果是this变量，则掠过
            return;
        }
        if(argLen-- > 0) {
            argumentNames.add(name);
        }
    }
  
}

```
```

import org.objectweb.asm.ClassReader;
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;
import org.objectweb.asm.Type;
public class ArgNameClassVisitor extends ClassVisitor {
      
    private Map<String, List<String>> nameArgMap = new HashMap<String, List<String>>();
    public ArgNameClassVisitor() {
        super(Opcodes.ASM5);
    }
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc,
            String signature, String[] exceptions) {
        Type methodType = Type.getMethodType(desc);
        int len = methodType.getArgumentTypes().length;
        List<String> argumentNames = new ArrayList<String>();
        nameArgMap.put(name, argumentNames);
        MethodVisitor visitor = new ArgNameMethodVisitor(Opcodes.ASM5,argumentNames,len);
        return visitor;
    }
    public static void test(BeanUtil str1){
         
    }
    public Map<String, List<String>> getNameArgMap(){
          return nameArgMap;
    }
    public static void main(String[] args) throws IOException {
        ClassReader cr=new ClassReader(ArgNameClassVisitor.class.getName());
        ArgNameClassVisitor cl=new ArgNameClassVisitor();
        cr.accept(cl, 0);
        for(Entry<String, List<String>> entry:cl.getNameArgMap().entrySet()){
            System.out.println(entry.getKey());
            for(String str:entry.getValue()){
                System.out.println("value:"+str);
            }
        }
    }
}

```

##  方式二:推荐 
```

package net.xby1993.springmvc.controller;

import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.objectweb.asm.ClassReader;
import org.objectweb.asm.Type;
import org.objectweb.asm.tree.ClassNode;
import org.objectweb.asm.tree.LocalVariableNode;
import org.objectweb.asm.tree.MethodNode;

public class MethodParamNamesScanner {

	/**
	 * 获取方法参数名列表
	 * 
	 * @param clazz
	 * @param m
	 * @return
	 * @throws IOException
	 */
	public static List<String> getMethodParamNames(Class<?> clazz, Method m) throws IOException {
		try (InputStream in = clazz.getResourceAsStream("/" + clazz.getName().replace('.', '/') + ".class")) {
			return getMethodParamNames(in,m);
		}

	}
	public static List<String> getMethodParamNames(InputStream in, Method m) throws IOException {
		try (InputStream ins=in) {
			return getParamNames(ins,
					new EnclosingMetadata(m.getName(),Type.getMethodDescriptor(m), m.getParameterTypes().length));
		}

	}
	/**
	 * 获取构造器参数名列表
	 * 
	 * @param clazz
	 * @param constructor
	 * @return
	 */
	public static List<String> getConstructorParamNames(Class<?> clazz, Constructor<?> constructor) {
		try (InputStream in = clazz.getResourceAsStream("/" + clazz.getName().replace('.', '/') + ".class")) {
			return getConstructorParamNames(in, constructor);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
		return new ArrayList<String>();
	}
	public static List<String> getConstructorParamNames(InputStream ins, Constructor<?> constructor) {
		try (InputStream in = ins) {
			return getParamNames(in, new EnclosingMetadata(constructor.getName,Type.getConstructorDescriptor(constructor),
					constructor.getParameterTypes().length));
		} catch (IOException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		}
		return new ArrayList<String>();
	}
	/**
	 * 获取参数名列表辅助方法
	 * 
	 * @param in
	 * @param m
	 * @return
	 * @throws IOException
	 */
	private static List<String> getParamNames(InputStream in, EnclosingMetadata m) throws IOException {
		ClassReader cr = new ClassReader(in);
		ClassNode cn = new ClassNode();
		cr.accept(cn, ClassReader.EXPAND_FRAMES);// 建议EXPAND_FRAMES
		// ASM树接口形式访问
		List<MethodNode> methods = cn.methods;
		List<String> list = new ArrayList<String>();
		for (int i = 0; i < methods.size(); ++i) {
			List<LocalVariable> varNames = new ArrayList<LocalVariable>();
			MethodNode method = methods.get(i);
			// 验证方法签名
			if (method.desc.equals(m.desc)&&method.name.equals(m.name)) {
//				System.out.println("desc->"+method.desc+":"+m.desc);
				List<LocalVariableNode> local_variables = method.localVariables;
				for (int l = 0; l < local_variables.size(); l++) {
					String varName = local_variables.get(l).name;
					// index-记录了正确的方法本地变量索引。(方法本地变量顺序可能会被打乱。而index记录了原始的顺序)
					int index = local_variables.get(l).index;
					if (!"this".equals(varName)) // 非静态方法,第一个参数是this
						varNames.add(new LocalVariable(index, varName));
				}
				LocalVariable[] tmpArr = varNames.toArray(new LocalVariable[varNames.size()]);
				// 根据index来重排序，以确保正确的顺序
				Arrays.sort(tmpArr);
				for (int j = 0; j < m.size; j++) {
					list.add(tmpArr[j].name);
				}
				break;

			}

		}
		return list;
	}

	/**
	 * 方法本地变量索引和参数名封装
	 * @author xby Administrator
	 */
	static class LocalVariable implements Comparable<LocalVariable> {
		public int index;
		public String name;

		public LocalVariable(int index, String name) {
			this.index = index;
			this.name = name;
		}

		public int compareTo(LocalVariable o) {
			return this.index - o.index;
		}
	}

	/**
	 * 封装方法描述和参数个数
	 * 
	 * @author xby Administrator
	 */
	static class EnclosingMetadata {
          	//method name
          	public String name;
		// method description
		public String desc;
		// params size
		public int size;

		public EnclosingMetadata(String name,String desc, int size) {
                  	this.name=name;
			this.desc = desc;
			this.size = size;
		}
	}

	public static void main(String[] args) throws IOException {
		for (Method m : AdminController.class.getDeclaredMethods()) {
			List<String> list = getMethodParamNames(AdminController.class, m);
			System.out.println(m.getName() + ":");
			for (String str : list) {
				System.out.println(str);
			}
			System.out.println("------------------------");
		}
	}
}


```

##  Spring中实现 
org.springframework.core.DefaultParameterNameDiscoverer
##  可用的第三方库 
https://github.com/paul-hammant/paranamer

参考：http://www.oschina.net/question/2356075_241498
http://stackoverflow.com/questions/2729580/how-to-get-the-parameter-names-of-an-objects-constructors-reflection
http://asm.ow2.org/asm50/javadoc/user/index.html -asm5官方api
最关键的一篇参考文:http://www.xuebuyuan.com/496191.html 说明了参数名顺序问题与排序方法。