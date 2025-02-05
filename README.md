# Job-文件分发(GITHUB版)

# 配置
1. 插件配置  
插件上架时，需要配置蓝鲸智云相关参数，路径：设置->私有配置
- BK_APP_ID:应用ID，别名bk_app_code，需要使用已添加至蓝鲸ESB接口调用白名单中的应用ID，建议使用蓝鲸开发者中心内的蓝盾调用ESB接口专用应用ID（默认为bk_ci）
- BK_APP_SECRET: 应用ID对应的安全密钥(应用TOKEN)
- ESB_HOST: 蓝鲸ESB网关地址, 例如： http://paas.service.consul:80
- JOB_HOST: 蓝鲸JOB独立地址，例如： http://job.bktencent.com:80

蓝鲸社区版6.0用户可在蓝鲸中控机执行如下代码获取配置信息.
```shell script
source ${CTRL_DIR:-/data/install}/load_env.sh

echo "BK_APP_ID      $BK_CI_APP_CODE"
echo "BK_APP_SECRET  $BK_CI_APP_TOKEN"
echo "ESB_HOST       $BK_PAAS_PRIVATE_URL"
echo "JOB_HOST       $BK_JOB_PUBLIC_URL"

# 参考输出
BK_APP_ID      bk_ci
BK_APP_SECRET  略
ESB_HOST       http://paas.service.consul:80
JOB_HOST       http://job.bktencent.com:80
```
蓝鲸社区版7.0及以上用户可在蓝鲸K8S中控机上执行以下命令获取配置变量：
```shell
namespace="blueking"
ESB_HOST=$(helm -n $namespace get values bk-job --all|grep bkComponentApiUrl|awk -F": " '{print $2}')
BK_APP_ID=$(helm -n $namespace get values bk-job --all|grep appCode|awk -F": " '{print $2}')
BK_APP_SECRET=$(helm -n $namespace get values bk-job --all|grep appSecret|awk -F": " '{print $2}')
# 读取 bkDomainScheme
scheme=$(helm -n $namespace get values bk-job --all|yq e '.bkDomainScheme')
# 读取 job.web.domain
domain=$(helm -n $namespace get values bk-job --all|yq e '.job.web.domain')
# 拼接成完整的 URL
JOB_HOST="${scheme}://${domain}"
echo "BK_APP_ID=$BK_APP_ID"
echo "BK_APP_SECRET=$BK_APP_SECRET"
echo "ESB_HOST=$ESB_HOST"
echo "JOB_HOST=$JOB_HOST"
```

2.作业平台配置  
请将蓝鲸持续集成平台后台微服务artifactory所在机器IP全部加入至作业平台的IP白名单中，在作业平台的“平台管理->IP白名单”中进行配置，生效范围选择文件分发，若artifactory扩容，需将扩容机器IP更新至白名单中。

# 常见问题&解决方案
#### 1.权限问题
插件使用**流水线最后保存人**的身份去调用作业平台接口。
需要确保流水线最后保存人这个账号拥有作业平台的【快速分发文件】权限，且快速分发文件权限关联的实例包含插件中指定的所有主机、节点与动态分组（申请权限时建议关联实例资源选择对应业务下的任意资源以拥有将来新增资源的权限），若无权限可到蓝鲸权限中心进行申请。

#### 2.填写动态分组ID执行报错的问题
需要注意的是：由于不同版本作业平台的接口差异，若使用该插件对接蓝鲸6.x中的作业平台，需要在动态分组ID前加上"具体的业务ID:"，对接5.x中的作业平台则不需要。
