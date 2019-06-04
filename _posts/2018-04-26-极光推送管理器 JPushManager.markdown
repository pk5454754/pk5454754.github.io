```java
import java.util.Map;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import cn.jpush.api.JPushClient;
import cn.jpush.api.common.resp.APIConnectionException;
import cn.jpush.api.common.resp.APIRequestException;
import cn.jpush.api.push.PushResult;
import cn.jpush.api.push.model.Message;
import cn.jpush.api.push.model.Platform;
import cn.jpush.api.push.model.PushPayload;
import cn.jpush.api.push.model.PushPayload.Builder;
import cn.jpush.api.push.model.audience.Audience;
import cn.jpush.api.push.model.audience.AudienceTarget;
import cn.jpush.api.push.model.notification.IosNotification;
import cn.jpush.api.push.model.notification.Notification;

/**
 * 极光推送管理器
 * @author ks
 * @date 2018-04-19
 *
 */
public class JPushManager {
	private static final Log LOG = LogFactory.getLog(JPushManager.class);

	public static final String PLATFORM_ALL="platform_all";
	public static final String PLATFORM_ANDROID="platform_android";
	public static final String PLATFORM_IOS="platform_ios";

	private JPushClient jpushClient = null;
	private String appKey;
	private String masterSecret;
	/**
	 * 机器id
	 */
	private String[] registrationIds;
	/**
	 * 平台
	 */
	private String platform;
	/**
	 * 通知附加信息
	 */
	private Map<String,String> extras;
	/**
	 * 通知内容(通知在客户端有现成的处理方式)
	 */
	private String notification;
	/**
	 * 通知标题
	 */
	private String notificationTitle;
	/**
	 * 消息内容（消息可以在客户端自定义处理方式）
	 */
	private String content;
	/**
	 * 消息附加信息
	 */
	private Map<String,String> mExtras;
	public void create(String key,String secret){
		setAppKey(key);
		setMasterSecret(secret);
		jpushClient=new JPushClient(masterSecret, appKey);
		//默认全平台
		platform=JPushManager.PLATFORM_ALL;
	}

	public PushPayload getPushPayload(){
		Builder builder=PushPayload.newBuilder();
		//设置平台
		if(JPushManager.PLATFORM_ALL.equals(platform)){
			builder.setPlatform(Platform.all());
		}else if(JPushManager.PLATFORM_ANDROID.equals(platform)){
			builder.setPlatform(Platform.android());
		}else if(JPushManager.PLATFORM_IOS.equals(platform)){
			builder.setPlatform(Platform.ios());
		}

		//设置发送对象
		cn.jpush.api.push.model.audience.Audience.Builder aBuilder=Audience.newBuilder();
		if(registrationIds!=null){
			aBuilder.addAudienceTarget(AudienceTarget.registrationId(registrationIds));
		}
		//这里可以设置其它对象
		builder.setAudience(aBuilder.build());
		//设置通知
		if(notification!=null){
			if(JPushManager.PLATFORM_ALL.equals(platform)){
				//全平台通知
				builder.setNotification(Notification.alert(notification));
			}else if(JPushManager.PLATFORM_ANDROID.equals(platform)){
				//安卓通知
				builder.setNotification(Notification.android(notification, notificationTitle, extras));
			}else if(JPushManager.PLATFORM_IOS.equals(platform)){
				//苹果通知
				builder.setNotification(
						Notification.newBuilder().addPlatformNotification(
								IosNotification.newBuilder().setAlert(notification)
								.addExtras(extras).build()).build()
						);
			}else{
				//新增平台在这里加逻辑，现在没用到就不写了。
			}
		}

		//设置消息
		if(content!=null){
			cn.jpush.api.push.model.Message.Builder mBuilder=Message.newBuilder();
			mBuilder.setMsgContent(content);
			if(!mExtras.isEmpty()){
				for(Map.Entry<String, String> map:extras.entrySet()){
					mBuilder.addExtra(map.getKey(), map.getValue());
				}
			}
			builder.setMessage(mBuilder.build());
		}

		PushPayload pp=builder.build();
		return pp;
	}

	public void sendPush(PushPayload pp){
		 try {
		        PushResult result = jpushClient.sendPush(pp);
		        LOG.info("Got result - " + result);
		    } catch (APIConnectionException e) {
		        // Connection error, should retry later
		        LOG.error("Connection error, should retry later", e);
		    } catch (APIRequestException e) {
		        // Should review the error, and fix the request
		        LOG.error("Should review the error, and fix the request", e);
		        LOG.info("HTTP Status: " + e.getStatus());
		        LOG.info("Error Code: " + e.getErrorCode());
		        LOG.info("Error Message: " + e.getErrorMessage());
		    }
	}

	public String getMasterSecret() {
		return masterSecret;
	}
	public void setMasterSecret(String masterSecret) {
		this.masterSecret = masterSecret;
	}
	public String getAppKey() {
		return appKey;
	}
	public void setAppKey(String appKey) {
		this.appKey = appKey;
	}
	public String[] getRegistrationIds() {
		return registrationIds;
	}
	public void setRegistrationIds(String[] registrationIds) {
		this.registrationIds = registrationIds;
	}
	public String getPlatform() {
		return platform;
	}
	public void setPlatform(String platform) {
		this.platform = platform;
	}
	public Map<String,String> getExtras() {
		return extras;
	}
	public void setExtras(Map<String,String> extras) {
		this.extras = extras;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}

	public JPushClient getJpushClient() {
		return jpushClient;
	}

	public void setJpushClient(JPushClient jpushClient) {
		this.jpushClient = jpushClient;
	}

	public String getNotification() {
		return notification;
	}

	public void setNotification(String notification) {
		this.notification = notification;
	}

	public Map<String,String> getmExtras() {
		return mExtras;
	}

	public void setmExtras(Map<String,String> mExtras) {
		this.mExtras = mExtras;
	}

	public String getNotificationTitle() {
		return notificationTitle;
	}

	public void setNotificationTitle(String notificationTitle) {
		this.notificationTitle = notificationTitle;
	};
}
```
