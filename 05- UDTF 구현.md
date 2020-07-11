# UDTF 구현

UDTF는 GenericUDTF를 상속하여 구현합니다.

 - initialize()
   - 입력 파라미터의 검증과 칼럼이름을 반환
 - process()
   - 실제 데이터 처리를 반환
   - forward() 함수에 데이터를 넘김
 - close()
   - 자원의 반환을 처리
   
# 구현
다음은 딜리미터를 입력받아 문자열을 분할하고 테이블 데이터를 반환하는 UDTF 예제 입니다.
```
import java.util.ArrayList;

import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.PrimitiveObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.io.Text;

@Description(name = "string_parse", value = "_FUNC_(delimiter, string) - ")
public class StringParseUDTF extends GenericUDTF {

    private transient final Object[] forwardListObj = new Object[1];
    protected PrimitiveObjectInspector inputOI1;
    protected PrimitiveObjectInspector inputOI2;

    @Override
    public StructObjectInspector initialize(ObjectInspector[] argOIs) throws UDFArgumentException {

        inputOI1 = (PrimitiveObjectInspector) argOIs[1];
        inputOI2 = (PrimitiveObjectInspector) argOIs[1];

        ArrayList<String> fieldNames = new ArrayList<String>();
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<ObjectInspector>();

        fieldNames.add("col");
        fieldOIs.add(inputOI1);
        fieldOIs.add(inputOI2);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);
    }

    @Override
    public void process(Object[] o) throws HiveException {


        String delim = (String) inputOI1.getPrimitiveJavaObject(o[0]);
        String datas = (String) inputOI2.getPrimitiveJavaObject(o[1]);

        for(String str: datas.split(delim)) {
            forwardListObj[0] = new Text(str);
            forward(forwardListObj);
        }
    }

    @Override
    public void close() throws HiveException { }
}
```

## 사용방법
```   
-- UDF가 포함된 JAR 추가 및 함수 생성 
ADD JAR hdfs:///user/hiveUDF.jar;
CREATE TEMPORARY FUNCTION parseStr AS 'com.sec.hive.udf.StringParseUDTF';


hive> SELECT parseStr(",", "1,2,3");
OK
1
2
3

hive> SELECT parseStr("-", "a-b-c");
OK
a
b
c
```   
   
 # 참고
  - [hive explode 함수 구현](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/udf/generic/GenericUDTFExplode.java)
  - [하이브 UDTF 구현 위키](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)
   
   
   
   
   
   
   
   
   
   
