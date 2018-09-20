# Binder连接池

Binder连接池的必要性：当一个应用中存在很多个需要运用AIDL的模块，如果有10个功能模块需要运用AIDL，不可能写10个Service，Service是一种非常重量级的组建，也不可能把所有业务都写在一个AIDL中，不利于维护，所以出现了Binder连接池的概念：**为各个模块定义单独的AIDL和实现，同一有服务端进行管理，服务端提供queryBinder(in code)方法供客户端根据AIDL标识进行查询，并返回对应的AIDL实现。从而避免了创建多个Service**。



