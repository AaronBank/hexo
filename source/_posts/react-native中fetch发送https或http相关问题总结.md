---
title: react-native中使用fetch发送https或http相关问题总结
date: 2019-07-30 16:37:19
tags: ReactNative
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/150542.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/150542.jpg) 



`react-native`再请求方面有点坑，`Ios`端默认支持`https`，而安卓默认支持的事`http`，如果你需要的是其中的某一种方式，那这篇文章可能能帮到你😍😍



### 当前环境

- OS: macOS High Sierra 10.13.5
- Node: 8.11.3
- Yarn: 1.7.0
- npm: 5.6.0
- Watchman: 4.9.0
- Xcode: Xcode 9.4.1
- react: 16.3.1 => 16.3.1
- react-native: 0.55.4 => 0.55.4



### 前置说明

1. 在ios平台ios9.0以上默认支持https而不支持http
2. 对于https，如果你的服务器端是CA签发的证书，那你ios平台不需要改动任何代码
3. 对于自签证书，那你需要对源代码进行改动，下面会详细说明方式
4. 对于http，有两个方式可以做到，下面会详细说明方式
    1. 直接使用ip地址
    2. 使用域名，对源代码进行配置

  

### 关于ios支持https自签证书


- 服务器端使用自签证书，免证书方法
- 打开XCode在当前项目路径为`Libraries/RCTNetwork.xcodeproj`找到`RCTHTTPRequestHandler.mm`
- 或者在`{project}/node_modules/react-native/Libraries/Network`目录下同样可以找到该文件
- 在文件中搜索`#pragma mark - NSURLSession delegate`并在其后面添加如下代码即可


```c++
- (void)URLSession:(NSURLSession *)session  didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge  completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * __nullable credential))completionHandler{
    if (RCT_DEBUG) {
        completionHandler(NSURLSessionAuthChallengeUseCredential,[NSURLCredential  credentialForTrust:challenge.protectionSpace.serverTrust]);
    }
}
```

- 配置如图所示

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/140902-1.png) 



### 关于ios支持http请求

- 关于http方式我们只介绍第二种因为直接使用ip地址真的是太。。简单，直接把域名换成ip就好了
- 对于第二种你需要改动`{project}/ios/App`下面的`Info.plist`文件


```xml
<key>NSAppTransportSecuritykey>
<dict>
    <key>NSAllowsArbitraryLoadskey></key>
    <true/>
<dict>
```

- 配置如图所示

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/140902.png) 

到此为止`Ios`相关问题就解决了，`Ios`部分还是比较好弄得O(∩_∩)O不是很麻烦，`Android`得话就麻烦些，下面我们说下`Android`相关的解决方案



### 关于Android端http和https你需要知道

1. 安卓默认是支持http请求儿不支持https
2. 想做到支持https那就需要改动react-native的源代码，进而重新编译才能成功


### 关于Android支持https
- 下载ndk（下载r10e版本的，其他的版本会有问题）
	1.  Mac OS (64-bit) - [http://dl.google.com/android/repository/android-ndk-r10e-darwin-x86_64.zip](http://dl.google.com/android/repository/android-ndk-r10e-darwin-x86_64.zip)
	2.  Linux (64-bit) - [http://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip](http://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip)
	3.  Windows (64-bit) - [http://dl.google.com/android/repository/android-ndk-r10e-windows-x86_64.zip](http://dl.google.com/android/repository/android-ndk-r10e-windows-x86_64.zip)
	4.  Windows (32-bit) - [http://dl.google.com/android/repository/android-ndk-r10e-windows-x86.zip](http://dl.google.com/android/repository/android-ndk-r10e-windows-x86.zip)

- 在项目`{project}/android`文件夹先创建`local.properties`文件（如果已经存在哪直接修改就好了）


```properties
sdk.dir=你自己的sdk路径
ndk.dir=你自己的ndk路径
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/140900-1.jpg)

- 在`{project}/android/build.gradle`中添加`gradle-download-task`依赖

```java
...
dependencies {
    classpath 'com.android.tools.build:gradle:2.2.3'
    classpath 'de.undercouch:gradle-download-task:3.1.2'
	
    // NOTE: Do not place your application dependencies here; they belong
    // in the individual module build.gradle files
}
...
```


- 在`{project}/android/settings.gradle`中添加`:ReactAndroid`项目


```java
...
    include ':ReactAndroid'
  
    project(':ReactAndroid').projectDir = new File(rootProject.projectDir, '../node_modules/react-native/ReactAndroid')
...
```

- 修改`{project}/android/app/build.gradle`文件，使用:ReactAndroid替换预编译库


```java
...
  
compile fileTree(dir: "libs", include: ["*.jar"])
compile "com.android.support:appcompat-v7:23.0.1"
compile project(':ReactAndroid')
  
...
```

- 重写第三方模块的依赖，否则会报错`-Error: more than one library with package name 'com.facebook.react'`
- 修改`{project}/android/app/build.gradle`文件，添加以下代码


```java
...
  
configurations.all {
    exclude group: 'com.facebook.react', module: 'react-native'
}
  
...
```



- 重写`okHttp`类

```java
package com.facebook.react.modules.network;

import android.net.Uri;
import android.util.Base64;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.GuardedAsyncTask;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.ReadableArray;
import com.facebook.react.bridge.ReadableMap;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.common.StandardCharsets;
import com.facebook.react.common.network.OkHttpCallUtil;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.modules.core.DeviceEventManagerModule.RCTDeviceEventEmitter;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.concurrent.TimeUnit;
import javax.annotation.Nullable;
import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSession;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.CookieJar;
import okhttp3.Headers;
import okhttp3.Interceptor;
import okhttp3.JavaNetCookieJar;
import okhttp3.MediaType;
import okhttp3.MultipartBody;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;
import okhttp3.ResponseBody;
import okio.ByteString;

/**
* Implements the XMLHttpRequest JavaScript interface.
*/
@ReactModule(name = NetworkingModule.NAME)

public final class NetworkingModule extends ReactContextBaseJavaModule {
	private SSLContext getSLLContext () {
		X509TrustManager xtm = new X509TrustManager() {

			@Override
			public void checkClientTrusted(X509Certificate[] chain, String authType) {}

			@Override
			public void checkServerTrusted(X509Certificate[] chain, String authType) {}

			@Override
			public X509Certificate[] getAcceptedIssuers() {
				X509Certificate[] x509Certificates = new X509Certificate[0];
				
				return x509Certificates;
			}
		};

		SSLContext sslContext = null;

		try {

			sslContext = SSLContext.getInstance("SSL");
			sslContext.init(null, new TrustManager[]{xtm}, new SecureRandom());

		} catch (NoSuchAlgorithmException e) {

			e.printStackTrace();
		} catch (KeyManagementException e) {

			e.printStackTrace();
		}

		return sslContext;
	}

	/**
	* Allows to implement a custom fetching process for specific URIs. It is the handler's job
	* to fetch the URI and return the JS body payload.
	*/
	public interface UriHandler {

		/**
		* Returns if the handler should be used for an URI.
		*/
		boolean supports(Uri uri, String responseType);
		
		/**
		* Fetch the URI and return the JS body payload.
		*/
		WritableMap fetch(Uri uri) throws IOException;
	}

	/**
	* Allows adding custom handling to build the {@link RequestBody} from the JS body payload.
	*/
	public interface RequestBodyHandler {

		/**
		* Returns if the handler should be used for a JS body payload.
		*/
		boolean supports(ReadableMap map);

		/**
		* Returns the {@link RequestBody} for the JS body payload.
		*/
		RequestBody toRequestBody(ReadableMap map, String contentType);
	}

	/**
	* Allows adding custom handling to build the JS body payload from the {@link ResponseBody}.
	*/

	public interface ResponseHandler {
		/**
		* Returns if the handler should be used for a response type.
		*/
		boolean supports(String responseType);

		/**
		* Returns the JS body payload for the {@link ResponseBody}.
		*/
		WritableMap toResponseData(ResponseBody body) throws IOException;
	}

	protected static final String NAME = "Networking";

	private static final String CONTENT_ENCODING_HEADER_NAME = "content-encoding";

	private static final String CONTENT_TYPE_HEADER_NAME = "content-type";

	private static final String REQUEST_BODY_KEY_STRING = "string";

	private static final String REQUEST_BODY_KEY_URI = "uri";

	private static final String REQUEST_BODY_KEY_FORMDATA = "formData";

	private static final String REQUEST_BODY_KEY_BASE64 = "base64";

	private static final String USER_AGENT_HEADER_NAME = "user-agent";

	private static final int CHUNK_TIMEOUT_NS = 100 * 1000000; // 100ms

	private static final int MAX_CHUNK_SIZE_BETWEEN_FLUSHES = 8 * 1024; // 8K

	private final OkHttpClient mClient;

	private final ForwardingCookieHandler mCookieHandler;

	private final @Nullable String mDefaultUserAgent;

	private final CookieJarContainer mCookieJarContainer;

	private final Set<Integer> mRequestIds;

	private final List<RequestBodyHandler> mRequestBodyHandlers = new ArrayList<>();

	private final List<UriHandler> mUriHandlers = new ArrayList<>();

	private final List<ResponseHandler> mResponseHandlers = new ArrayList<>();

	private boolean mShuttingDown;

	/* package */

	NetworkingModule(
		ReactApplicationContext reactContext,
		@Nullable String defaultUserAgent,
		OkHttpClient client,
		@Nullable List networkInterceptorCreators
	) {
		super(reactContext);
		if (networkInterceptorCreators != null) {
			OkHttpClient.Builder clientBuilder = client.newBuilder();

			for (NetworkInterceptorCreator networkInterceptorCreator : networkInterceptorCreators) {
				clientBuilder.addNetworkInterceptor(networkInterceptorCreator.create());
			}

			client = clientBuilder.build();
		}

		client = client.newBuilder().hostnameVerifier(new HostnameVerifier() {
			@Override
			public boolean verify(String s, SSLSession sslSession) {
				return true;
			}
		}).sslSocketFactory(getSLLContext().getSocketFactory()).build();

		mClient = client;
		mCookieHandler = new ForwardingCookieHandler(reactContext);
		mCookieJarContainer = (CookieJarContainer) mClient.cookieJar();
		mShuttingDown = false;
		mDefaultUserAgent = defaultUserAgent;
		mRequestIds = new HashSet<>();
	}

	/**
	* @param context the ReactContext of the application
	* @param defaultUserAgent the User-Agent header that will be set for all requests where the
	* caller does not provide one explicitly
	* @param client the {@link OkHttpClient} to be used for networking
	*/

	/* package */
	NetworkingModule(
		ReactApplicationContext context,
		@Nullable String defaultUserAgent,
		OkHttpClient client
	) {
		this(context, defaultUserAgent, client, null);
	}

	/**
	* @param context the ReactContext of the application
	*/
	public NetworkingModule(final ReactApplicationContext context) {
		this(context, null, OkHttpClientProvider.createClient(), null);
	}

	/**
	* @param context the ReactContext of the application
	* @param networkInterceptorCreators list of {@link NetworkInterceptorCreator}'s whose create()
	* methods would be called to attach the interceptors to the client.
	*/
	public NetworkingModule(
		ReactApplicationContext context,
		List networkInterceptorCreators) {
			this(context, null, OkHttpClientProvider.createClient(), networkInterceptorCreators);
		}
	/**
	* @param context the ReactContext of the application
	* @param defaultUserAgent the User-Agent header that will be set for all requests where the
	* caller does not provide one explicitly
	*/

	public NetworkingModule(ReactApplicationContext context, String defaultUserAgent) {
		this(context, defaultUserAgent, OkHttpClientProvider.createClient(), null);
	}

	@Override
	public void initialize() {
		mCookieJarContainer.setCookieJar(new JavaNetCookieJar(mCookieHandler));
	}

	@Override
	public String getName() {
		return NAME;
	}

	@Override
	public void onCatalystInstanceDestroy() {
		mShuttingDown = true;
		cancelAllRequests();
		mCookieHandler.destroy();
		mCookieJarContainer.removeCookieJar();
		mRequestBodyHandlers.clear();
		mResponseHandlers.clear();
		mUriHandlers.clear();
	}

	public void addUriHandler(UriHandler handler) {
		mUriHandlers.add(handler);
	}

	public void addRequestBodyHandler(RequestBodyHandler handler) {
		mRequestBodyHandlers.add(handler);
	}

	public void addResponseHandler(ResponseHandler handler) {
		mResponseHandlers.add(handler);
	}

	public void removeUriHandler(UriHandler handler) {
		mUriHandlers.remove(handler);
	}

	public void removeRequestBodyHandler(RequestBodyHandler handler) {
		mRequestBodyHandlers.remove(handler);
	}

	public void removeResponseHandler(ResponseHandler handler) {
		mResponseHandlers.remove(handler);
	}

	@ReactMethod
	/**
	* @param timeout value of 0 results in no timeout
	*/
	public void sendRequest(
		String method,
		String url,
		final int requestId,
		ReadableArray headers,
		ReadableMap data,
		final String responseType,
		final boolean useIncrementalUpdates,
		int timeout,
		boolean withCredentials
	) {
		final RCTDeviceEventEmitter eventEmitter = getEventEmitter();

		try {
			Uri uri = Uri.parse(url);
			// Check if a handler is registered
			for (UriHandler handler : mUriHandlers) {
				if (handler.supports(uri, responseType)) {
					WritableMap res = handler.fetch(uri);
					ResponseUtil.onDataReceived(eventEmitter, requestId, res);
					ResponseUtil.onRequestSuccess(eventEmitter, requestId);
					return;
				}
			}
		} catch (IOException e) {
			ResponseUtil.onRequestError(eventEmitter, requestId, e.getMessage(), e);
			return;
		}

		Request.Builder requestBuilder;

		try {
			requestBuilder = new Request.Builder().url(url);
		} catch (Exception e) {
			ResponseUtil.onRequestError(eventEmitter, requestId, e.getMessage(), null);
			return;
		}

		if (requestId != 0) {
			requestBuilder.tag(requestId);
		}

		OkHttpClient.Builder clientBuilder = mClient.newBuilder();

		if (!withCredentials) {
			clientBuilder.cookieJar(CookieJar.NO_COOKIES);
		}

		// If JS is listening for progress updates, install a ProgressResponseBody that intercepts the
		// response and counts bytes received.
		if (useIncrementalUpdates) {

			clientBuilder.addNetworkInterceptor(new Interceptor() {
				@Override
				public Response intercept(Interceptor.Chain chain) throws IOException {
					Response originalResponse = chain.proceed(chain.request());
					ProgressResponseBody responseBody = new ProgressResponseBody(
					originalResponse.body(),
					new ProgressListener() {
						long last = System.nanoTime();

						@Override
						public void onProgress(long bytesWritten, long contentLength, boolean done) {
							long now = System.nanoTime();
							if (!done && !shouldDispatch(now, last)) {
								return;
							}

							if (responseType.equals("text")) {
								// For 'text' responses we continuously send response data with progress info to
								// JS below, so no need to do anything here.
								return;
							}

							ResponseUtil.onDataReceivedProgress(
								eventEmitter,
								requestId,
								bytesWritten,
								contentLength
							);

							last = now;
						}
					});
					return originalResponse.newBuilder().body(responseBody).build();
				}
			});
		}

		// If the current timeout does not equal the passed in timeout, we need to clone the existing
		// client and set the timeout explicitly on the clone. This is cheap as everything else is
		// shared under the hood.
		// See https://github.com/square/okhttp/wiki/Recipes#per-call-configuration for more information

		if (timeout != mClient.connectTimeoutMillis()) {
			clientBuilder.readTimeout(timeout, TimeUnit.MILLISECONDS);
		}

		OkHttpClient client = clientBuilder.build();
		Headers requestHeaders = extractHeaders(headers, data);
		if (requestHeaders == null) {
			ResponseUtil.onRequestError(eventEmitter, requestId, "Unrecognized headers format", null);
			return;
		}

		String contentType = requestHeaders.get(CONTENT_TYPE_HEADER_NAME);
		String contentEncoding = requestHeaders.get(CONTENT_ENCODING_HEADER_NAME);
		requestBuilder.headers(requestHeaders);

		// Check if a handler is registered
		RequestBodyHandler handler = null;

		if (data != null) {

			for (RequestBodyHandler curHandler : mRequestBodyHandlers) {
				if (curHandler.supports(data)) {
					handler = curHandler;
					break;
				}
			}
		}

		RequestBody requestBody;
		if (data == null || method.toLowerCase().equals("get") || method.toLowerCase().equals("head")) {
			requestBody = RequestBodyUtil.getEmptyBody(method);
		} else if (handler != null) {
			requestBody = handler.toRequestBody(data, contentType);
		} else if (data.hasKey(REQUEST_BODY_KEY_STRING)) {
			if (contentType == null) {
				ResponseUtil.onRequestError(
					eventEmitter,
					requestId,
					"Payload is set but no content-type header specified",
					null
				);
				return;
			}

			String body = data.getString(REQUEST_BODY_KEY_STRING);
			MediaType contentMediaType = MediaType.parse(contentType);

			if (RequestBodyUtil.isGzipEncoding(contentEncoding)) {
				requestBody = RequestBodyUtil.createGzip(contentMediaType, body);
				if (requestBody == null) {
					ResponseUtil.onRequestError(eventEmitter, requestId, "Failed to gzip request body", null);
					return;
				}
			} else {
				requestBody = RequestBody.create(contentMediaType, body);
			}
		} else if (data.hasKey(REQUEST_BODY_KEY_BASE64)) {

			if (contentType == null) {
				ResponseUtil.onRequestError(
					eventEmitter,
					requestId,
					"Payload is set but no content-type header specified",
					null
				);
				return;
			}

			String base64String = data.getString(REQUEST_BODY_KEY_BASE64);
			MediaType contentMediaType = MediaType.parse(contentType);
			requestBody = RequestBody.create(contentMediaType, ByteString.decodeBase64(base64String));
		} else if (data.hasKey(REQUEST_BODY_KEY_URI)) {
			if (contentType == null) {
				ResponseUtil.onRequestError(
					eventEmitter,
					requestId,
					"Payload is set but no content-type header specified",
					null
				);
				return;
			}

			String uri = data.getString(REQUEST_BODY_KEY_URI);
			InputStream fileInputStream = RequestBodyUtil.getFileInputStream(getReactApplicationContext(), uri);

			if (fileInputStream == null) {
				ResponseUtil.onRequestError(
					eventEmitter,
					requestId,
					"Could not retrieve file for uri " + uri,
					null
				);
				return;
			}

			requestBody = RequestBodyUtil.create(MediaType.parse(contentType), fileInputStream);
		} else if (data.hasKey(REQUEST_BODY_KEY_FORMDATA)) {
			if (contentType == null) {
				contentType = "multipart/form-data";
			}

			ReadableArray parts = data.getArray(REQUEST_BODY_KEY_FORMDATA);
			MultipartBody.Builder multipartBuilder =
			constructMultipartBody(parts, contentType, requestId);

			if (multipartBuilder == null) {
				return;
			}

			requestBody = multipartBuilder.build();

		} else {
			// Nothing in data payload, at least nothing we could understand anyway.
			requestBody = RequestBodyUtil.getEmptyBody(method);
		}

		requestBuilder.method(
			method,
			wrapRequestBodyWithProgressEmitter(requestBody, eventEmitter, requestId)
		);
		addRequest(requestId);
		client.newCall(requestBuilder.build()).enqueue(
			new Callback() {
				@Override
				public void onFailure(Call call, IOException e) {
					if (mShuttingDown) {
						return;
					}
					removeRequest(requestId);

					String errorMessage = e.getMessage() != null
					? e.getMessage()
					: "Error while executing request: " + e.getClass().getSimpleName();

					ResponseUtil.onRequestError(eventEmitter, requestId, errorMessage, e);
				}

				@Override
				public void onResponse(Call call, Response response) throws IOException {
					if (mShuttingDown) {
						return;
					}
					removeRequest(requestId);

					// Before we touch the body send headers to JS
					ResponseUtil.onResponseReceived(
						eventEmitter,
						requestId,
						response.code(),
						translateHeaders(response.headers()),
						response.request().url().toString()
					);

					ResponseBody responseBody = response.body();

					try {
						// Check if a handler is registered
						for (ResponseHandler handler : mResponseHandlers) {

							if (handler.supports(responseType)) {
								WritableMap res = handler.toResponseData(responseBody);
								ResponseUtil.onDataReceived(eventEmitter, requestId, res);
								ResponseUtil.onRequestSuccess(eventEmitter, requestId);
								return;
							}
						}

						// If JS wants progress updates during the download, and it requested a text response,
						// periodically send response data updates to JS.
						if (useIncrementalUpdates && responseType.equals("text")) {
							readWithProgress(eventEmitter, requestId, responseBody);
							ResponseUtil.onRequestSuccess(eventEmitter, requestId);
							return;
						}

						// Otherwise send the data in one big chunk, in the format that JS requested.
						String responseString = "";

						if (responseType.equals("text")) {

							try {
								responseString = responseBody.string();
							} catch (IOException e) {
								if (response.request().method().equalsIgnoreCase("HEAD")) {
									// The request is an `HEAD` and the body is empty,
									// the OkHttp will produce an exception.
									// Ignore the exception to not invalidate the request in the
									// Javascript layer.
									// Introduced to fix issue #7463.
								} else {
									ResponseUtil.onRequestError(eventEmitter, requestId, e.getMessage(), e);
								}
							}
						} else if (responseType.equals("base64")) {
							responseString = Base64.encodeToString(responseBody.bytes(), Base64.NO_WRAP);
						}

						ResponseUtil.onDataReceived(eventEmitter, requestId, responseString);
						ResponseUtil.onRequestSuccess(eventEmitter, requestId);
					} catch (IOException e) {
						ResponseUtil.onRequestError(eventEmitter, requestId, e.getMessage(), e);
					}
				}
			}
		);
	}

	private RequestBody wrapRequestBodyWithProgressEmitter(
		final RequestBody requestBody,
		final RCTDeviceEventEmitter eventEmitter,
		final int requestId
	) {
		if(requestBody == null) {
			return null;
		}
		return RequestBodyUtil.createProgressRequest(
			requestBody,
			new ProgressListener() {
				long last = System.nanoTime();

				@Override
				public void onProgress(long bytesWritten, long contentLength, boolean done) {

					long now = System.nanoTime();

					if (done || shouldDispatch(now, last)) {
						ResponseUtil.onDataSend(eventEmitter, requestId, bytesWritten, contentLength);
						last = now;
					}
				}
			}
		);
	}

	private void readWithProgress(
		RCTDeviceEventEmitter eventEmitter,
		int requestId,
		ResponseBody responseBody
	) throws IOException {
		long totalBytesRead = -1;
		long contentLength = -1;

		try {
			ProgressResponseBody progressResponseBody = (ProgressResponseBody) responseBody;
			totalBytesRead = progressResponseBody.totalBytesRead();
			contentLength = progressResponseBody.contentLength();
		} catch (ClassCastException e) {
			// Ignore
		}

		Charset charset = responseBody.contentType() == null ? StandardCharsets.UTF_8 :
		responseBody.contentType().charset(StandardCharsets.UTF_8);
		ProgressiveStringDecoder streamDecoder = new ProgressiveStringDecoder(charset);
		InputStream inputStream = responseBody.byteStream();

		try {
			byte[] buffer = new byte[MAX_CHUNK_SIZE_BETWEEN_FLUSHES];
			int read;

			while ((read = inputStream.read(buffer)) != -1) {
				ResponseUtil.onIncrementalDataReceived(
					eventEmitter,
					requestId,
					streamDecoder.decodeNext(buffer, read),
					totalBytesRead,
					contentLength
				);
			}
		} finally {
			inputStream.close();
		}

	}

	private static boolean shouldDispatch(long now, long last) {
		return last + CHUNK_TIMEOUT_NS < now;
	}

	private synchronized void addRequest(int requestId) {
		mRequestIds.add(requestId);
	}

	private synchronized void removeRequest(int requestId) {
		mRequestIds.remove(requestId);
	}

	private synchronized void cancelAllRequests() {
		for (Integer requestId : mRequestIds) {
			cancelRequest(requestId);
		}

		mRequestIds.clear();
	}

	private static WritableMap translateHeaders(Headers headers) {
		WritableMap responseHeaders = Arguments.createMap();

		for (int i = 0; i < headers.size(); i++) {
			String headerName = headers.name(i);
			
			// multiple values for the same header
			if (responseHeaders.hasKey(headerName)) {
				responseHeaders.putString(
				headerName,
				responseHeaders.getString(headerName) + ", " + headers.value(i));
			} else {
				responseHeaders.putString(headerName, headers.value(i));
			}
		}

		return responseHeaders;
	}

	@ReactMethod
	public void abortRequest(final int requestId) {
		cancelRequest(requestId);
		removeRequest(requestId);
	}

	private void cancelRequest(final int requestId) {
		// We have to use AsyncTask since this might trigger a NetworkOnMainThreadException, this is an
		// open issue on OkHttp: https://github.com/square/okhttp/issues/869
		new GuardedAsyncTask(getReactApplicationContext()) {
			@Override
			protected void doInBackgroundGuarded(Void... params) {
				OkHttpCallUtil.cancelTag(mClient, Integer.valueOf(requestId));
			}
		}.execute();
	}

	@ReactMethod
	public void clearCookies(com.facebook.react.bridge.Callback callback) {
		mCookieHandler.clearCookies(callback);
	}

	private @Nullable MultipartBody.Builder constructMultipartBody(
		ReadableArray body,
		String contentType,
		int requestId
	) {
		RCTDeviceEventEmitter eventEmitter = getEventEmitter();
		MultipartBody.Builder multipartBuilder = new MultipartBody.Builder();
		multipartBuilder.setType(MediaType.parse(contentType));
		for (int i = 0, size = body.size(); i < size; i++) {
			ReadableMap bodyPart = body.getMap(i);

			// Determine part's content type.
			ReadableArray headersArray = bodyPart.getArray("headers");
			Headers headers = extractHeaders(headersArray, null);
			if (headers == null) {
				ResponseUtil.onRequestError(
					eventEmitter,
					requestId,
					"Missing or invalid header format for FormData part.",
					null
				);

				return null;
			}

			MediaType partContentType = null;
			String partContentTypeStr = headers.get(CONTENT_TYPE_HEADER_NAME);

			if (partContentTypeStr != null) {
				partContentType = MediaType.parse(partContentTypeStr);
				// Remove the content-type header because MultipartBuilder gets it explicitly as an
				// argument and doesn't expect it in the headers array.
				headers = headers.newBuilder().removeAll(CONTENT_TYPE_HEADER_NAME).build();
			}

			if (bodyPart.hasKey(REQUEST_BODY_KEY_STRING)) {
				String bodyValue = bodyPart.getString(REQUEST_BODY_KEY_STRING);
				multipartBuilder.addPart(headers, RequestBody.create(partContentType, bodyValue));
			} else if (bodyPart.hasKey(REQUEST_BODY_KEY_URI)) {
				if (partContentType == null) {
					ResponseUtil.onRequestError(
						eventEmitter,
						requestId,
						"Binary FormData part needs a content-type header.",
						null
					);
					return null;
				}

				String fileContentUriStr = bodyPart.getString(REQUEST_BODY_KEY_URI);
				InputStream fileInputStream = RequestBodyUtil.getFileInputStream(getReactApplicationContext(), fileContentUriStr);

				if (fileInputStream == null) {
					ResponseUtil.onRequestError(
						eventEmitter,
						requestId,
						"Could not retrieve file for uri " + fileContentUriStr,
						null
					);

					return null;
				}

				multipartBuilder.addPart(headers, RequestBodyUtil.create(partContentType, fileInputStream));
			} else {
				ResponseUtil.onRequestError(eventEmitter, requestId, "Unrecognized FormData part.", null);
			}
		}
		return multipartBuilder;
	}

	/**
	* Extracts the headers from the Array. If the format is invalid, this method will return null.
	*/
	private @Nullable Headers extractHeaders(
		@Nullable ReadableArray headersArray,
		@Nullable ReadableMap requestData
	) {
		if (headersArray == null) {
			return null;
		}

		Headers.Builder headersBuilder = new Headers.Builder();

		for (int headersIdx = 0, size = headersArray.size(); headersIdx < size; headersIdx++) {
			ReadableArray header = headersArray.getArray(headersIdx);

			if (header == null || header.size() != 2) {
				return null;
			}

			String headerName = header.getString(0);
			String headerValue = header.getString(1);

			if (headerName == null || headerValue == null) {
				return null;
			}

			headersBuilder.add(headerName, headerValue);
		}

		if (headersBuilder.get(USER_AGENT_HEADER_NAME) == null && mDefaultUserAgent != null) {
			headersBuilder.add(USER_AGENT_HEADER_NAME, mDefaultUserAgent);
		}

		// Sanitize content encoding header, supported only when request specify payload as string
		boolean isGzipSupported = requestData != null && requestData.hasKey(REQUEST_BODY_KEY_STRING);

		if (!isGzipSupported) {
			headersBuilder.removeAll(CONTENT_ENCODING_HEADER_NAME);
		}
		return headersBuilder.build();
	}

	private RCTDeviceEventEmitter getEventEmitter() {
		return getReactApplicationContext().getJSModule(RCTDeviceEventEmitter.class);
	}
}
```


- 在终端代开项目`cd {project}`执行
- `react-native run-android`开始编译，编译的过程很漫长，大概需要10到20分钟
- 如果中途失败记得删除`/node_modules/react-native/ReactAndroid/bulid`里面的所有内容，然后重新编译

- 关于源代码编译可以参考官方文档[中文](https://reactnative.cn/docs/building-from-source.html)或者[英文](https://facebook.github.io/react-native/docs/building-from-source)官网详细查看



- 关于详细代码可进入我的[Github](https://github.com/AaronBank/react-native.git)查看

*感谢支持*