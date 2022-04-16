# CAS简单概念

[参考](https://blog.csdn.net/hejingyuan6/article/details/44277023)

[CAS百度百科](https://baike.baidu.com/item/CAS/1329561?fr=aladdin)

![](https://bkimg.cdn.bcebos.com/pic/aa64034f78f0f736b69eb6580555b319ebc41302?x-bce-process=image/resize,m_lfit,w_934,limit_1/format,f_auto)

1、`TGC`：Ticket-granting cookie，存放用户身份认证凭证的cookie，在浏览器和CAS Server间通讯时使用，是CAS Server用来明确用户身份的凭证。TGT封装了TGC值以及此Cookie值对应的用户信息。

> *存放用户身份认证凭证的cookie，在浏览器和CAS Server间通讯时使用，并且只能基于安全通道传输（Https），是CASServer用来明确用户身份的凭证。*

2、`TGT`：ticket granting ticket，TGT对象的ID就是TGC的值，在服务器端，通过TGC查询TGT。

> *TGT是CAS为用户签发的登录票据，拥有了TGT，用户就可以证明自己在CAS成功登录过。TGT封装了Cookie值以及此Cookie值对应的用户信息。用户在CAS认证成功后，CAS生成cookie（叫TGC），写入浏览器，同时生成一个TGT对象，放入自己的缓存，TGT对象的ID就是cookie的值。当HTTP再次请求到来时，如果传过来的有CAS生成的cookie，则CAS以此cookie值为key查询缓存中有无TGT，如果有的话，则说明用户之前登录过，如果没有，则用户需要重新登录。*

3、`ST`：service ticket，CAS为用户签发的访问某一service的票据，ST是TGT签发的。
4、`PGT`：proxy granting ticket，代理模式下的TGT
5、`PT`：proxy ticket，代理模式下的ST
6、`service`：在cas系统中，接入的各个子系统叫服务。因为对普通用户来说，每一个接入到cas认证中心的子系统都提供特定的服务比如商城、bbs等，大家都听过软件即服务，平台即服务，这样理解service就通顺了
        SaaS：Software-as-a-Service，软件即服务
        PaaS：Platform as a Service，平台即服务
7、`credentials`，凭证，即待认证的用户的信息载体，如用户名+密码
8、`Principal`，当事人，即认证之后返回的已认证的当事人的信息载体，默认只返回用户ID即用户名。
9、`Authentication`表示一个完成的认证请求，当然，结果可能是凭证有效或无效，他有一个method可以获取Principal

# 解决方案

[场景](https://www.cnblogs.com/lichdr/p/10316972.html)

[1](https://www.cnblogs.com/coder-lzh/p/9222467.html)

[小鬼2](https://blog.csdn.net/qq_41258204/article/details/84036875)

```sh
keytool -import -trustcacerts -alias java1234 -file D:/cas/keystore/java1234.cer -keystore "D:/java/jre/lib/security/cacerts"


```

[token解析](https://blog.csdn.net/weixin_41485724/article/details/106164563)

```java
package com.ctsi.cas.util;

import com.ctsi.cas.domain.User;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * Created with Intellij IDEA
 *
 * @Author: liangjy
 * @Date: 2022/03/07/16:46
 * @Description:
 */
public class JwtTokenUtils {
    //用于签名的私钥
    private static final String PRIVATE_KEY = "17CS3Flight";
    //签发者
    private static final String ISS = "Liang";

    //过期时间1小时
    private static final long EXPIRATION_ONE_HOUR = 3600L;
    //过期时间1天
    private static final long EXPIRATION_ONE_DAY = 604800L;

    /**
     * 生成Token
     *
     * @param user
     * @return
     */
    public static String createToken(User user, int expireTimeType) {
        //过期时间
        long expireTime = 0;
        if (expireTimeType == 0) {
            expireTime = EXPIRATION_ONE_HOUR;
        } else {
            expireTime = EXPIRATION_ONE_DAY;
        }

        //Jwt头
        Map<String, Object> header = new HashMap<>();
        header.put("typ", "JWT");
        header.put("alg", "HS256");
        Map<String, Object> claims = new HashMap<>();
        //自定义有效载荷部分
        // claims.put("id",user.getId());
        claims.put("username", user.getUsername());
        // claims.put("password",user.getPassword());

        return Jwts.builder()
                //发证人
                .setIssuer(ISS)
                //Jwt头
                .setHeader(header)
                //有效载荷
                .setClaims(claims)
                //设定签发时间
                .setIssuedAt(new Date())
                //设定过期时间
                .setExpiration(new Date(System.currentTimeMillis() + expireTime * 1000))
                //使用HS256算法签名，PRIVATE_KEY为签名密钥
                .signWith(SignatureAlgorithm.HS256, PRIVATE_KEY)
                .compact();
    }

    /**
     * 验证Token,返回数据为含有username的User对象
     *
     * @param token
     * @return
     */
    public static User checkToken(String token) {
        //解析token后，从有效载荷取出值
        String account = (String) getClaimsFromToken(token).get("username");
        User user = new User();
        user.setUsername(account);
        return user;
    }

    /**
     * 获取有效载荷
     *
     * @param token
     * @return
     */
    public static Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    //设定解密私钥
                    .setSigningKey(PRIVATE_KEY)
                    //传入Token
                    .parseClaimsJws(token)
                    //获取载荷类
                    .getBody();
        } catch (Exception e) {
            return null;
        }
        return claims;
    }

}

```

```
 if (token == null || token.isEmpty()) {
            throw new AccountException("sorry , token not be null !!");
        }
        //解析token获取username
        User user = JwtTokenUtils.checkToken(token);
        //  校验token 并获取username 暂时测试用admin
        String username = user.getUsername();
```

```java
package com.ctsi.cas.util;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLConnection;
import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

/**
 * Created by Administrator on 2021/7/14.
 */

public class HttpUtils {
    private static final Logger log = LoggerFactory.getLogger(HttpUtils.class);

    public static String post(String url, Map<String,String> header, JSONObject json){
        String result = "";
        HttpPost post = new HttpPost(url);
        try{
            CloseableHttpClient httpClient = HttpClients.createDefault();
            if(header != null){
                header.forEach((k,v)->{
                    post.setHeader(k,v);
                });
            }
            StringEntity postingString = new StringEntity(JSON.toJSONString(json),"utf-8");
            post.setEntity(postingString);
            HttpResponse response = httpClient.execute(post);

            InputStream in = response.getEntity().getContent();
            BufferedReader br = new BufferedReader(new InputStreamReader(in, "utf-8"));
            StringBuilder strber= new StringBuilder();
            String line = null;
            while((line = br.readLine())!=null){
                strber.append(line+'\n');
            }
            br.close();
            in.close();
            result = strber.toString();
//            if(response.getStatusLine().getStatusCode()!= HttpStatus.SC_OK){
//                result = "服务器异常";
//            }
        } catch (Exception e){
            System.out.println("请求异常");
            throw new RuntimeException(e);
        } finally{
            post.abort();
        }
        return result;
    }

    public static String get(String destUrl,String requestBody) throws IOException {
        StringBuffer requestBodyBuf = new StringBuffer(requestBody);
        StringBuffer responseBody = new StringBuffer();
        request("GET",destUrl,new HashMap(),requestBodyBuf,new HashMap(),responseBody);

        return responseBody.toString();
    }

    // 用于发送HTTP请求
    public static void request(String method, String destUrl,
                               Map requestHeaderMap, StringBuffer requestBody,
                               Map responseHeaderMap, StringBuffer responseBody)
            throws IOException {

        log.info("/**************** " + method + ": " + destUrl
                + " ******/");
        log.info(method + ": " + destUrl);
        // 建立连接
        URL url = new URL(destUrl);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod(method);
        conn.setUseCaches(false);
        conn.setDoInput(true);
        conn.setDoOutput(true);

        // 添加请求头
        setRequestProperty(conn, requestHeaderMap);

        // 发送请求
        String reqStr = requestBody.toString();
        byte bufb[] = reqStr.getBytes();
        DataOutputStream buffout = new DataOutputStream(conn.getOutputStream());
        buffout.write(bufb);
        buffout.flush();
        log.info(reqStr);

        // 读取响应
        // conn.getHeaderFields();
        DataInputStream dis = new DataInputStream(conn.getInputStream());
        byte[] buffer = new byte[1024];
        int length = -1;
        String respBodyStr = "";
        while ((length = dis.read(buffer)) != -1) {
            respBodyStr += new String(buffer, 0, length);
        }
        getHeaderMap(conn, responseHeaderMap);

        // 返回响应体
        responseBody.append(respBodyStr);
        log.info(respBodyStr);

        log.info("/**************** END OF " + method + ": " + destUrl
                + " ******/\r\n");
    }

    /**
     * 向指定 URL 发送POST方法的请求
     *
     * @param url
     *            发送请求的 URL
     * @param param
     *            请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
     * @return 所代表远程资源的响应结果
     */
    public static String sendPost(String url, String param) {
        PrintWriter out = null;
        BufferedReader in = null;
        String result = "";
        try {
            URL realUrl = new URL(url);
            // 打开和URL之间的连接
            URLConnection conn = realUrl.openConnection();
            // 设置通用的请求属性
            conn.setRequestProperty("accept", "*/*");
            conn.setRequestProperty("connection", "Keep-Alive");
            conn.setRequestProperty("user-agent",
                    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            // 发送POST请求必须设置如下两行
            conn.setDoOutput(true);
            conn.setDoInput(true);
            // 获取URLConnection对象对应的输出流
            out = new PrintWriter(conn.getOutputStream());
            // 发送请求参数
            out.print(param);
            // flush输出流的缓冲
            out.flush();
            // 定义BufferedReader输入流来读取URL的响应
            in = new BufferedReader(
                    new InputStreamReader(conn.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
        } catch (Exception e) {
            System.out.println("发送 POST 请求出现异常！"+e);
            e.printStackTrace();
        }
        //使用finally块来关闭输出流、输入流
        finally{
            try{
                if(out!=null){
                    out.close();
                }
                if(in!=null){
                    in.close();
                }
            }
            catch(IOException ex){
                ex.printStackTrace();
            }
        }
        return result;
    }

    @SuppressWarnings("rawtypes")
    private static void setRequestProperty(HttpURLConnection conn, Map headerMap) {
        for (Iterator it = headerMap.entrySet().iterator(); it.hasNext();) {
            Map.Entry entry = (Map.Entry) it.next();
            String key = (String) entry.getKey();
            String value = (String) entry.getValue();
            conn.setRequestProperty(key, value);
            log.info(key + ": " + value);
        }
    }

    @SuppressWarnings("unchecked")
    private static void getHeaderMap(HttpURLConnection conn, Map headerMap) {
        for (Iterator it = conn.getHeaderFields().entrySet().iterator(); it
                .hasNext();) {
            Map.Entry entry = (Map.Entry) it.next();
            Object key = entry.getKey();
            Object value = entry.getValue();
            headerMap.put(key, value);
            log.info(key + ": " + value);
        }
    }
    //将map型转为请求参数型
    public static String urlencode(Map<String,Object>data) {
        StringBuilder sb = new StringBuilder();
        for (Map.Entry i : data.entrySet()) {
            try {
                sb.append(i.getKey()).append("=").append(URLEncoder.encode(i.getValue()+"","UTF-8")).append("&");
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }
        return sb.toString();
    }
}

```

