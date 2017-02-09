title: java_classloader自定义1 

#  Java自定义ClassLoader加载webapp外的jar包 
```

/**
 * ClassLoader to enable running webapp classes outside of webapp.  
 * You provide webappDir and jarsDir paths and the classloader will include 
 * webappDir/WEB-INF/classes, webappDir/WEB-INF/lib/*jar and jarsDir/*.jar.
 */
public class StandaloneWebappClassLoader extends URLClassLoader {
    public static String FS = File.separator;
    
    /** Use calling class's parent classloader */
    public StandaloneWebappClassLoader(String webappDir, String jarsDir) throws Exception {
        super(buildURLsArray(webappDir, jarsDir));
    }
    
    /** Use a specific parent classloader, or null for no parent */
    public StandaloneWebappClassLoader(String webappDir, String jarsDir, ClassLoader cl) throws Exception {
        super(buildURLsArray(webappDir, jarsDir), cl);
    }
    
    private static URL[] buildURLsArray(String webappDir, String jarsDir) throws Exception {
        // Create collection of URLs needed for classloader
        List urlList = new ArrayList();

        // Add WEB-INF/lib jars
        String libPath = webappDir + FS + "WEB-INF" + FS + "lib";
        addURLs(libPath, urlList);
        
        // Added WEB-INF/classes
        String classesPath = webappDir + FS + "WEB-INF" + FS + "classes" + FS;
        urlList.add(new URL("file://" + classesPath));
        
        // Add additional jars
        addURLs(jarsDir, urlList);
                
        return (URL[])urlList.toArray(new URL[urlList.size()]);  
    }
    
    private static void addURLs(String dirPath, List urlList) throws Exception {
        File libDir = new File(dirPath);
        String[] libJarNames = libDir.list(new FilenameFilter() {
            public boolean accept(File dir, String pathname) {
                if (pathname.endsWith(".jar")) {
                    return true;
                }
                return false;
            }
        });       
        for (int i=0; i<libJarNames.length; i++) {
            String url = "file://" + dirPath + FS + libJarNames[i];
            urlList.add(new URL(url));
        }
    }
}


```