# swr-login-workflow-samples
**本READEME指导是基于action: [Huawei Cloud Software Repository for Container (SWR) Login](https://github.com/marketplace/actions/huawei-cloud-software-repository-for-container-swr-login)登录使用华为容器镜像服务的workflows样例**    

## 前置条件
建议用户提前了解action输入参数和容器镜像服务的一些基础概念，以便迅速上手使用.swr-login-workflow-samples。
1. [管理IAM用户访问密钥(AK/SK)](https://support.huaweicloud.com/usermanual-iam/iam_02_0003.html)  
2. [华为云区域(region)](https://support.huaweicloud.com/iam_faq/iam_01_0011.html)  
3. 容器镜像服务SWR[创建组织](https://support.huaweicloud.com/usermanual-swr/swr_01_0014.html)和[授权管理](https://support.huaweicloud.com/usermanual-swr/swr_01_0072.html) 

## Action参数  
> 提示：下面参数标注 🔐 的参数属于敏感信息，建议在GitHub项目的setting--Secret--Actions下添加私密参数。比如添加参数ACCESSKEY，action参数使用为${{ secrets.ACCESSKEY }}。

| Name          | Sensitive | Require | Description |
| ------------- | ------- | ------- | ----------- |
| access-key-id    |   🔐    |   true      | 华为云访问密钥ID即AK,可以在[我的凭证](https://support.huaweicloud.com/usermanual-ca/ca_01_0003.html?utm_campaign=ua&utm_content=ca&utm_term=console)获取。|
| access-key-secret    |   🔐    |    true     | 华为云访问密钥即SK,可以在[我的凭证](https://support.huaweicloud.com/usermanual-ca/ca_01_0003.html?utm_campaign=ua&utm_content=ca&utm_term=console)获取。|
| region    |           |     true   | 华为云区域，可以在[我的凭证](https://console.huaweicloud.com/iam/?locale=zh-cn#/mine/apiCredential)获取。|

## 登录使用SWR的workflows参考样例
>登录使用SWR提供两种参考实例：  
1. 推送镜像到华为云SWR  
2. 拉取华为云SWR的镜像
>

### 1.推送镜像到华为云SWR
```yaml
name: Push Image to SWR Demo
on:
  push:
    branches:
       master
env:
  REGION_ID: '<region id>'   # example: cn-north-4
  SWR_ORGANIZATION:  '<swr_organization>'   # SWR 组织名
  IMAGE_NAME: '<image_name>'     # 镜像名称
jobs:
  swr-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Log in to Huawei Cloud SWR
        uses: huaweicloud/swr-login@v2.0.0
        with:
          access-key-id: ${{ secrets.ACCESSKEY }}
          access-key-secret: ${{ secrets.SECRETACCESSKEY }}
          region: ${{ env.REGION_ID }}

      - name: Build, Tag, and Push Image to Huawei Cloud SWR
        id: push-image
        env:
          SWR_REGISTRY: swr.${{ env.REGION_ID }}.myhuaweicloud.com
          SWR_ORGANIZATION: ${{ env.SWR_ORGANIZATION }}
          IMAGE_TAG: ${{ github.sha }} # 镜像版本,这里是使用代码commitid sha值， 用户可以修改成自己需要的。
          IMAGE_NAME: ${{ env.IMAGE_NAME }}
        run: |
          docker build -t $SWR_REGISTRY/$SWR_ORGANIZATION/$IMAGE_NAME:$IMAGE_TAG .
          docker push $SWR_REGISTRY/$SWR_ORGANIZATION/$IMAGE_NAME:$IMAGE_TAG
          echo "::set-output name=image::$SWR_REGISTRY/$SWR_ORGANIZATION/$IMAGE_NAME:$IMAGE_TAG"
```
详情可参考 [.github/workflows/push-image-to-swr-demo.yml](.github/workflows/push-image-to-swr-demo.yml)

### 2.拉取华为云SWR的镜像
> 1) 拉取SWR镜像时候，确保自己的SWR存在镜像  
> 2) SWR镜像地址格式:swr.<region id>.myhuaweicloud.com/<镜像组织名称>/<镜像名称>:<镜像版本>  
> 3) SWR镜像地址例子: **swr.cn-north-4.myhuaweicloud.com/test/image_name:v1.0.0**
```yaml
name: Pull Image from SWR Demo
on:
  push:
    branches:
       master
env:
  REGION_ID: '<region id>'   # example: cn-north-4
  IMAGE_URL: '<image url>'     # 镜像地址
jobs:
  swr-pull:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        
      - name: Log in to Huawei Cloud SWR
        uses: huaweicloud/swr-login@v2.0.0
        with:
          access-key-id: ${{ secrets.ACCESSKEY }}
          access-key-secret: ${{ secrets.SECRETACCESSKEY }}
          region: ${{ env.REGION_ID }}  

      - name: Pull Image from Huawei Cloud SWR
        id: pull-image
        run: |
          docker pull ${{ env.IMAGE_URL }}
```
详情可参考 [.github/workflows/pull-image-from-swr-demo.yml](.github/workflows/pull-image-from-swr-demo.yml)

## 公网域名说明
```
使用提供的Dockerfile模板制作镜像时，拉取基础镜像地址：'docker.io/library/alpine'
```