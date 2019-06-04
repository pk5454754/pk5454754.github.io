```java
/**
	 * ks
	 * 20180212
	 * 复制对象有对应set，get方法的属性
	 * @param c
	 * @param bean
	 * @return
	 */
	public static <T> T cloneBean(Class<T> c,Object bean){
		T obj=null;
		try {
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
				obj= c.newInstance();
				for(int i=0;i<setSize;i++){
					String getStr=setList.get(i).replaceFirst("set", "get");
					if(getList.contains(getStr)){//同时有set,get方法的字段
						Method getMethod=map.get(getStr);
						Method setMethod=map.get(setList.get(i));
						setMethod.invoke(obj, getMethod.invoke(bean));
					}
				}
				return obj;
			}
		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		}
		return null;
	}
```
