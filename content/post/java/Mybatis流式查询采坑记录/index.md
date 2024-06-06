# Mybatis流式查询采坑记录
# 1. 问题复现
	 报表模块导出数据量大的报表导出出现OOM

堆大小 `-Xmx512m -Xms512m` ， 问题成功复现
![[Pasted image 20240606111248.png]]

# 2. 代码分析
	数据库查询方式已经是流式查询，且excel导出也是流式输出，不应该出现OOM
	
源代码 mapper.xml
``` xml
<select id="findClueTrafficList" resultMap="clueTrafficDTO" resultSetType="FORWARD_ONLY" fetchSize="500000">  
    SELECT  xxx from xxx
</select>
```

导出代码
``` java
public File export(String exportType, LoginUser loginUser, String jsonStr){  
    Cursor<? extends Serializable> cursor = mapper.findClueTrafficList(jsonStr);  
    Map<String, List<DictItemDetailDTO>> dictItemMap = dictConvertHelper.getDictItemMap(reportExportTask.getExcelHeadClass());  
    //导出  
    Iterator<? extends Serializable> iterator = cursor.iterator();  
    // 根据模板写入数据，如果目标文件不存在，则自动创建文件  
    File file = FileUtil.createTempFile(IdUtil.fastSimpleUUID(), ExcelTypeEnum.XLSX.getValue(), null, true);  
    ExcelWriter excelWriter = EasyExcelFactory.write(file, reportExportTask.getExcelHeadClass()).includeColumnFieldNames(getIncludeColumnFiledNames(jsonStr))  
            .build();  
  
    List<Serializable> list = new ArrayList<>(1000);  
    int sheetNo = 1;  
    WriteSheet writeSheet = EasyExcelFactory.writerSheet(sheetNo).build();  
    int i = 0;  
    int count = 0;  
    while (iterator.hasNext()) {  
        list.add(iterator.next());  
        //存在字典翻译注解,翻译数据字典  
        if (CollUtil.isNotEmpty(dictItemMap.keySet())){  
            dictConvertHelper.convertDict(list.get(i),dictItemMap);  
        }  
        i++;  
        count++;  
        //每100w行一个sheet  
        if (count >= 1000000) {  
            sheetNo++;  
            count=0;  
        }  
        //每1000行写入一次磁盘  
        if (i >= 1000) {  
            i = 0;  
            writeSheet = EasyExcelFactory.writerSheet(sheetNo).build();  
            excelWriter.write(list, writeSheet);  
            list.clear();  
        }  
    }  
    excelWriter.write(list, writeSheet);  
    cursor.close();  
  
    excelWriter.finish();  
    return file;  
}
```

# 3. 尝试解决问题
	怀疑是fetchSize=500000 值定的太大了，导致OOM,将fetchSize改为1000，结果起初没有报错，执行一段时间后还是 OOM，并没有解决问题
	 源代码 mapper.xml
```xml
<select id="findClueTrafficList" resultMap="clueTrafficDTO" resultSetType="FORWARD_ONLY" fetchSize="1000">  
    SELECT  xxx from xxx
</select>
```
# 4. 跟读源码
对应方法
``` java
org.apache.ibatis.executor.statement.RoutingStatementHandler#queryCursor
org.apache.ibatis.executor.statement.PreparedStatementHandler#queryCursor
org.apache.ibatis.executor.resultset.ResultSetHandler#handleCursorResultSets
com.zaxxer.hikari.pool.ProxyPreparedStatement#execute
com.mysql.cj.jdbc.StatementImpl#setupStreamingTimeout
com.mysql.cj.jdbc.StatementImpl#createStreamingResultSet
```


![[Pasted image 20240606143726.png]]

##### 核心方法
> 需要同时满足三个条件，才会是流式查询，因为没有满足 this.query.getResultFetchSize() == Integer.MIN_VALUE，所有没有使用流式查询，最终出现OOM
![[Pasted image 20240606143839.png]]

# 5. 再次尝试
	修改fetchSize
	源代码 mapper.xml
```xml
<select id="findClueTrafficList" resultMap="clueTrafficDTO" resultSetType="FORWARD_ONLY" fetchSize="-2147483648">  
    SELECT  xxx from xxx
</select>
```

堆大小 `-Xmx512m -Xms512m` ， 成功导出100W+ 数据，问题解决
![[Pasted image 20240606133952.png]]
对应源代码

com.mysql.cj.jdbc.StatementImpl#createStreamingResultSet
![[Pasted image 20240606134034.png]]
![[Pasted image 20240606134213.png]]
![[Pasted image 20240606134547.png]]