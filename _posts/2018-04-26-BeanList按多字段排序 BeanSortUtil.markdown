```java

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**
 * @author ks
 * @date 2018-09-27
 * BeanList按多字段排序
 */
public class BeanSortUtil {
    
	/**
	 * 选择排序法
	 */
	public static final Integer SELECTION=1;
	/**
	 * 鸡尾酒排序法（冒泡排序法改进）
	 */
	public static final Integer COCKTAIL=2;
	/**
	 * 快速排序法
	 */
	public static final Integer FAST=3;

	/**
	 * @param list 排序的list
	 * @param sortColumn 排序的字段，优先级高的放前面，倒序需在字段后面加上《空格+desc》（和sql一样）
	 * @return 排序好的list
	 * demo
	 *
	 */
	public static List sortList(List list,List<String> sortColumn) {
		BeanSortUtil.sortList(list, sortColumn, SELECTION);
		return list;
	}
	
	/**
	 * @param list 排序的list
	 * @param sortColumn 排序的字段，优先级高的放前面，倒序需在字段后面加上《空格+desc》（和sql一样）
	 * @param type 排序类型
	 * @return 排序好的list
	 * demo
	 *
	 */
	public static List sortList(List list,List<String> sortColumn,Integer type) {
		if (sortColumn == null || sortColumn.size() <= 0) {
			return list;
		}
		if(list==null||list.size()<=0){
			return list;
		}
		Class c=list.get(0).getClass();
		
		//true：正序    false：倒序
		List<Boolean> methodSortType=new ArrayList<Boolean>();
		List<Method> methodList = new ArrayList<Method>();
		Method[] methodArr = c.getMethods();
		for (int i = 0; i < sortColumn.size(); i++) {
			if(sortColumn.get(i).endsWith(" desc")){
				sortColumn.set(i, sortColumn.get(i).substring(0, sortColumn.get(i).length()-5));
				methodSortType.add(false);
			}else{
				methodSortType.add(true);
			}
			String methodName = "get" + sortColumn.get(i).substring(0, 1).toUpperCase()
					+ sortColumn.get(i).substring(1);
			for (int j = 0; j < methodArr.length; j++) {
				if (methodArr[j].getName().equals(methodName)) {
					methodList.add(methodArr[j]);
				}
			}
		}
		if(SELECTION.equals(type)){
			BeanSortUtil.sortBySelection(methodList,methodSortType,list);
		}else if(COCKTAIL.equals(type)){
			BeanSortUtil.sortByCocktail(methodList,methodSortType,list);
		}
		return list;
	}

	/**
	 * 鸡尾酒排序法（稳定）
	 * @param methodList
	 * @param methodSortType
	 * @param list
	 */
	private static void sortByCocktail(List<Method> methodList, List<Boolean> methodSortType, List list) {
		boolean isSort=false;//是否有序
		for (int i = 0; i < methodList.size(); i++) {
			Method tMethod = methodList.get(i);
			int left = 0;                            // 初始化边界
		    int right = list.size() - 1;
		    while (left < right)
		    {
		    	isSort=true;//假设是有序的
		        for (int j = left; j < right; j++)   // 前半轮,将最大元素放到后面
		        {
		        	try {
						if (ifJump(methodList, i, list, j, j+1)) {
							continue;
						}
						int compareResult = BeanSortUtil.compareObject(tMethod.invoke(list.get(j)),
								tMethod.invoke(list.get(j+1)));
						if(ifSwap(compareResult,methodSortType.get(i))){
							BeanSortUtil.swap(list, j, j+1);
							isSort=false;//交换位置说明是无序的
						}
		        	} catch (IllegalAccessException e) {
						e.printStackTrace();
					} catch (IllegalArgumentException e) {
						e.printStackTrace();
					} catch (InvocationTargetException e) {
						e.printStackTrace();
					}
		        }
		        if(isSort) {
		        	//有序就退出循环
		        	break;
		        }
		        right--;//右边有序标识左移
		        isSort=true;//第二轮假设是有序的
		        for (int j = right; j > left; j--)   // 后半轮,将最小元素放到前面
		        {
		        	try {
						if (ifJump(methodList, i, list, j-1, j)) {
							continue;
						}
						int compareResult = BeanSortUtil.compareObject(tMethod.invoke(list.get(j-1)),
								tMethod.invoke(list.get(j)));
						if(ifSwap(compareResult,methodSortType.get(i))){
							BeanSortUtil.swap(list, j-1, j);
							isSort=false;//交换位置说明是无序的
						}
		        	} catch (IllegalAccessException e) {
						e.printStackTrace();
					} catch (IllegalArgumentException e) {
						e.printStackTrace();
					} catch (InvocationTargetException e) {
						e.printStackTrace();
					}
		        }
		        if(isSort) {
		        	//有序就退出循环
		        	break;
		        }
		        left++;//左边有序标识右移
		       
		       
		    }
		}
	}

	/**
	 * 选择排序法(不稳定)
	 * @param methodList
	 * @param methodSortType
	 * @param list
	 */
	private static void sortBySelection(List<Method> methodList,List<Boolean> methodSortType,List list) {
		for (int i = 0; i < methodList.size(); i++) {
			Method tMethod = methodList.get(i);
			for (int j = 0; j < list.size() - 1; j++) {
				for (int k = j + 1; k < list.size(); k++) {
					try {
						// 前面的列相同才进行当前列的比较
						if (ifJump(methodList, i, list, j, k)) {
							break;
						}
						int compareResult = BeanSortUtil.compareObject(tMethod.invoke(list.get(j)),tMethod.invoke(list.get(k)));
						if(ifSwap(compareResult,methodSortType.get(i))){
							BeanSortUtil.swap(list, j, k);
						}
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
			}
		}
	}
	
	/**
	 * j和k两个比较对象，正在比较第i个字段时，i以前的字段有一个不一样都不进行比较，因为之前比较过的字段优先级比较高，所以跳过
	 * @param methodList
	 * @param i
	 * @param list
	 * @param j
	 * @param k
	 * @return
	 * @throws IllegalAccessException
	 * @throws IllegalArgumentException
	 * @throws InvocationTargetException
	 */
	private static boolean ifJump(List<Method> methodList,int i,List list,int j,int k) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException{
		boolean flag = false;// 默认比较
		for (int l = i - 1; l >= 0; l--) {
			Method tMethodBefore = methodList.get(l);
			if(tMethodBefore.invoke(list.get(j))==null){
				if(tMethodBefore.invoke(list.get(k))!=null){
					flag = true;// 前面比较过的列有一项不一致都不比较
					break;
				}
			}else if (!tMethodBefore.invoke(list.get(j)).equals(tMethodBefore.invoke(list.get(k)))) {
				flag = true;// 前面比较过的列有一项不一致都不比较
				break;
			}
		}
		return flag;
	}
	
	/**
	 * 是否交换
	 * 当a>b，正序排列时交换
	 * 当a<b，倒序排列时交换
	 * @param compareResult
	 * @param methodSortType
	 * @return
	 */
	private static boolean ifSwap(int compareResult,boolean methodSortType){
		if(compareResult > 0&&methodSortType){
			return true;
		}
		if(compareResult<0&&!methodSortType){
			return true;
		}
		return false;
	}
	
	
	private static void swap(List list,int j,int k){
		Object temp = list.get(k);
		list.set(k, list.get(j));
		list.set(j, temp);
	}

	/**
	 * @param a
	 * @param b
	 * @return 1:a>b 0:a==b -1:a<b 支持类型：String,Integer,其它自己加
	 */
	public static int compareObject(Object a, Object b) {
		if (a == null && b == null) {
			return 0;
		}else if(a==null){
			return -1;
		}else if(b==null){
			return 1;
		}
		if (!a.getClass().equals(b.getClass())) {
			return 0;
		}
		if (a instanceof String) {
			if (a.toString().compareTo(b.toString()) > 0) {
				return 1;
			} else if (a.toString().compareTo(b.toString()) < 0) {
				return -1;
			} else if (a.toString().compareTo(b.toString()) == 0) {
				return 0;
			}
		} else if (a instanceof Integer) {
			Integer a1 = Integer.parseInt(a.toString());
			Integer b1 = Integer.parseInt(b.toString());
			if (a1 > b1) {
				return 1;
			} else if (a1 < b1) {
				return -1;
			} else {
				return 0;
			}
		}else if (a instanceof Date) {
			Date a1 = (Date) a;
			Date b1 = (Date) b;
			if (a1.after(b1) ) {
				return 1;
			}else if (a1.before(b1) ) {
				return -1;
			}else{
				return 0;
			}
		}else if (a instanceof Boolean) {
			Boolean a1 = (Boolean) a;
			Boolean b1 = (Boolean) b;
			if (a1.booleanValue()&&!b1.booleanValue()) {
				return 1;
			}else if (!a1.booleanValue()&&b1.booleanValue()) {
				return -1;
			}else{
				return 0;
			}
		}
		return 0;
	}
    public static void main(String[] args) {
        String m="house desc";
        System.out.println(m.substring(0, m.length()-5));;
    }

}
```
