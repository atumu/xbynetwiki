title: sword单元测试 

#  Sword单元测试 
```

public class TestApp {
	@Test
	public void test1() throws SwordBaseCheckedException{
		SwordPlatformManager.startPlatform();
		SwordServiceUtils.callService("/cms/ProcessInformationService/QueryProcessList2");//注意这个service没有参数
		
	
	}
}


```
```

/**junit测试*/
	@Service("QueryProcessList2")
	public ISwordResponse QueryProcessList2() throws SwordBaseCheckedException{
		logger.debug("---cms/ProcessInformationService中的QueryProcessList方法开始---");
	
		IPersistenceService dao = SwordPersistenceUtils.getPersistenceService();
		List<Map<String,Object>> resultAll=dao.findAllBySql("select file_id as myId from t_file",new Object[]{});
		Map<String,Object> result1=dao.findOneBySql("select COUNT(*) as dataCount from t_file",new Object[]{});
		System.out.println(result1.toString());
		return null;
	}

```