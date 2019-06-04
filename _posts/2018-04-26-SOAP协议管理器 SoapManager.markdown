### 管理器代码
```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import com.ialtus.nhs.inf.SoapTypeEditor;

/**
 * @author ks
 * @date 2018-04-02
 * soap生成管理器
 */
public class SoapManager {
	private StringBuffer soapString=new StringBuffer();

	private SoapTypeEditor soapTypeEditor=null;

	private Map<String,String> xmlns=new HashMap<String,String>();

	/**
	 * soap命名空间的名字
	 */
	private String soap="soap";

	private String encodingStyle="http://www.w3.org/2001/12/soap-encoding";

	private List<SoapElement> elements;


	/**
	 * 自定义添加内容
	 * @param str
	 */
	public void addString(String str){
		soapString.append(str);
	}

	public void addXMLHeader(){
		soapString.append("<?xmlversion=\"1.0\"?>");
	}

	public void addEnvelopeBegin(){
		soapString.append("<").append(soap).append(":Envelope ");
		soapString.append("xmlns:").append(soap).append("=").append("\"http://schemas.xmlsoap.org/soap/envelope/\"").append(" ");
		if(xmlns!=null&&xmlns.size()>0){
			for(Entry<String, String> entry : xmlns.entrySet()){
				if(entry.getValue()!=null){
					soapString.append("xmlns:").append(entry.getKey()).append("=")
					.append("\"").append(entry.getKey()).append("\"").append(" ");
				}
			}
		}
		if(encodingStyle!=null){
			soapString.append(soap).append(":encodingStyle=").append("\"").append(encodingStyle).append("\"").append(" ");
		}
		soapString.append(">");
	}

	/**
	 * 添加body元素
	 */
	public void addSoapElement(){
		if(elements!=null&&elements.size()>0){
			for(int i=0;i<elements.size();i++){
				if(elements.get(i).getChildren()!=null){
					soapString.append("<").append(elements.get(i).getXmlns()).append(":").append(elements.get(i).getKey()).append(" ").append(">");
					addChildren(elements.get(i).getChildren());
					soapString.append("</").append(elements.get(i).getXmlns()).append(":").append(elements.get(i).getKey()).append(" ").append(">");
				}else if(elements.get(i).getValue()!=null){
					addValue(elements.get(i));
				}else{
					continue;
				}
			}
		}
	}

	/**
	 * 添加子元素
	 * @param list
	 */
	public void addChildren(List<SoapElement> list){
		for(int i=0;i<list.size();i++){
			if(list.get(i).getChildren()!=null){
				soapString.append("<").append(list.get(i).getXmlns()).append(":").append(list.get(i).getKey()).append(" ").append(">");
				addChildren(list.get(i).getChildren());
				soapString.append("</").append(list.get(i).getXmlns()).append(":").append(list.get(i).getKey()).append(" ").append(">");
			}else if(elements.get(i).getValue()!=null){
				addValue(elements.get(i));
			}else{
				continue;
			}
		}
	}

	/**
	 * 添加值
	 * @param se
	 */
	public void addValue(SoapElement se){
		soapString.append(this.getSoapTypeEditor().addValue(se));
	}

	public void addEnvelopeEnd(){
		soapString.append("</").append(soap).append(":Envelope>");
	}
	public void addHeaderBegin(){
		soapString.append("<").append(soap).append(":Header>");
	}
	public void addHeaderEnd(){
		soapString.append("</").append(soap).append(":Header>");
	}
	public void addBodyBegin(){
		soapString.append("<").append(soap).append(":Body>");
	}
	public void addBodyEnd(){
		soapString.append("</").append(soap).append(":Body>");
	}

	/**
	 * 自动生成，上面的可以手动生成
	 * @return
	 */
	public String autoToSoap(){
		addXMLHeader();
		addEnvelopeBegin();
		addHeaderBegin();
		addHeaderEnd();
		addBodyBegin();
		addSoapElement();
		addBodyEnd();
		addEnvelopeEnd();
		return soapString.toString();
	}


	public StringBuffer getSoapString() {
		return soapString;
	}

	public void setSoapString(StringBuffer soapString) {
		this.soapString = soapString;
	}



	public String getSoap() {
		return soap;
	}

	public void setSoap(String soap) {
		this.soap = soap;
	}

	public String getEncodingStyle() {
		return encodingStyle;
	}

	public void setEncodingStyle(String encodingStyle) {
		this.encodingStyle = encodingStyle;
	}

	public Map<String, String> getXmlns() {
		return xmlns;
	}

	public void setXmlns(Map<String, String> xmlns) {
		this.xmlns = xmlns;
	}

	public List<SoapElement> getElement() {
		return elements;
	}

	public void setElement(List<SoapElement> element) {
		this.elements = element;
	}

	public SoapTypeEditor getSoapTypeEditor() {
		if(soapTypeEditor==null){
			soapTypeEditor=new CommonTypeEditor();
		}
		return soapTypeEditor;
	}

	public void setSoapTypeEditor(SoapTypeEditor soapTypeEditor) {
		this.soapTypeEditor = soapTypeEditor;
	}
}

```

### soap元素

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author ks
 * @date 2018-04-02
 * soap元素Bean
 */
public class SoapElement {
	private String xmlns;
	private String key;
	private Object value;
	private List<SoapElement> children;
	public SoapElement(){
		super();
	}
	public SoapElement(String xmlns,String key,Object value){
		this.xmlns=xmlns;
		this.key=key;
		this.value=value;
	}

	/**
	 * yjc
	 * 20180212
	 * 复制对象有对应set，get方法的属性，生成对应SoapElement
	 * @param c
	 * @param bean
	 * @return
	 */
	public static SoapElement cloneToSoapElement(String xmlns,Class<?> c,String name,Object bean){
		SoapElement obj=null;
		if(bean==null&&c==null){
			return null;
		}else if(bean==null&&c!=null){
			obj= new SoapElement();
			obj.setXmlns(xmlns);
			obj.setKey(name);
			obj.setValue(null);
			obj.setChildren(null);
			return obj;
		}

		try {
			obj= new SoapElement();
			obj.setXmlns(xmlns);
			obj.setKey(name);
			List<SoapElement> list=new ArrayList<SoapElement>();
			if(isSimpleObject(bean)){
				//简单类型返回只有Value的元素
				obj.setValue(bean);
				return obj;
			}else if(bean instanceof List){
				List<?> bList=(List<?>)bean;
				if(bList!=null&&bList.size()>0){
					for(int i=0;i<bList.size();i++){
						if(bList.get(i) instanceof SoapElement){
							SoapElement bListObject=(SoapElement) bList.get(i);
							list.add(bListObject);
						}else{
							Object bListObject= bList.get(i);
							if(bListObject!=null){
								SoapElement tSe=SoapElement.cloneToSoapElement(xmlns, bListObject.getClass(),bListObject.getClass().getName(), bListObject);
								obj.setValue(tSe);
								return obj;
							}
						}
					}
				}
				if(list.size()>0){
					obj.setChildren(list);
				}
				return obj;
			}else if(bean instanceof Map){
				//返回没有Value和没有Children的元素
				return obj;
			}else{
				//bean中所有公共方法
				Method[] bmt=bean.getClass().getMethods();
				Map<String,Method> bmap=new HashMap<String,Method>();
				for(int i=0;i<bmt.length;i++){
					bmap.put(bmt[i].getName(), bmt[i]);
				}
				//新对象方法
				Method[] mt=c.getMethods();
				List<String> getList=new ArrayList<String>();
				List<String> setList=new ArrayList<String>();
				Map<String,Method> map=new HashMap<String,Method>();
				for(int i=0;i<mt.length;i++){
					String methodName=mt[i].getName();
					//bean中有对应方法才处理
					if(bmap.get(methodName)!=null){
						map.put(methodName, mt[i]);
						if(methodName.startsWith("get")){
							getList.add(methodName);
						}else if(methodName.startsWith("set")){
							setList.add(methodName);
						}
					}
				}
				int setSize = setList.size();
				if(setSize>0){
					for(int i=0;i<setSize;i++){
						String getStr=setList.get(i).replaceFirst("set", "get");
						if(getList.contains(getStr)){//同时有set,get方法的字段
							Method getMethod=map.get(getStr);
							String key = getStr.substring(getStr.indexOf("get") + 3);
							key = key.toLowerCase().charAt(0) + key.substring(1);
							Object value = getMethod.invoke(bean);
							SoapElement cObj=SoapElement.cloneToSoapElement(xmlns, getMethod.getReturnType(),key, value);
							list.add(cObj);
						}
					}
					if(list.size()>0){
						obj.setChildren(list);
					}
					return obj;
				}
			}
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		}
		return null;
	}

	public static boolean isSimpleObject(Object obj){
		if(obj instanceof String||obj instanceof Integer||obj instanceof Double
				||obj instanceof Boolean||obj instanceof Date){
			return true;
		}
		return false;
	}

	public String getXmlns() {
		return xmlns;
	}
	public void setXmlns(String xmlns) {
		this.xmlns = xmlns;
	}
	public String getKey() {
		return key;
	}
	public void setKey(String key) {
		this.key = key;
	}
	public Object getValue() {
		return value;
	}
	public void setValue(Object value) {
		this.value = value;
	}
	public List<SoapElement> getChildren() {
		return children;
	}
	public void setChildren(List<SoapElement> children) {
		this.children = children;
	}
}

```
### soap通用类型处理器

```java
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import com.ialtus.nhs.inf.SoapTypeEditor;

/**
 * @author ks
 * @date 2018-04-02
 * soap通用的类型处理器
 */
public class CommonTypeEditor implements SoapTypeEditor {

	@Override
	public String addValue(SoapElement se) {
		if(se==null){
			return "";
		}
		StringBuffer sb=new StringBuffer();
		Object obj=se.getValue();
		sb.append("<").append(se.getXmlns()).append(":").append(se.getKey()).append(" ").append(">");
		if(se.getValue()!=null){
			if(obj instanceof Date){
				Date date=(Date) obj;
				SimpleDateFormat sdf =new SimpleDateFormat("yyyy-MM-dd");
				sb.append(sdf.format(date));
			}else if(obj instanceof SoapElement){
				sb.append(this.addValue((SoapElement)obj));
			}else if(obj instanceof List){
				List<SoapElement> list=(List<SoapElement>) obj;
				sb.append(this.addChildren(list));
			}else{
				sb.append(obj.toString());
			}
		}else if(se.getChildren()!=null){
			sb.append(this.addChildren(se.getChildren()));
		}

		sb.append("</").append(se.getXmlns()).append(":").append(se.getKey()).append(" ").append(">");
		return sb.toString();
	}
	@Override
	public String addChildren(List<SoapElement> list){
		StringBuffer sb=new StringBuffer();
		for(int i=0;i<list.size();i++){
			sb.append(addValue(list.get(i)));
		}
		return sb.toString();
	}

}

```
### 接口
```java
import java.util.List;

import com.ialtus.nhs.util.SoapElement;

/**
 * @author ks
 * @date 2018-04-02
 * soap类型编辑器，处理SoapElement不同类型的Value
 */
public interface SoapTypeEditor {
	public String addValue(SoapElement se);

	public String addChildren(List<SoapElement> list);
}

```
