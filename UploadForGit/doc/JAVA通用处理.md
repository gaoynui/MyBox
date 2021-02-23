## JAVA通用处理

### 日期处理

```
# 获取当前日期
Date date = new Date();
# 获取当前日期加时间
Date nowDate = new Date();
Calendar cal = Calendar.getInstance();
cal.get(Calendar.HOUR_OF_DAY);
cal.get(Calendar.MINUTE);
cal.get(Calendar.SECOND);
cal.get(Calendar.MILLISECOND);
nowDate = cal.getTime();
# Date转String
SimpleDateFromat sdf = new SimpleDateFormat("yyyy-MM-dd");
//SimpleDateFromat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date date = new Date();
String dateString = sdf.format(date);
# 给出天数获取其所在周和年月
Integer year = DateUtil.getYear(date);
Integer month = Dateutil.getMonth(date);
Integer week = Dateutil.getWeekOfYear(date);
# Calender获取当前日，时，分，秒                    //如：12-25 09:58:23
Calendar cal = Calendar.getInstance();        
int date = cal.get(Calendar.DATE);                //25
int hour = cal.get(Calendar.HOUR_OF_DAY);         //9
int min = cal.get(Calendar.MINUTE);               //58
int sec = cal.get(Calendar.SECOND);               //23
```

### BufferReader按行读取

```
FileInputStream inputStream = new FileInputStream(filePath);
BufferReader bufferReader = new BufferReader(new InputStreamReader(inputStream));
String str = null;
while((str = bufferReader.readLine()) != null) {}
inputStream.close();
bufferedReader.close();
```

## json格式的String转json

```
String str = "{a:1,b:2}";
JSONObject json = new JSONObject(str);
//json = {"a":1,"b":2}
# 判断是否存在该键值
if(json.has("a")) {}
# 获取键对应的值
String value = json.get("a").toString();
```

## 对象以标准String形式输出

```
import com.fasterxml.jackson.databind.ObjectMapper;

@Autowired
    ObjectMapper objectMapper;
    
String result = objectMapper.writeValueAsString(Class/JSON/Object);
```

