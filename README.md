# 教程：将 AWS Lambda 与 Amazon S3 结合使用

## 需求
假设您要为上传到存储桶的每个图像文件创建一个缩略图。您可以创建一个 Lambda 函数 (CreateThumbnailForShayne)，在创建对象后，Amazon S3 可调用该函数。之后，Lambda 函数可以从源存储桶读取图像对象并在目标存储桶中创建要保存的缩略图。

## 前提
linux 机器，装有nodejs，aws cli，并通过aws configure 配置过。


## 使用资源
Lambda 资源
* Lambda函数
* 与之关联的role

IAM 资源
* policy
* role

Amazon S3 资源
* shayne-chen
* shayne-chen-resized

## 步骤
### 1. 创建两个存储桶
一个名称为 shayne-chen，一个名称为 shayne-chen-resized。

shayne-chen 中有名为 HappyFace.jpg 的图片。


### 2. 创建 IAM policy
名为 AWSLambdaS3PolicyForShayne
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:*:*:*"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::shayne-chen/*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::shayne-chen-resized/*"
        }
    ]
}
```

### 3. 创建IAM Role
名为 shayne-lambda-s3-role。

Permissions policies 为第二步创建的 AWSLambdaS3PolicyForShayne。

Trusted entities 为 Lambda。

### 4. 创建部署程序包
（1）linux 机器上创建文件夹 lambda-s3

（2）在 lambda-s3 创建 index.js 文件

（3）进入 lambda-s3，运行如下命令
```
npm install sharp --unsafe-perm=true --allow-root
```
此时文件夹下会出现 node_modules 和 package-lock.json

再运行如下命令
```
zip -r function.zip .
```
再运行如下命令
```
aws lambda create-function --function-name CreateThumbnail --zip-file fileb://function.zip --handler index.handler --runtime nodejs12.x --timeout 10 --memory-size 1024 --role arn:aws:iam::875593617141:role/shayne-lambda-s3-role
```

(4) 测试
创建 inputFile.txt

执行如下命令
```
aws lambda invoke --function-name CreateThumbnailForShayne --invocation-type Event --payload file://inputFile.txt outputfile.txt
```

可以看到shayne-chen-resized中多了resized-HappyFace.jpg

