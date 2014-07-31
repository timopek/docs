##7. 数据库初始化

1. 初始化数据库
        
    curl --user "webmaster:1234567890" http://10.0.19.17:8080/system/database/setup    
    Response:
    {
      "action" : "cassandra setup",
      "status" : "ok",
      "timestamp" : 1404892488508,
      "duration" : 119
    }
    
    curl --user "webmaster:1234567890" http://10.0.19.17:8080/system/superuser/setup
    Response:
    {
       "action" : "superuser setup",
       "status" : "ok",
       "timestamp" : 1404892513166,
       "duration" : 3
    }
    

##8. usergrid初始化

1 创建“weiquan" org，并同时为这个org创建一个管理员。管理员的用户名为"admin"， 其注册邮件地址是admin@easemob.com (用于找回密码)，密码为"1234567890".   

    curl -X POST -i  "http://localhost:8080/management/organizations" -d '{"organization":"weiquan","username":"admin","name":"admin","email":"admin@easemob.com","password":"1234567890"}'
    
    Response:
    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    Set-Cookie: rememberMe=deleteMe; Path=/; Max-Age=0; Expires=Tue, 08-Jul-2014 08:01:37 GMT
    Access-Control-Allow-Origin: *
    Vary: Accept-Encoding
    Content-Type: application/json
    Transfer-Encoding: chunked
    Date: Wed, 09 Jul 2014 08:01:38 GMT
    {
      "action" : "new organization",
      "status" : "ok",
      "data" : {
         "owner" : {
            "applicationId" : "00000000-0000-0000-0000-000000000001",
            "username" : "admin",
            "name" : "admin",
            "email" : "admin@easemob.com",
            "activated" : true,
            "confirmed" : true,
            "disabled" : false,
            "properties" : { },
            "uuid" : "402d444a-073f-11e4-bfb3-a7b49c44b7ee",
            "adminUser" : true,
            "displayEmailAddress" : "admin <admin@easemob.com>",
            "htmldisplayEmailAddress" : "admin &lt;<a href=\"mailto:admin@easemob.com\">admin@easemob.com</a>&gt;"
         },
         "organization" : {
         "name" : "weiquan",
         "properties" : null,
         "uuid" : "4068517a-073f-11e4-8b5f-bfd0e8dee368",
         "passwordHistorySize" : 0
         }
      },
      "timestamp" : 1404892896859,
      "duration" : 1454
    }

这一步会遇到一个问题：
    
    HTTP/1.1 500 Internal Server Error
    Server: Apache-Coyote/1.1
    Access-Control-Allow-Origin: *
    Vary: Accept-Encoding
    Content-Type: application/json
    Transfer-Encoding: chunked
    Date: Wed, 09 Jul 2014 07:37:15 GMT
    Connection: close
    {"error":"h_unavailable","timestamp":1404891435964,"duration":19,"exception":"me.prettyprint.hector.api.exceptions.HUnavailableException","error_description":": May not be enough replicas present to handle consistency level."}

解决方案：
    
    1. stop掉cassandra和tomcat, 先删除/home/easemob/data/cassandra/*     
       supervisorctl start tomcat 
       supervisorctl start cassandra
    2. 修改 /home/easemob/apps/opt/tomcat/lib/usergrid-custom.properties, 下面的配置是单机部署的配置。
    
    cassandra.readcl=ONE    
    cassandra.writecl=ONE
    cassandra.keyspace.replication=1
    cassandra.keyspace.strategy=org.apache.cassandra.locator.SimpleStrategy
    
    3. 再次运行创建组织的命令

激活账号

curl -X PUT -i -H "Authorization: Bearer YWMt39RfMMOqEeKYE_GW7tu81AAAAT71lGijyjG4VUIC2AwZGzUjVbPp_4qRD5k" "https://a1.easemob.com/easemob-demo/chatdemo/users" -d '{"activated":true,"":true}'
	 
2 用#1创建的org admin账号登陆。
    
    curl -X POST -i  "http://localhost:8080/management/token" -d '{"grant_type":"password","username":"admin","password":"1234567890"}'
    Response:
    {"passwordChanged":1404892896984,"access_token":"YWMtVZ-vSAc_EeSKo1GT2ad0bQAAAUc-MW8SdIKgibYlPgPDgD3oDTSeJoNX8j0","expires_in":604800,"user":{"organizations":{"weiquan":{"users":{"admin":{"applicationId":"00000000-0000-0000-0000-000000000001","username":"admin","name":"admin","email":"admin@easemob.com","activated":true,"confirmed":true,"disabled":false,"properties":{},"uuid":"402d444a-073f-11e4-bfb3-a7b49c44b7ee","adminUser":true,"displayEmailAddress":"admin <admin@easemob.com>","htmldisplayEmailAddress":"admin &lt;<a href=\"mailto:admin@easemob.com\">admin@easemob.com</a>&gt;"}},"name":"weiquan","applications":{"weiquan/sandbox":"40a162d0-073f-11e4-ac07-45ccca4cb991"},"properties":{},"uuid":"4068517a-073f-11e4-8b5f-bfd0e8dee368"}},"applicationId":"00000000-0000-0000-0000-000000000001","properties":{},"htmldisplayEmailAddress":"admin &lt;<a href=\"mailto:admin@easemob.com\">admin@easemob.com</a>&gt;","username":"admin","confirmed":true,"email":"admin@easemob.com","adminUser":true,"name":"admin","activated":true,"uuid":"402d444a-073f-11e4-bfb3-a7b49c44b7ee","displayEmailAddress":"admin <admin@easemob.com>","disabled":false}}

3 创建weiquan_metadata （weiquan_metadata是一个特殊的app）. 注意: Bearer后面的token要用#2步返回的access_token的值.
    
    curl -X POST -i -H "Authorization: Bearer YWMtVZ-vSAc_EeSKo1GT2ad0bQAAAUc-MW8SdIKgibYlPgPDgD3oDTSeJoNX8j0" "http://localhost:8080/management/organizations/weiquan/applications" -d '{"name":"weiquan_metadata"}'
     
    Response:
    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    Set-Cookie: rememberMe=deleteMe; Path=/; Max-Age=0; Expires=Tue, 08-Jul-2014 08:03:00 GMT
    Access-Control-Allow-Origin: *
    Vary: Accept-Encoding
    Content-Type: application/json
    Transfer-Encoding: chunked
    Date: Wed, 09 Jul 2014 08:03:00 GMT
    {
       "action" : "new application for organization",
       "uri" : "http://10.0.19.17/null/null",
       "entities" : [ {
       "uuid" : "71d201c0-073f-11e4-9781-0726f6a46ee3",
       "type" : "application",
       "name" : "weiquan/weiquan_metadata",
       "created" : 1404892980198,
       "modified" : 1404892980198,
       "applicationName" : "weiquan_metadata",
       "organizationName" : "weiquan",
       "allow_open_registration" : true,
       "registration_requires_email_confirmation" : false,
       "registration_requires_admin_approval" : false,
       "notify_admin_of_new_users" : false
       } ],
    "data" : {
       "weiquan/weiquan_metadata" : "71d201c0-073f-11e4-9781-0726f6a46ee3"
    },
    "timestamp" : 1404892980183,
    "duration" : 655
    }

4 获得"weiquan"这个org的credential。这个credential会在#5中用到  注意: Bearer后面的token要用#2步返回的access_token的值.
    
    curl -X GET  -H "Authorization: Bearer YWMtVZ-vSAc_EeSKo1GT2ad0bQAAAUc-MW8SdIKgibYlPgPDgD3oDTSeJoNX8j0" "http://localhost:8080/management/organizations/weiquan/credentials"
    Response:
    {
       "action" : "get organization client credentials",
       "timestamp" : 1404893031648,
       "duration" : 3,
       "credentials" : {
       "client_id" : "b3U6QGhRegc_EeSLX7_Q6N7jaA",
       "client_secret" : "b3U6xrxDDabvOia_OKpKuTa3tsqVdVc"
       }
    }

5 修改Cloudcode代码
修改这个文件(https://github.com/jervisliu/qixin-mgmt/blob/master/qx/src/main/java/com/easemob/qixin/utils/ConstantUtils.java), 用#4得到的credential来设置DEFAULT_ORG_CREDENTIAL： 

    public static String EASEMOB_API_HOST = "10.0.19.18:8080";   // Usergrid服务器
    public static String EASEMOB_Cloudcode_HOST = "10.0.19.18:5222";  // IM服务器
    public static String EASEMOB_CHAT_URL = "10.0.19.18:8081";   // RESTAPI服务器
    public static final Credential DEFAULT_ORG_CREDENTIAL = new ClientCredential("b3U6QGhRegc_EeSLX7_Q6N7jaA", "b3U6xrxDDabvOia_OKpKuTa3tsqVdVc");


6 创建一个company
    
    curl -X POST  -H "Authorization: Bearer YWMtVZ-vSAc_EeSKo1GT2ad0bQAAAUc-MW8SdIKgibYlPgPDgD3oDTSeJoNX8j0" "http://localhost:8080/weiquan/weiquan_metadata/companies" -d '{"name": "smt","appname": "smt"}'
    Response:
    {
      "action" : "post",
      "application" : "71d201c0-073f-11e4-9781-0726f6a46ee3",
      "params" : { },
      "path" : "/companies",
      "uri" : "http://10.0.19.17/weiquan/weiquan_metadata/companies",
      "entities" : [ {
      "uuid" : "aac8adda-073f-11e4-ab65-f34af92328da",
      "type" : "company",
      "name" : "smt",
      "created" : 1404893075757,
      "modified" : 1404893075757,
      "appname" : "smt"
      } ],
      "timestamp" : 1404893075720,
      "duration" : 122,
      "organization" : "weiquan",
      "applicationName" : "weiquan_metadata"
    }


7 创建一个app

    curl -X POST  -H "Authorization: Bearer YWMtVZ-vSAc_EeSKo1GT2ad0bQAAAUc-MW8SdIKgibYlPgPDgD3oDTSeJoNX8j0" "http://localhost:8080/management/organizations/weiquan/applications" -d '{"name": "smt"}'       Response:
    {
      "action" : "new application for organization",
      "uri" : "http://10.0.19.17/null/null",
      "entities" : [ {
      "uuid" : "cdb25cb0-073f-11e4-b387-d1ceaf6aedee",
      "type" : "application",
      "name" : "weiquan/smt",
      "created" : 1404893134339,
      "modified" : 1404893134339,
      "applicationName" : "smt",
      "organizationName" : "weiquan",
      "allow_open_registration" : true,
      "registration_requires_email_confirmation" : false,
      "registration_requires_admin_approval" : false,
      "notify_admin_of_new_users" : false
    } ],
    "data" : {
        "weiquan/smt" : "cdb25cb0-073f-11e4-b387-d1ceaf6aedee"
     },
    "timestamp" : 1404893134329,
    "duration" : 542
     }


