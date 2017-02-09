title: java获取某个包下的类得到数据库修改语句 

#  java获取某个包下的类得到数据库修改语句 
```

public static void main(String[] args) {
		File f=new File("src/main/java/com/css/sword/cms/entity/");
		String[] fs=f.list(new FilenameFilter() {
			
			@Override
			public boolean accept(File dir, String name) {
				// TODO Auto-generated method stub
				if(name.contains("Rel")|| !name.contains("Cms")){
					return false;
				}
				return true;
			}
		});
//		System.out.println(fs[0]);
		for(String name:fs){
			name=name.split("\\.")[0];
			String fname="";
			for(int i=0;i<name.length();i++){
				char ch=name.charAt(i);
				if(Character.isUpperCase(ch)){
					if(i==0){
						fname=String.valueOf(ch).toLowerCase();
					}else{
						fname+="_"+String.valueOf(ch).toLowerCase();
					}
				}else{
					fname+=String.valueOf(ch);
				}
			}
			System.out.println("/*"+fname+"*/");
			System.out.println("ALTER table "+ fname+" add column create_time timestamp;");
			System.out.println("ALTER table "+ fname+" add column create_user_id VARCHAR(50);");
			System.out.println("ALTER table "+ fname+" add column last_modify_time timestamp;");
			System.out.println("ALTER table "+ fname+" add column last_modify_user_id VARCHAR(50);");
		}
		
	}

```