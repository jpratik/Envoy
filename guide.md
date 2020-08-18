# ENVOY Developer Guide

Welcome to the envoy developer guide.
These document will teach you how to build Android apps using 
APIs in the Android framework and other libraries.

# All About Envoy
We have built robust, unique, censorship-defeating tools and services that are having a big impact in some country where some website and their content are censored

This tool is derived from [Cronet](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/). 

[Cronet](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/) is the networking stack of Chromium put into a library for use on mobile. This is the same networking stack that is used in chrome browser by over a billion user.

So it will work with all request type which is mostly used in developement.
It is used to make app which is resistant to **censorship**

## What is Envoy And Where It can be used

As name suggest envoy is a **representative** or  **messenger**

Like it our tool will work as messenger or representative for certain website or web content access without worrying of censorship as our tool will work as representative.

As earlier said it can be used to make app resistant to  **censorship**.

Also it can be used as proxy tool communication for secure web access to your app.
E.g. if facebook and youtube is not accessible to country like china and you want to make app which shows facebook content or youtube content then you can use this tool to show your content.

It can be used as partial content showing or can be used to make whole server communication with this tool depending upon your requirement.

Also if you don't want to use envoy proxy then also you can use this tool to get benifit of cronet library hasselfree. And in future if you want then you just have to simply define envoy url to bypass censorship.

## What is Envoy URL

In its simplest form, an envoy URL is just an HTTP(s) URL, such as "https://allowed.example.com/app1/". But envoy also accepts a special URL scheme in this form: "envoy://k1=v1[&kn=vn]?" which includes more parameters.

In one format of https://DOMAIN/PATH or envoy://?k1=v1&k2=v2

### Parameters

keys for `envoy://?k1=v1&k2=v2` format:

* url: proxy URL, for example, https://allowed.example.com/path/
* header_xxx: HTTP header, header_Host=my-host` will send Host header with value my-host
* address: IP address for domain in proxy URL, to replace IP from DNS resolving
* resolve: resolve map, same as `--host-resolver-rules` command line for chromium, [Chromium docs](https://www.chromium.org/developers/design-documents/network-stack/socks-proxy), [lighthouse issue #2817](https://github.com/GoogleChrome/lighthouse/issues/2817), [firefox bug #1523367](https://bugzilla.mozilla.org/show_bug.cgi?id=1523367)
* disable_cipher_suites: cipher suites list to disable, same as `--cipher-suite-blacklist` command line, [chromium bug #58831](https://bugs.chromium.org/p/chromium/issues/detail?id=58831), [Firefox Support Forum](https://support.mozilla.org/en-US/questions/1119007#answer-867850)
* salt: a 16 characters long random string, unique to each combination of app-signing key, user, and device, such [ANDROID_ID}(https://developer.android.com/reference/android/provider/Settings.Secure.html#ANDROID_ID).

### Examples

1. a simple URL: `https://allowed.example.com/app1/`
2. set host:`envoy://?url=https%3A%2F%2Fexample.com%2Fapp1%2F%3Fk1%3Dv1&header_Host=forbidden.example.com`
3. only MAP url-host to address: `envoy://?url=https%3A%2F%2Fexample.com%2Fapp1%2F%3Fk1%3Dv1&header_Host=forbidden.example.com&address=1.2.3.4`
4. custom host override: `envoy://?url=https%3A%2F%2Fexample.com%2Fapp1%2F%3Fk1%3Dv1&header_Host=forbidden.example.com&resolve=MAP%20example.com%201.2.3.4`
5. disable some cipher suites:  `envoy://?url=https%3A%2F%2Fallowed.example.com%2Fapp1%2F%3Fk1%3Dv1&header_Host=forbidden.example.com&address=1.2.3.4&disabled_cipher_suites=0xc024,0xc02f`

In example 5: allowed.example.com will be TLS SNI, forbidden.example.com will be Host HTTP header, 1.2.3.4 will be IP for allowed.example.com.

The equivalent curl command(see below for nginx conf):

`curl --resolve allowed.example.com:443:1.2.3.4 \
      --header 'Host: forbidden.example.com' \
      --header 'Url-Orig: https://forbidden.example.com' --header 'Host-Orig: forbidden.example.com' \
      https://allowed.example.com/app1/ # --ciphers ECDHE-RSA-AES128-GCM-SHA256 `

Note: _hash=HASH` will be appended to url in all cases for cache ~~invalidation~~.

## Setup 

### Backend

Use Nginx as a reverse proxy

If you don't have any idea how to use Nginx as a reverse proxy then you can check this documentation for more info.
https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/

Proxying is typically used to distribute the load among several servers, seamlessly show content from different websites, or pass requests for processing to application servers over protocols other than HTTP.

When NGINX proxies a request, it sends the request to a specified proxied server, fetches the response, and sends it back to the client. It is possible to proxy requests to an HTTP server (another NGINX server or any other server) or a non-HTTP server (which can run an application developed with a specific framework, such as PHP or Python) using a specified protocol. Supported protocols include FastCGI, uwsgi, SCGI, and memcached.

To pass a request to an HTTP proxied server, the proxy_pass directive is specified inside a location. For example:

```
location /some/path/ {
    proxy_pass http://www.example.com/link/;
}
```
While for our project we will setup our Nginx like below.Also You can configure as your own as per your requirement.

```
location ~ ^/app1/ {
    proxy_ssl_server_name on;
    proxy_pass $http_url_orig;
    proxy_buffering off; # disable buffer for stream
    proxy_set_header Host $http_host_orig;
    proxy_hide_header Host-Orig;
    proxy_hide_header Url-Orig;
    proxy_pass_request_headers on;
}
```
### Native
[sample main.cc](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/native/sample/main.cc)

```bash
# ... after patching chromium source
export ENVOY_URL=https://example.com/path/
sed -i s#https://example.com/enovy_path/#$ENVOY_URL#g components/cronet/native/sample/main.cc
autoninja -C out/Cronet-Desktop cronet_sample cronet_sample_test
out/Cronet-Desktop/cronet_sample https://ifconfig.co/ip
```

### Android

[CronetSampleActivity.java](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/android/sample/src/org/chromium/cronet_sample_apk/CronetSampleActivity.java)

```bash
# ... after patching chromium source
export ENVOY_URL=https://example.com/path/
sed -i s#https://example.com/enovy_path/#$ENVOY_URL#g components/cronet/android/sample/src/org/chromium/cronet_sample_apk/CronetSampleActivity.java
autoninja -C out/Cronet cronet_sample_apk
adb install out/Cronet/apks/CronetSample.apk
```




# Android Project Setup

## Download

- [cronet-release.aar](https://envoy.greatfire.org/static/cronet-release.aar)
- [cronet-debug.aar](https://envoy.greatfire.org/static/cronet-debug.aar)

## Build

Copy `cronet-$BUILD.aar`(debug and release) to `envoy-master/android/cronet/`, then run `./gradlew assembleDebug` or `./gradlew assemble` to build the project.

## Get Started

Envoy has only one more extra API call than Google's [chromium](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/)/android [cronet library](https://developer.android.com/guide/topics/connectivity/cronet): `CronetEngine.Builder.setEnvoyUrl` .

Build the demo module to see it in action, or just call this in `Activity`'s `onCreate` method:

```java
CronetNetworking.initializeCronetEngine(getApplicationContext(), ""); // set envoy url here
```

## Use it as simple Cronet call 

```java
CronetEngine.Builder engineBuilder = new CronetEngine.Builder(view.getContext());
engineBuilder.setUserAgent("curl/7.66.0");
engineBuilder.setEnvoyUrl("");//set envoy url here

CronetEngine engine = engineBuilder.build();
........
```

## Use it with  HttpURLConnection

```java
CronetEngine.Builder engineBuilder = new CronetEngine.Builder(HttpURLConnectionFragment.this.getActivity());
engineBuilder.setEnvoyUrl("");//set envoy url here

CronetEngine engine = engineBuilder.build();
........
```
## Use it with  OkHttp
```java
CronetEngine.Builder engineBuilder = new CronetEngine.Builder(OkHttpFragment.this.getActivity());
engineBuilder.setEnvoyUrl("");//set envoy url here

CronetEngine engine = engineBuilder.build();

OkHttpClient client = new OkHttpClient.Builder()
        .addInterceptor(new CronetInterceptor(engine))
        .build();
........
```
## Use it  with Volley
```java
static class CronetStack extends HurlStack {
    private final CronetEngine engine;

    public CronetStack(Context context) {
        this("", context);
    }

    CronetStack(String envoyUrl, Context context) {
        CronetEngine.Builder engineBuilder = new 		CronetEngine.Builder(context);
        if (envoyUrl != null && !envoyUrl.isEmpty()) {
            engineBuilder.setEnvoyUrl(envoyUrl);
        }
        engine = engineBuilder.build();
    }

    @Override
    protected HttpURLConnection createConnection(URL url) throws IOException {
        return (HttpURLConnection) engine.openConnection(url);
    }
}

class VolleyRequestTask extends AsyncTask<String, String, String> {
    
	private final WeakReference<Context> contextRef;

    VolleyRequestTask(WeakReference<Context> contextRef) {
        this.contextRef = contextRef;
    }

    @Override
    protected String doInBackground(String... uri) {
        if (this.contextRef.get() == null) {
            return null;
        }

        RequestQueue queue = Volley.newRequestQueue(this.contextRef.get(), new CronetStack("", this.contextRef.get()));//set envoy url here

........
```
## Use it with Webview
```java
CronetEngine.Builder engineBuilder = new CronetEngine.Builder(view.getContext());
engineBuildersetEnvoyUrl("");//set envoy url here

CronetEngine engine = engineBuilder.build();
WebView webv;
webv.getSettings().setJavaScriptEnabled(true);
webv.setWebViewClient(new CronetWebViewClient(engine){
                @Override
                public void onPageFinished(WebView view, String url) {
                    super.onPageFinished(view, url);
                    .......                
                    }
            });
........
```
## Other Guide

If you are more interested to know more about how to use cronet then you can check the below link

1. [Quick Start Guide to Using Cronet](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/README.md), [native API](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/native/test_instructions.md), [Android API](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/android/test_instructions.md)
2. [Perform network operations using Cronet](https://developer.android.com/guide/topics/connectivity/cronet)
3. Samples with test: [native sample](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/native/sample), [Android sample](https://chromium.googlesource.com/chromium/src/+/master/components/cronet/android/sample/README), [GoogleChromeLabs / cronet_sample](https://github.com/GoogleChromeLabs/cronet-sample/blob/master/android/app/src/main/java/com/google/samples/cronet_sample/ViewAdapter.java#L80)

## FAQ

### library strip error
```
Task :app:stripDevDebugDebugSymbols
/Users/xxx/Library/Android/sdk/ndk-bundle/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-strip:/Users/me/Downloads/apps-android-wikipedia/app/build/intermediates/merged_native_libs/devDebug/out/lib/armeabi-v7a/libcronet.72.0.3626.122.so: File format not recognized

Unable to strip library /Users/xxx/apps-android-wikipedia/app/build/intermediates/merged_native_libs/devDebug/out/lib/armeabi-v7a/libcronet.72.0.3626.122.so due to error 1 returned from /Users/me/Library/Android/sdk/ndk-bundle/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-strip, packaging it as is
```

configure module build.gradle:
```
defaultConfig {
    packagingOptions {
        doNotStrip '**/libcronet*.so'
    }
}
```

## App examples

Below you can find the project link of wikipedia app

1. [Wikipedia](https://github.com/wikimedia/apps-android-wikipedia): `./gradlew clean assembleDevDebug`, 
and you can make this project with our tool to bypass censorship which apk you can find it here [demo apk](https://envoy.greatfire.org/static/wikipedia-prod.apk), and the migration guide for it how to use it here. [guide](apps/wikipedia.md).

2. [DuckDuckGo](https://github.com/duckduckgo/Android): `./gradlew assembleDebug`, [demo apk](https://envoy.greatfire.org/static/duckduckgo-5.41.0-debug.apk)

3. Wordpress:
   1. [WordPress-FluxC-Android](https://github.com/wordpress-mobile/WordPress-FluxC-Android): `echo "sdk.dir=YOUR_SDK_DIR" > local.properties && ./gradlew fluxc:build`
   2. [WordPress-Android](https://github.com/wordpress-mobile/WordPress-Android): set `wp.oauth.app_id` and `wp.oauth.app_secret`, then `cp gradle.properties-example gradle.properties && ./gradlew assembleVanillaDebug`
   
 You can submit more apps with `git -c diff.noprefix=false format-patch --numbered --binary HEAD~`.

## Release steps
1. Rebuild cronet-debug.aar and cronet-release.aar: run `./native/build_cronet.sh debug` and `./native/build_cronet.sh release`
2. Rebuild envoy: ./android/build-envoy.sh
3. Rebuild demo apps: ./apps/build-apps.sh

## History
1. [Google to reimplement curl in libcrurl | daniel.haxx.se](https://daniel.haxx.se/blog/2019/06/19/google-to-reimplement-curl-in-libcrurl/), [谷歌想实现自己的 curl，为什么？](https://www.oschina.net/news/107711/google-to-reimplement-curl-in-libcrurl)
2. [973603 - [Cronet] libcurl wrapper library using Cronet API - chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=973603)
