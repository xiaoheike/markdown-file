## ��ṹ ##
t_bussiness_type:�¼����ͣ����硰��½�������쳣�������Զ����¼����ȵ�
t_bussiness_type_table_field:��t_bussiness_typeΪ���һ��ϵ�������ֶ�����ά��ĳЩ��ϵ�����岢��֪����ʲô��
t_bussiness_type_table:��t_bussiness_typeΪ���һ��ϵ�����С�table_name���ֶ�ָ��һ���¼�������Ҫ������Щ������Login�¼�����Ҫ��t_collection_action��t_app_usage��д�����ݡ�
t_app_usage:�û���ÿ��LOGIN��������д�����ű�
t_collection_action:�û���ÿ��LOGIN��������д�����ű�
t_app_usage��t_collection_action��һ��һ��ϵ��ǰ�ߵ�identifier�ֶ�����ߵ�buss_idһһ��Ӧ��
����collection���̽ӿڻᴫ��t_bussiness_type��ͨ�������Ի����������������ݣ��õ�Ҫ������д��ı�
�ӳ����Ͽ�����������֮���Ǵ��������ϵ�������������ݿ���û�п�����z�����ȴ������ݿ��еı�֮�������г����µģ�
t_sqool_ins_fields:���ĳЩ����ֶΣ���ŷ�ʽ������+�ֶ����������ֶ�������ins_fields�����֡�
## д��flume ##
`FlumeAdapterServiceImpl`���з���`saveData()`�Ǻ��ģ�����߼����ڸô�����ʵ�֡�
`BaseAdapterService`�з���`dataGroupBybussinesType`������json��ʽ��������`bussinessType`���飬������ݣ�
```JAVA
{LOGIN=[com.nd.esp.cloudaltas.business.collect.domainmodel.request.CollectInfoRequestBean@1d317535]}
```
`MysqlTableStructMangerImpl`���з���`getTableStructBean`��ñ�Ľṹ�������Լ��������ݣ�����������£�
```JAVA
{"tableName":"T_COLLECT_ACTION","metaData":{"bussiness_type":"java.lang.String","insert_time":"java.sql.Timestamp","buss_id":"java.lang.String","ext_properties":"java.lang.String","app_key":"java.lang.String","identifier":"java.lang.String"},"fieldNames":["bussiness_type","insert_time","buss_id","ext_properties","app_key","identifier"]}
```
`SqoolInsFieldsServiceImpl`���з���`getFields`��ö�Ӧ��������ֶ����ƣ�ʵ����Ӧ���Ƕ�ȡt_sqool_ins_fields���е����ݣ�������ݣ�
```JAVA
[identifier, app_key, buss_id, bussiness_type, ext_properties, insert_time]
```
`AdapterCommonServiceImpl`��`buildDynamicBeans()`�������ؽ����
```JAVA
{T_COLLECT_ACTION={"tableName":"T_COLLECT_ACTION","metaData":{},"rowSize":1,"fieldNames":[]}, T_APP_USAGE={"tableName":"T_APP_USAGE","metaData":{},"rowSize":1,"fieldNames":[]}}
```
���ص�ֵ���Ѿ�����post��������ݣ����磺`[{createTime=2016-04-07T16:50:50.336+0800, session_id=ae45821d-5413-4921-84d2-ef22b07ef954, bussinessType=login, accountId=123456789, function_id=21, appVer=2016-04-07 16:33:33, account_id=123456789, bussiness_type=login, buss_id=b9d814ec-3b63-43fc-86ad-ba0e5e589c41, ext_properties=null, device_id=cyjcd08c754dd4c, app_key=sso, extProperties=null, sessionId=ae45821d-5413-4921-84d2-ef22b07ef954, create_time=2016-04-07T16:50:50.336+0800, functionId=21, identifier=b9d814ec-3b63-43fc-86ad-ba0e5e589c41, deviceId=cyjcd08c754dd4c, app_ver=2016-04-07 16:33:33}]` ����߰���������ȫһ�������ݣ�ֻ��keyֵ�ĸ�ʽ��һ�¡���Ӧ���Ƕ������Ĳ������ŵĵط��ɡ�
�����ϱߵ����ݾͿ��Ը���t_sqool_ins_fields��ȡ����Ҫ�����ݣ�ƴ�ӳ��ַ�����������д��flume��ͨ��`org.apache.flume.api.RpcClient`�ཫ����д��flume��֮��flume�Ὣ����д��hdfs��

`CollectInfoRequestBean`��`toMap()`����ת�����ݿ��ֶθ�ʽ������post������"{create_time:2016-04-28}"����ᱻת��Ϊ"{createTime:2016-04-28}"��������Ϊwaf���ƴ����ʽ��������Ҫ�������ֹ��޸ġ�
## д��mysql ##
д��mysql�Ĳ�����д��flume�Ĳ������ƣ�������Լ򵥣�ʹ��JdbcTemplateʵ�����ݲ������ݿ⡣
