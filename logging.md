We will be deploying Fluentd as a DaemonSet, or one pod per worker node. The fluentd log daemon will collect logs and forward to CloudWatch Logs. This will require the nodes to have permissions to send logs and create log groups and log streams. This can be accomplished with an IAM user, IAM role, or by using a tool like Kube2IAM.

In our example, we will create an IAM policy and attach it the the Worker node role.

```
mkdir ~/environment/iam_policy
cat <<EoF > ~/environment/iam_policy/k8s-logs-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
EoF

aws iam put-role-policy --role-name $ROLE_NAME --policy-name Logs-Policy-For-Worker --policy-document file://~/environment/iam_policy/k8s-logs-policy.json
```

Deploy FluentD

```
mkdir ~/environment/fluentd
cd ~/environment/fluentd
wget https://eksworkshop.com/intermediate/230_logging/deploy.files/fluentd.yml
```

Update REGION and CLUSTER_NAME environment variables in fluentd.yml to the ones for your values. Currently, they are set to us-west-2 and eksworkshop-eksctl by default. Adjust this change in the ‘env’ section of the fluentd.yml file:

```
    env:
      - name: REGION
        value: us-west-2
      - name: CLUSTER_NAME
        value: eksworkshop-eksctl
```

