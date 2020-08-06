# Java读取Unicode文件（UTF-8等）时碰到的BOM首字符问题，及处理方法
---------------------------
## 在Windows下用文本编辑器创建的文本文件，如果选择以UTF-8等Unicode格式保存，会在文件头（第一个字符）加入一个BOM标识。

## 这个标识在Java读取文件的时候，不会被去掉，而且String.trim()也无法删除。如果用readLine()读取第一行存进String里面，这个String的length会比看到的大1，而且第一个字符就是这个BOM。

## 这种情况会造成一些麻烦，比如在读取ini文件的时候，如果想判断第一行是不是以“[”开头就无法正确判断。
-----------------------------
+ 幸好，Java在读取Unicode文件的时候，会统一把BOM变成“\uFEFF”，这样的话，就可以自己手动解决了（判断后，用substring()或replace()去除掉这个BOM）：
```
 if(line.startsWith("\uFEFF")){
   //line = line.substring(1);
   line = line.replace("\uFEFF", "");
  }
```
-----------------------------------
+ 然而，这种方法并不是完美的，如果生成jar文件在windows下运行，还是有问题。终极的解决方法是使用apache commons io提供的BOMInputStream：
```
<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>
```
---------------------------------
```
BufferedReader reader = null;
        try {
            //reader = new BufferedReader(new FileReader(file));
        	
        	//使用BOMInputStream自动去除UTF-8中的BOM！！！
        	reader = new BufferedReader(new InputStreamReader(new BOMInputStream(new FileInputStream(file))));
 
        	String str = null;
            //一次读入一行（非空），直到读入null为文件结束
            while ((str = reader.readLine()) != null) {
            }
```
