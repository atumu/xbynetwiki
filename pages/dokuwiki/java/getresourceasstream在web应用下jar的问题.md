title: getresourceasstream在web应用下jar的问题 

#  getResourceAsStream在web应用涉及jar的问题 
webapp下WEB-INF/lib/xx.jar中的某个类通过class.getResourceAsStream("/"+Test.class.getName.replace('.','/')+".class");不仅可以获取本jar内的类，也可以获取WEB-INF/classes/下的类.也可以获取到其他WEB/INF/lib/vvv.jar包中的某个类。
WEB-INF/class/下的某个类通过class.getResourceAsStream("/"+Test.class.getName.replace('.','/')+".class");也可以完成上面的工作。