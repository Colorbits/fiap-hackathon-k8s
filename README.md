aws user access:
eksFullAccess:
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "eks:*",
                "ec2:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "iam:ListRoles"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInstances",
                "ec2:DescribeRouteTables",
                "ec2:DescribeVpcs",
                "autoscaling:DescribeAutoScalingGroups"
            ],
            "Resource": "*"
        }
    ]
}

AmazonEC2ContainerRegistryReadOnly
AmazonEC2FullAccess
AmazonEKSClusterPolicy
AmazonEKSLoadBalancingPolicy
AmazonEKSNetworkingPolicy
AmazonEKSWorkerNodePolicy